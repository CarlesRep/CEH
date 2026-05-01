# M20_01 — Cryptography Concepts and Encryption Algorithms

---

## 1. Conceptos y Definiciones

### Objetivos de la criptografía (CIA + N)

Los cuatro pilares son **Confidencialidad, Integridad, Autenticación y No repudio**. El no repudio es el único objetivo que garantiza que ni emisor ni receptor puedan negar haber enviado o recibido un mensaje; esto lo hacen las **firmas digitales**, no el cifrado simétrico. La confidencialidad e integridad pueden conseguirse con cifrado simétrico, pero la autenticación y el no repudio requieren asimetría.

---

### Criptografía simétrica vs. asimétrica

La criptografía simétrica (clave secreta) usa **una sola clave** para cifrar y descifrar. Su ventaja principal es la velocidad y el bajo coste computacional; su debilidad crítica es el **problema de distribución de claves**: antes de comunicarse, ambas partes deben intercambiar la clave de forma segura, lo que es inviable a escala en Internet.

La criptografía asimétrica (clave pública) resuelve ese problema mediante un par de claves: la **clave pública** (distribuible libremente) y la **clave privada** (secreta). El flujo de cifrado es:
1. El emisor localiza la clave pública del receptor.
2. Cifra el mensaje con esa clave pública.
3. Solo el receptor puede descifrarlo con su clave privada.

Para firmas digitales el flujo se invierte: el emisor **firma con su clave privada**; cualquiera puede verificar la firma con la clave pública del emisor.

| Dimensión | Simétrica | Asimétrica |
|---|---|---|
| Velocidad | Rápida, bajo coste CPU | Lenta, alto coste CPU |
| Distribución de clave | Problema crítico (canal seguro requerido) | No requiere transmitir clave privada |
| No repudio | ❌ No proporciona | ✅ Proporciona (firmas digitales) |
| Vulnerabilidades | Diccionario, fuerza bruta | MITM, fuerza bruta |
| Si se compromete la clave | Solo afecta a esa comunicación | Compromete todos los mensajes cifrados con esa clave pública |

---

### 🔴 Algoritmos simétricos — datos clave de examen

| Algoritmo | Tipo | Tamaño de clave | Tamaño de bloque | Nota clave |
|---|---|---|---|---|
| DES | Bloque | **56 bits** (clave efectiva; 64 bits totales con 8 de paridad) | 64 bits | Obsoleto |
| 3DES | Bloque | **112 o 168 bits** | 64 bits | Encrypt-K1, Decrypt-K2, Encrypt-K3 |
| AES | Bloque | **128, 192, 256 bits** | **128 bits** | Estándar NIST actual |
| RC4 | Stream | 40–2048 bits (variable) | — | Usado en WEP/WPA (obsoleto en seguridad Wi-Fi) |
| RC5 | Bloque | 0–2040 bits | 32, 64, 128 bits | Rondas: 0–255 |
| RC6 | Bloque | 128, 192, 256 bits | 128 bits | Finalista AES; 4 registros de 4 bits (vs. 2×2 en RC5) |
| Blowfish | Bloque | 32–448 bits | 64 bits | 16 rondas Feistel; sustituto de DES |
| Twofish | Bloque | Hasta 256 bits | 128 bits | Finalista AES; diseñado por Bruce Schneier |
| IDEA | Bloque | 128 bits | 64 bits | Usado en PGP |
| Threefish | Bloque | 256, 512, 1024 bits | = tamaño de clave | Parte de Skein/SHA-3 contest; operaciones ARX |
| Serpent | Bloque | 128, 192, 256 bits | 128 bits | **32 rondas**; finalista AES; más seguro que Rijndael pero más lento |
| Camellia | Bloque | 128, 192, 256 bits | 128 bits | **18 rondas** (128-bit) / **24 rondas** (256-bit); parte de TLS |
| TEA | Bloque | **128 bits** | **64 bits** | **64 rondas** (en pares/ciclos); usa δ = 2³²/φ |
| CAST-128 | Bloque | 40–128 bits | 64 bits | 12 o 16 rondas; default en PGP/GPG |
| CAST-256 | Bloque | 128–256 bits | 128 bits | Extensión de CAST-128 |
| ChaCha20 | Stream | 256 bits | — | Moderno; alternativa a RC4 |
| Salsa20 | Stream | 256 bits | — | Familia ChaCha |
| GOST (Magma) | Bloque | 256 bits | 64 bits | **32 rondas** Feistel; estándar ruso |

---

### DES y 3DES — mecanismo interno

DES usa una clave de **56 bits generados aleatoriamente + 8 bits de detección de errores** = 64 bits totales. Proporciona 72 cuatrillones de claves posibles. Su debilidad es la longitud de clave insuficiente frente a hardware moderno.

3DES opera con un **key bundle** (K1, K2, K3) usando la secuencia: **Encrypt(K1) → Decrypt(K2) → Encrypt(K3)**. Las tres opciones de configuración, de más a menos segura:
1. K1 ≠ K2 ≠ K3 (tres claves independientes) → **168 bits efectivos**
2. K1 = K3 ≠ K2 → **112 bits efectivos**
3. K1 = K2 = K3 → equivale a DES simple (menos seguro)

---

### AES — pseudocódigo y estructura

AES es un **bloque iterado** con tres variantes de número de rondas según el tamaño de clave:
- AES-128 → 10 rondas
- AES-192 → 12 rondas
- AES-256 → 14 rondas

Cada ronda (excepto la última) ejecuta: `SubBytes → ShiftRows → MixColumns → AddRoundKey`. La última ronda omite `MixColumns`. La ronda inicial solo hace `AddRoundKey`.

---

### RC4, RC5, RC6 — diferencias clave

RC4 es un **stream cipher** con operaciones orientadas a byte, basado en permutación aleatoria. Cada byte de salida usa 8–16 operaciones del sistema. Período estimado > 10¹⁰⁰.

RC5 es un bloque con tres parámetros configurables: tamaño de bloque (32/64/128 bits), número de rondas (0–255) y tamaño de clave (0–2040 bits). Sus tres rutinas: **expansión de clave, cifrado, descifrado**. Operaciones fundamentales: suma entera, XOR, rotación variable.

RC6 deriva de RC5 con dos diferencias: añade **multiplicación entera** (mayor difusión en menos rondas) y usa **cuatro registros de 4 bits** en lugar de dos de 2 bits (necesario para el bloque de 128 bits de AES).

---

### Blowfish — expansión de clave

Blowfish usa una estructura con **P-array de 18 subclaves de 32 bits** y **cuatro S-boxes de 256 entradas de 32 bits cada una**. La clave (máximo 448 bits) se divide en subgrupos de 4168 bytes. El proceso de expansión se repite **521 veces** para generar todas las subclaves. La función de ronda divide la entrada de 32 bits en cuatro partes de 8 bits que alimentan las S-boxes; las salidas se combinan con suma módulo 2³² y XOR.

---

### Threefish — características únicas

Solo usa operaciones **ARX (Addition-Rotation-XOR)** sobre palabras de 64 bits. **No usa S-boxes**, lo que evita ataques de temporización de caché (*cache timing attacks*). Número de rondas: 256 y 512 bits → 72 rondas; 1024 bits → 80 rondas. El tamaño de bloque es igual al tamaño de clave.

---

### Serpent vs. Rijndael (AES)

Serpent usa **32 rondas** con S-boxes de 8 variables (entrada y salida de 4 bits), todas ejecutadas en paralelo 32 veces. Minimiza más la correlación entre textos cifrados que Twofish o Rijndael. Sin embargo, NIST eligió Rijndael por su **velocidad de cifrado moderada** y menor complejidad. Rijndael es ahora el estándar AES.

---

### Algoritmos asimétricos — RSA

RSA basa su seguridad en la dificultad de **factorizar el producto de dos números primos grandes**. Generación de claves:
1. Se eligen dos primos p y q; se calcula n = p·q (módulo).
2. Se calcula φ = (p−1)(q−1).
3. Se elige e tal que gcd(e, φ) = 1 y 1 < e < φ. → **Clave pública: (n, e)**
4. Se calcula d tal que e·d ≡ 1 (mod φ). → **Clave privada: d**
5. Se destruyen p y q.

Cifrado: C = T^e mod n | Descifrado: T = C^d mod n

En la práctica, RSA se usa como **envoltorio (digital envelope)**: el mensaje se cifra con DES (velocidad) y la clave DES se cifra con RSA (gestión de claves). El receptor descifra primero la clave DES con su clave privada RSA, y luego el mensaje.

---

### DSA — Digital Signature Algorithm

DSA genera una **firma digital de 320 bits** con seguridad de 512–1024 bits. Es un estándar FIPS 186 propuesto por NIST.

- **Signature Generation**: usa la **clave privada**.
- **Signature Verification**: usa la **clave pública**.

DSA es un criptosistema de clave pública pero **solo sirve para firmas digitales**, no para cifrado de datos.

---

### Diffie-Hellman

Protocolo que permite establecer una clave compartida sobre un canal inseguro. Parámetros públicos: primo p y generador g. Flujo:
- Alice elige privado *a*; calcula público gᵃ mod p.
- Bob elige privado *b*; calcula público gᵇ mod p.
- Intercambian valores públicos.
- Alice calcula k = (gᵇ)ᵃ mod p; Bob calcula k = (gᵃ)ᵇ mod p → **misma clave k**.

**No proporciona autenticación** por sí solo → vulnerable a MITM. Es la base de la **forward secrecy** en los modos efímeros de TLS.

---

### ECC — Elliptic Curve Cryptography

ECC ofrece seguridad equivalente a RSA con claves mucho más pequeñas:

| ECC (bits) | RSA equivalente (bits) |
|---|---|
| 160–223 | 1024 |
| 224–255 | 2048 |
| 256–383 | 3072 |
| 384–511 | 7680 |
| 512+ | 15360 |

Clave más pequeña = cifrado más rápido. ECC está propuesto como reemplazo de RSA.

---

### 🔴 Funciones hash — tabla comparativa

| Algoritmo | Output (bits) | Block (bits) | Rondas | Notas clave |
|---|---|---|---|---|
| MD2 | 128 | 128 | 18 | Solo para máquinas de 8 bits; legacy |
| MD4 | 128 | 512 | 48 | Colisión en < 1 min en PC normal |
| MD5 | 128 | 512 | 64 | No collision-resistant; útil para integridad de ficheros |
| MD6 | 224/256/384/512 | 512 | Variable | Árbol Merkle; resistente a criptoanálisis diferencial |
| SHA-0 | 160 | 512 | 80 | Retirado por fallo sin publicar |
| SHA-1 | 160 | 512 | 80 | Obsoleto desde 2010; usado en PGP, TLS, SSH, SSL |
| SHA-2 | 224/256/384/512 | 512/1024 | 64/80 | SHA-256 usa palabras de 32 bits; SHA-512 usa 64 bits |
| SHA-3 | 224/256/384/512 | 1088/576 | Variable | **Sponge construction**; XOR de bloques de mensaje en estado inicial |
| RIPEMD-160 | 160 | 512 | 160 (5×16×2) | Función de compresión: 80 etapas repetidas 2 veces |
| WHIRLPOOL | 512 | 512 | 10 | Operaciones matriciales |
| BLAKE2 | 256/512 | 512/1024 | 10–14 | Alta velocidad |
| BLAKE3 | 256 | 512 | Variable | Máximo rendimiento |

MD5 produce un digest de **128 bits (16 bytes)**. SHA-1 produce **160 bits**. Ambos se usan en verificación de integridad de ficheros pero están criptográficamente comprometidos.

---

### HMAC

HMAC (Hash-based Message Authentication Code) combina una clave secreta con una función hash (SHA-1, MD5). Proceso en **dos etapas**:
1. Hash(clave_interna || mensaje) → hash interno.
2. Hash(clave_externa || hash_interno) → HMAC final.

Al ejecutar la función hash **dos veces**, protege contra **ataques de extensión de longitud** (*length extension attacks*). El tamaño del output depende de la función embebida (128 bits para MD5; 160 bits para SHA-1).

---

### Modos de operación de cifrado de bloque

| Modo | IV requerido | Encadenamiento | Propagación de errores | Característica diferencial |
|---|---|---|---|---|
| ECB | No | No | No | Bloques iguales → ciphertext igual. **Fallo de seguridad grave** |
| CBC | Sí | XOR con bloque anterior | Sí (al siguiente bloque) | Estándar para cifrado simétrico general |
| CFB | Sí | Ciphertext anterior como feedback | Parcial | Usa shift register; dificulta criptoanálisis |
| CTR | No (contador) | No | **No** (sin propagación) | El mismo algoritmo de cifrado para cifrar y descifrar; paralelizable |

---

### Authenticated Encryption (AE) — MAC approaches

Tres formas de combinar cifrado y MAC:

- **Encrypt-then-MAC (EtM)**: cifra primero → genera MAC sobre el ciphertext. **Mayor seguridad**.
- **Encrypt-and-MAC (E&M)**: genera MAC del plaintext → cifra el plaintext → transmite ambos.
- **MAC-then-Encrypt (MtE)**: genera MAC del plaintext → cifra (plaintext + MAC).

**AEAD** (Authenticated Encryption with Associated Data): añade datos no cifrados (cabecera) para verificar el origen; el payload sí va cifrado.

---

### GAK y Key Escrow

**GAK** (Government Access to Keys): obligación legal de revelar claves criptográficas a organismos gubernamentales. El mecanismo operativo es el **key escrow**: las claves se depositan en un tercero de confianza (normalmente el gobierno) que puede usarlas bajo autorización judicial. Riesgo: una sola clave maestra protege todas las demás; comprometer esa clave expone el árbol completo.

---

### Hardware encryption

| Dispositivo | Función principal |
|---|---|
| TPM | Chip en placa base; almacena claves; cifrado de disco completo; integridad de plataforma |
| HSM | Módulo externo; gestión/generación/almacenamiento de claves; simétrico > 256 bits; conectado por TCP/IP |
| USB Encryption | Cifrado en dispositivo USB; protección contra distribución de malware y pérdida de datos |
| Hard Drive Encryption | Requiere TPM o HSM; no usa teclado en dispositivo |

---

### Quantum Cryptography

Usa fotones con espín en lugar de matemáticas binarias. Los filtros de polarización codifican bits:
- **Horizontal (–) y forward slash (/) = 0**
- **Vertical (|) y backslash (\) = 1**

Cualquier intento de interceptar altera la polarización → el receptor detecta el error. Técnica base: **QKD (Quantum Key Distribution)**.

**Post-quantum cryptography**: algoritmos clásicos (mayormente de clave pública) diseñados para resistir ataques de computadores cuánticos. No requiere hardware cuántico.

**Homomorphic encryption**: permite operar matemáticamente sobre datos cifrados sin descifrarlos. Solo el titular de la clave puede descifrar el resultado. Útil para computación en cloud con datos sensibles.

**Lightweight cryptography**: orientada a dispositivos de bajo consumo (RFID, IoT, sensores). Objetivo: menor potencia y recursos sin sacrificar seguridad.

---

## 2. Exam Traps ⚠️

⚠️ **[DES key size]** — El examen pregunta sobre el tamaño de clave de DES. La clave es de **64 bits en total**, pero solo **56 bits son la clave efectiva**; los otros 8 se usan para detección de errores (paridad). Respuesta correcta cuando preguntan la clave "real" o "efectiva": **56 bits**.

⚠️ **[3DES key options]** — El examen puede preguntar cuál opción de 3DES es "la más segura" o cuál "equivale a DES simple". Tres claves independientes = más seguro (168 bits). K1=K2=K3 = equivale a DES (menos seguro). No confundir con 112 bits (K1=K3).

⚠️ **[AES block size]** — AES siempre usa **bloque de 128 bits** independientemente del tamaño de clave (128, 192 o 256). El examen puede presentar "256-bit block size" para AES como opción trampa.

⚠️ **[RC4 tipo]** — RC4 es un **stream cipher**, no un block cipher. El examen puede listar RC4 entre block ciphers. Verificar: si aparece "RC4 block cipher" → incorrecto.

⚠️ **[RC5 vs RC6 registros]** — RC5 usa **dos registros de 2 bits**; RC6 usa **cuatro registros de 4 bits**. RC6 añade multiplicación entera; RC5 no. Diferencia para examen: número de registros y multiplicación entera.

⚠️ **[DSA vs RSA — propósito]** — DSA sirve **solo para firmas digitales**, no para cifrado de datos. RSA sirve tanto para cifrado como para firmas. Si el escenario pide "cifrar datos + firmar", la respuesta es RSA, no DSA.

⚠️ **[Diffie-Hellman — autenticación]** — DH **no proporciona autenticación**. Es solo intercambio de claves. Es vulnerable a MITM precisamente por esto. No confundir con RSA o DSA que sí autentican.

⚠️ **[ECC key size equivalence]** — ECC 160 bits ≈ RSA 1024 bits. ECC 256 bits ≈ RSA 3072 bits. El examen puede preguntar "¿qué tamaño de clave ECC equivale a RSA 2048?" → respuesta: **224–255 bits**.

⚠️ **[SHA-1 obsolescencia]** — SHA-1 dejó de estar aprobado para uso criptográfico en **2010**. Si el examen pregunta "¿cuál hash ya no se recomienda?", SHA-1 es la respuesta habitual junto con MD5.

⚠️ **[SHA-3 construcción]** — SHA-3 usa **sponge construction**, no la misma estructura que SHA-1 o SHA-2. Es la diferencia arquitectónica crítica de SHA-3.

⚠️ **[HMAC — protección length extension]** — HMAC protege contra ataques de extensión de longitud porque ejecuta la función hash **dos veces**. Si preguntan por qué HMAC es más seguro que un simple hash con clave, esta es la razón técnica.

⚠️ **[ECB flaw]** — El fallo de ECB es que bloques de plaintext idénticos producen bloques de ciphertext idénticos, permitiendo análisis de patrones. Es el modo **menos seguro**. Si el examen pregunta cuál modo no se debe usar, la respuesta es ECB.

⚠️ **[CTR — propagación de errores]** — A diferencia de CBC, el modo CTR **no propaga errores** porque no usa ciphertext previo en el proceso de cifrado/descifrado.

⚠️ **[Encrypt-then-MAC]** — EtM es el esquema AE **más seguro** de los tres (EtM, E&M, MtE). Si el examen pregunta cuál proporciona mayor seguridad, la respuesta es EtM.

⚠️ **[Homomorphic vs. Public Key]** — En cifrado homomorfo, **cualquiera puede manipular el ciphertext** matemáticamente, pero **solo el titular de la clave puede descifrar**. Diferencia con cifrado de clave pública donde cualquiera puede cifrar (no manipular operacionalmente).

⚠️ **[CAST-128 default cipher]** — CAST-128 (CAST5) es el cifrado por defecto en **PGP y GPG**. El examen puede preguntar qué cifrador usa PGP por defecto.

⚠️ **[Threefish — no S-boxes]** — Threefish es notable por **no usar S-boxes** (para evitar cache timing attacks). Si el examen menciona un algoritmo que evita S-boxes como contramedida, la respuesta es Threefish.

---

## 3. Nemotécnicos

### Objetivos de criptografía — CIA+N
**"Con Integridad, Autenticamos y No-repudiamos"**  
→ **C**onfidencialidad · **I**ntegridad · **A**utenticación · **N**o repudio

### 3DES — secuencia de operaciones
**"EDE"** = **E**ncrypt → **D**ecrypt → **E**ncrypt (con K1, K2, K3)

### Finalistas AES (algoritmos que compitieron con Rijndael)
**"BRATS"** → **B**lowfish (no, Twofish) — en realidad los 5 finalistas fueron:  
**"MARS-RT"** = MARS, RC6, **R**ijndael (ganador), **T**wofish, **S**erpent

### Threefish — número de rondas
256 y 512 bits → **72** rondas | 1024 bits → **80** rondas  
Regla: "los dos pequeños comparten rondas (72), el grande tiene más (80)"

### Blowfish — expansión de clave
**"P18 + S4×256"** = P-array con 18 subclaves + 4 S-boxes de 256 entradas | repetir **521 veces**

### DES key: "56 real, 64 total, 8 paridad"

### SHA family — output sizes
SHA-**0/1** → **160** bits | SHA-**2** → 224/256/384/512 | SHA-**3** → igual que SHA-2 pero sponge

### Modos de cifrado — orden de seguridad (menor a mayor)
**ECB < CBC < CFB < CTR** (para resistencia a patrones y propagación de errores)

### ECC vs RSA equivalencia — ancla mínima
ECC **160** = RSA **1024** | ECC **256** = RSA **3072** | ECC **384** = RSA **7680**

### Camellia rondas
128-bit key → **18** rondas | 256-bit key → **24** rondas  
"18 con la pequeña, 24 con la grande"

---

## 4. Flashcards

**Q:** ¿Cuál es el tamaño efectivo de clave de DES y cómo se compone el total de 64 bits?  
**A:** 56 bits de clave efectiva + 8 bits de detección de errores (paridad) = 64 bits totales.

---

**Q:** En 3DES con K1=K3≠K2, ¿cuántos bits de seguridad efectivos se obtienen?  
**A:** 112 bits efectivos. Es la opción de seguridad media entre las tres configuraciones posibles.

---

**Q:** ¿Cuál es el tamaño de bloque de AES independientemente del tamaño de clave?  
**A:** 128 bits siempre. Los tamaños de clave (128/192/256 bits) no modifican el tamaño de bloque.

---

**Q:** ¿Qué diferencia técnica clave existe entre RC5 y RC6?  
**A:** RC6 añade multiplicación entera y usa cuatro registros de 4 bits en lugar de dos de 2 bits. Esto permite manejar el bloque de 128 bits requerido para AES.

---

**Q:** Un administrador necesita proteger correos electrónicos con cifrado simétrico de 128 bits en bloques de 64 bits. ¿Qué algoritmo cumple exactamente estas especificaciones?  
**A:** IDEA (International Data Encryption Algorithm): clave de 128 bits, bloque de 64 bits. También usado en PGP.

---

**Q:** ¿Qué algoritmo de cifrado simétrico no utiliza S-boxes y por qué?  
**A:** Threefish. Evita S-boxes para prevenir ataques de temporización de caché (cache timing attacks). Solo usa operaciones ARX (Addition-Rotation-XOR).

---

**Q:** ¿Para qué sirve DSA y para qué NO sirve?  
**A:** DSA sirve exclusivamente para generación y verificación de firmas digitales. No sirve para cifrado de datos. Genera firmas de 320 bits con seguridad de 512–1024 bits (FIPS 186).

---

**Q:** ¿Cuál es la vulnerabilidad principal de Diffie-Hellman?  
**A:** No proporciona autenticación, por lo que es vulnerable a ataques MITM. La clave compartida se establece correctamente, pero ninguna de las partes puede verificar la identidad de la otra.

---

**Q:** ¿Qué tamaño de clave ECC proporciona seguridad equivalente a RSA de 2048 bits?  
**A:** Entre 224 y 255 bits de ECC.

---

**Q:** ¿Qué distingue arquitectónicamente a SHA-3 del resto de la familia SHA?  
**A:** SHA-3 usa sponge construction: los bloques del mensaje se aplican XOR a los bits iniciales del estado interno, que luego se permuta de forma invertible. Las familias SHA-1 y SHA-2 usan construcción Merkle-Damgård.

---

**Q:** Un desarrollador implementa HMAC-SHA1. ¿Cuántos bits tendrá el output y por qué HMAC es más seguro que un simple keyed-hash?  
**A:** Output de 160 bits (igual que SHA-1). HMAC es más seguro porque ejecuta la función hash dos veces (hash interno + hash externo), lo que protege contra ataques de extensión de longitud.

---

**Q:** ¿Qué modo de cifrado de bloque presenta el fallo de que bloques de plaintext idénticos producen bloques de ciphertext idénticos?  
**A:** ECB (Electronic Code Book). Es el modo menos seguro y no debe usarse para datos con patrones repetitivos.

---

**Q:** ¿Qué modo de cifrado de bloque elimina la propagación de errores y por qué?  
**A:** CTR (Counter Mode). No usa ciphertext previo en el proceso de cifrado, sino un contador independiente. Un error en un bloque no afecta a los demás.

---

**Q:** De los tres esquemas AE (EtM, E&M, MtE), ¿cuál proporciona mayor seguridad y por qué?  
**A:** Encrypt-then-MAC (EtM). El MAC se calcula sobre el ciphertext, garantizando que el receptor verifique la integridad antes de descifrar, evitando ataques de ciphertext elegido sobre el proceso de descifrado.

---

**Q:** ¿Qué es el key escrow y cómo se relaciona con GAK?  
**A:** Key escrow es un mecanismo donde las claves criptográficas se depositan en un tercero de confianza. En el contexto de GAK (Government Access to Keys), ese tercero es un organismo gubernamental que puede acceder a las claves bajo autorización judicial.

---

**Q:** ¿Cuál es el cifrador por defecto de PGP/GPG?  
**A:** CAST-128 (también llamado CAST5). Es un bloque de 64 bits con clave de 40–128 bits, estructura Feistel de 12 o 16 rondas.

---

**Q:** ¿Qué algoritmo hash usa una estructura de árbol Merkle y es resistente a criptoanálisis diferencial?  
**A:** MD6.

---

**Q:** ¿Qué hardware almacena claves de cifrado directamente en la placa base del ordenador?  
**A:** TPM (Trusted Platform Module). Chip integrado en la placa base para almacenamiento seguro de claves, cifrado de disco completo y verificación de integridad de plataforma.

---

**Q:** En quantum cryptography, ¿qué polarización de fotón representa el bit "0"?  
**A:** Horizontal (–) y forward slash (/). Los bits "1" son vertical (|) y backslash (\).

---

## 5. Confusión frecuente

### Criptografía simétrica vs. asimétrica — propósito de cada clave
- **Simétrica**: misma clave para cifrar y descifrar. No hay clave pública ni privada diferenciadas.
- **Asimétrica**: clave pública cifra (o verifica firma); clave privada descifra (o genera firma).
- **Criterio de decisión**: si el escenario menciona "no repudio" o "firma digital", la respuesta es asimétrica. Si menciona "velocidad" o "cifrado masivo de datos", la respuesta es simétrica.

---

### DSA vs. RSA
- **DSA**: solo firmas digitales. No puede cifrar datos. Standard FIPS 186.
- **RSA**: cifrado + firmas digitales. Más versátil.
- **Criterio**: si la pregunta pide un algoritmo para cifrar mensajes, RSA. Si solo pide firmar, ambos son válidos, pero DSA es el estándar específico para firmas.

---

### SHA-1 vs. SHA-2 vs. SHA-3
- **SHA-1**: 160 bits, obsoleto desde 2010, estructura similar a MD5.
- **SHA-2**: familia (SHA-256/512/224/384), misma arquitectura que SHA-1 pero más robusta. SHA-256 usa palabras de 32 bits; SHA-512 usa 64 bits.
- **SHA-3**: mismo output que SHA-2 pero arquitectura completamente distinta (sponge construction). No es una mejora incremental de SHA-2; es una alternativa arquitectónica.
- **Criterio**: si la pregunta menciona "sponge construction" → SHA-3. Si menciona "diseñado por NSA" → SHA-1 o SHA-2. Si pregunta cuál está obsoleto → SHA-1.

---

### Blowfish vs. Twofish
- **Blowfish**: bloque de 64 bits, clave 32–448 bits, 16 rondas Feistel. Más antiguo.
- **Twofish**: bloque de 128 bits, clave hasta 256 bits. Finalista AES. Diseñado por Bruce Schneier (el mismo creador de Blowfish pero posterior).
- **Criterio**: tamaño de bloque diferencia → Blowfish = 64 bits; Twofish = 128 bits.

---

### ECB vs. CBC
- **ECB**: sin IV, sin encadenamiento, bloques procesados independientemente → patrón visible en ciphertext.
- **CBC**: con IV, cada bloque se XOR con el ciphertext anterior → sin patrones visibles, pero un error en un bloque se propaga al siguiente.
- **Criterio**: si la pregunta habla de "propagación de errores" → CBC. Si habla de "patrón en ciphertext" o "bloques idénticos" → ECB.

---

### Quantum Cryptography vs. Post-quantum Cryptography
- **Quantum cryptography**: usa principios físicos cuánticos (fotones, QKD) para asegurar la transmisión. Requiere hardware cuántico.
- **Post-quantum cryptography**: algoritmos matemáticos clásicos diseñados para resistir ataques de computadores cuánticos. No requiere hardware cuántico. También llamada quantum-resistant o quantum-proof.
- **Criterio**: si la pregunta habla de fotones o QKD → quantum cryptography. Si habla de algoritmos resistentes a ordenadores cuánticos sin hardware especial → post-quantum cryptography.

---

### TPM vs. HSM
- **TPM**: chip en placa base, integrado, gestiona claves del sistema local, cifrado de disco completo (BitLocker), integridad de plataforma.
- **HSM**: dispositivo externo, conectado por TCP/IP (alta gama), gestión centralizada de claves criptográficas, especialmente para simétrico > 256 bits en entornos empresariales.
- **Criterio**: si la pregunta menciona "placa base" o "disco cifrado local" → TPM. Si menciona "dispositivo externo", "red" o "gestión centralizada de claves" → HSM.
