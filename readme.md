# Documentación TP2 Embebidos

El objetivo del presente trabajo práctico es hacer uso de la herramienta Yakindu. La misma permite desarrollar la lógica de una maquina de estados mediante una interfaz gráfica, definiendo estados, eventos y acciones. Luego, mediante la herramiente se genera el código correspondiente, que se integra facilmente en el programa definiendo funciones asociadas a las acciones y eventos.

## Uso de la herramienta

### Instalación
A través de Menú *Help → Install New Software*, usando el link http://updates.yakindu.org/sct/mars/releases/ se installa la herramienta en el IDE MCUXpresso.

### Migración del proyecto
Se copia el ejemplo *statecharts_bare_metal* provisto en el firmware_v2 en */projects/TP2*. Al igual que en el TP!, se configura el archivo *project.mk* para debuggear el archivo correspondiente sobre la EDU-CIAA.
Como se observa en la siguiente imagen, el diagrama de estados de Yakindu esta definido por dos archivos:
- *Prefix.sct* correspondiente al modelo de Yakindu
- *Prefix.sgen* que contiene informacion para la generación de codigo
En este último se configura el proyecto y las carpetas donde guardara el codigo y las librerias generados, según:
```
			targetProject = "firmware_v2"
			targetFolder = "projects/TP2/gen"
			libraryTargetFolder = "projects/TP2/gen"
```
![archivos_yakindu](https://github.com/mtorcasso/TP1/blob/master/img/archivos_yakindu.png)