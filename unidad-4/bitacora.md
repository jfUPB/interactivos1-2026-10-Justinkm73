# Unidad 4
---
## Bitácora de proceso de aprendizaje
### DESARROLLO ACTIVIDAD #1
### SOLUCIÓN:
#### Explicación:

```
from microbit import *

uart.init(115200)
display.set_pixel(0,0,9)

while True:
    xValue = accelerometer.get_x() #Envía datos
    yValue = accelerometer.get_y() #Envía datos
    aState = button_a.is_pressed() #Envía datos
    bState = button_b.is_pressed() #Envía datos
    data = "{},{},{},{}\n".format(xValue, yValue, aState,bState) #Empaqueta información
    uart.write(data)
    sleep(100) #Envia datos a 10 Hz, cada 10 ms, enviaremos información

#el formato es tipo serial ASCII, viene códificado, el transmite caracteres
#En ASCII hay que cambiar esto: \n (Caracter de fin de trama)
```

**https://juanferfranco.github.io/serialTerminal/** (Está página muestra que datos me brindan cuando se lo cargo al micro:bit, que tipo de codificación estamos usando). 


**Aquí hay varias pistas que te dicen que es ASCII:**


 **1. El formato de la cadena** — está construyendo un texto con comas y números legibles, por ejemplo: `45,-12,True,False\n`. Eso es texto plano, no binario.

**2. El `\n` al final** — es un carácter de "nueva línea", típico de protocolos de texto ASCII. En binario puro no harías eso.

La diferencia clave es: en **ASCII** el número `45` viaja como los caracteres `'4'` y `'5'` (2 bytes), mientras que en **binario** viajaría como un solo byte con el valor `00101101`. El fabricante del dispositivo siempre te dice qué protocolo usa, como mencionan en los apuntes.

ASCII es un protocolo de comunicación en la programación, como vienen codificado los códigos

Luego de mirar que datos estoy enviando, me doy cuenta que es formato ASCCI según me muestra el programa, luego tengo que ir a una tabla ASCII para saber que información estoy enviando, ya que recordemos viene codificado, esos datos son bytes, pero tengo que saber como están códificados.


**¿Cómo distinguirlo en general?**

Si abres un **Serial Terminal** y ves cosas como `45,-12,True,False` → es **ASCII**, porque puedes leerlo como humano.

Si en cambio ves símbolos raros, caracteres extraños o números sin sentido → probablemente es **binario**, donde cada byte representa un valor directamente, no su representación en texto.

https://www.asciitable.com/ (Acá esta la tabla ASCII) para saber que información se está enviando

#### Como levantar un servidor:

Luego de tener el repositorio copiado en descargas o en el pc, vamos a instalar https://nodejs.org/en/download por que el navegador no puede hablar directamente con el micro:bit. El navegador vive en una "burbuja de seguridad" — por diseño no tiene acceso directo al puerto serial (USB) de tu computador, porque sería un riesgo de seguridad enorme que cualquier página web pudiera leer tus dispositivos.

   ```
   micro:bit (USB/Serial)
        ↓
   Node.js - bridgeServer.js   ← aquí está el "puente"
        ↓
   Navegador - p5.js sketch
   ```

Node.js puede, acceder al puerto serial del computador, leer los datos del micro:bit y pasarlos al navegador vía WebSocket. *Necesito instalar unas dependencias desde la terminal, para node.js*

$ npm install -> Abro la terminal y escribo esto, ejecutando la terminal desde la carpeta, escribo -> node.js (doy enter) -> $ npm install (doy enter)

Levantamos el servidor, luego de tener instalado las extenciones:

1. Abro la terminal desde la carpeta de caso de estudio
2. Escribo $ nodeb -> presiono tabulador y me lo completa $ nodebridge -> luego le agrego la S -> $ node bridgeS -> y doy tabulador para que se complete sola $ node bridgeServer.js, aquí el servidor empieza a correr.
3. En caso de querer pararlo presionar cmd + c.

#### Profundización:

En la programación orientada a objetos, cuando uno declara una interfaz, esta declarando muchos metodos.


<img width="1140" height="560" alt="image" src="https://github.com/user-attachments/assets/4b853814-279e-4894-b82e-e703c38a7991" />


Este patrón de diseño llamado estrategy, lo vamos a implementar, puedo utilizar metodos que me facilitan las clases, y en vez de aprender con cada clase, más bien lo definimos en una interfaz generica que nos permite interactuar con cualquier de las tres, este patron lo tenemos en un archivo que se llama **BaseAdapter**.

El BaseAdapter va conectado al server y los va a transformando, no importa de donde venga, el BaseAdapter le pasa la información al server como la necesite. Luego el server enviara datos normalizados y llegaran a una interfaz (Cliente). Este se encargara de comunicarse con el servidor. En este casó para el caso de estudio que son unas visuales necesitaremos unos archivos que son fsm.js (maquina de estados), sketch.js (Implementa toda la función gráfica) y el index cambiara información con el server y el server con el p5.js y asi sucesivamente entre archivos.

<img width="1600" height="775" alt="image (1)" src="https://github.com/user-attachments/assets/908c80c6-3a01-4c87-a499-51256a19ca05" />




---

## Bitácora de aplicación 
### DESARROLLO ACTIVIDAD #2
#### SOLUCIÓN:


**Diagrama de flujo de la activdad**
```
┌─────────────────────────────────────────────────────┐
│  1. MicrobitV2yAdapter.js  (CREAR desde cero)    │
│     → Parsea la trama $T:|X:|Y:|A:|B:|CHK:          │
│     → Valida el CHK                                 │
│     → Emite onData({ x, y, btnA, btnB })            │
└─────────────────────┬───────────────────────────────┘
                      │ onData
                      ↓
┌─────────────────────────────────────────────────────┐
│  2. bridgeServer.js  (MODIFICACIÓN MÍNIMA)          │
│     → Descomentar el require del adapter            │
│     → Agregar el if (DEVICE === "microbit-bin")     │
└─────────────────────┬───────────────────────────────┘
                      │ WebSocket
                      ↓
┌─────────────────────────────────────────────────────┐
│  3. sketch.js  (MODIFICAR)                          │
│     → updateLogic() → mapear x, y, btnA, btnB       │
│     → drawRunning() → dibujar el arte generativo    │
└─────────────────────────────────────────────────────┘
```

**Estos archivos no se tocan:**
- bridgeClient.js 
- fsm.js
- index.html
- style.css

---

Antes de cambiar los códigos del proyecto, en https://python.microbit.org/v/3 cambiamos nuestro código en nuestro **microbit** para que lea la integración de CHK y la nueva forma de parsearlo ya que está microbit funcionara como prototipo de nuestra aplicación.


```
//Código Microbit.editor

#Código Microbit.editor

from microbit import *
import time

uart.init(115200)
display.set_pixel(0, 0, 9)

while True:
    xValue = accelerometer.get_x()
    yValue = accelerometer.get_y()
    aState = 1 if button_a.is_pressed() else 0
    # Botones como 0/1 en vez de True/False
    bState = 1 if button_b.is_pressed() else 0
    t = running_time()  # milisegundos desde arranque → esto es el T
    
    # CHK = |X| + |Y| + |A| + |B| 
    # verificación de integridad.
    # Separador | en vez de , 
    chk = abs(xValue) + abs(yValue) + abs(aState) + abs(bState)
    # permite al adapter detectar y descartar tramas dañadas matemáticamente.
    
    data = "$T:{}|X:{}|Y:{}|A:{}|B:{}|CHK:{}\n".format(t, xValue, yValue, aState, bState, chk)
    uart.write(data)
    sleep(100)  # 10 Hz
```

**NOTAS:**


**- a.** Sí o sí tienes que cambiar este código en el micro:bit. El adapter NO transforma el formato, solo lo parsea.
```
micro:bit envía    →  $T:45020|X:-245|Y:12|A:1|B:0|CHK:258
adapter espera     →  $T:45020|X:-245|Y:12|A:1|B:0|CHK:258
                             ✅ coinciden, funciona
```
El adapter recibe la trama y la lee — no la transforma. Es como un traductor que solo entiende español, si le hablas en inglés no te entiende.

**- b.** Por que T existe pero no se envía?

- En el micro:bit lo enviamos porque el protocolo del fabricante lo exige — es parte de la trama.
- En el adapter lo contamos porque si no, el split("|") daría 5 valores y el adapter lo rechazaría como trama corrupta. 
- No lo reenviamos porque el bridgeServer usa su propio reloj con nowMs() — el tiempo del PC es más confiable que el del micro:bit.
- T lo recibimos para que la trama sea válida, pero lo descartamos porque nadie lo necesita aguas abajo
- El $ es solo el marcador de inicio de trama — le dice al adapter "aquí empieza un paquete nuevo".
- El $ entra, nadie lo usa, nadie lo reenvía — muere en el adapter

#### Códigos a corregir (bridgeServer.js - MicrobitV2Adapter.js - sketch.js):

---

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
// Descomentamos nuestro adapter 
const MicrobitV2Adapter = require("./adapters/MicrobitV2Adapter");

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

// Cuando inicio el servidor, diciendole `$ node bridgeServer.js y luego —device microbit-bin le estoy diciendo que adapter instanciar
    // --device es simplemente el argumento que le pasas por consola al arrancar el servidor.
    // --device no es nada especial de Node.js ni de JavaScript — tú lo inventas. Podría llamarse --hardware, --tipo, lo que quieras.
    // Levantamos nuestro servidor escribiendo el --device asi, por que en la const DEVICE = (getArg("device", "sim") busca --device en la terminal y guarda su valor en la variable DEVICE. El nombre de la variable puede ser cualquier cosa
    // Lo importante es que el string que escribes en la consola coincida exactamente con el que comparas en el if. 

// En la función `createAdapter`(Creamos nuestro adapter) if (DEVICE ==="microbit-bin")
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

// Descomentamos nuestro nuevo adapter, el docente lo había puesto
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

---


```

//Adapter -> MicrobitV2Adapter.js

const { SerialPort } = require("serialport");
const BaseAdapter = require("./BaseAdapter");

class ParseError extends Error { }

// PASO_1: DEFINIR CARACTER QUE SEPARARA LA ETIQUETA DEL VALOR

// El cambio esta aquí en el parseo de la información, donde Los valores se separan por el carácter | (pipe).
// Entonces cambiamos la "," por "|"
// Como vamos a recibir 6 valores entonces le decimos que vamos a parsiar 6 valores y no 4.
function parseCsvLine(line) {   
  const values = line.trim().split("|");
  if (values.length !== 6) throw new ParseError(`Expected 6 values, got ${values.length}`);

// PASO_2: EXTRAER CADA VALOR

// Escribimos los 6 valores que queremos recibir (T,X,Y,A,B,CHK)
// y le indicamos que usaremos el caracter : para que coja la segunda parte
  const T = Number(values[0].split(":")[1]);
//                   │              │
//                   │              └── segunda parte después del ":"  → "45020" 
//                   └── primer elemento del arreglo → "$T:45020" 
//        Number() convierte el string "45020" al número 45020
  const x = Number(values[1].split(":")[1]); 
  const y = Number(values[2].split(":")[1]); // ([2]=Etiquea) - ([1]=Valor), por eso hacemos : para separar la etiqueta del valor
  const A = Number(values[3].split(":")[1]);
  const B = Number(values[4].split(":")[1]);
  const CHK = Number(values [5].split(":")[1]);

// PASO_3: VALIDAR LOS VALORES

  if (!Number.isFinite(x) || !Number.isFinite(y)) throw new ParseError("Invalid numeric data");
  // -2048 a 2047. Eso no depende del protocolo, depende del hardware. Se deja tal cual.
  if (x < -2048 || x > 2047 || y < -2048 || y > 2047) throw new ParseError("Out of expected range");
  // if (!["true", "false"].includes(btnA) || !["true", "false"].includes(btnB))
  // Está era la linea anterior pero cambia debido a que venia con true o false y tomaba directamente este valor
  // (Un string es simplemente texto. En JavaScript se reconoce porque va entre comillas.)
  if (![1, 0].includes(A) || ![1, 0].includes(B)) throw new ParseError("Invalid button data");

// PASO_4: VALIDAR EL CHK (Se pone despues de validar los valores) 

  const calculo = Math.abs(x) + Math.abs(y) + Math.abs(A) + Math.abs(B);
  if (calculo !==CHK) throw new ParseError(`Trama corrupta. CHK esperado: ${CHK}, calculo: ${calculo}`);

// PASO_5: Si todo esta bien, empaqueto y envío

// Este es el valor que enviaremos a nuestro programa ya parseado donde solo enviara el valor apartir de ":"
// minúsculas porque así lo espera bridgeServer (x, y, btnA, btnB)
//adapter.onData = (d) => {
//    broadcast(wss, {
//      type: "microbit",
//      x: d.x,
//      y: d.y,
//      btnA: !!d.btnA,
//      btnB: !!d.btnB,
//     t: nowMs(),
//    });
//  };

// A, B: Estado de los botones, 1 presionado, 0 liberado
  return { x: x | 0, y: y | 0, btnA: A === 1, btnB: B === 1 };
}

//Aca cambiamos el nombre del adapter que vamos a llamar en este caso es: MicrobitV2Adapter
class MicrobitV2Adapter extends BaseAdapter {
  constructor({ path, baud = 115200, verbose = false } = {}) {
    super();
    this.path = path;
    this.baud = baud;
    this.port = null;
    this.buf = "";
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
    this.buf = "";
    this.onDisconnected?.("serial closed");
  }

  getConnectionDetail() {
    return `serial open ${this.path}`;
  }

  _onChunk(chunk) {
    this.buf += chunk.toString("utf8");

    let idx;
    while ((idx = this.buf.indexOf("\n")) >= 0) {
      const line = this.buf.slice(0, idx).trim();
      this.buf = this.buf.slice(idx + 1);

      if (!line) continue;

      try {
        const parsed = parseCsvLine(line);
        this.onData?.(parsed);
      } catch (e) {
        if (e instanceof ParseError) {
          console.warn("[MicrobitV2Adapter] Trama corrupta descartada:", e.message);
              //console.warn en la terminal de Node.js aparece resaltado en amarillo, que es la convención estándar para advertencias
              //  sí sirve porque es un aviso que aparece siempre, sin condiciones. Además console.warn en Node.js
              // tiene un significado semántico específico — es para advertencias, cosas que no rompen el sistema pero que el desarrollador debe saber que pasaron. 
              
          //if (this.verbose) console.log("Bad data:", e.message, "raw:", line);
              //no sirve porque es una opción que tú le pasas al servidor cuando lo arrancas con --verbose.
              // Sin ese flag, el warning nunca aparece
              // significa que el aviso depende de que tú recuerdes escribir --verbose en la terminal. 
        } else {
          this._fail(e);
        }
      }
    }

    if (this.buf.length > 4096) this.buf = "";
  }

  _fail(err) {
    this.onError?.(String(err?.message || err));
    this.disconnect();
  }

  _closed() {
    if (!this.connected) return;
    this.connected = false;
    this.port = null;
    this.buf = "";
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

//Cambiamos el modulo que vamos a exportar 
module.exports = MicrobitV2Adapter;
```

```
//sketch.js

// Copy from P_2_0_02
//
// Generative Gestaltung – Creative Coding im Web
// ISBN: 978-3-87439-902-9, First Edition, Hermann Schmidt, Mainz, 2018
// Benedikt Groß, Hartmut Bohnacker, Julia Laub, Claudius Lazzeroni
// with contributions by Joey Lee and Niels Poldervaart
// Copyright 2018
//
// http://www.generative-gestaltung.de
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at http://www.apache.org/licenses/LICENSE-2.0
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

const EVENTS = {
    CONNECT: "CONNECT",
    DISCONNECT: "DISCONNECT",
    DATA: "DATA",
    //KEY_PRESSED: "KEY_PRESSED", no usamos teclado
    //KEY_RELEASED: "KEY_RELEASED", no usamos teclado
};

class PainterTask extends FSMTask {
    constructor() {
        super();

        //this.c = color(181, 157, 0);
        //this.lineSize = 100;
        //this.angle = 0;
        //this.clickPosX = 0;
        //this.clickPosY = 0;

        this.rxData = {
            circleResolution: 2, //se agrega
            radius: 0, // se agrega
            //x: 0,
            //y: 0,
            btnA: false,
            btnB: false,
            //prevA: false, -> son lineas del caso anterior
            //prevB: false, -> son lineas del caso anterior
            ready: false
        };

        this.transitionTo(this.estado_esperando);
    }

    estado_esperando = (ev) => {
        if (ev.type === "ENTRY") {
            cursor();
            console.log("Waiting for connection...");
        } else if (ev.type === EVENTS.CONNECT) {
            this.transitionTo(this.estado_corriendo);
        }
    };

    estado_corriendo = (ev) => {
        if (ev.type === "ENTRY") {
            noCursor();
            strokeWeight(0.75);
            background(255);
            console.log("Microbit ready to draw");
            this.rxData = {
                circleResolution: 2,
                radius: 0,
                //x: 0,
                //y: 0,
                btnA: false,
                btnB: false,
                //prevA: false,
                //prevB: false,
                ready: false
            };
        }

        else if (ev.type === EVENTS.DISCONNECT) {
            this.transitionTo(this.estado_esperando);
        }

        else if (ev.type === EVENTS.DATA) {
            this.updateLogic(ev.payload);
        }

        //else if (ev.type === EVENTS.KEY_PRESSED) {
        //    this.handleKeys(ev.keyCode, ev.key);
        //}

        //else if (ev.type === EVENTS.KEY_RELEASED) {
        //    this.handleKeyRelease(ev.keyCode, ev.key);
        //}

        else if (ev.type === "EXIT") {
            cursor();
        }
    };

    updateLogic(data) {
        this.rxData.ready = true;
        //this.rxData.x = map(data.x,-2048,2047,0,width);
        this.rxData.circleResolution = int(map(data.y, -2048, 2047, 2, 10));
        //this.rxData.y = map(data.y,-2048,2047,0,height);
        this.rxData.radius = map(data.x, -2048, 2047, -width / 2, width / 2);

        this.rxData.btnA = data.btnA;
        this.rxData.btnB = data.btnB;

        //if (this.rxData.btnA && !this.prevA) {
        //    this.lineSize = random(50, 160);
        //    this.clickPosX = this.rxData.x;
        //    this.clickPosY = this.rxData.y;
        //    console.log("A pressed");
        //}

        //if (!this.rxData.btnB && this.prevB) {
        //    this.c = color(random(255), random(255), random(255), random(80, 100));
        //    console.log("B released");
        //}

        //this.prevA = this.rxData.btnA;
        //this.prevB = this.rxData.btnB;
    }
}

let painter;
let bridge;
let connectBtn;
const renderer = new Map();

function setup() {
    createCanvas(windowWidth, windowHeight);
    background(255);
    painter = new PainterTask();
    bridge = new BridgeClient();

    bridge.onConnect(() => {
        connectBtn.html("Disconnect");
        painter.postEvent({ type: EVENTS.CONNECT });
    });

    bridge.onDisconnect(() => {
        connectBtn.html("Connect");
        painter.postEvent({ type: EVENTS.DISCONNECT });
    });

    bridge.onStatus((s) => {
        console.log("BRIDGE STATUS:", s.state, s.detail ?? "");
    });

    bridge.onData((data) => {
        painter.postEvent({
            type: EVENTS.DATA, payload: {
                x: data.x,
                y: data.y,
                btnA: data.btnA,
                btnB: data.btnB
            }
        });
    });

    connectBtn = createButton("Connect");
    connectBtn.position(10, 10);
    connectBtn.mousePressed(() => {
        if (bridge.isOpen) bridge.close();
        else bridge.open();
    });

    renderer.set(painter.estado_corriendo, drawRunning);
}

function draw() {
    painter.update();
    renderer.get(painter.state)?.();
}

function drawRunning() {
    const mb = painter.rxData;
    if (!mb.ready) return;

    //Antes el dibujo se activaba apretando el mouse. Ahora se activa apretando el botón A del micro:bit.
    if (mb.btnA) {
        push();
    
    // Igual que el original — no cambia
    // Mueve el origen al centro del canvas para que los polígonos se dibujen desde el centro.
        translate(width / 2, height / 2);
    
    //circleResolution ya viene calculado desde updateLogic
        let angle = TAU / mb.circleResolution;

    // Original usaba el teclado, ahora usamos el boton B
        if (mb.btnB) {
            fill(34, 45, 122, 50);
        } else {
            noFill();
        }
        beginShape();
        for (let i = 0; i <= mb.circleResolution; i++) {
            let x = cos(angle * i) * mb.radius;
            let y = sin(angle * i) * mb.radius;
            vertex(x, y);
        }
        endShape();
        pop();
    }
}

function windowResized() {
    resizeCanvas(windowWidth, windowHeight);
}
```

**NOTAS:**

1. De mouseIsPressed a mb.btnA
El prototipo usaba el clic del mouse para activar el dibujo. En el sistema con micro:bit no hay mouse, entonces el botón A físico toma ese rol.

2. De mouseY a map(data.y, -2048, 2047, 2, 10)
El prototipo usaba la posición del mouse en pantalla (0 a 720px) para calcular circleResolution. El acelerómetro no habla en píxeles, habla en unidades de -2048 a 2047, entonces map() traduce ese rango al mismo rango que usaba el mouse.

3. De mouseX a map(data.x, -2048, 2047, -width/2, width/2)
Igual que el anterior — el radio antes dependía de cuánto movías el mouse horizontalmente. Ahora depende de cuánto inclinas el micro:bit.

4. De keyIsPressed a mb.btnB
El relleno azul antes se activaba con cualquier tecla del teclado. Ahora lo activa el botón B físico del micro:bit.

5. La separación updateLogic / drawRunning
Este es el cambio estructural más importante:
- updateLogic → recibe los datos crudos del hardware y los escala. Es el único lugar donde se hace matemática.
- drawRunning → solo dibuja usando los valores ya calculados. No sabe nada del hardware.

6. Cambios:
- mouseIsPressed  →  mb.btnA      (activa el dibujo)
- mouseY          →  mb.circleResolution  (cuántos lados tiene el polígono)
- mouseX          →  mb.radius    (qué tan grande es)
- keyIsPressed    →  mb.btnB      (activa el relleno)

---

**LEVANTAR SERVIDOR Y PONER A FUNCIONAR APLICACIÓN:**

**1. Corregimos nuestros archivos:**

-** bridgeServer.js (Servidor de nuestro proyecto)
**-** MicrobitV2Adapter.js (Nuevo adapter de nuestro proyecto)
**-** sketch.js (Adaptar a la aplicación de arte generativo que nos entrega el equipo de diseño)
**-** Código de nuestro microbit: funciona como prototipo (Cambiar  funciones de diseño para nuestro elemento que funcionara como prototipo)

**2. Terminal:**

- Vamos a la ubicación de nuestra carpeta, le damos clic derecho y abrimos la terminal desde ahi… levantamos el servidor y le decimos que el adapter que implementaremos sera el bin (desde nuestro archivo lo llamamos microbit-bin - y el —device se pone para hacer referencia al adapter que instanciaremos.)

**Escribir en la terminal para hacer lo anterior mencionado**

```jsx
node bridgeServer.js --device microbit-bin --baud 115200
```

**3. Visual studio code:**

- Vamos a visual studio code y abrimos el folder de nuestra aplicación, para que este corra y nos de la dirección IP - Necesitaremos un server, osea saber de donde nos llega información, este servidor vendra de una extensión que instalamos directamente en visual studio code, llamada p5.scode que trae el live Server y al instalar el p5.scode, automaticamente se instala la extensión solicitada. Live Server.
 
**a.** p5.vscode ——> Liver Server

**4. Conectar y probar:**

-** Le das clic a Go live, en la parte inferior derecha de la interfaz de visual studio code, asi mismo este te abrira una web con una IP especifica donde te aparecera un bonton que indica conectar los sistemas y protocolos de comunicación para que al mover la microbit esta información se vea reflejada en la pantalla…

**5. Como funciona?**
Levantamos un servidor Node.js que actúa como puente de comunicación entre un dispositivo físico (micro:bit) y una aplicación web de arte generativo. Para manejar diferentes protocolos de hardware sin modificar el servidor principal, aplicamos el patrón **Adapter** — creando archivos como `MicrobitV2Adapter.js` que se encargan de tres cosas: traducir el protocolo de comunicación del hardware, verificar la integridad de cada trama mediante el cálculo del checksum (descartando silenciosamente las tramas corruptas y avisando en consola), y emitir siempre el mismo objeto estandarizado `{ x, y, btnA, btnB }` sin importar de qué hardware venga.

Esta información viaja a través de dos conexiones:

- **Puerto serial USB** → entre el micro:bit y Node.js
- **Puerto WebSocket 8081** → entre Node.js y el navegador

En el frontend, una **Máquina de Estados Finitos (FSM)** separa la ingesta de datos del renderizado visual, permitiendo que la aplicación reaccione de forma ordenada a los eventos de conexión, desconexión y llegada de datos del hardware.

Las técnicas aplicadas fueron:

- **Patrón Adapter** — como sistema de traducción y validación entre protocolos
- **WebSocket** — como medio de transporte de datos entre capas
- **FSM** — como controlador de la lógica de estados del frontend
- **Instancias entre archivos** — para conectar las capas del sistema de forma desacoplada

---

**NOTAS:**
De la unica manera que el programa se muera es diciendole Cmd+c en la terminal

**CONCLUSIÓN:**
**Unidad 4** Se construyo el `MicrobitV2Adapter.js` para el protocolo ASCII con framing (`$T:|X:|Y:|A:|B:|CHK:\n`). 

- `BaseAdapter.js` → clase base
- `MicrobitASCIIAdapter.js` → referencia de cómo se hace
- `bridgeServer.js` → registra los adapters
- `bridgeClient.js` → transporte WebSocket
- `sketch.js` + `fsm.js` → frontend con máquina de estado

---

## Bitácora de reflexión
### DESARROLLO ACTIVIDAD #3
***Diagrama de flujo detallado**

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

title Flujo de Datos — Unidad 4 (Protocolo ASCII con CHK)

participant "micro:bit\n(firmware Python)" as MB
participant "MicrobitV2Adapter\n(Node.js)" as ADAPTER
participant "bridgeServer\n(Node.js)" as SERVER
participant "bridgeClient\n(Browser)" as CLIENT
participant "PainterTask FSM\n(sketch.js)" as FSM
participant "Canvas\n(p5.js)" as CANVAS

MB -> ADAPTER : ""$T:45020|X:-245|Y:12|A:1|B:0|CHK:258\n""\n〔ASCII · 115200 baud · 10 Hz · longitud variable〕

note over MB
  Firmware Python:
  t = running_time()
  chk = |X| + |Y| + |A| + |B|
  Botones como 0/1 (no True/False)
  $ = marcador de inicio de trama
  \n = delimitador de fin de trama
end note

note over ADAPTER
  PASO 1 — Acumula chunks en string buffer (buf)
  PASO 2 — Busca \n → extrae línea completa
  PASO 3 — split("|") → espera exactamente 6 valores
           split(":")[1] extrae valor tras etiqueta
           T, X, Y, A, B, CHK
  PASO 4 — Valida CHK:
           calculo = |X| + |Y| + |A| + |B|
           si calculo ≠ CHK → console.warn + descarta
  PASO 5 — Emite objeto limpio:
           { x, y, btnA: A===1, btnB: B===1 }
  $ y T entran al adapter y mueren aquí.
  buf > 4096 bytes → se limpia (overflow guard)
end note

ADAPTER -> SERVER : ""{ x: int, y: int, btnA: bool, btnB: bool }""

note over SERVER
  Instanciado con:
  node bridgeServer.js --device microbit-bin
  if (DEVICE === "microbit-bin")
    → new MicrobitV2Adapter(...)
  Reemplaza T del micro:bit por nowMs()
  (reloj del servidor, más confiable)
  Empaqueta y hace broadcast por WebSocket
end note

SERVER -> CLIENT : ""WebSocket · puerto 8081""\n""{ type:""microbit"", x, y, btnA, btnB, t }""

note over CLIENT
  bridgeClient.js — no se modifica.
  Recibe JSON del WebSocket.
  Dispara postEvent con EVENTS.DATA
  Es la frontera entre Node.js y p5.js.
end note

CLIENT -> FSM : ""postEvent({ type: EVENTS.DATA, payload })""\n〔payload = { x, y, btnA, btnB }〕

note over FSM
  estado_corriendo → updateLogic(data):
  circleResolution = int(map(y, -2048, 2047, 2, 10))
  radius = map(x, -2048, 2047, -width/2, width/2)
  btnA → activa el dibujo (antes: mouseIsPressed)
  btnB → activa el relleno (antes: keyIsPressed)
  updateLogic: única capa con matemática.
  drawRunning: solo lee rxData, no calcula.
end note

FSM -> CANVAS : ""drawRunning()""\nDibuja polígono si btnA == true\nFill azul si btnB == true

note over CANVAS
  translate(width/2, height/2)
  angle = TAU / circleResolution
  beginShape() → vertex(cos·r, sin·r) × circleResolution
  fill(34,45,122,50) si btnB, noFill() si no
end note

@enduml
```


