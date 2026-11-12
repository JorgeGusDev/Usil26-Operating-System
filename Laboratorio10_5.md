# Lab10_5: Automatización de usuarios, grupos y permisos de acceso en Ubuntu Linux

Objetivo: automatizar la creación de usuarios, la asignación de grupos y la configuración de permisos de acceso a archivos y directorios mediante comandos Bash, comprendiendo cómo Linux administra la autorización y protección de sus recursos.

## 1. Creación automática de usuarios

Utilizaremos un bucle `for` para crear tres usuarios locales y sus respectivos directorios personales.

```
for usuario in alumno1 alumno2 alumno3; do sudo useradd -m "$usuario"; done
```

Verificar los usuarios creados:

```
getent passwd alumno1 alumno2 alumno3
```

Consultar sus identificadores:

```
id alumno1
```

## 2. Creación de un grupo y asignación automática de usuarios

Crear el grupo `investigacion`:

```
sudo groupadd investigacion
```

Incorporar automáticamente los tres usuarios:

```
for usuario in alumno1 alumno2 alumno3; do sudo usermod -aG investigacion "$usuario"; done
```

Verificar los integrantes del grupo:

```
getent group investigacion
```

Resultado de ejemplo:

```
investigacion:x:1004:alumno1,alumno2,alumno3
```

El identificador numérico del grupo puede variar.

## 3. Generación automática de documentos

Crear un directorio compartido:

```
sudo mkdir -p /opt/investigacion
```

Generar cinco documentos automáticamente:

```
for i in {1..5}; do echo "Documento $i" | sudo tee "/opt/investigacion/documento$i.txt" > /dev/null; done
```

Verificar los archivos creados:

```
ls -l /opt/investigacion
```

## 4. Asignación automática de propietarios y permisos

Asignar el directorio y sus archivos al grupo `investigacion`:

```
sudo chown -R root:investigacion /opt/investigacion
```

Configurar los permisos del directorio:

```
sudo chmod 2770 /opt/investigacion
```

El valor `2770` representa:

| Valor | Significado                                                                                    |
| ----- | ---------------------------------------------------------------------------------------------- |
| `2`   | Activa SGID: los nuevos archivos y subdirectorios heredan el grupo propietario del directorio. |
| `7`   | Propietario: lectura, escritura y ejecución.                                                   |
| `7`   | Grupo: lectura, escritura y ejecución.                                                         |
| `0`   | Otros usuarios: sin permisos.                                                                  |

Configurar automáticamente los permisos de los documentos:

```
sudo chmod 660 /opt/investigacion/*.txt
```

Con `660`, el propietario y el grupo pueden leer y modificar los documentos, mientras que otros usuarios no tienen acceso.

## 5. Verificación de permisos

Consultar los permisos del directorio:

```
ls -ld /opt/investigacion
```

Consultar los permisos de los archivos:

```
ls -l /opt/investigacion
```

Resultado esperado:

```
drwxrws--- root investigacion /opt/investigacion
-rw-rw---- root investigacion documento1.txt
-rw-rw---- root investigacion documento2.txt
-rw-rw---- root investigacion documento3.txt
-rw-rw---- root investigacion documento4.txt
-rw-rw---- root investigacion documento5.txt
```

La letra `s` en `drwxrws---` indica que el directorio tiene activado el bit SGID.

## 6. Comprobación del control de acceso

Desde el usuario administrador `JorgeGus`, intentar acceder al documento:

```
cat /opt/investigacion/documento1.txt
```

Si `JorgeGus` no pertenece al grupo `investigacion` y no tiene otros privilegios aplicables, el resultado será:

```
Permission denied
```

Ahora realizar la misma operación utilizando la identidad de `alumno1`:

```
sudo -u alumno1 cat /opt/investigacion/documento1.txt
```

Resultado esperado:

```
Documento 1
```

Comprobar que el usuario puede modificar el documento:

```
sudo -u alumno1 sh -c 'echo "Modificado por alumno1" >> /opt/investigacion/documento1.txt'
```

Verificar el contenido:

```
cat /opt/investigacion/documento1.txt
```

De esta manera se comprueba que los permisos establecidos permiten el acceso a los integrantes del grupo autorizado.

## 7. Automatización de la eliminación de recursos

Una vez concluida la práctica, eliminar los usuarios creados:

```
for usuario in alumno1 alumno2 alumno3; do sudo userdel -r "$usuario"; done
```

Eliminar el directorio compartido:

```
sudo rm -r /opt/investigacion
```

Eliminar el grupo:

```
sudo groupdel investigacion
```


## Statement

El estudiante deberá implementar de manera individual un escenario de automatización del control de acceso en Ubuntu Linux, creando **cinco usuarios locales** identificados con nombres de su elección y distribuyéndolos automáticamente en **dos grupos de trabajo** mediante comandos Bash. Asimismo, deberá generar al menos **diez documentos**, organizarlos en dos directorios compartidos y configurar sus propietarios y permisos para garantizar que cada grupo pueda acceder únicamente a los documentos que le corresponden. Finalmente, deberá comprobar mediante comandos de terminal que los usuarios autorizados pueden leer y modificar sus archivos, mientras que los usuarios de otros grupos tienen el acceso restringido, y presentar un **informe individual** que incluya los comandos ejecutados, capturas de pantalla, resultados de las pruebas y una breve explicación de cómo la automatización contribuye a la administración segura de usuarios y recursos del sistema operativo.
