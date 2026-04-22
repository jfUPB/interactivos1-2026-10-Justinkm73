# Unidad 6

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación
### SOLUCIÓN ACTIVIDAD #2

**Cómo configuraste Strudel para emitir eventos;**


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



## Bitácora de reflexión
