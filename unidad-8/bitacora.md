# Unidad 8

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 

### Actividad #2 ITERACIÓN ESTETICA

#### CONCEPTO DE LA OBRA

*La Iglesia* es una performance de live coding audiovisual que usa la arquitectura visual y simbólica de una iglesia católica como materia prima para explorar tres estados de intensidad emocional: el orden representado (la nave, el vitral, la geometría sagrada), el cuerpo que sufre dentro de ese orden (los rostros, el llanto, el afán), y la violencia que lo funda (la crucifixión, la sangre, los clavos).

El referente sonoro es el dark ambient industrial: instrumental oscuro, textura de ruido blanco que se densifica hacia el clímax. La performance no ilustra ni explica la iglesia —la procesa, la desfigura y la reconstruye en tiempo real.



**El sistema tiene dos estados visuales que el performer navega durante la ejecución:**

**- Momento 1: (Puerta a la iglesia)**

Vitral gótico generativo. La arquitectura como símbolo: mandala, arcos, cruz de luz. El sistema está vivo pero en orden. La música lo pulsa, OSC lo tinta y lo escala, el cuerpo (micro:bit) puede romperlo.


**- Momento 2: (Puerta a la iglesia)**

Fotografías reales pixeladas como LEDs. La imagen documental degradada. El performer navega las fotos con el cuerpo, ajusta la escala con OSC. La cruz puede aparecer sobre las fotos como overlay audio-reactivo.



**La obra tiene tres capas de significado que corresponden a los tres niveles anotados en el boceto conceptual:**

a. Plano detalle, Acciones. La iglesia como institución simbólica: la geometría del vitral, el orden de los ritos, la arquitectura del poder.


b. Dolor, llanto, Rostros enojo. Los cuerpos dentro de esa institución: las fotografías de personas, de planos cerrados, de gestos. La experiencia humana atrapada en el símbolo.


c. Sangre, clavos, Cruzada. La violencia fundante.



<img width="1600" height="1011" alt="image" src="https://github.com/user-attachments/assets/0265240f-8142-4d60-89e1-51296cb59b41" />



---


#### ROL DEL MICRO:BIT, STRUDEL & OPEN STAGE CONTROl

**Micro:bit**

Es la única fuente que puede cambiar el estado visual del sistema. Botón B alterna entre los dos momentos. Botón A cicla fotografías en M2. El acelerómetro controla zoom y scatter de píxeles. Sin micro:bit, la obra no tiene dirección ni navegación.



**Strudel**

Cada instrumento del patrón activa una zona visual distinta. Define cuáles partes del vitral o de la fotografía respiran, pulsan o se iluminan en cada momento.



**bd**  ->  Cruz latina central + halos de luz

**cp**  ->  Burbujas celulares en el campo interior del vitral

**saw (orbit 1)**  ->  Anillos del mandala 

**saw (orbit 2)**  ->  Partículas exteriores del mandala

**metal**  ->  Arcos de red del abanico superior

**sd**  ->  Nervios del abanico



**Open Stage Control**

OSC no activa eventos: sostiene el estado. El color de la luz, la escala del vitral, el blur, la aparición de la cruz sobre las fotos. Son parámetros que el performer ajusta y que permanecen hasta el siguiente cambio. 


**/rgb_1**  ->  **glassColor [r,g,b]**  ->  Tinte de la luz del vitral — define la temperatura emocional del Momento 1

**/size_1**  ->  **waveSize (0.5–2.0×)**  ->  Escala global del vitral 

**/trail_1**  ->  **blurEnabled (toggle)**  ->  Feedback luminoso en M1

**/cruz_1**  ->  **overlayActive (toggle)**  ->  Superpone la cruz audio-reactiva sobre las fotografías en M2

**/zoom_1**  ->  **zoom de foto**  ->  Acercamiento a la imagen en Momento 2

**/knob_1**  ->  **pixelSize LED**  ->  Densidad del efecto pixelado en las fotografía

**/photo_N**  ->  **foto activa**  ->  Selección directa de fotografía desde la interfaz OSC



**Combinación de las 3 Fuentes**

La cruz audio-reactiva sobre las fotografías requiere simultáneamente: que micro:bit (botón B) haya puesto el sistema en Momento 2, que OSC (/cruz_1) haya activado el overlay, y que Strudel esté enviando eventos bd que pulsen la zona nucleus. Ninguna de las tres fuentes por sí sola produce ese estado. Si el micro:bit está en M1, la cruz no aparece sobre las fotos aunque OSC la active. Si OSC no la activó, no aparece aunque el bd golpee. Si Strudel no envía bd, la cruz existe pero no pulsa. La imagen resultante —una fotografía de iglesia con una cruz luminosa que late al ritmo del bombo— es el producto de las tres fuentes en simultáneo.



---



#### DESICIONES VISUALES, MUSICALES, PERFORMÁTICAS

**Desiciones visuales**

El vitral gótico no es una referencia decorativa: es la arquitectura misma del sistema visual. Cada función de dibujo (drawNucleus, drawConcentricMandala, drawFanWeb, drawCellularField) corresponde a un elemento arquitectónico real de una vidriera gótica. La decisión de usar la paleta sepia-cálida (glassColor: [195, 162, 118] por defecto) sitúa el visual en el tiempo de los materiales: vidrio envejecido, luz filtrada, opacidad acumulada.

La pixelación LED de las fotografías en M2 es una decisión deliberada de degradación: la imagen documental (la iglesia real, fotografiada) pasa por un filtro que la convierte en datos, en puntos de luz. El performer puede controlar cuánto se ve la imagen (zoom, pixelSize) y si la cruz aparece sobre ella o no.

El blur de persistencia de frames (/trail_1) en M1 crea rastro luminoso: el vitral no se borra entre frames sino que se acumula. A mayor waveSize, mayor probabilidad de desborde visual. Esa inestabilidad es intencional —el sistema puede volverse ilegible si el performer lo decide.


**Decisiones musicales**

La decisión de usar 180 bpm como base de referencia para la escritura en Strudel define una tensión constante. El dark ambient no es lento ni contemplativo: tiene urgencia. Los patrones están diseñados para activar zonas visuales de forma diferenciada —el bd no activa lo mismo que el metal, aunque ambos sean golpes percusivos. Esa diferenciación es la que hace que el visual sea polifónico y no una masa que parpadea uniforme.

**Tenemos tres momentos que coinciden con las capas conceptuales:**

**- Momento 1:**
Patrones dispersos, vitral en reposo, la geometría sagrada aparece lentamente.

**- Momento 2:**
Los patrones se acumulan, el performer transita a M2, las fotos aparecen.

**- Momento 3:**
Ruido → Sonido. Todos los canales activos, la cruz sobre las fotos, el blur al máximo.





## Bitácora de reflexión
