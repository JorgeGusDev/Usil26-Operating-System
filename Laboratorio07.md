
## LABORATORIO 06 — Page Fault en Linux

### Programa `page_fault.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/resource.h>

#define MB (1024 * 1024)
#define MEMORIA (100 * MB)

void mostrar_faults(const char *momento)
{
    struct rusage uso;

    getrusage(RUSAGE_SELF, &uso);

    printf("\n--- %s ---\n", momento);
    printf("Minor page faults: %ld\n", uso.ru_minflt);
    printf("Major page faults: %ld\n", uso.ru_majflt);
}

int main()
{
    long pagina = sysconf(_SC_PAGESIZE);

    printf("PID: %d\n", getpid());
    printf("Tamaño de página: %ld bytes\n", pagina);
    printf("Memoria reservada: 100 MB\n");

    char *memoria = malloc(MEMORIA);

    if (memoria == NULL) {
        perror("malloc");
        return 1;
    }

    mostrar_faults("ANTES DE ACCEDER A LA MEMORIA");

    printf("\nPresiona ENTER para acceder a las páginas...");
    getchar();

    for (long i = 0; i < MEMORIA; i += pagina) {
        memoria[i] = 'A';
    }

    mostrar_faults("DESPUÉS DE ACCEDER A LAS PÁGINAS");

    printf("\nPresiona ENTER para finalizar...");
    getchar();

    free(memoria);

    return 0;
}
```

### Compilar

```bash
gcc -O0 page_fault.c -o page_fault
```

### Ejecutar

```bash
./page_fault
```

Deberías obtener algo parecido a:

```text
PID: 4702
Tamaño de página: 4096 bytes
Memoria reservada: 100 MB

--- ANTES DE ACCEDER A LA MEMORIA ---
Minor page faults: 450
Major page faults: 0

Presiona ENTER para acceder a las páginas...
```

Presionas ENTER y aparecerá algo similar a:

```text
--- DESPUÉS DE ACCEDER A LAS PÁGINAS ---
Minor page faults: 26000
Major page faults: 0
```

