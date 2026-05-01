# M18_01 — IoT Hacking: Conceptos, Ataques y Metodología
**Módulo 18 / Subapartado 1 — IoT Hacking**

---

## 1. Conceptos y definiciones

### IoT — Arquitectura de 5 capas 🔴

| Capa (de arriba a abajo) | Función |
|--------------------------|---------|
| **Application Layer** | Entrega de servicios a usuarios por sectores (edificios, industrial, salud, automóvil, seguridad) |
| **Middleware Layer** | Opera en **modo bidireccional**; interfaz entre aplicación y hardware; gestión de datos/dispositivos, análisis, agregación, filtrado, descubrimiento de dispositivos, control de acceso |
| **Internet Layer** | Comunicación entre dos endpoints: device-to-device, device-to-cloud, device-to-gateway, back-end data sharing |
| **Access Gateway Layer** | Puente entre endpoints (dispositivo y cliente); manejo inicial de datos; message routing, message identification, subscribing |
| **Edge Technology Layer** | Hardware: sensores, RFID tags, readers, soft sensors; recopilación de datos y conexión de dispositivos a la red y al servidor |

**Middleware Layer** es la capa más crítica por operar en modo bidireccional y gestionar datos, dispositivos y control de acceso.

---

### Modelos de comunicación IoT (4) 🔴

| Modelo | Descripción | Protocolos/Tecnología | Ejemplo |
|--------|-------------|----------------------|---------|
| **Device-to-Device** | Los dispositivos se comunican directamente entre sí a través de Internet | ZigBee, Z-Wave, Bluetooth | Termostatos, bombillas, cerraduras, CCTV, frigoríficos; wearables |
| **Device-to-Cloud** | Los dispositivos se comunican directamente con la nube (no con el cliente directamente) | Wi-Fi, Ethernet, Cellular | Cámara CCTV accesible desde smartphone remoto |
| **Device-to-Gateway** | El dispositivo IoT comunica con un gateway intermedio (smartphone, hub) que conecta con la nube; el gateway proporciona seguridad y traducción de datos/protocolos | ZigBee, Z-Wave | Smart TV que conecta a cloud a través de una app de móvil |
| **Back-End Data Sharing** | Extensión del modelo Device-to-Cloud; datos accesibles por terceros autorizados | — | Análisis anual de consumo energético por consultoras externas |

---

### Protocolos IoT — Short Range 🔴

| Protocolo | Características clave |
|-----------|----------------------|
| **BLE (Bluetooth Low Energy)** | Red de área personal inalámbrica; sectores: salud, seguridad, entretenimiento, fitness |
| **Li-Fi** | VLC (Visible Light Communications); usa bombillas LED; velocidad **224 Gbps** |
| **NFC** | Inducción de campo magnético; pagos móviles contactless, redes sociales, identificación |
| **RFID** | Datos en tags leídos con campos electromagnéticos; industrial, automóvil, farmacéutico, mascotas |
| **Thread** | Protocolo basado en **IPv6** para IoT; home automation; comunicación en redes inalámbricas locales |
| **Wi-Fi** | Standard más común: **802.11n**; velocidad máxima: **600 Mbps**; alcance: ~**50 m** |
| **Wi-Fi Direct** | P2P sin access point; los dispositivos deciden quién actúa como AP |
| **Z-Wave** | Baja potencia, corto alcance; home automation (HVAC, termostatos, garajes, cines en casa) |
| **ZigBee** | IEEE **802.15.4**; transferencia infrecuente de datos a baja tasa; rango **10–100 m** |
| **ANT** | Adaptive Network Topology; multicast; sensores de deporte y fitness |
| **QR/Barcodes** | Tags legibles por máquina; QR = 2D; Barcode = 1D o 2D |

**Medium-Range:**
- **HaLow** — variante Wi-Fi; largo alcance; bajas tasas de datos; útil en zonas rurales
- **LTE-Advanced** — mejora LTE; mayor capacidad, rango extendido, eficiencia
- **6LoWPAN** — IPv6 sobre redes inalámbricas de área personal de baja potencia (dispositivos con capacidad limitada)
- **QUIC** — conexiones multiplexadas sobre **UDP** entre dispositivos IoT; seguridad equivalente a SSL/TLS

**Long-Range:**
- **LPWAN** → LoRaWAN (M2M, smart cities, healthcare), Sigfox (batería corta, datos limitados), Neul (TV white space)
- **VSAT** — antenas de disco pequeñas; broadband y narrowband
- **MQTT** — ISO standard; protocolo ligero para mensajería de largo alcance; comunicación via satellite
- **NB-IoT** — variante de LoRaWAN/Sigfox; comunicación M2M con tecnología física mejorada

**Wired:**
- **Ethernet** — LAN cableada; más común
- **MoCA** — vídeo HD sobre cables coaxiales existentes
- **PLC (Power-Line Communication)** — transmite datos y electricidad por cables eléctricos; home automation, industrial, BPL

---

### Protocolos de aplicación IoT

| Protocolo | Uso |
|-----------|-----|
| **CoAP** | Web transfer para nodos constrained e IoT; M2M (building automation, smart energy) |
| **LWM2M** | Capa de aplicación para comunicación y gestión de dispositivos IoT |
| **XMPP** | Comunicación en tiempo real; interoperabilidad de dispositivos IoT |
| **MQTT** | ISO standard; mensajería ligera; largo alcance; satellite links |
| **Physical Web** | Interacción rápida con dispositivos IoT cercanos via BLE beacons (lista de URLs emitidas) |
| **Mihini/M3DA** | Comunicación entre servidor M2M y apps en gateway embebido |

---

### OWASP Top 10 IoT Threats 🔴

| # | Amenaza | Descripción clave |
|---|---------|-------------------|
| 1 | **Weak/Guessable/Hardcoded Passwords** | Credenciales por defecto, hardcoded o guessable; backdoors en firmware |
| 2 | **Insecure Network Services** | Buffer overflow → DoS; puertos abiertos explotados con port scanners y fuzzers |
| 3 | **Insecure Ecosystem Interfaces** | Web, backend API, mobile, cloud interfaces; falta auth/authz, cifrado débil, sin filtrado I/O |
| 4 | **Lack of Secure Update Mechanisms** | Sin validación de firmware, entrega insegura, sin anti-rollback, sin notificaciones |
| 5 | **Use of Insecure/Outdated Components** | Versiones antiguas de OS/librerías; supply chain comprometida |
| 6 | **Insufficient Privacy Protection** | PII de usuarios/dispositivos comprometida |
| 7 | **Insecure Data Transfer and Storage** | Sin cifrado ni control de acceso; datos en tránsito o en reposo expuestos |
| 8 | **Lack of Device Management** | Sin gestión de activos, updates, decommissioning, monitorización, respuesta |
| 9 | **Insecure Default Settings** | Configuraciones que restringen al operador de hacer el dispositivo más seguro |
| 10 | **Lack of Physical Hardening** | Sin medidas físicas → acceso local o remoto |

---

### Tipos de ataques IoT 🔴

| Ataque | Mecanismo específico |
|--------|---------------------|
| **DDoS** | Dispositivos comprometidos como botnets atacan servidor objetivo |
| **HVAC Attack** | Explotar vulnerabilidades del sistema HVAC para robar credenciales y pivotar a la red corporativa; usa Shodan + defpass.com para credenciales por defecto |
| **Rolling Code Attack** | Jammer + sniffer simultáneo; captura dos códigos; reenvía el primero al coche (que se abre); usa el segundo para robar el vehículo. Herramientas: **HackRF One**, **RFCrack** |
| **BlueBorne Attack** | Explota vulnerabilidades Bluetooth; no requiere pairing; funciona aunque el dispositivo no esté en modo discoverable; obtiene MAC + OS → RCE o MITM; Android, Linux, Windows, iOS antiguo |
| **Jamming Attack** | Tráfico malicioso masivo en la frecuencia del objetivo → DoS; señales aparecen como "ruido" para los dispositivos wireless |
| **Sybil Attack** | Múltiples identidades forjadas para crear ilusión de congestión de tráfico; afecta VANETs |
| **Rolling Code Attack** | Ver arriba |
| **DNS Rebinding** | JavaScript malicioso en web page → acceso al router/dispositivos IoT locales; herramienta: Singularity of Origin |
| **Side-Channel Attack** | Extrae claves de cifrado observando emisiones de señales (consumo de energía, emanaciones EM) |
| **Fault Injection Attack** | Introduce fallas en el dispositivo; tipos: Optical/EMFI/BBI, Power/Clock/Reset Glitching, Frequency/Voltage Tampering, Temperature Attacks |
| **Replay Attack** | Captura y retransmisión de mensajes legítimos → DoS o manipulación |
| **Forged Malicious Device** | Reemplaza físicamente un dispositivo IoT legítimo por uno malicioso con backdoor |
| **SDR-Based Attack** | Radio definida por software para examinar/manipular señales IoT; tipos: Replay, Cryptanalysis, Reconnaissance |
| **FOTA Attack** | Intercepción y manipulación del proceso de actualización firmware OTA |
| **Network Pivoting** | Usa dispositivo IoT comprometido para pivotar hacia servidor cerrado y otros dispositivos |
| **Ransomware** | Cifra acceso al dispositivo/ficheros hasta pago; distribuido via email/malvertisements |

---

### Rolling Code Attack — Flujo detallado 🔴

1. Víctima pulsa el mando y envía rolling code para abrir el coche
2. El atacante usa un jammer que **simultáneamente** bloquea la recepción del código por el coche Y sniffa el primer código
3. El coche no se abre; la víctima intenta de nuevo enviando un segundo código
4. El atacante sniffa el segundo código
5. El atacante reenvía el primer código → el coche se abre
6. El segundo código grabado se usa posteriormente para abrir/robar el vehículo

**Clave:** el atacante tiene dos códigos; el primero ya fue usado por el coche pero el segundo es válido.

---

### BlueBorne Attack — Flujo 🔴

1. Descubrir dispositivos Bluetooth activos (incluso si no están en modo discoverable)
2. Obtener la dirección MAC del dispositivo
3. Enviar probes continuos para determinar el OS
4. Explotar vulnerabilidades del protocolo Bluetooth para acceder al dispositivo
5. Ejecutar RCE o MITM y tomar control total

**Características distintivas:** no requiere pairing, no requiere interacción del usuario, no requiere modo discoverable, compatible con todos los SO (Android, Linux, Windows, iOS antiguo), tiene altos privilegios en todos los OS.

---

### HVAC Attack — Flujo

1. Atacante usa **Shodan** para encontrar ICS (Industrial Control Systems) vulnerables
2. Busca credenciales por defecto en herramientas como **defpass.com**
3. Intenta acceder al ICS con credenciales por defecto
4. Desde el ICS, accede al sistema HVAC remotamente
5. Controla temperatura u otros ataques en la red local

---

### Fault Injection Attacks — Tipos

| Tipo | Mecanismo |
|------|-----------|
| **Optical/EMFI/BBI** | Láseres, pulsos electromagnéticos, pulsos de alto voltaje en bloques analógicos (RNGs) |
| **Power/Clock/Reset Glitching** | Inyectar faults en la alimentación o reloj del chip → ejecución remota, saltar instrucciones clave |
| **Frequency/Voltage Tampering** | Modificar nivel de alimentación o frecuencia de reloj → fault behavior en el chip |
| **Temperature Attacks** | Alterar temperatura de operación del chip → condiciones no nominales |

**Invasivo vs no-invasivo:** no-invasivo = atacante debe estar muy cerca del chip. Invasivo = superficie del chip visible y acceso físico.

---

### SDR-Based Attacks — 3 tipos

| Tipo | Diferencia clave |
|------|-----------------|
| **Replay Attack** | Captura la secuencia de comandos y la retransmite; usa URH (Universal Radio Hacker) para segregar la secuencia |
| **Cryptanalysis Attack** | Igual que replay + reverse-engineering del protocolo para obtener la señal original; requiere conocimientos de criptografía y modulación |
| **Reconnaissance Attack** | Adicional al cryptanalysis; obtiene info de las especificaciones del dispositivo; usa multímetros para investigar el chipset y comparar con reportes FCC publicados |

---

### IoT Hacking Methodology — 5 fases 🔴

**Fase 1: Information Gathering**
- Extraer: IP, protocolos (ZigBee, BLE, 5G, IPv6LoWPAN), puertos abiertos, tipo de dispositivo, geolocalización, número de fabricación, compañía fabricante, diseño HW, infraestructura
- Herramientas: **Shodan**, **Censys**, **FOFA**, MultiPing, FCC ID Search
- Sniffers para dispositivos fuera de red pero en rango: **Suphacap**, **CloudShark**, **Wireshark**

**Shodan filters:**
```
webcamxp country:US          # Webcams en USA
webcamxp city:paris           # Webcams en París
webcamxp geo:-50.81,201.80   # Webcam por coordenadas
net:                          # Por IP/CIDR
os:                           # Por OS del dispositivo
port:                         # Puertos abiertos
before/after:                 # Filtro temporal
```

**FCC ID:** compuesto por **Grantee ID** (3 o 5 caracteres iniciales) + **Product ID** (caracteres restantes). Proporciona: detalles del dispositivo, frecuencias, fotos externas/internas, informe de prueba, manual de usuario.

**Fase 2: Vulnerability Scanning**
- IoTSeeker: detecta dispositivos con credenciales por defecto: `perl iotScanner.pl <IP/range>`
- Genzai: detecta dashboards IoT (routers, cámaras, HMIs); fingerprinting via HTTP responses; `./genzai <target_host> -save scan.json`
- **Nmap** commands:
```bash
nmap -n -Pn -sS -pT:0-65535 -v -A -oX <Name> <IP>           # TCP scan
nmap -n -Pn -sSU -pT:0-65535,U:0-65535 -v -A -oX <Name> <IP> # TCP+UDP
nmap -6 -n -Pn -sSU -pT:0-65535,U:0-65535 -v -A -oX <Name> <IP> # IPv6
```
- beSTORM: smart fuzzer para buffer overflow; black-box; intenta combinaciones de ataque starting con los escenarios más probables

**Fase 3: Launch Attacks**
- RFCrack para rolling-code attack, replay, jamming
- KillerBee para ZigBee/IEEE 802.15.4
- HackRF One para BlueBorne, replay, fuzzing, jamming (rango 1 MHz a 6 GHz, half-duplex)

**HackRF One commands:**
```bash
hackrf_transfer -r connector.raw -f [frecuencia]   # Grabar señal
hackrf_transfer -t connector.raw -f [frecuencia]   # Reproducir señal
```

**RFCrack commands:**
```bash
python RFCrack.py -i                              # Live replay
python RFCrack.py -r -M MOD_2FSK -F 314350000   # Rolling code
python RFCrack.py -j -F 314000000               # Jamming
python RFCrack.py -k                              # Scan common frequencies
```

**Fase 4: Gain Remote Access**
- Telnet (puerto abierto) → acceso sin auth o con credenciales por defecto (root/root, system/system)
- Backdoor via phishing → acceso a red SCADA/ICS → reemplazo de firmware legítimo por malicioso

**Fase 5: Maintain Access**
- Firmware Mod Kit: extrae, modifica y reconstruye firmware (Linux-based routers, TRX/uImage, SquashFS/CramFS)
- Scripts clave: `extract-firmware.sh`, `build-firmware.sh`
- Eliminar logs, actualizar firmware, backdoors, trojans

---

### Firmware Analysis — Comandos 🔴

```bash
file *.bin                          # Identificar tipo de archivo
md5sum *.bin                        # Verificar firma MD5
strings -n 10 xyz.bin > strings.out # Extraer strings (mín. 10 chars)
hexdump -C -n 512 xyz.bin           # Hexdump (512 bytes)
binwalk xyz.bin                     # Análisis e identificación del filesystem
dd if=xyz.bin bs=1 skip=<offset> count=<size> of=xyz.squashfs  # Extraer filesystem
mkdir rootfs && sudo mount -t ext2 {filename} rootfs            # Montar filesystem
grep -rnw '/path/' -e "password"    # Buscar credenciales en filesystem
find . -name '*.conf'               # Encontrar ficheros de configuración
```

**Emulación con QEMU:**
```bash
file <binary>         # Identificar arquitectura CPU
qemu-mipsel -L <prefix> <binary>    # Emulación MIPS
qemu-arm -L <prefix> <binary>       # Emulación ARM
```

---

### Interfaces de hardware IoT — UART, JTAG, I2C, SPI

**JTAG (IEEE 1149.1):** 4 pines obligatorios: **TMS** (Test Mode Select), **TCK** (Test Clock), **TDI** (Test Data In), **TDO** (Test Data Out) + 1 opcional: **TRST** (Test Reset).

**UART:** acceso al shell via consola serial.

**I2C:** usa **SDA** (serial data) + **SCL** (serial clock).

**Comandos EXPLIoT framework:**
```bash
run busauditor.generic.uartscan -v 3.3 -p /dev/ttyACM0 -s 0 -e 1
run busauditor.generic.jtagscan -v 3.3 -p /dev/ttyACM0 -s 0 -e 10
run busauditor.generic.i2scan -v 3.3 -p /dev/ttyACM0 -s 0 -e 10
```

---

### NAND Glitching

Proceso de obtener acceso root privilegiado durante el boot del dispositivo, cortocircuitando el pin serial I/O de un chip de memoria flash a tierra. Explota el código de arranque de backup que concede acceso single-user privilegiado durante fallos de arranque. Un glitching bien temporizado puede durar solo **1 ms**.

```bash
minicom -D /dev/ttyUSB0 -w -C D-link_startup.txt  # Reconocimiento via UART
# Cortocircuitar pin serial I/O a tierra
setenv bootargs 'noinitrd console=ttyAM0,115200 ... init=/bin/sh'
nand read ${loadaddr} app-kernel 0x00400000 && bootm ${loadaddr}  # Obtener root
```

---

### OWASP Top 10 IoT — Soluciones 🔴

| Vulnerabilidad | Solución clave |
|----------------|---------------|
| Weak/Hardcoded Passwords | APM (Automated Password Management); contraseñas fuertes; no hardcoded |
| Insecure Network Services | Cerrar puertos abiertos; deshabilitar UPnP; cifrar antes de TLS |
| Insecure Ecosystem Interfaces | Account lockout; 2FA; sanity checking y output filtering; assessment periódico |
| Lack of Secure Update Mechanism | Verificar fuente e integridad de updates; cifrar comunicación; notificar a usuarios |
| Insecure/Outdated Components | Monitorizar componentes sin mantenimiento; eliminar dependencias innecesarias; evitar supply chain comprometida |
| Insufficient Privacy Protection | Minimizar recopilación de datos; anonimizar; dar control al usuario |
| Insecure Data Transfer and Storage | Cifrar comunicación; mantener SSL/TLS; **evitar soluciones de cifrado propias** |
| Lack of Device Management | Blacklist de dispositivos sospechosos; validar todos los atributos; decommissioning seguro |
| Insecure Default Settings | Cambiar usernames/passwords por defecto; modificar ajustes de privacidad; deshabilitar acceso remoto cuando no se use |
| Lack of Physical Hardening | Contraseña única para BIOS/firmware; configurar boot order para prevenir arranque no autorizado; **minimizar puertos externos como USB** |

---

### Countermeasures IoT — Puntos clave de examen 🔴

- Deshabilitar telnet (puerto **23**)
- Deshabilitar UPnP en routers
- Monitorizar tráfico en **puerto 48101** (dispositivos infectados intentan propagarse por este puerto)
- Implementar cifrado extremo a extremo + PKI
- Secure boot con firma de código criptográfica (solo código del OEM)
- 2-way authentication: **SHA con HMAC** (claves simétricas) + **ECDSA** (claves asimétricas)
- Filtrar IPs privadas de respuestas DNS con **dnswall** para prevenir DNS rebinding
- Usar **TEE (Trusted Execution Environment)** o SE, TrustZone para ARM
- Implementar masking/shielding activo contra side-channel attacks
- Almacenar claves en **SAM (Secure Access Module)**, **TPM (Trusted Platform Module)**, **HSM**, o trusted key store
- Deshabilitar **WebRTC** para evitar disclosure de IPs
- Frecuencia hopping (**FHSS**) para proteger contra SDR jamming
- Password mínima: **8–10 caracteres** con letras, números y caracteres especiales

---

### IoT Malware destacado

**KmsdBot (Kmsdx):** targets IoT; escaneo Telnet (puerto 23) + SSH; descarga lista de contraseñas débiles del C2 server (fichero `telnet.txt`); soporta múltiples arquitecturas CPU.

**IZ1H9 (Mirai-based):** targets Linux IoT; shellscript downloader (`lb.sh`); binarios para arquitecturas: x86, mips, mpsl, arm4/5/6/7, ppc, m68k, sh4, 86_64; descifrado XOR con key **0xBAADF00D**; propaga via HTTP, SSH y Telnet; **~100 combinaciones de usuario/contraseña débiles** para SSH/Telnet brute force; 4 vulnerabilidades RCE para HTTP.

---

## 2. Exam Traps ⚠️ 🔴

⚠️ **[Li-Fi — velocidad]** Li-Fi usa luz visible (VLC) para transferencia de datos a **224 Gbps**. No confundir con Wi-Fi. El dato de velocidad es directamente preguntable.

⚠️ **[Wi-Fi estándar IoT — 802.11n]** El estándar más común en hogares/empresas es **802.11n** con velocidad máxima **600 Mbps** y alcance ~**50 m**. El examen puede preguntar qué estándar se menciona específicamente para IoT.

⚠️ **[ZigBee — estándar IEEE]** ZigBee se basa en **IEEE 802.15.4**, no en 802.11. Rango: **10–100 m**.

⚠️ **[Thread — protocolo basado en IPv6]** Thread es un protocolo de networking IoT basado en **IPv6** para home automation. No confundir con Z-Wave (que también es home automation pero no es IPv6-based).

⚠️ **[QUIC — protocolo base]** QUIC usa **UDP**, no TCP. Proporciona seguridad equivalente a SSL/TLS.

⚠️ **[Rolling Code Attack — orden de eventos]** El atacante sniffa DOS códigos. Usa el primero para que el coche se abra DURANTE el ataque (reenvío mientras la víctima espera). Usa el SEGUNDO para robar el vehículo posteriormente. El examen puede preguntar qué código se usa para qué.

⚠️ **[BlueBorne — modo discoverable]** BlueBorne puede descubrir y atacar dispositivos Bluetooth aunque NO estén en modo discoverable. Este es el rasgo más diferenciador y el más preguntado.

⚠️ **[HVAC Attack — herramientas específicas]** Shodan para encontrar ICS + **defpass.com** para credenciales por defecto. Si el examen pregunta cómo un atacante encuentra credenciales por defecto para HVAC → defpass.com.

⚠️ **[Puerto 48101]** Los dispositivos IoT infectados intentan propagar ficheros maliciosos usando específicamente el **puerto 48101**. Dato directamente preguntable en countermeasures.

⚠️ **[Middleware Layer — bidireccional]** La capa Middleware es la más crítica porque opera en **modo bidireccional**, a diferencia de las otras capas. Es la interfaz entre aplicación y hardware.

⚠️ **[JTAG — número de pines]** JTAG tiene **4 pines obligatorios** (TMS, TCK, TDI, TDO) + **1 opcional** (TRST). Total máximo: 5 pines.

⚠️ **[IZ1H9 — clave XOR]** La clave XOR de descifrado de IZ1H9 es **0xBAADF00D**. Dato muy específico que puede aparecer en preguntas sobre malware IoT.

⚠️ **[DNS Rebinding — herramienta]** La herramienta específica para DNS rebinding en IoT es **Singularity of Origin**. La contramedida es **dnswall** para filtrar IPs privadas de respuestas DNS.

⚠️ **[NAND Glitching — duración]** Un glitch bien temporizado puede durar solo **1 ms** y aun así resultar en acceso root privilegiado.

⚠️ **[Countermeasures — 2-way auth algoritmos]** La autenticación bidireccional usa **SHA con HMAC** (claves simétricas) + **ECDSA** (claves asimétricas). Combinación específica mencionada en el CEH.

---

## 3. Nemotécnicos

### Arquitectura IoT — 5 capas (arriba a abajo)
**"AMIGA-E"**: **A**pplication → **M**iddleware → **I**nternet → **G**ateway (Access) → **E**dge Technology

### OWASP Top 10 IoT — orden
**"WEIRD PDUL"** (W-I-E-R-D-P-D-L = 8 primeras):
- **W**eak passwords → **I**nsecure network services → **I**nsecure ecosystem interfaces → lack of secure updates (**R**emovable/secure) → **I**nsecure components → **P**rivacy insuficiente → **D**ata insecure → **L**ack device management → **I**nsecure default settings → **P**hysical hardening absent

Simplificado: **"WII-RIPDLIP"** o memorizar por categorías:
- Credenciales (1) → Servicios de red (2) → Interfaces (3) → Updates (4) → Componentes (5) → Privacidad (6) → Datos (7) → Gestión (8) → Config (9) → Físico (10)

### Rolling Code — 2 códigos, 2 propósitos
- **Código 1** = usado DURANTE el ataque para que el coche se abra (la víctima ve que funciona)
- **Código 2** = guardado para robar el vehículo DESPUÉS

### BlueBorne — 5 características clave (ninguna requiere)
**"NIPUD"**: **N**o discoverable mode needed → **N**o pairing needed → **N**o interaction needed → **U**ser no action needed → **D**evice type varies (Android, Linux, Windows, iOS antiguo)

### Firmware análisis — flujo
**"OBAE-MF"**: **O**btener → **B**inwalk → **A**nalizar strings/hexdump → **E**xtraer filesystem con dd → **M**ontar → **F**irm análisis (grep, find)

### JTAG — 4+1 pines
**"TMS-TCK-TDI-TDO + TRST"**: **T**MS, **T**CK, **T**DI, **T**DO = obligatorios; **T**RST = opcional.
Mnemónico: "**T**odos **M**is **C**hicos **T**oman **D**atos **T**ecnológicos **O**pcionales"

### Protocolos — distancias
- **Short:** BLE, Li-Fi, NFC, RFID, Thread, Wi-Fi, Wi-Fi Direct, Z-Wave, ZigBee, ANT, QR
- **Medium:** HaLow, LTE-A, 6LoWPAN, QUIC
- **Long:** LPWAN (LoRa, Sigfox, Neul), VSAT, Cellular, MQTT, NB-IoT

---

## 4. Flashcards

**Q:** ¿Cuál es la velocidad de transferencia de datos de Li-Fi y qué tecnología usa?
**A:** **224 Gbps**. Usa **VLC (Visible Light Communications)** con bombillas domésticas LED. Es como Wi-Fi pero usa luz visible en lugar de ondas de radio.

---

**Q:** ¿Qué estándar IEEE usa ZigBee y cuál es su rango?
**A:** **IEEE 802.15.4**. Rango: **10–100 m**. Transferencia infrecuente de datos a baja tasa en área restringida.

---

**Q:** ¿Qué protocolo IoT usa UDP y proporciona seguridad equivalente a SSL/TLS?
**A:** **QUIC** (Quick UDP Internet Connections) — conexiones multiplexadas sobre UDP entre dispositivos IoT.

---

**Q:** ¿Cuál es la capa más crítica de la arquitectura IoT y por qué?
**A:** La capa **Middleware**, porque opera en **modo bidireccional** y es responsable de gestión de datos, gestión de dispositivos, análisis, agregación, filtrado, descubrimiento de dispositivos y control de acceso.

---

**Q:** ¿Por qué el BlueBorne attack es especialmente peligroso en IoT?
**A:** No requiere pairing, no requiere interacción del usuario, no requiere que el dispositivo esté en modo discoverable, y el proceso Bluetooth tiene **altos privilegios** en todos los SO. Compatible con Android, Linux, Windows e iOS antiguo.

---

**Q:** Describe el flujo de un Rolling Code Attack con los dos códigos capturados.
**A:** 1) Víctima envía código 1 → atacante lo sniffa Y bloquea la recepción del coche. 2) Coche no se abre; víctima reintenta → envía código 2 → atacante lo sniffa. 3) Atacante reenvía código 1 → coche se abre. 4) Código 2 se usa posteriormente para robar el vehículo.

---

**Q:** ¿Qué herramienta usa el atacante para encontrar credenciales por defecto de sistemas HVAC y cuál usa para encontrar los sistemas?
**A:** **Shodan** para encontrar ICS (Industrial Control Systems) vulnerables en Internet. **defpass.com** para obtener las credenciales por defecto.

---

**Q:** ¿Qué puerto específico deben monitorizar los administradores porque los dispositivos IoT infectados lo usan para propagar ficheros maliciosos?
**A:** **Puerto 48101**.

---

**Q:** ¿Cuáles son los 4 pines obligatorios y el 1 opcional de JTAG?
**A:** Obligatorios: **TMS** (Test Mode Select), **TCK** (Test Clock), **TDI** (Test Data In), **TDO** (Test Data Out). Opcional: **TRST** (Test Reset).

---

**Q:** ¿Cuáles son los 3 tipos de ataques SDR-Based en IoT y cuál es la diferencia entre Replay y Cryptanalysis?
**A:** Replay (captura + retransmisión de secuencia de comandos), Cryptanalysis (igual + reverse-engineering del protocolo para obtener señal original), Reconnaissance (análisis de especificaciones del dispositivo y chipset via multímetros). Diferencia clave: Cryptanalysis añade reverse-engineering del protocolo al Replay.

---

**Q:** ¿Qué es NAND Glitching y cuánto puede durar el glitch?
**A:** Obtención de acceso root privilegiado durante el boot cortocircuitando el pin serial I/O del chip de memoria flash a tierra. Un glitch bien temporizado puede durar solo **1 ms** y resultar en acceso root.

---

**Q:** ¿Qué hace la herramienta binwalk en el análisis de firmware IoT?
**A:** Analiza, hace reverse engineering y extrae datos de la imagen de firmware. Identifica el tipo de filesystem en uso dentro del firmware binario.

---

**Q:** ¿Qué clave XOR usa el malware IZ1H9 (Mirai-based) para descifrar sus strings de configuración?
**A:** **0xBAADF00D**.

---

**Q:** ¿Cuál es la diferencia entre Device-to-Device y Device-to-Gateway?
**A:** Device-to-Device: los dispositivos se comunican directamente entre sí (ZigBee, Z-Wave, Bluetooth). Device-to-Gateway: el dispositivo IoT comunica con un gateway intermedio (smartphone, hub) que a su vez conecta con la nube; el gateway añade seguridad y traducción de protocolos.

---

**Q:** ¿Qué modelo de comunicación IoT permite a terceros autorizados acceder a los datos de los dispositivos?
**A:** **Back-End Data-Sharing** — extensión del Device-to-Cloud donde los datos en la nube son accesibles por terceros autorizados para análisis.

---

**Q:** ¿Qué algoritmos específicos se mencionan para la autenticación bidireccional en IoT?
**A:** **SHA con HMAC** para claves simétricas + **ECDSA** para claves asimétricas.

---

**Q:** ¿Qué herramienta de análisis de espectro SDR se menciona en el CEH y con qué hardware se usa?
**A:** **Gqrx** (implementado con GNU Radio y Qt GUI). Se usa con hardware como FunCube dongle, Airspy, HackRF, RTL-SDR y USRP.

---

**Q:** ¿Cuál es la solución OWASP para "Lack of Physical Hardening" en IoT?
**A:** Contraseña única para BIOS/firmware; configurar el orden de arranque para prevenir booting no autorizado; **minimizar puertos externos como USB**.

---

**Q:** ¿Qué comando RFCrack se usa para un ataque de rolling code?
**A:** `python RFCrack.py -r -M MOD_2FSK -F 314350000` (rolling code bypass con modulación 2FSK en frecuencia 314350000 Hz).

---

## 5. Confusión frecuente

### Z-Wave vs ZigBee — protocolos de home automation
- **Z-Wave:** baja potencia, corto alcance; home automation (HVAC, termostatos, garajes, cines); no basado en IEEE 802.15.4.
- **ZigBee:** IEEE **802.15.4**; rango **10–100 m**; datos infrequentes a baja tasa; aplicaciones más variadas.
- **Criterio:** ¿menciona IEEE 802.15.4? → ZigBee. ¿Home automation con HVAC y termostatos? → Z-Wave (aunque ZigBee también se usa en home automation; Z-Wave se menciona más específicamente en ese contexto).

---

### Thread vs Z-Wave (home automation + protocolo)
- **Thread:** basado en **IPv6**; diseñado para que dispositivos IoT se comuniquen en redes inalámbricas locales.
- **Z-Wave:** baja potencia; home automation; no es IPv6-based.
- **Criterio:** ¿IPv6-based para IoT local? → Thread. ¿Home automation con foco en fiabilidad inalámbrica? → Z-Wave.

---

### Replay Attack vs Rolling Code Attack
- **Replay Attack:** captura y retransmite mensajes legítimos sin modificación para DoS o control del dispositivo.
- **Rolling Code Attack:** ataque específico contra sistemas de keyless entry que cambian el código en cada uso. El atacante captura DOS códigos y usa la dinámica del protocolo a su favor.
- **Criterio:** ¿sistema de entrada sin llave de vehículo/garaje? → Rolling Code. ¿Retransmisión genérica de mensajes? → Replay.

---

### Side-Channel Attack vs Fault Injection Attack
- **Side-Channel:** extrae información (claves) observando emisiones físicas del dispositivo (consumo de energía, EM, sonido, tiempo). No modifica el dispositivo.
- **Fault Injection:** introduce deliberadamente fallos en el dispositivo (glitching, temperatura, voltaje) para comprometer su seguridad. Modifica el comportamiento del dispositivo.
- **Criterio:** ¿observar emisiones para extraer claves? → Side-Channel. ¿Introducir fallos físicos para explotar comportamiento? → Fault Injection.

---

### MQTT vs CoAP (protocolos IoT de aplicación)
- **MQTT:** ISO standard; mensajería ligera; pensado para largo alcance y conexiones remotas (satellite links); publish-subscribe.
- **CoAP:** web transfer protocol; M2M; building automation y smart energy; similar a HTTP pero para redes constrained.
- **Criterio:** ¿mensajería ligera para largo alcance/satellite? → MQTT. ¿M2M en redes constrained similar a HTTP? → CoAP.
