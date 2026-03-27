# Unidad 5
## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 


## Bitácora de reflexión

### DESARROLLO ACTIVIDAD #3
## Respuesta:

# 1. Cuadro comparativo:

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



**2.** Puedo añadir un soporte para cada adapter ya que todos manejan el mismo contrato
`this.onData?.({ x, y, btnA, btnB })` 

Uno lo construye parseando texto con pipes (ASCII) y el otro leyendo bytes con `readInt16BE`, (BINARY) pero el resultado que sale es idéntico. Valor de x, y, btnA, btnB. El brigdServer no sabe que tipo de adapter o protocolo de comunicación estamos usando el solo recibe el objeto y lo envía por WebSocket.

El frontend (`bridgeClient.js`) recibe un JSON (Es una forma muy simple y ordenada de guardar y enviar datos.) (`{ type: "microbit", x, y, btnA, btnB }`) y dispara el evento `EVENTS.DATA` en  `sketch.js` y este a su vez en `updateLogic` lee los valores, como apesar de que se use otro adapter el contrato es el mismo para todos no presentara problemas por que leera son los valores que son los mismos para todos.

En conclusión no importa que adapter use lo que importa es los valores que estoy enviando y los que estoy recibiendo. Todos los adapters enviaran la misma información que soliciete el cliente sin importa que protocolo usemos.

`sketch.js` y `bridgeClient.js` nunca ven el protocolo crudo del hardware. No saben si llegaron bytes binarios o texto con pipes. Solo reciben el objeto ya limpio `{ x, y, btnA, btnB }` 

NOTAS:
En este sistema, **el frontend** es todo lo que corre en el navegador:

- `sketch.js` — la lógica de la FSM y el dibujo en p5.js
- `bridgeClient.js` — el que recibe los datos por WebSocket
- `index.html` — la página que los carga

Se le llama "frontend" porque es la **capa de cara al usuario**, la que muestra algo en pantalla. Vive en el navegador, no en Node.js.

En contraste, el **backend** es lo que corre en Node.js en tu computador:

- `bridgeServer.js` — el servidor WebSocket
- Los adapters (`MicrobitV2Adapter.js`, `MicrobitBinaryAdapter.js`) — los que leen el puerto serial




**3. Ejemplo 1 — Instalación de luz reactiva al kick de música Techno:**

Tengo un proyecto donde el BPM del kick activa y controla una luz mediante un estado de 1 (encendido) o 0 (apagado). Como el kick corre a 135 BPM, hay un cambio de estado constante y a alta frecuencia. Si intentara enviar esa información con ASCII, el volumen de datos podría saturar el canal y colapsar el hardware.

Por eso, **primero prototiparía con ASCII**: reduciría el tempo a un golpe cada 30 segundos para verificar en la terminal serial que el valor enviado es correctamente 0 o 1. Una vez confirmado, **implementaría el protocolo binario** para la ejecución real, ya que empaqueta los datos de forma compacta y permite sostener el ritmo 4/4 completo con cambios de BPM sin colapsar el sistema.

---

**Ejemplo 2 — Objeto interactivo musical con sensores de proximidad:**

Tengo una experiencia interactiva donde un objeto físico tiene sensores de proximidad que, según su valor, activan sonidos en el hardware.

**Primero prototiparía con ASCII**: con un solo sensor activo, verificaría en la terminal serial que los valores de proximidad que estoy enviando son correctos y tienen sentido antes de avanzar.

Una vez validado, **escalaría a binario**: al objeto se le añaden múltiples sensores enviando datos de proximidad simultáneamente. Con ASCII eso podría saturar el canal, pero empaquetando los datos en binario, toda esa información viaja de forma compacta y el hardware no colapsa cuando todos los sensores están activos al mismo tiempo.

---

**Conclusión:**
ASCII para prototipar y verificar, porque es legible directamente en la terminal serial. Binario para la ejecución real cuando el volumen y la frecuencia de datos es alto y complejo.


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
