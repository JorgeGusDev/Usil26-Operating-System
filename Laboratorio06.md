# Laboratorio06 — Gestión de memoria y paginación

## Objetivo

Analizar mediante programas en C ejecutados en Ubuntu los conceptos de **memoria principal, direcciones de memoria, tamaño de página, paginación, asignación de memoria y fragmentación**.

---

## Ejercicio 1 — Tamaño de página y direcciones de memoria

El estudiante deberá desarrollar un programa en C que obtenga el tamaño de página del sistema y muestre las direcciones de diferentes variables almacenadas en memoria.

```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    int entero = 10;
    double decimal = 3.14;
    char texto[] = "Sistema Operativo";

    long pagina = sysconf(_SC_PAGESIZE);

    printf("Tamaño de página: %ld bytes\n\n", pagina);

    printf("Dirección de entero : %p\n", (void *)&entero);
    printf("Dirección de decimal: %p\n", (void *)&decimal);
    printf("Dirección de texto  : %p\n", (void *)texto);

    return 0;
}
```

Compilar:

```bash
gcc memoria.c -o memoria
```

Ejecutar:

```bash
./memoria
```

### ¿Qué debe analizar el estudiante?

Debe identificar que:

* Las variables ocupan posiciones de memoria.
* El sistema trabaja con **direcciones**.
* La memoria se administra utilizando unidades como las **páginas**.
* El tamaño de página depende de la arquitectura y configuración del sistema.

Puede comprobar el tamaño de página directamente en Ubuntu:

```bash
getconf PAGESIZE
```

Normalmente obtendrá:

```text
4096
```

---

# Ejercicio 2 — Paginación: página y desplazamiento

Aquí hacemos que el estudiante **calcule la página y el desplazamiento de una dirección lógica**, siguiendo exactamente el modelo del Capítulo 9.

Si:

```text
Tamaño de página = 4096 bytes
Dirección lógica = 10000
```

Entonces:

```text
Página = 10000 / 4096 = 2
Desplazamiento = 10000 % 4096 = 1808
```

El programa puede hacerlo:

```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    long pagina = sysconf(_SC_PAGESIZE);
    unsigned long direccion;

    printf("Tamaño de página: %ld bytes\n", pagina);

    printf("Ingrese una dirección lógica: ");
    scanf("%lu", &direccion);

    unsigned long numero_pagina = direccion / pagina;
    unsigned long desplazamiento = direccion % pagina;

    printf("\nDirección lógica : %lu\n", direccion);
    printf("Número de página : %lu\n", numero_pagina);
    printf("Desplazamiento   : %lu\n", desplazamiento);

    return 0;
}
```

### Concepto que demuestra

```text
Dirección lógica
       ↓
 ┌───────────────┐
 │ página │ offset│
 └───────────────┘
       ↓
   Page Table
       ↓
     Frame
       ↓
Dirección física
```

Esto corresponde directamente al modelo de **paging del Capítulo 9**, donde la dirección lógica se divide en **page number + page offset**. 

---

# Ejercicio 3 — Simulación de una tabla de páginas

Este sería el ejercicio más importante del laboratorio.

El estudiante tendrá una tabla de páginas simplificada:

```text
Página    Frame
  0         5
  1         2
  2         7
  3         1
```

El programa recibirá una dirección lógica y realizará la traducción:

```text
Dirección lógica
       ↓
Número de página
       ↓
Tabla de páginas
       ↓
Número de frame
       ↓
Dirección física
```

Código:

```c
#include <stdio.h>

#define PAGE_SIZE 4096
#define NUM_PAGES 4

int page_table[NUM_PAGES] = {
    5, 2, 7, 1
};

int main()
{
    unsigned long direccion;

    printf("Ingrese una dirección lógica: ");
    scanf("%lu", &direccion);

    int pagina = direccion / PAGE_SIZE;
    int desplazamiento = direccion % PAGE_SIZE;

    if (pagina >= NUM_PAGES) {
        printf("Página fuera del espacio de direcciones.\n");
        return 1;
    }

    int frame = page_table[pagina];

    unsigned long direccion_fisica =
        frame * PAGE_SIZE + desplazamiento;

    printf("\nPágina        : %d\n", pagina);
    printf("Desplazamiento: %d\n", desplazamiento);
    printf("Frame         : %d\n", frame);
    printf("Dirección física: %lu\n", direccion_fisica);

    return 0;
}
```

Por ejemplo, para:

```text
Dirección lógica = 5000
```

tenemos:

```text
5000 / 4096 = página 1
5000 % 4096 = desplazamiento 904
```

La tabla indica:

```text
Página 1 → Frame 2
```

Entonces:

```text
Dirección física =
2 × 4096 + 904

= 9096
```

Esto permite que el alumno **vea realmente cómo la tabla de páginas traduce una dirección lógica a una dirección física**, que es uno de los contenidos centrales del capítulo. 

---

# Ejercicio 4 — Fragmentación en asignación contigua

Finalmente, podemos hacer una pequeña simulación de **asignación contigua de memoria**.

Por ejemplo:

```text
Memoria de 100 MB

┌──────┬──────────┬──────┬──────────┬──────┐
│ P1   │   Libre  │ P2   │   Libre  │ P3   │
└──────┴──────────┴──────┴──────────┴──────┘
```

El estudiante deberá implementar **First Fit** para asignar procesos a espacios libres.

Por ejemplo:

```text
Memoria disponible:

100 MB

P1 = 20 MB
P2 = 30 MB
P3 = 10 MB
```
