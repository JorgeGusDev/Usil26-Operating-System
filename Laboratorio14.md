# Laboratorio14: Security Auditing and Vulnerability Identification in Linux

**Curso:** Sistemas Operativos  
**Tema:** Seguridad y protección de sistemas operativos  
**Entorno:** Oracle VirtualBox, Kali Linux y Metasploitable 2  
**Duración:** 120 minutos

## 1. Objetivo

Realizar una auditoría de seguridad del sistema operativo Metasploitable 2, utilizando Kali Linux como estación de auditoría, para identificar vulnerabilidades y configuraciones inseguras relacionadas con servicios de red, kernel, usuarios, privilegios, permisos de archivos, procesos y mecanismos de autenticación.

## 2. Descarga de las máquinas virtuales

Para desarrollar el laboratorio, descargar e instalar los siguientes recursos:

| Recurso | Descripción | Enlace de descarga |
|---|---|---|
| **Oracle VirtualBox** | Software de virtualización para ejecutar las máquinas virtuales. | [Descargar VirtualBox](https://www.virtualbox.org/wiki/Downloads) |
| **Kali Linux** | Sistema operativo utilizado como estación de auditoría. Seleccionar la imagen preconfigurada para VirtualBox. | [Descargar Kali Linux](https://www.kali.org/get-kali/#kali-virtual-machines) |
| **Metasploitable 2** | Máquina virtual Linux intencionalmente vulnerable para realizar auditorías de seguridad. | [Descargar Metasploitable 2](https://sourceforge.net/projects/metasploitable/files/Metasploitable2/) |

### Instrucciones de instalación

1. Instalar Oracle VirtualBox en el equipo.
2. Descargar Kali Linux para VirtualBox, descomprimir el archivo e incorporar la máquina virtual a VirtualBox.
3. Descargar Metasploitable 2 y descomprimir el archivo.
4. Crear una nueva máquina virtual Linux de 32 bits para Metasploitable 2, utilizando el disco virtual `.vmdk` incluido.
5. Configurar ambas máquinas con el mismo adaptador de red Host-only.
6. Iniciar ambas máquinas virtuales.

**Credenciales predeterminadas:**

| Máquina virtual | Usuario | Contraseña |
|---|---|---|
| Kali Linux (imagen oficial preconfigurada) | `kali` | `kali` |
| Metasploitable 2 | `msfadmin` | `msfadmin` |

> **Importante:** Metasploitable 2 contiene vulnerabilidades deliberadas. Debe ejecutarse exclusivamente en una red de laboratorio aislada, sin acceso directo a Internet ni a redes externas.

## 3. Configuración del entorno virtual

Se utilizarán dos máquinas virtuales en Oracle VirtualBox, conectadas mediante una red **Host-only** para mantener aislado el sistema vulnerable.

| Máquina virtual | Función | Adaptador 1 | Adaptador 2 |
|---|---|---|---|
| Kali Linux | Estación de auditoría | Host-only | Deshabilitado |
| Metasploitable 2 | Sistema objetivo | Host-only | Deshabilitado |

**Instrucciones:**

1. Apagar ambas máquinas virtuales.
2. Ingresar a VirtualBox → Configuración → Red.
3. Configurar el Adaptador 1 de ambas máquinas como **Host-only Adapter**.
4. Seleccionar exactamente el mismo adaptador Host-only en ambas máquinas.
5. Deshabilitar los demás adaptadores.
6. Iniciar Kali Linux y Metasploitable 2.

> **Importante:** Metasploitable 2 debe permanecer aislada de Internet y de redes externas. Todas las pruebas se realizarán exclusivamente dentro del laboratorio.

## 4. Actividades prácticas

### Actividad 1. Identificación de direcciones IP y conectividad

**En Metasploitable 2:**

```bash
ifconfig
```

**En Kali Linux:**

```bash
ip -br a
```

Comprobar la conectividad hacia Metasploitable 2:

```bash
ping -c 4 192.168.56.102
```

Registrar las direcciones IP de ambas máquinas y verificar la comunicación.

> La dirección `192.168.56.102` es la identificada en el entorno de referencia. Sustituirla si la máquina obtiene una dirección diferente.

**Evidencia:** Direcciones IP y conectividad entre las máquinas virtuales.

### Actividad 2. Auditoría externa de servicios mediante Nmap

**En Kali Linux:**

```bash
nmap -sV 192.168.56.102
```

Identificar los puertos abiertos, los servicios activos y las versiones detectadas.

Registrar los principales servicios:

| Puerto | Servicio | Debilidad que debe investigarse |
|---|---|---|
| 21 | FTP (vsftpd 2.3.4) | Versión asociada a una distribución comprometida con puerta trasera |
| 22 | SSH | Algoritmos criptográficos heredados |
| 23 | Telnet | Comunicación sin cifrado |
| 139/445 | Samba | Posibles vulnerabilidades y exposición de recursos |
| 512–514 | r-services | Protocolos antiguos de acceso remoto |
| 1524 | bindshell | Posible shell remoto privilegiado expuesto |

**Evidencia:** Captura del escaneo y tabla de servicios identificados.

### Actividad 3. Verificación de servicios remotos inseguros

**En Kali Linux:**

```bash
nc -vz 192.168.56.102 23
```

```bash
nc -vz 192.168.56.102 1524
```

Comprobar la accesibilidad de los servicios Telnet y bindshell, sin establecer una sesión interactiva.

**Evidencia:** Resultados de las conexiones TCP y descripción de los riesgos asociados a los servicios expuestos.

### Actividad 4. Acceso remoto mediante SSH

**En Kali Linux:**

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa msfadmin@192.168.56.102
```

**Credenciales del laboratorio:**

- Usuario: `msfadmin`
- Contraseña: `msfadmin`

Si el servidor requiere un algoritmo antiguo de intercambio de claves, utilizar:

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa \
    -o KexAlgorithms=+diffie-hellman-group1-sha1 \
    msfadmin@192.168.56.102
```

Utilizar estas excepciones criptográficas únicamente para la máquina vulnerable del laboratorio.

Una vez conectado, verificar que el terminal muestre:

```text
msfadmin@metasploitable:~$
```

A partir de este punto, los comandos se ejecutarán dentro del sistema operativo Metasploitable 2 mediante la sesión SSH.

**Evidencia:** Captura de la conexión SSH establecida.

### Actividad 5. Auditoría del kernel y sistema operativo

**En Metasploitable 2 mediante SSH:**

```bash
uname -a
```

```bash
cat /etc/issue
```

```bash
cat /proc/version
```

Identificar:

- Distribución y versión del sistema operativo.
- Versión del kernel Linux.
- Arquitectura del sistema.
- Antigüedad de los componentes.

Registrar las versiones y documentar los riesgos asociados a sistemas operativos obsoletos.

**Evidencia:** Información del kernel y distribución Linux.

### Actividad 6. Auditoría de usuarios y privilegios

**En Metasploitable 2:**

```bash
whoami
```

```bash
id msfadmin
```

```bash
groups msfadmin
```

```bash
sudo -l
```

Identificar los grupos administrativos y los privilegios efectivos del usuario.

En el entorno de referencia se identificaron grupos como:

| Grupo | Función | Riesgo potencial |
|---|---|---|
| admin | Administración del sistema | Privilegios elevados |
| adm | Acceso a determinados registros | Exposición de información |
| lpadmin | Administración de impresoras | Privilegios innecesarios |
| sambashare | Gestión de recursos compartidos | Acceso a recursos compartidos |

**Evidencia:** Grupos del usuario, permisos sudo y posibles desviaciones del principio de mínimo privilegio.

### Actividad 7. Auditoría de ejecutables SUID

**En Metasploitable 2:**

```bash
find /usr/bin /bin -type f -perm -4000 2>/dev/null
```

Identificar los archivos ejecutables con el bit SUID activado.

En el entorno de referencia se encontraron 22 ejecutables SUID, incluidos:

```text
/usr/bin/nmap
/usr/bin/netkit-rsh
/usr/bin/netkit-rlogin
/usr/bin/sudo
/bin/su
```

Verificar los propietarios y permisos:

```bash
ls -l /usr/bin/nmap
```

```bash
ls -l /usr/bin/netkit-rsh
```

```bash
ls -l /usr/bin/netkit-rlogin
```

El bit SUID permite ejecutar un programa con el UID efectivo de su propietario. Cuando el propietario es root, esto puede representar un riesgo si el programa permite realizar operaciones no previstas.

**Evidencia:** Lista de ejecutables SUID, propietarios, permisos y riesgos potenciales de escalamiento de privilegios.

### Actividad 8. Auditoría de permisos de archivos críticos

**En Metasploitable 2:**

```bash
ls -l /etc/passwd
```

```bash
ls -l /etc/shadow
```

```bash
ls -ld /tmp
```

Verificar los permisos y propietarios.

| Recurso | Permisos observados | Interpretación |
|---|---|---|
| `/etc/passwd` | 644 | Lectura general; escritura exclusiva del propietario |
| `/etc/shadow` | 640 | Lectura restringida al propietario y grupo |
| `/tmp` | 1777 | Escritura compartida con sticky bit |

Identificar las diferencias entre permisos de lectura, escritura y ejecución, así como la función del sticky bit.

**Evidencia:** Permisos observados y evaluación de su adecuación para archivos críticos.

### Actividad 9. Auditoría de procesos y servicios del sistema

**En Metasploitable 2:**

```bash
ps aux
```

```bash
netstat -tulnp
```

Identificar procesos activos, usuarios propietarios y puertos asociados. Algunos detalles pueden requerir privilegios administrativos.

Revisar los servicios configurados para iniciarse automáticamente:

```bash
ls /etc/init.d/
```

```bash
ls /etc/rc2.d/
```

Relacionar los procesos identificados con los servicios detectados previamente mediante Nmap.

**Evidencia:** Procesos activos, servicios de red y mecanismos de inicio automático.

### Actividad 10. Auditoría de autenticación y registros

**En Metasploitable 2:**

```bash
sudo chage -l msfadmin
```

Consultar los parámetros generales de contraseñas:

```bash
grep -E 'PASS_MAX_DAYS|PASS_MIN_DAYS|PASS_WARN_AGE' /etc/login.defs
```

Revisar los últimos eventos de autenticación:

```bash
sudo tail -n 20 /var/log/auth.log
```

Consultar los accesos recientes:

```bash
last -n 10
```

Identificar políticas de caducidad de contraseñas, eventos de autenticación y mecanismos de trazabilidad.

**Evidencia:** Configuración de autenticación y registros del sistema operativo.

## 5. Reto individual — Vulnerability Assessment and OS Hardening

El estudiante deberá identificar **cinco vulnerabilidades, configuraciones inseguras o riesgos potenciales** del sistema operativo Metasploitable 2, sustentados en las evidencias obtenidas durante el laboratorio.

Elaborar una matriz con la siguiente estructura:

| Componente del SO | Hallazgo identificado | Riesgo de seguridad | Medida de hardening |
|---|---|---|---|
| Kernel | | | |
| Usuarios y privilegios | | | |
| Ejecutables SUID | | | |
| Servicios y procesos | | | |
| Autenticación y permisos | | | |

Cada hallazgo deberá incluir su evidencia técnica y una recomendación de seguridad.

Distinguir entre vulnerabilidades confirmadas, configuraciones inseguras y riesgos potenciales. No es necesario explotar vulnerabilidades ni modificar la configuración de Metasploitable 2.

## 6. Entregables

Presentar un informe en formato PDF que contenga evidencias de todas las actividades y matrices, en cuento corresponda.

