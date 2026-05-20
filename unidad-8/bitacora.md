# Unidad 8

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 

### ITERACIÓN ESTETICA // Actividad #2

### CONCEPTO DE LA OBRA
--

*La Iglesia* es una performance de live coding audiovisual que usa la arquitectura visual y simbólica de una iglesia católica como materia prima para explorar tres estados de intensidad emocional: el orden representado (la nave, el vitral, la geometría sagrada), el cuerpo que sufre dentro de ese orden (los rostros, el llanto, el afán), y la violencia que lo funda (la crucifixión, la sangre, los clavos).

El referente sonoro es el dark ambient industrial: instrumental oscuro, textura de ruido blanco que se densifica hacia el clímax. La performance no ilustra ni explica la iglesia —la procesa, la desfigura y la reconstruye en tiempo real.

--

**El sistema tiene dos estados visuales que el performer navega durante la ejecución:**

**- Momento 1: (Puerta a la iglesia)**

Vitral gótico generativo. La arquitectura como símbolo: mandala, arcos, cruz de luz. El sistema está vivo pero en orden. La música lo pulsa, OSC lo tinta y lo escala, el cuerpo (micro:bit) puede romperlo.


**- Momento 2: (Puerta a la iglesia)**

Fotografías reales pixeladas como LEDs. La imagen documental degradada. El performer navega las fotos con el cuerpo, ajusta la escala con OSC. La cruz puede aparecer sobre las fotos como overlay audio-reactivo.

--

**La obra tiene tres capas de significado que corresponden a los tres niveles anotados en el boceto conceptual:**

**a.** Plano detalle, Acciones. La iglesia como institución simbólica: la geometría del vitral, el orden de los ritos, la arquitectura del poder.


**b.** Dolor, llanto, Rostros enojo. Los cuerpos dentro de esa institución: las fotografías de personas, de planos cerrados, de gestos. La experiencia humana atrapada en el símbolo.


**c.** Sangre, clavos, Cruzada. La violencia fundante.

--

<img width="1600" height="1011" alt="image" src="https://github.com/user-attachments/assets/0265240f-8142-4d60-89e1-51296cb59b41" />



---


### ROL DEL MICRO:BIT, STRUDEL & OPEN STAGE CONTROl
--

**Micro:bit**

Es la única fuente que puede cambiar el estado visual del sistema. Botón B alterna entre los dos momentos. Botón A cicla fotografías en M2. El acelerómetro controla zoom y scatter de píxeles. Sin micro:bit, la obra no tiene dirección ni navegación.

--

**Strudel**

Cada instrumento del patrón activa una zona visual distinta. Define cuáles partes del vitral o de la fotografía respiran, pulsan o se iluminan en cada momento.



**bd**  ->  Cruz latina central + halos de luz

**cp**  ->  Burbujas celulares en el campo interior del vitral

**saw (orbit 1)**  ->  Anillos del mandala 

**saw (orbit 2)**  ->  Partículas exteriores del mandala

**metal**  ->  Arcos de red del abanico superior

**sd**  ->  Nervios del abanico

--

**Open Stage Control**

OSC no activa eventos: sostiene el estado. El color de la luz, la escala del vitral, el blur, la aparición de la cruz sobre las fotos. Son parámetros que el performer ajusta y que permanecen hasta el siguiente cambio. 


**/rgb_1**  ->  **glassColor [r,g,b]**  ->  Tinte de la luz del vitral — define la temperatura emocional del Momento 1

**/size_1**  ->  **waveSize (0.5–2.0×)**  ->  Escala global del vitral 

**/trail_1**  ->  **blurEnabled (toggle)**  ->  Feedback luminoso en M1

**/cruz_1**  ->  **overlayActive (toggle)**  ->  Superpone la cruz audio-reactiva sobre las fotografías en M2

**/zoom_1**  ->  **zoom de foto**  ->  Acercamiento a la imagen en Momento 2

**/knob_1**  ->  **pixelSize LED**  ->  Densidad del efecto pixelado en las fotografía

**/photo_N**  ->  **foto activa**  ->  Selección directa de fotografía desde la interfaz OSC

--

**Combinación de las 3 Fuentes**

La cruz audio-reactiva sobre las fotografías requiere simultáneamente: que micro:bit (botón B) haya puesto el sistema en Momento 2, que OSC (/cruz_1) haya activado el overlay, y que Strudel esté enviando eventos bd que pulsen la zona nucleus. Ninguna de las tres fuentes por sí sola produce ese estado. Si el micro:bit está en M1, la cruz no aparece sobre las fotos aunque OSC la active. Si OSC no la activó, no aparece aunque el bd golpee. Si Strudel no envía bd, la cruz existe pero no pulsa. La imagen resultante —una fotografía de iglesia con una cruz luminosa que late al ritmo del bombo— es el producto de las tres fuentes en simultáneo.



---



### DESICIONES VISUALES, MUSICALES, PERFORMÁTICAS
--

**Desiciones visuales**

El vitral gótico no es una referencia decorativa: es la arquitectura misma del sistema visual. Cada función de dibujo (drawNucleus, drawConcentricMandala, drawFanWeb, drawCellularField) corresponde a un elemento arquitectónico real de una vidriera gótica. La decisión de usar la paleta sepia-cálida (glassColor: [195, 162, 118] por defecto) sitúa el visual en el tiempo de los materiales: vidrio envejecido, luz filtrada, opacidad acumulada.

La pixelación LED de las fotografías en M2 es una decisión deliberada de degradación: la imagen documental (la iglesia real, fotografiada) pasa por un filtro que la convierte en datos, en puntos de luz. El performer puede controlar cuánto se ve la imagen (zoom, pixelSize) y si la cruz aparece sobre ella o no.

El blur de persistencia de frames (/trail_1) en M1 crea rastro luminoso: el vitral no se borra entre frames sino que se acumula. A mayor waveSize, mayor probabilidad de desborde visual. Esa inestabilidad es intencional —el sistema puede volverse ilegible si el performer lo decide.

--

**Decisiones musicales**

La decisión de usar 180 bpm como base de referencia para la escritura en Strudel define una tensión constante. El dark ambient no es lento ni contemplativo: tiene urgencia. Los patrones están diseñados para activar zonas visuales de forma diferenciada —el bd no activa lo mismo que el metal, aunque ambos sean golpes percusivos. Esa diferenciación es la que hace que el visual sea polifónico y no una masa que parpadea uniforme.


**Tenemos tres momentos que coinciden con las capas conceptuales:**

**- Momento 1: (INTRO)**
Patrones dispersos, vitral en reposo, la geometría sagrada aparece lentamente.

**- Momento 2: (GROOVE)**
Los patrones se acumulan, el performer transita a M2, las fotos aparecen.

**- Momento 3: (CLIMAX)**
Ruido → Sonido. Todos los canales activos, la cruz sobre las fotos, el blur al máximo.

--

**Decisiones performáticas**

El micro:bit es visible durante la performance: el gesto de cambiar de momento es deliberadamente físico. El performer no oculta que está manipulando el sistema —lo contrario: el movimiento del cuerpo y el cambio visual son simultáneos y causales.

Strudel se toca en vivo: el live coding es parte de la performance. El performer puede alterar los patrones durante la ejecución, agregar o quitar instrumentos, cambiar densidades. El código es partitura abierta, no preset fijo.

Open Stage Control se opera en paralelo, ajustando el estado del espacio: el color de la luz, la escala del vitral, la aparición de la cruz. Son decisiones de dirección artística tomadas en tiempo real.



---



### CAMBIOS REALIZADOS ENTRE LA ITERACIÓN INGENIERIL Y LA ITERACIÓN ESTÉTICA
--

**Orbit como distinguidor de bass1/bass2:**

en la iteración ingenieril, ambos canales "saw" llegaban a la misma zona. Se agregó el campo orbit al payload normalizado del StrudelAdapter para que el frontend pueda bifurcarlos hacia inner (mandala) y cells (partículas), dando a cada línea de bajo una expresión visual distinta.

--

**Cruz como overlay en M2:**

la cruz no existía sobre las fotografías en la iteración técnica. Se agregó como elemento audio-reactivo que aparece solo cuando OSC lo habilita (/cruz_1), creando la convergencia de las tres fuentes descrita arriba.

--

**Paleta cromática definida como intención:**

El color por defecto del vitral ([195, 162, 118]) fue elegido como punto de partida cálido-sepia. En la iteración ingenieril era un placeholder; ahora es una decisión estética fundamentada en el referente material de los vitrales.

--

**Arco performático de 3 momentos:**

La iteración ingenieril verificaba que las fuentes llegaban al sistema. La iteración estética define qué hace el performer con ellas y en qué orden, transformando una demo técnica en una partitura de acciones.

--

**ProcessQueue siempre activo independiente del momento visual:**

se modificó drawRunning para que Strudel y tickZones se ejecuten incluso en M2, de modo que la cruz sea audio-reactiva con bd aunque las fotografías sean el visual principal.

--

**Selección directa de foto por OSC (/photo_N):**

se agregó control desde la interfaz OSC para seleccionar fotografías específicas sin depender solo del botón A del micro:bit, dando al performer más opciones de composición visual durante la ejecución.



---


### EVIDENCIAS DE ENSAYO

<img width="1319" height="712" alt="image" src="https://github.com/user-attachments/assets/96bbf274-8649-46ec-998d-9155d55e5430" />

--

<img width="1334" height="705" alt="image" src="https://github.com/user-attachments/assets/87b3c07c-d9ac-43b4-b323-e95400ba82ea" />

--

<img width="1334" height="708" alt="image" src="https://github.com/user-attachments/assets/bb0cfc14-8417-4178-b0e9-ad8450fce107" />

--

<img width="1333" height="711" alt="image" src="https://github.com/user-attachments/assets/4eef0599-c1d8-498d-87c1-67289155e5f0" />


---


### REVISIÓN TÉCNICA, BITÁCORA Y SUSTENTACIÓN // Actividad #3

### Diagrama de flujo de datos del sistema
<img width="912" height="1095" alt="Diagrama" src="https://github.com/user-attachments/assets/ee5541ad-8bcc-44a4-b24b-7a714c93a7e0" />



---



### Tabla de roles
<img width="1030" height="529" alt="image" src="https://github.com/user-attachments/assets/a6c240a8-442d-431a-8be6-69d1f724f370" />



---


### Recorrido: Adapter -> bridgeServer -> bridgeClient -> FSMTask -> updateLogic -> drawRunning

--

**Adapter** — Cada adaptador tiene una sola responsabilidad: hablar el idioma de su fuente y entregar un objeto normalizado. MicrobitBinaryAdapter habla serial binario. StrudelAdapter habla WS + JSON. OSCAdapter habla UDP + OSC. Todos entregan datos a bridgeServer llamando this.onData(normalizado).

--

**bridgeServer** — Recibe de cualquier adaptador y hace broadcast a todos los clientes WebSocket conectados en el puerto 8081. Formatea el mensaje con formatBroadcast() si el adaptador lo define, o usa el formato microbit por defecto. No toma decisiones visuales — solo reenvía.

--

**bridgeClient** — Corre en el navegador. Recibe el mensaje JSON del WS y lo enruta según msg.type: "microbit" → callback onData, "strudel" → onData, "osc" → onData. El sketch registra ese callback con bridge.onData(msg => ...).

--

**FSMTask** — El sketch convierte cada mensaje en un evento tipado y lo postea a la FSM (MICROBIT_DATA, STRUDEL_EVENT, OSC_CONTROL). La FSM tiene dos estados: estado_esperando (ignora datos, muestra animación idle) y estado_corriendo (procesa todo). En estado_corriendo, cada tipo de evento llama al método correcto.

--

**updateLogic** — Tres caminos paralelos, cada uno actualiza una parte del estado:

```_updateMicrobit``` → modifica activeVisual (btnB) o llama a ledVisual.handleMicrobit (btnA, acelerómetro)

```processQueue``` → consume la cola de eventos Strudel ordenada por timestamp y llama a pulseZone("nucleus"), pulseZone("tracery"), etc.

```_updateControl``` → actualiza controlState.glassColor, waveSize, blurEnabled con los valores OSC

--

**drawRunning** — Cada frame, p5.js llama draw(). Si la FSM está en estado_corriendo, se llama drawRunning(), que lee activeVisual, controlState y Z[zona].beat (que ya fueron actualizados por los tres caminos anteriores) y produce el visual. No recibe datos directamente — solo consume estado.



---



### Justificación de la propuesta estética y performática

El recorrido busca habitar un espacio atmosférico construido en torno a la iglesia, contrastando su definición y mirada desde lo liminal, lo profundo, la perfección buscada en sus espacios y el sacrificio que conlleva alcanzarla. El vitral abre el camino —inicio, perfección geométrica, anhelo de lo divino—; las fotografías, en cambio, invitan a leer el desgaste, el sacrificio, el dolor y la sangre. Una tensión sostenida entre la forma idealizada y la materia que la sostiene.

El sonido articula y guía este tránsito en tres momentos: intro, groove y clímax.



---



### Evidencias de pruebas y ensayos


## Bitácora de reflexión
