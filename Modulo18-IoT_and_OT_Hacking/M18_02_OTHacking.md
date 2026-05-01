# M18_02 — OT Hacking: Conceptos, Arquitectura, Ataques y Metodología
**Módulo 18 / Subapartado 2 — OT Hacking**

---

## 1. Conceptos y definiciones

### OT vs IT — Diferencia fundamental

**OT (Operational Technology):** combinación de hardware y software diseñada para **detectar o causar cambios en operaciones industriales** mediante monitorización y/o control directo de dispositivos físicos industriales (switches, bombas, luces, sensores, elevadores, robots, válvulas, sistemas de climatización).

**IT/OT Convergence (IIoT):** integración de sistemas IT (computing) con sistemas OT (monitorización operacional). Beneficios: Industry 4.0 (smart manufacturing), mantenimiento predictivo, mejor control de calidad, decisiones basadas en datos OT en sistemas BI.

**Componentes ICS:** SCADA, DCS, BPCS, SIS, HMI, PLC, RTU, IED.

**Protocolos OT propietarios:** S7, CDA, SRTP
**Protocolos OT no propietarios:** Modbus, OPC, DNP3, CIP

---

### El Modelo Purdue — 6 niveles + IDMZ 🔴

El Purdue model (= Industrial Automation and Control System reference model, derivado de PERA) divide el entorno IT/OT en **3 zonas** con **6 niveles operativos**:

| Zona | Nivel | Nombre | Sistemas clave |
|------|-------|--------|----------------|
| **Enterprise Zone (IT)** | **5** | Enterprise Network | B2B, B2C, conectividad Internet |
| **Enterprise Zone (IT)** | **4** | Business Logistics Systems | Servidores de aplicaciones, ficheros, BBDD, email; planificación y scheduling de producción |
| **IDMZ** | **3.5** | Industrial DMZ | Microsoft domain controllers, database replication servers, proxy servers; barrera entre OT e IT |
| **Manufacturing Zone (OT)** | **3** | Operational Systems | MES/MOMS, data historians, scheduling, QA, optimización de procesos |
| **Manufacturing Zone (OT)** | **2** | Control Systems | DCS, SCADA software, HMIs, real-time software, PLCs supervisory |
| **Manufacturing Zone (OT)** | **1** | Basic Controls / Intelligent Devices | PLCs (función control), RTUs, IEDs, PID controllers, VFDs |
| **Manufacturing Zone (OT)** | **0** | Physical Process | Sensores, actuadores, dispositivos físicos; Equipment Under Control (EUC) |

**IDMZ propósito:** contener compromisos de red/sistema dentro de su capa, permitiendo producción sin interrupciones si hay intrusiones.

**PLC posición:** nivel 2 → función supervisory. Nivel 1 → función de control. El examen puede preguntar ambos.

---

### Componentes ICS — Definiciones 🔴

**SCADA (Supervisory Control and Data Acquisition):**
- Sistema centralizado de control y supervisión de instalaciones e infraestructuras industriales **distribuidas geográficamente**
- Componentes HW: control server (SCADA-MTU), dispositivos de comunicación, PLCs/RTUs distribuidos
- Almacena datos en **data historian** para análisis y setpoints
- Fault-tolerant con sistemas redundantes (no suficiente para proteger contra ataques maliciosos)
- Sectores: transporte de petróleo/gas, tratamiento de aguas, pipelines, telecom, power grids, transporte público

**DCS (Distributed Control System):**
- Control de sistemas de producción en la **misma localización geográfica**
- Alta ingeniería, gran escala; centralized supervisory control unit + múltiples local controllers + miles de I/O points
- Alta redundancia en todos los niveles (I/O → red); permite continuidad si falla un procesador
- Usa SCADA/MTU como supervisory loop con RTU/PLC locales
- Sectores: química, nuclear, refinerías, tratamiento de aguas, generación eléctrica, farmacéutica

**PLC (Programmable Logic Controller):**
- Ordenador digital en tiempo real para automatización industrial
- 3 módulos: **CPU** (procesador + RAM + ROM + retentive memory), **Power Supply** (AC→DC; 5V DC para circuitos del PLC; 24V DC para sensores/actuadores en algunos PLCs), **I/O Modules** (Digital I/O, Analog I/O, Communication I/O)
- **Retentive memory:** preserva programas y datos durante fallos de alimentación → no necesita monitor/teclado para reprogramar tras fallo de energía
- **CPU opera en 2 modos:** programming mode (descarga código remotamente desde cualquier ordenador) + run mode (ejecuta el código)
- Reemplazó a: drum sequencers, hard-wired relays, timers

**BPCS (Basic Process Control System):**
- Primera capa de protección contra condiciones inseguras en el equipamiento
- No tiene rutinas de diagnóstico para identificar fallos del sistema (diferencia vs SIS)
- Aplicable a todos los tipos de control loops: temperatura, batch, presión, flujo, feedback, feedforward

**SIS (Safety Instrumented System):**
- Sistema de control automatizado para proteger el entorno de fabricación en incidentes peligrosos
- **Anula/Override al BPCS** cuando BPCS no opera dentro de parámetros normales
- Compuesto por: sensores de campo, logic solvers, final control elements (válvulas on-off activadas neumáticamente controladas por solenoides)
- Funciona **independientemente** de otros sistemas de control
- Determinado mediante HAZOP, LOPA, risk graphs
- Última capa de protección antes de que el proceso entre en límites inseguros (relief valves, rupture disks, flare systems)

---

### Protocolos OT — Puertos críticos para examen 🔴

| Protocolo | Puerto | Nivel Purdue |
|-----------|--------|-------------|
| **Modbus** | **502** (TCP) | 3 |
| **Fieldbus** | **1089–91** | — |
| **Niagara Fox** | **1911, 4911** (TCP) | 0-1 |
| **PCWorx** | **1962** (TCP) | 0-1 |
| **DNP** | **19999** | — |
| **Ethernet/IP** | **2222** (TCP/UDP) | — |
| **IEC 60870-5-104 / Tase-2** | **2404** | 2-3 |
| **DNP3** | **20000** (TCP) | 2 |
| **ProConOS** | **20547** (TCP) | — |
| **PROFINET** | **34962–34964** | 0-1 |
| **EtherCAT** | **34980** | 0-1 |
| **Ethernet/IP (UDP)** | **44818** (UDP) | — |
| **Omron FINS** | **9600** (TCP/UDP) | 0-1 |
| **BACnet** | **47808** (UDP) | 0-1 |
| **Siemens S7** | **102** (TCP) | 0-1 |
| **HMI Sielco Sistemi Winlog** | **46824** (TCP) | — |
| **DNP3** | **20000** | 2 |
| **OpenOCD via GDB** | **333** | — |
| **OpenOCD via Telnet** | **4444** (TCP) | — |

**Modbus:** protocolo de comunicación serial para PLCs; comunicación en **plaintext sin autenticación**; usado también sobre Ethernet con TCP/IP.

**S7 Communication:** protocolo propietario de **Siemens** para familia S7-300/400; programación de PLC y acceso a datos PLC desde SCADA.

**DNP3:** Distributed Network Protocol 3; interconnects process automation components.

**BACnet:** Building Automation and Control network; estándares ASHRAE, ANSI, ISO 16484-5.

**WiMax:** IEEE **802.16**; frecuencias **2.5–5.8 GHz**; transfer rate **40 Mbps**; wireless MAN.

---

### Nmap para ICS/SCADA — Comandos esenciales 🔴

```bash
# Scan inicial ICS/SCADA (todos los puertos conocidos)
nmap -Pn -sT --scan-delay 1s --max-parallelism 1 \
  -p 80,102,443,502,530,593,789,1089-1091,1911,1962,2222,2404,\
     4000,4840,4843,4911,9600,19999,20000,20547,34962-34964,\
     34980,44818,46823,46824,55000-55003 <IP>

# Identificar sistemas HMI (Sielco Sistemi Winlog)
nmap -Pn -sT -p 46824 <IP>

# Siemens SIMATIC S7 PLCs (puerto 102)
nmap -Pn -sT -p 102 --script=s7-info <IP>

# Modbus devices (puerto 502) + Slave IDs
nmap -Pn -sT -p 502 --script modbus-discover <IP>
nmap -sT -Pn -p 502 --script modbus-discover \
  --script-args='modbus-discover.aggressive=true' <IP>

# BACnet devices (UDP 47808)
nmap -Pn -sU -p 47808 --script bacnet-info <IP>

# Ethernet/IP (UDP 44818)
nmap -Pn -sU -p 44818 --script enip-info <IP>

# Niagara Fox (TCP 1911, 4911)
nmap -Pn -sT -p 1911,4911 --script fox-info <IP>

# ProConOS (TCP 20547)
nmap -Pn -sT -p 20547 --script proconos-info <IP>

# Omron FINS (TCP/UDP 9600)
nmap -Pn -sT -p 9600 --script omron-info <IP>
nmap -Pn -sU -p 9600 --script omron-info <IP>

# PCWorx (TCP 1962)
nmap -Pn -sT -p 1962 --script pcworx-info <IP>
```

---

### modbus-cli — Hacking de PLC Schneider 🔴

```bash
# Instalar
gem install modbus-cli

# Leer registros (10 palabras)
modbus read <IP> %MW100 10          # Schneider address
modbus read <IP> 400101 10          # Modicon address

# Escribir registros (reemplazar 8 registros con valor 2)
modbus write <IP> %MW100 2 2 2 2 2 2 2 2
modbus write <IP> 400101 2 2 2 2 2 2 2 2

# Leer coils (valores booleanos ON/OFF)
modbus read <IP> 101 10
modbus read <IP> %M100 10

# Activar todos los coils (valor 1)
modbus write <IP> 101 1 1 1 1 1 1 1 1 1 1
modbus write <IP> %M100 1 1 1 1 1 1 1 1 1 1

# Capturar datos a fichero
modbus read --output SCADAregisters.txt <IP> 400101 200
modbus read --output SCADAcoils.txt <IP> 101 100
```

**Tipos de datos Modbus:**

| Tipo | Bits | Schneider | Modicon |
|------|------|-----------|---------|
| word (unsigned) | 16 | %MW100 | 400101 |
| integer (signed) | 16 | %MW100 | 400101 |
| float | 32 | %MF100 | 400101 |
| dword | 32 | %MD100 | N/A |
| Boolean (coils) | 1 | %M100 | 101 |

---

### Shodan para ICS/SCADA

```
port:502                         # Modbus-enabled ICS/SCADA
"Schneider Electric"             # Sistemas Schneider con Modbus
TM221ME16R                       # Schneider Electric TM221 PLCs
SCADA Country:"US"               # SCADA en USA
```

**CIRT.net:** base de datos de contraseñas por defecto para dispositivos ICS, routers, switches.

---

### COSMICENERGY — Malware OT 🔴

Malware ICS/OT diseñado para interrumpir el suministro eléctrico interactuando con dispositivos **IEC-104 (IEC 60870-5-104)** como RTUs. Similar a INDUSTROYER/INDUSTROYER.V2.

**Dos componentes:**
- **PIEHOP** (Python): conecta al servidor MSSQL, sube ficheros, emite comandos remotos al RTU
- **LIGHTWORK** (C++): genera mensajes ASDU IEC-104 para cambiar el estado de los IOAs (ON/OFF) → afecta a power-line switches y circuit breakers

**Comandos LIGHTWORK:**
1. `C_IC_NA_1` — station interrogation command (recupera estado del objetivo)
2. `C_SC_NA_1` — single command (cambia estado IOA a ON/OFF para cada dirección hardcoded)
3. `C_CS_NA_1` — clock synchronization command (sincroniza reloj remoto)

**Flujo del ataque:**
1. Reconocimiento (MSSQL servers con acceso a OT, credenciales, IPs IEC-104)
2. Acceso inicial (conexión a MSSQL remoto con credenciales robadas)
3. Ejecución de PIEHOP + LIGHTWORK → perturbaciones de energía ON/OFF
4. Lateral movement + limpieza de rastros (elimina ejecutables PIEHOP y LIGHTWORK)

---

### Fuxnet — Malware ICS

Malware ICS destructivo targeting OT environments (monitoring de safety systems e infraestructuras críticas). Capacidades: modificar datos cruciales, impedir acceso a sensor gateways, dañar sensores físicos. Daña gateways **reescribiendo el chip NAND** → deshabilita acceso remoto externo. Explota credenciales débiles para obtener root access. Usa **fuzzing** (paquetes malformados) para corromper sensores → lecturas incorrectas + daño físico.

---

### MITRE ATT&CK for ICS — Tácticas 🔴

| Táctica | Técnicas clave |
|---------|---------------|
| **Initial Access** | Drive-by compromise, exploiting public-facing apps, spear-phishing, removable media, supply-chain compromise, wireless compromise |
| **Execution** | CLI, cambio de modo operativo, APIs, GUI, hooking, scripting |
| **Persistence** | Modificar programa del controlador, module firmware (malicious), project file infection, system firmware |
| **Privilege Escalation** | Exploiting software, hooking (APIs redirection) |
| **Evasion** | Remover indicadores, rootkits, cambiar operator mode, masquerading |
| **Discovery** | Network sniffing, enumerar conexiones de red, identificar sistemas remotos, wireless sniffing |
| **Lateral Movement** | Default credentials, program download, remote services |
| **Collection** | Automated collection, information repositories, I/O image de PLC |
| **Command & Control** | Frequent ports (80, 443), connection proxy, standard app-layer protocols (HTTPS, Telnet, RDP) |
| **Inhibit Response** | Activar firmware update mode, bloquear mensajes de comando/reporte, alarm suppression, DoS |
| **Impair Process Control** | I/O brute-forcing, alterar parámetros, module firmware injection |
| **Impact** | Damage to property, loss of availability, denial of control, denial of view, loss of safety |

---

### HMI-Based Attacks — Vulnerabilidades 🔴

| Tipo | Descripción |
|------|-------------|
| **Memory Corruption** | Out-of-bound read/write, heap/stack-based buffer overflow; memoria alterada → crashes o ejecución no intencionada |
| **Credential Management** | Hardcoded passwords, almacenamiento en cleartext, protección inadecuada |
| **Lack of Auth/Authz + Insecure Defaults** | Transmisión en cleartext, sin cifrado, ActiveX inseguro; admin SCADA puede ver passwords de otros usuarios |
| **Code Injection** | SQL, OS, command injection; **Gamma** (lenguaje para HMIs) vulnerable a `EvalExpression` (evalúa código en runtime) |
| **Buffer Overflow** | Input excesivo overflow del buffer → ejecución de código arbitrario |
| **Path Traversal** | Acceso a directorios/ficheros fuera del web-root del servidor HMI web |

**HMI = "Hacker-Machine Interface"** (denominación en el contexto de ataques OT).

---

### Side-Channel Attacks en OT — Timing vs Power 🔴

| Tipo | Mecanismo | Detección |
|------|-----------|-----------|
| **Timing Analysis** | Mide el tiempo de autenticación por contraseña; loop carácter a carácter; cuantos más caracteres correctos, más tiempo | **Fácilmente detectable y bloqueada** |
| **Power Analysis** | Observa cambios en consumo de energía de semiconductores durante ciclos de reloj; oscilloscope mide tiempo entre pulsos; perfil de potencia revela datos procesados | **Difícil de detectar; dispositivo sigue operando tras ser infectado** |

**Por qué los atacantes prefieren power analysis:** el dispositivo atacado puede operar después de ser infectado → ataque más sigiloso y difícil de detectar.

---

### PLC Attacks 🔴

**PLC Rootkit Attack (= PLC Ghost Attack):**
1. Inyectar rootkit → control-flow attack contra el runtime del PLC para obtener la contraseña y root-level access
2. Mapear módulos I/O y sus ubicaciones en memoria para sobreescribir parámetros I/O
3. Manipular la secuencia de inicialización I/O → control total de operaciones PLC

El rootkit explota **flaws arquitectónicas de los microprocesadores** y bypasea mecanismos modernos de detección. El CPU del PLC opera en: **programming mode** (descarga código remotamente) y **run mode** (ejecuta código).

**Evil PLC Attack:**
1. Identificar PLC vulnerable via Shodan/Censys
2. Explotar firmware + weaponizar cambiando la lógica de programación via download procedures
3. Usar el PLC infectado para iniciar upload procedures en workstations conectados y ejecutar código arbitrario

---

### RF Remote Controllers — Ataques industriales

| Ataque | Descripción |
|--------|-------------|
| **Replay Attack** | Grabar y reproducir comandos RF para obtener control básico |
| **Command Injection** | Reverse engineering de protocolos RF → inyectar paquetes propios para control completo |
| **Abusing E-stop** | Enviar múltiples comandos de parada de emergencia → DoS |
| **Re-pairing malicious RF controller** | Hijack del controlador legítimo + pairing con controlador RF malicioso disfrazado |
| **Malicious Reprogramming** | Inyectar malware en el firmware de los remote controllers → acceso remoto persistente |

---

### OT Supply Chain Attacks

| Tipo | Mecanismo |
|------|-----------|
| Third-Party Software Compromise | Código malicioso inyectado en actualizaciones de software confiables |
| Hardware Manipulation | Alteración de componentes HW durante fabricación/distribución; firmware malicioso embebido |
| Service Provider Breach | Compromiso de proveedores de mantenimiento/soporte → acceso via credenciales robadas |
| Injection of Malicious Components | Substitución de partes legítimas durante envío por componentes comprometidos |
| Exploitation of Trusted Relationships | Explotar el acceso concedido a proveedores/subcontratistas para lateral movement |

---

### Fuzzing ICS Protocols — Fuzzowski

```bash
# BACnet (UDP 47808)
python -m fuzzowski 127.0.0.1 47808 -p udp -f bacnet -rt 0.5 -m BACnetMon

# Modbus (TCP 502)
python -m fuzzowski 127.0.0.1 502 -p tcp -f modbus -rt 1 -m modbusMon

# IPP (TCP 631)
python -m fuzzowski printer1 631 -f ipp -r get_printer_attribs --restart smartplug
```

---

### Herramientas OT — Mapa completo

| Herramienta | Categoría | Función |
|-------------|-----------|---------|
| Shodan | Info gathering | Buscar ICS/SCADA por puerto, país, PLC name |
| Kamerka-GUI | Info gathering | Localizar/mapear ICS expuestos a Internet + heatmaps |
| NetworkMiner | Sniffing | Passive sniffing + análisis PCAP sin generar tráfico |
| Malcolm | Protocol analysis | Network traffic analysis para ICS; dashboards OpenSearch + Arkime |
| Nessus (SCADA plugins) | Vuln scanning | Vulnerabilidades en ICS/SCADA con plugins SCADA |
| Skybox Vulnerability Control | Vuln scanning | Path analysis IT+OT; prioriza millones de vulnerabilidades |
| Microsoft Defender for IoT | Vuln scanning | Asset inventory IoT/ICS; vulnerabilidades nivel dispositivo y red |
| Fuzzowski | Fuzzing | Fuzz ICS protocols (Modbus, BACnet, IPP) |
| modbus-cli | Attack | Leer/escribir registros y coils Modbus en PLCs |
| mbtget | Attack | Transacciones Modbus TCP y RTU (Perl script) |
| Metasploit | Attack | Módulos SCADA para Modbus slaves |
| GDB | HW analysis | Debugging Linux para ejecuciones on-chip |
| OpenOCD | HW analysis | Conectar sistema al chip; GDB puerto 333, telnet 4444 |
| Binwalk | Firmware | Scan firmware binaries; identifica cifrado, particiones, filesystems |
| Radare2 | RE | Reverse engineering de binarios |
| Ghidra | RE | SRE framework; disassembly, decompilation, scripting |
| IDA Pro | RE | Disassembler; machine code → assembly legible |
| Fritzing | HW design | Diseño de diagramas y circuitos electrónicos |
| SmartRF Packet Sniffer | Sniffing | ZigBee, EasyLink, BLE; hardware USB + Wireshark |
| Conpot/GasPot | Honeypot | Decoys OT para atraer atacantes |
| Flowmon | Security | Monitorización continua y detección de anomalías en redes industriales |
| Nozomi Networks | Security | Visibilidad y seguridad OT/IoT |

---

### Zero-Trust en ICS — 5 pasos

1. **Definir la red:** identificar assets, datos vulnerables, aplicaciones críticas en control centers/factory floors
2. **Mapear el tráfico:** documentar flujos de tráfico para visibilidad completa
3. **Arquitecturar la red:** implementar ZTA con NGFW como segmentation gateway + capas adicionales de control de acceso
4. **Desarrollar política ZT:** whitelist de usuarios y dispositivos; definir quién, por qué, cuándo y qué recursos
5. **Monitorizar y mantener:** ZTA monitoriza tráfico + gestiona actualizaciones de todos los dispositivos

---

### Purdue Model — Seguridad por nivel

| Zona | Nivel | Ataques | Controles |
|------|-------|---------|-----------|
| Enterprise | 5-4 | Spear phishing, Ransomware | Firewalls, IPS, Anti-bot, URL filtering, SSL inspection, Antivirus, DLP |
| IDMZ | 3.5 | DoS | Anti-DoS, IPS, Antibot, Application control, ALF |
| Manufacturing | 3 | Ransomware, Bot infection, USB inseguros | Anti-bot, IPS, Sandboxing, Application control, Traffic encryption, Port protection |
| Manufacturing | 2-1 | DDoS, protocolos no cifrados, credenciales por defecto | IPS, Firewall, IPsec, Security gateways, comandos RTU/PLC autorizados |
| Manufacturing | 0 | Physical security breach | Point-to-point comm, MAC authentication, security gateways adicionales |

---

## 2. Exam Traps ⚠️ 🔴

⚠️ **[Purdue Model — número de niveles]** El Purdue Model tiene **3 zonas** (Enterprise/IDMZ/Manufacturing) divididas en **6 niveles operativos** (0 a 5) + el nivel 3.5 para el IDMZ. El examen puede preguntar cuántos niveles: **6** (del 0 al 5).

⚠️ **[PLC — 2 modos del CPU]** Programming mode: descarga código remotamente desde cualquier ordenador. Run mode: ejecuta el código real. El examen puede preguntar en qué modo un atacante puede descargar código malicioso → **programming mode**.

⚠️ **[PLC en nivel 1 vs nivel 2]** PLC en nivel 2 = función **supervisory**. PLC en nivel 1 = función de **control**. El examen puede usar ambos; el contexto determina la respuesta.

⚠️ **[Modbus — seguridad inherente]** Modbus comunica en **plaintext sin autenticación**. Es la vulnerabilidad fundamental de Modbus. Si el examen pregunta por qué Modbus es vulnerable → no tiene cifrado ni autenticación incorporados.

⚠️ **[SCADA vs DCS — distribución geográfica]** SCADA: **distribuido geográficamente** (wide area). DCS: producción en la **misma localización geográfica**. Criterio: ¿wide area? → SCADA. ¿Misma planta? → DCS.

⚠️ **[SIS vs BPCS — relación jerárquica]** SIS **anula/override** al BPCS cuando BPCS no opera dentro de parámetros normales. SIS funciona **independientemente** de otros sistemas de control. Si el examen pregunta qué sistema es el "override" de seguridad → SIS.

⚠️ **[BPCS — ausencia de diagnósticos]** BPCS NO tiene rutinas de diagnóstico para identificar fallos del sistema. Esta es su diferencia clave vs SIS. Si el examen pregunta qué sistema carece de rutinas de diagnóstico → BPCS.

⚠️ **[Power Analysis vs Timing Analysis]** Power Analysis es preferida por los atacantes porque es **difícil de detectar** y el dispositivo sigue operando tras el ataque. Timing Analysis es **fácilmente detectable**. Si el examen pregunta cuál es más sigilosa → Power Analysis.

⚠️ **[PLC Rootkit = PLC Ghost Attack]** Son sinónimos. El examen puede usar cualquiera de los dos nombres. Se requiere conocimiento profundo de la arquitectura PLC para ejecutarlo.

⚠️ **[COSMICENERGY — protocolo objetivo]** Ataca dispositivos **IEC-104** (IEC 60870-5-104). Sus dos componentes son **PIEHOP** (Python, MSSQL) y **LIGHTWORK** (C++, genera mensajes ASDU IEC-104). Si el examen menciona power grid + IEC-104 + Python/C++ → COSMICENERGY.

⚠️ **[HMI — Gamma language]** Gamma es el lenguaje específico para HMIs vulnerable a `EvalExpression` (code injection). Si el examen menciona lenguaje HMI con vulnerabilidad de code injection en runtime → Gamma / EvalExpression.

⚠️ **[Siemens S7 — puerto]** Puerto **102** para S7 Communication. Comando Nmap: `--script=s7-info`. Si el examen pregunta el puerto de comunicación Siemens PLC → 102.

⚠️ **[WiMax — IEEE standard y frecuencias]** WiMax = **IEEE 802.16**; frecuencias **2.5–5.8 GHz**; **40 Mbps**. No confundir con Wi-Fi (802.11n).

⚠️ **[PLC RAM vs ROM]** RAM almacena programas escritos por el usuario. ROM almacena OS, drivers y programas de aplicación. **Retentive memory** preserva programas de usuario durante fallos de alimentación.

---

## 3. Nemotécnicos

### Purdue Model — 7 niveles (0 a 5 + 3.5)
**"Físico-Básico-Control-Operativo-IDMZ-Logística-Enterprise"** (0→5):
- **0** = **F**ísico (sensores, actuadores)
- **1** = **B**ásico controls (PLCs, RTUs, IEDs)
- **2** = **C**ontrol systems (SCADA, DCS, HMIs)
- **3** = **O**peraciones (MES, historians)
- **3.5** = **IDMZ** (DMZ industrial)
- **4** = **L**ogística de negocio
- **5** = **E**nterprise (B2B, B2C)

### Componentes ICS — "SCADA DCS BPCS SIS PLC RTU HMI IED"
Mnemónico: "**S**iempre **D**escubro **B**uenas **S**oluciones **P**ensando **R**acionalmente **H**aciendo **I**nvestigación **E**xaustiva"

### Módulos PLC — 3
**"CPU-PS-IO"**: CPU Module → Power Supply Module → I/O Modules
Voltajes Power Supply: **5V DC** para circuitos del PLC, **24V DC** para sensores/actuadores

### Modos CPU PLC — 2
**"PRun"**: **P**rogramming (descarga código remoto) → **Run** (ejecuta código)

### COSMICENERGY — 2 componentes
- **PIEHOP** = **P**ython + MSSQL (**P**ipes de datos hacia el RTU)
- **LIGHTWORK** = **C++** + IEC-104 ASDU (**L**luz ON/OFF en el grid)

### Puertos ICS críticos (los 5 más preguntados)
- **502** = Modbus
- **102** = Siemens S7
- **20000** = DNP3
- **44818** = Ethernet/IP (UDP)
- **47808** = BACnet (UDP)

### Side-Channel attacks — detección
- **T**iming = **T**rivial to detect (fácilmente detectable)
- **P**ower = **P**rofound difficulty (difícil de detectar)

---

## 4. Flashcards

**Q:** ¿Cuál es la diferencia geográfica entre SCADA y DCS?
**A:** **SCADA** controla sistemas distribuidos en una **amplia área geográfica**. **DCS** controla sistemas de producción en la **misma localización geográfica**. SCADA: power grids, pipelines. DCS: planta química, refinería, nuclear.

---

**Q:** ¿Qué hace la retentive memory de un PLC y por qué es importante?
**A:** Preserva los programas de usuario y datos durante **fallos de alimentación**. Esto permite reanudar la ejecución del programa una vez restaurada la energía, sin necesidad de monitor o teclado para reprogramar el procesador.

---

**Q:** ¿En qué nivel del Purdue Model se sitúa el IDMZ y cuál es su propósito?
**A:** Nivel **3.5**. Es una barrera entre la manufacturing zone (OT) y la enterprise zone (IT). Contiene errores e intrusiones dentro de su capa permitiendo que la producción continúe sin interrupciones. Incluye domain controllers, database replication servers y proxy servers.

---

**Q:** ¿Por qué los atacantes prefieren el ataque de Power Analysis sobre el Timing Analysis?
**A:** El **Power Analysis** es **difícil de detectar** y el dispositivo atacado puede **seguir operando después de ser infectado**. El Timing Analysis es fácilmente detectable y bloqueable.

---

**Q:** ¿Cuáles son los dos componentes de COSMICENERGY y en qué lenguaje está escrito cada uno?
**A:** **PIEHOP** (Python): conecta al servidor MSSQL, sube ficheros, emite comandos remotos al RTU. **LIGHTWORK** (C++): genera mensajes ASDU IEC-104 para cambiar el estado de IOAs (ON/OFF), afectando power-line switches y circuit breakers.

---

**Q:** ¿Cuál es la diferencia entre un PLC Rootkit Attack y un Evil PLC Attack?
**A:** **PLC Rootkit (= Ghost Attack):** inyectar rootkit en el PLC, mapear I/O en memoria y manipular la secuencia de inicialización I/O para control total. **Evil PLC Attack:** convertir un PLC vulnerable en un Evil PLC cambiando su lógica de programación, luego usarlo para ejecutar código arbitrario en workstations conectados.

---

**Q:** ¿Qué vulnerabilidad fundamental tiene Modbus y cómo la explotan los atacantes?
**A:** Modbus comunica en **plaintext sin autenticación**. Los atacantes generan y envían query packets similares a los legítimos para **acceder y manipular los registros y coils** de los Modbus Slaves.

---

**Q:** ¿En qué modo del CPU del PLC puede un atacante descargar código malicioso remotamente?
**A:** En **programming mode** — el PLC puede descargar código remotamente desde cualquier ordenador. Run mode solo ejecuta el código ya almacenado.

---

**Q:** ¿Qué hace el SIS cuando el BPCS no opera dentro de parámetros normales?
**A:** El **SIS anula/override al BPCS** y proporciona un entorno de control automatizado para detectar y responder al proceso crítico, llevando el sistema a un estado seguro (preservar o cambiar a estado seguro = shutdown). Funciona **independientemente** de otros sistemas de control.

---

**Q:** ¿Qué puerto usa BACnet y para qué sirve en Nmap el script `bacnet-info`?
**A:** Puerto **UDP 47808**. El script `bacnet-info` recupera: nombre del vendor, nombre del dispositivo, número de serie, versión de firmware, y detecta BACnet Broadcast Management Devices (BBMDs).

---

**Q:** ¿Qué puerto usa Siemens SIMATIC S7 y qué script Nmap se usa para escanearlo?
**A:** Puerto **TCP 102**. Script Nmap: `--script=s7-info`. S7 Communication es el protocolo propietario Siemens para la familia S7-300/400 para programación de PLC y acceso SCADA.

---

**Q:** ¿Cuál es el estándar IEEE de WiMax y en qué rango de frecuencias opera?
**A:** **IEEE 802.16**. Frecuencias: **2.5–5.8 GHz**. Transfer rate: **40 Mbps**. Es un estándar para wireless MAN (Metropolitan Area Networks).

---

**Q:** ¿Qué es Gamma en el contexto de HMI y cuál es su vulnerabilidad específica?
**A:** Gamma es un lenguaje de dominio específico para HMIs diseñado para desarrollar aplicaciones UI y de control de alta velocidad. Su vulnerabilidad es **EvalExpression** (evalúa, compila y ejecuta código en runtime), explotable para ejecutar scripts/comandos arbitrarios en el sistema SCADA objetivo.

---

**Q:** ¿Cuáles son los 3 comandos IEC-104 que ejecuta LIGHTWORK en orden?
**A:** 1) `C_IC_NA_1` (station interrogation — recupera estado), 2) `C_SC_NA_1` (single command — cambia IOA ON/OFF), 3) `C_CS_NA_1` (clock synchronization — sincroniza reloj del RTU remoto).

---

**Q:** ¿Qué diferencia BPCS de SIS en términos de diagnósticos?
**A:** **BPCS** NO tiene rutinas de diagnóstico para identificar fallos del sistema. **SIS** sí las tiene y funciona independientemente de otros sistemas. El SIS es la capa de seguridad que actúa cuando el BPCS falla.

---

**Q:** ¿Cuál es el comando modbus-cli para leer 10 palabras del registro usando dirección Modicon?
**A:** `modbus read <IP> 400101 10` (Modicon address). Usando Schneider: `modbus read <IP> %MW100 10`.

---

**Q:** ¿Qué hace Malcolm en entornos ICS y qué dos interfaces proporciona?
**A:** Herramienta de análisis de tráfico de red para entornos ICS. Interfaces: **OpenSearch dashboard** (dashboards predefinidos de protocolos de red) y **Arkime** (identificar sesiones de red con incidentes de seguridad sospechosos).

---

## 5. Confusión frecuente

### SCADA vs DCS
- **SCADA:** wide area geográfica; sectores: power grids, pipelines, telecomunicaciones; recopila datos de field devices y los transmite a un sistema central.
- **DCS:** misma localización geográfica; high-redundancy; centralized supervisory unit con múltiples local controllers; sectores: química, nuclear, refinerías.
- **Criterio:** ¿múltiples ubicaciones geográficas? → SCADA. ¿Compleja producción en una misma planta? → DCS.

---

### BPCS vs SIS
- **BPCS:** primera capa de protección; sin diagnósticos; controla/optimiza el proceso normal.
- **SIS:** anula al BPCS cuando falla; funciona independientemente; usa HAZOP/LOPA para requerimientos; lleva el sistema a estado seguro.
- **Criterio:** ¿proceso normal dentro de parámetros? → BPCS. ¿Override de seguridad cuando BPCS falla? → SIS.

---

### PLC Rootkit Attack vs Evil PLC Attack
- **PLC Rootkit:** el atacante inyecta directamente en el PLC y toma control de los I/O; requiere conocimiento profundo de arquitectura PLC.
- **Evil PLC:** el atacante usa un PLC comprometido como vector para atacar los workstations conectados (ejecutar código arbitrario via upload procedures).
- **Criterio:** ¿control directo de I/O del PLC? → Rootkit. ¿Usar PLC como trampa para workstations? → Evil PLC.

---

### Modbus puerto 502 vs DNP3 puerto 20000
- **Modbus:** protocolo serial para PLCs; plaintext sin auth; TCP 502; comando Nmap: `modbus-discover`.
- **DNP3:** Distributed Network Protocol 3; TCP 20000; interconecta componentes de automatización de procesos; más seguro.
- **Criterio:** ¿protocolo sin cifrado ni auth, el más explotado? → Modbus (502). ¿Telecontrol eléctrico? → DNP3 (20000).

---

### Timing Analysis vs Power Analysis
- **Timing Analysis:** mide tiempo de respuesta del sistema durante autenticación → fácilmente detectable/bloqueable.
- **Power Analysis:** observa consumo de energía en semiconductores con oscilloscope → difícil de detectar; dispositivo sigue operando.
- **Criterio:** ¿ataque sigiloso preferido por atacantes? → Power Analysis. ¿Detectable y bloqueable? → Timing Analysis.
