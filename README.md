# Alferez-post1-u3
# Parte 1

En la salida del comando D (Dump) de DEBUG, la primera columna muestra la dirección de memoria en formato segmento:desplazamiento donde comienzan los datos de cada fila. La segunda columna contiene los datos almacenados en memoria representados en hexadecimal, normalmente 16 bytes por línea. La tercera columna muestra la representación ASCII de esos mismos bytes cuando los caracteres son imprimibles; los que no lo son aparecen como puntos (.). En resumen, la salida permite ver simultáneamente dónde están los datos, su valor hexadecimal y su equivalente ASCII.

## Parte 1, Paso 12: Verificación no destructiva de una escritura en memoria

El estudiante determina que el comando correcto para confirmar la escritura del Paso 11 es D, porque es el único de los cuatro que consulta el estado sin poder alterarlo. Invocar E 300 sin la lista de bytes no es seguro: el DEBUG entra en modo interactivo, muestra el valor actual del primer byte y espera una tecla. Cualquier valor tecleado por descuido sobrescribe esa posición, de modo que la verificación podría corromper el dato que se quiere comprobar. En el Paso 11, en cambio, E recibió la lista completa (78 56) y terminó de inmediato, sin ofrecer esa posibilidad de error. F tampoco sirve, aunque opere sobre un rango como D, porque su función es escribir: repite un patrón sobre todo el rango y sobrescribiría los bytes 78 56 que se desea verificar. R no muestra la memoria, así que no permite comprobar nada sobre DS:0300, y con un nombre de registro (por ejemplo R AX) también permite modificar su valor. D solo lee los bytes del rango indicado y los presenta en hexadecimal y ASCII, sin escribir en memoria ni cambiar registros. Esa propiedad es la que se necesita aquí: el estudiante puede repetir D 300 L10 las veces que haga falta y comprobar que 78 56 siguen en 1357:0300 y 1357:0301 y que los 14 bytes restantes siguen en 00, con la certeza de que la propia verificación no alteró el estado del programa.

## Parte 1, Paso 14: Modo de direccionamiento inmediato vs. directo a memoria

El estudiante determina que el direccionamiento directo a memoria, MOV AX,[0300] (A1 00 03), es el que requiere un ciclo adicional de acceso al bus. Ambas instrucciones ocupan 3 bytes, así que su costo de búsqueda (fetch) es el mismo. La diferencia está en el operando fuente. En MOV AX,0005 (B8 05 00), el valor 0x0005 viaja dentro del flujo de instrucción y llega al procesador junto con el opcode, sin ningún acceso posterior. En MOV AX,[0300], los bytes 00 03 solo contienen una dirección, no el dato. El procesador debe resolver DS:0300 y realizar una lectura adicional en memoria para obtener el valor, lo que añade un acceso al bus que el modo inmediato no necesita. El estudiante preferiría el direccionamiento directo cuando el objetivo no es la velocidad sino leer un valor que puede cambiar en tiempo de ejecución, como los bytes 78 56 escritos con E en el Paso 11. Con el modo inmediato, el valor queda fijado en el código al ensamblar, y cambiarlo exigiría reensamblar la instrucción o modificar sus bytes. Con el modo directo, la instrucción permanece igual y siempre lee el contenido actual de DS:0300, que aquí resulta en AX = 0x5678. Con U, sin ejecutar nada, el estudiante confirma qué codificación produjo cada comando A. MOV AX,0005 aparece como B80500, con el operando sin corchetes, mientras que MOV AX,[0300] aparece como A10003, con corchetes que indican acceso a memoria. Como U solo desensambla, la comprobación no altera registros ni memoria.

# Parte 2 
## Checkpoint 1: Tabla de traza del programa de suma

| # | Instrucción | AX | BX | CX | IP siguiente | ZF | CF | SF |
|---|-------------|------|------|------|--------------|---------|---------|---------|
| 1 | MOV AX,000A | 000A | 0000 | 0000 | 0103 | 0 (NZ) | 0 (NC) | 0 (PL) |
| 2 | MOV BX,0005 | 000A | 0005 | 0000 | 0106 | 0 (NZ) | 0 (NC) | 0 (PL) |
| 3 | MOV CX,0003 | 000A | 0005 | 0003 | 0109 | 0 (NZ) | 0 (NC) | 0 (PL) |
| 4 | ADD AX,BX   | 000F | 0005 | 0003 | 010B | 0 (NZ) | 0 (NC) | 0 (PL) |
| 5 | ADD AX,CX   | 0012 | 0005 | 0003 | 010D | 0 (NZ) | 0 (NC) | 0 (PL) |

## Checkpoint 2: Tabla de traza del programa con bucle LOOP

| Iteración | Instrucción | AX después | CX después | IP siguiente | ¿LOOP salta? |
|-----------|-------------|------------|------------|--------------|--------------|
| Inicio | MOV AX,0000 | 0000 | 0003 | 0103 | N/A |
| Inicio | MOV CX,0004 | 0000 | 0004 | 0106 | N/A |
| 1 | ADD AX,+02 | 0002 | 0004 | 0109 | N/A |
| 1 | LOOPW 0106 | 0002 | 0003 | 0106 | Sí |
| 2 | ADD AX,+02 | 0004 | 0003 | 0109 | N/A |
| 2 | LOOPW 0106 | 0004 | 0002 | 0106 | Sí |
| 3 | ADD AX,+02 | 0006 | 0002 | 0109 | N/A |
| 3 | LOOPW 0106 | 0006 | 0001 | 0106 | Sí |
| 4 | ADD AX,+02 | 0008 | 0001 | 0109 | N/A |
| 4 | LOOPW 0106 | 0008 | 0000 | 010B | No |
| Fin | INT 20 | 0008 | 0000 | - | N/A (Program terminated normally) |


