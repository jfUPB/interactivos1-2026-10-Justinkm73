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

**Problema 1 - Open Stage Control aparecía vacío** 
Al abrir http://127.0.0.1:8086, la interfaz de Open Stage Control aparecía completamente vacía y no mostraba ningún widget.

La causa del problema era que el archivo OSCUI.json no estaba cargado en la sesión actual de Open Stage Control.

La solución consistió en cargar manualmente el archivo de sesión desde el menú de Open Stage Control, permitiendo recuperar correctamente toda la interfaz gráfica

**Problema 2 - Los widgets no modificaban el visual**
Aunque los widgets aparecían correctamente en pantalla, mover los controles no generaba ningún cambio en el visual.

El problema ocurría porque el bridge de OSC no se encontraba corriendo correctamente.

La solución fue levantar el segundo proceso correspondiente al bridge OSC y verificar, utilizando el modo --verbose, que los mensajes UDP estuvieran llegando correctamente al servidor. Una vez confirmada la recepción de mensajes, la comunicación entre Open Stage Control y el visual volvió a funcionar correctamente.

**Problema 3 - Unificación de servidores en una sola terminal**
Inicialmente, la aplicación funcionaba utilizando dos servidores ejecutándose en dos terminales diferentes. Esto generaba complejidad al momento de levantar el entorno y mantener la comunicación entre Strudel y OSC.

Para solucionar este problema, se realizó una modificación en el Bridge Server, permitiendo que tanto Strudel como OSC funcionaran desde un único servidor WebSocket y en una sola terminal.

El cambio principal consistió en modificar la función ```crearAdaptadores()```. Antes retornaba un único adaptador; ahora devuelve un arreglo de adaptadores dependiendo del modo seleccionado.

```
// Modo principal: Strudel + OSC en una sola terminal
// Ambos adaptadores se amarran al mismo servidor WebSocket.
// El cliente los distingue por msg.type ("strudel" vs "osc").
// Solo necesitas un puerto WebSocket y una terminal.

if (DEVICE === "strudel+osc") {
  log.info("Creando adaptador Strudel (ws://127.0.0.1:8080)");
  log.info(`Creando adaptador OSC     (UDP :${OSC_PORT})`);

  const adaptadorStrudel = new StrudelAdapter({ verbose: VERBOSE });
  const adaptadorOSC     = new OSCAdapter({ port: OSC_PORT, verbose: VERBOSE });

  return [ adaptadorStrudel, adaptadorOSC ];
}
```

**Con esta modificación:**

1. Si el modo es "strudel", la lista contiene un solo adaptador.
2. Si el modo es "strudel+osc", la lista contiene dos adaptadores.
3. Ambos adaptadores se conectan al mismo servidor WebSocket y al mismo puerto.
4. El cliente distingue el origen de los mensajes mediante msg.type.



## Bitácora de reflexión

<img width="600" height="694" alt="image" src="https://github.com/user-attachments/assets/a332e090-874c-4fb7-b4ce-9fc7987a869d" />

<img width="1421" height="371" alt="image" src="https://github.com/user-attachments/assets/f40d5a3f-f6a2-44dc-9e1a-1b7efc11f556" />


### ¿Por qué OSC no debe tratarse igual que Strude?

La diferencia fundamental es la semántica temporal de cada fuente. Strudel produce eventos discretos con un timestamp que dice "activa este visual en exactamente este instante". El sistema necesita una cola ordenada cronológicamente, una función que la revise en cada frame y una lógica de expiración: el flash dura delta segundos y desaparece. El tiempo es el dato más importante del mensaje.
OSC hace lo contrario. Un slider de Open Stage Control envía su valor actual cuando el usuario lo mueve, y ese valor no tiene fecha de vencimiento: simplemente describe cómo debe verse el sistema ahora y en adelante. Si el usuario mueve el slider de color a las 10:00 y no vuelve a tocarlo, a las 10:05 el círculo del bombo debe seguir con ese color. No hay nada que "expirar" ni ningún timestamp que comparar.
Si se tratara OSC igual que Strudel, habría que encolar mensajes sin timestamp, y la cola nunca sabría cuándo despacharlos. O peor: si se les inventara un timestamp de "ahora", se despacharían en el siguiente frame y se descartarían, haciendo que cada movimiento del slider fuera ignorado hasta el siguiente evento de Strudel.
La arquitectura lo refleja directamente: los mensajes Strudel van a strudelQueue[] y esperan su turno. Los mensajes OSC van directamente a oscState sobrescribiendo el valor anterior, sin cola, sin temporizador. drawRunning() lee oscState en cada frame igual que un valor de configuración, no como un evento puntual.

### Justifica los tres controles que elegiste.

**Control /rgb_1 — color del bombo**
El bombo es el golpe más frecuente y visualmente dominante en un patrón de batería típico. Darle control de color permite que el operador en vivo diferencie secciones de la pieza (intro, clímax, coda) con un cambio de paleta sin interrumpir el flujo. En drawRunning() se refleja directamente en el fill(r, g, b) del ellipse() del bombo: cada vez que llega un /rgb_1, el color del círculo cambia de inmediato y permanece hasta el siguiente mensaje. El rango 0–255 por canal da control cromático completo.


**Control /size — tamaño del bombo**
El radio del círculo es el parámetro más legible perceptivamente: un círculo pequeño crea tensión, uno grande domina el canvas. Mapear /size (0–1) a un diámetro de 20–200 px con map(oc.size, 0, 1, 20, 200) hace que el slider sea intuitivo: el tamaño visual escala linealmente con el gesto físico. Permite crear dinámicas de intensidad creciente (crescendo visual) o reducir la presencia del bombo sin silenciarlo.

**Control /posY — posición vertical del bombo**
La altura en el canvas genera relaciones espaciales con los otros elementos visuales (la caja centrada, el hi-hat en el cuarto superior). Subir el bombo rompe el eje vertical esperado y crea tensión; bajarlo ancla el sonido en la parte inferior del frame. El mapeo map(oc.posY, 0, 1, 0, height) hace que 0 = tope y 1 = base del canvas, correspondiendo a la dirección del slider en Open Stage Control (origin: bottom). El operador puede "escribir" con el bombo en el espacio vertical de la pieza.

**Integrar una tercera fuente de control**


*Lo que se conservaría sin cambios:*

```BaseAdapter.js``` establece el contrato que cualquier fuente nueva debe cumplir: ```connect()```, ```disconnect()```, ```onData```, ```onConnected```, ```onDisconnected.``` Es el único requisito para integrarse al sistema. La función ```amarrarAdaptador()``` en ```bridgeServer.js``` ya es genérica: recibe cualquier instancia de ```BaseAdapter``` y la conecta al servidor ```WS.``` ```BridgeClient.js``` ya enruta por ```msg.type```, y la ```FSM ``` con ```PainterTask``` ya distingue entre fuentes por el tipo de evento. No habría que tocar ninguno de estos componentes.


*Lo que se extendería:*

Se crearía un nuevo archivo ```NuevaFuenteAdapter.js``` que extiende ```BaseAdapter```. La única decisión es si el dato es discreto con ```timestamp``` (va a una cola, igual que Strudel), persistente sin ```timestamp``` (va a un estado similar a ```oscState```) o continuo en tiempo real (reemplaza el estado anterior, igual que ```micro:bit```). En ```bridgeServer.js``` se agregaría el adapter al array de ```crearAdaptadores()``` para el ```--device``` correspondiente. En ```sketch.js``` se añadiría una entrada en el switch de ```onData()``` para enrutar el nuevo ```msg.type```, y si es persistente, un nuevo objeto de estado análogo a ```oscState```. La función ```drawRunning()``` recibiría una referencia a ese estado y lo usaría en el render.
La arquitectura en capas (adaptador → servidor → cliente → FSM → render) hace que cada decisión esté localizada: agregar una fuente nueva no exige modificar las capas que ya funcionan.

