# Unidad 7

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 

### Cómo configuraste Open Stage Control:
Open Stage Control se lanzó con el archivo launcherConfig.config que contiene:

```{"send":["127.0.0.1:9000"],"port":8086}```

Esto configura dos cosas: el servidor de la UI corre en el puerto 8086 (accesible desde http://127.0.0.1:8086), y todos los mensajes OSC se envían por UDP a 127.0.0.1:9000, que es exactamente el puerto donde OscAdapter.js escucha. La sesión de widgets se carga desde OSCUI.json al abrir la aplicación.

---

### ¿Qué widgets decidiste usar y por qué?
Se usaron tres widgets de tipos distintos, todos afectando únicamente el bombo (bd) (Para hacer más practica la prueba):

**Widget rgb (/rgb_1)** — controla el color del círculo del bombo. Se eligió porque permite modificar los tres canales de color simultáneamente desde una sola superficie. Tiene range: {min: 0, max: 255} para que los valores lleguen directamente como enteros usables en fill(r, g, b) sin conversión.

**Slider (/size)** — controla el tamaño del círculo. Tiene range: {min: 0, max: 1} con valor por defecto 0.5. Se eligió un slider vertical porque representa intuitivamente la idea de "más o menos". En drawRunning() se mapea con map(oc.size, 0, 1, 20, 200) para traducirlo a píxeles reales.

**Slider (/posY)** — controla la posición vertical del círculo en el canvas. Tiene el mismo rango 0–1, donde 0 es la parte superior y 1 la inferior. Se eligió en lugar de un toggle porque da control continuo y es visualmente más expresivo para una superficie de control performático.

---

### ¿Qué estructura final de mensaje decidiste usar?
El contrato de mensaje que viaja por todo el sistema es:

```
{
  "type": "osc",
  "payload": {
    "address": "/rgb_1",
    "args": [200, 100, 50]
  }
}
```

```OscAdapter.js``` produce este formato en ```_onMessage().``` ```bridgeServer.js``` lo retransmite sin modificarlo. ```bridgeClient.js``` lo recibe y lo despacha por ```onData().``` ```sketch.js``` lo convierte en evento ```EVENTS.OSC``` con ```data.payload``` como carga. ```updateOscState()``` es el único lugar del sistema que interpreta el address y actualiza ```oscState.```

---

### ¿Cómo conectaste bridgeClient.js, FSMTask, updateLogic y drawRunning?
El flujo completo para un mensaje ```OSC``` es:
```bridgeClient.js``` recibe el mensaje del WebSocket, lo identifica por ```msg.type === "osc"``` y llama ```this._onData?.(msg).``` No interpreta nada, solo enruta.


**```sketch.js (onData)``` recibe el mensaje y lo convierte en evento para la FSM:**
```
painter.postEvent({ type: EVENTS.OSC, payload: data.payload });
```

**FSMTask (estado_corriendo) recibe el evento EVENTS.OSC y delega:**
```
else if (ev.type === EVENTS.OSC) {
    this.updateOscState(ev.payload);
}
```

```updateOscState()``` es donde ocurre la actualización del estado persistente. Mapea cada address a su propiedad en ```oscState```. No dibuja nada, solo actualiza valores.


```drawRunning()``` lee ```painter.oscState``` cada frame y usa sus valores para dibujar. No sabe nada de ```OSC``` ni de red:

``` 
const bdSize = map(oc.size, 0, 1, 20, 200);
const bdY    = map(oc.posY, 0, 1, 0, height);
const [r, g, b] = oc.color;
fill(r, g, b);
ellipse(width / 2, bdY, bdSize, bdSize);
``` 

### ¿Cómo integraste ambas fuentes de datos en el mismo frontend?

La clave fue que ``` bridgeServer.js```  se lanzó en modo ``` --device strudel+osc``` , lo que instancia ambos adapters en un solo proceso y los conecta al mismo servidor ``` WebSocket```  en el puerto 8081. Así ``` bridgeClient.js```  abre una sola conexión y recibe los tres tipos de mensaje por el mismo canal.
En el frontend, los dos flujos se separan en ``` oscState```  y ``` strudelQueue```  — estructuras de naturaleza completamente distinta. Strudel alimenta una cola temporal que se consume frame a frame comparando ``` timestamps.```  ``` OSC```  sobreescribe propiedades persistentes que no expiran. ``` drawRunning()```  lee ambas al mismo tiempo: sv para saber si hay un evento activo y qué sonido es, y oc para saber con qué color, tamaño y posición dibujarlo.

---

### ¿Qué pruebas hiciste para verificar que el control paramétrico funciona sin romper la sincronización de Strudel?

**Prueba 1** — Control independiente: se movieron los sliders y el rgb con Strudel corriendo. Los flashes del bombo respondieron al nuevo color y tamaño en el siguiente evento, confirmando que ``` oscState```  se actualiza entre frames sin interferir con la cola.

**Prueba 2** — Strudel sin OSC: se apagó Open Stage Control mientras Strudel seguía corriendo. Los flashes continuaron con los últimos valores de ``` oscState```  congelados, sin interrupciones ni errores en consola.

**Prueba 3** — OSC sin Strudel: se movieron los controles sin correr ningún patrón en Strudel. El sistema no se rompió porque ``` updateOscState()```  funciona en cualquier estado de la FSM, incluyendo ``` estado_esperando``` .

**Prueba 4** — Verbose: se corrió el bridge con ``` --verbose```  para confirmar que los mensajes OSC llegaban correctamente: ``` [OscAdapter] /rgb_1 [200, 100, 50].``` 

---

### ¿Qué problemas encontraste y cómo los solucionaste?




## Bitácora de reflexión
