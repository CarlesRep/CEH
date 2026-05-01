# M16_01_WirelessConcepts.md
> Módulo 16 / Subapartado 1 — Wireless Concepts

---

## 1. Conceptos y definiciones

### 🔴 Terminología wireless clave

| Término | Definición clave |
|---|---|
| **GSM** | Global System for Mobile Communications — sistema universal para transmisión de datos móviles inalámbricos a nivel mundial |
| **Bandwidth** | Cantidad de información que puede transmitirse sobre una conexión; medida en bps |
| **Access Point (AP)** | Conecta dispositivos inalámbricos a red inalámbrica/cableada; actúa como switch/hub entre LAN cableada y red inalámbrica |
| **BSSID** | Dirección **MAC** del AP o base station que ha configurado un BSS; el usuario no suele ser consciente del BSS al que pertenece |
| **ISM Band** | Conjunto de frecuencias usadas por comunidades industriales, científicas y médicas internacionales |
| **Hotspot** | Lugar donde una red inalámbrica está disponible para uso público (Wi-Fi pública) |
| **Association** | Proceso de conectar un dispositivo inalámbrico a un AP |
| **SSID** | Identificador único de 32 caracteres alfanuméricos de una WLAN; case-sensitive; actúa como identificador inalámbrico de la red |
| **OFDM** | Orthogonal Frequency-Division Multiplexing — método de modulación digital que divide la señal en múltiples frecuencias portadoras ortogonales; soporta mayores tasas de bits |
| **MIMO-OFDM** | Influye en la eficiencia espectral de servicios 4G y 5G; reduce interferencias y aumenta la robustez del canal |
| **DSSS** | Direct-Sequence Spread Spectrum — multiplica la señal de datos con un código pseudoaleatorio; protege contra interferencias o jamming |
| **FHSS** | Frequency-Hopping Spread Spectrum — también llamado FH-CDMA; transmite señales cambiando rápidamente la portadora entre muchas frecuencias; usa secuencia pseudoaleatoria conocida por sender y receiver; disminuye la eficiencia de interceptación o jamming no autorizado |

---

### SSID — Detalles clave de seguridad

- Identificador de WLAN: **32 caracteres alfanuméricos**, **case-sensitive**
- Parte por defecto del **frame header** de paquetes sobre WLAN
- Los APs responden a **probe requests** con **probe responses** que incluyen el SSID (salvo si está oculto)
- Todos los dispositivos y APs en la misma WLAN deben usar el **mismo SSID**
- **No proporciona seguridad** — el SSID se obtiene fácilmente en texto plano de los paquetes
- Default SSID en muchos productos comerciales = **nombre del vendor**
- Un "non-secure access mode" permite conectarse con SSID configurado, blank SSID, o SSID `"any"`
- Si el SSID cambia, el admin debe reconfigurarlo en **todos** los clientes

---

### 🔴 Wi-Fi Authentication — PSK vs. Centralized

| Modo | Nombre alternativo | Contexto | Mecanismo |
|---|---|---|---|
| **PSK (Pre-Shared Key)** | WPA-PSK / WPA2-PSK / WPA/WPA2-Personal | Hogar, pequeñas oficinas | Contraseña compartida única introducida manualmente en el router y cada dispositivo |
| **Centralized Authentication** | WPA/WPA2-Enterprise / 802.1X mode | Empresas, universidades, gobierno | Servidor **RADIUS** gestiona credenciales individuales por usuario (username+password o certificado digital); envía authentication keys tanto al AP como al cliente |

La seguridad de WPA/WPA2-Personal depende de la **complejidad y secretismo de la PSK**.

---

### 🔴 Wireless Standards — Tabla comparativa

| Estándar | También llamado | Frecuencia (GHz) | Velocidad máx | Modulación | Característica clave |
|---|---|---|---|---|---|
| **802.11** | Wi-Fi | 2.4 | 1-2 Mbps | DSSS, FHSS | Estándar original (1997) |
| **802.11a** | — | **5** | **54 Mbps** | **OFDM** | Primer amendment; alta velocidad pero sensible a obstáculos |
| **802.11b** | — | **2.4** | **11 Mbps** | **DSSS** | 1999; ISM band |
| **802.11d** | — | — | — | — | Enhancement de 802.11a/b para portabilidad global (variaciones de frecuencia, potencia, bandwidth) |
| **802.11e** | — | — | — | — | QoS para voz, VoIP y video; define mecanismos QoS en **capa MAC (Layer 2)** |
| **802.11g** | — | **2.4** | **54 Mbps** | **OFDM** | Compatible con 802.11b; misma banda 2.4 GHz |
| **802.11i** | — | — | — | — | Seguridad WLAN mejorada con **TKIP y AES**; define **WPA2-Enterprise/WPA2-Personal** |
| **802.11n** | — | **2.4 y 5** | **54-600 Mbps** | **MIMO-OFDM** | Usa antenas MIMO; trabaja en ambas bandas |
| **802.11ah** | **Wi-Fi HaLow** | **0.9 (900 MHz)** | — | — | Para IoT; mayor rango, mayores tasas de datos que estándares anteriores |
| **802.11ac** | Wi-Fi 5 | **5** | Gigabit | — | High-throughput; Gigabit networking |
| **802.11ad** | — | **60** | — | — | Nueva capa física; velocidad propagación mucho mayor que 2.4/5 GHz |
| **802.11ax** | **Wi-Fi 6** | **2.4, 5, 6** | **2400 Mbps** | **OFDMA**, 1024-QAM | BSS Coloring, Target Wake Time (TWT); ideal para entornos densos |
| **802.11be** | **Wi-Fi 7** | — | **3000 Mbps** | — | Multilink Operation (MLO); 30 Gbps teórico; para VR, AR, IoT avanzado |
| **802.12** | — | — | **100 Mbps** | — | Demand priority protocol; compatible con 802.3 y 802.5 |
| **802.15** | WPAN | — | — | — | Wireless Personal Area Network |
| **802.15.1** | **Bluetooth** | **2.4** | — | GFSK, etc. | Distancias cortas; dispositivos fijos o móviles |
| **802.15.4** | **ZigBee** | 0.868, 0.915, 2.4 | **250 Kbps** | O-QPSK, GFSK, BPSK | Baja tasa de datos y complejidad; mesh network; mayor duración de batería |
| **802.15.5** | — | — | — | — | Full-mesh o half-mesh topology; network initialization, addressing, unicasting |
| **802.16** | **WiMAX** | 2-11 | 34-1000 Mbps | SOFDMA | Broadband wireless MANs; point-to-multipoint architecture |

---

### 🔴 Tipos de Antenas Wireless

| Antena | Característica clave | Uso en ataques |
|---|---|---|
| **Directional** | Emite y recibe en una sola dirección; reduce interferencias | — |
| **Omnidirectional** | Irradia en todas las direcciones; patrón horizontal 360°; buena para estaciones de radio | Cobertura general |
| **Parabolic Grid** | Semi-plato de rejilla de aluminio; transmisiones Wi-Fi de muy larga distancia (~10 millas); señales polarizadas horizontal o verticalmente | Escuchas a larga distancia, **Layer-1 DoS**, **MITM attacks** |
| **Yagi (Yagi-Uda)** | Unidireccional; frecuencias 10 MHz a VHF/UHF; **alta ganancia** y bajo SNR; patrón end-fire; componentes: reflector + dipolo + directors | Comunicaciones direccionales de largo alcance |
| **Dipole** | Conductor recto de media longitud de onda; también llamado doublet; bilateralmente simétrico; antena balanceada | Uso general |
| **Reflector** | Concentra energía EM en un punto focal; generalmente parabólicas; mayor reflector = mayor ganancia; alto coste de fabricación | Satélites, comunicaciones de largo alcance |

---

### Tipos de Redes Inalámbricas

| Tipo | Descripción |
|---|---|
| **Extension to Wired Network** | APs entre red cableada y dispositivos inalámbricos; SAPs (Software APs) y HAPs (Hardware APs) |
| **Multiple Access Points** | Múltiples APs con áreas superpuestas para permitir **roaming**; se pueden usar extension points como relays |
| **LAN-to-LAN Wireless** | APs interconectan LANs diferentes; todos los HAPs pueden interconectarse con otros HAPs |
| **3G/4G/5G Hotspot** | Proporciona Wi-Fi access a dispositivos Wi-Fi (MP3, tablets, cámaras, PDAs, notebooks) |

---

### Software APs vs. Hardware APs

| Tipo | Descripción |
|---|---|
| **Software APs (SAPs)** | Se ejecutan en un ordenador equipado con wireless NIC; se conectan a red cableada |
| **Hardware APs (HAPs)** | Soportan la mayoría de las funcionalidades wireless; pueden interconectarse con otros HAPs |

---

## 2. Exam Traps ⚠️

⚠️ **[BSSID = MAC address del AP]**
BSSID es la dirección **MAC** del AP (no la IP, no el SSID). El usuario normalmente no es consciente del BSS al que pertenece. El examen puede presentar BSSID como "IP del AP" o como "nombre del AP".

⚠️ **[SSID: 32 caracteres alfanuméricos, case-sensitive]**
SSID tiene exactamente 32 caracteres alfanuméricos y es **case-sensitive**. El examen puede presentar otros valores (24, 64 caracteres) o decir que no es case-sensitive.

⚠️ **[SSID no proporciona seguridad]**
A pesar de ser el identificador de la red, el SSID se obtiene fácilmente en texto plano de los paquetes. No es un mecanismo de seguridad. El examen puede presentarlo como una medida de seguridad.

⚠️ **[802.11a usa OFDM en 5 GHz; 802.11b usa DSSS en 2.4 GHz]**
La confusión más clásica: 802.11a = 5 GHz + OFDM + 54 Mbps. 802.11b = 2.4 GHz + DSSS + 11 Mbps. El examen intercambia las frecuencias o las modulaciones.

⚠️ **[802.11g: compatible con 802.11b, misma banda 2.4 GHz]**
802.11g trabaja en 2.4 GHz (mismo que 802.11b) y soporta 54 Mbps con OFDM. Es **compatible con 802.11b** — los dispositivos 802.11b pueden trabajar directamente con un AP 802.11g. El examen puede presentar 802.11g como incompatible con 802.11b.

⚠️ **[802.11i: define WPA2 + implementa TKIP y AES]**
802.11i es el estándar de seguridad que define **WPA2-Enterprise y WPA2-Personal** e implementa **TKIP y AES**. El examen puede confundirlo con 802.11e (QoS) o presentarlo como solo TKIP.

⚠️ **[802.11e: QoS en capa MAC (Layer 2)]**
802.11e define mecanismos de QoS específicamente en **Layer 2 (MAC layer)** para voz, VoIP y video. El examen puede ubicarlo en Layer 3 o Layer 1.

⚠️ **[802.11ax = Wi-Fi 6: OFDMA, no OFDM; soporta 6 GHz además de 2.4 y 5]**
Wi-Fi 6 (802.11ax) usa **OFDMA** (Multiple Access) no OFDM simple, y trabaja en **2.4, 5 y 6 GHz**. El examen puede presentarlo con la modulación de 802.11g o sin la banda de 6 GHz.

⚠️ **[802.11ah = Wi-Fi HaLow: 900 MHz para IoT]**
Wi-Fi HaLow trabaja en la banda de **900 MHz** (no 2.4, no 5 GHz). Diseñado específicamente para IoT con mayor alcance. El examen puede asignarle una frecuencia diferente.

⚠️ **[ZigBee (802.15.4): 250 Kbps, mesh network, mayor vida de batería]**
ZigBee tiene tasa de datos de **250 Kbps** (muy baja), transmite a larga distancia a través de **mesh network** y su uso **aumenta la vida de la batería**. El examen puede confundirlo con Bluetooth en velocidad o frecuencia.

⚠️ **[Parabolic Grid Antenna: ~10 millas de alcance, usada en Layer-1 DoS y MITM]**
La antena parabolic grid tiene alcance de aproximadamente **10 millas** y el libro la asocia específicamente con ataques Layer-1 DoS y MITM. El examen puede preguntar qué antena se usa para ataques de largo alcance.

---

## 3. Nemotécnicos

**802.11 estándares core — frecuencia y modulación:**
- **a = 5 GHz, OFDM, 54 Mbps** → "A está arriba (5 GHz)"
- **b = 2.4 GHz, DSSS, 11 Mbps** → "B está abajo (2.4 GHz)"
- **g = 2.4 GHz, OFDM, 54 Mbps** → "G = Good (mismo rango que b pero más rápido)"
- **n = 2.4+5 GHz, MIMO-OFDM** → "N = Not just one band"
- **ax = Wi-Fi 6 = 2.4+5+6 GHz, OFDMA** → "aXtra bands"
- **be = Wi-Fi 7, MLO, 3 Gbps** → "BE the future"

**Estándares especiales:**
- **802.11d** → "d=dominio" (portabilidad global, variaciones de frecuencia)
- **802.11e** → "e=enterprise QoS" (voz, VoIP, video, Layer 2)
- **802.11i** → "i=integrity/security" (TKIP+AES, WPA2)
- **802.11ah** → "ah=almost home (IoT)" (900 MHz, IoT)
- **802.11ad** → "ad=advanced speed" (60 GHz)

**No-Wi-Fi standards:**
- **802.15.1** = Bluetooth (2.4 GHz)
- **802.15.4** = ZigBee (250 Kbps, mesh, IoT, battery life)
- **802.16** = WiMAX (broadband MAN, point-to-multipoint)

**Antenas** → **"D-O-P-Y-D-R"**:
**D**irectional · **O**mnidirectional · **P**arabolic Grid (10 millas, DoS/MITM) · **Y**agi (end-fire, VHF/UHF) · **D**ipole (doublet, media longitud de onda) · **R**eflector (focal point, parabólica)

**PSK vs. RADIUS** → "Personal=Password, Enterprise=RADIUS"

**DSSS vs. FHSS:**
- DSSS: multiplica la señal con código pseudoaleatorio
- FHSS: salta entre frecuencias usando secuencia pseudoaleatoria (conocida por sender Y receiver)

---

## 4. Flashcards

**Q:** ¿Cuántos caracteres tiene un SSID y es case-sensitive?
**A:** 32 caracteres alfanuméricos; sí es case-sensitive.

---

**Q:** ¿Qué es el BSSID?
**A:** La dirección MAC del AP o base station que ha configurado un BSS (Basic Service Set).

---

**Q:** ¿Cuál es la diferencia entre WPA/WPA2-Personal y WPA/WPA2-Enterprise?
**A:** Personal (PSK): contraseña compartida única; ideal para hogar/pequeñas oficinas. Enterprise (802.1X): servidor RADIUS con credenciales individuales por usuario; para entornos corporativos con altos requisitos de seguridad.

---

**Q:** ¿Cuáles son la frecuencia, modulación y velocidad de 802.11a?
**A:** 5 GHz, OFDM, 54 Mbps.

---

**Q:** ¿Cuáles son la frecuencia, modulación y velocidad de 802.11b?
**A:** 2.4 GHz, DSSS, 11 Mbps.

---

**Q:** ¿Por qué 802.11g es compatible con 802.11b?
**A:** Porque ambos trabajan en la misma banda 2.4 GHz — los dispositivos 802.11b pueden trabajar directamente con un AP 802.11g.

---

**Q:** ¿Qué protocolos de cifrado implementa el estándar 802.11i?
**A:** TKIP (Temporal Key Integrity Protocol) y AES (Advanced Encryption Standard). También define WPA2-Enterprise y WPA2-Personal.

---

**Q:** ¿Qué estándar define QoS para voz, VoIP y video y en qué capa opera?
**A:** IEEE 802.11e — define mecanismos de QoS en Layer 2 (MAC layer).

---

**Q:** ¿Cuál es el nombre de Wi-Fi 6 y qué lo distingue de Wi-Fi 5?
**A:** 802.11ax (Wi-Fi 6). Usa OFDMA (no OFDM), soporta 2.4, 5 y 6 GHz, incluye BSS Coloring y Target Wake Time (TWT), ideal para entornos densos. Velocidad hasta 9.6 Gbps.

---

**Q:** ¿Qué es 802.11ah y cuál es su frecuencia?
**A:** Wi-Fi HaLow — trabaja en la banda de 900 MHz; diseñado para IoT con mayor alcance y tasas de datos más altas que estándares anteriores.

---

**Q:** ¿Qué es ZigBee (802.15.4) y qué lo caracteriza?
**A:** Estándar de baja tasa de datos (250 Kbps) y baja complejidad; transmite a larga distancia vía mesh network; aumenta la vida de la batería. Trabaja en 0.868, 0.915 y 2.4 GHz.

---

**Q:** ¿Qué es WiMAX y qué arquitectura usa?
**A:** IEEE 802.16 — estándar para broadband wireless MANs (Metropolitan Area Networks); arquitectura point-to-multipoint.

---

**Q:** ¿Cuál es la diferencia entre DSSS y FHSS?
**A:** DSSS: multiplica la señal original con un código pseudoaleatorio de ruido. FHSS: cambia rápidamente la frecuencia portadora entre muchos canales usando secuencia pseudoaleatoria conocida por sender y receiver.

---

**Q:** ¿Qué antena se usa para ataques de largo alcance incluyendo Layer-1 DoS y MITM?
**A:** Parabolic Grid Antenna — alcance de aproximadamente 10 millas; muy focused radio beams.

---

**Q:** ¿Cuáles son las características del Yagi antenna?
**A:** Unidireccional (Yagi-Uda); frecuencias 10 MHz a VHF/UHF; alta ganancia y bajo SNR; patrón end-fire; componentes: reflector + dipolo + directors.

---

**Q:** ¿Por qué el SSID no proporciona seguridad a una WLAN?
**A:** Porque se obtiene fácilmente en texto plano de los paquetes de la red.

---

**Q:** ¿Cuál es la velocidad máxima de 802.11be (Wi-Fi 7) y qué característica nueva introduce?
**A:** Hasta 30 Gbps teórico (3000 Mbps en la tabla). Introduce Multilink Operation (MLO) para agregar múltiples canales de diferentes bandas simultáneamente.

---

**Q:** ¿En qué se diferencia una antena omnidireccional de una directional?
**A:** Omnidireccional: irradia EM en todas las direcciones con patrón horizontal de 360°. Directional: emite y recibe solo en una dirección; reduce interferencias.

---

**Q:** ¿Qué es roaming en el contexto de Multiple Access Points?
**A:** La capacidad de moverse sin interrupciones entre APs cuyas áreas se superponen — el dispositivo cambia de AP automáticamente manteniendo la conectividad.

---

**Q:** ¿Cuáles son las características de 802.11n?
**A:** Trabaja en 2.4 GHz y 5 GHz (ambas bandas); usa antenas MIMO-OFDM; velocidad 54-600 Mbps.

---

## 5. Confusión frecuente

**802.11a vs. 802.11b — las dos más confundidas**
- 802.11a: 5 GHz, OFDM, 54 Mbps. Más sensible a obstáculos. Primera enmienda al estándar.
- 802.11b: 2.4 GHz, DSSS, 11 Mbps. Banda ISM. Creado en 1999.
- Criterio: "5 GHz + OFDM + 54 Mbps" → 802.11a. "2.4 GHz + DSSS + 11 Mbps" → 802.11b.

---

**802.11e vs. 802.11i — ambas son enmiendas de seguridad/calidad**
- 802.11e: QoS para voz/VoIP/video en Layer 2 (MAC layer). "e" = enhanced quality.
- 802.11i: seguridad WLAN con TKIP y AES; define WPA2. "i" = integrity/security.
- Criterio: "QoS para aplicaciones en tiempo real" → 802.11e. "Cifrado y seguridad WLAN (WPA2)" → 802.11i.

---

**OFDM vs. OFDMA — misma base, distinto acceso**
- OFDM: Orthogonal Frequency-Division Multiplexing — método de modulación. Usado en 802.11a, g, n.
- OFDMA: OFDM + Multiple Access — permite que múltiples usuarios compartan simultáneamente los sub-canales. Usado en 802.11ax (Wi-Fi 6). Más eficiente en entornos densos.
- Criterio: "modulación en 802.11a/g/n" → OFDM. "Gestión eficiente de múltiples conexiones en Wi-Fi 6" → OFDMA.

---

**Bluetooth (802.15.1) vs. ZigBee (802.15.4)**
- Bluetooth (802.15.1): 2.4 GHz; cortas distancias; intercambio de datos entre dispositivos fijos o móviles.
- ZigBee (802.15.4): 250 Kbps; mesh network; larga distancia; mayor vida de batería; IoT.
- Criterio: "intercambio de datos en distancias cortas (auriculares, móvil)" → Bluetooth. "Red mesh de baja potencia para IoT/sensores" → ZigBee.

---

**PSK vs. RADIUS (WPA-Personal vs. WPA-Enterprise)**
- PSK/Personal: una contraseña compartida para todos; manual; simple; hogar/PYME.
- RADIUS/Enterprise (802.1X): credenciales individuales por usuario; certificados digitales; servidor centralizado; entornos corporativos grandes.
- Criterio: "todos usan la misma contraseña" → PSK. "Cada usuario tiene credenciales únicas verificadas por servidor" → RADIUS/Enterprise.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un administrador de red configura una WLAN corporativa y quiere asegurarse de que todos los dispositivos puedan identificar correctamente la red. El SSID que desea usar es "CorpNet2024". ¿Cuántos caracteres puede tener un SSID como máximo y es sensible a mayúsculas?

A) 64 caracteres alfanuméricos; no es case-sensitive
B) 32 caracteres alfanuméricos; sí es case-sensitive
C) 32 caracteres alfanuméricos; no es case-sensitive
D) 48 caracteres alfanuméricos; sí es case-sensitive

**Respuesta correcta: B**
Un SSID tiene exactamente 32 caracteres alfanuméricos y es case-sensitive. "CorpNet2024" y "corpnet2024" serían SSIDs distintos. El estándar 802.11 define estas características específicas del SSID, que actúa como identificador único de la WLAN.

---

**P2.** Un analista de seguridad necesita identificar el AP al que está conectado un dispositivo cliente. El analista observa el campo BSSID en los paquetes capturados. ¿Qué valor representa el BSSID?

A) La dirección IP del Access Point en la red local
B) El identificador de 32 caracteres de la WLAN al que pertenece el AP
C) La dirección MAC del Access Point o base station que configuró el BSS
D) El nombre del fabricante del Access Point codificado en 48 bits

**Respuesta correcta: C**
El BSSID es la dirección MAC del AP o base station que ha configurado el BSS (Basic Service Set). No es la IP del AP (opción A) ni el SSID (opción B). El usuario normalmente no es consciente del BSS al que pertenece, a diferencia del SSID que es visible.

---

**P3.** Una organización pequeña quiere implementar autenticación wireless con credenciales individuales por usuario para sus 200 empleados, de forma que si un empleado es despedido se pueda revocar su acceso sin cambiar la contraseña de todos los demás. ¿Qué modo de autenticación wireless satisface este requisito?

A) WPA2-Personal con una passphrase compleja única para toda la organización
B) WPA2-Enterprise con servidor RADIUS que gestiona credenciales individuales por usuario
C) WPA3-Personal con SAE/Dragonfly que genera claves únicas por sesión
D) WPA2-Personal con MAC address filtering para controlar el acceso por dispositivo

**Respuesta correcta: B**
WPA2-Enterprise (802.1X mode) usa un servidor RADIUS que gestiona credenciales individuales por usuario. Esto permite revocar el acceso de un usuario específico sin afectar a los demás. WPA2-Personal (PSK) usa una contraseña compartida que todos conocen, por lo que revocar el acceso de uno implica cambiarla para todos.

---

**P4.** Un ingeniero de redes está desplegando una red wireless para dispositivos IoT de sensores que requieren mayor alcance que 2.4 GHz o 5 GHz, bajo consumo energético y arquitectura mesh. ¿Qué estándar wireless es el más adecuado?

A) 802.11ax (Wi-Fi 6) en la banda de 2.4 GHz para mayor penetración de señal
B) 802.11ah (Wi-Fi HaLow) en la banda de 900 MHz para IoT con mayor alcance
C) 802.15.1 (Bluetooth) por su bajo consumo energético en distancias cortas
D) 802.11n con antenas MIMO para mejor penetración de obstáculos

**Respuesta correcta: B**
802.11ah (Wi-Fi HaLow) trabaja en la banda de 900 MHz y está específicamente diseñado para IoT, ofreciendo mayor alcance y tasas de datos más altas que estándares anteriores. 802.15.4 (ZigBee) también es opción IoT con mesh pero con velocidad de solo 250 Kbps. Wi-Fi HaLow es la respuesta específica del temario CEH para IoT con mayor alcance.

---

**P5.** Una red wireless corporativa en un entorno de oficinas densamente poblado experimenta interferencias y colisiones porque muchos dispositivos compiten por el mismo canal. ¿Qué estándar wireless y qué característica técnica específica aborda mejor el problema de múltiples usuarios simultáneos en entornos densos?

A) 802.11n con MIMO-OFDM para usar múltiples antenas simultáneamente
B) 802.11ax (Wi-Fi 6) con OFDMA y BSS Coloring para gestión eficiente de múltiples conexiones
C) 802.11ac (Wi-Fi 5) con mayor ancho de banda en la banda de 5 GHz sin interferencias
D) 802.11g en la banda de 2.4 GHz con mayor compatibilidad con dispositivos existentes

**Respuesta correcta: B**
802.11ax (Wi-Fi 6) usa OFDMA (Orthogonal Frequency-Division Multiple Access) que permite que múltiples usuarios compartan simultáneamente los sub-canales, más BSS Coloring para reducir interferencias entre redes. Es el estándar específicamente diseñado para entornos densos. OFDM (sin Multiple Access) de versiones anteriores no tiene esta eficiencia multi-usuario.

---

**P6.** Un penetration tester necesita realizar ataques de largo alcance contra una WLAN objetivo situada a aproximadamente 8 kilómetros de distancia. ¿Qué tipo de antena debería usar y por qué está específicamente asociada con ataques DoS de capa 1 y MITM?

A) Antena Yagi (Yagi-Uda) por su alta ganancia y patrón end-fire unidireccional
B) Antena Omnidireccional por su patrón de radiación de 360° que cubre el área objetivo
C) Antena Parabolic Grid por su alcance de aproximadamente 10 millas y señales muy concentradas
D) Antena Dipole por su diseño balanceado y bilateralmente simétrico de media longitud de onda

**Respuesta correcta: C**
La antena Parabolic Grid tiene alcance de aproximadamente 10 millas (~16 km) con señales RF muy concentradas y polarizadas. El temario CEH la asocia específicamente con ataques Layer-1 DoS y MITM a larga distancia. La antena Yagi también es unidireccional y de largo alcance pero la Parabolic Grid es la mencionada específicamente para estos ataques en el material CEH.

---

**P7.** Un técnico compara las técnicas de spread spectrum DSSS y FHSS para implementar en una nueva red wireless. ¿Cuál es la diferencia fundamental entre ambas?

A) DSSS cambia la frecuencia portadora continuamente; FHSS multiplica la señal con un código pseudoaleatorio
B) FHSS multiplica la señal con código pseudoaleatorio; DSSS cambia entre frecuencias con secuencia conocida
C) DSSS multiplica la señal con código pseudoaleatorio; FHSS salta entre frecuencias usando secuencia pseudoaleatoria conocida por sender y receiver
D) DSSS usa múltiples frecuencias simultáneamente; FHSS usa una sola frecuencia que cambia periódicamente con sincronización automática

**Respuesta correcta: C**
DSSS (Direct-Sequence Spread Spectrum) multiplica la señal de datos con un código pseudoaleatorio de ruido, expandiendo el espectro. FHSS (Frequency-Hopping Spread Spectrum) cambia rápidamente la portadora entre muchas frecuencias usando una secuencia pseudoaleatoria conocida por ambos extremos de la comunicación (sender y receiver). Esta sincronización es clave para FHSS.

---

**P8.** Un administrador quiere que empleados en movilidad puedan moverse por las instalaciones sin perder la conexión wireless. Los APs tienen áreas de cobertura superpuestas. ¿Qué tipo de red wireless describe este escenario?

A) Extension to Wired Network con un único AP central de alta potencia
B) LAN-to-LAN Wireless donde los APs de distintas áreas intercomunican sus clientes
C) Multiple Access Points con áreas superpuestas para permitir roaming sin interrupciones
D) 3G/4G/5G Hotspot que proporciona cobertura móvil a dispositivos Wi-Fi

**Respuesta correcta: C**
El tipo "Multiple Access Points" describe exactamente este escenario: múltiples APs con áreas superpuestas que permiten que los dispositivos cambien de AP automáticamente (roaming) manteniendo la conectividad. Los extension points pueden actuar como relays adicionales en esta configuración.

---

**P9.** ¿Qué estándar wireless define específicamente los mecanismos de QoS para aplicaciones de voz, VoIP y video y en qué capa del modelo OSI opera?

A) 802.11i en la capa de red (Layer 3) para gestionar la prioridad de paquetes por dirección IP
B) 802.11e en la capa MAC (Layer 2) para gestionar prioridades de aplicaciones en tiempo real
C) 802.11n en la capa física (Layer 1) con MIMO para mayor throughput en aplicaciones multimedia
D) 802.11r en la capa de enlace (Layer 2) para fast roaming en entornos con VoIP

**Respuesta correcta: B**
802.11e define mecanismos de QoS específicamente en **Layer 2 (MAC layer)** para voz, VoIP y video. No hay que confundirlo con 802.11i (que define seguridad con WPA2) ni con 802.11r (fast roaming). La operación en Layer 2 (MAC) es el dato específico que diferencia 802.11e.

---

**P10.** Un estudiante de ciberseguridad observa que el SSID de una red está oculto (hidden SSID). El compañero argumenta que esto proporciona seguridad adicional efectiva. ¿Es correcto este argumento?

A) Sí, el SSID oculto impide que herramientas de scanning como Wireshark detecten la red
B) Sí, los clientes no pueden conectarse a una red con SSID oculto sin conocer previamente el SSID exacto
C) No, el SSID oculto no proporciona seguridad real porque se puede obtener en texto plano de los paquetes de la red
D) No, porque el SSID oculto solo funciona en redes WEP pero no en WPA2 o WPA3

**Respuesta correcta: C**
El SSID, aunque esté configurado como "oculto", aparece en texto plano en los paquetes de la red: probe requests, probe responses, association requests y re-association requests. Herramientas de análisis wireless lo capturan fácilmente. El SSID oculto proporciona oscuridad superficial pero no seguridad real. Es importante distinguirlo de medidas de seguridad reales como WPA3 o 802.1X.
