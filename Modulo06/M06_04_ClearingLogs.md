# M06 — Gaining Access
### Parte 4: Clearing Logs / Covering Tracks

---

## 1. Concepto de Covering Tracks

Covering tracks es la última etapa del sistema de hacking. El atacante elimina o manipula toda evidencia de su actividad para evitar ser detectado, identificado o rastreado. El objetivo es dejar el sistema en el estado aparente que tenía antes del compromiso.

Los logs son el primer recurso que usa un administrador cuando detecta actividad anómala. Por eso el atacante los manipula, borra o deshabilita antes de que puedan ser analizados. En algunos casos, los rootkits automatizan este proceso desactivando y descartando todos los logs existentes.

Archivos de log clave en Windows que el atacante manipula:
- **SECEVENT.EVT** (Security): logins fallidos, acceso a ficheros sin privilegios.
- **SYSEVENT.EVT** (System): fallos de drivers, servicios que no funcionan correctamente.
- **APPEVENT.EVT** (Application): eventos de aplicaciones.

---

## 2. Técnicas de Covering Tracks

### Disabling Auditing (Auditpol) 🔴
El primer paso tras obtener privilegios de administrador es **deshabilitar la auditoría** para que el sistema deje de registrar actividad. La herramienta es `auditpol.exe`, utilidad de línea de comandos para modificar la configuración de auditoría de seguridad a nivel de categoría y subcategoría.

```cmd
# Ver configuración actual de auditoría
auditpol /get /category:*

# Deshabilitar auditoría (cubrir tracks)
auditpol /set /category:"system","account logon" /success:disable /failure:disable

# Rehabilitar auditoría al salir (para no levantar sospechas)
auditpol /set /category:"system","account logon" /success:enable /failure:enable
```

El atacante deshabilita la auditoría al entrar, realiza sus operaciones, y la vuelve a habilitar al salir para no dejar evidencia de que estuvo desactivada.

### Clearing Logs

**Meterpreter (método más directo en pentesting):**
```
meterpreter> clearev
```
Borra todos los logs del sistema Windows desde la sesión Meterpreter.

**PowerShell (Clear-EventLog):**
```powershell
# Borrar un log específico
Clear-EventLog "Windows PowerShell"

# Borrar múltiples logs en múltiples sistemas
Clear-EventLog -LogName ODiag, OSession localhost, Server02

# Borrar con confirmación
Clear-EventLog -LogName application, system -confirm
```
Parámetros: `-ComputerName` (sistema remoto), `-LogName` (log a borrar), `-Confirm` (pide confirmación), `-WhatIf` (simula sin ejecutar).

**wevtutil (utilidad nativa de Windows):**
```cmd
# Listar todos los event logs disponibles
wevtutil el

# Borrar un log específico
wevtutil cl system
wevtutil cl application
wevtutil cl security
```

**Manualmente en Windows:** Event Viewer → Start → Control Panel → System and Security → Windows Tools → Event Viewer → borrar entradas.

**Manualmente en Linux:** los logs están en `/var/log/`. Abrir el fichero de texto y borrar las entradas.

### Covering BASH Shell Tracks 🔴
El historial de comandos de Bash se almacena en `~/.bash_history`. Un investigador forense puede usar este fichero para reconstruir exactamente qué comandos ejecutó el atacante.

```bash
# Ver el historial actual
more ~/.bash_history

# Deshabilitar el historial completamente (HISTSIZE=0)
export HISTSIZE=0
# Determina cuántos comandos se guardan. Con 0, no se guarda ninguno.

# Limpiar el historial de la sesión actual
history -c

# Limpiar el historial solo de la shell actual (otros shells no se ven afectados)
history -w

# Borrar completamente el historial de todas las shells y salir
cat /dev/null > ~/.bash_history && history -c && exit

# Destruir el fichero de historial (contenido ilegible aunque se encuentre)
shred ~/.bash_history

# Destruir + borrar + limpiar historial de sesión + salir
shred ~/.bash_history && cat /dev/null > ~/.bash_history && history -c && exit
```

### Covering Tracks on a Network

**Reverse HTTP Shells:** el atacante instala un reverse HTTP shell en la víctima que periódicamente consulta al servidor del atacante (como un cliente web haciendo GET). Como el tráfico sale como HTTP normal desde la red, los firewalls y DMZ lo consideran tráfico legítimo. El atacante responde como un servidor web con comandos; la víctima los ejecuta y envía resultados en la siguiente petición.

**Reverse ICMP Tunneling:** encapsula payload TCP dentro de paquetes ICMP echo/reply. La mayoría de organizaciones inspeccionan los ICMP entrantes pero no los salientes, por lo que el tráfico pasa desapercibido. El flujo: cliente local → víctima (encapsula TCP en ICMP echo) → proxy server (desencapsula y envía TCP al atacante).

**DNS Tunneling:** codifica datos maliciosos o de otros programas dentro de queries y respuestas DNS. DNS tunneling puede crear un backchannel para exfiltrar datos o acceder a servidores remotos. El DNS es raramente inspeccionado en profundidad. Proceso: comprometer sistema interno → usarlo como C2 → transferir ficheros encubiertos entre red interna y externa via DNS.

**TCP Parameters (Covert Channels):** distribuye payload en campos específicos del protocolo TCP/IP:
- **IP Identification Field:** un carácter por paquete, transferido bitwise en una sesión establecida.
- **TCP Acknowledgement Number:** usa un bounce server; un carácter oculto por paquete. Más difícil.
- **TCP Initial Sequence Number:** un carácter por SYN/RST, no necesita conexión establecida.

### Covering Tracks on OS (Windows)
**NTFS ADS:** ocultar ficheros usando Alternate Data Streams (ya explicado en el chunk anterior).

**Modificar timestamps (timestomp):**
```
# Windows - Metasploit/timestomp
timestomp file_name.doc -z "<Date> <time>"

# Windows - PowerShell
powershell -Command "(Get-Item $File_name).LastWriteTime = $(Get-Date).AddHours(-10)"
```

### Covering Tracks on OS (Linux/Unix)
**Ocultar ficheros en Linux:** añadir un punto (`.`) al inicio del nombre de fichero lo hace invisible para `ls` sin el flag `-a`. El atacante también puede usar nombres como `. ` (punto + espacio) que imitan los directorios del sistema (`.` y `..`). Ubicaciones frecuentes: `/dev`, `/tmp`, `/etc`.

**Modificar timestamps en Linux:**
```bash
# Cambiar timestamp de último acceso
touch -a -d '<date> <time>' $File_name

# Cambiar timestamp de última modificación
touch -m -d '<date> <time>' $File_name
```

### Delete Files Using Cipher.exe 🔴
`Cipher.exe` es una herramienta nativa de Windows para cifrado/descifrado en particiones NTFS, pero también permite **sobreescribir de forma segura** ficheros eliminados para evitar su recuperación. El proceso sobreescribe tres veces: primero con todos ceros (0x00), luego con todos unos (0xFF), y finalmente con números aleatorios.

```cmd
# Sobreescribir ficheros eliminados en una carpeta específica
cipher /w:C:\NombreCarpeta

# Sobreescribir todos los ficheros eliminados en una unidad
cipher /w:C:
```

Cuando el atacante cifra un fichero malicioso, se crea un backup temporal. Al terminar el cifrado ese backup se "elimina", pero puede recuperarse con herramientas forenses. `Cipher.exe /w` sobreescribe esos ficheros eliminados haciéndolos irrecuperables.

### Disable Windows Functionality 🔴

**Last Access Timestamp:** cada vez que se accede a un fichero, el timestamp se actualiza. El atacante desactiva esto para no dejar rastro de qué ficheros consultó.
```cmd
# Deshabilitar last access timestamp
fsutil behavior set disablelastaccess 1

# Habilitar (valor por defecto)
fsutil behavior set disablelastaccess 0
```

**Windows Hibernation:** el fichero `Hiberfil.sys` contiene el contenido de la RAM del sistema en el momento de hibernar. Para los investigadores es una fuente de información valiosa.
```cmd
# Deshabilitar hibernación via Command Prompt
powercfg.exe /hibernate off
```
También via registro: `HKLM\SYSTEM\CurrentControlSet\Control\Power` → `HibernateEnabledDefault` → valor `0`.

**Virtual Memory / Paging File:** el fichero de paginación puede contener contraseñas en texto claro y ficheros descifrados temporalmente. El atacante lo deshabilita para que esa información no quede en disco. Se desactiva en: Control Panel → System → Advanced → Performance → Virtual Memory → No paging file.

**System Restore Points:** los puntos de restauración pueden contener copias de ficheros eliminados. El atacante los elimina en: Control Panel → System → System Protection → Configure → Disable system protection + Delete.

**Thumbnail Cache (thumbs.db):** Windows almacena miniaturas de imágenes y documentos en `thumbs.db`. Si el atacante usó una imagen para ocultar un fichero y la borró, la miniatura puede revelar que existió. Se deshabilita via Group Policy: `gpedit.msc` → User Configuration → Administrative Templates → Windows Components → File Explorer → "Turn off the caching of thumbnails in hidden thumbs.db files" → Enabled.

**Prefetch:** Windows almacena datos de aplicaciones usadas frecuentemente para acelerar su carga. Si el atacante instaló y desinstalió una herramienta maliciosa, el Prefetch puede revelar que existió. Se deshabilita desactivando el servicio `SysMain (Superfetch)`.

### Delete Windows Activity History
Windows Activity History registra ficheros accedidos, aplicaciones usadas y historial de navegación. El atacante la borra en: Settings → Privacy & security → Activity history → Clear.

### Delete Incognito History
El modo incógnito no guarda historial local, pero las entradas DNS pueden revelar qué dominios se visitaron.

```cmd
# Windows: ver cache DNS (incluye visitas en modo incógnito)
ipconfig /displaydns

# Limpiar cache DNS
ipconfig /flushdns
```

```bash
# macOS: limpiar DNS cache
sudo killall -INFO mDNSResponder
```

---

## 3. Hiding Artifacts

### Windows
```cmd
# Ocultar fichero/carpeta (hidden + system + read-only)
attrib +h +s +r <FolderName>

# Crear usuario oculto
net user <UserName> /add
net user <UserName> /active:yes   # activar
net user <UserName> /active:no    # desactivar (ocultar)
```
Ocultar cuenta via registro: `HKLM\Software\Microsoft\WindowsNT\CurrentVersion\Winlogon` → crear clave con el nombre de usuario.

### Linux
```bash
# Ocultar fichero añadiendo punto al nombre
mv MaliciousFile.txt .MaliciousFile.txt

# Ver ficheros ocultos
ls -a

# Crear carpeta oculta
mkdir .HiddenFolder

# Crear fichero oculto en carpeta oculta
touch .HiddenFolder/.MaliciousFile.txt
```

### macOS
```bash
# Ocultar todos los ficheros del Finder
defaults write com.apple.finder AppleShowAllFiles FALSE && killall Finder

# Ocultar fichero específico
chflags hidden <filename>
```

---

## 4. Anti-Forensics Techniques

El anti-forensics es el conjunto de técnicas para **ocultar, alterar o destruir** evidencia de actividad maliciosa. Cada técnica ataca una fuente diferente de evidencia forense:

**Data/File Deletion:** eliminar el puntero al fichero. En Windows, los ficheros van a la Papelera de Reciclaje con delete normal. Con Shift+Delete se eliminan sin pasar por la Papelera pero son recuperables. El atacante oculta o borra los metadatos de la Papelera para evitar la recuperación.

**Password Protection:** proteger ficheros con contraseña o cifrado para impedir acceso por herramientas de recuperación. También dificulta el reverse engineering de aplicaciones.

**Steganography:** ocultar información cuando el cifrado no es viable o cuando la mera existencia del cifrado levaría sospechas. (Ya detallado en chunk 3.)

**Data Hiding in File System Structures:** usar zonas del disco normalmente no visibles:
- `$BadClus`: fichero sparse de NTFS que puede usarse para ocultar datos asignándole más clusters.
- **HPA (Host-Protected Area):** área reservada en discos duros originalmente para fabricantes. El atacante puede almacenar datos aquí; ni el BIOS ni el OS los ven sin herramientas especializadas.
- **DPA (Device Configuration Overlay)** y **slack spaces** son otros espacios para ocultar datos.

**Trail Obfuscation:** eliminar y falsificar rastros de actividad. Incluye: log tampering, false email header generation, timestamp modifications, file-header modifications. Herramientas: `Timestomp` y `Transmogrify` para modificar/borrar metadatos de fecha y hora. Otras técnicas: log cleaners, zombie accounts, spoofing, Trojan commands, misinformation.

**Artifact Wiping:** destrucción permanente de evidencia usando utilidades de borrado seguro de disco. La diferencia con la simple eliminación es que el espacio se sobreescribe para hacerlo irrecuperable. Herramientas: BCWipe, DriveScrubber, KillDisk, BleachBit, Blancco File Eraser, DBAN.

**Overwriting Data/Metadata:** sobreescribir todas las ubicaciones direccionables del medio de almacenamiento con caracteres aleatorios. Métodos: simple deletion, data shredding, data wiping (múltiples sobrescrituras). Tras múltiples pasadas de sobrescritura, la recuperación forense es prácticamente imposible.

**Program Packers:** comprimen/cifran ejecutables para ocultar herramientas de ataque dentro de contenedores. Dificultan el reverse engineering y el análisis forense. Si están protegidos con contraseña, los investigadores deben descifrar la contraseña antes de poder analizar el contenido. Ejemplos: UPX, PECompact, BurnEye, Exe Stealth Packer, Smart Packer Pro.

**Minimizing Footprints:** realizar el ataque sin dejar rastro desde el inicio: identidades robadas, máquinas virtuales, infraestructura cloud, criptomonedas no trazables, OSes ejecutados desde Live USB o disco externo.

**Access Anonymization:** ocultar la identidad del atacante durante el acceso. Técnicas: proxy servers, Tor networks, anonymization services, traffic padding, anonymous communication channels.

---

## 5. Track-Covering Tools

Herramientas que automatizan la limpieza de evidencia:
- **CCleaner:** limpieza de sistema, caché, cookies, historial de Internet, logs.
- **DBAN:** borrado seguro de discos completos.
- **BleachBit:** limpieza de archivos temporales y rastros de actividad.
- **Privacy Eraser Free, Wipe, east-tec Eraser:** variantes de limpieza de privacidad.

---

## 3. Exam Traps ⚠️

⚠️ **[auditpol: deshabilitar Y rehabilitar]** — El examen puede preguntar qué hace un atacante sofisticado tras deshabilitar la auditoría. Además de deshabilitarla antes de operar, **la vuelve a habilitar al salir** para que no sea evidente que estuvo desactivada.

⚠️ **[clearev en Meterpreter]** — El examen puede pedir el comando de Meterpreter para borrar todos los logs del sistema Windows. Es simplemente `clearev`. Sin parámetros adicionales.

⚠️ **[wevtutil cl vs wevtutil el]** — `wevtutil el` **lista** (enumerate logs) los logs disponibles. `wevtutil cl` **limpia** (clear log) un log específico. El examen puede confundirlos.

⚠️ **[HISTSIZE=0 vs history -c]** — `export HISTSIZE=0` **deshabilita** el guardado de historial futuro (no borra el existente). `history -c` **borra** el historial de la sesión actual. Para eliminar completamente todo: `cat /dev/null > ~/.bash_history && history -c && exit`.

⚠️ **[shred vs rm]** — `rm` elimina el fichero pero el contenido es recuperable. `shred` sobreescribe el contenido del fichero haciéndolo ilegible incluso si se recupera. El examen puede preguntar cuál deja la evidencia irrecuperable.

⚠️ **[Cipher.exe: 3 pasadas de sobrescritura]** — Cipher.exe sobrescribe primero con 0x00, luego con 0xFF, y finalmente con números aleatorios. El examen puede preguntar el número o el orden de las pasadas.

⚠️ **[fsutil disablelastaccess]** — El valor `1` **deshabilita** el last access timestamp. El valor `0` lo **habilita**. El examen puede invertirlos o preguntar el comando exacto.

⚠️ **[Hiberfil.sys]** — El archivo de hibernación está en el directorio raíz de la unidad donde está el OS. Contiene el contenido de la RAM en el momento de hibernar. El examen puede preguntar dónde está o qué contiene.

⚠️ **[Reverse HTTP Shell: quién es cliente y quién es servidor]** — El examen puede confundir los roles. La **víctima** actúa como cliente web (hace HTTP GET al servidor del atacante). El **atacante** actúa como servidor web y responde con comandos. Es lo inverso de la percepción habitual.

⚠️ **[ICMP tunneling: qué inspecciona el firewall]** — La mayoría de firewalls inspeccionan los ICMP **entrantes** pero no los **salientes**. El atacante explota esto encapsulando datos en paquetes ICMP de salida.

⚠️ **[DNS Tunneling vs DNS Poisoning]** — DNS Tunneling usa queries/respuestas DNS como canal encubierto de comunicación/exfiltración. DNS Poisoning modifica la resolución de nombres para redirigir tráfico. Son técnicas completamente distintas.

⚠️ **[TCP ISN Covert Channel]** — El TCP Initial Sequence Number (ISN) como canal oculto no requiere una conexión establecida. Un carácter se encapsula por SYN/RST. El examen puede preguntar cuál de los canales TCP no necesita conexión previa.

⚠️ **[$BadClus]** — Es un fichero sparse de NTFS normalmente usado para marcar sectores defectuosos. El atacante puede asignarle más clusters para ocultar datos arbitrariamente. El examen puede presentarlo como herramienta de anti-forensics.

⚠️ **[HPA vs DPA vs Slack Space]** — HPA (Host-Protected Area): área del disco invisible al BIOS y OS, originalmente para fabricantes. DPA (Device Configuration Overlay): similar. Slack Space: espacio no usado entre el final del fichero y el final del cluster. Todos son zonas de almacenamiento "invisible" para herramientas normales.

⚠️ **[Program Packers]** — Los packers comprimen/cifran ejecutables. Su propósito en anti-forensics es **ocultar herramientas** dentro de contenedores para dificultar el análisis forense y el reverse engineering, no para ofuscación de código directamente (eso es encoders).

⚠️ **[ipconfig /displaydns vs /flushdns]** — `/displaydns` muestra el cache DNS (revela dominios visitados incluso en incógnito). `/flushdns` lo borra. El examen puede preguntar cómo revelar o borrar el rastro de navegación incógnita en Windows.

⚠️ **[thumbs.db]** — El thumbnail cache puede revelar que existió un fichero aunque haya sido borrado. El atacante debe desactivarlo via Group Policy para evitar que registre miniaturas de sus herramientas.

---

## 4. Nemotécnicos

**Técnicas de covering tracks (8):**
> **"Dis Clear Manip Network OS Delete Disable Hide"**
> **Dis**abling auditing → **Clear** logs → **Manip**ulating logs → **Network** coverage → **OS** coverage → **Delete** files → **Disable** Windows features → **Hide** artifacts

**Comandos BASH para cubrir historial (4 niveles de destrucción):**
> **"HISTSIZE → history-c → /dev/null → shred"**
> Nivel 1 (prevenir): `HISTSIZE=0`
> Nivel 2 (limpiar sesión): `history -c`
> Nivel 3 (borrar fichero): `cat /dev/null > ~/.bash_history`
> Nivel 4 (destruir contenido): `shred ~/.bash_history`

**Anti-forensics técnicas (8):**
> **"Data Pass Stego DataHide Trail Artifact Overwrite Pack Mini Anon"**
> Data deletion → Password protection → Steganography → Data hiding → Trail obfuscation → Artifact wiping → Overwriting → Packers → Minimizing footprints → Access anonymization

**Windows functionality a deshabilitar (6):**
> **"Last Hiber Paging Restore Thumb Prefetch"**
> Last access timestamp → Hibernation → Paging file → Restore points → Thumbnail cache → Prefetch

**Cipher.exe: 3 pasadas:**
> **"Cero FF Random"** → 0x00 → 0xFF → números aleatorios

---

## 5. Flashcards

**Q:** ¿Qué herramienta nativa de Windows deshabilita la auditoría de seguridad?
**A:** `auditpol.exe`. Comando: `auditpol /set /category:"system","account logon" /success:disable /failure:disable`

**Q:** ¿Qué comando de Meterpreter borra todos los logs del sistema Windows?
**A:** `clearev`

**Q:** ¿Qué comando de PowerShell borra un event log específico?
**A:** `Clear-EventLog "Windows PowerShell"`

**Q:** ¿Qué utilidad nativa de Windows lista y borra event logs desde CMD?
**A:** `wevtutil`. `wevtutil el` para listar, `wevtutil cl <log>` para borrar.

**Q:** ¿Qué comando de Bash deshabilita el guardado de historial de forma permanente en la sesión?
**A:** `export HISTSIZE=0`

**Q:** ¿Qué comando borra el historial de todas las shells y sale?
**A:** `cat /dev/null > ~/.bash_history && history -c && exit`

**Q:** ¿Qué hace `shred ~/.bash_history`?
**A:** Sobreescribe el contenido del fichero de historial haciéndolo ilegible, aunque se localice el fichero.

**Q:** ¿Qué hace Cipher.exe con `cipher /w:C:`?
**A:** Sobreescribe todos los ficheros eliminados en la unidad C: con 0x00, 0xFF y números aleatorios para hacerlos irrecuperables.

**Q:** ¿Qué comando deshabilita el last access timestamp en Windows?
**A:** `fsutil behavior set disablelastaccess 1`

**Q:** ¿Dónde se almacena el fichero de hibernación de Windows y qué contiene?
**A:** En el directorio raíz de la unidad del OS (`Hiberfil.sys`). Contiene el contenido de la RAM en el momento de hibernar.

**Q:** ¿Cómo deshabilita el atacante la hibernación de Windows por línea de comandos?
**A:** `powercfg.exe /hibernate off`

**Q:** ¿Qué revela el thumbnail cache (thumbs.db) al investigador forense?
**A:** Miniaturas de ficheros que existieron en el sistema aunque hayan sido borrados, incluyendo imágenes usadas como containers de steganografía.

**Q:** ¿Cuáles son las 3 pasadas de sobrescritura que hace Cipher.exe?
**A:** Primera con todos ceros (0x00), segunda con todos unos (0xFF), tercera con números aleatorios.

**Q:** En un reverse HTTP shell, ¿quién actúa como cliente y quién como servidor?
**A:** La víctima actúa como cliente web (hace HTTP GET). El atacante actúa como servidor web (responde con comandos).

**Q:** ¿Por qué ICMP tunneling es efectivo para bypasear firewalls?
**A:** La mayoría de firewalls inspeccionan ICMP entrante pero no saliente. El atacante encapsula datos en paquetes ICMP de salida.

**Q:** ¿Qué campo TCP puede usarse como canal oculto sin necesitar una conexión establecida?
**A:** TCP Initial Sequence Number (ISN). Un carácter por SYN/RST.

**Q:** ¿Qué es $BadClus y cómo lo usan los atacantes?
**A:** Fichero sparse de NTFS para marcar sectores defectuosos. El atacante le asigna más clusters para almacenar datos ocultos de forma ilimitada.

**Q:** ¿Qué es el HPA y por qué es útil para ocultar datos?
**A:** Host-Protected Area: zona del disco duro invisible al BIOS y OS, originalmente para fabricantes. Almacena datos no visibles sin herramientas especializadas.

**Q:** ¿Qué hace `ipconfig /flushdns` en el contexto de covering tracks?
**A:** Borra el cache DNS, eliminando el registro de dominios visitados (incluyendo en modo incógnito).

**Q:** ¿Para qué sirven los program packers en anti-forensics?
**A:** Comprimen/cifran ejecutables para ocultar herramientas de ataque dentro de contenedores, dificultando el análisis forense y el reverse engineering.

**Q:** ¿Qué comando oculta un fichero en Linux?
**A:** Renombrarlo añadiendo un punto al inicio: `mv MaliciousFile.txt .MaliciousFile.txt`

**Q:** ¿Qué comando oculta una carpeta completa con sus atributos en Windows?
**A:** `attrib +h +s +r <FolderName>`

**Q:** ¿Qué logs de Windows son los más importantes para el atacante borrar?
**A:** SECEVENT.EVT (security), SYSEVENT.EVT (system), APPEVENT.EVT (application).

---

## 6. Confusión frecuente

**Disabling Auditing vs Clearing Logs** → Deshabilitar auditoría evita que se generen nuevos registros. Borrar logs elimina los registros ya generados. El atacante hace ambas cosas: deshabilita primero, opera, y borra después.

**history -c vs HISTSIZE=0** → `HISTSIZE=0` previene el guardado futuro pero no borra el historial existente. `history -c` borra el historial de la sesión actual. Para eliminar todo: combinar con `cat /dev/null > ~/.bash_history`.

**shred vs rm** → `rm` elimina el fichero (recuperable). `shred` sobreescribe el contenido del fichero antes de (opcionalmente) eliminarlo, haciéndolo irrecuperable.

**Cipher.exe cifrado vs Cipher.exe wiping** → Cipher.exe `/w` no cifra: **sobreescribe espacio libre** para evitar recuperación de ficheros eliminados. Cipher sin `/w` cifra ficheros existentes. Son funciones distintas del mismo comando.

**wevtutil el vs wevtutil cl** → `el` = enumerate logs (listar). `cl` = clear log (borrar). El examen puede poner el parámetro erróneo.

**Reverse HTTP Shell vs DNS Tunneling** → Ambos son técnicas de covert channel, pero usan protocolos distintos. HTTP Shell: usa HTTP GET/response para comandos y resultados. DNS Tunneling: codifica datos en queries/respuestas DNS. Ambos evaden firewalls pero de forma diferente.

**ICMP Tunneling vs DNS Tunneling** → ICMP: encapsula TCP payload en paquetes ICMP echo/reply. DNS: codifica datos en queries/respuestas DNS. Ambos usan protocolos legítimos como carriers.

**Trail Obfuscation vs Artifact Wiping** → Trail Obfuscation modifica/falsifica rastros (timestamps, logs) pero los deja en el sistema alterados. Artifact Wiping destruye permanentemente los datos sin dejar rastro. Obfuscation engaña; Wiping destruye.

**$BadClus vs HPA vs Slack Space** → $BadClus: fichero NTFS, visible con herramientas NTFS pero con datos ocultos en clusters marcados como defectuosos. HPA: zona del disco invisible al OS/BIOS, solo con herramientas especializadas. Slack Space: espacio no usado entre final del fichero y final del cluster, no visible con herramientas normales.

**Prefetch vs Thumbnail Cache** → Prefetch: almacena datos de arranque de aplicaciones para acelerar su carga. Revela qué aplicaciones se ejecutaron. Thumbnail Cache (thumbs.db): almacena miniaturas de imágenes/documentos. Revela qué ficheros existieron visualmente. Ambos pueden revelar actividad pasada del atacante.

**Last Access Timestamp vs Modified Timestamp** → Last Access: se actualiza cada vez que se lee el fichero. Modified: se actualiza cuando se cambia el contenido. El atacante deshabilita el Last Access con `fsutil` y puede falsificar el Modified con `timestomp` o `touch -m`.

---

## 6. Preguntas de Práctica — Formato CEH

---

### Pregunta 1 — Auditpol y Covering Tracks
Un red teamer con privilegios de administrador en Windows quiere deshabilitar el registro de eventos de seguridad antes de realizar operaciones de post-explotación. ¿Cuál es el comando correcto y qué hace el atacante inteligente al terminar?

A) `eventvwr /clear` — no necesita rehabilitarlo  
B) `auditpol /set /category:"system","account logon" /success:disable /failure:disable` — luego lo rehabilita al salir  
C) `wevtutil cl security` — luego borra el historial de comandos  
D) `Clear-EventLog "Security"` — no necesita rehabilitarlo  

> **Respuesta correcta: B** — El atacante sofisticado usa `auditpol` para deshabilitar la auditoría **antes de operar** y la **vuelve a habilitar al salir** (`/success:enable /failure:enable`). Esto evita que la desactivación en sí misma quede registrada como evidencia. Simplemente borrar logs es más detectable que nunca generarlos.

---

### Pregunta 2 — Wevtutil
Un administrador quiere usar la herramienta nativa de Windows `wevtutil` para borrar los logs de Application y Security. ¿Cuál es el comando correcto?

A) `wevtutil el application security`  
B) `wevtutil del application && wevtutil del security`  
C) `wevtutil cl application` y `wevtutil cl security`  
D) `wevtutil clear-log application security`  

> **Respuesta correcta: C** — `wevtutil cl <log>` = **clear log** (borrar un log específico). `wevtutil el` = **enumerate logs** (listar los logs disponibles). Deben ejecutarse por separado para cada log. No existe el subcomando `del` ni `clear-log` en wevtutil.

---

### Pregunta 3 — Bash History
Un atacante Linux quiere asegurarse de que ningún comando de su sesión actual sea guardado en el historial de Bash. Está a punto de comenzar sus operaciones. ¿Cuál es la acción más preventiva antes de empezar?

A) `history -c` al final de la sesión  
B) `cat /dev/null > ~/.bash_history` antes de empezar  
C) `export HISTSIZE=0` antes de empezar  
D) `shred ~/.bash_history` al terminar  

> **Respuesta correcta: C** — `export HISTSIZE=0` **previene que se guarden nuevos comandos** en el historial de forma prospectiva (hacia el futuro). Debe ejecutarse antes de comenzar las operaciones. `history -c` borra el historial de la sesión actual (retrospectivo). `cat /dev/null >` borra el fichero pero no previene escrituras futuras en la sesión. `shred` destruye el fichero existente.

---

### Pregunta 4 — Cipher.exe
Un administrador de seguridad forense intenta recuperar un ejecutable malicioso que el atacante eliminó de un sistema Windows NTFS. El atacante usó `cipher /w:C:\Tools\` antes de salir. ¿Qué encontrará el investigador?

A) El fichero en la Papelera de Reciclaje con los datos intactos  
B) El fichero recuperable con herramientas forenses estándar como Recuva  
C) El espacio del fichero sobreescrito tres veces (0x00, 0xFF, aleatorio), haciendo el fichero irrecuperable  
D) El fichero cifrado con AES-256, recuperable con la clave correcta  

> **Respuesta correcta: C** — `cipher /w:` no cifra el fichero sino que **sobreescribe el espacio libre** del directorio especificado con tres pasadas: primero con todos ceros (0x00), luego con todos unos (0xFF), y finalmente con números aleatorios. Después de tres pasadas de sobrescritura, la recuperación forense es prácticamente imposible.

---

### Pregunta 5 — DNS Tunneling
Un analista de seguridad revisa el tráfico de red y nota que un host interno está generando un gran número de queries DNS con nombres de dominio inusualmente largos y con alto contenido de caracteres aleatorios en los subdominios (ej. `7a4b9c3d.8e2f1a0b.evil.com`). ¿Qué técnica de covert channel está detectando?

A) DNS Cache Poisoning  
B) DNS Tunneling  
C) ICMP Tunneling  
D) HTTP Reverse Shell  

> **Respuesta correcta: B** — **DNS Tunneling** codifica datos en queries y respuestas DNS, usando los subdominios como carrier de datos. Los nombres de dominio largos con caracteres aparentemente aleatorios son una señal indicativa. DNS raramente es inspeccionado en profundidad por los firewalls, lo que lo convierte en un canal de exfiltración efectivo.

---

### Pregunta 6 — Reverse HTTP Shell
Un atacante ha comprometido un sistema dentro de una red corporativa con firewall estricto. El firewall bloquea conexiones entrantes pero permite tráfico HTTP saliente. ¿Qué técnica de covering tracks en red debe usar y cuál es el rol de cada parte?

A) DNS Tunneling; el sistema comprometido actúa como servidor DNS  
B) Reverse HTTP Shell; la víctima actúa como cliente HTTP haciendo GET al servidor del atacante que actúa como servidor web  
C) ICMP Tunneling; el atacante envía comandos en paquetes ICMP echo request  
D) Reverse HTTP Shell; el atacante actúa como cliente HTTP enviando comandos al servidor de la víctima  

> **Respuesta correcta: B** — En un **Reverse HTTP Shell**, la **víctima actúa como cliente web** (hace HTTP GET periódico al servidor del atacante). El **atacante actúa como servidor web** respondiendo con comandos. El tráfico parece HTTP normal saliente, lo que el firewall permite. El atacante responde "como servidor web" con comandos que la víctima ejecuta.

---

### Pregunta 7 — Hiberfil.sys
Un investigador forense obtiene un portátil Windows que estaba en modo hibernación cuando fue incautado. Está buscando evidencias de actividad maliciosa. ¿Qué fichero debe analizar primero y qué contiene?

A) `pagefile.sys` — contiene fragmentos de ficheros abiertos recientemente  
B) `Hiberfil.sys` — contiene el contenido completo de la RAM en el momento de hibernar  
C) `NTUSER.DAT` — contiene las preferencias del usuario y el historial de actividad  
D) `SAM` — contiene los hashes de contraseñas del sistema  

> **Respuesta correcta: B** — `Hiberfil.sys` está en el directorio raíz de la unidad del OS y contiene el **contenido completo de la RAM** en el momento en que el sistema entró en hibernación. Para el atacante, este fichero puede contener contraseñas en texto claro, claves de cifrado activas, y evidencias de actividad. Se deshabilita con `powercfg.exe /hibernate off`.

---

### Pregunta 8 — Covert Channels TCP
Un atacante quiere exfiltrar datos de una red sin establecer una conexión TCP completa y sin usar protocolos de nivel de aplicación. Usa el campo de número de secuencia inicial (ISN) de los paquetes SYN/RST para codificar caracteres. ¿Qué propiedad hace este canal especialmente útil?

A) Está cifrado automáticamente por TCP/IP  
B) No requiere una conexión TCP establecida para funcionar  
C) Usa HTTP como protocolo de transporte subyacente  
D) Es invisible para todos los tipos de IDS  

> **Respuesta correcta: B** — El canal oculto del **TCP Initial Sequence Number (ISN)** usa los campos ISN de paquetes **SYN/RST**, que no requieren establecer una conexión TCP completa (3-way handshake). Un carácter se codifica por paquete SYN o RST. Esto lo hace especialmente difícil de detectar porque los paquetes SYN son comunes y no levantan sospechas.

---

### Pregunta 9 — Anti-Forensics: HPA vs Slack Space
Un investigador forense avanzado examina un disco duro de un sospechoso. Herramientas forenses estándar como Autopsy no muestran ningún fichero sospechoso. Sin embargo, herramientas especializadas de bajo nivel revelan datos en un área del disco invisible al BIOS y al sistema operativo, originalmente reservada por el fabricante. ¿Qué área de almacenamiento oculto está usando el atacante?

A) Slack Space entre el final del fichero y el final del cluster  
B) NTFS Alternate Data Streams ($BadClus)  
C) Host-Protected Area (HPA)  
D) Device Configuration Overlay (DPA)  

> **Respuesta correcta: C** — La **Host-Protected Area (HPA)** es un área del disco duro invisible al BIOS y al OS sin herramientas especializadas, originalmente reservada por fabricantes para diagnósticos. El atacante puede almacenar datos arbitrarios aquí. Requiere herramientas ATA especializadas para detectarla. $BadClus es un fichero NTFS. Slack space es el espacio entre el final del fichero y el final del cluster.

---

### Pregunta 10 — Last Access Timestamp
Un atacante en un sistema Windows accede a múltiples ficheros sensibles del servidor. Quiere que el investigador forense no pueda determinar qué ficheros fueron accedidos durante el ataque. ¿Qué comando debe ejecutar antes de acceder a los ficheros?

A) `timestomp file.doc -z "01/01/2020 00:00:00"`  
B) `fsutil behavior set disablelastaccess 1`  
C) `attrib -a file.doc`  
D) `cipher /w:C:\Documents\`  

> **Respuesta correcta: B** — `fsutil behavior set disablelastaccess 1` **deshabilita la actualización del timestamp de último acceso** (Last Access Time) en todo el sistema. Con esto, el acceso a los ficheros no deja rastro temporal. `timestomp` modifica timestamps existentes. `attrib` modifica atributos de fichero. `cipher /w` sobreescribe espacio libre.

---

### Pregunta 11 — Diferencia shred vs rm
Un atacante Linux quiere eliminar sus herramientas de ataque del sistema de forma que sean irrecuperables para el investigador forense. ¿Cuál es la diferencia entre `rm` y `shred` en este contexto?

A) `rm` y `shred` son equivalentes en NTFS; en ext4 `shred` es más seguro  
B) `rm` elimina el puntero al fichero pero el contenido permanece en disco y es recuperable; `shred` sobreescribe el contenido antes de eliminar  
C) `shred` solo funciona en ficheros de texto; `rm` elimina cualquier tipo de fichero  
D) Ambos son irrecuperables en sistemas de ficheros modernos como ext4  

> **Respuesta correcta: B** — `rm` elimina el **puntero (inode)** al fichero pero el contenido sigue en los sectores del disco hasta ser sobreescrito por nuevos datos. Herramientas forenses como Recuva pueden recuperarlo. `shred` sobreescribe el **contenido del fichero** múltiples veces antes de eliminarlo, haciendo la recuperación prácticamente imposible.

---

### Pregunta 12 — Prefetch y Thumbnail Cache
Un investigador forense examina un sistema Windows. El atacante instaló herramientas de ataque, las usó, y luego las desinstalió y borró. ¿Qué dos fuentes de evidencia pueden revelar que dichas herramientas existieron en el sistema, incluso después de su eliminación?

A) Registry Run Keys y Event Logs de Security  
B) Prefetch (SysMain/Superfetch) y Thumbnail Cache (thumbs.db)  
C) hiberfil.sys y pagefile.sys  
D) NTFS ADS y Slack Space  

> **Respuesta correcta: B** — El **Prefetch** almacena datos de aplicaciones ejecutadas (qué herramientas se lanzaron y cuándo). El **Thumbnail Cache (thumbs.db)** guarda miniaturas de ficheros que existieron (útil si el atacante usó imágenes para steganografía). Ambos pueden revelar actividad pasada incluso si los ficheros originales han sido eliminados. El atacante debe deshabilitar ambos para no dejar rastro.

