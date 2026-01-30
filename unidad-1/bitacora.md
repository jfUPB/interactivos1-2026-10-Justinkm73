# Unidad 1

## Bitácora de proceso de aprendizaje
### Actividad_01 (21 DE ENERO, 2026)
#### ¿Qué es un sistema físico interactivo?
Para mí, un sistema físico interactivo es la relación y creación entre el cuerpo y el espacio, ya sea a través de una computadora, mediante código, o por medio de interacciones físicas en un entorno real y tangible. Esta relación permite la interactividad entre un elemento y otro, haciendo posible que ocurran acciones y respuestas reales.

#### ¿Cómo podrías aplicar lo que has visto en tu perfil profesional?
Desde mi perfil profesional puedo aplicar esto en muchas áreas del arte y la ciencia, como la creación de experiencias inmersivas para eventos, el uso de código creativo en videojuegos o proyectos musicales. Es un panorama muy amplio que me permite experimentar, mezclar disciplinas y crear propuestas artísticas y tecnológicas de forma más libre y cercana a las personas.

### Actividad_04 (28 DE ENERO, 2026)
#### ¿Por qué no funcionaba el programa con was_pressed() y por qué funciona con is_pressed()? Explica detalladamente.

#### PROGRAMA micro:bit.
```
from microbit import *

uart.init(baudrate=115200)

while True:

    //if button_a.was_pressed(): (Está es la función que se corrije) 
    if button_a.is_pressed(): (Está es la función correcta)
        uart.write('A')
    else:
        uart.write('N')

    sleep(100)
```
#### EXPLICACIÓN:
Tener en cuenta que:

- was_pressed = evento
- is_pressed = estado

Inicialmente se estaba utilizando was_pressed(), que funciona como un evento: detecta únicamente el momento en que el botón es presionado una vez y luego deja de responder hasta que ocurre un nuevo evento.

La acción solicitada para está actividad consistía en que el programa cambiara de color mientras el botón estuviera presionado y volviera a su color original al soltarlo. Para esta acción no es suficiente un evento, ya que se requiere información constante sobre el estado del botón.

la interacción requería un estado continuo y no un evento puntual, por está razón usamos el was_pressed
El cambio se realiza en el código del micro:bit.

### Actividad_05 (28 DE ENERO, 2026)
#### Crea un programa en p5.js que muestre un círculo en la pantalla. Utiliza los botones A y B del micro:bit para controlar la posición en x del círculo en el canvas de p5.js.

#### PROGRAMA P5.js.
```
let port;
let connectBtn;
let x;

function setup() {
    createCanvas(400, 400);
  
    x = width / 2;
    background(0);
    port = createSerial();
    connectBtn = createButton('Connect to micro:bit');
    connectBtn.position(width/3,300);
    connectBtn.mousePressed(connectBtnClick);
    fill('red');
    ellipse(x, height / 2, 100, 100);
    
}

function draw() {

    if(port.availableBytes() > 0){
        let dataRx = port.read(1);
        if(dataRx == 'A'){ 
          x += 10;
        }
  
        else if(dataRx == 'B'){
          x-=10;
        }
        background(0);
        ellipse(x, height / 2, 100, 100);
        fill('rgb(0,16,255)')

    }

    if (!port.opened()) {
        connectBtn.html('Connect to micro:bit');
    }
    else {
        connectBtn.html('Disconnect');
    }
}

function connectBtnClick() {
    if (!port.opened()) {
        port.open('MicroPython', 115200);
    } else {
        port.close();
    }
}
```
#### PROGRAMA micro:bit.
```
from microbit import *

uart.init(baudrate=115200)

while True:
    if button_a.was_pressed():
        uart.write('A')
        sleep(500)
    if button_b.was_pressed():
        uart.write('B')
        sleep(500)
```

#### EXPLICACIÓN:

Primero se inicializa el micro:bit importando todas las funciones necesarias para su funcionamiento:
```
from microbit import *
```
Luego se inicializa la comunicación serial, definiendo una velocidad de transmisión de 115200 baudios. luego los botones A y B funcionaran con un evento, por eso usare el was_pressed con un sleep de 500 para que no se envien muchos datos.

```
uart.init(baudrate=115200)
while True:
    if button_a.was_pressed():
        uart.write('A')
        sleep(500)
    if button_b.was_pressed():
        uart.write('B')
        sleep(500)
```

En p5.js, primero se declaran las variables principales:
```
let port;
let connectBtn; (botón que permite conectar y desconectar el puerto serial.)
let x; (almacena la posición horizontal del círculo)
```
Luego en el setup se define la posición inicial del círculo en el centro del eje horizontal y lo dibujamos usando la vairable x
```
x = width / 2;
ellipse(x, height / 2, 100, 100);
```
Continuamos diciendole que por el puerto entre un solo dato
```
let dataRx = port.read(1);
```
Luego continuamos escribiendo esta linea de código que indica que si se presiona el bonton A este, se movera en la posición horizontal 10 pixeles hacia la derecha y si es B se movera hacia la izquierda.
```
if (dataRx == 'A') {
    x += 10;
} else if (dataRx == 'B') {
    x -= 10;
}
```

#### NOTA:
Es importante mantener el mismo nivel de baudios en ambos programas para que la comunicación funcione.




## Bitácora de aplicación 



## Bitácora de reflexión



