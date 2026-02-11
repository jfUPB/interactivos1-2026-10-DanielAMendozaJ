# Unidad 1

## Bitácora de proceso de aprendizaje

### Actividad 01: ¿Qué es un sistema físico interactivo?

Un **Sistema Físico Interactivo (SFI)** es un conjunto de componentes físicos y/o digitales que responden a estímulos del entorno, transformando una **entrada (input)** en una **salida (output)** a través de algún tipo de **procesamiento**. Se usan comúnmente en instalaciones artísticas, videojuegos, experiencias de realidad aumentada y más.


### Actividad 02: ¿Qué es el diseño y el arte generativos?

El **diseño generativo** es una metodología que utiliza reglas, algoritmos y procesos automatizados para **generar resultados visuales o funcionales**. Parte de un principio similar a los sistemas físicos interactivos onde el diseñador actúa más como programador de comportamientos que como autor de una única pieza estática.
Los Sistemas Físicos Interactivos y el Arte Generativo comparten la estructura básica input → procesamiento → output, pero se aplican en contextos distintos: el primero busca generar experiencias sensibles a la interacción física, y el segundo permite automatizar la creación visual o artística mediante lógica y código.
Ambos permiten transformar la manera en que creamos, experimentamos y entendemos el mundo físico y digital.


### Actividad 03: Herramientas y tecnología

Se realizó una conexión entre **micro:bit** y **p5.js** mediante comunicación serial USB.  
Se cargó un programa en el micro:bit para enviar datos al presionar los botones o al detectar movimiento.  
Se configuró p5.js para recibir esos datos y modificar una figura en pantalla según la señal recibida.  
También se habilitó el envío de datos desde p5.js hacia el micro:bit para comprobar comunicación bidireccional.

**Resultados de las pruebas**

- **Presiona los botones A y B del micro:bit:**  
  Al presionar el botón `A` el círculo cambia a color rojo y muestra la letra "A".  
  Al presionar el botón `B` el círculo cambia a color amarillo y muestra la letra "B".

- **Sacude el micro:bit. ¿Qué pasa?**  
  Al sacudirlo, el sistema envía la letra `C`, el círculo cambia a color verde y se muestra la letra en pantalla.

- **Presiona el botón Send Love. ¿Qué pasa?**  
  Se envía la letra `h` al micro:bit, este muestra un corazón en su display y luego vuelve a la imagen anterior.


## Bitácora de aplicación 

### Actividad 4:

Rectangulo que cambia de color al presionar un botón (A en este caso)

En primer lugar recordemos que un sistema físico interactivo es el conjunto de Inputs, Procesamiento y Outputs; en este caso podemos dividir estos componentes dentro del programa:

Inputs: En este caso los botones generan la interacción y el mismo Micro:Bit es el que envía la información al programa.
Proceamiento: Micro:bit es quien envía los datos y P5js sería practicamente el que decide que dibujar.
Outputs: Recibimos el estímulo visual del cambio de color en el rectángulo.

🧭 Resumen paso a paso – ¿Cómo funciona el código?

1. **micro:bit (MicroPython)**
   - Se configura el puerto serial con `uart.init()`.
   - Dentro de un bucle infinito:
     - Si el botón A está presionado, se envía `"A"`.
     - Si no, se envía `"N"`.
   - Estos datos se transmiten al computador por USB cada 100 milisegundos.

2. **p5.js (JavaScript)**
   - Se crea un canvas y un botón para conectar el micro:bit.
   - Cada vez que se recibe un dato:
     - Si es `"A"`, cambia el color de relleno a **rojo**.
     - Si es `"N"`, cambia a **verde**.
   - El rectángulo se dibuja en el centro del canvas con el color correspondiente.

3. **Interacción visual**
   - El usuario presiona o suelta el botón A.
   - El sistema responde en tiempo real con un cambio de color.
   - Esta es la manifestación del **output** en el sistema interactivo.

**¿Por qué no funcionaba con `was_pressed()` y por qué sí funciona con `is_pressed()`?**

El problema radica en cómo se envían y reciben los datos a través del puerto serial.

Cuando se utilizó `button_a.was_pressed()`, el micro:bit enviaba la letra `"A"` **solo una vez**, en el momento exacto en que se detectaba el clic. Esto significa que p5.js recibía el mensaje únicamente en un frame.

En ese frame:
- Se leía el mensaje.
- El rectángulo cambiaba a rojo.

Sin embargo, en el siguiente frame ya no había mensajes disponibles en el puerto serial. Como el código estaba programado para pintar el rectángulo de verde cuando no había datos, el color volvía inmediatamente a verde.
Es decir, el cambio de color duraba solo un frame (una fracción de segundo).
En cambio, cuando se utilizó `button_a.is_pressed()`, el micro:bit envía constantemente `"A"` mientras el botón está presionado, y `"N"` cuando no lo está.

Esto genera un flujo continuo de información:
- Si el botón está presionado → se envía `"A"` repetidamente.
- Si el botón no está presionado → se envía `"N"` repetidamente.

De esta manera, p5.js siempre tiene un estado actual del botón y puede mantener el color correcto en cada frame.

**Conclusión**

`was_pressed()` detecta un evento puntual (un clic único).  
`is_pressed()` representa un estado continuo (botón presionado o no).

En sistemas físicos interactivos donde el dibujo se actualiza constantemente en cada frame, es necesario trabajar con estados continuos y no con eventos únicos, para evitar comportamientos visuales inconsistentes o pasará lo mismo que en este caso.


### Actividad 5:

Escribe el enlace a tu programa en el editor de p5.js.

[Mueve la bola](https://editor.p5js.org/DanielAMendozaJ/sketches/4E8K_HAig)


Copia el código de tu programa p5js en la bitácora.

```javascript
let port; // Variable que almacenará la conexión serial con el micro:bit
let connectBtn; // Variable que almacenará el botón para conectar/desconectar
let connectionInitialized = false; // Indica si ya se inicializó la conexión (evita limpiar el puerto cada frame)
let circleX; // Guarda la posición horizontal del círculo

function setup() {
  createCanvas(400, 400); // Crea un lienzo
  background(220); // Pinta el fondo

  circleX = width / 2; // Posiciona el círculo inicialmente en el centro horizontal

  port = createSerial(); // Crea el objeto que permite la comunicación serial
  connectBtn = createButton("Connect to micro:bit"); // Crea el botón de conexión
  connectBtn.position(80, 300); // Posiciona el botón en el canvas
  connectBtn.mousePressed(connectBtnClick); // Asigna la función que se ejecuta al hacer clic
}

function draw() {
  background(220); // Limpia el canvas en cada frame para redibujar

  if (port.opened() && !connectionInitialized) { // Si el puerto está abierto y no se ha inicializado
    port.clear(); // Limpia datos acumulados en el buffer serial
    connectionInitialized = true; // Marca que la conexión ya fue inicializada
  }

  if (port.availableBytes() > 0) { // Si hay datos disponibles en el puerto
    let dataRx = port.read(1); // Lee 1 byte del puerto serial

    if (dataRx == "I") { // Si recibe "I" (que representa al botón A en el Micro:bit)
      circleX -= 10; // Mueve el círculo 10 píxeles hacia la izquierda
    } else if (dataRx == "D") { // Si recibe "D" (que representa al botón B en el Micro:bit)
      circleX += 10; // Mueve el círculo 10 píxeles hacia la derecha
    }
  }

  circleX = constrain(circleX, 0, width); // Evita que el círculo salga del canvas horizontalmente

  fill("rgb(255,0,254)"); // Define el color del círculo
  noStroke(); // Elimina el borde del círculo
  ellipse(circleX, height / 2, 50, 50); // Dibuja el círculo en la posición actual

  if (!port.opened()) { // Si el puerto NO está abierto
    connectBtn.html("Connect to micro:bit"); // Muestra texto para conectar
  } else { // Si el puerto está abierto
    connectBtn.html("Disconnect"); // Muestra texto para desconectar
  }
}

function connectBtnClick() {
  if (!port.opened()) { // Si el puerto está cerrado
    port.open("MicroPython", 115200); // Abre la conexión serial con el micro:bit
    connectionInitialized = false; // Reinicia la bandera para limpiar el buffer al conectar, evita que se acumulen datos viejos.
  } else { // Si el puerto está abierto
    port.close(); // Cierra la conexión serial
  }
}
```

Copia el código del micro:bit en la bitácora.

```python
from microbit import *

uart.init(baudrate=115200)
display.show(Image.DIAMOND)

while True:
    if button_a.is_pressed():
        uart.write('I') # Mover Izquierda
    if button_b.is_pressed():
        uart.write('D') # Mover Derecha
    else:
        uart.write('N')  # Enviar 'N' si no se presiona ningún botón (indiferente)
    sleep(100)
```

**Explicación detallada del Sistema Físico Interactivo**

El sistema creado permite controlar el movimiento horizontal de un círculo en pantalla utilizando los botones físicos del micro:bit. Este sistema sigue la estructura clásica:
Input → Procesamiento → Output

**1. Input (Entrada)**

El input es generado por el usuario al presionar los botones físicos del micro:bit:
- Botón A → envía la letra `'I'` (mover a la izquierda).
- Botón B → envía la letra `'D'` (mover a la derecha).
- Si no se presiona ningún botón → se envía `'N'`.

El micro:bit detecta el estado de los botones mediante `is_pressed()` y transmite continuamente la información por el puerto serial cada 100 milisegundos.

**2. Procesamiento**

El procesamiento ocurre principalmente en p5.js:
- El programa verifica si hay datos disponibles en el puerto serial.
- Si recibe `'I'`, disminuye la posición `circleX`.
- Si recibe `'D'`, aumenta la posición `circleX`.
- Luego limita el movimiento con `constrain()` para evitar que el círculo salga del canvas.

Este procesamiento transforma datos seriales en cambios numéricos dentro del entorno gráfico.
También existe un procesamiento inicial en el micro:bit, que interpreta el estado físico de los botones y decide qué carácter enviar.

**3. Output (Salida)**

El output es visual:
- Un círculo que se mueve horizontalmente en el canvas.
- El movimiento ocurre en tiempo real mientras el botón esté presionado.

El sistema responde inmediatamente a la acción física del usuario, generando una experiencia interactiva directa.

**Funcionamiento general del sistema**

1. El usuario presiona un botón físico.
2. El micro:bit detecta el estado y envía un carácter por USB.
3. p5.js recibe ese carácter.
4. Se actualiza la posición del círculo.
5. El canvas se redibuja mostrando el nuevo estado.

Este flujo ocurre constantemente dentro del bucle `draw()`, lo que permite que la interacción sea continua y fluida.

**Conclusión**

El sistema demuestra cómo un dispositivo físico puede controlar un entorno digital mediante comunicación serial. Se establece una relación directa entre acción física y respuesta visual, lo que constituye un Sistema Físico Interactivo básico pero funcional.


## Bitácora de reflexión

### Actividad 06: Análisis detallado del Sistema Físico Interactivo (Actividad 04)

En la Actividad 04 se desarrolló un sistema físico interactivo en el que un botón del micro:bit controla el color de un cuadrado en p5.js. El objetivo no era solo lograr que funcionara, sino comprender cómo fluye la información entre hardware y software y cómo se mantiene un estado en el tiempo.

Este sistema sigue la estructura fundamental:
Input → Procesamiento → Output

**1. Arquitectura general del sistema**

El sistema está compuesto por dos partes principales:

- **Micro:bit (hardware + MicroPython)**
- **p5.js (entorno gráfico en el navegador)**

Ambos se comunican mediante **comunicación serial USB**, utilizando la misma velocidad de transmisión (115200 baudios).

**2. Análisis del código del micro:bit**

```python
from microbit import *  # Importa todas las funciones necesarias para usar el micro:bit

uart.init(baudrate=115200)  # Inicializa la comunicación serial a 115200 baudios (velocidad de transmisión de datos)

while True:  # Bucle infinito que se ejecuta constantemente
    if button_a.is_pressed():  # Si el botón A está presionado en este momento
        uart.write('A')  # Envía la letra "A" por el puerto serial
    else:  # Si el botón NO está presionado
        uart.write('N')  # Envía la letra "N" por el puerto serial
    sleep(100)  # Espera 100 milisegundos para evitar enviar datos demasiado rápido
```
Micro:bit nos permite aquí conectarnos con el hardware que tenemos y mediante un proceso infinito (While True) estar al pendiente de nuestras acciones como al presionar los botones del Micro:bit identificarlos y escribir un dato según el botón presionado. esto nos servirá despues para cambiar el color del cuadrado

**3. Análisis del código en p5.js**

```python
let port; // Variable global que almacenará el objeto de comunicación serial
let connectBtn; // Variable global que almacenará el botón de conexión
let connectionInitialized = false; // Bandera para saber si ya limpiamos el buffer al conectar

function setup() {
  createCanvas(400, 400); // Crea un canvas de 400x400 píxeles
  background(220); // Color de fondo gris claro

  port = createSerial(); // Crea el objeto que maneja la comunicación serial
  connectBtn = createButton("Connect to micro:bit"); // Crea un botón en pantalla
  connectBtn.position(80, 300); // Posiciona el botón en el canvas
  connectBtn.mousePressed(connectBtnClick); // Cuando se hace click ejecuta connectBtnClick()
}
function draw() {
  background(220); // Limpia la pantalla en cada frame

  if (port.opened() && !connectionInitialized) { // Si el puerto está abierto y aún no hemos inicializado la conexión

    port.clear(); // Limpia el buffer serial para eliminar datos viejos
    connectionInitialized = true; // Marcamos que ya se limpió
  }

  if (port.availableBytes() > 0) { // Si hay datos disponibles en el puerto serial
    
    let dataRx = port.read(1); // Lee 1 byte (1 carácter) del puerto serial
    
    if (dataRx == "A") { 
      fill("red"); // Si recibe "A", el cuadrado será rojo
    } else if (dataRx == "N") { 
      fill("green"); // Si recibe "N", el cuadrado será verde
    }
  }

  rectMode(CENTER); // Hace que el rectángulo se dibuje desde el centro
  
  rect(width / 2, height / 2, 50, 50); // Dibuja el cuadrado en el centro del canvas
  
  if (!port.opened()) { 
    connectBtn.html("Connect to micro:bit"); // Si el puerto está cerrado, el botón dice "Connect"
  } else { 
    connectBtn.html("Disconnect"); // Si el puerto está abierto, el botón dice "Disconnect"
  }
}

function connectBtnClick() {
  if (!port.opened()) { // Si el puerto está cerrado
    port.open("MicroPython", 115200); // Abre la conexión serial usando MicroPython a 115200 baudios
    connectionInitialized = false; // Reinicia la bandera para que el buffer se limpie nuevamente
  } else { // Si el puerto está abierto
    port.close(); // Cierra la conexión serial    
  }
}
```
**4. El error inicial y su análisis**

Cuando se usó was_pressed() el sistema fallaba.

Porque:
was_pressed() envía 'A' solo una vez.
p5.js recibe el dato en un solo frame.
En el siguiente frame no hay datos.
El cuadrado vuelve a verde inmediatamente.
El error no era de conexión, sino de modelo mental del sistema.
p5.js funciona con actualización constante (frames).
Por lo tanto, necesita un flujo continuo de estado.

La solución? cambiar a is_pressed() y enviar información constantemente.

**5. Reflexión final**

Este ejercicio permitió comprender que un sistema físico interactivo no solo depende del hardware y el software, sino de cómo se gestionan:

El tiempo.
Los estados.
La comunicación.
La sincronización entre dispositivos.

La clave no fue solo cambiar una función, sino entender la diferencia entre eventos y estados continuos.
Un sistema físico interactivo exitoso requiere pensar en flujo de información constante, no en acciones aisladas.
