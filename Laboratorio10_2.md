# Lab10_2. Administración de usuarios y grupos en Ubuntu Linux

Guía para crear, comprobar, utilizar y eliminar cuentas locales mediante la terminal de Ubuntu.


## 1. Identificar el usuario actual

Consultar el usuario de la sesión:

```bash
whoami
```

Consultar los grupos del usuario actual:

```bash
groups
```

Consultar UID, GID y grupos:

```bash
id
```

## 2. Crear una cuenta local

Solicitar la creación de la cuenta y establecer su contraseña:

```bash
sudo adduser #nombre
```

Ubuntu solicitará una contraseña y algunos datos adicionales. Los datos adicionales pueden dejarse en blanco si no son necesarios.

## 3. Verificar la creación

Consultar la identidad y los grupos de la cuenta:

```bash
id #nombre
```

Consultar la entrada de la cuenta:

```bash
getent passwd #nombre
```

Listar las cuentas registradas en el sistema:

```bash
cat /etc/passwd
```

`/etc/passwd` contiene información de las cuentas del sistema; no todas las entradas corresponden a personas que inician sesión de forma interactiva.

## 4. Crear un grupo

Crear un grupo llamado `desarrollo`:

```bash
sudo groupadd desarrollo
```

Verificar que existe:

```bash
getent group desarrollo
```

## 5. Agregar el usuario al grupo

Agregar la cuenta al grupo secundario `desarrollo`:

```bash
sudo usermod -aG desarrollo #nombre
```

Verificar los grupos de la cuenta:

```bash
groups #nombre
```

También puedes consultar UID, GID y grupos:

```bash
id #nombre
```

Consultar los integrantes del grupo:

```bash
getent group desarrollo
```

**Importante:** `-aG` agrega el grupo secundario sin reemplazar los grupos secundarios existentes. La pertenencia se aplica a las nuevas sesiones; si el usuario ya tenía una sesión abierta, debe cerrarla y volver a iniciar sesión para que se actualice su lista de grupos en esa sesión.

## 6. Cambiar de usuario

Cambiar a la cuenta creada:

```bash
su - #nombre
```

Introduce la contraseña de esa cuenta.

Comprobar la identidad de la sesión:

```bash
whoami
```

Consultar sus grupos:

```bash
id
```

Para regresar a la sesión anterior:

```bash
exit
```

## 7. Eliminar la cuenta

Eliminar la cuenta, conservando su directorio personal:

```bash
sudo deluser #nombre
```

Si también se desea eliminar el directorio personal y los archivos de esa cuenta:

```bash
sudo deluser --remove-home #nombre
```

Verificar que la cuenta ya no existe:

```bash
id #nombre
```

El sistema debe indicar que no existe ese usuario.

**Importante:** utiliza `--remove-home` solo si estás seguro de que no necesitas conservar los archivos del usuario.

## Statement

El estudiante creará, configurará, verificará y eliminará una cuenta de usuario local en Ubuntu Linux, utilizando su propio apellido como identificador. Asimismo, creará un grupo, incorporará su cuenta y comprobará la pertenencia a los grupos mediante comandos de terminal, comprendiendo su relación con la autenticación, autorización y seguridad del sistema operativo.

Finalmente, presentará un informe individual que incluya los comandos ejecutados, capturas de pantalla de los procedimientos realizados y una breve explicación de los resultados obtenidos.


