

# Lab10_3. Administración de permisos de archivos y directorios en Ubuntu

## 1. Comprender los permisos de Linux

Linux utiliza tres categorías de usuarios para establecer permisos:

| Categoría    | Significado             |
| ------------ | ----------------------- |
| `u` (User)   | Propietario del archivo |
| `g` (Group)  | Grupo propietario       |
| `o` (Others) | Los demás usuarios      |

Existen tres permisos fundamentales:

| Permiso   | Símbolo | Archivo             | Directorio                        |
| --------- | ------- | ------------------- | --------------------------------- |
| Lectura   | `r`     | Leer contenido      | Listar archivos                   |
| Escritura | `w`     | Modificar contenido | Crear o eliminar entradas         |
| Ejecución | `x`     | Ejecutar archivo    | Acceder o atravesar el directorio |

## 2. Crear archivos y directorios para la práctica

Crear un directorio:

```
mkdir laboratorio
```

Ingresar al directorio:

```
cd laboratorio
```

Crear un archivo:

```
touch documento.txt
```

Escribir contenido:

```
echo "Practica de permisos en Linux" > documento.txt
```

## 3. Consultar los permisos actuales

Ejecutar:

```
ls -l
```

Ejemplo de resultado:

```
-rw-rw-r-- 1 usuario usuario 30 Nov 10  documento.txt
```

Interpretación:

Descomposición de los permisos

# -rw-rw-r--

\-

Tipo

rw-

Propietario

rw-

Grupo

r--

Otros

- `-`: archivo regular.
- `rw-`: propietario puede leer y escribir.
- `rw-`: integrantes del grupo pueden leer y escribir.
- `r--`: otros usuarios solamente pueden leer.

## 4. Modificar permisos con chmod

El comando `chmod` permite cambiar los permisos de archivos y directorios.

Ejemplo 1. Dar todos los permisos al propietario:

```
chmod u+rwx documento.txt
```

Ejemplo 2. Quitar escritura al grupo:

```
chmod g-w documento.txt
```

Ejemplo 3. Quitar todos los permisos a otros usuarios:

```
chmod o-rwx documento.txt
```

Ejemplo 4. Establecer permisos para todos:

```
chmod a+r documento.txt
```

Verificar los cambios:

```
ls -l
```

## 5. Permisos mediante valores numéricos

Linux también permite representar los permisos mediante números:

| Permiso         | Valor |
| --------------- | ----- |
| Lectura (`r`)   | 4     |
| Escritura (`w`) | 2     |
| Ejecución (`x`) | 1     |

Se suman los valores para obtener cada conjunto de permisos.

| Valor | Permisos |
| ----- | -------- |
| 7     | `rwx`    |
| 6     | `rw-`    |
| 5     | `r-x`    |
| 4     | `r--`    |
| 0     | `---`    |

Ejemplo:

```
chmod 754 documento.txt
```

Interpretación:

- `7`: propietario puede leer, escribir y ejecutar.
- `5`: grupo puede leer y ejecutar.
- `4`: otros usuarios solamente pueden leer.

Representación de chmod 754

Propietario

7 · rwx

Grupo

5 · r-x

Otros

4 · r--

## 6. Cambiar el propietario y grupo con chown

El comando `chown` permite modificar el propietario y el grupo de un archivo.

Cambiar el propietario:

```
sudo chown #APELLIDO documento.txt
```

Cambiar propietario y grupo:

```
sudo chown #APELLIDO:desarrollo documento.txt
```

Verificar:

```
ls -l documento.txt
```

Recuerda sustituir `#APELLIDO` por el nombre de usuario que creaste en la práctica anterior.

## 7. Crear un directorio compartido

Crear un directorio:

```
sudo mkdir /opt/desarrollo
```

Asignar el grupo `desarrollo`:

```
sudo chown root:desarrollo /opt/desarrollo
```

Permitir al propietario y al grupo acceder y modificar el directorio:

```
sudo chmod 770 /opt/desarrollo
```

Verificar:

```
ls -ld /opt/desarrollo
```

Resultado esperado:

```
drwxrwx--- root desarrollo /opt/desarrollo
```

Esto significa que el propietario `root` y los integrantes del grupo `desarrollo` tienen acceso completo, mientras que los demás usuarios no tienen permisos.


## Statement

El estudiante identificará, interpretará y modificará los permisos de acceso a archivos y directorios en Ubuntu Linux, utilizando los comandos `chmod` y `chown`. Asimismo, configurará permisos para propietarios, grupos y otros usuarios, y creará un directorio compartido para analizar cómo el sistema operativo controla el acceso a los recursos mediante mecanismos de autorización.

Finalmente, presentará un informe individual que incluya los comandos ejecutados, capturas de pantalla de los cambios de permisos y una explicación de los resultados obtenidos.

Concepto fundamental: en Linux, los permisos determinan qué operaciones puede realizar cada usuario sobre los archivos y directorios. Esto permite aplicar el principio de mínimo privilegio y proteger los recursos del sistema operativo.
