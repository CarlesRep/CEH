# M17_01 — Mobile Platform Attack Vectors
**Módulo 17 / Subapartado 1 — Mobile Platform Attack Vectors**

---

## 1. Conceptos y definiciones

### OWASP Top 10 Mobile Risks (2024) 🔴

El OWASP Mobile Top 10 de 2024 no es una lista de ataques, sino de **categorías de riesgo por fallos de diseño e implementación**. Cada categoría tiene una causa raíz, un vector de explotación y un impacto diferenciados.

| ID | Nombre | Causa raíz principal | Impacto clave |
|----|--------|----------------------|---------------|
| M1 | Improper Credential Usage | Credenciales hardcodeadas, almacenamiento sin protección, canales no cifrados | Acceso no autorizado, data breach |
| M2 | Inadequate Supply Chain Security | Componentes de terceros desactualizados/vulnerables, firma de app insegura | Inserción de backdoors, compromiso del backend |
| M3 | Insecure Authentication/Authorization | Políticas débiles de contraseña, tokens inseguros, checks de autorización ausentes | Suplantación, bypass de autenticación |
| M4 | Insufficient Input/Output Validation | Sin sanitización de inputs, lógica de aplicación errónea | SQLi, XSS, command injection, crashes |
| M5 | Insecure Communication | Protocolos deprecados, SSL incorrecto, cifrado débil | Intercepción, impersonation, PII leaks |
| M6 | Inadequate Privacy Controls | Control de acceso a datos pobre, incumplimiento de leyes de privacidad | Robo de identidad, multas regulatorias |
| M7 | Insufficient Binary Protections | Sin protección anti-tampering, sin ofuscación | Reverse engineering, apps falsas, acceso a features de pago |
| M8 | Security Misconfiguration | Debugging habilitado, permisos innecesarios, credenciales por defecto | Account hijacking, acceso a backend |
| M9 | Insecure Data Storage | Plain text, BBDDs sin protección, cifrado débil | Data breach, tampering, identity theft |
| M10 | Insufficient Cryptography | Algoritmos débiles, gestión de claves deficiente, RNG inseguro | Bypass cifrado, acceso a cuentas, compromiso de datos |

---

### Anatomía de un ataque móvil — Las tres capas

El modelo de ataque móvil se estructura en **tres superficies**. Entender esto es crítico porque el examen pregunta qué capa corresponde a cada técnica.

#### Capa 1: El dispositivo

**Browser-based:**
- **Phishing** → páginas falsas que imitan sitios legítimos; los móviles son más vulnerables por la visualización truncada de URLs e iconos de candado reducidos.
- **Framing** → web embebida mediante iFrames HTML; el atacante explota el iFrame del sitio legítimo para superponer su página maliciosa + clickjacking.
- **Clickjacking** (= UI redress attack) → el usuario hace clic creyendo interactuar con algo legítimo; el atacante captura datos o toma control del dispositivo.
- **Man-in-the-Mobile (MitMo)** → malware implantado en el dispositivo que intercepta OTPs enviadas por SMS/voz y las reenvía al atacante. Diseñado específicamente para bypass de 2FA.
- **Buffer Overflow** → escritura más allá del límite del buffer → sobrescritura de memoria adyacente → crashes, errores de acceso, comportamiento errático.
- **Data Caching** → explotación de las cachés de datos del navegador/app que almacenan información para mejorar el rendimiento.

**Phone/SMS-based:**
- **Baseband Attacks** → explotan vulnerabilidades en el procesador baseband GSM/3GPP que gestiona las comunicaciones de radio con las torres celulares.
- **SMiShing** → SMS phishing; el atacante envía enlaces maliciosos o números de tarificación especial por SMS haciéndose pasar por entidad legítima.

**Application-based:**
- Sensitive Data Storage, No/Weak Encryption, Improper SSL Validation, Configuration Manipulation, Dynamic Runtime Injection, Unintended Permissions, Escalated Privileges, UI overlay/pin stealing, Intent Hijacking, ZIP Directory Traversal, Side-channel attacks.

**OS-based:**
- **iOS Jailbreaking** → elimina los mecanismos de seguridad de Apple, proporciona acceso root y elimina las restricciones del sandbox.
- **Android Rooting** → obtención de control privilegiado (root access) en el subsistema Android; equivalente funcional al jailbreaking en iOS.
- **OS Data Caching** → el atacante reinicia el dispositivo víctima con un OS malicioso y vuelca la memoria para extraer datos sensibles.
- **Carrier-loaded Software** → apps preinstaladas por el operador que pueden contener vulnerabilidades explotables.
- **User-initiated Code** → ingeniería social que lleva al usuario a instalar apps maliciosas o hacer clic en enlaces que instalan código malicioso.

#### Capa 2: La red

| Técnica | Mecanismo |
|---------|-----------|
| Wi-Fi (weak/no encryption) | Interceptación por eavesdropping sobre wireless; aunque usen SSL/TLS, ataques contra estos protocolos pueden exponer datos |
| Rogue Access Point | AP ilegítimo instalado físicamente para secuestrar conexiones de usuarios legítimos |
| Packet Sniffing | Captura y análisis de paquetes (Wireshark, Capsa Free Network Analyzer); objetivo: credenciales en claro |
| MITM | Intercepción de conexiones existentes; lectura/modificación/inserción de datos |
| Session Hijacking | Robo de session IDs válidos para acceso no autorizado |
| DNS Poisoning | Sustitución de IPs legítimas por IPs del atacante a nivel DNS |
| SSLStrip | MITM que degrada conexiones HTTPS a HTTP sin cifrado; difícil de detectar en navegadores móviles porque el usuario debe validar la presencia del HTTPS |
| Fake SSL Certificates | MITM con certificado SSL falso para interceptar tráfico HTTPS |

#### Capa 3: Data Center / Cloud

**Web server:**
- Platform Vulnerabilities (OS, IIS, módulos de aplicación)
- Server Misconfiguration
- XSS → inyección de scripts client-side en páginas dinámicas
- CSRF → peticiones HTTP no intencionadas desde la sesión del usuario hacia un sitio de confianza
- Weak Input Validation → el servidor confía en que la app cliente ya validó; el atacante forja comunicaciones directas al servidor
- Brute-Force

**Database:**
- SQL Injection → comandos SQL a través de inputs no validados → acceso/extracción directa de BD
- Privilege Escalation → acceso elevado mediante exploit → robo de datos sensibles de BD
- Data Dumping → volcado parcial o total de la BD
- OS Command Execution → inyección de comandos OS en queries → acceso root al servidor

---

### Bluetooth: Bluesnarfing vs Bluebugging

Ambas explotan conexiones Bluetooth abiertas, pero con objetivos distintos:

**Bluesnarfing:** robo de información (contactos, emails, SMS, fotos, vídeos, datos de negocio) desde un dispositivo Bluetooth con la conexión en modo "discoverable". Explota vulnerabilidades en el software del fabricante. El atacante accede sin conocimiento del usuario.

**Bluebugging:** acceso remoto completo al dispositivo Bluetooth para usar sus funciones (hacer/recibir llamadas, interceptar SMS, reenviar llamadas, conectarse a Internet, acceder a contactos/fotos/vídeos). Implica un ataque de backdoor antes de devolver el control al propietario.

---

### SS7 y Simjacker

**SS7 (Signaling System 7):** protocolo de comunicación que permite el roaming entre redes celulares. Opera bajo un modelo de **confianza mutua entre operadores sin autenticación**. Al no estar aislada la red SS7, el atacante puede:
- Realizar MITM sobre llamadas y SMS
- Interceptar credenciales bancarias y OTPs
- Bypass de 2FA y cifrado extremo a extremo vía SMS
- Exposición de identidad de suscriptor, tapping, DoS al operador, tracking de ubicación geográfica

**Simjacker:** vulnerabilidad en el **S@T browser (SIMalliance Toolbox Browser)**, software preinstalado en la SIM card. El ataque se inicia enviando código spyware oculto en un SMS (STK - SIM Application Toolkit). La SIM lo procesa automáticamente sin interacción del usuario. Permite: captura de ubicación, monitorización de llamadas, obtención del IMEI, llamadas premium, conexión del navegador a sitios maliciosos, DoS sobre la SIM.

**Flujo del ataque Simjacker:**
1. Atacante envía SMS fraudulento con código/instrucciones STK ocultos
2. El S@T browser en la SIM reconoce y procesa automáticamente las instrucciones
3. El código inyectado ejecuta actividades sin consentimiento del usuario
4. El dispositivo cómplice recibe la información del usuario vía SMS

---

### Agent Smith Attack

El atacante publica apps maliciosas disfrazadas de juegos, editores de fotos u otras herramientas en tiendas de terceros (ej. 9Apps). Al instalarlas, el código malicioso interno **infecta o reemplaza apps legítimas** (WhatsApp, SHAREit, MX Player) con versiones infectadas. A veces se presenta como producto auténtico de Google (Google Updater, Themes). Objetivo: generar publicidad fraudulenta masiva para beneficio económico + robo de credenciales y datos bancarios vía C&C.

---

### OTP Hijacking / 2FA Hijacking

Los OTPs se envían por SMS, app autenticadora o email. Los métodos de secuestro incluyen:
- **Social engineering sobre telecom providers:** el atacante convence al operador de transferir el control de la SIM de la víctima (SIM swap) alegando pérdida del dispositivo.
- **SIM Jacking:** infección de la SIM con malware para interceptar y leer OTPs.
- **Lock screen notifications:** robo físico del OTP visualizando las notificaciones en pantalla bloqueada.
- Herramientas: **AdvPhishing** (bypass de 2FA en redes sociales, compatible con Linux/Termux, usa NGrok tunnelling), **mrphish** (bash script, funciona en Android rooteado y no rooteado).

---

### Camfecting vs Android Camera Hijack

**Camfecting:** el atacante instala un RAT (Remote Access Trojan) en el dispositivo (PC o móvil) via phishing/sitio malicioso → acceso remoto a cámara y micrófono; puede deshabilitar la luz de la cámara para evitar detección.

**Android Camera Hijack:** explota vulnerabilidades de bypass de permisos en Android para acceder a cámara/micrófono incluso con el dispositivo bloqueado. Se propaga mediante app maliciosa que instala un Troyano. La conexión persiste aunque el usuario cierre la aplicación. Permisos explotados: `android.permission.CAMERA`, `android.permission.RECORD_AUDIO`, `android.permission.ACCESS_COARSE_LOCATION`, `android.permission.ACCESS_FINE_LOCATION`.

Herramienta destacada: **StormBreaker** — accede a ubicación, webcam y micrófono sin solicitar permisos explícitos.

---

### App Sandboxing

El sandbox limita los recursos accesibles por una app a su funcionalidad prevista, aislándola del resto del sistema y de otras apps. Un sandbox vulnerable permite que una app maliciosa explote vulnerabilidades para traspasar el perímetro y acceder a datos/recursos de otras apps o del sistema.

---

### SMiShing — Por qué es efectivo

Alta tasa de apertura de SMS (vs email), mayor confianza del usuario en SMS, limitación de caracteres (dificulta identificar señales de phishing), menor concienciación sobre riesgos en SMS, dispositivos móviles sin software de seguridad robusto, facilidad de spoofing de número, creación de urgencia, URLs acortadas (ocultan destino malicioso), ausencia de filtros anti-phishing nativos en SMS.

---

## 2. Exam Traps ⚠️ 🔴

⚠️ **[MitMo vs MITM]** El examen puede presentar ambos como equivalentes. MitMo (Man-in-the-Mobile) es específico de dispositivos móviles: implanta malware para interceptar OTPs en el propio dispositivo. MITM es un ataque de red que intercepta comunicaciones entre dos sistemas. Son capas diferentes (dispositivo vs red).

⚠️ **[Bluesnarfing vs Bluebugging]** El examen pregunta cuál permite "tomar control del dispositivo". La respuesta es **Bluebugging** (acceso remoto a funciones del dispositivo). Bluesnarfing solo roba datos (información estática almacenada). Criterio: ¿control activo? → Bluebugging. ¿Robo de datos almacenados? → Bluesnarfing.

⚠️ **[SSLStrip]** El examen lo clasifica como MITM, no como un ataque de cifrado. SSLStrip no rompe SSL: lo elimina degradando la conexión a HTTP. El usuario debe validar la presencia de HTTPS para detectarlo; en móviles esto es especialmente difícil.

⚠️ **[Simjacker vs SS7]** Simjacker ataca la SIM card vía S@T browser mediante SMS con código STK. SS7 explota el protocolo de señalización de red. El examen puede preguntar cuál permite bypass de 2FA vía SMS: **ambos**, pero Simjacker es el ataque a la SIM, SS7 es el ataque a la red.

⚠️ **[Jailbreaking vs Rooting]** Jailbreaking = iOS, elimina mecanismos de seguridad de Apple + acceso root + elimina sandbox. Rooting = Android, obtiene control privilegiado (root access). El examen puede preguntar cuál elimina el sandbox: **jailbreaking** (iOS). Android sandbox no se elimina explícitamente con rooting.

⚠️ **[Agent Smith Attack — vector de distribución]** El examen especificará tiendas de terceros. Agent Smith se distribuye desde **third-party app stores** (ej. 9Apps), no desde Google Play. La característica distintiva es que **reemplaza apps legítimas** con versiones infectadas (no las elimina).

⚠️ **[Camfecting vs Android Camera Hijack]** Camfecting = RAT en PC/móvil vía phishing. Android Camera Hijack = bypass de permisos en Android específicamente, funciona con el dispositivo **bloqueado**, conexión persiste tras cerrar la app. Si el escenario menciona "dispositivo bloqueado", la respuesta es Android Camera Hijack.

⚠️ **[M1 vs M3 OWASP Mobile]** M1 (Improper Credential Usage) trata el almacenamiento y transmisión inseguros de credenciales. M3 (Insecure Authentication/Authorization) trata la implementación deficiente de los mecanismos de autenticación. El examen puede usar "hardcoded credentials" → M1, "weak password policy" → M3.

⚠️ **[Framing vs Clickjacking]** Son técnicas relacionadas pero distintas. Framing embebe una web en otra mediante iFrame. Clickjacking (= UI redress attack) engaña al usuario para que haga clic en algo diferente de lo que cree. El framing puede ser un mecanismo para ejecutar clickjacking, pero no son sinónimos.

⚠️ **[OTP Hijacking — método de detección]** El usuario generalmente no detecta el ataque porque interpreta la no recepción del OTP como un problema de red, no como un ataque. Este es el motivo por el que es difícil de detectar.

---

## 3. Nemotécnicos

### OWASP Mobile Top 10 — 2024
**"I-SA-IA-IC-SI"** (orden M1→M10):
> **I**mproper Credentials → **S**upply Chain → **A**uth/Authorization → **I**nput Validation → **C**ommunication → **P**rivacy → **B**inary → **M**isconfiguration → **D**ata Storage → **C**ryptography

Alternativa por inicial: **IC-SA-AI-PC-BD-MC** → agrupa en pares por temática:
- Credenciales + Supply Chain (M1-M2) → **problemas de origen**
- Auth + Input (M3-M4) → **problemas de validación**
- Communication + Privacy (M5-M6) → **problemas de exposición**
- Binary + Misconfiguration (M7-M8) → **problemas de configuración**
- Data Storage + Cryptography (M9-M10) → **problemas de protección de datos**

### Capas del ataque móvil
**"DNA"**: **D**evice → **N**etwork → **A**pplication/Data Center

### Bluetooth attacks
- **Snarfing = Steal** (robo de datos almacenados)
- **Bugging = Bug/Backdoor** (acceso remoto + control activo)

### SS7 Threats (6 riesgos)
**"IRSPDT"**: **I**dentity exposure → **R**evealing network identity → **S**pying/Interception → **P**hone tapping → **D**oS → **T**racking location

### Simjacker — 4 pasos
**"EPIA"**: **E**nvía SMS STK → **P**rocesa automáticamente el S@T browser → **I**nicia actividades sin consentimiento → **A**ctúa el dispositivo cómplice recibe info vía SMS

### Permisos Android Camera Hijack (4)
**"CARA"**: **C**AMERA → **A**UDIO (RECORD) → **R**ecord (ya incluido) → **A**CCESS LOCATION (COARSE + FINE)
Simplificado: cámara, audio, ubicación gruesa, ubicación fina.

---

## 4. Flashcards

**Q:** ¿Qué es Man-in-the-Mobile (MitMo) y cuál es su objetivo principal?
**A:** Malware implantado en el dispositivo móvil de la víctima que intercepta OTPs enviados por SMS/voz y los reenvía al atacante, con el objetivo de bypass de 2FA/verificación por contraseña de un solo uso.

---

**Q:** ¿Cuál es la diferencia entre Bluesnarfing y Bluebugging?
**A:** Bluesnarfing = robo de datos almacenados (contactos, SMS, fotos) sin conocimiento del usuario desde un dispositivo en modo discoverable. Bluebugging = acceso remoto completo al dispositivo para usar sus funciones (llamadas, SMS, Internet); implica un backdoor.

---

**Q:** Un atacante degrada silenciosamente una conexión HTTPS a HTTP en un navegador móvil. ¿Qué ataque es?
**A:** SSLStrip, un tipo de MITM que elimina el cifrado degradando HTTPS a HTTP; difícil de detectar en móviles porque el usuario debe verificar activamente la presencia de HTTPS.

---

**Q:** ¿Qué protocolo explota SS7 y cuál es su vulnerabilidad fundamental?
**A:** SS7 (Signaling System 7), protocolo de comunicación para roaming entre redes celulares. Su vulnerabilidad fundamental es operar bajo confianza mutua entre operadores **sin autenticación**, lo que permite MITM sobre llamadas/SMS y bypass de 2FA.

---

**Q:** ¿Qué es Simjacker y qué componente de la SIM explota?
**A:** Vulnerabilidad en el **S@T browser (SIMalliance Toolbox Browser)** preinstalado en la SIM. El atacante envía un SMS con código STK oculto que la SIM procesa automáticamente sin interacción del usuario.

---

**Q:** ¿Cuál es el vector de distribución principal del Agent Smith Attack y qué hace al instalarse?
**A:** Se distribuye desde **tiendas de apps de terceros** (ej. 9Apps). Al instalarse, reemplaza apps legítimas (WhatsApp, SHAREit, MX Player) con versiones infectadas para mostrar publicidad fraudulenta y robar datos vía C&C.

---

**Q:** ¿Qué diferencia un Camfecting attack de un Android Camera Hijack attack?
**A:** Camfecting usa un RAT instalado vía phishing/sitio malicioso, aplica a PC y móvil. Android Camera Hijack explota vulnerabilidades de bypass de permisos en Android, funciona con el dispositivo **bloqueado** y la conexión persiste aunque el usuario cierre la app.

---

**Q:** ¿Qué categoría OWASP Mobile 2024 corresponde a credenciales hardcodeadas en el código de la app?
**A:** M1 — Improper Credential Usage.

---

**Q:** ¿Qué herramienta usa un atacante para capturar cámara/micrófono sin solicitar permisos explícitos?
**A:** **StormBreaker** (GitHub). Accede a ubicación, webcam y micrófono del dispositivo sin solicitar permisos de forma explícita.

---

**Q:** Un atacante instala un AP inalámbrico ilegítimo físicamente en la red para secuestrar conexiones legítimas. ¿Qué técnica es?
**A:** Rogue Access Point.

---

**Q:** ¿Qué es Clickjacking y cómo se denomina también en el contexto del CEH?
**A:** Técnica maliciosa que engaña al usuario para que haga clic en algo diferente de lo que cree. También conocida como **UI redress attack** (ataque de redireccionamiento de interfaz de usuario).

---

**Q:** ¿Qué es SMiShing y por qué es más efectivo que el phishing por email?
**A:** SMS phishing; envío de SMS fraudulentos con enlaces maliciosos o números premium. Es más efectivo porque los SMS tienen mayor tasa de apertura, generan más confianza, carecen de filtros anti-phishing nativos, y los usuarios tienen menos concienciación sobre riesgos en SMS.

---

**Q:** ¿Qué hace el iOS Jailbreaking en términos de seguridad?
**A:** Elimina los mecanismos de seguridad de Apple, proporciona acceso root al OS y elimina las restricciones del sandbox, exponiendo el dispositivo a malware, pérdida de datos sensibles y rendimiento degradado.

---

**Q:** ¿Qué categoría OWASP Mobile cubre la protección de apps contra reverse engineering y tampering?
**A:** M7 — Insufficient Binary Protections.

---

**Q:** ¿Cuál es el objetivo de un ataque de DNS Poisoning en el contexto móvil?
**A:** Sustituir IPs legítimas por IPs del atacante a nivel DNS, redirigiendo a los usuarios a sitios web controlados por el atacante.

---

**Q:** ¿Cuáles son los 4 permisos Android que explota el Android Camera Hijack Attack?
**A:** `android.permission.CAMERA`, `android.permission.RECORD_AUDIO`, `android.permission.ACCESS_COARSE_LOCATION`, `android.permission.ACCESS_FINE_LOCATION`.

---

**Q:** ¿Qué herramientas se usan para packet sniffing en ataques a redes móviles según el CEH?
**A:** **Wireshark** y **Capsa Free Network Analyzer**.

---

**Q:** Un atacante explota vulnerabilidades del procesador que gestiona las comunicaciones de radio con torres celulares. ¿Qué ataque es?
**A:** Baseband Attack — explota el procesador baseband GSM/3GPP del teléfono.

---

**Q:** ¿Qué es el ataque de Framing en el contexto mobile y qué tecnología HTML utiliza?
**A:** Integración de una página web maliciosa dentro de otra usando elementos **iFrame** de HTML. El atacante explota la funcionalidad iFrame del sitio objetivo para superponer su contenido y ejecutar clickjacking para robar información sensible.

---

**Q:** ¿Por qué el OTP Hijacking es difícil de detectar por la víctima?
**A:** Porque la víctima interpreta la no recepción del OTP como un problema de red, sin sospechar que ha sido redirigido al dispositivo del atacante.

---

## 5. Confusión frecuente

### Bluesnarfing vs Bluebugging
- **Bluesnarfing:** roba datos almacenados (contactos, fotos, SMS). Acceso puntual. Sin control activo del dispositivo.
- **Bluebugging:** toma control remoto del dispositivo para usar sus funciones (llamadas, mensajes, Internet). Implica backdoor.
- **Criterio:** si el escenario dice "accedió a los contactos/fotos" → Bluesnarfing. Si dice "realizó llamadas desde el dispositivo de la víctima" → Bluebugging.

---

### SS7 vs Simjacker
- **SS7:** vulnerabilidad en el **protocolo de red** de señalización de telecomunicaciones; no requiere interacción con el dispositivo físico; afecta a toda la red.
- **Simjacker:** vulnerabilidad en el **S@T browser de la SIM card**; se explota enviando un SMS con código STK al dispositivo objetivo.
- **Criterio:** ¿protocolo de red/roaming? → SS7. ¿SMS con código a la SIM? → Simjacker.

---

### Camfecting vs Android Camera Hijack
- **Camfecting:** usa RAT; aplica a PC y móvil; requiere phishing o visita a sitio malicioso; deshabilita la luz de cámara.
- **Android Camera Hijack:** bypass de permisos específico de Android; funciona con dispositivo **bloqueado**; la conexión persiste aunque se cierre la app.
- **Criterio:** ¿dispositivo bloqueado o conexión persistente tras cerrar app? → Android Camera Hijack. ¿RAT multiplataforma? → Camfecting.

---

### MitMo vs MITM
- **MITM (Man-in-the-Middle):** ataque de **red** que intercepta comunicaciones entre dos sistemas.
- **MitMo (Man-in-the-Mobile):** ataque sobre el **dispositivo** que implanta malware para interceptar OTPs en el propio dispositivo y reenviarlos al atacante.
- **Criterio:** ¿intercepción de tráfico de red? → MITM. ¿Bypass de OTP/2FA desde el dispositivo móvil infectado? → MitMo.

---

### App Sandboxing: sandbox seguro vs vulnerable
- **Sandbox seguro:** la app tiene privilegios limitados a su funcionalidad; no puede acceder a datos de otras apps ni recursos del sistema.
- **Sandbox vulnerable:** una app maliciosa explota vulnerabilidades del sandbox, traspasa su perímetro y accede a datos/recursos de otras apps o del sistema.
- **Criterio:** el sandbox no es infalible; el examen puede preguntar qué ocurre cuando se "bypasea" → acceso a datos y recursos fuera del alcance previsto de la app.

---

### M9 (Insecure Data Storage) vs M10 (Insufficient Cryptography)
- **M9:** datos sensibles guardados en plain text, BBDDs sin protección, cifrado débil **en almacenamiento**.
- **M10:** uso de algoritmos de cifrado débiles u obsoletos, mala gestión de claves, vulnerabilidades en implementación criptográfica **en el proceso de cifrado/descifrado**.
- **Criterio:** ¿dónde se guardan los datos sin protección? → M9. ¿El cifrado usado es débil o está mal implementado? → M10.
