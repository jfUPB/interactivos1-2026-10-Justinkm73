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
        background('black');
        ellipse(x, height / 2, 100, 100);
        fill('red')

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
Luego se inicializa la comunicación serial, definiendo una velocidad de transmisión de 115200 baudios. Los botones A y B funcionaran con un evento, por eso usare el was_pressed con un sleep de 500 para que no se envien muchos datos.

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
let port; (Puerto)
let connectBtn; (botón que permite conectar y desconectar el puerto serial.)
let x; (usaremos X para almacenar la posición horizontal del círculo)
```
Luego en el setup se define la posición inicial de la variable X y creamos un ellipse donde su posición en su eje horizontal sera la variable X y en el vertical lo pondremos en el centro del Canva.
```
createCanvas(400, 400); (Creamos un canva)
  
    x = width / 2; (Posición inicial de la variable X)
    background(0); (Color del fondo)
    port = createSerial();
    connectBtn = createButton('Connect to micro:bit'); (Creamos un boton)
    connectBtn.position(width/3,300); (Damos posición al boton)
    connectBtn.mousePressed(connectBtnClick); (Cuando se presiones, aparecera que el boton se conecto)
    ellipse(x, height / 2, 100, 100); (Posición del ellipse usando en su eje horizontal la variable X)
```
Creamos la variable dataRx que es igual al valor permitido que entrara por el puerto. (A ó B) 
```
let dataRx = port.read(1);
```
Escribiremos esta linea de código que indica que si se presiona el bonton A este, se movera en la posición horizontal (Variable X) 10 pixeles hacia la derecha y si es B se movera hacia la izquierda la misma cantidad de pixeles.
```
if (dataRx == 'A') {
    x += 10;
} else if (dataRx == 'B') {
    x -= 10;
}
```
Le daremos un color al fondo y un color a la ellipse. Es importante ponerle un fondo ya que si no lo hacemos esto generara o dejara por decirlo así como una marca de agua. 

```
background('black');
ellipse(x, height / 2, 100, 100);
fill('red')
```
Acá utilizamos el lenguaje de programación html con el fín de mejorar la experiencia visualmente indicando que cuando el puerto este abierto lo conectemos y cuando este se conecte nos aparezca desconectarlo.
```
}
if (!port.opened()) {
        connectBtn.html('Connect to micro:bit');
}
else {
connectBtn.html('Disconnect');
}
}
```



#### NOTA:
#### 1. Es importante mantener el mismo nivel de baudios en ambos programas para que la comunicación funcione.
#### 2. Incluir en el Index.html de p5.js. la siguiente librería:
```
<script src="https://unpkg.com/@gohai/p5.webserial@^1/libraries/p5.webserial.js"></script>
```


### Actividad_06
Vas a repasar lo aprendido en esta unidad. Regresa a la actividad 4 y trata de explicar en tus propias palabras de la manera más detallada que puedas cómo funciona el sistema físico interactivo. Analiza cada parte del código y su función dentro del sistema. Si aún tienes dudas sobre alguna parte, aprovecha para aclararlas.


Se inicializa el micro:bit importando todas las funciones necesarias para su funcionamiento
```
from microbit import *
```

Creamos un bucle con la función de was_pressed, esta función nos ayudara a definir como funcionara nuestro sistema interactivo. La idea es crear un bucle para que está se repita cada vez que lo presionemos

```
while True:
      if button_a.was_pressed():
```

La idea es utilizar los baudios 115200 antes de enviar el mensaje para inicializar la comunicación serial, este tambien debe ser el mismo en el programa de p5.js y siguiendo este orden de idea enviaremos el mensaje A (uart.write('A'))
```
  from microbit import *

  uart.init(baudrate=115200)

  while True:
      if button_a.was_pressed():
          uart.write('A')
```

En el programa de p5.js. es importante incluir la librería:
```
<script src="https://unpkg.com/@gohai/p5.webserial@^1/libraries/p5.webserial.js"></script>
```

#### Entonces...
En el programa de p5.js. definos las variables del puerto, el boton para conectar el puerto y una condición inicial la cual sera false...
Pasamos al setup y creamos el canva, le damos un fondo y le decimos que cree un puerto y un boton con un mensaje indicando que conecte el boton, a este boton que es el de la variable boton le damos una posición y una acción... que cuando se de clic sobre este se conecte.

```
 let port;
  let connectBtn;
  let connectionInitialized = false;

  function setup() {
    createCanvas(400, 400);
    background(220);
    port = createSerial();
    connectBtn = createButton("Connect to micro:bit");
    connectBtn.position(80, 300);
    connectBtn.mousePressed(connectBtnClick);
  }
  ```

  En la función de draw, volvemos a pintar el canva y luego por buena practica del código limpiamos el frame por eso en la variable
  let connectionInitialized = false; para que cuando abra el puerto y la conexión incial sea falsa este limpie el puerto y luego inicie el la conexión.

```
function draw() {
background(220);

if (port.opened() && !connectionInitialized) {
port.clear();
connectionInitialized = true;
    }
```

A continuación empezaremos con la animación del recuadro, diciendole cuantos datos el tiene disponible para leer y que solo lea un dato, si el dato que llega es A el recuadro se pintara de rojo pero si el dato que llega es N el recuadro se pintara verde.

```
    if (port.availableBytes() > 0) {
      let dataRx = port.read(1);
      if (dataRx == "A") {
        fill("red");
      } else if (dataRx == "N") {
        fill("green");
      }
    }
```
Luego centraremos el recuadro y desde html le daremos estetica y texto al boton para conectar y desconectar, si el puerto esta abierto aparece que se puede conectar y cuando se conecte aparecera desconectar en el boton que creamos.

```
}

    rectMode(CENTER);
    rect(width / 2, height / 2, 50, 50);

    if (!port.opened()) {
      connectBtn.html("Connect to micro:bit");
    } else {
      connectBtn.html("Disconnect");
    }
  }

```


Aca le damos la función al boton que si está abierto se comunique por 115200 baudios y use el lenguaje MicroPython y la conexión, es un proceso de logica
```
 function connectBtnClick() {
    if (!port.opened()) {
      port.open("MicroPython", 115200);
      connectionInitialized = false;
    } else {
      port.close();
    }
  }
```




## Bitácora de aplicación 



## Bitácora de reflexión




