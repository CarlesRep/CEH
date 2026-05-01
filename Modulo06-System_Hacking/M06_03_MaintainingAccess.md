# M06 — Gaining Access
### Parte 3: Maintaining Access

---

## 1. Concepto de Maintaining Access

Tras ganar acceso y escalar privilegios, el atacante convierte el sistema comprometido en una **base de operaciones permanente**: instala herramientas maliciosas, oculta su presencia, y mantiene acceso incluso tras reinicios o cambios de contraseña. Los mecanismos principales son: ejecución remota de código, keyloggers, spyware, rootkits, NTFS ADS, steganografía, y técnicas de persistencia específicas de dominio AD.

---

## 2. Ejecución de Aplicaciones

Una vez con privilegios de administrador, el atacante ejecuta aplicaciones maliciosas para "adueñarse" del sistema (**owning**). Los tipos de programas instalados son:

- **Backdoors:** permiten acceso futuro sin autenticación normal.
- **Crackers:** rompen contraseñas o cifrado.
- **Keyloggers:** registran pulsaciones de teclado.
- **Spyware:** monitoriza y exfiltra información del usuario.

### Técnicas de Remote Code Execution (RCE)

**Web-Browser-Based Exploitation:** mediante spear phishing o drive-by compromise. No requiere interacción del usuario para ejecutarse.

**Office-Applications-Based Exploitation:** documentos maliciosos enviados por email que el usuario debe abrir (Word, Excel, etc.).

**Third-Party Applications Exploitation:** Adobe Reader, Flash y similares como vector de acceso.

**Service Execution:** crear o modificar servicios Windows para ejecutar binarios maliciosos a través del Service Control Manager.

**WMI (Windows Management Instrumentation):** permite gestión local y remota del sistema. Los atacantes lo abusan para movimiento lateral, ejecutar código, y establecer persistencia. Acceso remoto via **DCOM** (puerto 135) y **WinRM** (HTTP 5985 / HTTPS 5986).

**WinRM (Windows Remote Management):** protocolo Windows para ejecución remota. El atacante usa `winrm` para ejecutar payloads en sistemas remotos como parte del movimiento lateral.

---

## 3. Keyloggers

Un keylogger registra todas las pulsaciones del teclado del usuario sin su conocimiento. Captura contraseñas, números de tarjeta, conversaciones de chat, emails, etc. **La clave:** captura la información antes de que sea cifrada, por lo que ni HTTPS protege contra un keylogger bien instalado.

### Hardware Keyloggers
Dispositivos físicos que se conectan entre el teclado y el ordenador. El OS no los detecta porque operan a nivel hardware. Su desventaja: detección física visible.

Tipos principales:
- **PC/BIOS Embedded:** firmware modificado que captura pulsaciones antes de que el OS las reciba. Requiere acceso físico o privilegios de administrador.
- **Keylogger Keyboard:** circuito hardware en el cable del teclado. Almacena en memoria interna. **No depende del OS** y es imposible de detectar con software anti-keylogger.
- **External (PS/2 y USB):** se conecta en línea entre teclado y PC. Sin software ni drivers.
- **Acoustic/CAM:** convierte las ondas sonoras electromagnéticas de las teclas en datos, o usa cámara para grabar el teclado.
- **Bluetooth:** requiere acceso físico solo en la instalación; luego transmite vía Bluetooth.
- **Wi-Fi:** se conecta a un punto de acceso Wi-Fi local y envía emails con los keystrokes capturados. Accesible también via TCP/IP.

### Software Keyloggers
Instalados remotamente via red o adjunto de email. Tipos:

- **Application Keylogger:** registra todo lo que el usuario escribe en aplicaciones, emails y chats. Funciona en modo invisible.
- **Kernel/Rootkit/Device Driver:** opera a nivel de kernel (Ring 0). Muy difícil de detectar. Actúa como driver de dispositivo de teclado. El rootkit-based keylogger es un driver Windows falsificado, completamente indetectable incluso con herramientas dedicadas.
- **Hypervisor-Based:** opera dentro de un hypervisor malicioso que corre bajo el OS.
- **Form-Grabbing-Based:** captura datos de formularios web al hacer submit, **bypassando HTTPS**. Registra en el evento `submit`.
- **JavaScript-Based:** inyecta código JavaScript malicioso en páginas comprometidas para escuchar eventos `onKeyUp()` y `onKeyDown()`. Vectores: MitB (manipulator-in-the-browser), XSS.
- **Memory-Injection-Based:** modifica tablas de memoria del navegador y funciones del sistema. También puede bypassar UAC en Windows.

### Keylogger con Metasploit (Meterpreter)
Desde una sesión Meterpreter en el sistema comprometido:
```
getpid                    # Ver el PID actual del proceso
migrate <PID>             # Migrar al proceso explorer.exe para estabilidad
keyscan_start             # Iniciar captura de keystrokes
keyscan_dump              # Volcar los keystrokes capturados
keyscan_stop              # Detener la captura
```
También existe el módulo `lockout_keylogger` para automatizar todo el proceso.

---

## 4. Spyware

El spyware monitoriza **todas las actividades del sistema** del usuario y las transmite al atacante via Internet (email, FTP, HTTP, DNS, tráfico cifrado C2). Se instala sin conocimiento del usuario mediante:
- **Piggybacking:** se incluye como componente oculto de software gratuito.
- **Drive-by download:** se instala automáticamente al visitar y hacer clic en una web de distribución de spyware.
- **Bundled software:** incluido en el instalador de programas aparentemente legítimos.

### Capacidades del spyware:
Roba información personal y la envía a servidor remoto, monitoriza actividad online, captura screenshots, activa micrófono y cámara web, modifica configuración del navegador, cambia la página de inicio, modifica DLLs del sistema, desactiva antivirus y firewall, distribuye spam, y puede **controlar remotamente el sistema comprometido**.

### Tipos de Spyware
- **Desktop Spyware:** monitorea actividad en el escritorio, graba audio/vídeo, modifica configuración del sistema.
- **Email Spyware:** captura y reenvía todos los emails entrantes y salientes. Opera en stealth mode.
- **Internet Spyware:** registra todas las URLs visitadas. Carga al inicio del sistema.
- **Child-Monitoring Spyware:** diseñado para parental monitoring; también usado maliciosamente.
- **Screen-Capturing Spyware:** toma screenshots a intervalos definidos y los envía por email o FTP.
- **USB Spyware:** copia automáticamente spyware desde un USB al disco duro al conectarlo. Opera en modo oculto.
- **Audio Spyware:** graba sonido del sistema, monitoriza llamadas y conferencias. **No requiere privilegios de administrador**.
- **Video Spyware:** graba webcams y conversaciones de vídeo. Acceso remoto para ver imágenes en vivo.
- **Print Spyware:** monitoriza la actividad de impresora: documentos impresos, número de copias, contenido. Guarda logs cifrados.
- **Telephone/Cellphone Spyware:** acceso completo al teléfono: historial de llamadas, SMS (incluso borrados), historial web, GPS. Se oculta completamente del usuario.
- **GPS Spyware:** rastrea ubicación en tiempo real, almacena historial en log y envía alertas de proximidad.

---

## 5. Rootkits 🔴

Un rootkit es malware diseñado para **obtener y mantener acceso root sin ser detectado**. Reemplaza funciones del OS por versiones modificadas, construye backdoors en el proceso de login, y oculta procesos, ficheros, y puertos. Un rootkit típico incluye: backdoors, programas DDoS, sniffers de paquetes, utilidades de borrado de logs, e IRC bots.

### Tipos de Rootkits (6 tipos) 🔴

**Hypervisor-Level Rootkit:** explota características hardware (Intel VT, AMD-V). Corre en **Ring -1**, por debajo del OS. Convierte el OS original en una máquina virtual que controla. Intercepta todas las llamadas hardware del OS objetivo.

**Hardware/Firmware Rootkit:** se aloja en firmware (BIOS, disco duro, tarjeta de red). Persiste incluso tras reinstalar el OS porque el firmware no se inspecciona para integridad.

**Kernel-Level Rootkit:** corre en **Ring 0**, con los máximos privilegios del OS. Añade o sustituye código del kernel via drivers (Windows) o módulos del kernel (Linux). Si tiene bugs, desestabiliza el sistema. Mismos privilegios que el OS, muy difícil de detectar.

**Boot-Loader-Level Rootkit (Bootkit):** modifica o reemplaza el bootloader. Se activa **antes de que arranque el OS**, por lo que puede capturar claves de cifrado y contraseñas antes de que el sistema operativo las procese. Amenaza muy grave para sistemas con cifrado de disco.

**Application-Level/User-Mode Rootkit:** corre en **Ring 3** junto con el resto de aplicaciones. Reemplaza binarios de aplicaciones legítimas o inyecta código malicioso en ellas. Explota el comportamiento estándar de las APIs.

**Library-Level Rootkit:** opera a nivel de librerías del sistema, reemplazando o hookeando llamadas al sistema para ocultar información al atacante.

**Memory Rootkit (Volatile Rootkit):** reside **únicamente en RAM**, sin dejar rastro en disco. Desaparece al reiniciar. Muy difícil de detectar con herramientas tradicionales.

### Técnicas de funcionamiento de rootkits

**System Hooking:** reemplaza el puntero original de una función con un puntero del rootkit. Todo proceso que llame a esa función pasa primero por el rootkit.

**Inline Function Hooking:** modifica bytes de funciones en DLLs del sistema (kernel32.dll, ntdll.dll) para redirigir las llamadas al rootkit.

**DKOM (Direct Kernel Object Manipulation):** manipula estructuras de datos en memoria del kernel. Puede ocultar procesos y puertos desvinculándolos de las listas del OS, cambiar privilegios, y manipular el visor de eventos de Windows.

### Detección de Rootkits 🔴

**Integrity-Based:** crea un baseline limpio del sistema con herramientas como Tripwire o AIDE y compara periódicamente. Detecta modificaciones por diferencia.

**Signature-Based:** compara procesos y ejecutables con firmas conocidas de rootkits. Puede detectar rootkits invisibles escaneando la memoria del kernel. Limitación: el rootkit puede interrumpir el path de ejecución del detector.

**Heuristic/Behavior-Based:** identifica desviaciones del comportamiento normal del OS. Puede detectar rootkits nuevos sin firma conocida. Busca execution path hooking como indicador.

**Runtime Execution Path Profiling:** compara el path de ejecución de procesos en tiempo real. El rootkit añade código cerca de la ruta de ejecución que altera el comportamiento.

**Cross-View-Based:** llama a APIs del OS para enumerar ficheros/procesos/registros y compara con traversal de bajo nivel. La discrepancia revela lo que el rootkit oculta mediante DKOM o API hooking.

**Alternative Trusted Medium:** apagar el sistema infectado y arrancar desde un USB limpio (WinPE). Inspeccionar el almacenamiento desde fuera del OS comprometido. El método más fiable.

**Memory Dump Analysis:** volcar la RAM completa y analizarla offline para detectar rootkits activos. Puede requerir hardware especializado.

**Virtualization-Based:** usar introspección del hypervisor para monitorizar el OS invitado en busca de rootkits virtuales.

### Detección manual de rootkits (procedimiento)

**Examinando el filesystem:**
```
# Dentro del OS infectado:
dir /s /b /ah       # Ficheros con atributos ocultos
dir /s /b /a-h      # Ficheros sin atributos ocultos
# Arrancar desde USB limpio y repetir los mismos comandos
# Comparar ambos conjuntos de resultados con WinMerge
# Discrepancias = ficheros ocultos por rootkit
```

**Examinando el registro:**
```
# Dentro del OS infectado: exportar hives HKLM\SOFTWARE y HKLM\SYSTEM
# Arrancar desde USB (WinPE)
# Cargar los hives desde c:\windows\system32\config\software y \system
# Exportar en texto y comparar con WinMerge
```

---

## 6. NTFS Alternate Data Streams (ADS) 🔴

NTFS almacena ficheros con **dos data streams**: uno para el security descriptor y otro para los datos del fichero. Los **Alternate Data Streams (ADS)** son streams adicionales que se pueden adjuntar a cualquier fichero sin modificar su tamaño visible, funcionalidad, ni timestamp (excepto la fecha de modificación).

Los ADS son invisibles para el explorador de Windows y la línea de comandos nativa. El atacante puede ocultar ejecutables maliciosos, keyloggers o rootkits dentro de ficheros de texto inofensivos. El tamaño del fichero original **no cambia**.

### Crear y usar NTFS ADS
```
# Ocultar Trojan.exe dentro de Readme.txt
type c:\Trojan.exe > c:\Readme.txt:Trojan.exe

# Crear un enlace simbólico para ejecutar el Trojan desde el ADS
mklink backdoor.exe Readme.txt:Trojan.exe

# Ejecutar el Trojan oculto
backdoor.exe

# Ver fichero oculto con notepad
notepad sample.txt:secret.txt

# Crear stream de texto normal
notepad myfile.txt:lion.txt
```

La sintaxis de ADS es siempre: `fichero.ext:nombreStream`

**Defensa:** mover el fichero sospechoso a una partición FAT (que no soporta ADS, eliminándolos automáticamente), o usar herramientas como Stream Armor, AlternateStreamView, GMER, o Sysinternals Streams.

---

## 7. Steganography 🔴

La steganografía oculta la **existencia** del mensaje, no solo su contenido. Incrusta datos en bits no usados de ficheros ordinarios: imágenes, audio, vídeo, texto. A diferencia del cifrado, si no sabes que hay un mensaje oculto, no puedes intentar descifrar nada.

Casos de uso malicioso: ocultar keyloggers en imágenes, exfiltrar datos sin activar DLP, canales de C2 evasivos.

### Clasificación de Steganografía

**Técnica:** métodos físicos/químicos. Tinta invisible, microdots (texto/imagen reducido a un punto de ~1mm), métodos computacionales.

**Lingüística:** oculta el mensaje en el carrier de otro fichero.
- **Semagrams:** usa signos o símbolos para ocultar información (visual: dibujos, pinturas; text: cambios de fuente, espacios en blanco).
- **Open Codes:** mensaje oculto en comunicación legítima. Incluye jargon codes (lenguaje solo comprensible para el destinatario) y covered ciphers (null ciphers: mezcla el mensaje en datos inútiles; grille ciphers: plantilla de decodificación).

### Técnicas de steganografía en imágenes

**LSB Insertion (Least Significant Bit):** técnica más común. Reemplaza el bit menos significativo de cada pixel con un bit del mensaje oculto. El cambio visual es imperceptible. En una imagen de 24 bits se puede ocultar 1 bit por canal RGB (3 bits por pixel). Detectable con análisis estadístico del LSB.

**Masking y Filtering:** explota las limitaciones de la visión humana ajustando luminosidad y opacidad. Resistente a compresión JPEG. El mensaje se oculta en áreas significativas de la imagen, no en el ruido.

**Algorithms and Transformation:** oculta información durante la compresión usando transformadas matemáticas (DCT, FFT, Wavelet). Más resistente a ataques de procesado de señal. JPEG usa DCT internamente.

### Tipos de Steganografía por medio

- **Image:** más popular; PNG, JPG, BMP. Técnicas: LSB, masking, DCT.
- **Audio:** WAV, AU, MP3. Técnicas: echo hiding, spread spectrum (DSSS/FHSS), LSB coding, tone insertion, phase encoding.
- **Video:** mayor capacidad que imagen; AVI, MP4, WMV. Usa DCT. Difícil de detectar.
- **Document:** añade whitespace y tabs al final de líneas.
- **Whitespace:** espacios y tabs en texto ASCII. Herramienta: **Snow**.
- **Folder:** ficheros ocultos dentro de carpetas invisibles para Windows Explorer.
- **Spam/Email:** mensajes ocultos en spam. Herramienta: **Spam Mimic**.
- **Web:** objetos web ocultos detrás de otros objetos en servidor web.
- **Compressed Data:** LSB o bits reservados de ficheros comprimidos (ZIP, RAR, JPEG, PNG).

### Herramientas clave
- **OpenStego:** data hiding y watermarking en imágenes.
- **Snow:** whitespace steganography en texto.
- **DeepSound:** datos en audio (WAV, FLAC, MP3).
- **OmniHide Pro:** cualquier fichero en imagen/vídeo/audio.
- **zsteg:** detecta datos ocultos en PNG y BMP (herramienta de steganalysis).

### Steganalysis
Proceso inverso: detectar la existencia de información oculta.

Tipos de ataques:
- **Stego-only:** solo tiene el stego-object; prueba todos los algoritmos posibles.
- **Known-stego:** conoce el algoritmo Y el stego-object original.
- **Known-message:** tiene el mensaje Y el stego-medium; detecta la técnica usada.
- **Known-cover:** tiene el stego-object Y el cover original; compara diferencias.
- **Chosen-message:** genera stego-objects con mensaje conocido para detectar patrones.
- **Chosen-stego:** conoce el stego-object Y la herramienta/algoritmo.
- **Chi-square:** análisis probabilístico: si diferencia entre cover y stego ≈ 0, sin datos ocultos.
- **Distinguishing statistical:** analiza cambios estadísticos del algoritmo embebido.
- **Blind classifier:** entrena detector con datos originales para clasificar diferencias.

---

## 8. Maintaining Persistence

### Sticky Keys Persistence
El atacante escala privilegios con `bypassuac` en Metasploit y usa el módulo `sticky_keys`. Al reiniciar y pulsar Shift x5, se abre un Command Prompt con acceso SYSTEM sin iniciar sesión.

### Boot/Logon Autostart Execution
**Registry Run Keys:** el atacante añade su payload a una registry key vinculada a un servicio para que se ejecute en cada login. Enumeración con WinPEAS:
```
winPEASx64.exe quiet applicationinfo
```

**Startup Folder:** inyecta un RAT u otro payload en la carpeta de inicio de Windows. Enumeración de permisos:
```
icacls "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
accesschk.exe /accepteula "C:\ProgramData\...\Startup"
```

---

## 9. Domain Dominance 🔴

Domain dominance es tomar control sobre los activos críticos del dominio, principalmente los Domain Controllers, para mantener acceso persistente con máximos privilegios.

### Remote Code Execution en DC
```
# Crear usuario en el DC via WMI
wmic /node:<DCName> process call create "net user /add PiratedProcess Du^^Y01"

# Añadir al grupo Admins
PsExec.exe \\<DCName> -accepteula net localgroup "Admins" PiratedProcess /add
```

### DPAPI Abuse (Data Protection API)
Los DCs contienen la **master key** para descifrar ficheros protegidos por DPAPI (contraseñas de navegadores, ficheros cifrados). Comandos Mimikatz:
```
# Recuperar master key con contraseña del usuario comprometido
dpapi::masterkey /in:"<ruta>" /sid:<SID> /password:******* /protected

# Obtener todas las master keys locales con credenciales de admin
sekurlsa::dpapi

# Obtener todas las backup master keys
lsadump::backupkeys /system:dc01.offense.local /export
```

### Malicious Replication
Replica cuentas sensibles como `krbtgt` usando DCSync:
```
Invoke-Mimikatz -command '"lsadump::dcsync /domain:<Domain> /user:krbtgt"'
```

### Skeleton Key Attack 🔴
Inyecta una **contraseña maestra falsa** directamente en el proceso LSASS de los DCs. Permite al atacante autenticarse como **cualquier usuario** con esa contraseña maestra, mientras los usuarios legítimos siguen pudiendo usar sus contraseñas normales. Requiere privilegios de Domain Admin.
```
# Ejecutar skeleton key via Mimikatz
Invoke-Mimikatz -Command '"privilege::debug" "misc::skeleton"' -<DCName>

# Via Empire
powershell/persistence/misc/skeleton_key
```
El error "LSASS patched" indica que ya hay un skeleton key activo.

### Golden Ticket Attack 🔴
Forja **TGTs** comprometiendo el hash NTLM de la cuenta **KRBTGT** del dominio. Permite acceso a cualquier recurso, grupo o dominio de todo el entorno AD. El ticket puede tener cualquier validez definida por el atacante.

Pasos:
1. Obtener info del dominio: `whoami`
2. Obtener el hash NTLM de KRBTGT via DCSync:
   ```
   lsadump::dcsync /domain:<domain> /user:krbtgt
   ```
3. Crear el golden ticket:
   ```
   kerberos::golden /domain:<domain> /sid:<SID> /rc4:<KRBTGT_hash> /id:<id> /user:<username>
   ```

### Silver Ticket Attack 🔴
Forja un **TGS** para un servicio específico usando el hash NTLM de la **cuenta de servicio**. Acceso limitado a un único servicio. **No requiere comunicación con el DC** para su creación: más silencioso y difícil de detectar.

Pasos clave:
```
# Extraer credenciales del sistema comprometido
mimikatz "privilege::debug" "sekurlsa::logonpasswords"
# Crear silver ticket con el hash de la cuenta de servicio
```

**Diferencia fundamental:** Golden = usa hash KRBTGT + acceso total al dominio. Silver = usa hash de cuenta de servicio + acceso a un solo servicio. Silver no requiere DC en su creación.

### AdminSDHolder Persistence
`AdminSDHolder` es un objeto AD que protege cuentas y grupos con altos privilegios. El proceso `SDProp` reaplica sus ACLs **cada 60 minutos**. El atacante con privilegios de admin añade su cuenta al ACL de AdminSDHolder con "GenericAll" y SDProp propaga esos permisos automáticamente cada hora.
```
# Añadir usuario Martin con GenericAll al AdminSDHolder
Add-ObjectAcl -PrincipalSamAccountName Martin -Verbose -Rights All -TargetADSprefix 'CN=AdminSDHolder,CN=System'

# Cambiar el tiempo de SDProp a 3 minutos (300 segundos)
REG ADD HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NTDS\Parameters /V AdminSDProtectFrequency /T REG_DWORD /F /D 300

# Añadir al grupo Domain Admins
net group "Domain Admins" Martin /add /domain
```

### WMI Event Subscription Persistence
Crea suscripciones WMI maliciosas que se ejecutan automáticamente al producirse eventos (arranque, login, intervalo). Persisten a través de reinicios.

Herramientas: wmic (Command Prompt), Wmi-Persistence (PowerShell), PowerLurk.
```powershell
# Via Wmi-Persistence
Install-Persistence -Trigger Startup -Payload "c:\windows\system32\ethicalhacker.exe"

# Via PowerLurk
Import-Module .\PowerLurk.ps1
Register-MaliciousWmiEvent -EventName Logonlog -PermanentCommand "ethicalhacker.exe" -Trigger UserLogon -Username any
Get-WmiEvent   # Ver eventos WMI activos
```

### Overpass-the-Hash (OPtH) Attack
Extensión del Pass-the-Hash: usa el **hash NTLM para obtener un TGT de Kerberos** en lugar de autenticarse directamente con NTLM. Combina PtH y PtT. Los hashes se reutilizan hasta que el usuario cambie su contraseña.
```
# Extraer hashes y claves AES del DC
privilege::debug
sekurlsa::ekeys
```

---

## 10. Post-Exploitation: Comandos de Referencia

### Linux Post-Exploitation
```bash
find / -perm -3000 -ls 2>/dev/null           # Binarios SUID ejecutables
find / -perm -o=w -type f -ls 2>/dev/null    # Ficheros world-writable
sudo -l                                       # Comandos sudo permitidos
cat /etc/crontab                              # Cron jobs del sistema
cat /etc/exports                              # Directorios exportados via NFS
uname -a                                      # Version del kernel
cat /etc/issue                                # Distribucion OS
```

### Windows Post-Exploitation
```
dir /a:h                           # Directorios con atributos ocultos
Get-FileHash <file> -a md5         # Hash MD5
Get-FileHash <file>                # SHA-256 por defecto
sc queryex type=service state=all  # Todos los servicios
netsh firewall show state          # Estado del firewall
netsh advfirewall set allprofiles state off   # Desactivar firewall
arp -a                             # Tabla ARP
ipconfig /all                      # Configuracion IP completa
wmic useraccount get name,sid      # Usuarios y SIDs
reg query HKEY_LOCAL_MACHINE /f credential /t REG_SZ /s   # Buscar credenciales
```

---

## 3. Exam Traps ⚠️

⚠️ **[Hardware vs Software Keylogger]** — El examen puede preguntar cuál es imposible de detectar con anti-keylogger software. La respuesta es el **hardware keylogger** (especialmente el tipo keyboard/cable). Los software keyloggers SÍ pueden ser detectados; los hardware NO, porque no son software ni interactúan con el OS.

⚠️ **[Form-Grabbing Keylogger]** — Bypasa HTTPS. El examen puede preguntar qué tipo de keylogger captura datos de formularios web aunque la conexión esté cifrada. Es el **form-grabbing-based keylogger** porque intercepta antes del cifrado, en el evento `submit`.

⚠️ **[Audio Spyware: sin privilegios]** — El audio spyware **no requiere privilegios de administrador** para instalarse y ejecutarse. Excepción notable respecto a otros tipos de spyware.

⚠️ **[6 tipos de rootkits + Ring levels]** — Pregunta frecuente sobre qué tipo opera en Ring -1 (Hypervisor), Ring 0 (Kernel), Ring 3 (Application/User-mode). Memorizar: Hypervisor = Ring -1, Kernel = Ring 0, Application = Ring 3.

⚠️ **[Bootkit]** — Se activa ANTES de que arranque el OS. Puede capturar claves de cifrado de disco completo. El examen puede preguntar qué tipo de rootkit compromete sistemas con full disk encryption: bootkit.

⚠️ **[Memory Rootkit]** — El único rootkit que no deja rastro en disco. Desaparece al reiniciar. El examen puede preguntar cuál es el más difícil de analizar con herramientas forenses tradicionales basadas en disco.

⚠️ **[NTFS ADS: tamaño del fichero]** — El tamaño visible del fichero original **no cambia** cuando se le añade un ADS. El único indicador puede ser la marca de tiempo de modificación.

⚠️ **[NTFS ADS: defensa con FAT]** — Para eliminar ADS, mover el fichero a una partición FAT. FAT no soporta ADS, por lo que los elimina automáticamente al copiar el fichero.

⚠️ **[Steganografía vs Criptografía]** — Criptografía oculta el **contenido** del mensaje. Steganografía oculta la **existencia** del mensaje. El examen puede preguntar la diferencia: si no sabes que hay un mensaje oculto, no puedes intentar descifrar nada.

⚠️ **[LSB Insertion: detectable con análisis estadístico]** — Los LSBs de una stego-image ya no son aleatorios. El **chi-square attack** explota precisamente esto. El examen puede preguntar qué técnica de steganalysis detecta LSB modification.

⚠️ **[Skeleton Key: contraseña universal sin eliminar contraseñas originales]** — El skeleton key NO elimina las contraseñas originales de los usuarios. Añade una **contraseña maestra adicional**. Los usuarios legítimos siguen funcionando con sus contraseñas normales.

⚠️ **[Golden Ticket: validez configurable]** — El golden ticket puede configurarse con cualquier tiempo de validez (incluso 10 años). El examen puede preguntar qué hace especialmente peligroso el golden ticket frente a otros tickets.

⚠️ **[Silver Ticket vs Golden Ticket: comunicación con DC]** — Golden Ticket requiere el hash KRBTGT. Silver Ticket **no requiere comunicación con el DC** para su creación, solo el hash de la cuenta de servicio. Silver Ticket es más silencioso y difícil de detectar.

⚠️ **[AdminSDHolder: frecuencia de SDProp]** — Por defecto, SDProp se ejecuta cada **60 minutos**. El examen puede preguntar la frecuencia por defecto.

⚠️ **[WMI: puertos]** — WMI usa **DCOM en el puerto 135** para DCOM y **WinRM en HTTP 5985 / HTTPS 5986**. El examen puede preguntar los puertos de WMI.

⚠️ **[Overpass-the-Hash vs Pass-the-Hash]** — PtH usa el hash NTLM para autenticación NTLM directa. OPtH usa el hash NTLM para **solicitar un TGT de Kerberos**. El resultado es un ticket Kerberos, no una sesión NTLM.

---

## 4. Nemotécnicos

**7 tipos de rootkits (Ring levels de mayor a menor privilegio):**
> **"Hyper Hard Kernel Boots Apps Library Memory"**
> Hypervisor (Ring -1) → Hardware/Firmware → Kernel (Ring 0) → Bootloader → Application (Ring 3) → Library → Memory (volatile)

**8 métodos de detección de rootkits:**
> **"I Shall Have Run Cross Alternative Memory Virtual"**
> Integrity → Signature → Heuristic → Runtime exec path → Cross-view → Alternative trusted medium → Memory dump → Virtualization

**Domain Dominance técnicas (6):**
> **"RCE DPAPI Malicious Skeleton Golden Silver"**
> Remote Code Exec → DPAPI Abuse → Malicious Replication → Skeleton Key → Golden Ticket → Silver Ticket

**Software keylogger types (6):**
> **"Application Kernel Hypervisor Form-Grab JavaScript Memory"**
> A → K → H → FG → JS → Mem

**Steganography attacks (9):**
> **"Stego-only Known-stego Known-msg Known-cover Chosen-msg Chosen-stego Chi Distrib Blind"**

---

## 5. Flashcards

**Q:** ¿Qué tipo de keylogger es imposible de detectar con software anti-keylogger?
**A:** Hardware keylogger (tipo keyboard/cable), porque opera a nivel hardware fuera del alcance del OS.

**Q:** ¿Qué tipo de keylogger captura credenciales de formularios HTTPS?
**A:** Form-grabbing-based keylogger. Intercepta en el evento submit antes del cifrado.

**Q:** ¿Qué secuencia de comandos Meterpreter activa y vuelca un keylogger?
**A:** `migrate <PID>` → `keyscan_start` → `keyscan_dump` → `keyscan_stop`

**Q:** ¿Qué método de propagación instala spyware automáticamente al visitar una web?
**A:** Drive-by download.

**Q:** ¿Qué tipo de spyware no requiere privilegios de administrador?
**A:** Audio spyware.

**Q:** ¿En qué Ring opera un Hypervisor-Level Rootkit?
**A:** Ring -1 (por debajo del OS).

**Q:** ¿En qué Ring opera un Kernel-Level Rootkit?
**A:** Ring 0 (máximos privilegios del OS).

**Q:** ¿Qué tipo de rootkit puede activarse antes de que arranque el OS?
**A:** Boot-Loader-Level Rootkit (Bootkit).

**Q:** ¿Qué tipo de rootkit no deja rastro en disco?
**A:** Memory Rootkit (Volatile Rootkit). Desaparece al reiniciar.

**Q:** ¿Cambia el tamaño visible de un fichero al añadirle un NTFS ADS?
**A:** No. El tamaño del fichero original permanece igual.

**Q:** ¿Cuál es la sintaxis para ocultar un fichero en un ADS?
**A:** `type c:\malware.exe > c:\readme.txt:malware.exe`

**Q:** ¿Qué diferencia la steganografía de la criptografía?
**A:** La steganografía oculta la existencia del mensaje; la criptografía oculta su contenido.

**Q:** ¿Qué técnica de steganografía de imagen es la más común?
**A:** LSB Insertion (Least Significant Bit).

**Q:** ¿Qué herramienta de steganografía usa whitespace en ficheros de texto?
**A:** Snow.

**Q:** ¿Qué ataque de steganalysis usa análisis probabilístico para detectar datos embebidos?
**A:** Chi-square attack.

**Q:** ¿Qué es un Skeleton Key y qué privilegios requiere?
**A:** Malware que inyecta una contraseña maestra en LSASS del DC para acceder como cualquier usuario. Requiere privilegios de Domain Admin.

**Q:** ¿Qué hash necesita el atacante para crear un Golden Ticket?
**A:** El hash NTLM de la cuenta KRBTGT.

**Q:** ¿Qué diferencia Silver Ticket de Golden Ticket respecto al DC?
**A:** Silver Ticket no requiere comunicación con el DC para su creación. Golden Ticket sí.

**Q:** ¿Con qué frecuencia ejecuta SDProp sus actualizaciones por defecto?
**A:** Cada 60 minutos.

**Q:** ¿Qué puertos usa WMI para acceso remoto?
**A:** DCOM: puerto 135. WinRM: HTTP 5985 / HTTPS 5986.

**Q:** ¿Qué distingue Overpass-the-Hash de Pass-the-Hash?
**A:** PtH usa el hash NTLM para autenticación NTLM directa. OPtH usa el hash NTLM para obtener un TGT de Kerberos.

**Q:** ¿Qué método de detección de rootkits es el más fiable?
**A:** Alternative Trusted Medium: arrancar desde USB limpio y examinar el sistema desde fuera del OS infectado.

**Q:** ¿Qué herramienta de Sysinternals detecta y analiza ADS en ficheros?
**A:** Streams (parte de Sysinternals Suite).

**Q:** ¿Qué comando de PowerLurk lista todos los eventos WMI activos?
**A:** `Get-WmiEvent`

---

## 6. Confusión frecuente

**Keylogger hardware vs software** → Hardware: no detectable por software, independiente del OS. Software: instalable remotamente, detectable por anti-keyloggers.

**Form-grabbing vs JavaScript-based keylogger** → Form-grabbing: intercepta datos al hacer submit, bypasa HTTPS. JavaScript: escucha eventos de teclado en tiempo real vía JS inyectado. Ambos operan en el navegador pero de forma diferente.

**Kernel-Level Rootkit vs Hypervisor-Level Rootkit** → Kernel: Ring 0, modifica el kernel del OS. Hypervisor: Ring -1, el OS se convierte en VM controlada. El hypervisor es más poderoso porque está por debajo del kernel.

**Bootkit vs Kernel Rootkit** → Bootkit: se activa antes de que el OS arranque, puede capturar claves de cifrado de disco. Kernel rootkit: opera una vez el OS está en ejecución.

**NTFS ADS vs Steganografía** → ADS: adjunta datos a ficheros NTFS sin cambiar el tamaño visible; mecanismo del sistema de ficheros. Steganografía: oculta datos dentro del contenido del fichero (ej. LSB de pixeles).

**LSB Insertion vs Masking/Filtering** → LSB: modifica el bit menos significativo de cada pixel, detectable con análisis estadístico. Masking/Filtering: oculta en zonas de luminosidad, resistente a JPEG y cropping.

**Golden Ticket vs Silver Ticket** → Golden: hash KRBTGT, forja TGT, acceso total al dominio. Silver: hash de cuenta de servicio, forja TGS, acceso a un solo servicio, sin comunicación con DC.

**Skeleton Key vs Golden Ticket** → Skeleton Key: inyecta contraseña maestra en LSASS del DC. Golden Ticket: forja TGTs usando hash KRBTGT. Skeleton Key es más directo pero requiere acceso al LSASS del DC en cada reinicio (es memory-resident).

**Malicious Replication vs DCSync** → Son prácticamente lo mismo: Malicious Replication es el término de domain dominance para replicar cuentas como krbtgt. DCSync es el nombre del ataque/técnica general. Ambos usan los mismos comandos de Mimikatz.

**AdminSDHolder vs ACL modification directa** → AdminSDHolder persiste porque SDProp la restaura cada 60 min aunque se elimine manualmente. Una ACL modification directa puede ser detectada y revertida sin regenerarse automáticamente.

**WMI Event Subscription vs Scheduled Task** → WMI Event Subscription es más sigilosa (no aparece en la lista de tareas) y persiste a través de reinicios. Scheduled Task es más visible pero igualmente efectiva.

**Overpass-the-Hash (OPtH) vs Pass-the-Hash (PtH)** → PtH: usa hash NTLM para autenticarse directamente en recursos via NTLM. OPtH: convierte el hash NTLM en un TGT de Kerberos. El resultado de OPtH es un ticket Kerberos.

---

## 11. Preguntas de Práctica — Formato CEH

---

### Pregunta 1 — Tipos de Rootkit por Ring Level
Un analista forense investiga un sistema comprometido. Los AV no detectan nada, pero al arrancar desde un USB limpio encuentra ficheros ocultos que no aparecen al arrancar el sistema normalmente. El rootkit opera entre el hardware y el sistema operativo, convirtiendo el OS en una máquina virtual. ¿Qué tipo de rootkit es?

A) Kernel-Level Rootkit (Ring 0)  
B) Hypervisor-Level Rootkit (Ring -1)  
C) Boot-Loader-Level Rootkit (Bootkit)  
D) Application-Level Rootkit (Ring 3)  

> **Respuesta correcta: B** — El **Hypervisor-Level Rootkit** opera en **Ring -1**, por debajo del OS. Convierte el OS original en una máquina virtual que controla, interceptando todas las llamadas hardware. El Kernel rootkit opera en Ring 0. El Bootkit actúa antes del arranque del OS pero no virtualiza el sistema. El Application rootkit opera en Ring 3.

---

### Pregunta 2 — Bootkit
Un administrador de seguridad está evaluando el riesgo de un rootkit que se activa antes de que el sistema operativo arranque. El sistema usa cifrado de disco completo con BitLocker. ¿Por qué este tipo de rootkit es especialmente peligroso en este escenario?

A) Porque puede desinstalar BitLocker desde el firmware  
B) Porque puede capturar las claves de cifrado antes de que BitLocker las aplique, cuando se introducen  
C) Porque puede acceder a la memoria RAM y exfiltrar datos en tiempo real  
D) Porque bypasa el firmware UEFI Secure Boot automáticamente  

> **Respuesta correcta: B** — El **Bootkit** (Boot-Loader-Level Rootkit) se activa **antes de que el OS arranque**, lo que le permite capturar las claves de cifrado de disco completo (como las de BitLocker) en el momento en que el usuario las introduce, antes de que el OS las procese. Es la amenaza más grave contra sistemas con FDE.

---

### Pregunta 3 — Memory Rootkit
Un incidente de seguridad revela evidencias de un rootkit activo. Sin embargo, tras apagar el sistema y reiniciarlo con un disco limpio, no se encuentra ningún rastro del rootkit en el disco. ¿Qué tipo de rootkit puede explicar esto?

A) Hardware/Firmware Rootkit  
B) Library-Level Rootkit  
C) Memory Rootkit (Volatile Rootkit)  
D) Kernel-Level Rootkit  

> **Respuesta correcta: C** — El **Memory Rootkit (Volatile Rootkit)** reside únicamente en la RAM. Desaparece al reiniciar el sistema porque no escribe nada en disco. Es muy difícil de detectar con herramientas forenses tradicionales basadas en análisis de disco. La única forma de capturarlo es mediante análisis de memoria RAM (memory dump) mientras el sistema está activo.

---

### Pregunta 4 — NTFS Alternate Data Streams
Un analista forense examina un fichero `readme.txt` en un sistema Windows con NTFS. El tamaño del fichero es de 200 bytes, pero sospecha que contiene un ejecutable malicioso oculto. ¿Cuál de las siguientes afirmaciones es correcta sobre NTFS ADS?

A) El tamaño del fichero aumenta en función del tamaño del ADS  
B) El ADS es visible en el Explorador de Windows nativo  
C) El tamaño del fichero original no cambia al añadir un ADS  
D) Los ADS solo pueden contener texto, no ejecutables  

> **Respuesta correcta: C** — El tamaño visible del fichero original **no cambia** cuando se le añade un ADS. Esto lo hace especialmente difícil de detectar. El ADS es invisible al Explorador de Windows y a `dir` sin flags especiales. Puede contener cualquier tipo de datos, incluyendo ejecutables. El único indicador puede ser la marca de tiempo de modificación.

---

### Pregunta 5 — Steganografía vs Criptografía
Un atacante quiere exfiltrar datos sin activar los sistemas DLP de la organización. Decide incrustar los datos robados en una imagen JPEG enviada por email. El receptor, que usa el mismo software, puede extraer los datos. ¿Qué técnica está usando y en qué se diferencia de la criptografía?

A) Criptografía; oculta el contenido del mensaje cifrándolo  
B) Steganografía; oculta la existencia del mensaje incrustando datos en el carrier  
C) Esteganografía; cifra los datos usando el JPEG como clave de cifrado  
D) Criptografía con envelope; usa el JPEG como contenedor del mensaje cifrado  

> **Respuesta correcta: B** — La **Steganografía** oculta la **existencia** del mensaje (si no sabes que hay datos ocultos, no puedes intentar extraerlos ni detectarlos con DLP). La **Criptografía** oculta el **contenido** del mensaje pero no su existencia. La steganografía es invisible para sistemas DLP que no están específicamente buscando patrones estadísticos.

---

### Pregunta 6 — LSB Insertion
Un investigador de seguridad analiza una imagen PNG sospechosa y detecta que los bits menos significativos de los canales RGB de cada pixel siguen un patrón estadístico no aleatorio. ¿Qué técnica de steganografía ha identificado y qué ataque de steganalysis puede detectarla?

A) Masking and Filtering; detectado con Runtime Execution Path Profiling  
B) LSB Insertion; detectable con Chi-square attack (análisis estadístico)  
C) DCT Transformation; detectable con análisis espectral de frecuencias  
D) Spread Spectrum; detectable con Cross-View Analysis  

> **Respuesta correcta: B** — **LSB Insertion** modifica el bit menos significativo de cada pixel. En imágenes con datos ocultos, los LSBs ya no son estadísticamente aleatorios. El **Chi-square attack** detecta este patrón analizando la distribución de probabilidad de los pares de valores consecutivos (POVs).

---

### Pregunta 7 — Skeleton Key
Un auditor de seguridad examina un Domain Controller y encuentra que el proceso LSASS tiene un módulo inyectado no autorizado. Tras investigar, descubre que cualquier usuario puede autenticarse con la contraseña "mc$ft" independientemente de su contraseña real, y que los usuarios legítimos siguen funcionando normalmente. ¿Qué ha ocurrido?

A) Se ha realizado un Golden Ticket Attack con un ticket de larga duración  
B) Se ha instalado un Skeleton Key en el LSASS del DC  
C) Se ha realizado un DCSync Attack y todos los hashes han sido comprometidos  
D) Se ha modificado el esquema de Active Directory para añadir una contraseña master  

> **Respuesta correcta: B** — El **Skeleton Key** inyecta una **contraseña maestra adicional** directamente en el proceso LSASS del DC. No elimina ni modifica las contraseñas originales de los usuarios (que siguen funcionando). Cualquier usuario puede autenticarse con la contraseña del skeleton key. Requiere privilegios de Domain Admin y es memory-resident (desaparece al reiniciar).

---

### Pregunta 8 — AdminSDHolder
Un red teamer con privilegios de Domain Admin quiere mantener acceso persistente al dominio incluso si su cuenta es detectada y sus permisos son revocados manualmente. Ha añadido su cuenta al ACL del objeto AdminSDHolder. ¿Por qué esta técnica es especialmente persistente?

A) Porque AdminSDHolder está protegido por Kerberos y no puede ser modificado  
B) Porque el proceso SDProp restaura automáticamente las ACLs del AdminSDHolder cada 60 minutos  
C) Porque AdminSDHolder replica sus permisos a todos los DCs del bosque  
D) Porque AdminSDHolder tiene acceso de escritura sobre todas las GPOs del dominio  

> **Respuesta correcta: B** — El proceso **SDProp** reaplica las ACLs del AdminSDHolder **cada 60 minutos** automáticamente. Aunque el administrador elimine manualmente los permisos del atacante, SDProp los restaura en la siguiente ejecución. La única forma de eliminar esta persistencia es modificar el ACL del propio objeto AdminSDHolder.

---

### Pregunta 9 — Form-Grabbing Keylogger
Un investigador de seguridad examina un keylogger instalado en un sistema con comunicaciones HTTPS. Descubre que el keylogger captura los datos de formularios bancarios aunque la comunicación esté cifrada con TLS 1.3. ¿Cómo es posible esto?

A) El keylogger ha comprometido el certificado TLS del banco  
B) El keylogger es Form-Grabbing-based: intercepta los datos en el evento `submit` antes de que sean cifrados  
C) El keylogger usa un certificado raíz malicioso instalado en el sistema para hacer MITM de TLS  
D) El keylogger explota una vulnerabilidad de TLS 1.3 conocida  

> **Respuesta correcta: B** — El **Form-Grabbing keylogger** captura los datos de formularios web en el momento del **evento `submit`**, **antes** de que la aplicación los cifre para transmitirlos por TLS. Los datos existen en texto claro en el proceso del navegador antes de ser cifrados. Por eso ni HTTPS ni TLS 1.3 protegen contra este tipo de keylogger.

---

### Pregunta 10 — Golden Ticket vs Silver Ticket
Un atacante necesita acceder a un share SMB específico (`\\fileserver\finance$`) sin ser detectado por los logs del Domain Controller. Tiene el hash NTLM de la cuenta de servicio del servidor de ficheros. ¿Qué ticket debe forjar y por qué es más silencioso?

A) Golden Ticket; porque solo requiere comunicación inicial con el DC  
B) Silver Ticket; porque no requiere ninguna comunicación con el DC para su creación  
C) Golden Ticket; porque da acceso a todos los servicios sin renegociación  
D) TGT normal; porque es el ticket más común y menos sospechoso  

> **Respuesta correcta: B** — El **Silver Ticket** usa el hash NTLM de la **cuenta de servicio** del target (no el KRBTGT). Al crearse directamente sin comunicar con el DC, **no genera eventos de autenticación en el DC**, lo que lo hace más difícil de detectar. El Golden Ticket requiere el hash KRBTGT y sí puede generar eventos en el DC.

---

### Pregunta 11 — WMI Event Subscription
Un red teamer quiere establecer persistencia en un sistema Windows que sobreviva reinicios y que no sea visible en la lista de tareas programadas del sistema. ¿Qué mecanismo debe usar?

A) Scheduled Task via `schtasks /create`  
B) Registry Run Key en `HKCU\...\Run`  
C) WMI Event Subscription con trigger de arranque del sistema  
D) Startup Folder con enlace simbólico al payload  

> **Respuesta correcta: C** — **WMI Event Subscription** crea suscripciones que se ejecutan ante eventos específicos (arranque, login, intervalo de tiempo). Son persistentes a través de reinicios y **no aparecen en la lista de tareas programadas** de Windows, lo que las hace más difíciles de detectar. Herramientas: PowerLurk, Wmi-Persistence, WMIC.

---

### Pregunta 12 — Tipos de Hardware Keylogger
Un investigador de seguridad física examina el equipo de un CEO sospechoso de espionaje corporativo. Encuentra un dispositivo conectado entre el conector USB del teclado y el puerto USB del ordenador. El dispositivo no tiene drivers ni software instalado en el sistema. ¿Cuál es la característica clave de este dispositivo?

A) Es un USB Keylogger de tipo software que captura eventos del OS  
B) Es un hardware keylogger externo (PS/2 o USB) que opera independientemente del OS y es imposible de detectar con software anti-keylogger  
C) Es un dispositivo de cifrado de teclado que protege las pulsaciones  
D) Es un adaptador de conectividad Bluetooth para el teclado  

> **Respuesta correcta: B** — El **hardware keylogger externo (USB/PS/2)** se conecta físicamente en línea entre el teclado y el PC. No requiere software ni drivers, opera **independientemente del sistema operativo**, y es **imposible de detectar con herramientas software anti-keylogger**. Solo puede ser detectado mediante inspección física del hardware.

