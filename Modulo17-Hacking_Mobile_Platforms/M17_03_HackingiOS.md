# M17_03 — Hacking iOS
**Módulo 17 / Subapartado 3 — Hacking iOS**

---

## 1. Conceptos y definiciones

### Arquitectura iOS — 5 capas 🔴

iOS actúa como intermediario entre las apps y el hardware subyacente mediante interfaces de sistema bien definidas. La UI se basa en **direct manipulation** con gestos multi-touch.

| Capa (de arriba a abajo) | Contenido principal |
|--------------------------|---------------------|
| Cocoa Application | Frameworks clave para construir apps iOS; apariencia, infraestructura, multitasking, notificaciones push, touch-based input. Usa el **AppKit framework**. |
| Media | Gráficos, audio y vídeo; tecnologías multimedia |
| Core Services | Servicios fundamentales del sistema: Core Foundation, Foundation frameworks; redes, iCloud, localización, social media |
| Core OS | Funciones de bajo nivel: seguridad, comunicación con hardware externo y redes; depende de Kernel y Device Drivers |
| Kernel and Device Drivers | Capa más baja: kernel, drivers, BSD, file systems, infraestructura de red |

**Regla de capas iOS:** las capas inferiores contienen servicios fundamentales; las superiores construyen sobre ellas para proporcionar servicios más sofisticados.

---

### Jailbreaking iOS

**Definición:** proceso de instalar un conjunto modificado de parches del kernel que permite ejecutar aplicaciones de terceros no firmadas por el fabricante del OS. Implica:
- Bypass de las limitaciones de usuario impuestas por Apple
- Modificación del OS, obtención de privilegios de administrador
- Instalación de apps no aprobadas oficialmente mediante **side loading**
- Acceso root al OS
- Eliminación de las restricciones del sandbox → las apps maliciosas pueden acceder a recursos e información restringidos

**Riesgos (idénticos a Android rooting):** anulación de garantía, rendimiento degradado, infección por malware, bricking del dispositivo.

**Herramientas:** Hexxa Plus, Sileem, checkra1n, palera1n, Redensa, Zeon, Sileo, Cydia.

---

### Tipos de Jailbreak — por exploit 🔴

| Tipo | Loophole explotado | Nivel de acceso | ¿Parcheable con firmware? |
|------|--------------------|-----------------|---------------------------|
| **Userland Exploit** | Aplicación del sistema | User-level (sin iboot-level) | Sí (solo firmware updates) |
| **iBoot Exploit** | iBoot (3er bootloader del dispositivo) | User-level + iboot-level | Sí (firmware updates) |
| **Bootrom Exploit** | SecureROM (1er bootloader del dispositivo) | User-level + iboot-level | **No** (solo hardware update por Apple) |

**Clave diferenciadora:** Bootrom Exploit es el único que **no puede parcharse con actualizaciones de firmware**; requiere actualización de hardware del bootrom por Apple.

---

### Técnicas de Jailbreak — por persistencia 🔴

| Técnica | Comportamiento tras reinicio | Necesita ordenador para reiniciar |
|---------|------------------------------|-----------------------------------|
| **Untethered** | El kernel se parchea automáticamente en cada arranque; el dispositivo sigue jailbroken tras cada reinicio | No |
| **Semi-tethered** | Arranca completamente pero **sin kernel parcheado**; funcional para uso normal; para usar addons jailbroken necesita arrancar con la herramienta de jailbreak | Sí (para funciones jailbroken) |
| **Tethered** | Si arranca solo → sin kernel parcheado → puede quedarse en estado parcialmente iniciado; debe ser "re-jailbroken" con un ordenador en **cada** arranque | Sí (obligatoriamente) |
| **Semi-untethered** | Similar a semi-tethered; el kernel no se parchea al reiniciar; pero se puede parchear **desde una app instalada en el dispositivo** (sin ordenador) | No |

**Diferencia clave semi-tethered vs semi-untethered:** ambos requieren acción adicional para parchear el kernel tras reinicio, pero semi-tethered necesita un ordenador y semi-untethered usa una app del propio dispositivo.

---

### iOS Trustjacking

Vulnerabilidad que explota la función **iTunes Wi-Fi Sync**. La víctima conecta su iPhone a un ordenador de confianza (infectado por el atacante) y pulsa "Trust" en el cuadro de diálogo del dispositivo.

**Consecuencias tras pulsar "Trust" una sola vez:**
- La comunicación entre el dispositivo y el ordenador infectado **continúa después de desconectar físicamente el cable**
- El atacante puede leer mensajes, emails, capturas de pantalla y actividad del dispositivo
- El sistema infectado puede hacer backup/restore para leer historial de SMS, fotos eliminadas y apps
- Puede reemplazar apps originales con apps maliciosas desde el PC conectado previamente
- La monitorización persiste **aunque el dispositivo salga de la zona de comunicación**, hasta que el teléfono resetea la configuración de conexión

**Vector:** el atacante no necesita estar físicamente presente en el momento del ataque; basta con haber infectado el ordenador previamente.

---

### SeaShell Framework — Post-explotación iOS

Framework de post-explotación iOS que explota la vulnerabilidad **CoreTrust** para bypass de verificaciones de seguridad y ejecución de software no autorizado. Usa **TrollStore** para explotar bugs de CoreTrust e instalar software malicioso.

**Payload:** Pwny (con extensión dinámica y cifrado TLS).

**Flujo del ataque:**
```bash
seashell                                    # Lanzar el framework
ipa patch Instagram.ipa                     # Parchear un IPA con IP y puerto del listener
listener on <IP> <Puerto>                   # Iniciar listener (se recibe conexión al abrir la app)
devices -i <id>                             # Interactuar con el dispositivo comprometido
safari_history                              # Extraer historial de navegación
# Base de datos ubicada en: /var/mobile/Library/Safari/
```

---

### Cycript — Manipulación en Runtime

Cycript es un **intérprete JavaScript** que entiende Objective-C, Objective-C++ y JS. Se usa para manipular la funcionalidad de apps iOS en tiempo de ejecución tras descompilación y análisis del código fuente.

Capacidades:
- **Method swizzling** (= monkey patching): modificar métodos existentes o añadir funcionalidad en runtime
- Authentication bypass
- Jailbreak detection bypass
- Logging, JS injections en WebViews

**Method swizzling — pasos:**
1. Identificar el selector del método existente a intercambiar
2. Crear un nuevo método con funcionalidad personalizada
3. Ejecutar la app en el dispositivo
4. Intercambiar la funcionalidad proporcionando la referencia del nuevo método al Objective-C runtime

---

### objection — Análisis y manipulación iOS 🔴

objection incorpora la funcionalidad de **Frida** y soporta method hooking, SSL pinning bypass, iOS keychain dumping y monitorización del portapapeles.

**Comandos clave:**
```bash
# Conectar al target
objection --gadget <AppName> explore

# Method hooking
ios hooking watch class <Class_Name>                        # Monitorizar llamadas de métodos de una clase
ios hooking watch method "-[Class_Name Method_Name]"        # Hook a método específico
ios hooking set return_value "-[Class_Name iFunction:]" true/false  # Cambiar valor de retorno (Boolean)

# SSL pinning bypass
ios sslpinning disable

# Jailbreak detection bypass
ios jailbreak disable
```

---

### Keychain Dumper

El Keychain de iOS es un sistema de almacenamiento cifrado que guarda passwords, certificados y claves de cifrado. Keychain Dumper usa un binario con **certificado autofirmado y wildcard entitlement** para volcar los keychains. En versiones recientes de iOS el wildcard entitlement no está permitido; es necesario añadir un entitlement explícito que exista en el dispositivo para acceder a todos los items del keychain.

---

### Análisis de dispositivos iOS — Comandos y Técnicas

**Acceso al shell (via SSH — requiere jailbreak + Cydia/Sileo):**
```bash
# Via Wi-Fi (mismo segmento de red)
ssh root@<device_ip_address>
# Credenciales por defecto: usuarios "root" y "mobile", contraseña "alpine"

# Via USB (usando usbmuxd/iproxy en macOS)
ssh -p 2222 root@localhost
# Nota: USB Restricted Mode limita la conexión de datos a máximo 1 hora en estado bloqueado
```

**Listar apps instaladas:**
```bash
frida-ps -Uai          # -a: todas, -i: instaladas, -U: dispositivo USB conectado
```

**Network sniffing (via rvictl en macOS):**
```bash
rvictl -s <UDID_del_dispositivo>    # Crea interfaz virtual "rvi0"
# Luego abrir Wireshark y seleccionar interfaz "rvi0"
# Filtro ejemplo: ip.addr == 192.168.2.4 && http
```

**Conexiones abiertas:**
```bash
lsof -i                    # Puertos de red abiertos para todos los procesos activos
lsof -i -a -p <pid>        # Puertos de red abiertos para un proceso específico
```

**Process exploration (r2frida):**
```bash
r2 frida://usb//iGoat-Swift    # Iniciar sesión r2frida
:dm                             # Recuperar mapas de memoria de la app
:il                             # Listar binarios y librerías cargadas
\e~search                       # Búsqueda en memoria (oculta progreso, muestra solo resultados)
```

---

### iOS Malware destacado

- **GoldPickaxe** (Trojan): engaña a víctimas para que escaneen su cara y su ID oficial. Se distribuye mediante smishing/phishing haciéndose pasar por fuente gubernamental legítima. En iOS obliga a instalar un **perfil MDM** (Mobile Device Management) falso → explotación de remote wiping, device tracking y app management para instalar apps dañinas y robar datos bancarios.
- Pegasus, LightSpy, KingsPawn, SpectralBlur, Mercenary Spyware.

---

### Herramientas iOS — mapa rápido

| Herramienta | Función |
|-------------|---------|
| Cycript | Runtime manipulation; JS interpreter para Obj-C; method swizzling, auth bypass |
| objection | Method hooking, SSL pinning bypass, keychain dump, portapapeles (integra Frida) |
| Keychain Dumper | Extraer keychains cifrados del dispositivo |
| SeaShell Framework | Post-explotación iOS; explota CoreTrust; payload Pwny |
| r2frida | Process exploration: mapas de memoria, librerías, búsqueda en memoria |
| Frida | Análisis dinámico; listar apps (`frida-ps -Uai`); hooking |
| Elcomsoft Phone Breaker | Adquisición lógica de iOS, break de backups cifrados, descifrar iCloud Keychain |
| Spyzie | Hackeo remoto de SMS, call logs, GPS; **sin jailbreak** |
| checkra1n / palera1n | Herramientas de jailbreak |

---

### GoldPickaxe — MDM como vector de ataque

El perfil MDM instalado en iOS proporciona al atacante capacidades de administración remota del dispositivo, incluyendo:
- Remote wiping
- Device tracking
- App management (instalar apps dañinas)
- Recopilar información deseada

El mecanismo de distribución es smishing/phishing con suplantación de fuente gubernamental.

---

## 2. Exam Traps ⚠️ 🔴

⚠️ **[Arquitectura iOS — número de capas]** iOS tiene **5 capas**, no 4 ni 6. Android tiene 5 capas con 6 secciones; iOS tiene exactamente 5 capas. No confundir la arquitectura de ambos sistemas.

⚠️ **[Bootrom Exploit — parcheable o no]** Es el único tipo de jailbreak que **no se puede parchear con actualizaciones de firmware**. Requiere actualización de hardware del bootrom por Apple. Userland e iBoot exploits sí se parchean con firmware updates.

⚠️ **[Semi-tethered vs Semi-untethered — diferencia clave]** Ambos requieren acción adicional para parchear el kernel tras reinicio. La diferencia: semi-tethered necesita **un ordenador** con la herramienta de jailbreak; semi-untethered usa **una app en el propio dispositivo** (sin ordenador).

⚠️ **[Tethered jailbreak — consecuencia crítica]** Con tethered jailbreak, si el dispositivo arranca solo (sin ordenador), puede quedarse en un **estado parcialmente iniciado**. No simplemente "pierde el jailbreak"; puede quedar inutilizable sin un ordenador.

⚠️ **[iOS Trustjacking — protocolo explotado]** Trustjacking explota **iTunes Wi-Fi Sync**, no Bluetooth ni NFC. El vector es conectar físicamente el dispositivo a un ordenador infectado y pulsar "Trust". La comunicación persiste post-desconexión física.

⚠️ **[Cycript — lenguaje]** Cycript es un intérprete **JavaScript** que entiende también Objective-C y Objective-C++. No es un intérprete Python ni un decompilador; es para manipulación en runtime.

⚠️ **[Method swizzling = monkey patching]** El examen puede usar cualquiera de los dos nombres. Son sinónimos. Implica modificar métodos existentes o añadir funcionalidad en runtime mediante el Objective-C runtime.

⚠️ **[Credenciales SSH por defecto en iOS jailbroken]** Las credenciales por defecto son usuario **root** (o mobile) con contraseña **alpine**. Si el examen pregunta por la contraseña por defecto de un dispositivo iOS con jailbreak: "alpine". Una contraimedida explícita en la guía de seguridad es **cambiar esta contraseña**.

⚠️ **[USB Restricted Mode]** Limita la conexión de datos a **máximo 1 hora** en estado bloqueado. No impide la conexión física, solo la transferencia de datos.

⚠️ **[GoldPickaxe — vector iOS específico]** Para iOS usa un **perfil MDM** (no una app directamente), porque iOS restringe la instalación de apps de terceros. Para Android usaría una app directamente. El MDM es el vector que bypasea las restricciones del App Store en iOS.

⚠️ **[objection vs Cycript]** Cycript = manipulación en runtime (JS interpreter, method swizzling). objection = suite más completa que integra Frida; soporta hooking, SSL pinning bypass, jailbreak detection bypass, keychain dump.

⚠️ **[Erase Data — número de intentos]** La función "Erase Data" de iOS borra todos los datos y ajustes tras **10 intentos fallidos** de contraseña. Dato numérico específico para el examen.

---

## 3. Nemotécnicos

### Arquitectura iOS — 5 capas (de arriba a abajo)
**"CoMCoCoK"**: **Co**coa → **M**edia → **Co**re Services → **Co**re OS → **K**ernel & Device Drivers

Alternativa visual: "Como Mickey Come Cosas Ketchup"

### Tipos de jailbreak por exploit (3) — ¿qué bootloader?
- **U**serland → aplicación del **U**suario (sin iboot)
- i**B**oot → **B**ootloader #3 (iBoot)
- **B**ootrom → **B**ootloader #1 (SecureROM) → **no parcheable**

Regla: cuanto más bajo el bootloader, más persistente y menos parcheable.

### Técnicas de jailbreak — persistencia tras reinicio
**"USTS"** (orden de más a menos autónomo):
- **U**ntethered = siempre jailbroken
- **S**emi-untethered = app en dispositivo parchea kernel
- **S**emi-tethered = necesita ordenador para funciones jailbroken
- **T**ethered = necesita ordenador obligatoriamente (puede quedar "brick")

### Credenciales iOS por defecto
**"Root Alpine"** — usuario: root (o mobile), contraseña: **alpine**

### SeaShell — flujo de 5 pasos
**"Launch → Patch → Listen → Interact → Extract"**

### objection — 3 bypasses clave
**"SSL-Jail-Return"**: `ios sslpinning disable` → `ios jailbreak disable` → `ios hooking set return_value`

---

## 4. Flashcards

**Q:** ¿Cuántas capas tiene la arquitectura iOS y cuál es la más baja?
**A:** **5 capas**. La más baja es **Kernel and Device Drivers** (kernel, drivers, BSD, file systems, networking).

---

**Q:** ¿Qué tipo de jailbreak exploit no puede ser parcheado con actualizaciones de firmware y por qué?
**A:** **Bootrom Exploit**. Explota la SecureROM (primer bootloader del dispositivo). Solo puede ser parcheado mediante una actualización de hardware del bootrom por Apple.

---

**Q:** ¿Cuál es la diferencia entre jailbreak semi-tethered y semi-untethered?
**A:** Ambos requieren acción adicional para parchear el kernel tras reinicio. Semi-tethered necesita **un ordenador** con la herramienta de jailbreak. Semi-untethered usa **una app instalada en el propio dispositivo** (sin ordenador).

---

**Q:** ¿Qué ocurre en un jailbreak tethered si el dispositivo arranca sin estar conectado a un ordenador?
**A:** El dispositivo puede quedarse en un **estado parcialmente iniciado** (parcialmente arrancado). Debe ser "re-jailbroken" con un ordenador usando la función "boot tethered" en cada encendido.

---

**Q:** ¿Qué función de iOS explota Trustjacking y qué acción del usuario activa la vulnerabilidad?
**A:** Explota **iTunes Wi-Fi Sync**. La vulnerabilidad se activa cuando el usuario conecta su dispositivo a un ordenador infectado y pulsa **"Trust"** en el cuadro de diálogo del dispositivo.

---

**Q:** ¿Qué puede hacer el atacante tras un Trustjacking exitoso incluso después de desconectar el cable?
**A:** Leer mensajes, emails, monitorizar actividad y capturas de pantalla; hacer backup/restore para leer SMS e historial, fotos eliminadas y apps; reemplazar apps originales con versiones maliciosas. La monitorización persiste aunque el dispositivo salga de la zona de comunicación.

---

**Q:** ¿Qué vulnerabilidad explota el SeaShell Framework y qué payload usa?
**A:** Explota la vulnerabilidad **CoreTrust** para bypass de verificaciones de seguridad. Usa el payload **Pwny** con extensión dinámica y cifrado TLS, ejecutado mediante TrollStore.

---

**Q:** ¿Cuáles son las credenciales SSH por defecto de un dispositivo iOS con jailbreak?
**A:** Usuario: **root** (o mobile), contraseña: **alpine**.

---

**Q:** ¿Qué es Cycript y para qué se usa en el contexto del hacking iOS?
**A:** Intérprete **JavaScript** que entiende Objective-C, Objective-C++ y JS. Se usa para manipular funcionalidad de apps iOS en runtime: method swizzling, authentication bypass, jailbreak detection bypass.

---

**Q:** ¿Qué es method swizzling y cómo se denomina también?
**A:** También conocido como **monkey patching**. Técnica que modifica métodos existentes o añade funcionalidad en runtime mediante el Objective-C runtime. Pasos: identificar selector → crear nuevo método → ejecutar app → intercambiar funcionalidad.

---

**Q:** ¿Qué hace el comando `ios jailbreak disable` en objection?
**A:** Deshabilita la funcionalidad de **detección de jailbreak** en la app hooked, permitiendo que la app siga funcionando en dispositivos jailbroken aunque tenga mecanismos de detección activos.

---

**Q:** ¿Qué hace `ios sslpinning disable` en objection?
**A:** Deshabilita el SSL pinning en la app hooked, permitiendo interceptar el tráfico HTTPS sin que la app rechace el certificado del proxy.

---

**Q:** ¿Qué vector usa GoldPickaxe para infectar dispositivos iOS (no Android)?
**A:** Instala un **perfil MDM** (Mobile Device Management) falso haciéndose pasar por una fuente gubernamental legítima (via smishing/phishing). El MDM permite al atacante instalar apps dañinas y gestionar el dispositivo remotamente.

---

**Q:** ¿Cómo se crea la interfaz virtual para sniffing de red en iOS desde macOS?
**A:** Con el comando `rvictl -s <UDID_del_dispositivo>`, que crea la interfaz **rvi0**. Luego se abre Wireshark seleccionando esa interfaz.

---

**Q:** ¿Cuántos intentos fallidos de contraseña activan el "Erase Data" en iOS?
**A:** **10 intentos fallidos**.

---

**Q:** ¿Qué hace Elcomsoft Phone Breaker en el contexto del hacking iOS?
**A:** Adquisición lógica y over-the-air de dispositivos iOS; break de backups cifrados con aceleración GPU; descifrar iCloud Keychain; obtener y analizar backups, datos sincronizados y contraseñas de iCloud.

---

**Q:** ¿Qué comando r2frida recupera los mapas de memoria de una app iOS?
**A:** `:dm` (dentro de una sesión r2frida iniciada con `r2 frida://usb//iGoat-Swift`).

---

**Q:** ¿Qué limitación impone el USB Restricted Mode de iOS?
**A:** Limita la conexión de datos a un máximo de **1 hora** en estado bloqueado. Esto impide el acceso via ADB/SSH prolongado en dispositivos bloqueados.

---

**Q:** ¿Qué iBoot es la SecureROM y en qué se diferencia de iBoot?
**A:** SecureROM es el **primer bootloader** (iBoot #1) del dispositivo; es el más bajo en la cadena de arranque y no puede ser parcheado por software. iBoot es el **tercer bootloader** y sí puede ser parcheado mediante actualizaciones de firmware.

---

## 5. Confusión frecuente

### Tipos de jailbreak exploit: Userland vs iBoot vs Bootrom
- **Userland:** loophole en app del sistema; solo user-level; parcheado con firmware.
- **iBoot:** loophole en iBoot (3er bootloader); user + iboot level; parcheado con firmware.
- **Bootrom:** loophole en SecureROM (1er bootloader); user + iboot level; **NO parcheable con firmware**.
- **Criterio:** ¿menciona SecureROM o primer bootloader? → Bootrom. ¿No se puede parchear con updates? → Bootrom.

---

### Untethered vs Semi-untethered
- **Untethered:** kernel se parchea automáticamente en cada reinicio; completamente autónomo.
- **Semi-untethered:** el kernel NO se parchea solo al reiniciar; necesita una app en el dispositivo para parchear. El dispositivo arranca y funciona normalmente, pero las funciones jailbroken requieren lanzar la app primero.
- **Criterio:** ¿jailbroken automáticamente tras cada reinicio? → Untethered. ¿Requiere app en el dispositivo? → Semi-untethered.

---

### iOS Trustjacking vs MITM
- **MITM:** intercepción de comunicaciones de red entre dos sistemas.
- **Trustjacking:** explota la confianza establecida entre el dispositivo iOS y un ordenador infectado mediante iTunes Wi-Fi Sync. No intercepta red; explota una sesión de confianza ya establecida físicamente. Persiste tras la desconexión física.
- **Criterio:** ¿sesión "Trust" establecida con iTunes Wi-Fi Sync? → Trustjacking.

---

### Cycript vs objection
- **Cycript:** intérprete JS/Obj-C para manipulación en runtime; más básico; enfocado en method swizzling y modificación de código.
- **objection:** suite completa que integra Frida; soporta hooking, SSL pinning bypass, jailbreak detection bypass, keychain dump, monitorización de portapapeles.
- **Criterio:** ¿solo runtime manipulation/method swizzling? → Cycript. ¿Suite completa con bypass de protecciones múltiples? → objection.

---

### Arquitectura iOS vs Android
- **iOS:** 5 capas (Cocoa → Media → Core Services → Core OS → Kernel & Drivers).
- **Android:** 5 capas / 6 secciones (System Apps → Java API → Native Libs + ART → HAL → Linux Kernel).
- **Criterio:** ¿menciona Cocoa o AppKit? → iOS. ¿Menciona HAL o ART? → Android.

---

### GoldPickaxe en iOS vs Android
- **iOS:** instala un perfil **MDM** falso (no puede instalar apps directamente).
- **Android:** instala directamente una app maliciosa.
- **Criterio:** ¿perfil MDM como vector? → iOS. ¿App directa? → Android.
