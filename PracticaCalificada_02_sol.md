# **Práctica Calificada 2 — Solucionario**

## **Pregunta 1. Interbloqueos en un sistema de procesamiento concurrente (5 puntos)**

**1. ¿Qué mutex ha adquirido cada hilo y cuál está intentando obtener cuando la ejecución deja de avanzar? (1 punto)**

`actualizar_stock` adquirió `inventario` y espera `reporte`. `generar_reporte` adquirió `reporte` y espera `inventario`.

**2. ¿Por qué ninguno consigue completar la segunda adquisición, aunque los recursos no estén siendo utilizados por otros procesos? (1 punto)**

Porque existe una espera circular: cada hilo mantiene bloqueado un recurso que el otro necesita, produciendo un interbloqueo (*deadlock*).

**3. ¿Reducir sleep(2) a sleep(1) garantiza que ambas operaciones finalicen? (1 punto)**

No. Reducir el tiempo no elimina el interbloqueo. Si ambos hilos adquieren sus primeros mutex antes de solicitar los segundos, volverán a bloquearse.

**4. ¿Cómo modificaría el orden de bloqueo y desbloqueo para evitar el problema? (2 puntos)**

Ambos hilos deben adquirir los mutex en el mismo orden: primero `inventario` y después `reporte`, liberándolos en orden inverso.

```
pthread_mutex_lock(&inventario);
pthread_mutex_lock(&reporte);

/* Operaciones */

pthread_mutex_unlock(&reporte);
pthread_mutex_unlock(&inventario);
```

Esto elimina la espera circular y mantiene la exclusión mutua.

## **Pregunta 2. Hardware y software de E/S: procesamiento de interrupciones en xv6 (5 puntos)**

**1. ¿En qué punto se interrumpe el procesamiento necesario para que los caracteres lleguen al shell? (1 punto)**

En `consoleintr(c)`, porque al estar comentada los caracteres recibidos no se entregan al subsistema de consola.

**2. ¿Por qué el sistema inicia correctamente, pero no interpreta los comandos introducidos? (1 punto)**

Porque UART continúa recibiendo caracteres, pero estos no llegan al búfer de entrada de la consola y, por tanto, el shell no puede leerlos.

**3. ¿Modificar únicamente uartstart() resolvería el problema? (1 punto)**

No. `uartstart()` gestiona la transmisión de salida, mientras que `consoleintr(c)` procesa los caracteres de entrada.

**4. ¿Qué instrucción debe restaurarse y cómo llega un carácter desde UART hasta el shell? (2 puntos)**

Restaurar:

```
consoleintr(c);
```

Flujo: `uartgetc()` recibe el carácter → `consoleintr(c)` lo procesa y almacena en el búfer de consola → `consoleread()` permite su lectura → el shell interpreta y ejecuta el comando.

## **Pregunta 3. Sistemas de archivos: asignación incorrecta de bloques en xv6 (5 puntos)**

**1. ¿Por qué inventario muestra información escrita posteriormente en despacho, aunque sean archivos diferentes? (1 punto)**

Porque ambos archivos pueden referenciar el mismo bloque físico debido a una asignación incorrecta. La escritura de `despacho` sobrescribe los datos de `inventario`.

**2. ¿Qué información deja de actualizarse al comentar la instrucción de balloc()? (1 punto)**

No se actualiza el *bitmap* de bloques ocupados. El bloque permanece marcado como libre y puede asignarse nuevamente a otro archivo.

**3. ¿Cambiar cat inventario por otro comando solucionaría el problema? (1 punto)**

No. El problema está en la asignación de bloques del sistema de archivos, no en `cat`. Otro comando leería los mismos datos sobrescritos.

**4. ¿Qué modificación realizaría en balloc() y por qué log_write(bp) no es suficiente? (2 puntos)**

Restaurar:

```
bp->data[bi / 8] |= m;
```

Esta instrucción marca el bloque como ocupado. `log_write(bp)` registra la modificación, pero no cambia el bitmap por sí mismo. Sin corregirlo, pueden producirse asignaciones duplicadas, corrupción y pérdida de datos.

## **Pregunta 4. Administración de usuarios, políticas de contraseñas y control de acceso en Linux (5 puntos)**

**1. Situación A. ¿Por qué dev01 recibe Permission denied si ya pertenece al grupo desarrollo? ¿Cómo solucionarlo? (1 punto)**

Porque la sesión actual conserva los grupos anteriores. Debe cerrar sesión e ingresar nuevamente para actualizar su pertenencia al grupo `desarrollo`.

También puede ejecutar:

```
newgrp desarrollo
```

**2. Situación B. ¿Por qué auditor01 no puede leer plan.txt y qué tendría que cambiar para permitir únicamente la lectura? (1 punto)**

Porque el directorio tiene permisos `2770` y el archivo `660`, impidiendo el acceso a usuarios externos al grupo `desarrollo`.

Una alternativa es autorizar el acceso mediante un grupo de lectura específico, configurando permisos `750` en el directorio y `640` en el archivo, con el grupo autorizado como propietario de grupo. Esto requiere reorganizar los grupos para conservar también el acceso de Desarrollo. No debe concederse acceso de escritura a Auditoría.

**3. Situación C. ¿Qué instrucción provoca el cambio inmediato de contraseña y qué ocurriría al eliminarla? (1 punto)**

La instrucción es:

```
sudo passwd -e dev01
```

Fuerza la caducidad inmediata de la contraseña, independientemente del plazo de 30 días establecido con `chage`. Al eliminarla, no se forzaría el cambio inmediato.

**4. Situación D. ¿Por qué avance.txt hereda el grupo desarrollo, pero no los permisos de escritura? ¿Cómo solucionarlo? (2 puntos)**

El bit SGID (`2770`) permite heredar el grupo `desarrollo`, pero no garantiza que los archivos nuevos tengan permiso de escritura grupal.

Para corregir los permisos del archivo existente:

```
sudo chmod 660 /srv/proyectos/operaciones/avance.txt
```

Para los nuevos archivos, se debe establecer el permiso de escritura del grupo al crearlos, por ejemplo:

```
touch /srv/proyectos/operaciones/nuevo.txt
chmod 660 /srv/proyectos/operaciones/nuevo.txt
```

Así, los integrantes de `desarrollo` podrán leer y modificar los archivos, mientras que los usuarios externos no tendrán acceso. **El permiso debe aplicarse a cada archivo nuevo**, ya que SGID por sí solo no garantiza la herencia de permisos de escritura.
