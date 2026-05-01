# M07 — Malware Threats
### Parte 3: Trojan Concepts

---

## 1. ¿Qué es un Troyano?

Un troyano es un programa en el que código malicioso o dañino está contenido dentro de un programa o dato aparentemente inofensivo. A diferencia de los virus, los troyanos no se replican por sí mismos. Se activan mediante acciones predefinidas del usuario (instalar software malicioso, hacer clic en un link malicioso) y conceden al atacante acceso irrestricto a todos los datos del sistema comprometido.

Características clave:
- Trabaja con el **mismo nivel de privilegios que la víctima**. Si la víctima puede borrar ficheros o instalar programas, el troyano también puede hacerlo.
- Puede intentar explotar vulnerabilidades para **elevar sus privilegios** más allá del usuario que lo ejecuta.
- Puede **falsamente implicar a un sistema remoto** como origen de un ataque mediante spoofing.
- Entra al sistema principalmente via: adjuntos de email, descargas, y mensajes instantáneos.

---

## 2. Indicios de Ataque de Troyano

Los más relevantes para el examen:
- La pantalla parpadea, se voltea o muestra todo al revés.
- El fondo de escritorio cambia automáticamente.
- La impresora imprime documentos por sí sola.
- Se abren páginas web sin input del usuario.
- El antivirus se deshabilita automáticamente y los datos se corrompen.
- El cursor del ratón se mueve solo o desaparece.
- Las funciones izquierda/derecha del ratón se intercambian.
- El teclado y el ratón se congelan.
- Los contactos reciben emails desde la dirección del usuario que éste no envió.
- El Task Manager se deshabilita.
- El disco duro pierde espacio repentinamente.
- Actividad de red inusual cuando el usuario no está activo.
- Uso elevado e inusual de CPU o memoria (troyano en background).
- Programas que se abren o cierran solos sin intervención del usuario.

---

## 3. Usos de los Troyanos por los Atacantes

- Borrar o reemplazar ficheros críticos del OS.
- Generar tráfico falso para ataques DoS.
- Grabar screenshots, audio y vídeo del PC de la víctima.
- Usar el PC de la víctima para spam y blasting de emails.
- Descargar spyware, adware y ficheros maliciosos.
- Deshabilitar firewalls y antivirus.
- Crear backdoors para acceso remoto.
- Usar el PC como proxy server para relayar ataques.
- Usar el PC como zombie en una botnet para ataques DDoS.
- Robar información sensible: tarjetas de crédito, contraseñas, proyectos de empresa.
- Cifrar la máquina de la víctima (ransomware).
- Conscriptar el dispositivo en una botnet para DDoS, spam, o criptominería.
- Servir como mecanismo de entrega para otros tipos de malware: virus, gusanos, spyware.

---

## 4. Puertos Comunes Usados por Troyanos 🔴

Los troyanos usan el estado "listening" al reiniciar el sistema. Los más importantes para el examen:

| Puerto | Troyanos asociados |
|---|---|
| 21 | Blade Runner, Doly Trojan, DarkFTP, WinCrash |
| 22 | Shaft, SSH RAT, Linux Rabbit |
| 23 | Tiny Telnet Server, EliteWrap |
| 25 | Antigen, Email Password Sender, WinSpy, Lazarus Group, Night Dragon |
| 53 | Denis, Ebury, FIN7, Lazarus Group, RedLeaves |
| 80 | Necurs, NetWire, Poison Ivy, Codered, APT 18/19/32, Carbanak, Lazarus Group, FIN7 |
| 139 | Nuker, Dragonfly 2.0 |
| 443 | APT 29/3/33, BADCALL, Carbanak, Empire, FIN7/8, gh0st RAT, Lazarus Group, TrickBot |
| 445 | WannaCry, Petya, Dragonfly 2.0 |
| 1243 | Sub Seven |
| 3389 | RDP (Remote Desktop Protocol) |
| 12345–46 | GabanBus, NetBus |
| 31337–38 | Back Orifice / Back Orifice 1.20 |

---

## 5. Tipos de Troyanos (14 categorías) 🔴

### 1. Remote Access Trojans (RATs)
Proporcionan al atacante **control total** sobre el sistema de la víctima. El RAT actúa como servidor y escucha en un puerto. Permite: acceso al GUI completo, captura de pantalla y cámara, ejecución de código, keylogging, acceso a ficheros, sniffing de contraseñas, gestión del registro.

**Diferencia clave con backdoor:** el RAT tiene un componente **cliente** con interfaz de usuario que el atacante usa para emitir comandos; el backdoor no tiene interfaz.

Ejemplos: Remcos RAT, Parallax RAT, AsyncRAT, njRAT, AhMyth, MagicRAT, NetSupport RAT.

### 2. Backdoor Trojans
Bypasan la autenticación estándar del sistema y mecanismos convencionales (IDS, firewalls) sin ser detectados. La instalación se realiza sin conocimiento del usuario. Frecuentemente usados en la segunda (punto de entrada) o tercera (C&C) etapa del proceso de ataque dirigido. A menudo se usan para agrupar computadoras en botnets.

**Diferencia clave con RAT:** el backdoor NO tiene interfaz de usuario/cliente; el RAT SÍ.

Ejemplos: TinyTurla-NG (TTNG), SmokeLoader, BazarLoader, Kovter, ServHelper.

### 3. Botnet Trojans
Infectan gran número de computadoras para crear una red de bots controlada via **C&C**. Técnicas de distribución: phishing, SEO hacking, URL redirection. Una vez ejecutado, se conecta al atacante via **canales IRC** y espera instrucciones. Algunos tienen características de gusano y se propagan automáticamente.

Usos: DoS, spam, click fraud, robo de números de serie, IDs, tarjetas de crédito.

Ejemplos: RDDoS, Horabot, hailBot, catDDoS, ZeroBot, Qakbot, Ramnit, Panda.

### 4. Rootkit Trojans
"Root" = administrador en UNIX/Linux. "Kit" = programas para obtener acceso root/admin.

**No se propagan por sí mismos.** Son parte de una **blended threat** con tres componentes:
- **Dropper:** instala el rootkit. Requiere intervención humana (hacer clic). Luego se elimina.
- **Loader:** lanzado por el dropper. Típicamente causa un **buffer overflow** para cargar el rootkit en memoria.
- **Rootkit:** proporciona control total del OS al atacante.

A diferencia de los backdoors, **los rootkits NO pueden detectarse observando servicios, listas de tareas del sistema o registros.**

Ejemplos: Reptile (Linux, módulo kernel con reverse shell), ZeroAccess, Fire Chili, Purple Fox, MoonBounce.

### 5. E-Banking Trojans
Interceptan información de cuentas **antes de que el sistema la cifre** y la envían al C&C. Los atacantes programan umbrales mínimo y máximo de robo para no vaciar la cuenta. Crean screenshots del estado de cuenta para que la víctima no note el fraude.

**Mecanismos de robo:**
- **TAN Grabber:** intercepta TANs (Transaction Authentication Numbers) y los reemplaza por números aleatorios. El banco rechaza los aleatorios; el atacante usa el TAN legítimo interceptado.
- **HTML Injection:** crea campos de formulario falsos en páginas bancarias para recopilar datos de cuenta, tarjeta de crédito, fecha de nacimiento.
- **Form Grabber:** captura IDs y contraseñas de formularios del navegador. Analiza peticiones POST.
- **Covert Credential Grabber:** permanece dormido hasta que el usuario realiza una transacción financiera online. Edita entradas del registro en cada arranque. Al detectar la transacción, roba las credenciales.

Otros métodos: keylogging, captura de formularios, inserción de campos fraudulentos, screenshots/grabación de vídeo, imitación de sitios financieros, MITM.

Ejemplos: CHAVECLOAK, Grandoreiro, Ursnif (Gozi), DanaBot, QakBot.

### 6. Point-of-Sale (POS) Trojans
Atacan dispositivos POS y lectores de tarjetas. Roban las pistas **TRACK1 y TRACK2** de la banda magnética (número de tarjeta, nombre del titular, CVV). Una vez comprometidas, el atacante tiene control total de la tarjeta.

Ejemplo: **Prilex** (grupo brasileño). Genera criptogramas EMV para ejecutar transacciones fantasma evadiendo anti-fraude PIN/CHIP. Los atacantes se hacen pasar por técnicos POS.

Otros: LockPOS, BlackPOS, FastPOS.

### 7. Defacement Trojans
Destruyen o modifican el contenido completo de una base de datos o web. Modifican físicamente el HTML subyacente. Usan resource editors para modificar strings, bitmaps, logos e iconos de programas Windows. Emplean UCAs (User-styled Custom Applications).

### 8. Service Protocol Trojans
Explotan protocolos vulnerables: VNC, HTTP/HTTPS, ICMP.

**VNC Trojans:** inician un daemon de servidor VNC en el sistema infectado. El atacante conecta con cualquier visor VNC. Difícil de detectar porque VNC es una utilidad legítima.

**HTTP/HTTPS Trojans:** trabajan en **sentido inverso**. Bypasan firewalls usando puerto 80. El host interno (víctima) realiza una petición HTTP aparentemente legítima al servidor del atacante. La respuesta contiene comandos codificados en Base64. El tráfico se codifica como cgi-string para evitar detección.

**ICMP Trojans:** transportan payload en los campos de datos de paquetes `ICMP_ECHO` y `ICMP_ECHOREPLY`. Los dispositivos de capa de red y firewalls proxy **no filtran ni inspeccionan el contenido del tráfico ICMP_ECHO**, lo que hace este canal atractivo.

### 9. Mobile Trojans
Atacan móviles. Roban credenciales bancarias y de redes sociales, cifran datos y bloquean dispositivos.

Ejemplo: **Chameleon** (Android). Distribuido via phishing. Explota privilegios de Accessibility Service para ejecutar device takeover (DTO) y account takeover (ATO).

Otros: Vultur, PixPirate, GoldPickaxe, GoldDigger.

### 10. IoT Trojans
Atacan redes IoT. Aprovechan botnets para atacar máquinas fuera de la red IoT.

Ejemplo: **OpenSSH Trojan** (Microsoft Research). Explota versión modificada de OpenSSH para robar credenciales SSH, movimiento lateral y criptominería.

Otros: Ttint, XorDdos, Mozi, Silex BrickerBot, Gafgy Botnet, Dark Nexus.

### 11. Security Software Disabler Trojans
Detienen firewalls e IDS. Son **entry Trojans** que permiten al atacante lanzar el siguiente nivel de ataque.

Ejemplos: Chameleon, CertLock, GhostHook.

### 12. Destructive Trojans
Su único propósito es **borrar ficheros** en el sistema objetivo. Borran aleatoriamente ficheros, carpetas, entradas del registro y drives, a menudo causando fallo del OS. Escritos como batch files con `DEL`, `DELTREE` o `FORMAT`. Compilados como `.ini`, `.exe`, `.dll` o `.com`.

Ejemplos: SilverRAT, HermeticWiper, WhisperGate, FoxBlade.

### 13. DDoS Attack Trojans
Convierten a la víctima en zombie que escucha comandos de un DDoS Server. Cuando el servidor envía la orden simultáneamente a todos los infectados, inundan el objetivo.

El más notorio: **Mirai IoT botnet Trojan**.
Otros recientes: RDDoS, Horabot, hailBot, kiraiBot, catDDoS.

### 14. Command Shell Trojans
Proporcionan control remoto de un command shell en la máquina de la víctima. Un servidor Trojan se instala en la víctima abriendo un puerto; el cliente en la máquina del atacante lanza el shell.

Ejemplos: Netcat, DNS Messenger, GCat.

---

## 6. Proceso de Infección con un Troyano (7 pasos) 🔴

**Paso 1 — Crear el Trojan:** usar herramientas como **njRAT**, SET, THorse. Los troyanos nuevos no tienen firmas conocidas en los AV.

**Paso 2 — Emplear Dropper o Downloader:**
- **Dropper** (ej. Amadey, SecuriDropper): lleva el malware incrustado. Se ejecuta cargando su código en memoria, extrae el payload al sistema de ficheros, inicia la instalación. Incluye señuelos (imágenes, juegos) para distraer al usuario. Evade firewalls.
- **Downloader** (ej. Fruity Trojan, Downloader.DN, sLoad): NO lleva el malware, contiene el link para descargarlo. Puede pasar por escáneres anti-malware. Se distribuye como adjunto en email disfrazado (ej. `accounts.exe`, `invoices`).

**Paso 3 — Emplear Wrapper:** une el ejecutable del troyano con una app `.EXE` aparentemente genuina. Al ejecutar la app wrapeada, **primero instala el troyano en background y luego ejecuta la app legítima en foreground**. Herramienta: **IExpress Wizard**.

**Paso 4 — Emplear Crypter:** cifra el código binario del `.exe` para hacerlo indetectable por antivirus. Herramienta: **Attacker-Crypter**. Otros: Muck Crypter, Pure Crypter, DarkTortilla.

**Paso 5 — Propagar:** via canales overt/covert, exploit kits, emails, instant messengers.

**Paso 6 — Desplegar:** ejecutar el dropper o downloader en la máquina víctima. El fichero desplegado contiene el malware wrappeado y cifrado.

**Paso 7 — Ejecutar la Damage Routine:** entrega el payload final (borrar ficheros, reformatear discos, mostrar mensajes). Puede incluir **malware beaconing**.

---

## 7. Canales Overt vs Covert

**Canal Overt:** path de comunicación legítimo en la red. Sus componentes inactivos pueden explotarse para crear canales covert.

**Canal Covert:** path oculto e ilegal que viola la política de seguridad. Ejemplo: comunicación entre un troyano y su C&C. Usa **tunneling** (un protocolo transporta a otro: DNS, SSH, ICMP, HTTP/S). Método preferido para evadir antivirus y firewalls.

Herramientas para crear canales covert: Raccoon, QEMU, ELECTRICFISH.

---

## 8. Técnicas para Evadir Antivirus

1. Dividir el fichero en múltiples partes y comprimirlo como zip.
2. Escribir el troyano propio (sin firma en BD del AV).
3. Cambiar sintaxis: convertir EXE a VB script; cambiar extensión a `.DOC`, `.PPT`, `.PDF.EXE` (Windows oculta extensiones conocidas → muestra solo `.PDF`).
4. Cambiar contenido con editor hex.
5. Cambiar checksum y cifrar el fichero.
6. No usar troyanos descargados de la web (los AV los detectan fácilmente).
7. Usar binder/splitter para cambiar los primeros bytes del programa.
8. **Code obfuscation o morphing:** previene que el AV diferencie código malicioso de benigno.

---

## 9. Exploit Kits (Crimeware Toolkits)

Explotan vulnerabilidades en aplicaciones (Adobe Reader, Flash) para distribuir malware. Tienen código preescrito y son **fáciles de usar sin experiencia técnica**. Incluyen interfaz para rastrear estadísticas de infección y control remoto del sistema comprometido.

**Proceso de explotación:**
1. Víctima visita web legítima comprometida.
2. Es redirigida a través de servidores intermediarios.
3. Aterriza en el servidor del exploit kit (landing page).
4. El kit recopila info sobre la víctima y determina el exploit adecuado.
5. Si el exploit tiene éxito, se descarga y ejecuta el malware.

**BotenaGo** (escrito en Go, 30+ variantes de exploits, ataca IoT y routing devices). Inserta backdoor via puerto 31412; escucha respuesta por puerto 19412. Sin comunicación activa con C&C durante la explotación.

Otros exploit kits: RIG, Magnitude, Angler, Fallout, Nuclear, Neutrino, Underminer, Terror, Sundown.

---

## 3. Exam Traps ⚠️

⚠️ **[RAT vs Backdoor: interfaz de usuario]** — El RAT tiene un componente cliente con **interfaz de usuario** para emitir comandos. El backdoor NO tiene interfaz. Ambos dan acceso remoto pero el RAT es más funcional para el atacante.

⚠️ **[Rootkit: NO se propaga solo]** — Los rootkits no se propagan por sí mismos. Requieren un dropper para instalarse. Son parte de blended threat: dropper + loader + rootkit.

⚠️ **[Rootkit vs Backdoor: detección]** — Backdoors pueden detectarse observando servicios, tareas del sistema o registros. Los rootkits NO porque ocultan su presencia a nivel del OS.

⚠️ **[Blended Threat: dropper se elimina, loader causa buffer overflow]** — El dropper lanza el loader y luego se elimina. El loader causa un buffer overflow para cargar el rootkit en memoria. El examen puede preguntar el orden o qué hace cada componente.

⚠️ **[TAN Grabber: reemplaza por número aleatorio]** — Intercepta el TAN válido y lo reemplaza por un número aleatorio. El banco rechaza el aleatorio; el atacante usa el TAN legítimo. El examen puede invertir el mecanismo.

⚠️ **[E-banking Trojan: umbrales de robo]** — Programados para robar entre un mínimo y un máximo para no vaciar la cuenta y evitar sospechas. También crean screenshots del estado de cuenta.

⚠️ **[HTTP/HTTPS Trojan: sentido inverso]** — La víctima inicia la conexión hacia el servidor del atacante, no al revés. Esta inversión bypasa el firewall.

⚠️ **[ICMP Tunnel: firewalls no inspeccionan ICMP_ECHO]** — Los firewalls de capa de red y proxy NO filtran ni inspeccionan el contenido del tráfico ICMP_ECHO.

⚠️ **[Dropper vs Downloader: ¿lleva el malware?]** — Dropper: lleva el malware incrustado. Downloader: NO lleva el malware, solo tiene el link. El downloader puede pasar por escáneres anti-malware.

⚠️ **[Wrapper: orden de ejecución]** — Primero instala el troyano en background, luego ejecuta la app legítima en foreground.

⚠️ **[Puerto 31337: Back Orifice]** — Troyano clásico CEH asociado al puerto 31337.

⚠️ **[Puerto 445: WannaCry/Petya]** — WannaCry y Petya usan el puerto 445 (SMB).

⚠️ **[Extensiones falsas con Windows]** — Windows oculta extensiones conocidas por defecto. `.PDF.EXE` aparece como `.PDF`.

⚠️ **[Exploit kit: sin experiencia técnica]** — Tienen código preescrito y son fáciles de usar para atacantes sin experiencia en IT o seguridad.

---

## 4. Nemotécnicos

**14 tipos de troyanos:**
> **"RAT Back Bot Root E-Bank POS Defac Service Mobile IoT SecDis Destr DDoS CmdShell"**

**7 pasos de infección:**
> **"Create → Drop → Wrap → Crypt → Prop → Deploy → Damage"**

**Banking Trojan mechanisms (4):**
> **"TAN-HTML-Form-Covert"**
> TAN Grabber → HTML Injection → Form Grabber → Covert Credential Grabber

**Blended Threat (rootkit):**
> **"Drop → Load → Root"**
> Dropper (instala loader y se elimina) → Loader (buffer overflow) → Rootkit

**RAT vs Backdoor:**
> **"RAT = Remote + UI | Backdoor = Remote, NO UI"**

---

## 5. Flashcards

**Q:** ¿Cuál es la diferencia principal entre un RAT y un Backdoor Trojan?
**A:** El RAT tiene un componente cliente con interfaz de usuario para emitir comandos. El backdoor no tiene interfaz de usuario.

**Q:** ¿Por qué los rootkits no pueden detectarse observando servicios y listas de tareas?
**A:** Porque ocultan su presencia a nivel del OS, manipulando las estructuras de datos del kernel.

**Q:** ¿Qué tres componentes forman una blended threat que instala un rootkit?
**A:** Dropper (instala el loader y se elimina) → Loader (buffer overflow para cargar rootkit en memoria) → Rootkit.

**Q:** ¿Qué hace un TAN Grabber?
**A:** Intercepta el TAN válido y lo reemplaza por un número aleatorio. El banco rechaza el aleatorio; el atacante usa el TAN legítimo interceptado.

**Q:** ¿Por qué los banking Trojans programan límites mínimo y máximo de robo?
**A:** Para no vaciar la cuenta y evitar que la víctima o el banco noten el fraude.

**Q:** ¿Qué información de la tarjeta roban los POS Trojans?
**A:** Las pistas TRACK1 y TRACK2 de la banda magnética: número de tarjeta, nombre del titular y CVV.

**Q:** ¿En qué sentido trabajan los HTTP/HTTPS Trojans?
**A:** En sentido inverso: la víctima inicia la conexión hacia el servidor del atacante, bypassando el firewall.

**Q:** ¿Por qué el ICMP es atractivo para tunneling de troyanos?
**A:** Los firewalls de capa de red y proxy no filtran ni inspeccionan el contenido del tráfico ICMP_ECHO.

**Q:** ¿Qué diferencia un Dropper de un Downloader?
**A:** El dropper lleva el malware incrustado. El downloader no lo lleva, solo contiene el link para descargarlo.

**Q:** ¿Cuál es el troyano asociado al puerto 31337?
**A:** Back Orifice / Back Orifice 1.20.

**Q:** ¿Qué troyanos usan el puerto 445?
**A:** WannaCry y Petya (también Dragonfly 2.0).

**Q:** ¿Qué hace un Wrapper en el proceso de instalación?
**A:** Une el ejecutable del troyano con una app legítima. Primero instala el troyano en background, luego ejecuta la app legítima en foreground.

**Q:** ¿Qué herramienta de wrapper menciona explícitamente el libro CEH?
**A:** IExpress Wizard.

**Q:** ¿Cuál es el IoT botnet Trojan más notorio para DDoS?
**A:** Mirai IoT botnet Trojan.

**Q:** ¿Qué es un canal Covert en el contexto de troyanos?
**A:** Path oculto e ilegal para transferir datos violando la política de seguridad. Usa tunneling (un protocolo sobre otro). Ejemplo: comunicación C&C via DNS o ICMP.

**Q:** ¿Qué hace la damage routine de un troyano?
**A:** Entrega el payload final: borrar ficheros, reformatear discos, mostrar mensajes. Puede incluir malware beaconing.

---

## 6. Confusión frecuente

**RAT vs Backdoor** → RAT: control total con interfaz cliente (GUI, keylogger, cámara). Backdoor: acceso remoto sin interfaz. Ambos dan acceso remoto pero el RAT es más funcional y requiere una interfaz de control del atacante.

**Rootkit vs Backdoor: detección** → Backdoor: detectable observando servicios/tareas/registros. Rootkit: NO detectable así porque manipula el OS para ocultarse.

**Dropper vs Downloader** → Dropper: lleva el malware consigo (detectable con forense). Downloader: solo tiene el link (más sigiloso inicialmente ante escáneres).

**Botnet Trojan vs DDoS Trojan** → Botnet Trojan: crea la red de bots para múltiples propósitos (DoS, spam, click fraud, robo). DDoS Trojan: específicamente diseñado para ataques DDoS.

**E-Banking Form Grabber vs TAN Grabber** → Form Grabber: captura credenciales estáticas de formularios web. TAN Grabber: intercepta TANs de un solo uso en transacciones activas y los sustituye por números aleatorios.

**Blended Threat dropper vs componente Dropper del malware** → En blended threat del rootkit: Dropper instala el loader y se elimina. En taxonomía general: Dropper lleva malware incrustado directamente. Mismo término, contextos ligeramente distintos.

**ICMP Tunnel vs DNS Tunnel** → ICMP Tunnel: datos en paquetes echo/reply, ignorado por firewalls de red. DNS Tunnel: datos en queries/respuestas DNS, ignorado porque rara vez se inspecciona en profundidad.

**Exploit Kit vs Trojan** → Exploit Kit: herramienta de distribución con exploits preescritos. Trojan: payload entregado tras la explotación. El exploit kit es el vehículo; el troyano es la carga.

---

## 7. Preguntas de Práctica — Formato CEH

**P1.** Un CISO investiga una intrusión en la que el atacante podía ver la pantalla de la víctima, registrar sus pulsaciones de teclado y activar la cámara web remotamente. ¿Qué tipo de troyano usó el atacante?

A) Backdoor Trojan  
B) Remote Access Trojan (RAT)  
C) Proxy Server Trojan  
D) Rootkit Trojan  

**Respuesta correcta: B**
Un **RAT (Remote Access Trojan)** proporciona control total del sistema de la víctima incluyendo: acceso remoto a la pantalla (VNC-like), keylogger, acceso a cámara/micrófono, y ejecución de comandos arbitrarios. Un Backdoor da acceso remoto pero sin la interfaz GUI y funcionalidades completas del RAT.

---

**P2.** Un analista de malware descubre que un troyano redirige todo el tráfico web de la víctima a través del sistema infectado, convirtiendo el equipo comprometido en un intermediario para actividades del atacante. ¿Qué tipo de troyano es?

A) HTTP/HTTPS Trojan  
B) Banking Trojan  
C) Proxy Server Trojan  
D) Defacement Trojan  

**Respuesta correcta: C**
Un **Proxy Server Trojan** convierte el sistema infectado en un **servidor proxy** que enruta el tráfico del atacante a través de la víctima. Esto permite al atacante ocultar su dirección IP real usando la dirección de la víctima como intermediario para actividades ilegales.

---

**P3.** Un pentester crea un ejecutable malicioso que incluye el payload dentro del propio fichero y lo entrega directamente a la víctima. ¿Cómo se denomina este componente del troyano y cuál es su diferencia con un Downloader?

A) Downloader — descarga el payload desde Internet  
B) Dropper — lleva el payload consigo (es detectable con forense)  
C) Crypter — cifra el payload para evadir detección  
D) Injector — inyecta el payload en un proceso legítimo  

**Respuesta correcta: B**
Un **Dropper** lleva el malware **consigo** en el propio ejecutable y lo instala en el sistema. Un **Downloader** solo contiene el enlace de descarga y obtiene el payload desde Internet después de ejecutarse. El Dropper es más detectable forense pero no requiere acceso a internet en el momento de la infección.

---

**P4.** ¿En qué puerto TCP opera el troyano **NetBus** según la tabla de puertos de troyanos del CEH?

A) Puerto 1234  
B) Puerto 12345  
C) Puerto 6667  
D) Puerto 31337  

**Respuesta correcta: B**
**NetBus** opera en el **puerto 12345** (TCP). Este dato es específicamente preguntado en el examen CEH. Puerto 6667 es IRC (usado por algunos bots C2). Puerto 31337 es Back Orifice. Puerto 1234 es Ultors Trojan.

---

**P5.** Un troyano comunica con su servidor C2 usando el protocolo DNS para enviar datos, ocultando el tráfico malicioso dentro de consultas DNS aparentemente legítimas. ¿Qué tipo de canal usa?

A) Canal Overt  
B) Canal Covert  
C) Canal HTTP  
D) Canal cifrado  

**Respuesta correcta: B**
Un **canal Covert** es un path oculto e ilegal para transferir datos que viola la política de seguridad. Usa tunneling (un protocolo sobre otro — en este caso datos C2 sobre DNS). El DNS tunneling es un ejemplo clásico de canal covert. Un canal Overt es visible, conocido y explícito (ej. enviar un email normal).

---

**P6.** Un analista forense analiza la carpeta `Startup` de Windows y el registro del sistema en busca de malware. Encuentra una entrada sospechosa pero el troyano no es visible en ningún proceso ni servicio del sistema. ¿Qué tipo de troyano presenta este comportamiento?

A) Backdoor Trojan  
B) Remote Access Trojan  
C) Rootkit Trojan  
D) Defacement Trojan  

**Respuesta correcta: C**
Un **Rootkit Trojan** manipula el **sistema operativo** para ocultar su presencia — no aparece en listas de procesos, servicios ni registros visibles. A diferencia de un Backdoor (detectable observando servicios y registros), el rootkit intercepta las llamadas al sistema para ocultar sus artefactos.

---

**P7.** Un script kiddie quiere crear un troyano combinando un programa legítimo con un ejecutable malicioso en un único fichero. ¿Qué tipo de herramienta CEH se usa para este propósito?

A) RAT builder  
B) Exploit kit  
C) Trojan binder/wrapper  
D) Crypter  

**Respuesta correcta: C**
Un **Trojan binder/wrapper** une un programa legítimo (ej. juego, screensaver) con el ejecutable malicioso en un único fichero. Cuando la víctima ejecuta el programa legítimo, el troyano se instala en segundo plano. Es diferente de un crypter (que cifra el malware) y de un exploit kit (que automatiza la explotación).

---

**P8.** Un Banking Trojan intercepta transacciones bancarias online, modifica los datos de la transacción en tiempo real sin que la víctima lo detecte. ¿Cuál es la técnica técnica que usa?

A) Form grabbing  
B) Man-in-the-Browser (MitB)  
C) Keylogging de credenciales  
D) SSL stripping  

**Respuesta correcta: B**
La técnica es **Man-in-the-Browser (MitB)**: el troyano se inyecta en el navegador e intercepta y modifica las transacciones **en tiempo real entre el navegador y el servidor bancario** antes de que los datos sean enviados. La víctima ve la transacción correcta en pantalla mientras el troyano envía datos modificados al banco.
