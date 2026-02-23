# Unidad 2

## Bitácora de proceso de aprendizaje

### Actividad 01 – Análisis de una Máquina de Estados Simple**

El programa controla dos píxeles del micro:bit que parpadean a diferentes velocidades utilizando una máquina de estados y una clase Timer para manejar el tiempo sin bloquear la ejecución (sin usar sleep()).
Cada objeto Pixel funciona como una máquina de estados independiente.

**¿Cuáles son los estados en el programa?**
Primera versión (un solo estado)

En la primera implementación existe un único estado:
estado_waitTimeout
Este estado:
En ENTRY enciende el píxel.
En Timeout cambia el valor del píxel (ON ↔ OFF).
Aquí el comportamiento ON/OFF se controla dentro del mismo estado, usando una condición sobre pixelState.

**Segunda versión (modelo más claro)**
En la segunda implementación existen dos estados explícitos:
estado_waitInON
estado_waitInOFF
Aquí el comportamiento es más limpio porque:
estado_waitInON → El píxel está encendido.
estado_waitInOFF → El píxel está apagado.
Cada estado representa claramente una condición del sistema.

**¿Cuáles son los eventos en el programa?**

Los eventos que maneja la máquina de estados son:
"ENTRY" → Se ejecuta al entrar a un estado.
"EXIT" → Se ejecuta al salir de un estado.
"Timeout" → Evento generado por el Timer cuando se cumple el tiempo configurado.

El evento más importante del sistema es:

```python
"Timeout"
```
Este evento es generado automáticamente por la clase Timer cuando el tiempo se cumple:
```python
self.owner.post_event(self.event)
```
Es decir:
El temporizador detecta que pasó el tiempo.
Publica el evento "Timeout".
La máquina de estados lo procesa.

**¿Cuáles son las acciones en el programa?**

Las acciones son las operaciones que ejecuta el sistema cuando ocurre un evento.
Las principales acciones son:

Encender el píxel
```python
self.pixelState = 9
display.set_pixel(self.x,self.y,self.pixelState)
```
Apagar el píxel
```python
self.pixelState = 0
display.set_pixel(self.x,self.y,self.pixelState)
```
Iniciar el temporizador
```python
self.myTimer.start()
```
Transicionar a otro estado (segunda versión)
```python
self.transicion_a(self.estado_waitInOFF) /// o "self.transicion_a(self.estado_waitInON)"
```

**Reflexión**
Este ejercicio demuestra cómo una máquina de estados permite modelar comportamientos temporizados sin bloquear la ejecución del programa.
A diferencia del uso de sleep(), aquí el sistema sigue funcionando de manera concurrente porque:
Los temporizadores no detienen el programa.
Cada objeto Pixel gestiona sus propios eventos.
El while True solo coordina actualizaciones periódicas.
La segunda versión del modelo es más clara porque separa explícitamente los estados:
ON
OFF

Esto mejora:
La legibilidad
La escalabilidad
La mantenibilidad del código


## Bitácora de aplicación 



## Bitácora de reflexión
