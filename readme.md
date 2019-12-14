# Documentación TP2 Embebidos

El objetivo del presente trabajo práctico es hacer uso de la herramienta Yakindu. La misma permite desarrollar la lógica de una maquina de estados mediante una interfaz gráfica, definiendo estados, eventos y acciones. Luego, mediante la herramiente se genera el código correspondiente, que se integra facilmente en el programa definiendo funciones asociadas a las acciones y eventos.

## Uso de la herramienta

### Instalación
A través de Menú *Help → Install New Software*, usando el link http://updates.yakindu.org/sct/mars/releases/ se installa la herramienta en el IDE MCUXpresso.

### Migración del proyecto
Se copia el ejemplo *statecharts_bare_metal* provisto en el firmware_v2 en */projects/TP2*. Al igual que en el TP1, se configura el archivo *project.mk* para debuggear el archivo correspondiente sobre la EDU-CIAA.
Como se observa en la siguiente imagen, el diagrama de estados de Yakindu esta definido por dos archivos:
- *prefix.sct* correspondiente al modelo de Yakindu
- *prefix.sgen* que contiene informacion para la generación de codigo
En este último se configura el proyecto y las carpetas donde guardara el codigo y las librerias generados, según:
```
	targetProject = "firmware_v2"
	targetFolder = "projects/TP2/gen"
	libraryTargetFolder = "projects/TP2/gen"
```
![archivos_yakindu](https://github.com/mtorcasso/TP2/blob/master/img/archivos_yakindu.png)

### Edición del modelo y generación de codigo
Haciendo doble click en el modelo prefix.sct se abre la herramienta de Yakindu que permite editar el diagrama de estados. Haciendo click derecho sobre el modelo, y *Run Us -> 1 Satechart Simulation* se puede simular el modelo, induciendo eventos y observando la evolución del diagrama de estados.
Para generar el codigo, luego de configurar el archivo *prefix.sgen*, se hace click derecho sobre el mismo y se selecciona la opcion *Generate code artifacts*. Los archivos generados serán *Prefix.c*, *Prefix.h*, *PrefixRequired.h* y *sc_types.h* según se muestra en la imagen anterior.

### Debug de cada ejemplo
Los modelos asociados a los distintos ejemplos realizados en el trabajo práctico estan guardados con extensión *.-sct* para debuggear cada ejemplo se debe:
1. Copiar el modelo a la carpeta gen
2. Renombrar el modelo a *prefix.sct*
3. Generar el código acorde a lo explicado previamente
4. Editar la definición "TEST" en el main para coincidir con el modelo generado (ver imagen debajo)
5. Definir si se utilizarán Time Events mediante #define __USE_TIME_EVENTS (true/false)
5. Debuggear
![comp_condicional](https://github.com/mtorcasso/TP2/blob/master/img/comp_condicional.png)

Nota: Para generar el código con Yakindu, el proyecto debe estar previamente compilado; de lo contrario arrojará error.

## Ejemplo Blinky sin Time Events
En primera instancia se debe configurar en el *main.c* la compilación condicional según 
```#define TEST (SCT_1)```
Ademas, para el caso sin Time Events, se configura 
```#define __USE_TIME_EVENTS (false)```

### Diagrama de estados y main.c
![diagrama_blinky](https://github.com/mtorcasso/TP2/blob/master/img/diagrama_blinky.png)
![main_blinky](https://github.com/mtorcasso/TP2/blob/master/img/main_blinky.png)

### Funciones sin TimeEvents
- Los primeros tres llamados corresponden a las funciones *boardConfig(), tickConfig() y tickCallbackSet()* que ya fueron analizadas en el TP1, y corresponden a la configuración de puertos, de la frecuencia de interrupciones del tick, y de la ISR del tick, junto con la función *myTickHook()*
- Luego se realiza un llamado a *prefix_init(&statechart)*
![prefix_init](https://github.com/mtorcasso/TP2/blob/master/img/prefix_init.png)
Esta funcion inicializa la máquina de estados definiendo su estructura de datos interna. Además, mediante prefix_clearInEvents() setea todos los eventos en estado false.
- Luego se llama a *prefix_enter()* que tras sucesivos llamados configura la secuencia de entrada de la maquina de estados, en este caso configurando el estado APAGADO.
- En el bucle principal del programa, se espera a que ocurra una interrupción de tick, y en ese momento se ejecuta la función *prefixIface_raise_evTick(&statechart)* donde se setea true en el evento de tick, permitiendo a la maquina de estado cambiar de estado o ejecutar una acción.
![raise_tick](https://github.com/mtorcasso/TP2/blob/master/img/raise_tick.png)
- Mediante la función *prefix_runCycle()* se actualiza el estado y se ejecutan las acciones. Finalmente mediante la función *prefixIface_opLED()* definida por el usuario, se enciende o apaga el LED utilizando gpioWrite().
![prefix_runcycle](https://github.com/mtorcasso/TP2/blob/master/img/prefix_runcycle.png)

## Ejemplo Blinky con Time Events
En este caso, en vez de levantar un evento de tick cada vez que ocurra una interrupción, se alimentará directamente al statechart con una base de tiempo dada por el tick, e internamente se podrá utilizar esta base de tiempo en el diagrama de estado a traves de eventos predefinidos, como por ejemplo *"after 250ms"*, según se observa en la siguiente imagen.
![diagrama_blinkytime](https://github.com/mtorcasso/TP2/blob/master/img/diagrama_blinkytime.png)

### Funciones con TimeEvents
- Al comienzo del programa, se debe habilitar el uso de timeevents como se observa en la siguiente figura. Ademas se configura el numero de timers acorde a los eventos de tiempo que hay en el diagrama.
![timeevent_hab](https://github.com/mtorcasso/TP2/blob/master/img/timeevent_hab.png)

- Luego se definen las funciones *prefix_setTimer()* y *prefix_unsetTimer()* que configuran los timer para los distintos eventos, para la entrada y salida de un estado respectivamente.
![timer_set_unset](https://github.com/mtorcasso/TP2/blob/master/img/timer_set_unset.png)

- El resto de las inicializaciónes son equivalentes al caso sin TimeEvents, habiendo una diferencia unicamente en el bucle principal: en lugar de levantar un evento de tick para que la maquina cambie de estado, se realiza la siguiente operación:
1. Mediante la función *UpdateTimers()* se actualizan los contadores de timer
2. Se ejecuta un bucle evaluando la función *IsPendEvent()* que devuelve true cuando se cumplió el tiempo de un evento de timer
3. En ese caso, mediante *prefix_raiseTimeEvent()* se dispara un evento de timer
4. La función *MarkAsAttEvent()* setea el evento pendiente en false

En la siguiente figura se observa el bucle principal en caso de uso de TimeEvents:
![blucle_timeevent](https://github.com/mtorcasso/TP2/blob/master/img/blucle_timeevent.png)

## Generador de funciones
El objetivo del siguiente programa es simular el panel de control de un generador de funciones. El mismo permite seleccionar la forma de onda mediante un pulsador, seleccionando entre senoidal, triangular y cuadrada. Con otro pulsador se elige la opción que se desea modificar: frecuencia o magnitud. Finalmente posee dos pulsadores up/down para modificar estas caracteristicas. En la siguiente figura se muestra el bloque asociado a la detección de tecla pulsada, junto con el anti rebote:
![sct_gen_key](https://github.com/mtorcasso/TP2/blob/master/img/sct_gen_key.png)

El siguiente bloque dispara los eventos asociados a cada tecla:
![sct_gen_espera](https://github.com/mtorcasso/TP2/blob/master/img/sct_gen_espera.png)

Los siguientes bloques se encargan de cambiar el estado asociado a la forma de onda, y el estado asociado a la caracteristica que se modificará: frecuencia o magnitud. Ademas ejecuta la acción correspondiente al presionar el boton up o down.
![sct_gen_acciones](https://github.com/mtorcasso/TP2/blob/master/img/sct_gen_acciones.png)

Por último, las variables, acciones y eventos declarados:
![sct_gen_vareventacc.png](https://github.com/mtorcasso/TP2/blob/master/img/sct_gen_vareventacc.png)

Luego, en el main, bajo la opcion 4 de compilación condicional, se definen las funciones asociadas a las acciones:
![codigo_gen.png](https://github.com/mtorcasso/TP2/blob/master/img/codigo_gen.png)

Y en el bucle principal, se levanta el evento de presión de tecla junto con la variable button_status para detectar cual tecla fue presionada:
![codigo_gen2.png](https://github.com/mtorcasso/TP2/blob/master/img/codigo_gen2.png)

## Porton automático
El objetivo es implementar un portón de cochera con pulsadores de apertura y cierre, fines de carrera de porton abierto y cerrado, movimiento de motor y señal luminosa. Para la detección de los pulsadores y fines de carrera se utiiza el mismo mecanismo que el utilizado en el generador de funciones. Luego, se definen cuatro estados: porton abierto, cerrado, abriendose y cerrandose. En la siguiente figura se muestra el diagrama de estados correspondiente:
![sct_porton.png](https://github.com/mtorcasso/TP2/blob/master/img/sct_porton.png)

El codigo (opción 6 de compilacion condicional) asociado a los pulsadores y fines de carrera es el mismo que el mostrado para el caso del generador de funciones. El codigo asociado a las acciones se presenta en la siguiente figura:
![porton_codigo.png](https://github.com/mtorcasso/TP2/blob/master/img/porton_codigo.png)

## Horno microondas
El objetivo es implementar un horno microondas, con tres modos de operación: grill, microondas o micoondas+grill. Además tendrá una llave de "puerta abierta", que en caso de encenderse se detendrá el funcionamiento del horno, y cuando se cierre la puerta volvera a ejecutar la operación previa a la apertura. La deteccion de teclas (modo y start) y de la llave de puerta abierta es similar a la explicada para el generador de funciones. La unica diferencia, es que tambien se sensará cuando la llave de puerta abierta deje de estar oprimida, de forma tal de que el horno automáticamente reanude la operación. En la siguiente figura se observa esto:
![micro_botones.png](https://github.com/mtorcasso/TP2/blob/master/img/micro_botones.png)

En la siguiente figura se presenta la etapa asociada a la operación del horno. Al presionar el pulsador de modo se conmuta entre los distintos modos de operación. Al presionar el pulsador de start(stop) el horno inicia el modo seleccionado, o se detiene si ya estaba iniciado. Al abrir la puerta, se enciende la luz, y si el horno estaba activado se detiene. Luego al cerrar la puerta vuelve a funcionar en el modo seleccionado.
![micro_operacion.png](https://github.com/mtorcasso/TP2/blob/master/img/micro_operacion.png)

En la siguiente figura se muestra la declaración de variables, eventos y acciones:
![micro_var.png](https://github.com/mtorcasso/TP2/blob/master/img/micro_var.png)

Finalmente, el codigo de las funciones asociadas a las acciones:
![micro_codigo.png](https://github.com/mtorcasso/TP2/blob/master/img/micro_codigo.png)



