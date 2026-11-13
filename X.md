# Preparación y prueba del entorno para la Práctica Calificada de Linux

**Entorno:** Windows + Oracle VirtualBox + Ubuntu  
**Carpeta compartida en Windows:** `C:\UBUNTUSHARED`  
**Punto de montaje esperado en Ubuntu:** `/media/sf_UBUNTUSHARED`  
**Objetivo:** ejecutar comandos en Ubuntu, registrar la sesión y entregar un archivo Markdown sin usar copiar y pegar.

> **Importante:** estas actividades deben realizarse **antes de la práctica calificada**. Durante el examen, respete las indicaciones de Proctorizer y del docente. La configuración de la carpeta compartida no evita otras restricciones de la plataforma.

## 1. Preparar la carpeta en Windows

1. Abra el Explorador de archivos.
2. Cree la carpeta `C:\UBUNTUSHARED` si todavía no existe.
3. Verifique que pueda crear y borrar un archivo de prueba dentro de ella.

## 2. Configurar la carpeta compartida en VirtualBox

Con Ubuntu iniciado, en la ventana de VirtualBox abra **Devices → Shared Folders → Shared Folders Settings** y pulse **Add Share (+)**.

| Campo | Configuración |
|---|---|
| Folder Path | `C:\UBUNTUSHARED` |
| Folder Name | `UBUNTUSHARED` |
| Mount Point | Dejar vacío |
| Read-only | Desactivado |
| Auto-mount | Activado |
| Make Machine-permanent | Activado |

Pulse **OK** para guardar. Si la carpeta no se monta, reinicie Ubuntu después de guardar la configuración.

**Nota:** el nombre `UBUNTUSHARED` debe coincidir exactamente con el configurado en VirtualBox; no basta con que la carpeta de Windows tenga ese nombre.

## 3. Comprobar el acceso desde Ubuntu

Abra la terminal y ejecute:

```bash
whoami
groups
sudo VBoxControl sharedfolder list
ls -ld /media/sf_UBUNTUSHARED
```

La lista de grupos debe incluir `vboxsf`, y VirtualBox debe mostrar un recurso compartido llamado `UBUNTUSHARED`.

Si `vboxsf` no aparece en `groups`, ejecute:

```bash
sudo usermod -aG vboxsf "$USER"
```

Cierre completamente la sesión de Ubuntu y vuelva a entrar. Compruebe otra vez con `groups`.

Si `sudo VBoxControl sharedfolder list` muestra `Shared Folder mappings (0)`, revise la configuración de VirtualBox del paso 2.

## 4. Probar escritura directa en la carpeta compartida

```bash
echo "Prueba de conexión Ubuntu-Windows" > /media/sf_UBUNTUSHARED/prueba_conexion.txt
cat /media/sf_UBUNTUSHARED/prueba_conexion.txt
ls -l /media/sf_UBUNTUSHARED/
```

Abra `C:\UBUNTUSHARED` en Windows y confirme que aparece `prueba_conexion.txt`. Si aparece **Permission denied**, compruebe la pertenencia al grupo `vboxsf` y vuelva a iniciar sesión.

## 5. Preparar una terminal con salida sencilla

Para reducir los códigos de formato en el registro, ejecute antes de iniciar la grabación:

```bash
export TERM=dumb
export PS1='$ '
```

Esto ayuda a simplificar el registro, pero no garantiza que desaparezcan todos los caracteres especiales.

## 6. Iniciar el registro directamente en la carpeta compartida

**No hace falta utilizar `cp` al finalizar.** Ejecute:

```bash
script -q -f /media/sf_UBUNTUSHARED/examen_linux.log
```

Desde este momento, la terminal registra los comandos y sus salidas directamente en la carpeta compartida de Windows.

**No cierre la ventana mientras desarrolla las actividades.**

## 7. Actividades de prueba

Ejecute los siguientes comandos **dentro de la sesión de `script`**.

### Actividad 1. Identificación del sistema

```bash
whoami
hostname
pwd
uname -a
```

### Actividad 2. Navegación y archivos

```bash
mkdir -p ~/prueba_calificada
cd ~/prueba_calificada
pwd
touch archivo1.txt archivo2.txt
ls -l
```

### Actividad 3. Crear contenido

```bash
echo "Primera evidencia" > archivo1.txt
echo "Segunda evidencia" > archivo2.txt
cat archivo1.txt
cat archivo2.txt
```

### Actividad 4. Bucle Bash

```bash
for i in 1 2 3 4 5; do
    echo "Actividad $i completada"
done
```

### Actividad 5. Permisos de archivos

```bash
chmod 640 archivo1.txt
ls -l archivo1.txt
stat -c '%a %n' archivo1.txt
```

### Actividad 6. Verificación de la carpeta compartida

```bash
ls -ld /media/sf_UBUNTUSHARED
ls -l /media/sf_UBUNTUSHARED
```

## 8. Finalizar la grabación

Cuando termine todas las actividades, escriba:

```bash
exit
```

Esto cierra la sesión de `script` y deja el registro en:

```text
/media/sf_UBUNTUSHARED/examen_linux.log
```

Verifique:

```bash
ls -lh /media/sf_UBUNTUSHARED/examen_linux.log
```

## 9. Crear el conversor de registro a Markdown

Cree el programa en la carpeta personal de Ubuntu:

```bash
nano ~/convertir_md.py
```

Introduzca el siguiente código Python. El conversor elimina secuencias ANSI comunes, intenta reconstruir líneas con retornos de carro y genera un Markdown legible. **No puede reconstruir perfectamente todas las ediciones interactivas de una terminal.**

```python
import re
from pathlib import Path
from datetime import datetime

CARPETA = Path('/media/sf_UBUNTUSHARED')
ORIGEN = CARPETA / 'examen_linux.log'
DESTINO = CARPETA / 'examen_linux.md'

if not ORIGEN.is_file():
    raise SystemExit(f'No existe el registro: {ORIGEN}')

texto = ORIGEN.read_bytes().decode('utf-8', errors='replace')

# Eliminar secuencias de control OSC, DCS y CSI habituales.
texto = re.sub(r'\x1b\][^\x07\x1b]*(?:\x07|\x1b\\)', '', texto)
texto = re.sub(r'\x1bP.*?\x1b\\', '', texto, flags=re.DOTALL)
texto = re.sub(r'\x1b\[[0-?]*[ -/]*[@-~]', '', texto)
texto = re.sub(r'\x1b[@-_]', '', texto)

# Tratar los retrocesos y los retornos de carro de manera aproximada.
texto = re.sub(r'[^\n\b]\x08', '', texto)
texto = texto.replace('\r\n', '\n').replace('\r', '\n')
texto = re.sub(r'[\x00-\x08\x0b-\x1f\x7f]', '', texto)
texto = re.sub(r'\n{3,}', '\n\n', texto).strip()

# Utilizar una cerca de Markdown que no colisione con el contenido.
cerca = '````'
while cerca in texto:
    cerca += '`'

contenido = (
    '# Evidencias de la Práctica Calificada de Linux\n\n'
    f'**Fecha de conversión:** {datetime.now():%d/%m/%Y %H:%M}\n\n'
    '**Origen:** `examen_linux.log`\n\n'
    '## Registro de comandos y resultados\n\n'
    f'{cerca}text\n{texto}\n{cerca}\n'
)

DESTINO.write_text(contenido, encoding='utf-8')
print(f'Markdown generado: {DESTINO}')
```

Para guardar en Nano: **Ctrl+O**, **Enter** y **Ctrl+X**.

## 10. Generar el Markdown directamente en Windows

Una vez finalizado `script`, ejecute:

```bash
python3 ~/convertir_md.py
```

Compruebe que ambos archivos existen:

```bash
ls -lh /media/sf_UBUNTUSHARED/examen_linux.log /media/sf_UBUNTUSHARED/examen_linux.md
```

El archivo `.md` se genera **directamente** en la carpeta compartida. No se requiere `cp` ni portapapeles.

## 11. Verificar el Markdown antes de entregar

Desde Ubuntu:

```bash
head -n 25 /media/sf_UBUNTUSHARED/examen_linux.md
```

Desde Windows, abra:

```text
C:\UBUNTUSHARED\examen_linux.md
```

Compruebe que aparecen los comandos ejecutados y sus resultados. Si el archivo presenta texto desordenado, revise el registro original y repita la prueba con comandos sencillos; `script` no es un generador nativo de Markdown.

## 12. Entrega de la práctica calificada

1. Finalice la grabación mediante `exit`.
2. Ejecute el conversor Python.
3. Compruebe que `examen_linux.md` está en `C:\UBUNTUSHARED`.
4. Adjunte el archivo `.md` en la plataforma indicada por el docente.
5. Conserve `examen_linux.log` como respaldo, si el docente lo solicita.

> La transferencia mediante carpeta compartida no utiliza las funciones de copiar y pegar del portapapeles. Sin embargo, la posibilidad de cambiar de aplicación y adjuntar archivos depende de la configuración autorizada del examen supervisado.

## 13. Lista de comprobación previa

- [ ] Existe `C:\UBUNTUSHARED` en Windows.
- [ ] VirtualBox registra la carpeta con el nombre `UBUNTUSHARED`.
- [ ] La carpeta aparece en Ubuntu como `/media/sf_UBUNTUSHARED`.
- [ ] El usuario pertenece al grupo `vboxsf`.
- [ ] Se pudo crear `prueba_conexion.txt` desde Ubuntu y verlo en Windows.
- [ ] `script` guarda `examen_linux.log` directamente en la carpeta compartida.
- [ ] Se finalizó correctamente la grabación con `exit`.
- [ ] `python3 ~/convertir_md.py` generó `examen_linux.md`.
- [ ] El Markdown se abre correctamente desde Windows.
- [ ] Se probó la entrega del archivo en la plataforma autorizada.

## 14. Solución de problemas frecuentes

| Problema | Verificación o solución |
|---|---|
| `No Shared Folders available` | Revisar que VirtualBox tenga registrado `UBUNTUSHARED`. |
| No existe `/media/sf_UBUNTUSHARED` | Activar Auto-mount y reiniciar Ubuntu; comprobar el nombre del recurso. |
| `Permission denied` | Verificar `groups`; agregar al usuario a `vboxsf` y volver a iniciar sesión. |
| `script: cannot open ...` | Comprobar que la carpeta esté montada y permita escritura. |
| No se genera el `.md` | Finalizar primero `script` y verificar que exista el `.log`. |
| El `.md` contiene símbolos extraños | Revisar el conversor; evitar comandos interactivos complejos y cambios de cursor. |
| No aparece el archivo en Windows | Verificar que la carpeta del anfitrión sea `C:\UBUNTUSHARED`. |

---

**Resultado esperado:** `C:\UBUNTUSHARED\examen_linux.md`, con el registro de la práctica, generado desde Ubuntu sin copiar ni pegar y sin transferencias posteriores mediante `cp`.
