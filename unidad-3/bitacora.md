# Unidad 3

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 
### DESARROLLO ACTIVIDAD #4

**RESPUESTA**

*PROGRAMAS*




**style.css**
```
html, body {
  margin: 0;
  padding: 0;
}

canvas {
  display: block;
}
```

## Bitácora de reflexión

### DESARROLLO ACTIVIDAD #5

**RESPUESTA**

Para este problema, el docente en la unidad 3, actividad 5 del repositorio del curso, nos indica leer la data de “Radio” de microbit.

<img width="1458" height="688" alt="image" src="https://github.com/user-attachments/assets/0fa31bc5-4781-4025-9cd5-33bde8cf4ee9" />
<img width="1474" height="634" alt="image" src="https://github.com/user-attachments/assets/c631c026-f6bf-4ff0-8a52-e84b25dae2d5" />
<img width="1504" height="1006" alt="image" src="https://github.com/user-attachments/assets/98b52ed4-c1ec-411f-b014-e9c0762b34ee" />
<img width="1502" height="552" alt="image" src="https://github.com/user-attachments/assets/ce74e418-7aa4-4474-ba3d-81a501a96253" />



**CÓDIGOS**

```
// CÓDIGO P5.JS
// ─────────────────────────────────────────────────────────────
//  CONSTANTES DE CONFIGURACIÓN
// ─────────────────────────────────────────────────────────────
const TIMER_LIMITS = {
  min: 15,
  max: 25,
  defaultValue: 20,
};

const EVENTS = {
  DEC: "A",      // Botón A  → decrementar / pausar / parte de la secuencia
  INC: "B",      // Botón B  → incrementar / parte de la secuencia
  START: "S",    // Tecla S o agitar micro:bit → iniciar
  TICK: "Timeout",
};
6
const UI = {
  dialSize: 250,
  ringWeight: 20,
  bigText: 100,
  configText: 120,
  helpText: 18,
};

// ─────────────────────────────────────────────────────────────
//  SERIAL  (Web Serial API → micro:bit por USB)
// ─────────────────────────────────────────────────────────────
let serialPort = null;
let serialConnected = false;
let serialBuffer = "";
let connectButton;

// El usuario hace clic en el botón → abre el diálogo del navegador
// para seleccionar el puerto USB del micro:bit
async function connectSerial() {
  try {
    serialPort = await navigator.serial.requestPort();
    await serialPort.open({ baudRate: 115200 });
    serialConnected = true;
    connectButton.hide();        // Oculta el botón una vez conectado
    readSerialLoop();            // Inicia la lectura continua
  } catch (e) {
    console.warn("No se pudo conectar al micro:bit:", e);
  }
}

// Lee el puerto serie en un loop asíncrono.
// Cada línea recibida ("A\n", "B\n", "S\n") se traduce en un evento.
async function readSerialLoop() {
  const decoder = new TextDecoderStream();
  serialPort.readable.pipeTo(decoder.writable);
  const reader = decoder.readable.getReader();

  try {
    while (true) {
      const { value, done } = await reader.read();
      if (done) break;

      serialBuffer += value;
      const lines = serialBuffer.split("\n");
      serialBuffer = lines.pop();          // deja la línea incompleta para después

      for (const line of lines) {
        const cmd = line.trim().toUpperCase();
        if (cmd === "A") temporizador.postEvent(EVENTS.DEC);
        else if (cmd === "B") temporizador.postEvent(EVENTS.INC);
        else if (cmd === "S") temporizador.postEvent(EVENTS.START);
      }
    }
  } catch (e) {
    serialConnected = false;
    connectButton.show();
    console.warn("Serial desconectado:", e);
  }
}

// ─────────────────────────────────────────────────────────────
//  CLASE TEMPORIZADOR  (extiende FSMTask de fsm.js)
// ─────────────────────────────────────────────────────────────
class Temporizador extends FSMTask {
  constructor(minValue, maxValue, defaultValue) {
    super();

    this.minValue = minValue;
    this.maxValue = maxValue;
    this.defaultValue = defaultValue;
    this.configValue = defaultValue;
    this.totalSeconds = defaultValue;
    this.remainingSeconds = defaultValue;

    // PUNTO 1: bandera para saber si está pausado
    this.paused = false;

    // PUNTO 2: historial de últimas teclas para detectar A-B-A
    this.sequence = [];
    this.password = [EVENTS.DEC, EVENTS.INC, EVENTS.DEC]; // ["A","B","A"]

    this.myTimer = this.addTimer(EVENTS.TICK, 1000);
    this.transitionTo(this.estado_config);
  }

  get currentState() {
    return this.state;
  }

  // ── Estado 1: Configuración ──────────────────────────────────
  estado_config = (ev) => {
    if (ev === ENTRY) {
      this.configValue = this.defaultValue;
    } else if (ev === EVENTS.DEC) {
      if (this.configValue > this.minValue) this.configValue--;
    } else if (ev === EVENTS.INC) {
      if (this.configValue < this.maxValue) this.configValue++;
    } else if (ev === EVENTS.START) {
      this.totalSeconds = this.configValue;
      this.remainingSeconds = this.totalSeconds;
      this.transitionTo(this.estado_armed);
    }
  };

  // ── Estado 2: Corriendo ──────────────────────────────────────
  estado_armed = (ev) => {
    if (ev === ENTRY) {
      this.myTimer.start();
      this.sequence = [];    // limpia la secuencia al entrar
      this.paused = false;
    }

    // PUNTO 2: A y B se acumulan en la secuencia
    else if (ev === EVENTS.DEC || ev === EVENTS.INC) {
      this.sequence.push(ev);

      // Cuando hay 3 elementos, verificamos si es la contraseña A-B-A
      if (this.sequence.length === 3) {
        if (this.sequence.join("") === this.password.join("")) {
          // ¡Secuencia correcta! → volver a configuración
          this.transitionTo(this.estado_config);
          return; // salimos antes de ejecutar el toggle de pausa
        } else {
          // No coincide: ventana deslizante (guarda el último elemento)
          // Así el usuario puede intentar de nuevo solapando
          this.sequence = [this.sequence[2]];
        }
      }

      // PUNTO 1: solo la tecla A pausa / reanuda
      if (ev === EVENTS.DEC) {
        if (!this.paused) {
          this.paused = true;
          this.myTimer.stop();        // detiene el contador
        } else {
          this.paused = false;
          this.myTimer.start();       // reanuda desde donde se quedó
        }
      }
    }

    // El tick llega cada 1 segundo cuando no está pausado
    else if (ev === EVENTS.TICK) {
      if (this.remainingSeconds > 0) {
        this.remainingSeconds--;
        if (this.remainingSeconds === 0) {
          this.transitionTo(this.estado_timeout);
        } else {
          this.myTimer.start();  // reinicia el timer para el siguiente segundo
        }
      }
    }

    else if (ev === EXIT) {
      this.myTimer.stop();
      this.paused = false;
    }
  };

  // ── Estado 3: Tiempo agotado ─────────────────────────────────
  estado_timeout = (ev) => {
    if (ev === ENTRY) {
      console.log("¡TIEMPO!");
    } else if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_config);
    }
  };
}

// ─────────────────────────────────────────────────────────────
//  P5.JS — setup y draw
// ─────────────────────────────────────────────────────────────
let temporizador;
const renderer = new Map();

function setup() {
  createCanvas(windowWidth, windowHeight);
  temporizador = new Temporizador(
    TIMER_LIMITS.min,
    TIMER_LIMITS.max,
    TIMER_LIMITS.defaultValue
  );
  textAlign(CENTER, CENTER);

  // Cada estado tiene su propia función de dibujo
  renderer.set(temporizador.estado_config, () =>
    drawConfig(temporizador.configValue)
  );
  renderer.set(temporizador.estado_armed, () =>
    drawArmed(temporizador.remainingSeconds, temporizador.totalSeconds, temporizador.paused)
  );
  renderer.set(temporizador.estado_timeout, () => drawTimeout());

  // Botón HTML para conectar el micro:bit (solo aparece si no está conectado)
  connectButton = createButton("🔌 Conectar micro:bit");
  connectButton.position(width/2.25, height/5);
  connectButton.style("font-size", "14px");
  connectButton.style("padding", "8px 16px");
  connectButton.style("cursor", "pointer");
  connectButton.style("border-radius", "6px");
  connectButton.style("border", "none");
  connectButton.style("background", "#2600ff");
  connectButton.style("color", "white");
  connectButton.mousePressed(connectSerial);
}

function draw() {
  temporizador.upd6666666ate();
  renderer.get(temporizador.currentState)?.();

  // Indicador de conexión serial (esquina superior derecha)
  drawSerialStatus();
}

// ─────────────────────────────────────────────────────────────
//  FUNCIONES DE DIBUJO
// ─────────────────────────────────────────────────────────────
function drawConfig(val) {
  background("#000000");
  fill(255);
  textSize(UI.configText);
  text(val, width / 2, height / 2);
  textSize(UI.helpText);
  fill(200);
  text("A(−)   B(+)   S / agitar micro:bit (iniciar)", width / 2, height / 2 + 100);
}

function drawArmed(val, total, paused) {
  background(20, 20, 20);

  // Anillo de fondo (apagado)
  noFill();
  strokeWeight(UI.ringWeight);
  stroke(255, 100, 0, 50);
  ellipse(width / 2, height / 2, UI.dialSize);

  // Arco de progreso — azul si pausado, naranja si corriendo
  stroke(paused ? color(80, 130, 255) : color(255, 150, 0));
  let angle = map(val, 0, total, 0, TWO_PI);
  arc(width / 2, height / 2, UI.dialSize, UI.dialSize, -HALF_PI, angle - HALF_PI);

  // Número central — sin pulso si pausado
  let pulse = paused ? 0 : sin(frameCount * 0.1) * 10;
  fill(paused ? color(80, 130, 255) : 255);
  noStroke();
  textSize(UI.bigText + pulse);
  text(val, width / 2, height / 2);

  // Texto de ayuda según estado
  textSize(UI.helpText);
  if (paused) {
    fill(80, 130, 255);
    text("⏸ PAUSADO  —  A para reanudar", width / 2, height / 2 + 180);
  } else {
    fill(160);
    text("A pausar  ·  secuencia A-B-A → configurar", width / 2, height / 2 + 180);
  }
}

function drawTimeout() {
  let bg = frameCount % 20 < 10 ? color(150, 0, 0) : color(255, 0, 0);
  background(bg);
  fill(255);
  textSize(UI.bigText);
  text("¡TIEMPO!", width / 2, height / 2);
  textSize(UI.helpText);
  fill(255, 200, 200);
  text("A para volver a configurar", width / 2, height / 2 + 80);
}

function drawSerialStatus() {
  noStroke();
  fill(serialConnected ? color(0, 220, 100) : color(120));
  ellipse(width - 20, 20, 12);
  textSize(12);
  fill(serialConnected ? color(0, 220, 100) : color(120));
  textAlign(RIGHT, CENTER);
  text(serialConnected ? "micro:bit conectado" : "sin micro:bit", width - 30, 20);
  textAlign(CENTER, CENTER); // restaura la alineación global
}

// ─────────────────────────────────────────────────────────────
//  TECLADO  (funciona en paralelo con el micro:bit)
// ─────────────────────────────────────────────────────────────
function keyPressed() {
  if (key === "a" || key === "A") temporizador.postEvent(EVENTS.DEC);
  if (key === "b" || key === "B") temporizador.postEvent(EVENTS.INC);
  if (key === "s" || key === "S") temporizador.postEvent(EVENTS.START);
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}
```

```
//main.py radio.receive

from microbit import *
import utime
import radio

radio.config(group=12)
radio.on()
uart.init(baudrate=115200)

while True:
    if button_a.was_pressed():
        uart.write('A\n')                  

    if button_b.was_pressed():
        uart.write('B\n')            

    if accelerometer.was_gesture("shake"):
        uart.write('S\n')  
        
    message = radio.receive()
    if message:
        
        if message=='A':
            uart.write('A\n') 
            
        if message=='B':
            uart.write('B\n') 

        if message=='S':
            uart.write('S\n')
            
    utime.sleep_ms(20)
```


```
//main.py radio.send

from microbit import *
import utime
import radio

radio.config(group=12)
radio.on()

uart.init(baudrate=115200)

while True:
    if button_a.was_pressed():
        radio.send("A")        

    if button_b.was_pressed():
        radio.send("B")         

    if accelerometer.was_gesture("shake"):
        radio.send("S")        

    utime.sleep_ms(20)
```

**NOTAS**

1. Con `uart.write('A\n')` tú decides manualmente cuándo va el salto de línea. Eso es útil porque puedes enviar datos sin enter si lo necesitas, o enviarlo en el momento exacto que quieras.

2. `print('A')` automáticamente agrega un `\n` al final. No tienes que escribirlo tú.

3. Esto aplica al código del **emisor** (el que envía por radio), no al receptor. El `utime.sleep_ms(20)` está en ese código para no saturar el radio enviando mensajes demasiado rápido.

4. En el código del **receptor** que viste después, **no se usa `utime`** para nada, por eso el profe dijo que no era necesario ahí.

- `print` → agrega `\n` automático, menos control
- `uart.write('A\n')` → tú controlas el `\n`, más control
- `utime.sleep_ms(20)` → solo en el **emisor**, para no saturar el radio

