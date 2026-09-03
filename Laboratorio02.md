# Laboratorio: CPU Scheduling con procesos en C

Este laboratorio crea dos procesos hijos mediante `fork()` y los coloca en condición de competir por la CPU. La sincronización mediante una `pipe` permite liberar a ambos procesos para que el scheduler del sistema operativo determine cuál ejecuta primero.

## Código

```c
%%writefile scheduling.c

#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

void trabajar(char *nombre) {

    printf("%s | PID: %d | Listo para ejecutar\n",
           nombre, getpid());


    volatile long resultado = 0;

    for (long i = 0; i < 1000000000; i++) {
        resultado += i;
    }

    printf("%s | PID: %d | Finalizó\n",
           nombre, getpid());
}

int main() {

    int tuberia[2];
    pipe(tuberia);

    printf("Padre | PID: %d\n", getpid());

    // Crear Hijo 1
    if (fork() == 0) {

        close(tuberia[1]);

        char señal;
        read(tuberia[0], &señal, 1);

        trabajar("Hijo 1");

        return 0;
    }

    // Crear Hijo 2
    if (fork() == 0) {

        close(tuberia[1]);

        char señal;
        read(tuberia[0], &señal, 1);

        trabajar("Hijo 2");

        return 0;
    }

    // Padre
    close(tuberia[0]);

    printf("Padre | Ambos hijos creados\n");
    printf("Padre | Liberando a los hijos...\n");

    write(tuberia[1], "A", 1);
    write(tuberia[1], "B", 1);

    close(tuberia[1]);

    wait(NULL);
    wait(NULL);

    printf("Padre | Ambos hijos terminaron\n");

    return 0;
}
```

## Compilación

```bash
gcc scheduling.c -o scheduling
```

## Ejecución

```bash
./scheduling
```

## Conceptos observados

- `fork()` → creación de procesos hijos.
- `pipe()` → mecanismo de comunicación y sincronización.
- `read()` → los hijos esperan antes de comenzar.
- `write()` → el padre libera a los hijos.
- `wait()` → el padre espera la finalización de los hijos.
- `getpid()` → identificación de cada proceso.
- `volatile` + cálculo intensivo → trabajo **CPU-bound**.
- Scheduler de Linux → decide qué proceso obtiene la CPU.

> **Importante:** el programa no implementa FCFS, SJF, SRTF ni Round Robin. Los procesos son reales y la planificación la realiza el scheduler del sistema operativo, que, por el Kernel de colab actual 6.6.122+ podemos inferir que se usa un algoritmo EEVDEF (Earliest Eligible Virtual Deadline First) 
