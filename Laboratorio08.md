
### LABORATORIO08: Detección y corrección de un Deadlock en C

**Entorno:** Ubuntu  
**Lenguaje:** C  
**Librería:** POSIX Threads (`pthread`)

### Objetivo

Analizar un programa que presenta un **interbloqueo (deadlock)** debido al uso incorrecto de dos mutex y modificarlo para evitar el deadlock.

### Código inicial

```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

pthread_mutex_t mutex1;
pthread_mutex_t mutex2;

void *hilo1(void *arg) {
    pthread_mutex_lock(&mutex1);
    printf("Hilo 1: tiene mutex1\n");

    sleep(1);

    pthread_mutex_lock(&mutex2);
    printf("Hilo 1: tiene mutex2\n");

    pthread_mutex_unlock(&mutex2);
    pthread_mutex_unlock(&mutex1);

    return NULL;
}

void *hilo2(void *arg) {
    pthread_mutex_lock(&mutex2);
    printf("Hilo 2: tiene mutex2\n");

    sleep(1);

    pthread_mutex_lock(&mutex1);
    printf("Hilo 2: tiene mutex1\n");

    pthread_mutex_unlock(&mutex1);
    pthread_mutex_unlock(&mutex2);

    return NULL;
}

int main() {
    pthread_t t1, t2;

    pthread_mutex_init(&mutex1, NULL);
    pthread_mutex_init(&mutex2, NULL);

    pthread_create(&t1, NULL, hilo1, NULL);
    pthread_create(&t2, NULL, hilo2, NULL);

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    pthread_mutex_destroy(&mutex1);
    pthread_mutex_destroy(&mutex2);

    return 0;
}
```

### Ejecución

En Ubuntu:

```bash
gcc deadlock.c -o deadlock -pthread
./deadlock
```

El programa puede quedar ejecutándose indefinidamente porque ambos hilos quedan esperando el mutex que posee el otro.

### Actividad del estudiante

El estudiante debe:

**1. Identificar el deadlock**

Determinar qué mutex posee cada hilo y cuál está esperando.

**2. Identificar las condiciones**

Explicar cuáles de las cuatro condiciones del deadlock están presentes:

```text
Mutual Exclusion
Hold and Wait
No Preemption
Circular Wait
```

**3. Corregir el programa**

Modificar el código para garantizar que ambos hilos adquieran los mutex **siempre en el mismo orden**.

Por ejemplo:

```text
T₁ → mutex1 → mutex2

T₂ → mutex1 → mutex2
```

**4. Comprobar la solución**

Ejecutar nuevamente el programa y verificar que ambos hilos puedan finalizar.

### Pregunta de análisis

**¿Por qué establecer un orden global para adquirir los mutex permite eliminar la posibilidad de deadlock en este programa?**

Esto queda muy bien como laboratorio porque el estudiante **primero observa el interbloqueo, identifica la causa y finalmente corrige el código**, en lugar de limitarse a estudiar la definición.

También podemos hacer una **versión más completa del laboratorio usando `lockdep` o herramientas de Linux para detectar el problema**, pero para una primera práctica yo empezaría con este C + `pthread`.
