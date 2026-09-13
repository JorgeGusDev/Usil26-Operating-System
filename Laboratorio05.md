# Laboratorio 04 - Administración de Ubuntu

## Administración básica de Ubuntu

---

## 1. Identificar la versión del sistema

Abrir una **Terminal** en Ubuntu.

Ejecutar:

```bash
uname -r
```

Este comando permite identificar la versión del **kernel de Linux** utilizada por el sistema.

También ejecutar:

```bash
lsb_release -a
```

Este comando permite consultar información de la distribución de Ubuntu.



---

## 2. Identificar procesos

Un proceso es un programa que se encuentra en ejecución.

Ejecutar:

```bash
ps aux
```

Observar los procesos actualmente ejecutándose en el sistema.


También ejecutar:

```bash
top
```

Observar en tiempo real los procesos que se encuentran ejecutándose.

Para salir de `top`, presionar:

```text
q
```


## 3. Crear y ejecutar un proceso en C

En esta actividad se creará un programa sencillo en lenguaje C que generará un proceso hijo.

### Paso 1. Crear el archivo

En la terminal ejecutar:

```bash
nano proceso.c
```

Ingresar el siguiente código:

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {

    printf("Proceso padre iniciado. PID: %d\n", getpid());

    pid_t pid = fork();

    if (pid < 0) {
        printf("Error al crear el proceso\n");
        return 1;
    }

    if (pid == 0) {
        printf("Proceso hijo ejecutándose. PID: %d\n", getpid());

        for (int i = 1; i <= 10; i++) {
            printf("Hijo: trabajando... %d/10\n", i);
            sleep(1);
        }

        printf("Proceso hijo finalizado. PID: %d\n", getpid());

    } else {
        printf("Proceso padre. PID del hijo: %d\n", pid);

        wait(NULL);

        printf("Proceso padre: el hijo terminó.\n");
    }

    return 0;
}
```

Guardar el archivo:

```text
CTRL + O
ENTER
CTRL + X
```

### Paso 2. Compilar el programa

Ejecutar:

```bash
gcc proceso.c -o proceso
```

Si no se presentan errores, se habrá generado el ejecutable `proceso`.

### Paso 3. Ejecutar el programa

Ejecutar:

```bash
./proceso
```

El programa crea un **proceso hijo** mediante:

```c
fork();
```

El proceso padre obtiene el PID del hijo y espera su finalización mediante:

```c
wait(NULL);
```

Mientras el programa está ejecutándose, abrir **otra Terminal** y ejecutar:

```bash
ps aux
```

Buscar el proceso `proceso`.

También se puede utilizar:

```bash
ps -ef | grep proceso
```
