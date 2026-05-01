# M16_04_WirelessHackingMethodology.md
> Módulo 16 / Subapartado 4 — Wireless Hacking Methodology

---

## 1. Conceptos y definiciones

### 🔴 5 Fases de la metodología de hacking wireless

| Fase | Descripción |
|---|---|
| **1. Wi-Fi Discovery** | Localizar la red wireless objetivo (footprinting, chalking) |
| **2. Wireless Traffic Analysis** | Analizar el tráfico para determinar vulnerabilidades, víctimas y estrategia |
| **3. Launch of Wireless Attacks** | Fragmentation, MAC spoofing, DoS, ARP poisoning, etc. |
| **4. Wi-Fi Encryption Cracking** | Crackear WPA/WPA2/WPA3 |
| **5. Wi-Fi Network Compromising** | Comprometer la red y acceder a recursos |

---

### Fase 1: Wi-Fi Discovery

#### Footprinting: Métodos pasivo y activo

| Método | Mecanismo | Características |
|---|---|---|
| **Passive Footprinting** | Snifea paquetes del airwaves para detectar APs, dispositivos y SSIDs | No conecta con APs ni inyecta paquetes; totalmente silencioso |
| **Active Footprinting** | Envía probe request con SSID (o SSID vacío) al AP y espera respuesta | La mayoría de APs responde a probe request vacío con su propio SSID; el AP puede configurarse para ignorar probe requests vacíos |

SSID está presente en: beacons, probe requests, probe responses, association requests, re-association requests.

#### 🔴 Wi-Fi Chalking Techniques

| Técnica | Descripción |
|---|---|
| **WarWalking** | Camina con laptop Wi-Fi + herramienta de discovery para mapear redes abiertas |
| **WarChalking** | Dibuja símbolos en lugares públicos para advertir sobre redes Wi-Fi abiertas |
| **WarFlying** | Usa drones para detectar redes inalámbricas abiertas |
| **WarDriving** | Conduce con laptop Wi-Fi + herramienta de discovery para mapear redes abiertas |

#### Herramientas de Wi-Fi Discovery

| Herramienta | Características |
|---|---|
| **inSSIDer** | Optimización y troubleshooting Wi-Fi; inspecciona WLAN y redes circundantes; rastrea señal en dBm; exporta a KML para Google Earth; muestra canales solapados |
| **Sparrow-wifi** | GUI-based, 2.4 y 5 GHz; integra HackRF (SDR), Ubertooth (Bluetooth), GPS; importa/exporta CSV y JSON; produce Google Maps |
| **Wash** | Identifica APs con WPS activado; verifica si el AP está bloqueado; soporta 5 GHz (`-5` flag) |

**Comando Wash para descubrir APs:**
```bash
sudo wash -i wlan0
```
Descubre AP, ESSID y BSSID de dispositivos/routers.

**Nota sobre WPS:** la mayoría de routers WPS se bloquean automáticamente tras **>5 intentos incorrectos consecutivos**; solo pueden desbloquearse manualmente en la interfaz admin.

---

### Fase 2: Wireless Traffic Analysis

Para sniffar tráfico wireless, el atacante necesita habilitar el **monitor mode** en su Wi-Fi card.

**No todos los adaptadores Wi-Fi soportan monitor mode en Windows.** Verificar en: `https://secwiki.org/w/Npcap/WiFi_adapters`

**Windows:** puede escuchar tráfico de red pero **NO** puede inyectar paquetes.
**Linux:** puede tanto escuchar como **inyectar** paquetes.

**Herramientas de sniffing wireless:**

| Herramienta | Características |
|---|---|
| **Wireshark** | Captura y análisis; soporta Ethernet, Token Ring, 802.11, PPP, SLIP, ATM, FDDI; integración con **Npcap** para análisis completo WLAN; captura management frames, control frames, data frames; analiza Radiotap header fields |
| **CommView for Wi-Fi** | Monitor y analyzer para 802.11 a/b/g/n; descifra paquetes con WPA-PSK keys; muestra APs, stations, estadísticas por nodo/canal, signal strength |
| **Kismet** | Sniffing wireless |

---

### 🔴 Aircrack-ng Suite — Herramientas y funciones

| Herramienta | Función |
|---|---|
| **Airbase-ng** | Captura WPA/WPA2 handshake; actúa como AP ad-hoc |
| **Aircrack-ng** | Herramienta de cracking de WEP y WPA/WPA2 PSK (de facto) |
| **Airdecap-ng** | Descifra WEP/WPA/WPA2; elimina wireless headers de paquetes Wi-Fi |
| **Airdrop-ng** | De-autenticación dirigida y basada en reglas de usuarios |
| **Aireplay-ng** | Recopila IVs WEP y handshakes WPA; efectivo para inyección de paquetes |
| **Airgraph-ng** | Crea gráficos de relaciones cliente-AP y probe comunes desde archivo airodump |
| **Airmon-ng** | Cambia interfaz wireless de managed mode a **monitor mode** y viceversa |
| **Airodump-ng** | Captura raw 802.11 frames; recopila WEP IVs |
| **Airolib-ng** | Almacena y gestiona listas de ESSID y contraseñas para cracking WPA/WPA2 |
| **Airtun-ng** | Crea interfaz de túnel virtual para monitorizar tráfico cifrado e inyectar tráfico arbitrario |

---

### Fase 3: Launch of Wireless Attacks

#### Detección de SSIDs Ocultos

```bash
# 1. Poner en modo monitor
airmon-ng start <Wireless interface>
airmon-ng check kill    # si hay procesos problemáticos

# 2. Descubrir SSIDs
airodump-ng <interface>

# 3. Brute-force del SSID oculto con mdk3
mdk3 <Wireless Interface> p -b 1 -c <Channel> -t <Target BSSID>
```

Parámetros mdk3: `p`=Basic probing + ESSID brute-force, `-b`=Beacon flood, `1`=EAPOL logoff, `-c`=Canal, `-t`=Target BSSID.

#### DoS Wireless

Mecanismo: transmite comandos de **de-autenticación** que fuerzan a los clientes a desconectarse del AP.

- **Disassociation Attack:** destruye conectividad entre AP y cliente (targeted)
- **De-authentication Attack:** inunda stations con de-authenticates/disassociates forjados

#### 🔴 MITM Attack — Pasos

1. Atacante snifea parámetros wireless de la víctima (MAC address, ESSID/BSSID, canales)
2. Envía request **DEAUTH** a la víctima con dirección origen spoofada del AP legítimo
3. La víctima se de-autentica y busca un nuevo AP válido en todos los canales
4. Atacante configura **forged AP** en nuevo canal con BSSID y ESSID originales del AP legítimo
5. La víctima se conecta al AP falso
6. Atacante engaña a la víctima para que conecte al AP original (posicionándose entre ambos)
7. El atacante escucha todo el tráfico

**MITM con Aircrack-ng:**
```bash
airmon-ng start <interface>      # monitor mode
airodump-ng <interface>          # descubrir SSIDs
aireplay-ng <params>             # de-autenticar cliente
aireplay-ng <params>             # fake association con el AP
```

#### MAC Spoofing Attack

**AP MAC Spoofing:** el atacante identifica la MAC del AP legítimo → configura rogue AP con misma MAC y SSID → envía deauth para forzar reconexión al rogue AP.

**Client MAC Spoofing:** cambia la MAC para bypassar MAC filtering del AP.
- Linux: `ifconfig <interface> hw ether aa:bb:cc:dd:ee:ff` (con sudo)
- Herramientas: **Technitium MAC Address Changer**, **LizardSystems Change MAC Address**

#### Wireless ARP Poisoning

Herramienta: **Ettercap** (GUI).

**Pasos con Ettercap:**
1. Sniff → Unified Sniffing → seleccionar interfaz primaria
2. Hosts → Scan for Hosts → Hosts → Hosts List
3. View → Connections
4. Targets → Current Targets → seleccionar hosts objetivo
5. MITM → ARP poisoning → Sniff remote connections → OK

Si el tráfico no está cifrado con HTTPS → se pueden sniffar credenciales de login.

#### 🔴 Rogue AP — Instalación y Setup

**Tipos de instalación:**
1. Rogue AP compacto conectado a puerto Ethernet de la red corporativa
2. Rogue AP conectado vía Wi-Fi a la red corporativa (requiere credenciales de la red)
3. **USB-based rogue AP** en máquina Windows conectada (no requiere puerto Ethernet libre ni credenciales Wi-Fi)
4. Software-based rogue AP en adaptador Wi-Fi embebido de la máquina objetivo

**Pasos de despliegue:**
1. Elegir ubicación para máxima cobertura
2. **Deshabilitar SSID broadcast** (silent mode) y funciones de gestión para evitar detección
3. Colocar detrás de firewall para evitar network scanners
4. Desplegar por período corto

**Creación con MANA Toolkit:**
```bash
# 1. Modificar hostapd-mana.conf (interface, BSSID/SSID)
# 2. Modificar start-nat-simple.sh (phy=wlan0, upstream=eth0)
# 3. Ejecutar
bash <Path>/mana-toolkit/run-mana/start-nat-simple.sh
```

#### Evil Twin

El atacante configura un AP falso que imita el SSID de un AP legítimo. Herramienta: **KARMA** — escucha pasivamente probe request frames y adopta cualquier SSID como el suyo propio.

Setup de fake hotspot:
1. Habilitar Internet Connection Sharing (Windows) o Internet Sharing (macOS)
2. Broadcast la conexión Wi-Fi y ejecutar sniffer para capturar contraseñas

#### 🔴 KRACK — Key Reinstallation Attack

**Explota el four-way handshake de WPA2** forzando la **reutilización de nonces**. El atacante captura la ANonce ya en uso para manipular y reproducir mensajes del handshake criptográfico.

**Alcance:** funciona contra WPA y WPA2, personal y enterprise, ciphers WPA-TKIP, AES-CCMP y GCMP. Sistemas vulnerables: Android, Linux, Windows, Apple, OpenBSD, MediaTek.

**Permite robar:** números de tarjeta de crédito, contraseñas, mensajes, emails, fotos.

#### Jamming Signal Attack

**Mecanismo:** volúmenes abrumadores de tráfico malicioso → DoS para usuarios autorizados. Las señales de jamming aparecen como ruido → los dispositivos retienen sus transmisiones hasta que la señal cesa.

**Impacto en 802.11:** al ser un protocolo CSMA/CA, los algoritmos de collision-avoidance requieren un período de silencio antes de transmitir → el jammer hace imposible transmitir.

Herramienta: dispositivos jammer de hardware especializado (PCB-4510, CPB-2920, etc.).

#### aLTEr Attack

**Objetivo:** dispositivos LTE/4G que usan AES-CTR (sin protección de integridad).

**Mecanismo:** el atacante instala una torre falsa entre dos endpoints reales → intercepta y manipula tráfico usando DNS spoofing → redirige a víctima a sitios maliciosos.

**Capa afectada:** Layer 2 (data link layer).

**Dos fases:**
1. **Information gathering:** Identity mapping (localizar el dispositivo objetivo) + Website fingerprinting (registrar tráfico del usuario)
2. **Attack phase:** MITM con torre falsa + DNS spoofing → capturar credenciales

#### Wi-Jacking Attack

Obtiene acceso a redes WPA/WPA2 **sin necesidad de crackear el handshake**. Requiere:
- Cliente activo conectado a la red objetivo
- El cliente debe haberse conectado antes a alguna red abierta y permitir reconexión automática
- Browser **chromium-based** con credenciales de interfaz admin del router almacenadas
- Router con **HTTP no cifrado** para interfaz de configuración

**Pasos:**
1. Enviar de-auth a la víctima con aireplay-ng
2. KARMA attack con hostapd-wpe (lure a red maliciosa)
3. Inyectar URL maliciosa con dnsmasq + Python scripts
4. El browser carga la URL con credenciales almacenadas automáticamente
5. Usar XMLHttpRequest para login al router y extraer WPA2 PSK

#### RFID Cloning

Captura datos de un RFID tag legítimo y crea un clon. Cambia el TID (Tag ID); el form factor y datos pueden mantenerse. El clon puede detectarse fácilmente.

Herramientas: **iCopy-X** (standalone sin PC), **RFIDler**, **Flipper Zero**.

---

### Fase 4: Wi-Fi Encryption Cracking

#### 🔴 WPA/WPA2 Cracking — Técnicas

| Técnica | Descripción |
|---|---|
| **WPA PSK** | La contraseña no puede crackearse directamente (es per-packet key); pero se puede brute-force con dictionary attacks |
| **Offline attack** | Captura el WPA/WPA2 authentication handshake (unos segundos cerca del AP) → crackeo offline |
| **De-authentication attack** | Fuerza desconexión del cliente → captura el paquete de autenticación al reconectar (incluye PMK → crackeable por dictionary/brute-force) |
| **Brute forcing** | Compute-intensive; puede llevar horas, días o semanas |

**WPA handshake:** el protocolo NO envía la contraseña en la red (aunque el handshake ocurre en texto plano).

#### 🔴 Proceso de cracking WPA/WPA2 con Aircrack-ng

```bash
# 1. Monitor mode
airmon-ng start <interface>
airmon-ng check kill    # si hay procesos conflictivos

# 2. Listar APs
airodump-ng <interface>

# 3. Capturar paquetes del AP objetivo
airodump-ng --bssid <BSSID> -c 1 -w <ESSID> <interface>

# 4. Enviar de-auth (en otra terminal)
aireplay-ng -0 11 -a <BSSID> -c <Client_MAC> <interface>
# -0 = de-auth, 11 = número de paquetes

# 5. Esperar captura del WPA handshake en airodump-ng

# 6. Crackear con diccionario
aircrack-ng -a2 <BSSID> -w password.txt <captured_file>.cap
```

**Herramientas adicionales WPA/WPA2:** hashcat, EAPHammer, Portable Penetrator, WepCrackGui, Wifite.

**Fern Wifi Cracker:** Python-based GUI; cracking de WPA/WPS; usa wordlist (ej. rockyou.txt en `/usr/share/wordlists/`); paso clave: captura WPA handshake con de-auth attack.

#### WPA3 Cracking — Técnicas (Dragonblood)

**Dragonblood:** conjunto de vulnerabilidades en WPA3 → permite recuperar claves, degradar mecanismos de seguridad.

**Herramientas:** Dragonslayer, Dragonforce, Dragondrain, Dragontime.

| Técnica | Descripción |
|---|---|
| **Downgrade Security — backward compatibility** | Instala rogue AP con solo WPA2 compatibility → fuerza four-way handshake WPA2 → explota con herramientas WPA2 |
| **Downgrade Security — Dragonfly exploit** | El atacante masquerade como AP auténtico; informa al cliente que no soporta WPA3 → sugiere WPA2 → explota WPA2 |
| **Side-channel — Timing-based** | Analiza tiempo del Dragonfly handshake para codificar contraseñas → lista de contraseñas posibles |
| **Side-channel — Cache-based** | Inyecta JavaScript o web app malicioso en el browser → observa patrones de acceso a memoria para recuperar info de contraseñas |

**Cracking WPA3 con Aircrack-ng + hashcat:**
```bash
# 1. Monitor mode
airmon-ng start <interface>

# 2. Capturar handshake
airodump-ng --bssid <BSSID> --channel <CH> --write capture wlan0mon

# 3. De-auth para capturar handshake
aireplay-ng --deauth 10 -a <BSSID> -c <Client_MAC> wlan0mon

# 4. Convertir a formato hashcat
hcxpcapngtool -o capture.hccapx <capture>.cap

# 5. Crackear
hashcat -m 22000 capture.hccapx </path/to/wordlist.txt>
```

#### Cracking WPS con Reaver

**Reaver:** ataca WPS registrar PINs para recuperar WPA/WPA2 passphrases.

```bash
# 1. Monitor mode
airmon-ng start wlan0

# 2. Detectar APs con WPS
wash -i mon0
# Alternativa:
airodump-ng wlan0mon    # muestra BSSIDs disponibles

# 3. Crackear WPS PIN
reaver -i wlan0mon -b <BSSID> -vv
# Escanea todos los WPS PINs hasta encontrar uno válido
```

---

## 2. Exam Traps ⚠️

⚠️ **[Passive vs. Active footprinting: inyección de paquetes]**
En passive footprinting el atacante NO conecta ni inyecta paquetes. En active footprinting, el atacante SÍ envía probe requests (con SSID vacío o con el SSID correcto). El examen puede presentar active como "no inyecta paquetes" → incorrecto.

⚠️ **[Windows vs. Linux: inyección de paquetes]**
Windows solo puede escuchar tráfico wireless; **NO puede inyectar paquetes**. Linux puede hacer ambas cosas. El examen puede preguntar qué OS necesita el atacante para ataques de inyección → Linux.

⚠️ **[WarFlying usa drones — no confundir con WarDriving]**
WarFlying específicamente usa **drones** (no vehículos terrestres). WarDriving = conducir. WarWalking = caminar. El examen puede presentar WarFlying como equivalente a WarDriving.

⚠️ **[WPS lockout: >5 intentos incorrectos]**
Los routers WPS se bloquean automáticamente después de **más de 5 intentos incorrectos consecutivos**. Solo pueden desbloquearse manualmente en la interfaz admin. El examen puede cambiar el número.

⚠️ **[Airmon-ng: managed → monitor mode]**
Airmon-ng cambia la interfaz de **managed mode a monitor mode** (y viceversa). El monitor mode es el necesario para sniffar tráfico wireless. El examen puede confundir managed con monitor.

⚠️ **[Aireplay-ng: para IVs WEP y WPA handshakes]**
Aireplay-ng es especialmente efectivo para recopilar **WEP IVs y WPA handshakes**. No es la herramienta de cracking (eso es aircrack-ng). El examen puede confundir sus funciones.

⚠️ **[KRACK: fuerza reutilización de nonce, no captura contraseñas directamente]**
KRACK explota el four-way handshake forzando la **reutilización del nonce**. No captura ni crackea contraseñas directamente. Permite sniff, hijacking e inyección de malware. El examen puede presentarlo como captura de contraseñas.

⚠️ **[aLTEr Attack: Layer 2, AES-CTR (sin integridad)]**
aLTEr afecta dispositivos LTE que usan **AES-CTR** (que no tiene protección de integridad). Opera en **Layer 2 (data link layer)**. El examen puede ubicarlo en otra capa o atribuirlo a otro protocolo.

⚠️ **[Wi-Jacking: no requiere crackear handshake; requiere browser chromium + HTTP router]**
Wi-Jacking obtiene la WPA2 PSK **sin crackear ningún handshake**. Los requisitos específicos son: browser chromium, credenciales admin del router almacenadas y router con HTTP no cifrado. El examen puede presentarlo como equivalente a WPA cracking.

⚠️ **[Rogue AP USB: no requiere Ethernet port ni credenciales Wi-Fi]**
El USB-based rogue AP es el más sigiloso porque **no necesita puerto Ethernet libre ni credenciales de la red Wi-Fi objetivo**. Comparte el acceso de red de la máquina donde se conecta. El examen puede preguntar cuál escenario requiere menos prerequisitos.

---

## 3. Nemotécnicos

**5 fases metodología wireless** → **"D-A-L-C-C"**: **D**iscovery · **A**nalysis · **L**aunch attacks · **C**racking · **C**ompromising

**Wi-Fi Chalking** → **"W-W-W-W"** (4 Ws): **W**arWalking · **W**arChalking · **W**arFlying · **W**arDriving
- Drone = Flying · Símbolos = Chalking · Caminando = Walking · Conduciendo = Driving

**Aircrack-ng suite (10 tools)** → **"Base-Crack-Decap-Drop-Play-Graph-Mon-Dump-Lib-Tun"**:
- Airbase · Aircrack · Airdecap · Airdrop · Aireplay · Airgraph · Airmon · Airodump · Airolib · Airtun

**Función de cada herramienta aircrack-ng:**
- **Mon** = Monitor mode | **Dump** = Capture frames | **Play** = Inject/IVs/handshake
- **Crack** = Cracking WEP/WPA | **Base** = Fake AP | **Decap** = Decrypt/strip headers

**WPA/WPA2 cracking pasos** → **"Monitor → Dump → DeAuth → Wait handshake → Crack"**

**WPA3 downgrade attacks** → **"Backward compat (WPA2 only rogue AP) · Dragonfly masquerade (suggest WPA2)"**

**Requisitos Wi-Jacking** → **"Active client + Open network auto-reconnect + Chromium + Stored creds + HTTP router"**

**aLTEr** → **"LTE + AES-CTR (no integrity) + Layer 2 + fake tower + DNS spoofing"**

---

## 4. Flashcards

**Q:** ¿Cuáles son las 5 fases de la metodología de hacking wireless en orden?
**A:** Wi-Fi Discovery → Wireless Traffic Analysis → Launch of Wireless Attacks → Wi-Fi Encryption Cracking → Wi-Fi Network Compromising.

---

**Q:** ¿Cuál es la diferencia entre passive y active footprinting en wireless?
**A:** Passive: snifea paquetes del airwaves sin conectar ni inyectar. Active: envía probe requests (con SSID o vacías) esperando respuesta del AP.

---

**Q:** ¿Qué técnica de Wi-Fi chalking usa drones?
**A:** WarFlying.

---

**Q:** ¿Qué herramienta de Aircrack-ng cambia la interfaz a monitor mode?
**A:** Airmon-ng (`airmon-ng start <interface>`).

---

**Q:** ¿Para qué sirve Airodump-ng?
**A:** Captura raw 802.11 frames y recopila WEP IVs. También se usa para listar APs y clientes conectados.

---

**Q:** ¿Para qué sirve Aireplay-ng?
**A:** Recopila IVs WEP y handshakes WPA; especialmente efectivo para inyección de paquetes incluyendo de-autenticación.

---

**Q:** ¿Cuál es el comando aircrack-ng para crackear WPA/WPA2 con diccionario?
**A:** `aircrack-ng -a2 <BSSID> -w password.txt <captured_file>.cap`

---

**Q:** ¿Cuál es el mecanismo del KRACK attack?
**A:** Explota el four-way handshake de WPA2 forzando la reutilización de nonces (Nonce reuse), manipulando y reproduciendo mensajes criptográficos del handshake.

---

**Q:** ¿Qué capa afecta el aLTEr attack y qué protocolo de cifrado explota?
**A:** Layer 2 (data link layer). Explota dispositivos LTE que usan AES-CTR, que no proporciona protección de integridad.

---

**Q:** ¿Cuáles son las dos fases del aLTEr attack?
**A:** (1) Information gathering: identity mapping + website fingerprinting. (2) Attack phase: MITM con torre falsa + DNS spoofing.

---

**Q:** ¿Qué es Dragonblood y qué herramientas lo explotan?
**A:** Conjunto de vulnerabilidades en WPA3 que permite recuperar claves y degradar mecanismos de seguridad. Herramientas: Dragonslayer, Dragonforce, Dragondrain, Dragontime.

---

**Q:** ¿Cuál es la diferencia entre los dos métodos de downgrade de seguridad en WPA3?
**A:** (1) Backward compatibility: instala rogue AP solo con WPA2 → cliente se conecta via WPA2. (2) Dragonfly exploit: el atacante se hace pasar por AP auténtico y dice no soportar WPA3 → sugiere WPA2.

---

**Q:** ¿Qué prerequisitos necesita un Wi-Jacking attack?
**A:** Cliente activo conectado, reconexión automática a redes abiertas habilitada, browser chromium-based con credenciales admin del router almacenadas, y router con HTTP no cifrado para interfaz de configuración.

---

**Q:** ¿Por qué el USB-based rogue AP es más sigiloso que los otros métodos?
**A:** Porque no requiere puerto Ethernet libre ni credenciales de la red Wi-Fi objetivo — solo se conecta a cualquier máquina Windows de la red y comparte su acceso.

---

**Q:** ¿Qué hace MANA Toolkit y cuál es el archivo de configuración principal?
**A:** Crea rogue APs y realiza sniffing y MITM. Archivo de configuración principal: `hostapd-mana.conf`. Script de lanzamiento: `start-nat-simple.sh`.

---

**Q:** ¿Cuál es la herramienta para crackear WPS PINs y recuperar WPA/WPA2 passphrases?
**A:** Reaver (`reaver -i wlan0mon -b <BSSID> -vv`).

---

**Q:** ¿Cuántos paquetes de de-auth envía el comando aireplay-ng con `-0 11`?
**A:** 11 paquetes de de-autenticación (`-0` = de-auth attack, `11` = número de paquetes).

---

**Q:** ¿Qué herramienta se usa para convertir archivos .cap a formato .hccapx para usar con hashcat?
**A:** `hcxpcapngtool -o capture.hccapx <capture>.cap`

---

**Q:** ¿Cuáles son los pasos básicos del MITM attack wireless?
**A:** (1) Snifear parámetros wireless de la víctima. (2) Enviar DEAUTH con MAC del AP legítimo spoofada. (3) La víctima busca nuevo AP. (4) Crear forged AP con BSSID/ESSID originales. (5) Víctima conecta al AP falso. (6) Atacante se posiciona entre víctima y AP real.

---

**Q:** ¿Cuál es la diferencia entre el Jamming Signal Attack y un De-authenticate Flood?
**A:** Jamming Signal: usa hardware especializado para generar ruido en la misma frecuencia → todos los dispositivos retienen transmisiones (Layer 1, físico). De-authenticate Flood: inunda con frames forjados de desautenticación a nivel 802.11 (Layer 2).

---

## 5. Confusión frecuente

**WarDriving vs. WarWalking vs. WarFlying — misma función, diferente transporte**
- WarDriving: en vehículo (conducir).
- WarWalking: a pie (caminar).
- WarFlying: con drones (volar).
- WarChalking: diferente — dibuja símbolos en lugares públicos para ADVERTIR sobre redes, no para descubrirlas activamente.
- Criterio: "activamente buscando redes" → Driving/Walking/Flying. "Marcando ubicaciones de redes abiertas con símbolos" → WarChalking.

---

**Rogue AP vs. Evil Twin vs. Honeypot AP**
- Rogue AP: AP no autorizado instalado en la red para dar backdoor access o para que los usuarios se conecten; puede estar dentro del perímetro corporativo.
- Evil Twin: AP que imita el SSID de un AP legítimo; usa KARMA para adoptar SSIDs de probe requests.
- Honeypot AP: AP con señal más fuerte que el AP legítimo + mismo SSID; atrae pasivamente a dispositivos buscando la señal más potente.
- Criterio: "AP no autorizado que da backdoor" → Rogue AP. "Imitar SSID + KARMA" → Evil Twin. "Señal más fuerte + mismo SSID" → Honeypot.

---

**KRACK vs. Dragonblood — ataques a WPA2 vs. WPA3**
- KRACK: ataca el four-way handshake de WPA2 forzando reutilización de nonce.
- Dragonblood: conjunto de vulnerabilidades en WPA3 (Dragonfly/SAE handshake) → downgrade attacks + side-channel attacks.
- Criterio: "nonce reuse en four-way handshake WPA2" → KRACK. "Vulnerabilidades en SAE/Dragonfly de WPA3" → Dragonblood.

---

**Airmon-ng vs. Airodump-ng vs. Aireplay-ng**
- Airmon-ng: cambia la interfaz a monitor mode (no captura, no inyecta).
- Airodump-ng: CAPTURA raw 802.11 frames y WEP IVs; lista APs y clientes.
- Aireplay-ng: INYECTA paquetes; especialmente de-auth y recopilación de IVs/handshakes.
- Criterio: "poner en modo monitor" → Airmon-ng. "Capturar paquetes y listar APs" → Airodump-ng. "Inyectar/de-autenticar" → Aireplay-ng.

---

**WPA cracking offline vs. de-authentication attack**
- Offline attack: el atacante captura el handshake (unos segundos cerca del AP) y crackea offline sin interacción activa.
- De-authentication attack: el atacante FUERZA activamente al cliente a desconectarse → cuando se reconecta, captura el paquete de autenticación con el PMK.
- Criterio: "pasivo, espera handshake natural" → offline. "Activo, fuerza desconexión para capturar auth" → de-authentication attack.
