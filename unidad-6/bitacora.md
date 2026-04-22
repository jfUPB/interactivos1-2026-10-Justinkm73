# Unidad 6

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación
### SOLUCIÓN ACTIVIDAD #2

**1. Cómo configuraste Strudel para emitir eventos**


Strudel por defecto solo genera audio en el navegador, necesitamos activar su canal de salida por WebSocket, que usa el protocolo OSC. Strudel puede enviar cada evento musical que va a reproducir a un servidor WebSocket antes de sonarlo.

```
setcps(0.5)

const pat = s("[bd*2 sd hh oh]").bank("tr909")

$: stack(
  pat.gain('0.05'),
  pat.osc()   // ← esta línea activa el envío por WebSocket al puerto 8080
)
```

```.osc()``` es el método clave. Le dice a Strudel que además de reproducir el audio, envíe cada evento al puerto 8080 como mensaje WebSocket. Se usa el puerto 8080 por que se define como valor por defecto en el StrudelAdapter.js ```constructor({ port = 8080, verbose = false } = {}) { ``` y en nuestro bridgeSer.js lo instanciamos...

```
if (DEVICE === "strudel") {
    return new StrudelAdapter({ port: 8080, verbose: VERBOSE });
}
```

Esos dos números tienen que coincidir. Strudel envía al 8080, y nuestro adapter escucha en el 8080.

---

**2. Qué estructura final de mensaje decidiste usar**

Strudel me manda esto:

```
Mensaje crudo que llega de Strudel:
{
  address: '/dirt/play',
  args: [
    'cps',   0.5,
    'cycle', 15.25,
    'delta', 0.5,
    's',     'tr909sd',
    'bank',  'tr909'
  ],
  timestamp: 1774966984435.2805
}
```

Si quisiera acceder al nombre del sonido sin convertir, tendría que escrbir en que posición se encontraría, ejemplo: args[7] -> tr909sd, se dice que se encuentra en esa posición al contarlo manualmente, esto podría hacer más complejo su ejecución ya que strudel tendría que mandarlos en ese orden, de haber un cambio se rompería el código.

El bucle convierte el array en un objeto donde puedo acceder por nombre y no por posición, la solución esta en _normalize() en el adapter que creamos para el strudel

```
// StrudelAdapter.js
_normalize(msg) {
    if (!msg || msg.address !== "/dirt/play") return null;

    // Convertir array plano a objeto clave-valor
    const args = {};
    const arr = Array.isArray(msg.args) ? msg.args : [];
    for (let i = 0; i + 1 < arr.length; i += 2) { //BucLe array
      args[String(arr[i])] = arr[i + 1];  //<- esto convierte ['cps',0.5,'s','bd'] en {cps:0.5, s:'bd'}
```

Dando como resultado esto:

```
args = {
    cps:   0.5,
    cycle: 15.25,
    delta: 0.5,
    s:     'tr909sd',
    bank:  'tr909'
}
```

Ahora para acceder al sonido simplemente se escribe ```args.s``` sin importar la posición así strudel lo cambie.

*NOTA:*

[] es un array — una lista ordenada donde accedes a los elementos por su posición numérica (0, 1, 2...).

{} es un objeto — una colección de pares clave-valor donde accedes por nombre.

---

**3. Cómo conectaste bridgeClient.js, FSMTask, updateLogic y drawRunning**

*Paso_1 (bridgeClient.js recibe y despacha)*

```
// bridgeClient.js
this._ws.onmessage = (event) => {
    let msg = JSON.parse(event.data);
    
    if (msg.type === "strudel") {
        this._onData?.(msg);   // <- despacha el mensaje completo
        return;
    }
```
Igual que el microbit...

*Paso_2 (sketch.js distingue el tipo y lo pone en la cola de la FSM)*

```
// sketch.js
bridge.onData((data) => {
    if (data.type === "strudel") {
        painter.postEvent({
            type:    EVENTS.STRUDEL,
            payload: data   // <- el mensaje completo entra como payload del evento
        });
    } else {
        painter.postEvent({
            type:    EVENTS.DATA,
            payload: { x: data.x, y: data.y, btnA: data.btnA, btnB: data.btnB } //Con el nuevo sistema de tipos, DATA es solo para micro:bit
        });
    }
});
```

*NOTA:*

Para Strudel se manda data completo porque updateStrudelQueue necesita el timestamp y el payload anidado que vienen dentro de ese objeto.

```payload```significa "carga útil". Es el contenido real que importa dentro de un mensaje. 

```
painter.postEvent({
    type:    EVENTS.STRUDEL,   // <- le dice a la FSM qué tipo de evento es
    payload: data              // <- el "contenido": los datos reales del evento (← data completo: tiene type, timestamp, payload anidado)
});
```

La FSM necesita saber el type para decidir a qué estado va el evento. El payload es lo que ese estado va a usar para hacer algo útil. Son dos responsabilidades distintas en el mismo objeto.

*Paso_3 (La FSM entra en estado_corriendo y delega a updateStrudelQueue)*

```
// sketch.js - en estado_corriendo
else if (ev.type === EVENTS.STRUDEL) {
    this.updateStrudelQueue(ev.payload);  // <- no dibuja, solo encola, lo correcto es guardarlo para dibujarlo en el momento que indique su timestamp
}
```

```
// sketch.js
updateStrudelQueue(msg) {
    this.strudelQueue.push({
        timestamp: msg.timestamp, // nivel superior: cuándo activar
        s:         msg.payload.s, // nivel anidado: nombre del sonido
        delta:     msg.payload.delta // nivel anidado: duración
    });
}
```

```updateStrudelQueue``` Recibe ```msg``` que es ```ev.payload```, o sea el objeto completo de Strudel. Se escribe ```msg.payload.s``` y no simplemente ```msg.s```. La estructura viene así porque el StrudelAdapter la construyó con ese anidamiento:

```
// StrudelAdapter.js 
return {
    type: "strudel",
    timestamp,           // <- nivel superior
    payload: {           // <- aquí adentro van los datos del evento
        eventType: "noteEvent",
        s,
        delta
    }
}
```

Después de ```updateStrudelQueue```, la cola ```strudelQueue``` tiene un objeto simplificado y listo para usar:

```
strudelQueue = [
    {
        timestamp: 1774966984435,   // cuándo activarlo
        s: "tr909bd",               // qué dibujar
        delta: 0.5                  // cuánto tiempo mostrarlo
    }
]
```

Ya no tiene ```type```, ni ```eventType```, ni ```payload``` anidado. Solo los tres datos que ```checkStrudelQueue``` necesita para hacer su trabajo. El resto se descartó porque no sirve para nada después de este punto.

*Paso_4 (drawRunning solo lee estado)*

```
// sketch.js
function drawRunning() {
    const sv = painter.strudelVisual;  // <- solo lee, no decide nada

    if (!sv.active) return;

    const name = sv.sound.toLowerCase();
    push();
    noStroke();
    if (name.includes("bd")) {
        fill(30, 80, 200);
        ellipse(width / 2, height * 0.75, 100, 100);
    } else if (name.includes("sd") || name.includes("cp")) {
        fill(200, 50, 50);
        rectMode(CENTER);
        rect(width / 2, height / 2, 100, 40);
    } else if (name.includes("hh") || name.includes("oh")) {
        fill(220, 180, 30);
        ellipse(width / 2, height * 0.25, 40, 40);
    }
    pop();
}
```

La cadena completa es:
```StrudelAdapter``` -> ```bridgeServer``` -> ```WebSocket``` -> ```bridgeClient```
-> ```postEvent``` -> ```FSM``` -> ```strudelQueue``` -> ```checkStrudelQueue``` -> ```strudelVisual``` -> ```drawRunning```

---

**4. Cómo separaste recepción, cola temporal y renderizado**

**Recepción**

Ocurre en ```updateStrudelQueue()``` Su única responsabilidad es tomar el mensaje normalizado y meterlo en la cola con los datos mínimos necesarios. No decide cuándo se activa, no decide qué se dibuja.

```
updateStrudelQueue(msg) {
    this.strudelQueue.push({
        timestamp: msg.timestamp,   // cuándo activar
        s:         msg.payload.s,   // qué sonido
        delta:     msg.payload.delta // cuánto dura
    });
}
```

**Cola temporal**
ocurre en checkStrudelQueue(), que se llama cada frame desde draw(). Compara el reloj local con el timestamp de cada evento.

```
checkStrudelQueue() {
    const now = Date.now();

    // Activar si ya llegó el momento
    if (this.strudelQueue.length > 0 && this.strudelQueue[0].timestamp <= now) {
        const ev = this.strudelQueue.shift();
        this.strudelVisual.active      = true;
        this.strudelVisual.sound       = ev.s;
        this.strudelVisual.delta       = ev.delta;
        this.strudelVisual.triggerTime = now;
    }

    // Desactivar si ya expiró la duración
    if (this.strudelVisual.active) {
        const elapsed  = now - this.strudelVisual.triggerTime;
        const duration = this.strudelVisual.delta * 1000;  // segundos → ms
        if (elapsed >= duration) {
            this.strudelVisual.active = false;
        }
    }
}
```

*NOTA:*
Esta función no vive dentro de la FSM directamente porque necesita ejecutarse cada frame, independientemente de si llegó un evento nuevo. La FSM solo reacciona cuando hay eventos en su cola — ```checkStrudelQueue``` necesita correr siempre.

**Renderizado**
Ocurre en ```drawRunning()```, que solo lee ```strudelVisual```. No sabe nada de WebSockets, no sabe nada de timestamps, no sabe nada de Strudel. Solo pregunta: ¿```sv.active``` es verdadero? ¿Qué dice ```sv.sound```? Y dibuja en consecuencia.

---

**5. Qué pruebas hiciste para verificar la sincronización**

1. La verificación puede ser de observar el comportamiento de la viusal en sincronia con el golpe del audio.

2. Console.log en ```checkStrudelQueue``` -> ```sketch.js``` Agrego está linea de código despues de ```const ev = this.strudelQueue.shift()``` tiene que ser despues de que se genera el evento:

```
console.log("[Strudel] activando:", ev.s, "timestamp:", ev.timestamp, "now:", now);
```

Ese es el momento exacto donde el sistema decide activar un evento visual. Si el log muestra ```ev.s``` con el nombre del sonido correcto y ```timestamp``` cercano a ```now```, confirmas que el ```scheduling``` funciona. La línea simplemente imprime esos tres datos en ese momento exacto.

Para ver está información abrimos el navegador, le damos a los 3 puntos -> más herramientas -> herramientas del desarrollador -> console

<img width="1295" height="924" alt="image" src="https://github.com/user-attachments/assets/62377359-7ee4-463b-b9ca-326b9c4e059b" />

Miramos los números. La diferencia entre ```timestamp``` y ```now``` es de unos 16-17ms en cada evento — eso es exactamente un frame de p5.js a 60fps. Eso confirma tres cosas:

El transporte funciona — los mensajes llegan del bridge al frontend
La cola funciona — los eventos se están activando en orden
La sincronización es buena — ```timestamp``` y ```now``` están muy cerca, no hay acumulación ni retraso

Y mira el orden de los sonidos en la consola: hh → oh → bd → bd → sd → hh → oh → bd → bd → sd. Eso coincide exactamente con el patrón [bd*2 sd hh oh] de Strudel. El sistema está funcionando correctamente.
















## Bitácora de reflexión
