# Laboratorio 06 — Gestión de memoria y paginación en Linux

## Objetivo

Analizar mediante programas en C y herramientas del sistema operativo **Ubuntu** los conceptos de **memoria principal, direcciones de memoria, tamaño de página, espacio de direcciones, asignación de memoria, paginación y memoria residente**, observando el comportamiento real de los procesos.

## Requisitos

El laboratorio deberá realizarse en un equipo con **Ubuntu Linux**.

Instalar las herramientas necesarias:

```bash
sudo apt update
sudo apt install build-essential procps
```

Verificar la instalación de GCC:

```bash
gcc --version
```

Verificar el sistema operativo:

```bash
lsb_release -a
```

Verificar el tamaño de página:

```bash
getconf PAGESIZE
```

---

# Ejercicio 1 — Tamaño de página y direcciones de memoria

El estudiante deberá desarrollar un programa en C que obtenga el tamaño de página del sistema y muestre las direcciones de diferentes variables almacenadas en memoria.

```c
#include <stdio.h>
#include <unistd.h>

int global = 100;

int main()
{
    int entero = 10;
    double decimal = 3.14;
    char texto[] = "Sistema Operativo";

    long pagina = sysconf(_SC_PAGESIZE);

    printf("PID: %d\n", getpid());
    printf("Tamaño de página: %ld bytes\n\n", pagina);

    printf("Dirección de global : %p\n", (void *)&global);
    printf("Dirección de entero : %p\n", (void *)&entero);
    printf("Dirección de decimal: %p\n", (void *)&decimal);
    printf("Dirección de texto  : %p\n", (void *)texto);

    printf("\nPresione ENTER para finalizar...\n");
    getchar();

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

Comprobar el tamaño de página directamente desde Ubuntu:

```bash
getconf PAGESIZE
```

---

# Ejercicio 2 — Exploración del espacio de direcciones del proceso

Utilizando el **PID obtenido en el Ejercicio 1**, el estudiante deberá explorar el espacio de direcciones virtuales del proceso mediante las herramientas de Ubuntu.

Consultar:

```bash
cat /proc/<PID>/maps
```

y:

```bash
pmap <PID>
```

Identificar las regiones correspondientes a:

```text
[heap]
[stack]
código del programa
bibliotecas compartidas
```

Localizar las direcciones mostradas por el programa C dentro de las regiones correspondientes.

Consultar también:

```bash
cat /proc/<PID>/status
```

Registrar:

```text
VmSize
VmRSS
VmData
VmStk
VmExe
VmLib
```

---


