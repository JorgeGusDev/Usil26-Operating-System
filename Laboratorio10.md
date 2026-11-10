

# Administración de cuentas en Windows

Guía para crear, comprobar, utilizar y eliminar cuentas locales mediante PowerShell. Ejecuta los comandos administrativos en **PowerShell como administrador**.

## 1. Comprobar las cuentas existentes

```powershell
Get-LocalUser | Format-Table Name, Enabled, Description
```

## 2. Crear una cuenta local

Ejemplo: crear `UserLocal01`.

**Solicitar la contraseña:**

```powershell
$Password = Read-Host "Ingrese una contraseña" -AsSecureString
```

**Crear el usuario:**

```powershell
New-LocalUser -Name "UserLocal01" -Password $Password -FullName "Usuario Local" -Description "Cuenta local para prácticas"
```

La contraseña se solicita de forma segura y no queda escrita directamente en el comando de creación.

**Verificar la creación:**

```powershell
Get-LocalUser -Name "UserLocal01" | Format-List Name, Enabled, SID
```

Debe aparecer la cuenta y `Enabled : True`.

## 3. Consultar grupos locales

Listar los grupos existentes:

```powershell
Get-LocalGroup | Select-Object Name
```

En Windows en español, el grupo estándar suele llamarse `Usuarios`; en inglés, `Users`.

Consultar sus integrantes:

```powershell
Get-LocalGroupMember -Group "Usuarios"
```

Agregar el usuario al grupo es opcional:

```powershell
Add-LocalGroupMember -Group "Usuarios" -Member "UserLocal01"
```

Comprobar:

```powershell
Get-LocalGroupMember -Group "Usuarios"
```

**Importante:** no es necesario agregar manualmente una cuenta a `Usuarios` para que pueda iniciar sesión. Los grupos administran permisos; no determinan por sí solos si una cuenta aparece en la pantalla de inicio. No agregues usuarios a `Administrators` salvo que necesiten privilegios administrativos.

## 4. Comprobar el inicio de sesión con Win + L

1. Presiona `Win + L` para bloquear el equipo.
2. Comprueba si `UserLocal01` aparece en la lista de cuentas.
3. Si aparece, selecciónalo e introduce su contraseña.
4. Si aparece **Otro usuario**, escribe como usuario:

   ```text
   .\\UserLocal01
   ```

   Luego introduce su contraseña.

5. Una vez dentro, abre PowerShell y ejecuta:

   ```powershell
   whoami
   ```

   El resultado esperado será similar a `NOMBRE-EQUIPO\\UserLocal01`.

### Si la cuenta no aparece

Que la cuenta exista y esté habilitada no garantiza que aparezca como icono en la pantalla de inicio.

**A. Confirmar que está habilitada**

```powershell
Get-LocalUser -Name "UserLocal01" | Format-List Name, Enabled
```

Si está deshabilitada:

```powershell
Enable-LocalUser -Name "UserLocal01"
```

**B. Comprobar si existe una entrada que oculte específicamente la cuenta**

```powershell
$path = "HKLM:\\SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Winlogon\\SpecialAccounts\\UserList"
```

```powershell
Get-ItemProperty -Path $path -Name "UserLocal01" -ErrorAction SilentlyContinue
```

Interpretación:
- `UserLocal01 : 0`: esa entrada oculta la cuenta en la pantalla de inicio.
- `UserLocal01 : 1`: esa entrada permite mostrarla.
- Sin resultados: no hay una entrada específica para esa cuenta en esa ubicación.

Si se confirma que el valor es `0` y se desea mostrar la cuenta:

```powershell
Set-ItemProperty -Path $path -Name "UserLocal01" -Value 1
```

No crees ni modifiques esta entrada si no existe.

**C. Comprobar el cambio rápido de usuario**

```powershell
Get-ItemProperty -Path "HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Policies\\System" -Name "HideFastUserSwitching" -ErrorAction SilentlyContinue
```

Interpretación:
- `HideFastUserSwitching : 0`: el cambio rápido no está oculto por este valor.
- `HideFastUserSwitching : 1`: el cambio rápido está oculto por este valor.
- Sin resultados: el valor no está definido en esa ubicación.

Si el valor existe y es `1`, un administrador puede habilitar el cambio rápido:

```powershell
Set-ItemProperty -Path "HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Policies\\System" -Name "HideFastUserSwitching" -Value 0 -Type DWord
```

Después, cierra sesión o reinicia y vuelve a comprobar.

**Nota:** `HideFastUserSwitching = 0` permite el cambio rápido de usuario, pero no garantiza que una cuenta aparezca como icono. No es necesario modificarlo si ya está en `0`. Si la cuenta sigue sin aparecer, revisa las directivas de inicio de sesión y las restricciones aplicadas al equipo.

### Comandos para cambiar de sesión

Desde una sesión abierta, desconectar la sesión y volver a la pantalla de inicio:

```cmd
tsdiscon
```

Para cerrar completamente la sesión:

```cmd
shutdown /l
```

`Win + L` bloquea la sesión; no la cierra ni cambia automáticamente de usuario.

## 5. Eliminar una cuenta local

Ejemplo: eliminar `UserLocal01`.

```powershell
Remove-LocalUser -Name "UserLocal01"
```

Verificar que ya no exista:

```powershell
Get-LocalUser
```

La cuenta ya no debe aparecer.

**Importante:** `Remove-LocalUser` elimina la cuenta, pero no necesariamente elimina su carpeta de perfil y sus archivos de `C:\\Users\\UserLocal01`. Comprueba que no necesitas esos archivos antes de borrarlos manualmente.

## 6. Resumen de comandos

| Operación | Comando |
|---|---|
| Listar usuarios | `Get-LocalUser` |
| Crear usuario | `New-LocalUser` |
| Consultar usuario | `Get-LocalUser -Name "UserLocal01"` |
| Habilitar usuario | `Enable-LocalUser -Name "UserLocal01"` |
| Listar grupos | `Get-LocalGroup` |
| Consultar integrantes | `Get-LocalGroupMember -Group "Usuarios"` |
| Agregar a un grupo | `Add-LocalGroupMember -Group "Usuarios" -Member "UserLocal01"` |
| Eliminar usuario | `Remove-LocalUser -Name "UserLocal01"` |
| Identificar sesión actual | `whoami` |

## 7. Consideraciones de seguridad

- Ejecuta las operaciones administrativas desde PowerShell como administrador.
- Utiliza contraseñas robustas y no reutilices contraseñas personales en prácticas.
- Mantén las cuentas de uso cotidiano como usuarios estándar.
- Concede privilegios administrativos solo cuando sean necesarios.
- Distingue entre **autenticación** (comprobar quién inicia sesión) y **autorización** (determinar qué puede hacer esa cuenta).
"""
path = "/mnt/data/guia_cuentas_locales_windows10_pro.md"
pypandoc.convert_text(md, "md", format="md", outputfile=path, extra_args=["--standalone"])
print(path)
