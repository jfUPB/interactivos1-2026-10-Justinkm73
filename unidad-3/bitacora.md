# Unidad 3

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 
### DESARROLLO ACTIVIDAD #4

**RESPUESTA**
Para este ejercicio use lo aprendido en la unidad #1, como conectar sistemas embebidos, Unidad #2 Machin State Fine, Unidad #3 Continuidad FSM y unión con p5.js

**PROGRAMAS p5.js**

```
//fsm.js

const ENTRY = "ENTRY";
const EXIT = "EXIT";

class Timer {
  constructor(owner, eventToPost, duration) {
    this.owner = owner;
    this.event = eventToPost;
    this.duration = duration;
    this.startTime = 0;
    this.active = false;
  }

  start(newDuration = null) {
    if (newDuration !== null) this.duration = newDuration;
    this.startTime = millis();
    this.active = true;
  }

  stop() {
    this.active = false;
  }

  update() {
    if (this.active && millis() - this.startTime >= this.duration) {
      this.active = false;
      this.owner.postEvent(this.event);
    }
  }
}

class FSMTask {
  constructor() {
    this.queue = [];
    this.timers = [];
    this.state = null;
  }

  postEvent(ev) {
    this.queue.push(ev);
  }

  addTimer(event, duration) {
    let t = new Timer(this, event, duration);
    this.timers.push(t);
    return t;
  }

  transitionTo(newState) {
    if (this.state) this.state(EXIT);
    this.state = newState;
    this.state(ENTRY);
  }

  update() {
    for (let t of this.timers) {
      t.update();
    }
    while (this.queue.length > 0) {
      let ev = this.queue.shift();
      if (this.state) this.state(ev);
    }
  }
}
```




```
//index.html

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Sketch</title>

    <link rel="stylesheet" type="text/css" href="style.css">

    <script src="https://cdn.jsdelivr.net/npm/p5@1.11.11/lib/p5.js"></script>

<script src="https://unpkg.com/@gohai/p5.webserial@^1/libraries/p5.webserial.js"></script>

  </head>

  <body>
    <script src="fsm.js"></script>
    <script src="sketch.js"></script>
  </body>
</html>
```


```
//sketch.js

const TIMER_LIMITS = {
  min: 15,
  max: 25,
  defaultValue: 20,
};

const EVENTS = {
  DEC: "A",
  INC: "B",
  START: "S",
  TICK: "Timeout",
};

// ─────────────────────────────────────────────
//  Temporizador — Máquina de Estados
// ─────────────────────────────────────────────
class Temporizador extends FSMTask {
  constructor(minValue, maxValue, defaultValue) {
    super();
    this.minValue     = minValue;
    this.maxValue     = maxValue;
    this.defaultValue = defaultValue;
    this.configValue  = defaultValue;
    this.totalSeconds = defaultValue;
    this.remainingSeconds = defaultValue;

    // Secuencia para detectar A-B-A mientras está corriendo / pausado
    this.sequence = [];

    this.myTimer = this.addTimer(EVENTS.TICK, 1000);
    this.transitionTo(this.estado_config);
  }

  get currentState() {
    return this.state;
  }

  // ── Estado 1: Configuración ──────────────────
  estado_config = (ev) => {
    if (ev === ENTRY) {
      this.configValue = this.defaultValue;
      this.sequence    = [];
    } else if (ev === EVENTS.DEC) {
      if (this.configValue > this.minValue) this.configValue--;
    } else if (ev === EVENTS.INC) {
      if (this.configValue < this.maxValue) this.configValue++;
    } else if (ev === EVENTS.START) {
      this.totalSeconds     = this.configValue;
      this.remainingSeconds = this.totalSeconds;
      this.transitionTo(this.estado_armed);
    }
  };

  // ── Estado 2: Corriendo ──────────────────────
  estado_armed = (ev) => {
    if (ev === ENTRY) {
      this.sequence = [];       // limpia secuencia al entrar
      this.myTimer.start();
    } else if (ev === EXIT) {
      this.myTimer.stop();      // siempre detiene el timer al salir
    } else if (ev === EVENTS.TICK) {
      if (this.remainingSeconds > 0) {
        this.remainingSeconds--;
        if (this.remainingSeconds === 0) {
          this.transitionTo(this.estado_timeout);
        } else {
          this.myTimer.start(); // reinicia para el siguiente segundo
        }
      }
    } else if (ev === EVENTS.DEC) {
      // "A" mientras corre → PAUSA y arranca la secuencia A-B-A
      this.sequence = ["A"];
      this.transitionTo(this.estado_paused);
    }
    // "B" mientras corre no hace nada (solo sirve en config)
  };

  // ── Estado 3: Pausado ────────────────────────
  //   Detecta A-B-A para volver a config
  //   Cualquier "A" que no complete la secuencia → reanuda
  estado_paused = (ev) => {
    if (ev === ENTRY) {
      // El timer ya se detuvo por el EXIT de estado_armed
    } else if (ev === EVENTS.INC) {
      // "B": solo avanza la secuencia si el paso previo fue "A"
      if (this.sequence.length === 1 && this.sequence[0] === "A") {
        this.sequence.push("B");
      } else {
        // Secuencia rota → reinicia pero sigue pausado
        this.sequence = [];
      }
    } else if (ev === EVENTS.DEC) {
      // "A": intenta completar la secuencia
      this.sequence.push("A");
      if (this.sequence.join("") === "ABA") {
        // ¡Contraseña correcta! → vuelve a configurar
        this.sequence = [];
        this.transitionTo(this.estado_config);
      } else {
        // No era A-B-A → reanuda el conteo
        this.sequence = [];
        this.transitionTo(this.estado_armed);
      }
    }
  };

  // ── Estado 4: Timeout ────────────────────────
  estado_timeout = (ev) => {
    if (ev === ENTRY) {
      console.log("¡TIEMPO!");
    } else if (ev === EVENTS.DEC) {
      this.transitionTo(this.estado_config);
    }
  };
}

// ─────────────────────────────────────────────
//  Variables globales
// ─────────────────────────────────────────────
let temporizador;
let serial;
const renderer = new Map();

// ─────────────────────────────────────────────
//  p5.js setup
// ─────────────────────────────────────────────
function setup() {
  createCanvas(windowWidth, windowHeight);
  textAlign(CENTER, CENTER);

  temporizador = new Temporizador(
    TIMER_LIMITS.min,
    TIMER_LIMITS.max,
    TIMER_LIMITS.defaultValue
  );

  // WebSerial (p5.webserial)
  serial = createSerial();

  // Mapa estado → función de dibujo
  renderer.set(temporizador.estado_config,  () => drawConfig(temporizador.configValue));
  renderer.set(temporizador.estado_armed,   () => drawArmed(temporizador.remainingSeconds, temporizador.totalSeconds));
  renderer.set(temporizador.estado_paused,  () => drawPaused(temporizador.remainingSeconds, temporizador.totalSeconds));
  renderer.set(temporizador.estado_timeout, () => drawTimeout());
}

// ─────────────────────────────────────────────
//  p5.js draw  ←  equivale al while True del microbit
// ─────────────────────────────────────────────
function draw() {
  // 1. Leer datos del micro:bit por serial
  if (serial.available() > 0) {
    let data = serial.readUntil("\n").trim();
    if (data === "A" || data === "B" || data === "S") {
      temporizador.postEvent(data);
    }
  }

  // 2. Actualizar la máquina de estados
  temporizador.update();

  // 3. Dibujar el estado actual
  renderer.get(temporizador.currentState)?.();

  // 4. Botón de conexión serial (siempre encima)
  drawSerialButton();
}

// ─────────────────────────────────────────────
//  Pantallas de cada estado
// ─────────────────────────────────────────────

function drawConfig(val) {
  background(10, 20, 50);

  // Título
  fill(80, 120, 255);
  textSize(16);
  text("CONFIGURAR TIEMPO", width / 2, height / 2 - 140);

  // Número grande
  fill(255);
  textSize(130);
  text(val, width / 2, height / 2 - 10);

  // Barra de progreso
  let prog = map(val, TIMER_LIMITS.min, TIMER_LIMITS.max, 0, 1);
  noFill();
  stroke(50, 60, 100);
  strokeWeight(8);
  rectMode(CENTER);
  rect(width / 2, height / 2 + 90, 260, 12, 6);
  fill(80, 120, 255);
  noStroke();
  rectMode(CORNER);
  rect(width / 2 - 130, height / 2 + 84, 260 * prog, 12, 6);
  rectMode(CENTER);

  // Ayuda
  textSize(16);
  fill(130, 150, 200);
  text("A  →  bajar    B  →  subir    S  →  iniciar", width / 2, height / 2 + 130);
  text("micro:bit: botón A / B / agitar", width / 2, height / 2 + 155);
}

function drawArmed(val, total) {
  background(15, 15, 20);

  let pulse = sin(frameCount * 0.08) * 8;
  let ratio = val / total;
  let ringColor = lerpColor(color(255, 60, 60), color(80, 220, 120), ratio);

  // Anillo de fondo
  noFill();
  strokeWeight(18);
  stroke(red(ringColor), green(ringColor), blue(ringColor), 35);
  ellipse(width / 2, height / 2, 280);

  // Arco de progreso
  stroke(ringColor);
  strokeWeight(18);
  let angle = map(val, 0, total, 0, TWO_PI);
  arc(width / 2, height / 2, 280, 280, -HALF_PI, angle - HALF_PI);

  // Número
  fill(255);
  noStroke();
  textSize(110 + pulse);
  text(val, width / 2, height / 2);

  // Instrucciones
  textSize(15);
  fill(140);
  text("A → pausar     A-B-A → volver a configurar", width / 2, height / 2 + 175);
}

function drawPaused(val, total) {
  background(15, 15, 20);

  let ratio  = val / total;
  let blink  = frameCount % 50 < 25;

  // Anillo atenuado
  noFill();
  strokeWeight(18);
  stroke(80, 100, 220, 40);
  ellipse(width / 2, height / 2, 280);

  stroke(80, 100, 220, blink ? 200 : 80);
  let angle = map(val, 0, total, 0, TWO_PI);
  arc(width / 2, height / 2, 280, 280, -HALF_PI, angle - HALF_PI);

  // Número (parpadea)
  fill(blink ? color(100, 130, 255) : color(60, 80, 180));
  noStroke();
  textSize(110);
  text(val, width / 2, height / 2);

  // Etiqueta PAUSADO
  fill(100, 130, 255);
  textSize(20);
  text("⏸  PAUSADO", width / 2, height / 2 - 170);

  // Secuencia actual
  let seqDisplay = temporizador.sequence.join(" - ") || "—";
  textSize(14);
  fill(100, 100, 160);
  text("secuencia: " + seqDisplay, width / 2, height / 2 + 145);

  textSize(15);
  fill(140);
  text("A → reanudar     A-B-A → volver a configurar", width / 2, height / 2 + 175);
}

function drawTimeout() {
  let blink = frameCount % 24 < 12;
  background(blink ? color(160, 0, 0) : color(220, 20, 20));

  fill(255);
  noStroke();
  textSize(110);
  text("¡TIEMPO!", width / 2, height / 2 - 20);

  textSize(18);
  fill(255, 180, 180);
  text("A → volver a configurar", width / 2, height / 2 + 90);
}

// ─────────────────────────────────────────────
//  Botón de conexión WebSerial
// ─────────────────────────────────────────────
function drawSerialButton() {
  let connected = serial.opened();
  let bx = 20, by = 20, bw = 200, bh = 38;

  // Fondo del botón
  fill(connected ? color(0, 140, 80) : color(140, 40, 0));
  noStroke();
  rectMode(CORNER);
  rect(bx, by, bw, bh, 8);

  // Texto
  fill(255);
  textSize(13);
  textAlign(LEFT, CENTER);
  text(connected ? "✅  micro:bit conectado" : "🔌  Conectar micro:bit", bx + 12, by + bh / 2);
  textAlign(CENTER, CENTER);
}

// ─────────────────────────────────────────────
//  Eventos de ratón y teclado
// ─────────────────────────────────────────────
function mousePressed() {
  // Clic en el botón de conexión
  if (mouseX >= 20 && mouseX <= 220 && mouseY >= 20 && mouseY <= 58) {
    if (!serial.opened()) {
      serial.open(115200);
    } else {
      serial.close();
    }
  }
}

function keyPressed() {
  if (key === "a" || key === "A") temporizador.postEvent("A");
  if (key === "b" || key === "B") temporizador.postEvent("B");
  if (key === "s" || key === "S") temporizador.postEvent("S");
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
}
```


```
//style.css

html, body {
  margin: 0;
  padding: 0;
}
canvas {
  display: block;
}
```

**PROGRAMAS MICROBIT**
```
//main.py

from microbit import *
import utime

while True:
    if button_a.was_pressed():
        print("A")        

    if button_b.was_pressed():
        print("B")         

    if accelerometer.was_gesture("shake"):
        print("S")        

    utime.sleep_ms(20)
```

**NOTAS**:
EJEMPLOS DE COMO FUNCIONA UNA MAQUINA DE ESTÁDO:

ESTADO —— > EVENTO —-> Transiciones

EJEMPLO CON  EL CÓDIGO DEL SEMAFORO:

Estados: ("situaciones”)
estado_config → configurando el tiempo
estado_armed → contando regresivamente
estado_timeout → la alarma sonando

Eventos: ("cosas que pasan" )
Alguien presionó el botón A → evento "A"
Se acabó el tiempo → evento "Timeout"
Se agitó el micro:bit → evento "S"

Transiciones: (“Reglas”)
Si estás en config y pasa "S" → ve a config y pasa "S" → ve a armedontador llega a 0 → ve a timeout
Si estás en timeout y pasa "A" → regresa a config

<img width="674" height="600" alt="image" src="https://github.com/user-attachments/assets/538f1cb2-06dd-4b3a-b78c-a2553fc0c729" />


**Tener en cuenta que:**

was_pressed = evento
is_pressed = estado


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

