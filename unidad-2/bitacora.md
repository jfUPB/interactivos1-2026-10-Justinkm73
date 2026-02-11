# Unidad 2

## Bitácora de proceso de aprendizaje

### ACTIVIDAD_01

#### CÓDIGO:
```
from microbit import *
import utime

class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration

        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
                self.active = False
                self.owner.post_event(self.event)


class Pixel:
    def __init__(self,_x,_y,_interval):
        self.event_queue = []
        self.timers = []
        self.x = _x
        self.y = _y
        self.pixelState = 0
        self.myTimer = self.createTimer("Timeout",_interval)

        self.estado_actual = None
        self.transicion_a(self.estado_waitInON)

    def createTimer(self,event,duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t

    def post_event(self, ev):
        self.event_queue.append(ev)

    def update(self):
        # 1. Actualizar todos los timers internos automáticamente
        for t in self.timers:
            t.update()

        # 2. Procesar la cola de eventos resultante
        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)

    def transicion_a(self, nuevo_estado):
        if self.estado_actual: self.estado_actual("EXIT")
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")

    def estado_waitInON(self, ev):
        if ev == "ENTRY":
            self.pixelState = 9
            display.set_pixel(self.x,self.y,self.pixelState)
            self.myTimer.start()
        elif ev == "Timeout":
            self.transicion_a(self.estado_waitInOFF)

    def estado_waitInOFF(self, ev):
        if ev == "ENTRY":
            self.pixelState = 0
            display.set_pixel(self.x,self.y,self.pixelState)
            self.myTimer.start()
        elif ev == "Timeout":
            self.transicion_a(self.estado_waitInON)

pixel1 = Pixel(0,0,1000)
pixel2 = Pixel(4,4,600)

while True:
    pixel1.update()
    pixel2.update()
    utime.sleep_ms(20)
```

#### DIAGRAMA:

```
@startuml

<style>
root {
  BackgroundColor #4a556822
  FontColor #718096
  LineColor #718096
  Margin 30
  Padding 10
}

stateDiagram {
  state {
    BackgroundColor #edf2f7
    LineColor #4a5568
    FontColor #2d3748
    RoundCorner 10
  }
  arrow {
    LineColor #718096
    FontColor #718096
  }
}
</style>


title Pixel - UML State Machine

[*] --> WaitInON : Pixel() (constructor)
WaitInON : entry /\n  pixelState = 9\n  display.set_pixel(x,y,pixelState)\n  myTimer.start()
WaitInON --> WaitInOFF : \n Timeout /
WaitInOFF: entry /\n  pixelState = 0\n  display.set_pixel(x,y,pixelState)\n  myTimer.start()
WaitInOFF --> WaitInON : \n Timeout /

@enduml
```
    
### 1. ¿Cuáles son los estados en el programa?

### ESTADOS
```
def estado_waitInON(self, ev):
def estado_waitInOFF(self, ev)

```

### 2. ¿Cuáles son los eventos en el programa?

### EVENTOS
```
ENTRY
Timeout
EXIT
```

### 3. ¿Cuáles son las acciones en el programa?

### ACCIONES
```
self.pixelState = 0 # Define el brillo del LED (apagado).
self.pixelState = 9 # Define el brillo del LED (encendido).

display.set_pixel(self.x,self.y,self.pixelState) # Aplica físicamente el valor de brillo al LED.

self.myTimer.start() # Inicia el conteo del tiempo que el pixel permanecerá en este estado, cuando el tiempo termina, se genera el evento "Timeout", el tiempo que permanecera el pixel en cada estado se define en pixel1 = Pixel(0,0,1000), entendiendo esto le decimos en que posición ubicar el pixel y cuanto tiempo permanece el pixel en este estado sea ON/OFF.

# NOTA, COMO ESTÁ CONSTRUIDA Y FUNCIONA LA CLASE PIXEL
def __init__(self,_x,_y,_interval): # Es el constructor de la clase pixel, aquí se inicializan los atributos del objeto que se está creando, (Cuando hacemos = pixel1 = Pixel(0,0,1000) se crea el objeto) -> Self = representa ese objeto específico (Cada objeto creado tiene un self), _X = posición, _Y = posición, Interval =  tiempo que permanecerá en un estado antes de cambiar.

```

### ACTIVIDAD_02

#### 1. Vas a realizar una modificación. Cuando el semáforo esté en verde, si se presiona el botón A, el semáforo debe cambiar inmediatamente a amarillo (sin esperar a que termine el tiempo de verde). El evento que se debe postear es “A” (post_event(“A”)).

#### CÓDIGO A MODIFICAR:
```
from microbit import *
import utime

class Timer:
    def __init__(self, owner, event_to_post, duration):
        self.owner = owner
        self.event = event_to_post
        self.duration = duration

        self.start_time = 0
        self.active = False

    def start(self, new_duration=None):
        if new_duration is not None:
            self.duration = new_duration
        self.start_time = utime.ticks_ms()
        self.active = True

    def stop(self):
        self.active = False

    def update(self):
        if self.active:
            if utime.ticks_diff(utime.ticks_ms(), self.start_time) >= self.duration:
                self.active = False
                self.owner.post_event(self.event)


class Semaforo:
    def __init__(self,_x,_y,_timeInRed,_timeInGreen,_timeInYellow):
        self.event_queue = []
        self.timers = []
        self.x = _x
        self.y = _y
        self.timeInRed = _timeInRed
        self.timeInGreen = _timeInGreen
        self.timeInYellow = _timeInYellow
        self.myTimer = self.createTimer("Timeout",self.timeInRed)

        self.estado_actual = None
        self.transicion_a(self.estado_waitInRed)

    def createTimer(self,event,duration):
        t = Timer(self, event, duration)
        self.timers.append(t)
        return t

    def post_event(self, ev):
        self.event_queue.append(ev)

    def update(self):
        # 1. Actualizar todos los timers internos automáticamente
        for t in self.timers:
            t.update()

        # 2. Procesar la cola de eventos resultante
        while len(self.event_queue) > 0:
            ev = self.event_queue.pop(0)
            if self.estado_actual:
                self.estado_actual(ev)

    def transicion_a(self, nuevo_estado):
        if self.estado_actual: self.estado_actual("EXIT")
        self.estado_actual = nuevo_estado
        self.estado_actual("ENTRY")

    def clear(self):
        display.set_pixel(self.x,self.y,0)
        display.set_pixel(self.x,self.y+1,0)
        display.set_pixel(self.x,self.y+2,0)

    def estado_waitInRed(self, ev):
        if ev == "ENTRY":
            self.clear()
            display.set_pixel(self.x,self.y,9)
            self.myTimer.start(self.timeInRed)
        if ev == "Timeout":
            display.set_pixel(self.x,self.y,0)
            self.transicion_a(self.estado_waitInGreen)

    def estado_waitInGreen(self, ev):
        if ev == "ENTRY":
            self.clear()
            display.set_pixel(self.x,self.y+2,9)
            self.myTimer.start(self.timeInGreen)

        if ev == "Timeout":
            display.set_pixel(self.x,self.y+2,0)
            self.transicion_a(self.estado_waitInYellow)

        if ev == "A":
            display.set_pixel(self.x,self.y+2,0)
            self.transicion_a(self.estado_waitInYellow)

    def estado_waitInYellow(self, ev):
        if ev == "ENTRY":
            self.clear()
            display.set_pixel(self.x,self.y+1,9)
            self.myTimer.start(self.timeInYellow)
        if ev == "Timeout":
            display.set_pixel(self.x,self.y+1,0)
            self.transicion_a(self.estado_waitInRed)

semaforo1 = Semaforo(0,0,2000,1000,500)

while True:
    semaforo1.update()
    utime.sleep_ms(20)
```

#### Solución Punto #1

La maquina sabe que hacer si recibe A,

```
if ev == "A":
            display.set_pixel(self.x,self.y+2,0)
            self.transicion_a(self.estado_waitInYellow)
```

Pero nadie le envía ese evento, entonces agregamos la variable de la
detección del botón antes del Update, ya que el update:

1. Actualizar todos los timers internos automáticamente
2. Procesar la cola de eventos resultante

Entonces:
¿Por qué es buena práctica poner entradas antes?
Porque en sistemas reactivos el flujo ideal es:

1. Leer entradas
2. Generar eventos
3. Procesar eventos
4. Esperar

```
# Acción fisica
if button_a.was_pressed(A):
```
y postearlo en el evento A para que realice la acción del arreglo del pixel

```
while True:
    if button_a.was_pressed():
        semaforo1.post_event("A")

    semaforo1.update()
    utime.sleep_ms(20)
```

NOTA:
1. Una máquina de estados no debería:
- Leer botones
- Leer sensores
- Tomar entradas directamente
- Debe recibir eventos ya procesados.

Por eso el botón va afuera.

2. Hay dos tipos de transiciones:

- Transiciones temporizadas → dependen del timer
- Transiciones por evento externo → dependen de algo que ocurre

El botón A es del tipo 2. El cambio inmediato ocurre porque la transición por evento no espera tiempo.

3. función clear()
```
def clear(self):
    display.set_pixel(self.x,self.y,0)
    display.set_pixel(self.x,self.y+1,0)
    display.set_pixel(self.x,self.y+2,0)
```
En sistemas embebidos (como micro:bit) el hardware no se “auto-limpia”. Si tú enciendes algo, se queda encendido
hasta que tú lo apagues. Entonces una buena práctica es:

Antes de establecer un nuevo estado, asegúrate de limpiar el anterior, de lo contrario empezarían  a aparecer bugs visuales difíciles de rastrear.

4. En máquinas de estados:

Cuando entra, debe dejar el sistema exactamente como ese estado lo define. No debe depender de cómo estaba antes.

#### 2. Construye la máquina de estados que modela el problema usando PlantUML. Puedes encontrar el editor aquí y la documentación básica con ejemplos aquí.

#### Solución Punto #2
#### DIAGRAMA:

@startuml
title Semáforo - UML State Machine

[*] --> WaitInRed : Semaforo() (constructor)

WaitInRed : entry /\n  clear()\n  display.set_pixel(x,y,9)\n  myTimer.start(timeInRed)
WaitInRed --> WaitInGreen : Timeout

WaitInGreen : entry /\n  clear()\n  display.set_pixel(x,y+2,9)\n  myTimer.start(timeInGreen)
WaitInGreen --> WaitInYellow : Timeout
WaitInGreen --> WaitInYellow : A

WaitInYellow : entry /\n  clear()\n  display.set_pixel(x,y+1,9)\n  myTimer.start(timeInYellow)
WaitInYellow --> WaitInRed : Timeout

@enduml




## Bitácora de aplicación 



## Bitácora de reflexión


