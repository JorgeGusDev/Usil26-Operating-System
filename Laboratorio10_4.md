# Lab10_4: Automatización de cuentas de usuario y políticas de contraseñas en Ubuntu Linux

Objetivo: administrar el ciclo de vida de las cuentas de usuario mediante comandos de terminal, automatizando su creación y configurando políticas de seguridad relacionadas con la caducidad de contraseñas, los cambios obligatorios y el vencimiento de cuentas.


## 1. Creación automática de usuarios

Crear tres cuentas locales mediante un bucle Bash:

```
for usuario in empleado1 empleado2 empleado3; do sudo useradd -m -s /bin/bash "$usuario"; done
```

Verificar las cuentas:

```
getent passwd empleado1 empleado2 empleado3
```

## 2. Asignación de contraseñas iniciales

Asignar una contraseña temporal a cada usuario:

```
for usuario in empleado1 empleado2 empleado3; do sudo passwd "$usuario"; done
```

El sistema solicitará ingresar y confirmar una contraseña para cada cuenta.

Nota: para este laboratorio, las contraseñas se introducen de manera interactiva, evitando dejarlas escritas directamente en los comandos o en el historial de la terminal.

## 3. Obligar a cambiar la contraseña en el primer inicio de sesión

Ejecutar:

```
for usuario in empleado1 empleado2 empleado3; do sudo passwd -e "$usuario"; done
```

La opción `-e` marca la contraseña como caducada, por lo que el usuario deberá cambiarla al autenticarse mediante un mecanismo compatible.

Comprobar el estado:

```
sudo chage -l empleado1
```

## 4. Configurar una política de caducidad de contraseñas

Establecer las siguientes condiciones para los tres usuarios:

| Parámetro                           | Política |
| ----------------------------------- | -------- |
| Tiempo mínimo entre cambios       | 2 días  |
| Duración máxima de contraseña         | 30 días   |
| Aviso antes del vencimiento         | 7 días   |
| Inactividad después del vencimiento | 5 días   |

Aplicar automáticamente:

```
for usuario in empleado1 empleado2 empleado3; do sudo chage -m 2 -M 30 -W 7 -I 5 "$usuario"; done
```

Explicación de los parámetros:

- `-m 2`: establece un mínimo de dos días entre cambios de contraseña.
- `-M 30`: obliga a renovar la contraseña cada 30 días.
- `-W 7`: advierte al usuario siete días antes de su vencimiento.
- `-I 5`: deshabilita el acceso mediante contraseña después de cinco días de inactividad posteriores al vencimiento, según el mecanismo de autenticación.

Consultar la política:

```
sudo chage -l empleado1
```

Ciclo de vida de una contraseña

## 5. Configurar la fecha de vencimiento de una cuenta

Además de la contraseña, Linux permite establecer una fecha límite para la cuenta.

Por ejemplo, configurar que `empleado3` venza el 31 de diciembre de 2026:

```
sudo chage -E 2026-12-31 empleado3
```

Verificar:

```
sudo chage -l empleado3
```

Diferencia importante:

- Caducidad de contraseña: obliga al usuario a cambiar su contraseña.
- Caducidad de cuenta: establece una fecha a partir de la cual la cuenta deja de estar disponible para iniciar sesión.

## 6. Comprobar el cambio obligatorio de contraseña

Para probar la política del usuario `empleado1`, primero restablecer el mínimo entre cambios a cero:

```
sudo chage -m 0 empleado1
```

Marcar nuevamente su contraseña como caducada:

```
sudo passwd -e empleado1
```

Intentar iniciar sesión:

```
su - empleado1
```

El sistema deberá solicitar la contraseña temporal y, si la autenticación PAM está configurada normalmente, pedir una nueva contraseña antes de permitir el acceso.

Una vez dentro, comprobar:

```
whoami
```

Salir:

```
exit
```

Después de la prueba, restaurar el mínimo de dos días:

```
sudo chage -m 2 empleado1
```

## 7. Bloqueo y desbloqueo de cuentas

Bloquear la contraseña de `empleado2`:

```
sudo passwd -l empleado2
```

Consultar su estado:

```
sudo passwd -S empleado2
```

Desbloquear la contraseña:

```
sudo passwd -u empleado2
```

El bloqueo con `passwd -l` impide la autenticación mediante esa contraseña, pero no necesariamente todas las formas de acceso a la cuenta, como determinadas autenticaciones por claves SSH.

## 8. Verificación automatizada de las políticas

Mostrar el estado de las contraseñas de los tres usuarios:

```
for usuario in empleado1 empleado2 empleado3; do echo "===== $usuario ====="; sudo passwd -S "$usuario"; sudo chage -l "$usuario"; done
```

Esto permite al administrador revisar las políticas configuradas sin consultar cada cuenta manualmente.

## 9. Eliminación de las cuentas de prueba

Una vez finalizado el laboratorio:

```
for usuario in empleado1 empleado2 empleado3; do sudo userdel -r "$usuario"; done
```


## Individual Challenge: Automated Password and Account Lifecycle Management

El estudiante deberá implementar de manera individual un escenario de administración automatizada de cuentas en Ubuntu Linux, creando **cinco usuarios locales** mediante comandos Bash y asignándoles contraseñas iniciales. Asimismo, deberá configurar una política que obligue a cambiar la contraseña en el primer inicio de sesión, establecer una vigencia máxima de **45 días**, un período mínimo de **3 días entre cambios** y una advertencia de vencimiento con **10 días de anticipación**. Adicionalmente, deberá configurar una fecha de expiración para dos cuentas, bloquear y desbloquear una cuenta de prueba y verificar todas las políticas mediante los comandos `passwd` y `chage`. Finalmente, presentará un **informe individual** que incluya los comandos ejecutados, capturas de pantalla, resultados de las verificaciones y una breve explicación sobre la importancia de la automatización del ciclo de vida de las cuentas para la seguridad del sistema operativo.

