# M20_03 — Cryptanalysis

---

## 1. Conceptos y Definiciones

### Cryptanalysis — definición y alcance

Cryptanalysis es el estudio de cifrados, ciphertext y criptosistemas con el objetivo de identificar vulnerabilidades y extraer el plaintext sin conocer la clave o el algoritmo. No es lo mismo que atacar la implementación (side-channel) que atacar el algoritmo en sí. La distinción entre ambos tipos de ataque es relevante para el examen.

---

### 🔴 Métodos de cryptanalysis

| Método | Tipo de entrada requerida | Quién lo inventó | Aplicación principal |
|---|---|---|---|
| **Linear Cryptanalysis** | Known plaintext (pares P/C suficientes) | Mitsuru Matsui | Block ciphers (ej. DES: requiere ~2⁴³ plaintexts vs. 2⁵⁶ brute-force) |
| **Differential Cryptanalysis** | Chosen plaintext originalmente; también known P/C | Eli Biham y Adi Shamir | Algoritmos simétricos; analiza diferencias en input y su efecto en output |
| **Integral Cryptanalysis** | Chosen plaintext con bits fijos + barrido de combinaciones | Lars Knudsen | Block ciphers basados en redes sustitución-permutación; extensión de differential |
| **Quantum Cryptanalysis** | Datos cifrados + recursos cuánticos | — | RSA/ECC (Shor's algorithm); AES/SHA (Grover's algorithm) |

**Linear cryptanalysis** usa aproximaciones lineales del comportamiento del cifrador. Para cada bit de clave se construye una ecuación XOR de bits de P y C. El algoritmo de Matsui (Algorithm 2) cuenta cuántas veces se cumple la aproximación sobre todos los pares P/C; la clave parcial con mayor desviación respecto a N/2 es la más probable.

**Differential cryptanalysis** examina cómo diferencias específicas en el input se propagan a diferencias en el output. Trabaja con pares de plaintexts elegidos con una diferencia controlada.

**Integral cryptanalysis**: para tamaño de bloque b, mantiene b-k bits constantes y varía los k bits restantes en todos sus 2^k valores posibles. Con k=1 es equivalente a differential; con k>1 es una técnica propia.

**Quantum cryptanalysis** — dos algoritmos cuánticos relevantes:
- **Shor's algorithm**: factorización de números grandes en tiempo polinomial → rompe RSA, ECDH, DSA.
- **Grover's algorithm**: búsqueda cuántica → reduce fuerza efectiva de claves simétricas (AES) y funciones hash (SHA) a la mitad de sus bits.

Recursos cuánticos necesarios para cryptanalysis: Circuit Width (qubits por paso temporal), Circuit Depth (pasos temporales), Number of Gates, Number of T-Gates, T-Depth, MAXDEPTH.

---

### 🔴 Tipos de ataques criptográficos — taxonomía por acceso del atacante

| Ataque | Qué tiene el atacante | Nivel de acceso | Nota clave |
|---|---|---|---|
| **Ciphertext-only** | Solo ciphertext | Más probable, menos efectivo | Análisis de patrones; resultado habitual: break parcial |
| **Known-plaintext** | Algunos bloques P + C correspondiente + algoritmo | Pasivo | Base de linear cryptanalysis; los bloques P se deducen por lógica, no se accede al canal |
| **Chosen-plaintext** | El atacante elige el plaintext y obtiene el C resultante | Semi-activo | Alta efectividad; el atacante controla los inputs |
| **Adaptive Chosen-plaintext** | Igual que chosen-plaintext + puede modificar queries en función de respuestas anteriores | Activo | Requiere acceso al dispositivo de cifrado; más potente que chosen-plaintext estático |
| **Chosen-ciphertext** | El atacante elige ciphertexts y obtiene plaintexts | Semi-activo | Requiere acceso al canal entre emisor y receptor |
| **Adaptive Chosen-ciphertext** | Igual que chosen-ciphertext + selecciona series basándose en observaciones previas | Activo | Variante de lunchtime attack |
| **Lunchtime / Midnight Attack** | Chosen-ciphertext con acceso limitado en tiempo | Activo limitado | Subvariante de chosen-ciphertext |
| **Related-Key** | Ciphertexts de dos claves relacionadas matemáticamente | Semi-activo | Útil en entornos donde las claves se derivan de claves anteriores (ej. WEP) |
| **Dictionary** | Diccionario precomputado P↔C | Pasivo | Para descifrar claves, passwords, passphrases |
| **Rubber Hose** | Coerción/tortura de la persona | Físico/social | Extrae la clave directamente de la persona |
| **Chosen-key** | Rompe cifrado + sistema más amplio dependiente | Activo | Rompe cifrado de n bits en 2^(n/2) operaciones |
| **Timing** | Medición del tiempo de ejecución de operaciones criptográficas | Side-channel pasivo | Explota variaciones de tiempo en exponenciaciones modulares |
| **MITM (crypto)** | Intercepción + negociación de parámetros criptográficos | Activo en canal | Afecta principalmente a criptosistemas de clave pública durante el intercambio de claves |

---

### Código breaking — metodologías adicionales

**Brute Force**: prueba todas las combinaciones posibles de clave. Tiempo estimado por coste y longitud de clave:

| Coste | 40 bits | 56 bits | 64 bits | 128 bits |
|---|---|---|---|---|
| 2K$ (PC individual) | 1.4 min | 73 días | 50 años | 10²⁰ años |
| 100K$ (empresa) | 2 seg | 35 horas | 1 año | 10¹⁹ años |
| 1M$ (organización grande/estado) | 0.2 seg | 3.5 horas | 37 días | 10¹⁸ años |

Regla: por cada bit adicional de clave, el tiempo se **dobla**. Si AES-128 tarda 10²⁰ años con 2K$, AES-129 tardaría 2×10²⁰ años.

**Frequency Analysis**: estudia la frecuencia de aparición de letras o grupos en el ciphertext. Funciona porque en los idiomas naturales ciertas letras tienen frecuencias estables (ej. "e" es la letra más frecuente en inglés). Vulnerable a cifrados clásicos (sustitución, transposición). El código fuente cifrado es especialmente vulnerable porque palabras como `#define`, `struct`, `return` se repiten.

**One-Time Pad**: el único cifrado teóricamente irrompible, incluso con recursos infinitos. Condiciones: clave tan larga como el mensaje, aleatoria, de un solo uso y destruida tras usarse. Debilidad práctica: la clave debe ser tan larga como el mensaje, lo que lo hace impráctico para mensajes grandes.

---

### Birthday Attack

Basada en la **birthday paradox**: en un grupo de 23 personas hay >50% de probabilidad de que dos compartan cumpleaños. El mismo principio se aplica a colisiones de hashes.

Para garantizar una colisión en MD5 (128 bits de output) se necesitarían 2¹²⁸ intentos. Aplicando el birthday paradox, se necesitan aproximadamente **1.174 × √(2¹²⁸) ≈ 2⁶⁴** intentos — muchos órdenes de magnitud menor.

El birthday attack se usa para encontrar **colisiones en funciones hash**: encontrar dos mensajes diferentes a₁ y a₂ tal que hash(a₁) = hash(a₂). Permite falsificar firmas digitales.

---

### Meet-in-the-Middle Attack

Ataque contra algoritmos con **múltiples claves** (ej. doble-DES). Usa el principio de **space-time trade-off**: cifra desde un extremo con K1 y descifra desde el otro con K2, buscando coincidencias en el medio.

Para doble-DES: el atacante cifra el plaintext con los 2⁵⁶ valores posibles de K1 y guarda los resultados. Luego descifra el ciphertext con los 2⁵⁶ valores posibles de K2. Cuando encuentra una coincidencia, ha recuperado ambas claves. Coste máximo: 2⁵⁷ operaciones (vs. 2¹¹² que se esperaría de doble-DES teórico). También es un tipo de **birthday attack** porque explota la misma matemática del paradox.

---

### Side-Channel Attack

Ataque **físico** sobre la implementación hardware/software de un criptosistema, no sobre el algoritmo en sí. Explota información que "se filtra" del dispositivo durante la operación.

**Canales de filtración:**

| Canal | Mecanismo | Tipo de análisis |
|---|---|---|
| **Consumo de energía** | Las operaciones consumen diferente potencia según los datos procesados | SPA (Simple Power Analysis): instrucciones específicas. DPA (Differential Power Analysis): estadístico, no requiere conocer la implementación |
| **Campo electromagnético** | Las variaciones del campo EM sobre la superficie del chip correlacionan con operaciones internas | Medición de variaciones sobre la superficie |
| **Emisión de luz** | LEDs de estado y CRT emiten señales ópticas que revelan datos procesados (Kuhn; Loughry y Umphress) | Análisis de radiación óptica |
| **Temporización** | Las operaciones criptográficas pueden variar en tiempo según el input (por optimizaciones de rendimiento) | Medición de tiempos de exponenciación modular |
| **Sonido** | Emisiones acústicas de teclados y componentes (CPU, memoria) | Acoustic cryptanalysis |

**Mitigaciones clave para el examen:**
- Algoritmos de tiempo fijo (sin delays dependientes de datos).
- Añadir ruido temporal o de amplitud para reducir la SNR del atacante.
- Técnicas de **blinding**: cambiar input/output del algoritmo a estados aleatorios.
- DPA-proof protocols con rotación de claves antes de acumular suficiente leakage.
- HSMs con enclosures tamper-resistant, secure enclaves, PUFs (Physically Unclonable Functions).

---

### Hash Collision Attack

Encuentra dos mensajes distintos que producen el mismo hash. Permite **falsificar firmas digitales**: si hash(a₁) = hash(a₂), una firma válida de a₁ es también válida para a₂. SHA-1 es el objetivo habitual en este ataque. Una vez detectada una colisión, se pueden encontrar más concatenando datos a los mensajes coincidentes.

---

### Rainbow Table Attack

Usa una tabla precomputada de hashes (rainbow table) para invertir funciones hash. La técnica aplica **time-memory trade-off**: se sacrifica memoria (la tabla) para ahorrar tiempo de cómputo en el momento del ataque.

La rainbow table contiene: listas de palabras (diccionario + brute-force) + sus hash values. El atacante computa el hash del ciphertext capturado y lo busca en la tabla.

**Contramedida principal**: salting (añadir un valor aleatorio único antes de hashear, invalidando tablas precomputadas).

---

### Padding Oracle Attack (Vaudenay Attack)

Explota la validación de padding en algoritmos de bloque, principalmente en **modo CBC**. Si el servidor devuelve mensajes de error diferenciados ("Invalid Padding" vs. "Decryption Failed"), el atacante puede usar esas respuestas como un oráculo para descifrar el ciphertext byte a byte, sin conocer la clave. El servidor actúa involuntariamente como **oráculo de descifrado**.

---

### DUHK Attack

**Don't Use Hard-Coded Keys**. Afecta a implementaciones que usan el **ANSI X9.31 RNG** con una **seed key hardcodeada**. El PRNG genera claves criptográficas a partir de una semilla y el estado actual. Si la semilla está hardcodeada, un atacante que conozca el algoritmo y la semilla puede combinarlos para derivar las claves de sesión. Vector habitual: ataques MITM para observar el estado de la sesión + semilla conocida → obtención de claves de cifrado VPN/web.

---

### DROWN Attack

**Decrypting RSA with Obsolete and Weakened eNcryption**. Afecta a servidores HTTPS/TLS que soportan **SSLv2** (o comparten clave privada con un servidor que lo soporta). Es un ataque **cross-protocol Bleichenbacher padding oracle**.

Un servidor es vulnerable si:
1. Permite conexiones SSLv2 (configuración incorrecta por defecto).
2. Comparte la misma clave privada con otro servidor que permite SSLv2.

El atacante lanza **probes SSLv2 maliciosos** para descifrar conexiones TLS recientes que usen la misma clave privada. También puede forzar el uso de **RSA key exchange** (en lugar de DHE/ECDH que tienen forward secrecy). Ejecutado como parte de un ataque MITM online.

---

### 🔴 Ataques a Blockchain

| Ataque | Mecanismo | Objetivo | Requisito |
|---|---|---|---|
| **51% Attack** (Majority Attack) | Control de >50% del hash rate o staking power. Se mina en una cadena privada más larga y luego se publica | Double-spending, reversión de transacciones, DoS | >50% del poder de minado de la red |
| **Finney Attack** | Se premina un bloque con transacción propia (no publicado). Se paga al victim, luego se publica el bloque preminado que invalida el pago | Double-spending; el atacante retiene bienes + monedas | Atacante es minero; víctima acepta confirmación cero |
| **Eclipse Attack** | Se llenan las peer tables del nodo objetivo con IPs del atacante + DoS para forzar reinicio → el nodo se reconecta solo a nodos del atacante | Aislar nodo: manipular su visión del blockchain; facilitar double-spending, split del hash rate | Control de múltiples nodos maliciosos |
| **Race Attack** | Se crean dos transacciones con la misma moneda (A→víctima, B→atacante propio) y se compite por cuál se confirma primero | Double-spending sin pre-minar. Víctima acepta zero-confirmation | Latencia de red explotable; víctima acepta zero-confirmation |
| **DeFi Sandwich Attack** | (1) Compra front-running del token antes de la transacción de la víctima. (2) Víctima compra a precio inflado. (3) Atacante vende. | Manipulación de precio en DEXs/AMMs | Transacción de la víctima visible en el mempool |

---

### 🔴 Ataques de Computación Cuántica — taxonomía

| Ataque | Mecanismo | Target |
|---|---|---|
| Quantum Cryptanalysis | Shor's (factorización) + Grover's (búsqueda) | RSA, ECC, DSA, AES, SHA |
| Quantum Side-Channel | Análisis de ruido cuántico, tasas de error, timing, EM durante QKD | Implementaciones físicas de sistemas cuánticos |
| Classical-to-Quantum Transition | Explota vulnerabilidades en sistemas híbridos durante la transición | Sistemas en migración a post-quantum |
| **Harvest-Now, Decrypt-Later** | Captura y archiva datos cifrados hoy para descifrarlos cuando los ordenadores cuánticos sean suficientemente potentes | Datos con confidencialidad a largo plazo (gobierno, finanzas) |
| Quantum Trojan Horse | Embedding de dispositivos cuánticos maliciosos en el sistema | Protocolos de criptografía cuántica (QKD) |
| Quantum Supply Chain | Tampering de hardware/software cuántico en la cadena de suministro | Infraestructura cuántica |
| Quantum DoS | Flooding de sistema cuántico para degradar rendimiento | Disponibilidad de sistemas cuánticos |
| Quantum Replay | Intercepción y reenvío de señales cuánticas (ej. QKD) para engañar al receptor | Protocolos QKD |
| Quantum Bit-Flipping | Voltear estados de qubits (0→1 o 1→0) para corromper computaciones | Integridad de cálculos cuánticos |
| Fault-Injection on Quantum HW | Manipulación de condiciones ambientales (temperatura, EM) para inducir fallos | Hardware cuántico |
| Quantum Error Correction Exploitation | Patrones de error diseñados que los algoritmos de corrección no pueden manejar | Fiabilidad de computaciones cuánticas |

**Riesgos cuánticos clave para criptografía clásica:**
- Shor's rompe RSA, DSA, ECC (clave pública) → PKI y firmas digitales comprometidas.
- Grover's reduce la fuerza efectiva de AES y SHA a la mitad de sus bits.
- **Harvest-Now, Decrypt-Later**: riesgo inmediato aunque los ordenadores cuánticos aún no sean suficientemente potentes.
- SSL/TLS, VPNs y PKI son vulnerables por depender de clave pública.

---

### Brute-Force de VeraCrypt con hashcat

Extracción del hash del contenedor VeraCrypt:
```bash
dd.exe if=<contenedor> of=<fichero.tc> bs=512 count=1
```
Brute-force numérico de 4 dígitos:
```bash
hashcat.exe -a 3 -w 1 -m 13721 <fichero.tc> ?d?d?d?d
```
- `-a 3`: modo brute-force
- `-w 1`: perfil de workload (1=bajo, 4=alto)
- `-m 13721`: modo de descifrado VeraCrypt
- `?d`: máscara para dígito numérico (0-9)

Con wordlist:
```bash
hashcat.exe -w 1 -m 13721 hash.tc wordlist.txt
```

---

## 2. Exam Traps ⚠️

⚠️ **[Linear vs. Known-plaintext]** — Linear cryptanalysis **es** un ataque de known-plaintext. El examen puede presentarlos como cosas distintas. Regla: linear cryptanalysis es la técnica matemática; known-plaintext es la categoría de acceso del atacante que usa.

⚠️ **[Differential Cryptanalysis — inventor]** — Inventada por **Eli Biham y Adi Shamir** (el mismo Shamir de RSA). Linear cryptanalysis fue inventada por **Mitsuru Matsui**. Integral cryptanalysis por **Lars Knudsen**. El examen puede cruzar nombres.

⚠️ **[Ciphertext-only — efectividad]** — Es el ataque **más probable** para el atacante (solo necesita interceptar tráfico), pero el **menos efectivo** criptográficamente. El examen puede invertir esta descripción.

⚠️ **[Chosen-plaintext vs. Adaptive Chosen-plaintext]** — En chosen-plaintext, el atacante elige un conjunto fijo de plaintexts. En adaptive, puede **modificar las queries interactivamente** basándose en respuestas previas. La diferencia clave es la **interactividad** y el acceso al dispositivo de cifrado.

⚠️ **[Meet-in-the-Middle — no es MITM de red]** — Meet-in-the-middle es un ataque **criptográfico** contra cifrado múltiple (doble-DES). MITM es un ataque de red donde se intercepta la comunicación. No confundir. Ambos aparecen en el mismo módulo.

⚠️ **[Meet-in-the-Middle — coste real de doble-DES]** — Doble-DES debería proporcionar 2¹¹² seguridad, pero meet-in-the-middle lo reduce a **2⁵⁷ operaciones máximo**. Si el examen pregunta por qué doble-DES no es seguro, esta es la razón técnica.

⚠️ **[Birthday Attack — número mágico]** — 23 personas para >50% de probabilidad de colisión de cumpleaños. En hash, el número de intentos es **~√(2ⁿ) = 2^(n/2)** donde n es el tamaño del output, no 2ⁿ. Para MD5 (128 bits): ~2⁶⁴ intentos, no 2¹²⁸.

⚠️ **[Rubber Hose Attack — categoría]** — Es un ataque **físico/social** (coerción o tortura), no un ataque técnico al algoritmo. Si el examen pregunta "¿qué tipo de ataque extrae claves directamente de una persona?", la respuesta es rubber hose.

⚠️ **[Side-Channel Attack — qué ataca]** — Ataca la **implementación física** del criptosistema, no el algoritmo matemático en sí. El examen puede preguntar "¿qué tienen en común SPA, DPA y timing attacks?" → son todos side-channel attacks.

⚠️ **[DPA vs. SPA]** — SPA (Simple Power Analysis) da información sobre instrucciones específicas ejecutadas. DPA (Differential Power Analysis) es estadístico y **no requiere conocer detalles de la implementación**. DPA es más potente que SPA.

⚠️ **[Padding Oracle — modo de operación]** — El padding oracle attack se ejecuta principalmente sobre **modo CBC**. Si el examen pregunta a qué modo es más vulnerable este ataque, la respuesta es CBC.

⚠️ **[DUHK — afectación]** — DUHK afecta a implementaciones que usan **ANSI X9.31 RNG con seed hardcodeada**. No es un ataque genérico a VPNs; el requisito específico es el RNG con semilla hardcodeada.

⚠️ **[DROWN — condición de vulnerabilidad]** — El servidor es vulnerable si **permite SSLv2** O si **comparte clave privada** con otro servidor que permite SSLv2. El segundo caso es el más sutil: un servidor TLS perfectamente configurado puede ser vulnerable si comparte clave con un servidor SSLv2 antiguo.

⚠️ **[51% Attack — qué permite y qué no]** — Un 51% attack permite double-spending y reversión de transacciones, pero **no** permite crear monedas de la nada ni acceder a wallets de otros usuarios. El examen puede presentar capacidades exageradas de este ataque.

⚠️ **[Finney vs. Race Attack]** — Finney **requiere que el atacante sea minero** y premina un bloque antes del ataque. Race attack **no requiere pre-minar**; simplemente difunde dos transacciones contradictorias y espera a que una se confirme primero. Ambos son double-spending, pero el mecanismo difiere.

⚠️ **[Harvest-Now, Decrypt-Later]** — Es un riesgo **actual**, no futuro. Los atacantes ya capturan y almacenan datos cifrados hoy esperando que los ordenadores cuánticos los descifren en el futuro. El examen puede presentarlo como un riesgo teórico lejano, lo cual es incorrecto.

---

## 3. Nemotécnicos

### Inventores de métodos de cryptanalysis
**"Matsui Linea, Biham+Shamir Diferencial, Knudsen Integral"**  
- Linear → **Matsui**  
- Differential → **Biham + Shamir**  
- Integral → **Knudsen**

### Ataques ordenados por acceso del atacante (menor → mayor)
**"Cipher → Known → Chosen → Adaptive"**  
Solo C → C+P conocido → C+P elegido → C+P elegido + interactivo

### Birthday paradox — números clave
**"23 personas, 50% colisión | 2^(n/2) intentos para hash de n bits"**

### Ataques blockchain — distinción Finney vs. Race
**"Finney premina (es minero), Race compite (cualquiera)"**  
Finney = pre-mined block | Race = zero-confirmation + latencia

### Side-channel — canales de filtración
**"PELTS"** = **P**ower · **E**lectromagnético · **L**uz · **T**iming · **S**ound

### Quantum attacks — los dos algoritmos más importantes
**"Shor Factoriza (RSA/ECC), Grover Busca (AES/SHA)"**

### Meet-in-the-Middle — mecánica
**"Cifra desde la izquierda, descifra desde la derecha, choca en el medio"**  
Coste: 2⁵⁷ para doble-DES (no 2¹¹²)

### DES brute-force vs. Linear cryptanalysis
**"Brute DES: 2⁵⁶ | Linear DES: 2⁴³ known plaintexts"**  
Linear es mejor que brute-force pero aún impractical sin datos suficientes.

---

## 4. Flashcards

**Q:** ¿Qué método de cryptanalysis inventó Mitsuru Matsui y qué tipo de ataque utiliza?  
**A:** **Linear cryptanalysis**. Es un ataque de known-plaintext que usa aproximaciones lineales del comportamiento del bloque cifrador mediante ecuaciones XOR de bits de plaintext, ciphertext y clave.

---

**Q:** ¿Qué algoritmos cuánticos amenazan a RSA/ECC y a AES respectivamente, y cómo?  
**A:** **Shor's algorithm** factoriza números grandes en tiempo polinomial → rompe RSA, ECC, DSA. **Grover's algorithm** acelera búsquedas cuánticas → reduce la fuerza efectiva de AES y SHA a la mitad de sus bits (AES-128 quedaría con seguridad equivalente a ~64 bits).

---

**Q:** ¿Cuántos pares plaintext/ciphertext necesita linear cryptanalysis para atacar DES y cuántos la fuerza bruta?  
**A:** Linear cryptanalysis necesita ~**2⁴³ known plaintexts**. Brute-force necesita hasta **2⁵⁶** intentos. Linear es más eficiente pero aún impracticable sin suficientes datos.

---

**Q:** ¿Qué diferencia al ataque chosen-plaintext del adaptive chosen-plaintext?  
**A:** En chosen-plaintext el atacante elige un conjunto de plaintexts de una vez. En **adaptive chosen-plaintext** el atacante puede **modificar sus queries de forma interactiva**, eligiendo el siguiente plaintext basándose en los resultados de encriptaciones anteriores. Requiere acceso directo al dispositivo de cifrado.

---

**Q:** ¿Por qué doble-DES no proporciona 2¹¹² bits de seguridad?  
**A:** Por el **meet-in-the-middle attack**. El atacante cifra el plaintext con los 2⁵⁶ posibles valores de K1, descifra el ciphertext con los 2⁵⁶ posibles valores de K2, y busca coincidencias. Coste máximo: **2⁵⁷ operaciones**, no 2¹¹².

---

**Q:** ¿Cuántos intentos aproximados necesita un birthday attack para encontrar una colisión en MD5 (128 bits)?  
**A:** ~**2⁶⁴** intentos (√(2¹²⁸) = 2⁶⁴), no 2¹²⁸. El birthday paradox reduce el coste a la raíz cuadrada del espacio total.

---

**Q:** ¿Qué tipo de ataque extrae claves criptográficas mediante coerción física de una persona?  
**A:** **Rubber Hose Attack**. No es un ataque técnico al algoritmo; explota la presión psicológica o física sobre el custodio de la clave.

---

**Q:** ¿Qué distingue a un side-channel attack de un ataque criptográfico clásico?  
**A:** El side-channel attack ataca la **implementación física** del criptosistema (consumo de energía, timing, EM, luz, sonido), no el algoritmo matemático en sí. Puede comprometer un algoritmo matemáticamente sólido si su implementación filtra información.

---

**Q:** ¿En qué se diferencia SPA de DPA y cuál es más potente?  
**A:** **SPA** (Simple Power Analysis) proporciona información sobre instrucciones específicas en ejecución en un momento dado. **DPA** (Differential Power Analysis) usa métodos estadísticos y **no requiere conocer los detalles de la implementación**. DPA es más potente y más difícil de mitigar.

---

**Q:** ¿Sobre qué modo de cifrado de bloque se ejecuta principalmente el padding oracle attack y cómo lo explota?  
**A:** Sobre **modo CBC**. Si el servidor devuelve mensajes de error diferenciados sobre el padding (en lugar de un error genérico), el atacante puede usarlo como oráculo para descifrar el ciphertext byte a byte sin conocer la clave.

---

**Q:** ¿Qué condiciones hacen vulnerable a un servidor al ataque DROWN?  
**A:** (1) El servidor permite conexiones **SSLv2**; o (2) el servidor TLS comparte su **clave privada** con otro servidor que permite SSLv2. La segunda condición hace vulnerable a un servidor TLS bien configurado por asociación.

---

**Q:** ¿Qué es el ataque DUHK y cuál es su requisito técnico específico?  
**A:** DUHK (Don't Use Hard-Coded Keys) afecta a implementaciones que usan el **ANSI X9.31 RNG con una seed key hardcodeada**. Un atacante que conozca la semilla + el estado actual del PRNG puede derivar las claves criptográficas usadas para proteger sesiones VPN/web.

---

**Q:** ¿Qué diferencia un Finney attack de un Race attack en blockchain?  
**A:** **Finney**: el atacante debe ser **minero** y premina un bloque con una transacción propia antes de pagar a la víctima. **Race**: el atacante simplemente **difunde dos transacciones contradictorias** simultáneamente y espera que la suya llegue primero. Race no requiere ser minero ni pre-minar.

---

**Q:** En un Eclipse attack sobre blockchain, ¿qué hace el atacante en el primer paso y cuál es su objetivo final?  
**A:** Primer paso: llena las **peer tables** del nodo objetivo con sus propias IPs. Objetivo final: aislar el nodo de participantes legítimos y controlar completamente su visión del blockchain, facilitando double-spending o split del hash rate.

---

**Q:** ¿Qué es un Harvest-Now, Decrypt-Later attack y por qué es un riesgo actual?  
**A:** El atacante **captura y almacena datos cifrados hoy** con la intención de descifrarlos cuando los ordenadores cuánticos sean suficientemente potentes. Es un riesgo actual porque la captura ocurre ahora; datos con confidencialidad a largo plazo (gobierno, finanzas) ya están siendo cosechados.

---

**Q:** ¿Qué sintaxis de hashcat se usa para brute-force de un password numérico de 4 dígitos sobre un hash VeraCrypt?  
**A:** `hashcat.exe -a 3 -w 1 -m 13721 <hash.tc> ?d?d?d?d` — `-a 3` = brute-force mode; `-m 13721` = modo VeraCrypt; `?d` = dígito numérico (0-9).

---

**Q:** ¿Qué mecanismo usa el DeFi Sandwich attack y qué tipo de plataforma ataca?  
**A:** Ataca **DEXs (Decentralized Exchanges) y AMMs**. El atacante (1) coloca una orden de compra **front-running** antes de la transacción de la víctima (visible en mempool), (2) la víctima compra al precio inflado, (3) el atacante vende obteniendo beneficio. Explota el orden de ejecución de transacciones y la latencia del mempool.

---

**Q:** ¿En qué se diferencia una rainbow table attack de un diccionario attack?  
**A:** El **diccionario attack** prueba passwords candidatos y los compara con el hash en tiempo real. La **rainbow table attack** usa una tabla **precomputada** de hashes (time-memory trade-off); el ataque real es solo una búsqueda en tabla. La rainbow table es mucho más rápida en el momento del ataque pero requiere más memoria y preparación previa.

---

## 5. Confusión frecuente

### Meet-in-the-Middle Attack vs. MITM (Man-in-the-Middle)
- **Meet-in-the-Middle**: ataque **criptográfico** contra cifrado múltiple (doble/triple encriptación). Trabaja cifrando desde un extremo y descifrando desde el otro para encontrar coincidencias. No requiere presencia en el canal de comunicación.
- **MITM (Man-in-the-Middle)**: ataque de **red** donde el atacante se interpone entre dos partes en comunicación, intercepta y potencialmente modifica el tráfico.
- **Criterio**: si el contexto es "cifrado múltiple" o "doble-DES" → meet-in-the-middle criptográfico. Si el contexto es "interceptación de comunicación en red" → MITM de red.

---

### Chosen-Plaintext vs. Known-Plaintext vs. Chosen-Ciphertext
- **Known-plaintext**: el atacante tiene pares P/C pero no los eligió; los obtuvo o dedujo.
- **Chosen-plaintext**: el atacante **elige** los plaintexts y obtiene los ciphertexts resultantes.
- **Chosen-ciphertext**: el atacante **elige** ciphertexts y obtiene los plaintexts correspondientes (acceso al oráculo de descifrado).
- **Criterio**: la palabra "chosen" implica que el atacante controla la selección. "Known" implica que los datos están disponibles pero no fueron elegidos deliberadamente.

---

### Side-Channel Attack vs. Timing Attack
- **Side-channel attack**: categoría amplia de ataques físicos que incluye consumo de energía, EM, luz, sonido y temporización.
- **Timing attack**: subcategoría específica de side-channel que mide el **tiempo de ejecución** de operaciones criptográficas para inferir la clave.
- **Criterio**: timing attack es siempre un side-channel attack, pero no al revés. Si la pregunta menciona "tiempo de ejecución de exponenciación modular" → timing attack. Si menciona cualquier observable físico → side-channel genérico.

---

### Birthday Attack vs. Hash Collision Attack
- **Birthday Attack**: técnica general basada en el birthday paradox para reducir el coste de encontrar colisiones. Define el número de intentos necesarios (~2^(n/2)).
- **Hash Collision Attack**: el objetivo específico de encontrar dos mensajes con el mismo hash. El birthday attack es el método para lograrlo eficientemente.
- **Criterio**: si la pregunta habla de probabilidad/número de intentos para encontrar colisiones → birthday attack. Si habla de falsificar firmas digitales encontrando dos mensajes con mismo hash → hash collision attack.

---

### Finney Attack vs. 51% Attack vs. Race Attack (blockchain double-spending)
- **51% Attack**: requiere control de >50% del hash rate de toda la red; escala amplia; permite revertir transacciones confirmadas.
- **Finney Attack**: requiere que el atacante sea minero; premina un bloque; ataca a víctimas con zero-confirmation; escala individual.
- **Race Attack**: no requiere ser minero ni pre-minar; explota zero-confirmation y latencia de red; difunde dos transacciones contradictorias simultáneamente.
- **Criterio**: si la pregunta menciona "más del 50% del hash rate" → 51%. Si menciona "minero que premina un bloque" → Finney. Si menciona "zero-confirmation" sin pre-minar → Race.
