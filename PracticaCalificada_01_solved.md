### PREGUNTA 1

### a) Procesos

4 procesos: Padre → Hijo 1 → Hijo 2 → Nieto.

### b) Orden de mensajes

`Inicio → Hijo 1 → Hijo 2 → Nieto → Hijo 2 termina → Hijo 1 termina → Padre termina`

### c) Nieto antes de Hijo 1

No. El Nieto se crea después de que Hijo 1 imprime su mensaje.

### d) `wait()`

- Padre → espera a Hijo 1.
- Hijo 1 → espera a Hijo 2.
- Hijo 2 → espera al Nieto.

### Cadena de finalización

`Nieto → Hijo 2 → Hijo 1 → Padre`

### PREGUNTA 2


### a) Código fuente — 2 puntos

Código `procesos.c`:

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

void proceso_trabajo(int id, int fd)
{
    printf("Proceso %d | PID: %d | Procesando...\n", id, getpid());

    // Actividad de procesamiento
    volatile long resultado = 0;
    for (long i = 0; i < 500000000; i++)
        resultado += i;

    printf("Proceso %d | PID: %d | Esperando E/S...\n", id, getpid());

    // Operación de E/S bloqueante
    char dato;
    read(fd, &dato, 1);

    printf("Proceso %d | PID: %d | E/S completada\n", id, getpid());

    close(fd);
}

int main()
{
    int tuberia[2];

    pipe(tuberia);

    printf("Padre | PID: %d\n", getpid());

    for (int i = 1; i <= 3; i++)
    {
        pid_t pid = fork();

        if (pid == 0)
        {
            close(tuberia[1]);

            proceso_trabajo(i, tuberia[0]);

            return 0;
        }
    }

    // El padre no lee de la tubería
    close(tuberia[0]);

    printf("Padre | Tres procesos creados\n");
    printf("Padre | Esperando 10 segundos...\n");

    sleep(10);

    // Envía un dato para desbloquear a cada proceso
    write(tuberia[1], "A", 1);
    write(tuberia[1], "B", 1);
    write(tuberia[1], "C", 1);

    close(tuberia[1]);

    // Esperar a los tres procesos
    wait(NULL);
    wait(NULL);
    wait(NULL);

    printf("Padre | Todos los procesos terminaron\n");

    return 0;
}
```

### b) Evidencias de ejecución — 2 puntos

Compilación:

```bash
gcc procesos_io.c -o procesos
```

Ejecución:

```bash
./procesos
```

En otra terminal:

```bash
ps -e -o pid,ppid,stat,comm | grep procesos
```


### PREGUNTA 3 — Sincronización de procesos en VM

### a) Código fuente — 2 puntos

Código `race_condition.c`:

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/mman.h>
#include <pthread.h>
#include <sched.h>

#define ITERACIONES 100000

typedef struct {
    int contador;
    pthread_mutex_t mutex;
} Datos;

int main()
{
    Datos *datos = mmap(NULL, sizeof(Datos),
                       PROT_READ | PROT_WRITE,
                       MAP_SHARED | MAP_ANONYMOUS, -1, 0);

    datos->contador = 0;

    pthread_mutexattr_t attr;
    pthread_mutexattr_init(&attr);
    pthread_mutexattr_setpshared(&attr, PTHREAD_PROCESS_SHARED);
    pthread_mutex_init(&datos->mutex, &attr);

    pid_t p1 = fork();

    if (p1 == 0)
    {
        for (int i = 0; i < ITERACIONES; i++)
        {
            int temp = datos->contador;
            sched_yield();
            datos->contador = temp + 1;
        }
        return 0;
    }

    pid_t p2 = fork();

    if (p2 == 0)
    {
        for (int i = 0; i < ITERACIONES; i++)
        {
            int temp = datos->contador;
            sched_yield();
            datos->contador = temp + 1;
        }
        return 0;
    }

    wait(NULL);
    wait(NULL);

    printf("SIN MUTEX\n");
    printf("Valor esperado: %d\n", ITERACIONES * 2);
    printf("Valor obtenido: %d\n", datos->contador);

    datos->contador = 0;

    pid_t p3 = fork();

    if (p3 == 0)
    {
        for (int i = 0; i < ITERACIONES; i++)
        {
            pthread_mutex_lock(&datos->mutex);
            datos->contador++;
            pthread_mutex_unlock(&datos->mutex);
        }
        return 0;
    }

    pid_t p4 = fork();

    if (p4 == 0)
    {
        for (int i = 0; i < ITERACIONES; i++)
        {
            pthread_mutex_lock(&datos->mutex);
            datos->contador++;
            pthread_mutex_unlock(&datos->mutex);
        }
        return 0;
    }

    wait(NULL);
    wait(NULL);

    printf("\nCON MUTEX\n");
    printf("Valor esperado: %d\n", ITERACIONES * 2);
    printf("Valor obtenido: %d\n", datos->contador);

    pthread_mutex_destroy(&datos->mutex);
    pthread_mutexattr_destroy(&attr);
    munmap(datos, sizeof(Datos));

    return 0;
}
```

### Ejecución en Ubuntu

Compilar:

```bash
gcc race_condition.c -o race_condition -pthread
```

Ejecutar:

```bash
./race_condition
```

### b) Evidencia de ejecución — 2 puntos

Resultado esperado:

```text
SIN MUTEX
Valor esperado: 200000
Valor obtenido: <valor menor a 200000>

CON MUTEX
Valor esperado: 200000
Valor obtenido: 200000
```

El valor sin mutex puede variar en cada ejecución debido a la condición de carrera.


```

