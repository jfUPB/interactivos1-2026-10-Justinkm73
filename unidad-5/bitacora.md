# Unidad 5
## Bitácora de proceso de aprendizaje


## Bitácora de aplicación
### DESARROLLO ACTIVIDAD #2
**Respuesta:**

```
#Código microbit -> Código brindado por el docente.

from microbit import *
import struct

uart.init(115200)
display.set_pixel(0, 0, 9)

while True:
    xValue = accelerometer.get_x()
    yValue = accelerometer.get_y()
    aState = button_a.is_pressed()
    bState = button_b.is_pressed()
    data = struct.pack('>2h2B', xValue, yValue, int(aState), int(bState))
    checksum = sum(data) % 256
    packet = b'\xAA' + data + bytes([checksum])
    uart.write(packet)
    sleep(100)
```

```
// MicrobitBinaryAdapter.js

const { SerialPort } = require("serialport");
const BaseAdapter = require("./BaseAdapter");

//Marca el inicio de cada paquete, necesario para que nuestro receptor lo busque y pueda iniciar la trama
const HEADER = 0xAA
// Tamañao del paquete de bytes que recibiremos en realidad son 6 los valores que vamos a recibri.
// Los otros dos valores son el incio de trama 0xAA y el ultimo valor es el checksum
// Es necesario que lea los 8 valores para separar las tramas y para validar con el checksum
const PACKET_SIZE = 8;


// Acá no usamos está función de parseo porque el protocolo binario ya viene estructurado en posiciones fijas. No hay texto que dividir ni etiquetas que extraer.

/*class ParseError extends Error { }

function parseCsvLine(line) {
  const values = line.trim().split(",");
  if (values.length !== 4) throw new ParseError(`Expected 4 values, got ${values.length}`);

  const x = Number(values[0]);
  const y = Number(values[1]);
  const btnA = String(values[2]).trim().toLowerCase();
  const btnB = String(values[3]).trim().toLowerCase();

  if (!Number.isFinite(x) || !Number.isFinite(y)) throw new ParseError("Invalid numeric data");
  if (x < -2048 || x > 2047 || y < -2048 || y > 2047) throw new ParseError("Out of expected range");
  if (!["true", "false"].includes(btnA) || !["true", "false"].includes(btnB)) throw new ParseError("Invalid button data");

  return { x: x | 0, y: y | 0, btnA: btnA === "true", btnB: btnB === "true" };
}*/


class MicrobitBinaryAdapter extends BaseAdapter {
  constructor({ path, baud = 115200, verbose = false } = {}) {
    super();
    this.path = path;
    this.baud = baud;
    this.port = null;
    // A diferencia del ASCII adapter que usa un string,
    // acá acumulamos bytes en un Buffer de Node.js
    //this.buf = ""; -> Buffer anterior -> es un string vacío. Acumula texto, caracteres.
    this.buf = Buffer.alloc (0)
    //Es un Buffer vacío de Node.js. Acumula bytes crudos.
    //Crea un buffer vacío (0 bytes reservados) listo para ir acumulando bytes a medida que llegan por el puerto serial
    //.alloc → "alloc" viene de allocate (asignar/reservar). Es decir, reserva espacio en memoria.
    this.verbose = verbose;
  }

  async connect() {
    if (this.connected) return;
    if (!this.path) throw new Error("serialPort is required for microbit device mode");

    this.port = new SerialPort({
      path: this.path,
      baudRate: this.baud,
      autoOpen: false,
    });

    await new Promise((resolve, reject) => {
      this.port.open((err) => (err ? reject(err) : resolve()));
    });

    this.connected = true;
    this.onConnected?.(`serial open ${this.path} @${this.baud}`);

    this.port.on("data", (chunk) => this._onChunk(chunk));
    this.port.on("error", (err) => this._fail(err));
    this.port.on("close", () => this._closed());
  }

  async disconnect() {
    if (!this.connected) return;
    this.connected = false;

    if (this.port && this.port.isOpen) {
      await new Promise((resolve, reject) => {
        this.port.close((err) => {
          if (err) reject(err);
          else resolve();
        });
      });
    }
    this.port = null;
    //this.buf = ""; -> Buffer anterior -> es un string vacío. Acumula texto, caracteres.
    this.buf = Buffer.alloc(0);
    /* acá nos indica que si el microbit esta enviando datos y se rompe la conexión
    el quedaría con datos, entonces acá le decimos que cuando haya una nueva conexión
    el vuelva a tener 0 datos */
    this.onDisconnected?.("serial closed");
  }

  getConnectionDetail() {
    return `serial open ${this.path}`;
  }

  _onChunk(chunk) {
    // Acumulamos los bytes nuevos al buffer existente
    this.buf = Buffer.concat([this.buf, chunk]);
 
    // Procesamos todos los paquetes completos que haya en el buffer
    while (true) {
 
      // PASO 1: Buscar el header 0xAA
      const headerIdx = this.buf.indexOf(HEADER);
 
      // Si no hay header en el buffer, lo descartamos todo
      if (headerIdx === -1) {
        this.buf = Buffer.alloc(0);
        break;
      }
 
      // Si el header no está al inicio, descartamos los bytes basura anteriores
      if (headerIdx > 0) {
        this.buf = this.buf.slice(headerIdx);
      }
 
      // PASO 2: Esperar a tener al menos 8 bytes desde el header (const PACKET_SIZE = 8; -> es la condición que se debe de cumplir 8)
      if (this.buf.length < PACKET_SIZE) break; // esperamos más bytes
 
      // PASO 3: Extraer el paquete candidato
      const packet = this.buf.slice(0, PACKET_SIZE);
 
      // PASO 4: Verificar el checksum
      // Sumamos los bytes 1 al 6 y le sacamos % 256
      let sum = 0;
      for (let i = 1; i <= 6; i++) {
        sum += packet[i];
      }
      const calculatedChecksum = sum % 256;
      const receivedChecksum = packet[7];
 
      if (calculatedChecksum !== receivedChecksum) {
        // Checksum inválido: descartamos SOLO el byte 0xAA y seguimos buscando
        // (era un falso positivo — un byte de datos que valía 0xAA)
        console.warn(
          `[MicrobitBinaryAdapter] Trama corrupta descartada. CHK esperado: ${calculatedChecksum}, recibido: ${receivedChecksum}`
        );
        this.buf = this.buf.slice(1);
        continue; // volvemos a buscar el próximo 0xAA
      }
 
      // PASO 5: Paquete válido — Aquí parseamos los valores
      // readInt16BE lee 2 bytes como entero con signo en big-endian
      const x    = packet.readInt16BE(1);  // bytes 1-2
      const y    = packet.readInt16BE(3);  // bytes 3-4
      const btnA = packet.readUInt8(5) === 1; // byte 5
      const btnB = packet.readUInt8(6) === 1; // byte 6
 
      // Emitir el mismo contrato que los otros adapters
      this.onData?.({ x, y, btnA, btnB });
 
      // Consumir el paquete del buffer y seguir con el resto
      this.buf = this.buf.slice(PACKET_SIZE);
    }
 
    // Protección: si el buffer crece demasiado, lo reseteamos
    /* el while(true) del _onChunk solo sale del loop cuando:
        1. No encuentra 0xAA
        2. No tiene 8 bytes suficientes
        3. Procesa el paquete y sigue */

    /* El 4096 es simplemente un límite arbitrario.
    Si el buffer llegó a 4096 sin poder procesar nada, algo está muy mal,
    el micro:bit probablemente está enviando pura basura. En ese punto es
    más seguro borrar todo y empezar de cero que seguir acumulando. */
    if (this.buf.length > 4096) {
      this.buf = Buffer.alloc(0);
      /* el buffer en _onChunk se limpia como protección si acumula demasiada
      basura sin poder procesar nada (los 4096 bytes)*/
    }
  }    

  _fail(err) {
    this.onError?.(String(err?.message || err));
    this.disconnect();
  }

  _closed() {
    if (!this.connected) return;
    this.connected = false;
    this.port = null;
    /*el buffer en _closed se limpia porque la conexión se cayó inesperadamente,
    y no queremos que esos bytes queden para la próxima conexión*/
    this.buf = Buffer.alloc(0);
    this.onDisconnected?.("serial closed (event)");
  }

  async writeLine(line) {
    if (!this.port || !this.port.isOpen) return;
    await new Promise((resolve, reject) => {
      this.port.write(line, (err) => (err ? reject(err) : resolve()));
    });
  }

  async handleCommand(cmd) {
    if (cmd?.cmd === "setLed") {
      const x = Math.max(0, Math.min(4, Math.trunc(cmd.x)));
      const y = Math.max(0, Math.min(4, Math.trunc(cmd.y)));
      const v = Math.max(0, Math.min(9, Math.trunc(cmd.value)));
      await this.writeLine(`LED,${x},${y},${v}\n`);
    }
  }
}

module.exports = MicrobitBinaryAdapter;
```

```
//bridgeServer.js

//   Uso:
//     node bridgeServer.js --device sim --wsPort 8081 --hz 30
//     node bridgeServer.js --device microbit --wsPort 8081 --serialPort COM5 --baud 115200

//   WS contract:
//    * bridge To client:
//        {type:"status", state:"ready|connected|disconnected|error", detail:"..."}
//        {type:"microbit", x:int, y:int, btnA:bool, btnB:bool, t:ms}
//    * client To bridge:
//        {cmd:"connect"} | {cmd:"disconnect"}
//        {cmd:"setSimHz", hz:30}
//        {cmd:"setLed", x:2, y:3, value:9}


const { WebSocketServer } = require("ws");
const { SerialPort } = require("serialport");
const SimAdapter = require("./adapters/SimAdapter");
const MicrobitAsciiAdapter = require("./adapters/MicrobitASCIIAdapter");
// Descomentamos nuestro adapterv2 - En ASCII
const MicrobitV2Adapter = require("./adapters/MicrobitV2Adapter");
// Agregamos nuestro adapter para datos Binary
const MicrobitBinaryAdapter = require("./adapters/MicrobitBinaryAdapter");

const log = {
  info: (...args) => console.log(`[${new Date().toISOString()}] [INFO]`, ...args),
  warn: (...args) => console.warn(`[${new Date().toISOString()}] [WARN]`, ...args),
  error: (...args) => console.error(`[${new Date().toISOString()}] [ERROR]`, ...args)
};


function getArg(name, def = null) {
  const i = process.argv.indexOf(`--${name}`);
  if (i >= 0 && i + 1 < process.argv.length) return process.argv[i + 1];
  return def;
}

function hasFlag(name) {
  return process.argv.includes(`--${name}`);
}

function nowMs() { return Date.now(); }

function safeJsonParse(s) {
  try {
    return JSON.parse(s);

  } catch (e) {
    log.warn("Failed to parse JSON: ", s, e);
    return null;
  }
}

function broadcast(wss, obj) {
  const text = JSON.stringify(obj);
  for (const client of wss.clients) {
    if (client.readyState === 1) client.send(text);
  }
}

function status(wss, state, detail = "") {
  broadcast(wss, { type: "status", state, detail, t: nowMs() });
}

const DEVICE = (getArg("device", "sim") || "sim").toLowerCase();
const WS_PORT = parseInt(getArg("wsPort", "8081"), 10);
const SERIAL_PATH = getArg("serialPort", null);
const BAUD = parseInt(getArg("baud", "115200"), 10);
const SIM_HZ = parseInt(getArg("hz", "30"), 10);
const VERBOSE = hasFlag("verbose");

async function findMicrobitPort() {
  const ports = await SerialPort.list();
  const microbit = ports.find(p =>
    p.vendorId && parseInt(p.vendorId, 16) === 0x0D28
  );
  return microbit?.path ?? null;
}

// Cuando inicio el servidor, diciendole `$ node bridgeServer.js y luego -—device microbit-bin ó -—device microbit-bin-2  le estoy diciendo que adapter instanciar
    // --device es simplemente el argumento que le pasas por consola al arrancar el servidor.
    // --device no es nada especial de Node.js ni de JavaScript — tú lo inventas. Podría llamarse --hardware, --tipo, lo que quieras.
    // Levantamos nuestro servidor escribiendo el --device asi, por que en la const DEVICE = (getArg("device", "sim") busca --device en la terminal y guarda su valor en la variable DEVICE. El nombre de la variable puede ser cualquier cosa
    // Lo importante es que el string que escribes en la consola coincida exactamente con el que comparas en el if. 

// En la función `createAdapter`(Creamos nuestro adapter) if (DEVICE ==="microbit-bin") -> MicrobitV2Adapter
// instanciar el adapter que usare, en la siguiente linea: return new MicrobitV2Adapter({ path, baud: BAUD, verbose: VERBOSE });

// En la función `createAdapter`(Creamos nuestro adapter) if (DEVICE ==="microbit-bin-2") -> MicrobitBinaryAdapter
// instanciar el adapter que usare, en la siguiente linea: return new MicrobitBinaryAdapter({ path, baud: BAUD, verbose: VERBOSE });

async function createAdapter() {
  if (DEVICE === "microbit") {
    const path = SERIAL_PATH ?? await findMicrobitPort();
    if (!path) {
      log.error("micro:bit not found. Use --serialPort to specify manually.");
      process.exit(1);
    }
    log.info(`micro:bit found at ${path}`);   
    return new MicrobitAsciiAdapter({ path, baud: BAUD, verbose: VERBOSE });
  }

// Escribimos nuestro nuevo adapter
  if (DEVICE === "microbit-bin") { // OJO! Cuando levantemos el servidor instanciamos con el nombre que le pusimos acá, no con el del archivo .js
    const path = SERIAL_PATH ?? await findMicrobitPort();
    if (!path) {
      log.error("micro:bit not found. Use --serialPort to specify manually.");
      process.exit(1);
    }
    log.info(`micro:bit found at ${path}`);
// Acá es donde instanciamos el nuevo adapter que cree    
    return new MicrobitV2Adapter({ path, baud: BAUD, verbose: VERBOSE });
  }

// Descomentamos nuestro nuevo adapter, el docente lo había puesto.
  if (DEVICE === "microbit-bin-2") { // OJO! Cuando levantemos el servidor instanciamos con el nombre que le pusimos acá, no con el del archivo .js
    const path = SERIAL_PATH ?? await findMicrobitPort();
    if (!path) {
      log.error("micro:bit not found. Use --serialPort to specify manually.");
      process.exit(1);
    }
    log.info(`micro:bit found at ${path}`);
// Acá es donde instanciamos el nuevo adapter que cree    
    return new MicrobitBinaryAdapter({ path, baud: BAUD, verbose: VERBOSE });
  }

  return new SimAdapter({ hz: SIM_HZ });
}

async function main() {
  const wss = new WebSocketServer({ port: WS_PORT });
  log.info(`WS listening on ws://127.0.0.1:${WS_PORT} device=${DEVICE}`);

  const adapter = await createAdapter();

  adapter.onConnected = (detail) => {
    log.info(`[ADAPTER] Device Connected: ${detail}`);
    status(wss, "connected", detail);
  };

  adapter.onDisconnected = (detail) => {
    log.warn(`[ADAPTER] Device Disconnected: ${detail}`);
    status(wss, "disconnected", detail);
  };

  adapter.onError = (detail) => {
    log.error(`[ADAPTER] Device Error: ${detail}`);
    status(wss, "error", detail);
  };

  adapter.onData = (d) => {
    broadcast(wss, {
      type: "microbit",
      x: d.x,
      y: d.y,
      btnA: !!d.btnA,
      btnB: !!d.btnB,
      t: nowMs(), // ← usa el tiempo del servidor, no el del micro:bit
    });
  };

  status(wss, "ready", `bridge up (${DEVICE})`);

  wss.on("connection", (ws, req) => {
    log.info(`[NETWORK] Remote Client connected from ${req.socket.remoteAddress}. Total clients: ${wss.clients.size}`);

    const state = adapter.connected ? "connected" : "ready";

    const detail = adapter.connected
      ? adapter.getConnectionDetail()
      : `bridge (${DEVICE})`;

    ws.send(JSON.stringify({ type: "status", state, detail, t: nowMs() }));

    ws.on("message", async (raw) => {
      const msg = safeJsonParse(raw.toString("utf8"));
      if (!msg) return;

      if (msg.cmd === "connect") {
        log.info(`[NETWORK] Client requested adapter connect`);

        if (adapter.connected) {
          log.info(`[HW-POLICY] Adapter already open. Sending current status to incoming client.`);
          ws.send(JSON.stringify({ type: "status", state: "connected", detail: adapter.getConnectionDetail(), t: nowMs() }));
          return;
        }
        
        try {
          await adapter.connect();
        } catch (e) {
          const detail = `connect failed: ${e.message || e}`;
          log.error(`[ADAPTER] ` + detail);
          status(wss, "error", detail);
        }
        return;
      }

      if (msg.cmd === "disconnect") {
        log.info(`[NETWORK] Client requested adapter disconnect`);
        if (wss.clients.size > 1) {
          log.info(`[HW-POLICY] Adapater kept open. Shared with ${wss.clients.size - 1} other active client(s).`);
          ws.send(JSON.stringify({ type: "status", state: "disconnected", detail: "logical disconnect only", t: nowMs() }));
          return;
        }
        
        try {
          await adapter.disconnect();
        } catch (e) {
          const detail = `disconnect failed: ${e.message || e}`;
          log.error(`[ADAPTER] ` + detail);
          status(wss, "error", detail);
        }
        return;
      }

      if (msg.cmd === "setSimHz" && adapter instanceof SimAdapter) {
        log.info(`Setting Sim Hz to ${msg.hz}`);
        await adapter.handleCommand(msg);
        status(wss, "connected", `sim hz=${adapter.hz}`);
        return;
      }

      if (msg.cmd === "setLed") {
        try {
          await adapter.handleCommand?.(msg);
        } catch (e) {
          const detail = `command failed: ${e.message || e}`;
          log.error(`[ADAPTER] ` + detail);
          status(wss, "error", detail);
        }
        return;
      }
    });

    ws.on("close", () => {
      log.info(`[NETWORK] Remote Client disconnected. Total clients left: ${wss.clients.size}`);
      if (wss.clients.size === 0) {
        log.info("[HW-POLICY] No more remote clients. Auto-disconnecting adapter device to free resources...");
        adapter.disconnect();
      }
    });
  });

  if (DEVICE === "sim") {
    await adapter.connect();
  }
}

main().catch((e) => {
  log.error("Fatal:", e);
  process.exit(1);
});
```

**NOTA:**

---

**¿Qué es un byte?**

Un byte es simplemente 8 bits, puede guardar un número del 0 al 255. Nada más. Si tu número es más grande, necesitas más de un byte.



**El problema de los 2 bytes**

El número `500` no cabe en 1 byte (máximo 255). Necesita 2 bytes. En hexadecimal `500` se escribe como `01 F4`. Ahí tienes dos "pedazos":

- `01` → la parte del valor grande
- `F4` → la parte del valor pequeño

La pregunta es: **¿en qué orden los mando por el cable?**


**Big-Endian**

"Big" = el pedazo grande va primero.

`Número 500 → se envía:  01  F4
                        │   │
                      grande pequeño`

Es como escribir un número normal: de izquierda a derecha, el dígito más significativo primero. Como nosotros leemos.



**Little-Endian**

"Little" = el pedazo pequeño va primero, al revés.

`Número 500 → se envía:  F4  01
                        │   │
                      pequeño grande`

Suena raro, pero muchos procesadores (como el de tu PC) trabajan así internamente.


**¿Por qué importa esto?**

Si el micro:bit envía `01 F4` (Big-Endian) pero Node.js lo lee al revés como Little-Endian, obtendría `F4 01` = **63489**. Un número completamente incorrecto. Por eso debes saber cuál usa el dispositivo que envía.

El micro:bit usa `struct.pack('>2h...')` — ese `>` significa Big-Endian. Entonces en Node.js usas `readInt16BE` para leer igual.

El micro:bit es quien **envía** los datos, entonces él define las reglas. Node.js solo recibe, así que tiene que adaptarse a lo que el micro:bit decidió.

Si en Node.js hubieras usado `readInt16LE` (Little-Endian), estarías "leyendo en otro idioma" y los números saldrían incorrectos.


**Ahora, ¿por qué cada función para cada variable?**

**`readInt16BE` para X y Y**

- `Int` → con signo, puede ser **negativo**. El acelerómetro va de -2048 a 2047, necesita negativos.
- `16` → lee **2 bytes** (16 bits). Necesario porque -2048 a 2047 no cabe en 1 byte.
- `BE` → Big-Endian, porque así lo envía el micro:bit.

**`readUInt8` para btnA y btnB**

- `U` → sin signo, solo **positivo**. Un botón solo vale 0 o 1, nunca negativo.
- `Int8` → lee **1 solo byte**. Con 1 byte sobra para guardar 0 o 1.
- No necesita `BE` o `LE` porque con 1 solo byte no hay "orden" que definir — es un único byte.



**En resumen:**

| Variable | Función | Por qué |
| --- | --- | --- |
| x, y | `readInt16BE` | Necesitan 2 bytes y pueden ser negativos |
| btnA, btnB | `readUInt8` | 1 byte basta, nunca negativos |

---
**Evidencia validación CHECKSUM**




<img width="566" height="364" alt="Captura de pantalla 2026-03-27 a la(s) 3 06 04 p m" src="https://github.com/user-attachments/assets/4a8bbb0a-9bc9-4ec5-ae47-3255774727ee" />




---
## Bitácora de reflexión

### DESARROLLO ACTIVIDAD #3
**Respuesta:**

**1.** **Cuadro comparativo:**

**a)** *Tamaño de cada paquete en bytes*

`MicrobitV2Adapter.js` = El paquete tiene tamaño **variable**. (En nuestro ejemplo es de 30 bytes pero es viarable) Depende de los valores que recibamos puede ser menos o más, el tiempo varía según el largo del string. Es un problema en procesamiento, ya que mientras hace el parseo de un paquete que es más largo que el que viene, lo retarda y para cuando termine, el otro paquete que es más corto ha llegado y se acumulan en el buffer. El paquete no esta completo hasta que no aparezca el salto de linea \n

`MicrobitBinaryAdapter.js` = El paquete tiene tamaño **fijo de exactamente 8 bytes**, por la códificación binaria podemos resumir la información de un paquete en 8 bytes y ya no en 30 o hasta 40 bytes en nuestros ejemplos, optimizando memoria.



**b)** *Mecanismo de delimitación/framing.*

`MicrobitV2Adapter.js` = usa `$` para el inicio de la trama y el `\n` como delimitador de fin de trama. El receptor simplemente espera el salto de línea para saber que la trama terminó.

`$T:123|X:45|Y:67|A:0|B:1|CHK:AB\n` — cada carácter como 1 byte en ASCII.

Sin framing serían solo los valores: `123 45 67 0 1`
Con framing se incluye: `$`, `T:`, `|X:`, `|Y:`, `|A:`, `|B:`, `|CHK:`, `\n`

`MicrobitBinaryAdapter.js` = usa el byte `0xAA` como header de inicio de trama y tiene un checksum para verificar integridad. Sin framing serían solo 6 bytes (Valores de las variables)



**c)** *Mecanismo de verificación de integridad (checksum).*

`MicrobitV2Adapter.js` = suma los valores absolutos de X, Y, A y B

`MicrobitBinaryAdapter.js` = suma los bytes en posiciones 1 a 6 y aplica módulo 256. El % 256 existe porque el resultado debe caber en exactamente 1 byte (0–255). y luego compara el byte 7 para verificar si coinciden los valores sobrantes, si es así, la trama es correcta. El byte inicial no se suma por que es el inicio de la trama



**d)** *Complejidad de implementación del parser.*

`MicrobitV2Adapter.js` = el parser trabaja con texto. Los pasos son: dividir por `|`, separar etiqueta de valor con `:`, convertir strings a números, validar rangos. Más pasos, más conversiones, más superficie de error de parsing.

`MicrobitBinaryAdapter.js` = Trabaja con bytes crudos. Requiere conocer: 

— **Big-endian**: el byte más significativo va primero. (Depende de como lo comuniquemos desde nuestra microbit, puede ser al reves tambien)

— `readInt16BE(1)`: lee 2 bytes como entero con signo.

— `readUInt8(5)`: lee 1 byte sin signo.

— Complemento a dos para valores negativos de X e Y.



**e)** *Facilidad de depuración (¿cuál es más fácil de leer con un terminal serial?).*

`MicrobitV2Adapter.js` = Abrir cualquier terminal serial y leer directamente.

`MicrobitBinaryAdapter.js` = En terminal serial se ven caracteres sin sentido. Necesitamos un analizador que muestre hex (https://juanferfranco.github.io/serialTerminal/)

---



**2.** Puedo añadir un soporte para cada adapter ya que todos manejan el mismo contrato
`this.onData?.({ x, y, btnA, btnB })` 

Uno lo construye parseando texto con pipes (ASCII) y el otro leyendo bytes con `readInt16BE`, (BINARY) pero el resultado que sale es idéntico. Valor de x, y, btnA, btnB. El brigdServer no sabe que tipo de adapter o protocolo de comunicación estamos usando el solo recibe el objeto y lo envía por WebSocket.

El frontend (`bridgeClient.js`) recibe un JSON (Es una forma muy simple y ordenada de guardar y enviar datos.) (`{ type: "microbit", x, y, btnA, btnB }`) y dispara el evento `EVENTS.DATA` en  `sketch.js` y este a su vez en `updateLogic` lee los valores, como apesar de que se use otro adapter el contrato es el mismo para todos no presentara problemas por que leera son los valores que son los mismos para todos.

En conclusión no importa que adapter use lo que importa es los valores que estoy enviando y los que estoy recibiendo. Todos los adapters enviaran la misma información que soliciete el cliente sin importa que protocolo usemos.

`sketch.js` y `bridgeClient.js` nunca ven el protocolo crudo del hardware. No saben si llegaron bytes binarios o texto con pipes. Solo reciben el objeto ya limpio `{ x, y, btnA, btnB }` 

**NOTAS:**
En este sistema, **el frontend** es todo lo que corre en el navegador:

- `sketch.js` — la lógica de la FSM y el dibujo en p5.js
- `bridgeClient.js` — el que recibe los datos por WebSocket
- `index.html` — la página que los carga

Se le llama "frontend" porque es la **capa de cara al usuario**, la que muestra algo en pantalla. Vive en el navegador, no en Node.js.

En contraste, el **backend** es lo que corre en Node.js en tu computador:

- `bridgeServer.js` — el servidor WebSocket
- Los adapters (`MicrobitV2Adapter.js`, `MicrobitBinaryAdapter.js`) — los que leen el puerto serial


---

**3. Ejemplo 1 — Instalación de luz reactiva al kick de música Techno:**

Tengo un proyecto donde el BPM del kick activa y controla una luz mediante un estado de 1 (encendido) o 0 (apagado). Como el kick corre a 135 BPM, hay un cambio de estado constante y a alta frecuencia. Si intentara enviar esa información con ASCII, el volumen de datos podría saturar el canal y colapsar el hardware.

Por eso, **primero prototiparía con ASCII**: reduciría el tempo a un golpe cada 30 segundos para verificar en la terminal serial que el valor enviado es correctamente 0 o 1. Una vez confirmado, **implementaría el protocolo binario** para la ejecución real, ya que empaqueta los datos de forma compacta y permite sostener el ritmo 4/4 completo con cambios de BPM sin colapsar el sistema.


**Ejemplo 2 — Objeto interactivo musical con sensores de proximidad:**

Tengo una experiencia interactiva donde un objeto físico tiene sensores de proximidad que, según su valor, activan sonidos en el hardware.

**Primero prototiparía con ASCII**: con un solo sensor activo, verificaría en la terminal serial que los valores de proximidad que estoy enviando son correctos y tienen sentido antes de avanzar.

Una vez validado, **escalaría a binario**: al objeto se le añaden múltiples sensores enviando datos de proximidad simultáneamente. Con ASCII eso podría saturar el canal, pero empaquetando los datos en binario, toda esa información viaja de forma compacta y el hardware no colapsa cuando todos los sensores están activos al mismo tiempo.



**Conclusión:**
ASCII para prototipar y verificar, porque es legible directamente en la terminal serial. Binario para la ejecución real cuando el volumen y la frecuencia de datos es alto y complejo.

---

**4. DIAGRAMA DE FLUJO
a) PROTOCOLO DE COMUNICACIÓN BINARY**

```
@startuml
<style>
root {
  BackgroundColor #f7f9fc
  FontColor #2d3748
  LineColor #718096
  Margin 20
  Padding 10
}
sequenceDiagram {
  arrow {
    LineColor #4a5568
    FontColor #4a5568
  }
  participant {
    BackgroundColor #edf2f7
    LineColor #4a5568
    FontColor #2d3748
    RoundCorner 8
  }
  note {
    BackgroundColor #fefcbf
    LineColor #d69e2e
    FontColor #744210
  }
}
</style>

title Flujo de Datos — Unidad 5 (Protocolo Binario)

participant "micro:bit\n(firmware Binario)" as MB
participant "MicrobitBinaryAdapter\n(Node.js) ★ NUEVO" as ADAPTER
participant "bridgeServer\n(Node.js)" as SERVER
participant "bridgeClient\n(Browser)" as CLIENT
participant "PainterTask FSM\n(sketch.js)" as FSM
participant "Canvas\n(p5.js)" as CANVAS

MB -> ADAPTER : ""AA | 01 F4 | 02 0C | 01 | 00 | FE""\n〔Binario · tamaño FIJO · siempre 8 bytes〕

note over ADAPTER #ffe0e0
  ★ CAMBIA respecto a U4:
  1. Acumula bytes en Buffer (no string)
  2. Busca header 0xAA (no \n)
  3. Espera exactamente 8 bytes
  4. Valida CHK = (bytes 1–6) % 256
  5. Parsea con readInt16BE() (Big-Endian)
  6. Descarta solo el 0xAA si CHK inválido
end note

ADAPTER -> SERVER : { x, y, btnA, btnB }\n〔contrato idéntico a U4〕

note over SERVER #e0ffe0
  ✓ SIN CAMBIOS
  Mismo empaquetado JSON
  Mismo WebSocket
end note

SERVER -> CLIENT : WebSocket JSON\n""{ type:"microbit", x, y, btnA, btnB, t }""

note over CLIENT #e0ffe0
  ✓ SIN CAMBIOS
  bridgeClient.js intacto
end note

CLIENT -> FSM : postEvent\n""{ type: EVENTS.DATA, payload }""

note over FSM #e0ffe0
  ✓ SIN CAMBIOS
  sketch.js, fsm.js intactos
  updateLogic() igual
end note

FSM -> CANVAS : drawRunning()\nDibuja polígono si btnA
@enduml
```
