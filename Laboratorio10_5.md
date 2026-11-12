
# Lab10_5: Automatización de usuarios, grupos y permisos de acceso en Ubuntu Linux

## 1. Creación automática de usuarios

Crear tres usuarios locales:

```
for usuario in alumno1 alumno2 alumno3; do sudo useradd -m "$usuario"; done
```

Verificar los usuarios:

```
getent passwd alumno1 alumno2 alumno3
```

Consultar sus identificadores:

```
id alumno1
```

## 2. Creación del grupo y asignación de usuarios

Crear el grupo `investigacion`:

```
sudo groupadd investigacion
```

Incorporar automáticamente los tres usuarios:

```
for usuario in alumno1 alumno2 alumno3; do sudo usermod -aG investigacion "$usuario"; done
```

Verificar:

```
getent group investigacion
```

## 3. Generación automática de documentos

Crear el directorio compartido dentro de `JorgeGus`:

```
mkdir -p ~/investigacion
```

Generar cinco documentos automáticamente:

```
for i in {1..5}; do echo "Documento $i" > "$HOME/investigacion/documento$i.txt"; done
```

Verificar:

```
ls -l ~/investigacion
```

Estructura resultante:

```
/home/JorgeGus/
└── investigacion/
    ├── documento1.txt
    ├── documento2.txt
    ├── documento3.txt
    ├── documento4.txt
    └── documento5.txt
```

## 4. Asignación de propietarios y permisos

Asignar la carpeta y sus documentos al usuario `JorgeGus` y al grupo `investigacion`:

```
sudo chown -R JorgeGus:investigacion ~/investigacion
```

Configurar permisos del directorio:

```
chmod 2770 ~/investigacion
```

Configurar permisos de los documentos:

```
chmod 660 ~/investigacion/*.txt
```

| Valor | Significado                                                |
| ----- | ---------------------------------------------------------- |
| `2`   | SGID: los nuevos archivos heredan el grupo del directorio. |
| `7`   | Propietario: lectura, escritura y ejecución.               |
| `7`   | Grupo: lectura, escritura y ejecución.                     |
| `0`   | Otros usuarios: sin acceso.                                |

### Permitir que los alumnos lleguen a la carpeta compartida

Como la carpeta está dentro de `/home/JorgeGus`, debemos autorizar el recorrido por ese directorio personal sin permitir listar sus archivos.

Instalar la herramienta ACL si es necesario:

```
sudo apt install acl -y
```

Conceder únicamente permiso de recorrido al grupo:

```
sudo setfacl -m g:investigacion:--x /home/JorgeGus
```

Este comando no modifica `/root` ni `/`. Solo añade una autorización específica sobre `/home/JorgeGus`.

## 5. Verificación de permisos

Consultar los permisos del directorio:

```
ls -ld ~/investigacion
```

Consultar los permisos de los documentos:

```
ls -l ~/investigacion
```

Resultado esperado:

```
drwxrws--- JorgeGus investigacion investigacion
-rw-rw---- JorgeGus investigacion documento1.txt
-rw-rw---- JorgeGus investigacion documento2.txt
-rw-rw---- JorgeGus investigacion documento3.txt
-rw-rw---- JorgeGus investigacion documento4.txt
-rw-rw---- JorgeGus investigacion documento5.txt
```

## 6. Comprobación del acceso compartido

### Prueba 1. JorgeGus consulta un documento

```
cat ~/investigacion/documento1.txt
```

Resultado:

```
Documento 1
```

### Prueba 2. alumno1 modifica el documento

```
sudo -u alumno1 sh -c 'echo "Investigación realizada por alumno1" >> /home/JorgeGus/investigacion/documento1.txt'
```

### Prueba 3. alumno2 consulta el mismo documento

```
sudo -u alumno2 cat /home/JorgeGus/investigacion/documento1.txt
```

Resultado esperado:

```
Documento 1
Investigación realizada por alumno1
```

### Prueba 4. alumno3 modifica el documento

```
sudo -u alumno3 sh -c 'echo "Revisión realizada por alumno3" >> /home/JorgeGus/investigacion/documento1.txt'
```

### Prueba 5. JorgeGus verifica los cambios

```
cat ~/investigacion/documento1.txt
```

Resultado esperado:

```
Documento 1
Investigación realizada por alumno1
Revisión realizada por alumno3
```

De esta manera, los tres alumnos colaboran sobre un mismo archivo mediante los permisos del grupo `investigacion`.

### Prueba 6. Verificar el acceso denegado a otros usuarios

Crear un usuario que no pertenezca al grupo:

```
sudo useradd -m visitante1
```

Intentar leer el documento:

```
sudo -u visitante1 cat /home/JorgeGus/investigacion/documento1.txt
```

Resultado esperado:

```
Permission denied
```

Esto demuestra que los usuarios ajenos al grupo no pueden acceder al recurso.

## 7. Eliminación de recursos del laboratorio

Eliminar los usuarios de prueba:

```
for usuario in alumno1 alumno2 alumno3 visitante1; do sudo userdel -r "$usuario"; done
```

Eliminar únicamente la carpeta creada para el laboratorio:

```
rm -r ~/investigacion
```

Retirar la autorización ACL añadida a la carpeta personal:

```
sudo setfacl -x g:investigacion /home/JorgeGus
```

Eliminar el grupo:

```
sudo groupdel investigacion
```

No se elimina ni modifica el resto del contenido personal de `JorgeGus`.

## Statement – Individual Challenge

El estudiante deberá implementar de manera individual un escenario de automatización del control de acceso en Ubuntu Linux, creando cinco usuarios locales y distribuyéndolos automáticamente en dos grupos de trabajo mediante comandos Bash. Asimismo, deberá generar al menos diez documentos y organizarlos en dos directorios compartidos ubicados dentro de su carpeta personal (`/home/usuario/`), configurando los propietarios, grupos y permisos correspondientes para garantizar que cada grupo pueda acceder únicamente a los documentos autorizados. Finalmente, deberá demostrar la colaboración entre usuarios mediante operaciones de lectura y escritura, comprobar que los usuarios externos tienen el acceso restringido y presentar un informe individual con los comandos ejecutados, capturas de pantalla, resultados de las pruebas y una breve explicación de cómo Linux implementa el control de acceso discrecional (DAC).

Presentación: Un informe con capturas, captions y parrafos explicativos
