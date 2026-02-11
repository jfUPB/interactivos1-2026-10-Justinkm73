# Unidad 2

## Bitácora de proceso de aprendizaje
### Actividad_01

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



## Bitácora de aplicación 



## Bitácora de reflexión
