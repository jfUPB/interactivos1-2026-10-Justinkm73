# Unidad 1

## Bitácora de proceso de aprendizaje
### Actividad_01 (21 DE ENERO, 2026)
#### ¿Qué es un sistema físico interactivo?
Para mí, un sistema físico interactivo es la relación y creación entre el cuerpo y el espacio, ya sea a través de una computadora, mediante código, o por medio de interacciones físicas en un entorno real y tangible. Esta relación permite la interactividad entre un elemento y otro, haciendo posible que ocurran acciones y respuestas reales.

#### ¿Cómo podrías aplicar lo que has visto en tu perfil profesional?
Desde mi perfil profesional puedo aplicar esto en muchas áreas del arte y la ciencia, como la creación de experiencias inmersivas para eventos, el uso de código creativo en videojuegos o proyectos musicales. Es un panorama muy amplio que me permite experimentar, mezclar disciplinas y crear propuestas artísticas y tecnológicas de forma más libre y cercana a las personas.

### Actividad_04 (28 DE ENERO, 2026)
#### ¿Por qué no funcionaba el programa con was_pressed() y por qué funciona con is_pressed()? Explica detalladamente.

Tener en cuenta que:

- was_pressed = evento
- is_pressed = estado

```
CÓDIGO:
from microbit import *

uart.init(baudrate=115200)

while True:

    //if button_a.was_pressed(): (Está 
    if button_a.is_pressed(): (Está es la función correcta)
        uart.write('A')
    else:
        uart.write('N')

    sleep(100)
```

Inicialmente se estaba utilizando was_pressed(), que funciona como un evento: detecta únicamente el momento en que el botón es presionado una vez y luego deja de responder hasta que ocurre un nuevo evento.

La acción solicitada para está actividad consistía en que el programa cambiara de color mientras el botón estuviera presionado y volviera a su color original al soltarlo. Para esta acción no es suficiente un evento, ya que se requiere información constante sobre el estado del botón.

la interacción requería un estado continuo y no un evento puntual, por está razón usamos el was_pressed
El cambio se realiza en el código del micro:bit.

### Actividad_04 (28 DE ENERO, 2026)
#### ¿Por qué no funcionaba el programa con was_pressed() y por qué funciona con is_pressed()? Explica detalladamente.

## Bitácora de aplicación 



## Bitácora de reflexión


