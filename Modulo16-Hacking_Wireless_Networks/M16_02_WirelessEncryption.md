# M16_02_WirelessEncryption.md
> Módulo 16 / Subapartado 2 — Wireless Encryption

---

## 1. Conceptos y definiciones

### 🔴 Comparativa completa WEP / WPA / WPA2 / WPA3

| Atributo | WEP | WPA | WPA2 | WPA3 |
|---|---|---|---|---|
| **Algoritmo de cifrado** | RC4 | RC4 + TKIP | AES-CCMP | AES-GCMP-256 |
| **Tamaño de IV** | **24 bits** | **48 bits** | **48 bits** | Arbitrary (1 - 2⁶⁴) |
| **Longitud de clave** | 40/104 bits | **128 bits** | **128 bits** | **192 bits** |
| **Integridad** | CRC-32 | Michael MIC (64-bit) | CCMP MIC | BIP-GMAC-256 |
| **Autenticación** | PSK compartida | PSK o EAP | PSK o RADIUS/EAP | SAE (Dragonfly) |
| **Gestión de claves** | Sin mecanismo | TKIP rekeying (cada 10.000 paquetes) | Four-way handshake | SAE/Dragonfly handshake |

---

### WEP — Detalles técnicos

**Algoritmo:** RC4 (stream cipher). **Capa:** Data Link layer (enlace de datos).

**Longitudes de clave WEP:**
- 64-bit WEP → clave de **40 bits** (+ 24-bit IV = 64 bits total)
- 128-bit WEP → clave de **104 bits** (+ 24-bit IV = 128 bits total)
- 256-bit WEP → clave de **232 bits**

**Funcionamiento:**
1. CRC-32 calcula un **ICV de 32 bits** → se añade al frame de datos
2. IV de **24 bits** se añade a la WEP key → juntos forman el **WEP seed**
3. WEP seed → input del algoritmo RC4 → genera **keystream**
4. Keystream XOR (datos + ICV) → datos cifrados
5. Campo IV (IV + PAD + KID) se añade al ciphertext → **MAC frame**

**Ventajas declaradas:** confidencialidad, access control, data integrity, efficiency.

---

### 🔴 Issues con WEP — Lista de fallos

| Fallo | Descripción |
|---|---|
| **IV de 24 bits demasiado pequeño** | Se agota en pocas horas en un AP ocupado (1500-byte packets a 11 Mbps → 5 horas); enviado en **cleartext** |
| **No requiere IVs únicos** | El estándar no exige que cada paquete tenga un IV único; los vendors usan solo una pequeña parte de los 2²⁴ valores disponibles |
| **RC4 reutilizado** | RC4 diseñado como cipher de un solo uso, no para múltiples mensajes |
| **Clave estática compartida** | Todos los usuarios comparten la misma clave; cambiarla requiere reconfigurar todos los dispositivos |
| **Vulnerable a known plaintext attacks** | Con colisión de IV → reconstrucción del keystream RC4 |
| **Vulnerable a dictionary attacks** | Espacio pequeño de IV → tabla de descifrado de ~24 GB |
| **Vulnerable a FMS attack** | El método de añadir el IV al inicio de la clave hace la red vulnerable al **ataque Fluhrer–Mantin–Shamir (FMS)**; permite recuperar la clave con análisis estadístico |
| **Sin protección contra replay attacks** | No tiene mecanismo para prevenir retransmisión de paquetes capturados |
| **CRC-32 no es hash criptográfico** | Vulnerable a **bit-flipping attacks** — el atacante modifica el paquete y ajusta el checksum |
| **Autenticación solo unidireccional** | El cliente autentica el AP pero el AP no autentica al cliente |
| **Sin gestión centralizada de claves** | Dificulta cambiar claves frecuentemente |
| **Sin forward secrecy** | Capturar la PSK permite descifrar todos los paquetes pasados y futuros |
| **Clave maestra usada directamente** | Sin mecanismo built-in para actualizar claves |

**Herramientas para crackear WEP:** Fern Wifi Cracker, WEP-key-break, **aircrack-ng**, Wifi-Cracker.

---

### WPA — Detalles técnicos

**Mejora respecto a WEP:** elimina la clave estática; usa **TKIP** con claves dinámicas.

**Componentes clave:**
- **TKIP** — Temporal Key Integrity Protocol: RC4 con 128-bit keys + 64-bit MIC
- **MIC** — Message Integrity Check (usando Michael algorithm) → previene cambio o reenvío de paquetes
- **IV extendido:** 48 bits (vs. 24 bits en WEP)
- **Rekeying:** TKs se renuevan cada **10.000 paquetes**

**Four-way handshake (instalación de TKs):**
1. AP envía **ANonce** al cliente → cliente construye el **PTK** (Pairwise Transient Key)
2. Cliente envía su **SNonce** al AP + MIC
3. AP envía **GTK** (Group Temporal Key) + número de secuencia + MIC
4. Cliente confirma que las temporal keys están instaladas

**TKIP vs. WEP:**
- Usa per-packet mixing functions → clave diferente por paquete
- Implementa sequence counter → protección contra **replay attacks**
- Extended IVs (48 bits) → mayor espacio
- Mecanismo de rekeying automático

---

### 🔴 WPA2 — Detalles técnicos

**Mejoras sobre WPA:** AES reemplaza RC4; CCMP reemplaza TKIP.

**Componentes:**
- **AES-CCMP** — Counter Mode Cipher Block Chaining Message Authentication Code Protocol
- **FIPS 140-2 compliant** (NIST)
- Compatible con el estándar 802.11i

**WPA2 Personal vs. Enterprise:**
- **WPA2-Personal (PSK):** clave de **256 bits** generada de passphrase de **8-63 caracteres ASCII**; el router combina passphrase + SSID + TKIP para generar clave única por cliente
- **WPA2-Enterprise:** EAP o RADIUS para autenticación centralizada; credenciales únicas por usuario; clave cifrada oculta al usuario

**CCMP funcionamiento:** AAD (Additional Authentication Data) + Nonce (basado en PN y MAC header) → input para AES+CCMP → ciphertext + MIC cifrado; PN en CCMP header protege contra replay attacks.

---

### 🔴 Issues con WPA

| Issue | Descripción |
|---|---|
| **Weak passwords** | PSK vulnerable a password-cracking attacks |
| **Lack of forward secrecy** | Capturar PSK → descifrar todos los paquetes |
| **Packet spoofing/decryption** | Clientes WPA-TKIP vulnerables a packet-injection y decryption attacks → permite hijack de TCP connections |
| **Predictability of GTK** | RNG inseguro → atacante puede descubrir el GTK generado por el AP → inyectar tráfico malicioso + descifrar transmisiones |
| **IP address guessing** | Vulnerabilidades TKIP → adivinar IP del subnet + inyectar paquetes pequeños para degradar rendimiento |

---

### 🔴 Issues con WPA2

| Issue | Descripción |
|---|---|
| **Weak passwords** | PSK vulnerable a eavesdropping, dictionary y password-cracking attacks |
| **Lack of forward secrecy** | Capturar PSK → descifrar todos los paquetes |
| **MITM y DoS — Hole96** | Vulnerabilidad **Hole96** en WPA2: atacante explota GTK compartida → MITM y DoS |
| **Predictability of GTK** | RNG inseguro → descubrir GTK → inyectar tráfico + descifrar transmisiones |
| **KRACK** | **Key Reinstallation Attack (KRACK)** — permite sniff de paquetes, hijacking de conexiones, inyección de malware y descifrado de paquetes |
| **Replay attack DoS** | Feature de detección de replay attack de WPA2 explotable enviando GTK con PN grande → DoS |
| **Insecure WPS PIN** | Con WPA2 + WPS activos → atacante puede descubrir la clave WPA2 determinando el PIN de WPS |

---

### 🔴 WPA3 — Detalles técnicos y características

**Anunciado por Wi-Fi Alliance en enero 2018.**

**WPA3-Personal:**
- Usa **SAE (Simultaneous Authentication of Equals)**, también llamado **Dragonfly Key Exchange** — reemplaza PSK
- Resistente a: offline dictionary attacks, key recovery
- Permite contraseñas débiles/populares sin comprometer la seguridad
- **SAE/Dragonfly handshake es OBLIGATORIO** para certificación WPA3

**WPA3-Enterprise:**

| Función | Protocolo/Estándar |
|---|---|
| **Authenticated encryption** | **GCMP-256** (256-bit Galois/Counter Mode Protocol) |
| **Key derivation and validation** | **HMAC-SHA-384** (384-bit HMAC con Secure Hash Algorithm) |
| **Key establishment and verification** | **ECDH** (Elliptic Curve Diffie-Hellman) + **ECDSA** (Elliptic Curve Digital Signature Algorithm) usando **384-bit elliptic curve** |
| **Frame protection** | **BIP-GMAC-256** (256-bit Broadcast/Multicast Integrity Protocol) |

**4 características clave de WPA3:**
1. **Secured handshake** — SAE/Dragonfly resiste dictionary y brute-force attacks; previene descifrado offline
2. **Wi-Fi Easy Connect** — usa **DPP (Device Provisioning Protocol)**; conecta dispositivos IoT con código QR o password; una interfaz gestiona múltiples conexiones
3. **Unauthenticated encryption** — **OWE (Opportunistic Wireless Encryption)** reemplaza el 802.11 "open" authentication en hotspots públicos
4. **Bigger session keys** — WPA3-Enterprise soporta claves de **192 bits o más**

---

### 🔴 Issues con WPA3

| Issue | Descripción |
|---|---|
| **Implementation challenges** | Dispositivos antiguos pueden no soportar WPA3; problemas de compatibilidad |
| **Limited adoption** | Muchos dispositivos aún usan WPA2; adopción lenta |
| **Resource intensive** | Algoritmos más complejos → mayor consumo de CPU; afecta dispositivos legacy |
| **Configuration errors** | Contraseñas débiles y configuración incorrecta dejan redes vulnerables |
| **Timing attacks** | Ciertas implementaciones SAE son vulnerables a timing attacks → recuperación de contraseña |
| **Cache-based side-channel attacks** | Extracción de información de patrones de acceso a caché → revela detalles de operaciones criptográficas |
| **Transition mode weakness** | En modo de transición (WPA2+WPA3 coexistiendo), los atacantes pueden explotar el WPA2 menos seguro (**KRACK** sigue siendo posible) |
| **Hardware requirements** | Hardware actualizado requerido; dispositivos antiguos no actualizables; coste elevado |

---

### Protocolos relacionados con autenticación wireless

| Protocolo | Descripción |
|---|---|
| **EAP** | Extensible Authentication Protocol; soporta múltiples métodos: token cards, Kerberos, certificados |
| **LEAP** | Lightweight EAP; versión propietaria de Cisco |
| **PEAP** | Encapsula EAP dentro de un túnel TLS cifrado y autenticado |
| **RADIUS** | Sistema centralizado de gestión de autenticación y autorización |
| **TKIP** | Temporal Key Integrity Protocol; sustituto de WEP en WPA; usa RC4 con 128-bit keys |
| **CCMP** | Counter Mode Cipher Block Chaining MAC Protocol; usa AES; sustituto de TKIP en WPA2 |

---

## 2. Exam Traps ⚠️

⚠️ **[Longitudes de clave WEP: la clave secreta NO es el tamaño total]**
64-bit WEP = **40-bit key** (no 64-bit). 128-bit WEP = **104-bit key** (no 128-bit). El número "64" o "128" incluye los 24 bits del IV. El examen puede preguntar el tamaño de la clave secreta y esperar 40 o 104.

⚠️ **[IV de WEP: 24 bits, en cleartext, agotable en ~5 horas]**
El IV de WEP tiene exactamente **24 bits**, se envía en **cleartext** y un AP ocupado transmitiendo paquetes de 1500 bytes a 11 Mbps lo agota en **5 horas**. El examen puede cambiar el tiempo o el tamaño del IV.

⚠️ **[TKIP renueva TKs cada 10.000 paquetes]**
El rekeying de TKIP ocurre cada **10.000 paquetes**. Dato numérico específico y preguntable directamente.

⚠️ **[WPA2-Personal: passphrase de 8-63 caracteres ASCII → clave 256 bits]**
La passphrase tiene entre 8 y 63 caracteres ASCII. La clave derivada tiene 256 bits. El examen puede usar longitudes fuera de este rango como distractores.

⚠️ **[KRACK = Key Reinstallation Attack → WPA2]**
KRACK afecta específicamente a **WPA2**. Permite sniff, hijacking, inyección de malware y descifrado. El examen puede asociar KRACK con WPA3 → incorrecto (WPA3 lo resuelve).

⚠️ **[Hole96 = vulnerabilidad WPA2 específica → MITM y DoS vía GTK]**
**Hole96** es una vulnerabilidad específica de WPA2 que permite explotar el GTK compartido para MITM y DoS. El examen puede confundirla con una vulnerabilidad de WPA3 o con KRACK.

⚠️ **[WPA3: SAE también llamado Dragonfly; OBLIGATORIO para certificación WPA3]**
SAE = Simultaneous Authentication of Equals = **Dragonfly Key Exchange**. El Dragonfly handshake es **obligatorio** para la certificación WPA3. El examen puede preguntar el nombre alternativo de SAE o si es opcional.

⚠️ **[WPA3: GCMP-256 para cifrado, HMAC-SHA-384 para key derivation]**
En WPA3-Enterprise: cifrado = **GCMP-256**. Key derivation = **HMAC-SHA-384**. El examen puede intercambiar los protocolos entre las funciones.

⚠️ **[OWE reemplaza 802.11 "open" authentication en WPA3]**
**OWE (Opportunistic Wireless Encryption)** es el mecanismo de unauthenticated encryption de WPA3 para hotspots públicos. Reemplaza la autenticación "open" del 802.11. El examen puede presentar OWE como una forma de autenticación → incorrecto (es unauthenticated).

⚠️ **[WPA3 Transition Mode: sigue siendo vulnerable a KRACK via WPA2]**
El modo de transición de WPA3 (que permite coexistencia con WPA2 para compatibilidad) es vulnerable porque los atacantes pueden forzar el uso de WPA2 y explotar KRACK. El examen puede presentar el modo de transición como totalmente seguro.

---

## 3. Nemotécnicos

**Algoritmos de cifrado por protocolo** → **"WEP=RC4 · WPA=RC4+TKIP · WPA2=AES+CCMP · WPA3=AES+GCMP-256"**

**Tamaños de IV** → **"WEP=24 bits · WPA=48 bits · WPA2=48 bits · WPA3=Arbitrary"**

**Tamaños de clave** → **"WEP=40/104 · WPA=128 · WPA2=128 · WPA3=192"**

**WEP key sizes** → "64→40, 128→104, 256→232" (restar 24 para obtener la clave secreta)

**WPA3-Enterprise protocolos** → **"GCMP-256 + HMAC-SHA-384 + ECDH/ECDSA(384) + BIP-GMAC-256"**:
- **G**CMP-256 = cifrado
- **H**MAC-SHA-384 = key derivation
- **E**CDH/ECDSA = key establishment (384-bit curve)
- **B**IP-GMAC-256 = frame protection

**4 características WPA3** → **"S-E-O-B"**:
- **S**ecured handshake (SAE/Dragonfly)
- **E**asy Connect (DPP + QR code)
- **O**WE (Opportunistic Wireless Encryption)
- **B**igger session keys (192+ bits)

**Issues WEP clave** → **"IV-FMS-RC4-CRC-Replay-Shared"**:
- IV demasiado pequeño (24 bits) + enviado en cleartext
- FMS attack (IV al inicio de la clave)
- RC4 no diseñado para múltiples mensajes
- CRC-32 no es hash criptográfico → bit-flipping
- Sin protección contra replay attacks
- Clave compartida estática entre todos los usuarios

**KRACK → WPA2; Hole96 → WPA2; SAE/Dragonfly → WPA3**

---

## 4. Flashcards

**Q:** ¿Cuál es la longitud de la clave secreta en un sistema WEP de 128 bits?
**A:** 104 bits (128 bits totales − 24 bits del IV = 104 bits de clave secreta).

---

**Q:** ¿Cuántos bits tiene el IV de WEP y cuánto tiempo tarda en agotarse en un AP ocupado?
**A:** 24 bits; se agota en aproximadamente 5 horas en un AP transmitiendo paquetes de 1500 bytes a 11 Mbps.

---

**Q:** ¿Qué es el WEP seed?
**A:** La combinación del IV de 24 bits + la WEP key; se usa como input del algoritmo RC4 para generar el keystream.

---

**Q:** ¿Cada cuántos paquetes TKIP renueva las Temporal Keys?
**A:** Cada 10.000 paquetes.

---

**Q:** ¿Cuáles son los algoritmos de cifrado e integridad de WPA2?
**A:** Cifrado: AES-CCMP (Counter Mode Cipher Block Chaining MAC Protocol). WPA2 es FIPS 140-2 compliant (NIST).

---

**Q:** ¿Cuál es el rango de longitud de la passphrase en WPA2-Personal y qué tamaño de clave genera?
**A:** 8-63 caracteres ASCII; genera una clave de 256 bits.

---

**Q:** ¿Qué es KRACK y qué protocolo afecta?
**A:** Key Reinstallation Attack — afecta a WPA2. Permite sniff de paquetes, hijacking de conexiones, inyección de malware y descifrado de paquetes.

---

**Q:** ¿Qué es la vulnerabilidad Hole96 en WPA2?
**A:** Vulnerabilidad que permite explotar el GTK (Group Temporal Key) compartido para realizar ataques MITM y DoS.

---

**Q:** ¿Qué es SAE en WPA3 y cuál es su nombre alternativo?
**A:** Simultaneous Authentication of Equals; también llamado Dragonfly Key Exchange. Reemplaza el PSK de WPA2-Personal. Es obligatorio para la certificación WPA3.

---

**Q:** ¿Cuáles son los 4 protocolos de WPA3-Enterprise y sus funciones?
**A:** GCMP-256 (cifrado), HMAC-SHA-384 (key derivation y validación), ECDH/ECDSA con curva de 384 bits (establecimiento y verificación de claves), BIP-GMAC-256 (protección de frames).

---

**Q:** ¿Qué es OWE en WPA3 y a qué reemplaza?
**A:** Opportunistic Wireless Encryption — mecanismo de unauthenticated encryption de WPA3 que reemplaza el 802.11 "open" authentication en hotspots públicos.

---

**Q:** ¿Cuál es el protocolo de WPA3 que simplifica la conexión de dispositivos IoT y qué usa?
**A:** Wi-Fi Easy Connect — usa el Device Provisioning Protocol (DPP) y permite conectar dispositivos con código QR o password.

---

**Q:** ¿Por qué el modo de transición de WPA3 puede ser inseguro?
**A:** Porque permite coexistencia de WPA3 y WPA2, y los atacantes pueden forzar el uso de WPA2 (que sigue siendo vulnerable a KRACK).

---

**Q:** ¿Qué es el FMS attack en el contexto de WEP?
**A:** Fluhrer–Mantin–Shamir attack — explota que el IV se añade al inicio de la clave de seguridad, permitiendo recuperar la clave WEP mediante análisis estadístico de IVs débiles.

---

**Q:** ¿Cuál es la diferencia entre TKIP y CCMP como protocolos de cifrado?
**A:** TKIP: usa RC4 con 128-bit keys + 64-bit MIC; sustituto de WEP en WPA. CCMP: usa AES con autenticación; sustituto de TKIP en WPA2; más seguro y FIPS 140-2 compliant.

---

**Q:** ¿Cuáles son los pasos del four-way handshake para la instalación de Temporal Keys en WPA?
**A:** (1) AP envía ANonce → cliente construye PTK. (2) Cliente envía SNonce + MIC al AP. (3) AP envía GTK + número de secuencia + MIC. (4) Cliente confirma que las TKs están instaladas.

---

**Q:** ¿Por qué WEP no proporciona forward secrecy?
**A:** Porque usa una clave estática compartida sin mecanismo built-in de renovación. Si un atacante obtiene la PSK, puede descifrar todos los paquetes pasados y futuros.

---

**Q:** ¿Qué algoritmos de cifrado usa WPA?
**A:** RC4 con TKIP (Temporal Key Integrity Protocol) — 128-bit keys con 64-bit MIC usando el algoritmo Michael. Mantiene RC4 pero con gestión dinámica de claves.

---

**Q:** ¿Cuáles son las herramientas mencionadas en el libro para crackear WEP?
**A:** Fern Wifi Cracker, WEP-key-break, aircrack-ng, Wifi-Cracker.

---

**Q:** ¿Cuál es la vulnerabilidad específica de WPA relacionada con el GTK?
**A:** La predictabilidad del GTK: un RNG inseguro en WPA permite al atacante descubrir el GTK generado por el AP, pudiendo inyectar tráfico malicioso y descifrar transmisiones en progreso.

---

## 5. Confusión frecuente

**WEP 64-bit vs. 128-bit — el número incluye el IV**
- WEP 64-bit: clave secreta = **40 bits** (24 bits IV + 40 bits clave = 64 bits total).
- WEP 128-bit: clave secreta = **104 bits** (24 bits IV + 104 bits clave = 128 bits total).
- Criterio: cuando el examen pregunta "tamaño de clave WEP" → dar el tamaño del secreto (40 o 104), no el total.

---

**TKIP vs. CCMP — WPA vs. WPA2**
- TKIP: protocolo de WPA; usa RC4 mejorado; renewing cada 10.000 paquetes; sustituto de WEP; 128-bit keys + 64-bit MIC.
- CCMP: protocolo de WPA2; usa AES; FIPS 140-2 compliant; más seguro; sustituto de TKIP.
- Criterio: "WPA encryption protocol" → TKIP. "WPA2 encryption protocol" → CCMP (AES).

---

**SAE vs. PSK — WPA3 vs. WPA2**
- PSK (Pre-Shared Key): WPA/WPA2-Personal. Una contraseña estática compartida. Vulnerable a dictionary attacks offline.
- SAE (Simultaneous Authentication of Equals / Dragonfly): WPA3-Personal. Establece sesión sin revelar la contraseña. Resiste dictionary attacks offline y proporciona forward secrecy.
- Criterio: "contraseña compartida que se puede capturar y atacar offline" → PSK. "Handshake que no revela la contraseña" → SAE.

---

**KRACK vs. Hole96 — ambas vulnerabilidades de WPA2**
- KRACK (Key Reinstallation Attack): explota el four-way handshake de WPA2 para reinstalar claves ya en uso → sniff, hijacking, inyección.
- Hole96: explota el GTK compartido en WPA2 para MITM y DoS.
- Criterio: "reinstalación de claves durante handshake" → KRACK. "Explotación del GTK para MITM/DoS" → Hole96.

---

**WPA3-Personal vs. WPA3-Enterprise**
- WPA3-Personal: SAE/Dragonfly en lugar de PSK; mayor resistencia a ataques de contraseña; OWE para redes abiertas.
- WPA3-Enterprise: GCMP-256 (cifrado), HMAC-SHA-384 (key derivation), ECDH/ECDSA con curva 384 bits (key establishment), BIP-GMAC-256 (frame protection); claves de 192+ bits.
- Criterio: "reemplaza PSK con SAE" → WPA3-Personal. "GCMP-256 + HMAC-SHA-384 + ECDH/ECDSA" → WPA3-Enterprise.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un auditor de seguridad analiza una red WEP de 128 bits. Un colega afirma que la clave de seguridad de esta red tiene 128 bits de longitud. ¿Es correcto y cuál es la longitud real de la clave secreta?

A) Correcto; 128 bits es la longitud de la clave secreta en WEP-128
B) Incorrecto; la clave secreta es de 64 bits porque se divide entre los dos extremos
C) Incorrecto; la clave secreta es de 104 bits (128 bits totales − 24 bits del IV)
D) Incorrecto; la clave secreta es de 96 bits porque el IV ocupa 32 bits del total

**Respuesta correcta: C**
En WEP-128, el número 128 incluye los 24 bits del IV (Initialization Vector) más la clave secreta de 104 bits. La clave secreta real es de 104 bits. Esto es un dato específico de WEP que el examen suele explotar: 64-bit WEP → clave de 40 bits; 128-bit WEP → clave de 104 bits; 256-bit WEP → clave de 232 bits.

---

**P2.** Durante una prueba de penetración wireless, un atacante captura tráfico de una red WEP. Observa que el mismo IV se está reutilizando en múltiples paquetes y planea usar el ataque FMS. ¿Qué mecanismo específico hace a WEP vulnerable al ataque Fluhrer-Mantin-Shamir (FMS)?

A) El IV de 24 bits es demasiado pequeño y se repite frecuentemente en un AP activo
B) El método de concatenar el IV al inicio de la clave de seguridad hace la red vulnerable al análisis estadístico del FMS
C) El algoritmo RC4 es simétrico y puede ser invertido matemáticamente para recuperar la clave
D) El ICV calculado con CRC-32 puede ser manipulado mediante bit-flipping para recuperar la clave

**Respuesta correcta: B**
El ataque FMS (Fluhrer-Mantin-Shamir) explota específicamente que WEP concatena el IV al inicio de la clave de seguridad para formar el WEP seed. Esta construcción específica permite al atacante realizar análisis estadístico de IVs débiles para recuperar la clave completa. El IV pequeño (opción A) es otro problema de WEP pero no es el mecanismo específico del FMS.

---

**P3.** Un administrador de red configura WPA en lugar de WEP para mejorar la seguridad. Necesita saber cada cuántos paquetes se renuevan automáticamente las Temporal Keys (TKs) con TKIP. ¿Cuál es el valor correcto?

A) Cada 1.000 paquetes para garantizar rotación frecuente de claves
B) Cada 5.000 paquetes como compromiso entre seguridad y rendimiento
C) Cada 10.000 paquetes según la especificación TKIP de WPA
D) Cada 100.000 paquetes para minimizar el overhead de reconfiguración

**Respuesta correcta: C**
TKIP en WPA renueva las Temporal Keys (TKs) cada 10.000 paquetes. Este es un dato numérico específico del temario CEH. El rekeying automático fue una de las mejoras principales de WPA sobre WEP, que usaba una clave estática sin renovación.

---

**P4.** Un equipo de seguridad detecta que su red WPA2 ha sido comprometida mediante KRACK. ¿Cuál es el mecanismo técnico exacto que KRACK explota para vulnerar WPA2?

A) Adivina la PSK mediante ataques de diccionario offline sobre handshakes capturados
B) Explota el GTK compartido en WPA2 para realizar ataques MITM y DoS (vulnerabilidad Hole96)
C) Explota el four-way handshake de WPA2 forzando la reutilización de nonces para reinstalar claves ya en uso
D) Aprovecha el IV de 48 bits de WPA2 para realizar análisis estadístico similar al ataque FMS

**Respuesta correcta: C**
KRACK (Key Reinstallation Attack) explota el four-way handshake de WPA2 forzando la reutilización de nonces (nonce reuse). El atacante captura la ANonce ya en uso para manipular y reproducir mensajes del handshake, reinstalando claves ya usadas. Hole96 (opción B) es una vulnerabilidad diferente de WPA2 que explota el GTK. Las opciones A y D describen ataques distintos.

---

**P5.** Una empresa necesita configurar WPA2-Personal para su red de invitados. El administrador quiere usar la passphrase más larga posible según el estándar. ¿Cuáles son los límites de longitud de la passphrase y qué tamaño de clave genera?

A) 6 a 32 caracteres ASCII; genera una clave de 128 bits
B) 8 a 63 caracteres ASCII; genera una clave de 256 bits
C) 8 a 128 caracteres ASCII; genera una clave de 128 bits
D) 10 a 64 caracteres; genera una clave de 192 bits para compatibilidad con WPA3

**Respuesta correcta: B**
WPA2-Personal acepta passphrases de entre 8 y 63 caracteres ASCII. El router combina la passphrase con el SSID y TKIP para generar una clave única de 256 bits por cliente. Los valores 8-63 caracteres y 256 bits son los datos específicos preguntables del temario CEH.

---

**P6.** Un analista evalúa si una red WPA3 es vulnerable a ataques de diccionario offline. Un colega argumenta que WPA3-Personal es tan vulnerable como WPA2-Personal a este tipo de ataque porque ambos usan contraseñas. ¿Está en lo correcto?

A) Sí, ambos protoclos usan contraseñas que pueden atacarse offline capturando el handshake
B) No; WPA3-Personal usa SAE (Dragonfly), que establece la sesión sin revelar la contraseña y resiste ataques de diccionario offline
C) No; WPA3 usa cifrado AES-256 que hace inviable cualquier ataque de fuerza bruta offline
D) Sí, si la contraseña es débil WPA3 es vulnerable exactamente igual que WPA2 a ataques offline

**Respuesta correcta: B**
WPA3-Personal usa SAE (Simultaneous Authentication of Equals), también llamado Dragonfly Key Exchange, que establece la sesión sin revelar ni transmitir la contraseña. Esto hace que los ataques de diccionario offline sean inviables porque no hay un handshake capturável que contenga material criptoanalizable. WPA2-PSK sí transmite material explotable en el handshake.

---

**P7.** ¿Cuál es la función del mecanismo OWE (Opportunistic Wireless Encryption) en WPA3 y a qué tipo de red está destinado?

A) Reemplaza el PSK en redes empresariales que no pueden implementar un servidor RADIUS
B) Proporciona cifrado autenticado para redes de invitados con credenciales de usuario limitadas
C) Reemplaza el 802.11 "open" authentication para proporcionar cifrado no autenticado en hotspots públicos
D) Cifra el canal de gestión wireless para prevenir ataques de de-autenticación en redes públicas

**Respuesta correcta: C**
OWE (Opportunistic Wireless Encryption) es la característica de "unauthenticated encryption" de WPA3 que reemplaza el 802.11 "open" authentication en hotspots públicos. Aunque no autentica al usuario (no requiere contraseña), cifra el tráfico para prevenir eavesdropping. Es importante notar que es "unauthenticated" — no es una forma de autenticación.

---

**P8.** Un equipo implementa WPA3-Enterprise para la red corporativa. El administrador consulta qué protocolo debe usar para la función de key derivation y validación en WPA3-Enterprise. ¿Cuál es el protocolo correcto?

A) GCMP-256 para derivación y validación de claves criptográficas
B) HMAC-SHA-384 para derivación y validación de claves
C) ECDH con curva de 384 bits para key derivation en el handshake Dragonfly
D) BIP-GMAC-256 para key derivation como parte de la protección de frames

**Respuesta correcta: B**
En WPA3-Enterprise, HMAC-SHA-384 es el protocolo específico para key derivation y validation (derivación y validación de claves). La distribución de roles es: GCMP-256 = cifrado autenticado; HMAC-SHA-384 = key derivation/validation; ECDH/ECDSA con curva 384 bits = key establishment/verification; BIP-GMAC-256 = frame protection.

---

**P9.** Una organización ha desplegado WPA3 pero algunos dispositivos legacy solo soportan WPA2. El administrador habilita el modo de transición WPA3 para compatibilidad. ¿Qué vulnerabilidad de seguridad introduce este modo de transición?

A) El modo de transición desactiva el SAE/Dragonfly handshake para todos los dispositivos de la red
B) Los atacantes pueden forzar el uso de WPA2 (menos seguro) y explotar KRACK, que sigue siendo posible
C) El modo de transición limita el tamaño de las claves a 128 bits para compatibilidad con WPA2
D) El modo de transición deshabilita automáticamente OWE en hotspots públicos

**Respuesta correcta: B**
El modo de transición WPA3 (WPA2+WPA3 coexistiendo) introduce la vulnerabilidad de que los atacantes pueden forzar a los dispositivos a usar WPA2, que sigue siendo vulnerable a KRACK. La solución recomendada es deshabilitar el modo de transición si todos los dispositivos de la red soportan WPA3, usando WPA3 de forma exclusiva.

---

**P10.** Un atacante captura el four-way handshake de una red WPA y quiere recuperar la PSK. ¿Cuáles son las herramientas específicas mencionadas en el temario CEH para este tipo de ataque?

A) Aircrack-ng y WEPCrack para crackeado de PSK mediante análisis estadístico
B) Cowpatty y Fern Wifi Cracker para PSK cracking mediante dictionary attack sobre frames de handshake
C) Asleap y THC-LEAPcracker para crackeado de PSK en redes WPA Personal
D) Reaver y Pixie Dust para recuperar PSK mediante ataque de brute-force al WPS PIN

**Respuesta correcta: B**
Para PSK Cracking (recuperar WPA PSK de frames de handshake capturados) las herramientas específicas mencionadas en el temario CEH son Cowpatty y Fern Wifi Cracker, usando dictionary attacks. Aircrack-ng se menciona para WEP cracking, Asleap para LEAP cracking (protocolo propietario Cisco), y Reaver para ataques WPS.
