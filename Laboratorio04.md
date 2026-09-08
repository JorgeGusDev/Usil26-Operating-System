# Laboratorio4: Condición de Carrera y Mutex en C

Cree el archivo `race_condition.c`:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/mman.h>
#include <sys/wait.h>
#include <sched.h>

int main() {

    int *contador = mmap(NULL, sizeof(int),
                         PROT_READ | PROT_WRITE,
                         MAP_SHARED | MAP_ANONYMOUS, -1, 0);

    *contador = 0;

    pid_t p1 = fork();

    if (p1 == 0) {

        for (int i = 0; i < 100000; i++) {

            int temp = *contador;
            sched_yield();
            temp = temp + 1;
            *contador = temp;
        }

        exit(0);
    }

    pid_t p2 = fork();

    if (p2 == 0) {

        for (int i = 0; i < 100000; i++) {

            int temp = *contador;
            sched_yield();
            temp = temp + 1;
            *contador = temp;
        }

        exit(0);
    }

    wait(NULL);
    wait(NULL);

    printf("Valor esperado: 200000\n");
    printf("Valor obtenido: %d\n", *contador);

    return 0;
}
```

## Compilación y ejecución

```bash
gcc race_condition.c -o race_condition
```

Ejecute:

```bash
./race_condition
```

El resultado esperado matemáticamente es:

```text
Valor esperado: 200000
```

Sin embargo, el resultado obtenido puede ser menor y puede variar entre ejecuciones:

```text
Valor esperado: 200000
Valor obtenido: 103452
```


## solución mediante Mutex

Modifique el programa anterior para utilizar un **mutex** que proteja la sección crítica y elimine la condición de carrera.

La solución deberá garantizar que solo un proceso pueda modificar `contador` a la vez y que, al finalizar la ejecución, el valor obtenido sea:

```text
Valor esperado: 200000
Valor obtenido: 200000
```

El estudiante deberá implementar un **mutex compartido entre procesos**, aplicar `lock` antes de ingresar a la sección crítica y `unlock` al finalizarla, y demostrar mediante la ejecución del programa que la condición de carrera ha sido solucionada.

