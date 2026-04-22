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

El bucle convierte el array en un objeto donde puedo acceder por nombre y no por posición:

```
// StrudelAdapter.js
_normalize(msg) {
    if (!msg || msg.address !== "/dirt/play") return null;

    // Convertir array plano a objeto clave-valor
    const args = {};
    const arr = Array.isArray(msg.args) ? msg.args : [];
    for (let i = 0; i + 1 < arr.length; i += 2) { //BucLe array
      args[String(arr[i])] = arr[i + 1];  //← esto convierte ['cps',0.5,'s','bd'] en {cps:0.5, s:'bd'}
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






## Bitácora de reflexión
