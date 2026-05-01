# M06 — Gaining Access
### Parte 2: Escalating Privileges

---

## 1. Concepto de Privilege Escalation

Un ataque de privilege escalation consiste en obtener más privilegios de los que se tenían inicialmente. El atacante entra con una cuenta sin privilegios de administrador y usa fallos de diseño, errores de programación, bugs o malas configuraciones del OS/aplicación para escalar hasta tener control total del sistema.

Existen dos tipos:

**Horizontal Privilege Escalation:** el atacante accede a recursos de otro usuario con el **mismo nivel de privilegios**. Ejemplo: el usuario A accede a la cuenta bancaria del usuario B sin tener privilegios superiores, solo accediendo a lo que no le corresponde.

**Vertical Privilege Escalation:** el atacante escala hacia un nivel **superior** de privilegios. Ejemplo: un usuario normal accede a funciones de administrador del sitio. Este es el tipo más buscado porque lleva al control total del sistema.

---

## 2. DLL Hijacking (Windows)

La mayoría de aplicaciones Windows no especifican la ruta completa al cargar DLLs externas. Windows las busca primero en el directorio desde el que se lanzó la aplicación. Si el atacante coloca una DLL maliciosa con el mismo nombre en ese directorio antes de que arranque la aplicación, ésta carga la DLL maliciosa en lugar de la legítima, ejecutando el código del atacante con los privilegios del proceso.

Por tanto, el requisito es simple: tener acceso de escritura al directorio de la aplicación y conocer qué DLLs carga. Si el proceso corre con privilegios elevados (ej. SYSTEM), el atacante hereda esos privilegios.

Herramientas de ataque: **Spartacus**, DLLirant, ImpulsiveDLLHijack, PowerSploit.
Herramientas de defensa: **Dependency Walker**, DLL Hijack Audit Kit, DLLSpy.

**Defensa clave:** usar siempre rutas completamente cualificadas en las aplicaciones Windows. Colocar los ejecutables en directorios de solo lectura.

---

## 3. Dylib Hijacking (macOS)

El equivalente al DLL hijacking en macOS. El sistema permite cargar librerías dinámicas débiles (weak dylibs) y busca en múltiples rutas. El atacante coloca una dylib maliciosa en una de las rutas de búsqueda. Además, macOS permite técnicas como `DYLD_INSERT_LIBRARIES` (variable de entorno que fuerza la carga de librerías maliciosas en procesos en ejecución).

Usos para el atacante: persistencia sigilosa, inyección en procesos en tiempo de ejecución, bypass de software de seguridad y bypass de Gatekeeper.

Herramienta de detección: **Dylib Hijack Scanner (DHS)**.

---

## 4. Spectre y Meltdown

Son vulnerabilidades de hardware en procesadores modernos (Intel, AMD, ARM, Apple) causadas por las optimizaciones de rendimiento de los procesadores: **ejecución especulativa**, **predicción de ramas**, **caché** y **ejecución fuera de orden**.

**Spectre:** engaña al procesador para que ejecute especulativamente instrucciones y lea memoria fuera de los límites permitidos. Afecta prácticamente a todos los procesadores modernos. Permite leer datos de otros procesos (como credenciales almacenadas en el navegador) o incluso memoria del kernel.

**Meltdown:** específico de Intel y ARM. Explota que el procesador completa la evaluación especulativa de dos peticiones antes de comprobar si la primera es válida. Aunque el procesador rechaza ambas peticiones al detectar el acceso ilegal, los resultados quedan en caché. El atacante puede recuperar esa información cacheada. Permite a un proceso sin privilegios leer memoria del kernel.

La diferencia clave: Meltdown rompe el aislamiento entre usuario y kernel. Spectre rompe el aislamiento entre procesos.

Herramientas de detección: **InSpectre** (Windows), **Spectre & Meltdown Checker** (Linux).
Defensa: parchear OS y firmware, actualizar BIOS, retpoline, speculative load hardening.

---

## 5. Named Pipe Impersonation (Windows)

Los named pipes en Windows permiten comunicación entre procesos mediante ficheros. Cuando un proceso crea un pipe, actúa como **pipe server**. Cualquier otro proceso que se conecte es el **pipe client**. El servidor puede **impersonar** al cliente, adoptando su contexto de seguridad y privilegios.

El atacante crea un pipe server con pocos privilegios y espera que un proceso con más privilegios (como SYSTEM) se conecte como cliente. Al conectarse, el servidor hereda los privilegios del cliente.

En Metasploit, el comando `getsystem` automatiza esta técnica para obtener privilegios de administrador desde una sesión Meterpreter.

---

## 6. Misconfigured Services

### Unquoted Service Paths 🔴
Cuando Windows arranca un servicio cuyo path ejecutable tiene espacios y no está entre comillas, el sistema busca el binario probando todas las combinaciones posibles de separación del path. Por ejemplo, si el path es `C:\Program Files\My App\service.exe`, Windows prueba primero `C:\Program.exe`, luego `C:\Program Files\My.exe`, y finalmente `C:\Program Files\My App\service.exe`.

El atacante coloca un ejecutable malicioso en una de esas rutas intermedias (donde tenga permisos de escritura). Si el servicio corre bajo SYSTEM, el ejecutable malicioso se lanza con privilegios SYSTEM.

### Service Object Permissions
Una mala configuración de permisos sobre el objeto de servicio puede permitir al atacante modificar los atributos del servicio, incluyendo la ruta del binario que ejecuta. El atacante cambia esa ruta a su propio ejecutable malicioso y reinicia el servicio. También puede añadir nuevos usuarios al grupo de administradores locales.

### Unattended Installs
Los instaladores desatendidos de Windows guardan su configuración en el fichero `Unattend.xml`. Este fichero puede contener credenciales en texto claro o en base64, nombres de usuario y configuraciones de cuentas locales. Ubicaciones donde buscarlo:
- `C:\Windows\Panther\`
- `C:\Windows\Panther\UnattendGC\`
- `C:\Windows\System32\`
- `C:\Windows\System32\sysprep\`

Si el atacante accede a este fichero, obtiene credenciales que puede usar para escalar privilegios.

---

## 7. Pivoting y Relaying

Estas técnicas se usan **después de comprometer** un sistema para acceder a otros sistemas en la red interna que no son directamente accesibles desde la red del atacante.

### Pivoting
El sistema comprometido actúa como **puente** para llegar a otros sistemas internos. El tráfico del atacante viaja a través del sistema comprometido, por lo que el sistema objetivo no puede identificar el origen real del ataque.

Pasos con Metasploit:

**1. Descubrir hosts activos en la red interna:**
```
run post/windows/gather/arp_scanner RHOSTS <subnet>
```

**2. Configurar regla de routing** para que todo el tráfico hacia la red privada pase por la sesión Meterpreter:
```
background
route add <IP> <subnet_mask> <session_number>
```

**3. Escanear puertos de los sistemas descubiertos:**
```
use auxiliary/scanner/portscan/tcp
set RHOSTS <IPs>
set PORTS 1-1000
run
```

**4. Explotar servicios vulnerables** en los sistemas alcanzados (ej. BypassUAC).

### Relaying
Si el pivoting no funciona, el atacante usa **port forwarding** para acceder a recursos específicos del sistema interno a través del sistema comprometido. Los recursos del sistema interno aparecen como si estuvieran en localhost del atacante.

Pasos con Meterpreter:
1. Crear reglas de port forwarding para puertos como 80, 22 y 445.
2. Acceder a los recursos remapeados:
   - HTTP: `http://localhost:10080`
   - SSH: `ssh myadmin@localhost`

**Diferencia clave:** en pivoting se abre un shell remoto y se explota directamente. En relaying se accede a recursos del sistema comprometido a través de un túnel.

---

## 8. NFS Misconfiguration

NFS (Network File System) usa el **puerto 2049** y permite compartir ficheros en red. Una mala configuración puede permitir a un usuario sin privilegios montar shares y obtener acceso root.

Pasos de explotación:
```bash
# 1. Verificar si NFS está activo
nmap -sV <Target IP>

# 2. Instalar cliente NFS
sudo apt-get install nfs-common

# 3. Ver shares disponibles
showmount -e <Target IP>

# 4. Crear directorio de montaje
mkdir /tmp/nfs

# 5. Montar el share
sudo mount -t nfs <Target IP>:/<Share> /tmp/nfs

# 6. Copiar /bin/bash al share y verificar permisos
cd /tmp/nfs && sudo cp /bin/bash . && ls -la

# 7. Conectar por SSH para obtener acceso
ssh -l <user> <Target IP>
```

---

## 9. UAC Bypass (Windows) 🔴

UAC (User Account Control) es el mecanismo de Windows que solicita confirmación del usuario antes de ejecutar acciones con privilegios elevados. Los atacantes tienen 5 técnicas de bypass en Metasploit:

**bypassuac** — bypass mediante inyección en proceso. Genera una nueva sesión sin flag UAC:
```
use exploit/windows/local/bypassuac
```

**bypassuac_injection** — usa reflective DLL para inyectar solo el payload DLL en memoria. Obtiene `AUTHORITY\SYSTEM`:
```
use exploit/windows/local/bypassuac_injection
```

**bypassuac_fodhelper** — hijack de una clave del registro `HKCU` y la vincula a `fodhelper.exe`. Los comandos personalizados se ejecutan cuando arranca fodhelper:
```
use exploit/windows/local/bypassuac_fodhelper
```

**bypassuac_eventvwr** — hijack de clave `HKCU` ligada al Event Viewer. La clave se borra tras ejecutar el payload:
```
use exploit/windows/local/bypassuac_eventvwr
```

**bypassuac_comhijack** — crea entradas COM en `HKCU` que referencian procesos de alto nivel. Cuando se ejecutan, cargan DLLs maliciosas con sesiones elevadas:
```
use exploit/windows/local/bypassuac_comhijack
```

Tras el bypass, se ejecutan `getsystem` y `getuid` para confirmar privilegios SYSTEM.

---

## 10. Boot/Logon Initialization Scripts

Los atacantes abusan de scripts de arranque y login para mantener persistencia y escalar privilegios. Según el OS:

**Windows — Logon Script:** el atacante añade la ruta de su script en la clave de registro:
`HKEY_CURRENT_USER\Environment\UserInitMprLogonScript`
El script se ejecuta cada vez que el usuario inicia sesión.

**macOS — Login Hooks:** script ejecutado automáticamente al hacer login. Se ejecuta como **root**, lo que lo convierte en un vector de escalada directa.

**Network Logon Scripts:** asignados mediante AD o GPOs. Se ejecutan con las credenciales de cualquier usuario válido. Permiten acceso a múltiples sistemas de la red.

**RC Scripts (Unix/Linux):** scripts ejecutados durante el arranque del sistema (`rc.common`, `rc.local`). El atacante embebe un shell o binario malicioso en estos scripts. Al reiniciar el sistema, se ejecutan con privilegios root automáticamente.

**Startup Items (macOS):** se ejecutan al final del proceso de arranque con privilegios root. El atacante coloca ficheros maliciosos en `/Library/StartupItems/`. El fichero clave es `StartupParameters.plist`.

---

## 11. Domain Policy Modification

### Group Policy Modification
Las GPOs gestionan configuraciones de seguridad en todo el dominio. Por defecto, todos los usuarios tienen acceso de lectura a las GPOs, pero solo usuarios específicos tienen escritura. Si el atacante obtiene permisos de escritura:

- Puede crear nuevas cuentas.
- Puede deshabilitar herramientas internas.
- Puede modificar `ScheduledTasks.xml` para ejecutar tareas maliciosas:
  `<GPO_PATH>\Machine\Preferences\ScheduledTasks\ScheduledTasks.xml`
- Puede modificar `GptTmpl.inf` para cambiar derechos de usuario como `SeEnableDelegationPrivilege`:
  `<GPO_PATH>\MACHINE\Microsoft\Windows NT\SecEdit\GptTmpl.inf`

### Domain Trust Modification
El atacante usa `nltest /domain_trusts` para mapear las relaciones de confianza entre dominios y luego añade o modifica trusts para facilitar Kerberoasting y pass-the-ticket.

---

## 12. DCSync Attack 🔴

Un DC replica sus datos con otros DCs usando el protocolo **MS-DRSR** (Microsoft Directory Replication Service Remote Protocol). El DCSync attack explota esto: el atacante se hace pasar por un DC legítimo y solicita replicación para obtener los hashes NTLM de cualquier cuenta, incluido el administrador.

**Permisos necesarios:**
- Replicating Directory Changes
- Replicating Directory Changes All
- Replicating Directory Changes in Filtered Set

**8 etapas del ataque:**
1. Reconocimiento externo.
2. Comprometer la máquina objetivo.
3. Reconocimiento interno.
4. Escalada de privilegios local.
5. Comprometer credenciales enviando comandos al DC.
6. Reconocimiento a nivel de administrador.
7. Ejecución remota de código malicioso.
8. Obtener credenciales de Domain Admin.

**Ejecución con Mimikatz:**
```
mimikatz "lsadump::dcsync /domain:<domain> /user:Administrator"
```

Con los hashes obtenidos, el atacante puede lanzar Golden Ticket attacks, manipulación de cuentas y ataques LOTL (Living off the Land).

**Defensa:** limitar el permiso "Replicate Directory Changes" solo a cuentas autorizadas. Monitorizar qué cuentas solicitan replicación.

---

## 13. ADCS Abuse (ESC1) 🔴

Active Directory Certificate Services (ADCS) gestiona la PKI del dominio. Una plantilla de certificado mal configurada con la vulnerabilidad **ESC1** permite a cualquier usuario del dominio solicitar un certificado como administrador de dominio.

La condición es que la plantilla permita a usuarios de **Domain Users** o **Authenticated Users** inscribirse y especificar cualquier UPN (User Principal Name) en el certificado.

Pasos con **Certipy**:
```bash
# Enumerar plantillas vulnerables
certipy find -u '<user>@<domain>' -p <password> -dc-ip <DC_IP> -vulnerable -enabled

# Buscar ESC1 en los resultados
grep ESC1 <output>

# Solicitar certificado como Domain Admin
certipy req -u '<user>@foobar.com' -p '<pass>' -dc-ip '<ip>' \
  -target '<domain>' -ca 'foobar-CA' -template '<template>' -upn '<admin_UPN>'
```

Con el certificado obtenido, el atacante puede autenticarse como Domain Admin.

---

## 14. Otras Técnicas de Privilege Escalation

### Access Token Manipulation
Windows usa access tokens para determinar el contexto de seguridad de un proceso. Cualquier usuario Windows puede manipular su token para que un proceso parezca pertenecer a otro usuario. El atacante obtiene o falsifica el token de un usuario con más privilegios y lo usa para ejecutar procesos en su contexto. Comando legítimo que se abusa: `runas`.

### PPID Spoofing (Parent PID Spoofing)
Los procesos nuevos heredan el contexto de seguridad de su proceso padre. Usando la API `CreateProcess`, el atacante puede especificar un PPID diferente (ej. `svchost.exe` o `consent.exe`) para que el nuevo proceso parezca ser hijo de un proceso de confianza. Esto bypasa herramientas que analizan relaciones padre-hijo para detectar anomalías.

### Application Shimming
La Windows Application Compatibility Framework usa **shims** para hacer compatibles programas de versiones antiguas de Windows. Los shims se interponen entre la aplicación y el OS mediante API hooking. Un atacante puede instalar shims maliciosos para: bypassar UAC (`RedirectEXE`), inyectar DLLs maliciosas (`InjectDLL`), capturar direcciones de memoria (`GetProcAddress`), deshabilitar Windows Defender, etc. Los shims se almacenan en `%WINDIR%\AppPatch\sysmain.sdb`.

### Filesystem Permission Weakness
Si los permisos del sistema de ficheros sobre un binario que ejecuta automáticamente un proceso con privilegios altos no están bien configurados, el atacante puede reemplazar ese binario con uno malicioso. Cuando el proceso lo ejecute, lo hará con sus privilegios elevados. Objetivo frecuente: binarios de servicios Windows e instaladores autoextractibles.

### Path Interception
El atacante coloca un ejecutable malicioso en una ruta que el sistema busca antes que la ruta legítima. Variantes: unquoted paths, misconfiguration de la variable PATH, search order hijacking. Es especialmente efectivo si la variable PATH incluye el directorio actual (`.`).

### Abusing '.' in the PATH
Si un usuario privilegiado tiene `.` en su PATH, cualquier comando que ejecute buscará primero en el directorio actual. El atacante coloca un script malicioso con el nombre de un comando común (ej. `ls`) en un directorio donde pueda escribir. Cuando el usuario privilegiado ejecute `ls`, se ejecuta el script malicioso con sus privilegios.

### Accessibility Features Abuse
Las características de accesibilidad de Windows pueden ejecutarse incluso antes de que el usuario inicie sesión. El atacante reemplaza uno de estos binarios con `cmd.exe`. Cuando se presiona la combinación de teclas en la pantalla de login, se abre una CMD con privilegios SYSTEM. Binarios objetivo:
- Sticky Keys: `sethc.exe` (tecla Shift x5)
- On-screen keyboard: `osk.exe`
- Magnifier: `Magnify.exe`
- Narrator: `Narrator.exe`

### SID-History Injection
El atributo `SID-History` de una cuenta AD permite mantener SIDs de dominios anteriores (útil en migraciones). El atacante inyecta el SID de una cuenta de administrador en el atributo SID-History de una cuenta comprometida de bajos privilegios. Windows evalúa todos los SIDs del historial al autorizar acceso, por lo que la cuenta comprometida hereda los privilegios del administrador cuyo SID fue inyectado.

### COM Hijacking
El Component Object Model (COM) permite que componentes software interactúen entre sí. Windows busca las referencias COM primero en `HKEY_CURRENT_USER\Software\Classes\CLSID\` y luego en `HKEY_LOCAL_MACHINE\SOFTWARE\Classes\CLSID\`. El atacante crea una entrada maliciosa en HKCU (donde puede escribir sin privilegios) que shadea la legítima en HKLM. Cuando un proceso ejecuta ese COM object, carga el payload del atacante.

### Scheduled Tasks (Windows)
Usando `schtasks` o `at`, el atacante puede programar la ejecución de código malicioso en el arranque del sistema, mantener persistencia, ejecutar código remotamente o escalar privilegios.

### Cron Jobs (Linux)
Linux usa `cron` para automatizar tareas. El fichero de configuración principal es `/etc/crontab`. Si el atacante puede modificar los scripts que cron ejecuta, puede forzar la ejecución de código malicioso con privilegios root en cada reinicio.

Comandos clave:
- `crontab -l` → listar crontabs activos
- `crontab -e` → editar el crontab del usuario actual
- `crontab -u <user> -e` → editar el crontab de un usuario específico

### Launch Daemons (macOS)
Los launch daemons se cargan en el arranque desde `/System/Library/LaunchDaemons` y `/Library/LaunchDaemons`. Son ficheros plist ligados a ejecutables que corren al arrancar. El atacante instala un nuevo daemon malicioso o modifica uno existente para ejecutar su payload con privilegios root en cada arranque.

### Plist Modification (macOS)
Los ficheros plist configuran cuándo y cómo se ejecutan las aplicaciones. Los ubicados en `/Library/Preferences/` se ejecutan con privilegios altos. Si el atacante modifica estos ficheros, puede ejecutar código malicioso en nombre de un usuario legítimo.

### Setuid / Setgid (Linux/macOS)
Si un binario tiene el flag **SUID** activado, se ejecuta con los privilegios del propietario del fichero, no del usuario que lo lanza. Si el propietario es root y el binario tiene SUID, cualquier usuario puede ejecutarlo con privilegios root. Binarios comunes con SUID explotables: `nmap`, `vim`, `less`, `more`, `bash`, `find`, `cp`.

Encontrar binarios SUID/SGID:
```bash
find / -perm -u=s -type f 2>/dev/null   # SUID
find / -perm -g=s -type f 2>/dev/null   # SGID
```

### Web Shell
Script alojado en un servidor web que da acceso remoto al atacante. Puede ejecutarse bajo los privilegios del usuario del servidor web y usarse como punto de partida para escalar privilegios explotando vulnerabilidades locales.

### Abusing Sudo Rights
El fichero `/etc/sudoers` define qué comandos puede ejecutar cada usuario con privilegios de superusuario. Si el atacante tiene sudo sobre comandos que permiten escritura arbitraria (ej. `cp`), puede sobreescribir `/etc/sudoers` o `/etc/shadow` con versiones maliciosas que le den acceso root total.

### Abusing SUID/SGID Permissions
Similar a setuid/setgid pero con enfoque específico en binarios del sistema. La diferencia con setuid es que SUID es el flag de permiso específico en Linux. El atacante busca binarios con SUID del sistema que permitan spawning de shells o escritura arbitraria.

### Kernel Exploits (Linux)
Exploits que aprovechan vulnerabilidades en el kernel para ejecutar código con privilegios root. Para encontrar exploits aplicables, el atacante obtiene información del sistema:
```bash
cat /etc/issue        # Distribución OS
uname -a              # Versión del kernel
cat /proc/version     # Información del kernel
```
Luego busca en exploit-db.com o usa scripts como `linprivchecker.py`.

### Named Pipe Impersonation → ya explicado en sección 5.

### MSI Abuse (Windows)
Los ficheros MSI en `C:\Windows\Installer\` pueden ser reparados por usuarios estándar. La funcionalidad "repair" puede lanzar acciones (Custom Actions) en el contexto de `NT AUTHORITY\SYSTEM`. Si esas Custom Actions operan en directorios donde el usuario tiene permisos de escritura, el atacante puede manipular los ficheros para ejecutar código arbitrario con privilegios SYSTEM.

### NoFilter Attack (Windows 11)
Explota la **Windows Filtering Platform (WFP)** para obtener privilegios SYSTEM sin ser detectado. El atacante usa RPC para invocar `BfeRpcOpenToken` y obtiene el handle del token de un proceso con privilegios SYSTEM. Luego duplica ese token mediante `WfpAle` o una conexión IPsec y lo usa para crear procesos hijo o impersonar otros usuarios con privilegios elevados.

### Process Injection via Ptrace (Linux)
El atacante usa la llamada al sistema `ptrace` (normalmente usada para depuración) para inyectar código malicioso en un proceso en ejecución. Permite modificar la memoria y los registros del proceso objetivo. Técnicas: `malloc` para escribir código + `PTRACE_SETREGS` para redirigir la ejecución, o `PTRACE_POKETEXT`/`PTRACE_POKEDATA` para escribir en direcciones específicas.

### AuthorizationExecuteWithPrivileges API Abuse (macOS)
API diseñada para que los desarrolladores puedan ejecutar operaciones con root (instalaciones de software). No valida si la solicitud viene de una fuente de confianza. El atacante la invoca para ganar privilegios root sin verificación. Además, el programa que usa la API puede cargar ficheros world-writable que el atacante puede modificar para ejecutar acciones maliciosas con privilegios elevados.

---

## 15. Herramientas de Privilege Escalation

**BeRoot:** post-explotación; detecta misconfigurations comunes (permisos de servicios, directorios con escritura, claves de startup).

**pwncat:** enumera y explota vectores de escalada de privilegios. Comandos clave:
```
pwncat$ escalate list           # Listar vectores de escalada para cualquier usuario
pwncat$ escalate list -u root   # Listar vectores hacia root
pwncat$ escalate run            # Ejecutar la escalada
```

**PowerSploit:** framework PowerShell para post-explotación en Windows.

**PEASS-ng:** enumera paths de escalada en Windows y Linux (WinPEAS, LinPEAS).

**linpostexp / Windows Exploit Suggester:** identifican kernel exploits y misconfigurations aplicables al sistema objetivo.

---

## 3. Exam Traps ⚠️

⚠️ **[Horizontal vs Vertical Privilege Escalation]** — El examen los confunde. Horizontal = mismo nivel de privilegios, acceso a datos de otro usuario. Vertical = escalar a un nivel superior (usuario → admin). El más buscado por atacantes es el vertical.

⚠️ **[DLL Hijacking: condición necesaria]** — No basta con que la aplicación no use rutas completas. El atacante necesita **permisos de escritura en el directorio de la aplicación**. Sin eso, no puede colocar la DLL maliciosa.

⚠️ **[Spectre vs Meltdown]** — Spectre: rompe aislamiento entre procesos, afecta casi todos los procesadores. Meltdown: rompe aislamiento usuario-kernel, específico de Intel/ARM. El examen puede pedir cuál afecta a qué tipos de procesadores.

⚠️ **[Unquoted Service Paths]** — El examen puede presentar un path con espacios sin comillas y preguntar qué técnica explota eso. Es "Unquoted Service Paths". El atacante necesita permisos de escritura en alguna de las rutas intermedias que Windows probará.

⚠️ **[Pivoting vs Relaying]** — Pivoting: el atacante abre una shell remota en el sistema interno a través del sistema comprometido. Relaying: el atacante accede a recursos del sistema interno a través de un túnel (port forwarding), sin shell directa. El examen los puede presentar como sinónimos pero no lo son.

⚠️ **[NFS port]** — NFS usa el **puerto 2049**. El examen puede preguntar el puerto específico de NFS o del protocolo RPC que usa.

⚠️ **[UAC Bypass Metasploit modules]** — El examen puede preguntar qué módulo usa reflective DLL injection: `bypassuac_injection`. El que hijackea fodhelper.exe: `bypassuac_fodhelper`. El que se limpia solo tras ejecutar: `bypassuac_eventvwr`.

⚠️ **[DCSync: permisos necesarios]** — No basta con ser usuario de dominio. Se necesitan los permisos específicos: "Replicating Directory Changes", "Replicating Directory Changes All" y "Replicating Directory Changes in Filtered Set". El examen puede preguntar exactamente cuáles.

⚠️ **[DCSync: herramienta y comando]** — La herramienta es **Mimikatz** con el módulo `lsadump::dcsync`. El examen puede pedir el nombre del protocolo que emula: **MS-DRSR** (Microsoft Directory Replication Service Remote Protocol).

⚠️ **[ADCS ESC1]** — ESC1 permite que usuarios de **Domain Users o Authenticated Users** soliciten certificados como cualquier cuenta del dominio, incluido Domain Admin. La herramienta de explotación es **Certipy**. El examen puede preguntar qué vulnerabilidad de ADCS permite request de certificados con UPN arbitrario.

⚠️ **[SID-History Injection]** — No confundir con Pass-the-Ticket. SID-History inyecta el SID de un admin en el atributo SID-History de otra cuenta, sin necesitar el hash ni el ticket. Windows evalúa todos los SIDs del historial en la autorización.

⚠️ **[COM Hijacking: registros]** — COM busca primero en `HKEY_CURRENT_USER` (HKCU) antes que en `HKEY_LOCAL_MACHINE` (HKLM). El atacante puede escribir en HKCU sin privilegios elevados. Esto es la base del ataque.

⚠️ **[Setuid vs Sudo]** — SUID es un flag de permiso en el fichero; hace que el binario se ejecute con los privilegios del propietario. Sudo es un mecanismo de delegación de comandos. Son mecanismos distintos aunque ambos pueden dar acceso root.

⚠️ **[crontab: fichero principal]** — El fichero cron del sistema está en `/etc/crontab`. Los crontabs de usuario se gestionan con el comando `crontab -e`. El examen puede preguntar la ruta del fichero de configuración principal de cron.

⚠️ **[Unattend.xml]** — El examen puede preguntar dónde buscar credenciales de instalación desatendida. Ubicaciones clave: `C:\Windows\Panther\` y `C:\Windows\System32\sysprep\`.

⚠️ **[NoFilter Attack]** — Específico de Windows 11. Explota WFP (Windows Filtering Platform), no UAC. El examen puede presentar WFP como distractor o confundirlo con otros bypass de UAC.

---

## 4. Nemotécnicos

**Tipos de privilege escalation:**
> **H-V** → **H**orizontal (mismo nivel, otro usuario), **V**ertical (nivel superior)

**UAC Bypass modules (5):**
> **"Bypass Injection Fod Event COM"**
> bypassuac → bypassuac_**i**njection → bypassuac_**f**odhelper → bypassuac_**e**ventvwr → bypassuac_**c**omhijack

**DCSync 8 stages:**
> **"Ext-Comp-Int-Local-Cred-Admin-RCE-Domain"**
> External recon → Compromise → Internal recon → Local privesc → Credentials via DC → Admin recon → Remote code exec → Domain admin

**Misconfigurations explotables (3 principales):**
> **U-S-U** → **U**nquoted service paths, **S**ervice object permissions, **U**nattended installs

**Técnicas Linux de escalada:**
> **S-K-C-'.'** → **S**etuid/Setgid, **K**ernel exploits, **C**ron jobs, **'.'** in PATH

**Técnicas Windows de escalada:**
> **D-A-S-P-F-C** → **D**LL hijacking, **A**ccess token manipulation, **S**himming, **P**PID spoofing, **F**ilesystem permissions, **C**OM hijacking

---

## 5. Flashcards

**Q:** ¿Qué diferencia hay entre escalada horizontal y vertical de privilegios?
**A:** Horizontal = accede a recursos de otro usuario con el mismo nivel de privilegios. Vertical = obtiene privilegios superiores a los que tenía (ej. user → admin).

**Q:** ¿Qué condición se necesita para explotar DLL hijacking?
**A:** Que la aplicación no use rutas completamente cualificadas al cargar DLLs Y que el atacante tenga permisos de escritura en el directorio de la aplicación.

**Q:** ¿Qué vulnerabilidad de CPU rompe el aislamiento entre usuario y kernel?
**A:** Meltdown (Intel/ARM). Spectre rompe el aislamiento entre procesos.

**Q:** ¿Qué es un Unquoted Service Path y cómo se explota?
**A:** Path de ejecutable de servicio con espacios y sin comillas. Windows prueba rutas intermedias; el atacante coloca un binario malicioso en una de esas rutas con permisos de escritura.

**Q:** ¿En qué directorio busca Windows credenciales de instalación desatendida?
**A:** Principalmente en `C:\Windows\Panther\` y `C:\Windows\System32\sysprep\` (fichero `Unattend.xml`).

**Q:** ¿Qué puerto usa NFS?
**A:** Puerto 2049.

**Q:** ¿Qué módulo de Metasploit usa reflective DLL injection para bypassar UAC?
**A:** `bypassuac_injection`.

**Q:** ¿Qué módulo de Metasploit hijackea el registro de fodhelper.exe para bypassar UAC?
**A:** `bypassuac_fodhelper`.

**Q:** ¿Qué permisos necesita el atacante para realizar un DCSync attack?
**A:** Replicating Directory Changes, Replicating Directory Changes All, Replicating Directory Changes in Filtered Set.

**Q:** ¿Qué protocolo emula Mimikatz para realizar el DCSync attack?
**A:** MS-DRSR (Microsoft Directory Replication Service Remote Protocol).

**Q:** ¿Qué comando de Mimikatz ejecuta un DCSync attack?
**A:** `mimikatz "lsadump::dcsync /domain:<domain> /user:Administrator"`

**Q:** ¿Qué vulnerabilidad ADCS permite solicitar certificados como Domain Admin desde una cuenta sin privilegios?
**A:** ESC1. Herramienta: Certipy.

**Q:** ¿Qué diferencia hay entre pivoting y relaying?
**A:** Pivoting: shell remota en el sistema interno a través del comprometido. Relaying: port forwarding para acceder a recursos del sistema interno sin shell directa.

**Q:** ¿Cómo se inyecta una sesión de administrador en el contexto de Named Pipe Impersonation?
**A:** El atacante crea un pipe server con pocos privilegios y espera que un proceso privilegiado se conecte como cliente. Al conectarse, el servidor adopta el contexto de seguridad del cliente.

**Q:** ¿En qué clave del registro buscan los COM objects antes que en HKLM?
**A:** `HKEY_CURRENT_USER\Software\Classes\CLSID\`. El atacante puede escribir aquí sin privilegios.

**Q:** ¿Qué fichero contiene la configuración de sudo en Linux?
**A:** `/etc/sudoers`.

**Q:** ¿Qué fichero contiene los cron jobs del sistema en Linux?
**A:** `/etc/crontab`.

**Q:** ¿Qué comando encuentra binarios con SUID en Linux?
**A:** `find / -perm -u=s -type f 2>/dev/null`

**Q:** ¿Qué es el ataque NoFilter y qué componente de Windows explota?
**A:** Técnica de privilege escalation en Windows 11 que explota la Windows Filtering Platform (WFP) para duplicar tokens de acceso SYSTEM sin ser detectado.

**Q:** ¿Qué atributo de AD permite a los atacantes inyectar el SID de un admin en una cuenta comprometida?
**A:** SID-History.

**Q:** ¿Qué herramienta de post-explotación para Linux enumera vectores de escalada con comandos `escalate list` y `escalate run`?
**A:** pwncat.

---

## 6. Confusión frecuente

**Horizontal vs Vertical Escalation** → Horizontal: mismo nivel, distinto usuario (ej. acceder a cuenta bancaria ajena). Vertical: nivel superior (ej. user → admin/root). El vertical es el objetivo final del atacante.

**DLL Hijacking vs Dylib Hijacking** → DLL Hijacking es Windows; Dylib Hijacking es macOS. Mismo principio: aplicación carga librería dinámica desde ruta no completamente cualificada.

**Spectre vs Meltdown** → Spectre: todos los procesadores modernos, rompe aislamiento entre procesos. Meltdown: Intel/ARM, rompe aislamiento usuario-kernel.

**Pivoting vs Relaying** → Pivoting: shell remota a través del sistema comprometido. Relaying: acceso a recursos mediante port forwarding. Pivoting es más agresivo; relaying es más sigiloso.

**DCSync vs Pass-the-Hash** → DCSync extrae hashes del DC haciéndose pasar por otro DC (requiere permisos de replicación). PtH usa un hash ya obtenido para autenticarse directamente. Son ataques distintos que pueden encadenarse.

**UAC Bypass bypassuac vs bypassuac_injection** → `bypassuac`: process injection genérica. `bypassuac_injection`: usa reflective DLL, solo inyecta el DLL payload en memoria (más sigiloso).

**bypassuac_eventvwr vs bypassuac_fodhelper** → Ambos hijackean claves HKCU. `eventvwr`: asociado al Event Viewer, se auto-limpia. `fodhelper`: asociado a fodhelper.exe, persiste hasta nueva ejecución.

**Setuid vs Sudo** → Setuid es un flag de permiso en el fichero binario. Sudo es un mecanismo de autorización de comandos en `/etc/sudoers`. Ambos pueden dar root, pero el mecanismo es distinto.

**SUID vs SGID** → SUID: el binario se ejecuta con los privilegios del propietario del fichero. SGID: se ejecuta con los privilegios del grupo propietario. Ambos son flags de permiso en Linux/macOS.

**SID-History Injection vs Pass-the-Ticket** → SID-History: inyecta SID de admin en el atributo historial de la cuenta. PtT: roba y reutiliza tickets Kerberos. Mecanismos completamente distintos; SID-History no requiere tickets.

**Application Shimming vs DLL Injection** → Shimming usa la Windows Application Compatibility Framework para interceptar llamadas API legítimamente. DLL Injection fuerza la carga de una DLL en un proceso en ejecución. Shimming es más difícil de detectar porque usa infraestructura legítima del OS.

**COM Hijacking vs DLL Hijacking** → COM Hijacking manipula entradas de registro COM en HKCU. DLL Hijacking coloca una DLL maliciosa en el directorio de la aplicación. Ambos consiguen que se ejecute código malicioso cuando se invoca un componente legítimo.

---

## 16. Preguntas de Práctica — Formato CEH

---

### Pregunta 1 — Unquoted Service Paths
Un administrador revisa los servicios de Windows y encuentra que el servicio "MyCorpApp" tiene la siguiente ruta de binario: `C:\Program Files\Corp Applications\MyApp\service.exe`. El servicio corre bajo SYSTEM. ¿Qué rutas intentará Windows ejecutar antes de llegar al binario correcto?

A) Solo `C:\Program Files\Corp Applications\MyApp\service.exe`  
B) `C:\Program.exe`, luego `C:\Program Files\Corp.exe`, luego `C:\Program Files\Corp Applications\MyApp\service.exe`  
C) `C:\Program.exe`, luego `C:\Program Files\Corp.exe`, luego `C:\Program Files\Corp Applications\MyApp.exe`  
D) `C:\Program.exe`, luego `C:\Program Files\Corp.exe`, luego `C:\Program Files\Corp Applications\MyApp.exe`, luego `C:\Program Files\Corp Applications\MyApp\service.exe`  

> **Respuesta correcta: D** — Windows prueba todas las combinaciones posibles de separación del path antes de llegar al correcto. Con 4 palabras separadas por espacios, hay 4 rutas que se prueban. Si el atacante puede escribir en alguna de esas carpetas intermedias, puede colocar un binario malicioso que se ejecutará con privilegios SYSTEM.

---

### Pregunta 2 — DLL Hijacking
Un pentester quiere explotar DLL Hijacking en una aplicación que corre con privilegios elevados. La aplicación no usa rutas completamente cualificadas para cargar `helplib.dll`. ¿Cuál es el requisito adicional imprescindible?

A) Que la aplicación esté ejecutándose actualmente  
B) Que el atacante tenga permisos de escritura en el directorio desde el que se lanza la aplicación  
C) Que el sistema no tenga antivirus activo  
D) Que la DLL original no esté presente en el sistema  

> **Respuesta correcta: B** — Además de que la aplicación no use rutas completamente cualificadas, el atacante **necesita permisos de escritura en el directorio de la aplicación** para colocar la DLL maliciosa. Sin permisos de escritura, no puede colocar el fichero malicioso. La presencia del antivirus y del DLL original son factores secundarios.

---

### Pregunta 3 — DCSync Attack
Un analista de seguridad detecta que una cuenta de usuario del dominio (no admin) está ejecutando el comando `mimikatz "lsadump::dcsync /domain:corp.local /user:Administrator"`. ¿Qué permisos debe tener esa cuenta para que el ataque sea exitoso?

A) Membership en el grupo Domain Admins  
B) Replicating Directory Changes, Replicating Directory Changes All, y Replicating Directory Changes in Filtered Set  
C) Full Control sobre el objeto Domain Controller en Active Directory  
D) SeDebugPrivilege en el Domain Controller  

> **Respuesta correcta: B** — El DCSync attack requiere específicamente los permisos de replicación de AD. Una cuenta con esos permisos puede solicitar replicación al DC como si fuera otro DC, obteniendo hashes NTLM de cualquier cuenta. No requiere ser Domain Admin directamente, aunque normalmente los Domain Admins tienen esos permisos por defecto.

---

### Pregunta 4 — UAC Bypass
Durante un engagement de red team, un atacante tiene una sesión Meterpreter con privilegios de usuario estándar en Windows 10 y quiere bypassar UAC de la forma más sigilosa posible (solo inyectando el DLL payload en memoria, sin tocar el disco). ¿Qué módulo de Metasploit debe usar?

A) `exploit/windows/local/bypassuac`  
B) `exploit/windows/local/bypassuac_fodhelper`  
C) `exploit/windows/local/bypassuac_injection`  
D) `exploit/windows/local/bypassuac_eventvwr`  

> **Respuesta correcta: C** — `bypassuac_injection` usa **reflective DLL injection**: inyecta únicamente el DLL payload en memoria sin escribir en disco, lo que lo hace más sigiloso. `bypassuac_fodhelper` hijackea el registro de fodhelper.exe. `bypassuac_eventvwr` hijackea Event Viewer y se auto-limpia. `bypassuac` usa inyección en proceso más genérica.

---

### Pregunta 5 — Spectre vs Meltdown
Un auditor de seguridad evalúa el riesgo de dos vulnerabilidades hardware en procesadores de un datacenter. Meltdown afecta procesadores Intel y algunos ARM. Spectre afecta prácticamente a todos los procesadores modernos. ¿Cuál es la diferencia fundamental entre ambas en cuanto al aislamiento que rompen?

A) Meltdown rompe el aislamiento entre procesos; Spectre rompe el aislamiento entre usuario y kernel  
B) Spectre rompe el aislamiento entre procesos; Meltdown rompe el aislamiento entre usuario y kernel  
C) Ambas rompen el mismo aislamiento pero en distintos tipos de procesadores  
D) Meltdown afecta a la caché L1; Spectre afecta a la caché L2  

> **Respuesta correcta: B** — **Spectre** rompe el aislamiento entre **procesos** (puede leer memoria de otros procesos). **Meltdown** rompe el aislamiento entre **espacio de usuario y kernel** (permite a procesos sin privilegios leer memoria del kernel). Ambas explotan la ejecución especulativa pero con objetivos distintos.

---

### Pregunta 6 — Setuid en Linux
Un pentester ha accedido a un sistema Linux con una cuenta de usuario sin privilegios. Ejecuta el siguiente comando: `find / -perm -u=s -type f 2>/dev/null` y encuentra `/usr/bin/find` con SUID activado y propietario root. ¿Cómo puede usar esto para escalar privilegios?

A) No puede; `find` no tiene funcionalidades de shell  
B) Puede ejecutar `find /bin/bash -exec bash -p \;` para obtener una shell root  
C) Puede modificar el binario `/usr/bin/find` para inyectar código malicioso  
D) Puede usar `find` para localizar otros binarios SUID explotables  

> **Respuesta correcta: B** — Cuando `find` tiene SUID activado y es propietario root, se ejecuta con privilegios root. El flag `-exec` permite ejecutar comandos arbitrarios en el contexto del SUID. `bash -p` abre una shell en modo privilegiado preservando el EUID root. Este es uno de los GTFOBins clásicos del CEH.

---

### Pregunta 7 — ADCS ESC1
Un auditor identifica una plantilla de certificado en Active Directory Certificate Services que tiene habilitada la opción que permite a "Authenticated Users" inscribirse y especificar cualquier Subject Alternative Name (UPN). ¿Qué vulnerabilidad ADCS representa esto y qué puede hacer un atacante?

A) ESC2; el atacante puede emitir certificados de CA subordinada  
B) ESC1; el atacante puede solicitar un certificado como Domain Admin y autenticarse con él  
C) ESC4; el atacante puede modificar la plantilla para obtener permisos adicionales  
D) ESC8; el atacante puede realizar NTLM relay contra los endpoint HTTP de ADCS  

> **Respuesta correcta: B** — **ESC1** es la vulnerabilidad donde una plantilla permite a usuarios estándar especificar cualquier UPN en el certificado. Con la herramienta **Certipy**, el atacante puede solicitar un certificado con el UPN de un Domain Admin y usarlo para autenticarse. Herramienta: `certipy req -upn 'administrator@corp.local'`.

---

### Pregunta 8 — SID History Injection
Un atacante ha comprometido una cuenta de usuario estándar en el dominio y quiere obtener privilegios de administrador sin usar Pass-the-Ticket ni Pass-the-Hash. ¿Qué técnica le permite aprovechar el atributo SID-History de Active Directory?

A) Añadir el SID del Domain Admin al atributo SID-History de la cuenta comprometida  
B) Modificar el SID de la cuenta comprometida para que coincida con el del Domain Admin  
C) Añadir la cuenta comprometida directamente al grupo Domain Admins  
D) Crear un Golden Ticket con el SID de la cuenta comprometida  

> **Respuesta correcta: A** — **SID-History Injection**: el atacante inyecta el SID del Domain Admin en el atributo SID-History de la cuenta comprometida. Windows evalúa todos los SIDs del historial durante la autorización, por lo que la cuenta comprometida hereda los privilegios del admin cuyo SID fue inyectado. No requiere tickets ni hashes; opera a nivel de atributos de AD.

---

### Pregunta 9 — COM Hijacking
Un red teamer quiere usar COM Hijacking para escalar privilegios en Windows sin necesitar privilegios de administrador. ¿En qué clave del registro debe crear la entrada maliciosa?

A) `HKEY_LOCAL_MACHINE\SOFTWARE\Classes\CLSID\`  
B) `HKEY_CURRENT_USER\Software\Classes\CLSID\`  
C) `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\`  
D) `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run\`  

> **Respuesta correcta: B** — Windows busca referencias COM primero en **HKCU** antes que en HKLM. El atacante puede escribir en HKCU sin privilegios elevados, creando una entrada que shadea la entrada legítima de HKLM. Cuando el proceso privilegiado invoca ese COM object, carga el payload del atacante. HKLM requiere privilegios de administrador para escribir.

---

### Pregunta 10 — Named Pipe Impersonation
Un atacante con una sesión Meterpreter de bajos privilegios ejecuta el comando `getsystem`. ¿Qué técnica subyacente usa este comando para intentar escalar a SYSTEM?

A) DLL Hijacking del proceso winlogon.exe  
B) Named Pipe Impersonation: crea un pipe server y espera que un proceso SYSTEM se conecte como cliente  
C) Kernel Exploit utilizando vulnerabilidades de EternalBlue  
D) Token Impersonation usando el token de un proceso SYSTEM en ejecución  

> **Respuesta correcta: B** — `getsystem` usa **Named Pipe Impersonation** (entre otras técnicas). Crea un pipe server con bajos privilegios y espera que un proceso privilegiado (SYSTEM) se conecte como cliente al pipe. Al conectarse, el servidor pipe hereda el contexto de seguridad del cliente privilegiado. Es la técnica más usada y documentada en Metasploit.

---

### Pregunta 11 — Pivoting en Metasploit
Tras comprometer un sistema Windows en la red 10.10.10.0/24, un atacante necesita acceder a un servidor en la red interna 192.168.1.0/24 que no es directamente accesible desde su máquina atacante. ¿Qué comando configura la regla de routing en Metasploit para pivoting?

A) `route add 192.168.1.0 255.255.255.0 1`  
B) `route add 192.168.1.0/24 1`  
C) `use post/multi/manage/autoroute` con `set SUBNET 192.168.1.0`  
D) Tanto A como C son correctas  

> **Respuesta correcta: D** — Ambas opciones son válidas. `route add <IP> <subnet_mask> <session_number>` añade la ruta manualmente. El módulo `post/multi/manage/autoroute` lo hace automáticamente. Una vez configurada la ruta, el tráfico hacia 192.168.1.0/24 pasa automáticamente a través de la sesión Meterpreter comprometida.

---

### Pregunta 12 — Horizontal vs Vertical Escalation
Un atacante ha comprometido la cuenta del usuario "alice" en una aplicación web bancaria. Usando esa sesión, accede a la cuenta bancaria del usuario "bob" modificando el parámetro `account_id=bob` en la URL, sin obtener privilegios administrativos. ¿Qué tipo de escalada de privilegios ha ocurrido?

A) Vertical Privilege Escalation  
B) Horizontal Privilege Escalation  
C) Remote Code Execution  
D) Authorization Bypass via SQL Injection  

> **Respuesta correcta: B** — **Horizontal Privilege Escalation**: acceder a recursos de otro usuario con el **mismo nivel de privilegios**. Alice accede a los datos de Bob sin necesidad de privilegios superiores, solo accediendo a lo que no le corresponde. Si hubiera accedido a funciones de administrador, sería escalada vertical.

