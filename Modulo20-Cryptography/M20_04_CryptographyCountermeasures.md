# M20_04 — Cryptography Attack Countermeasures

---

## 1. Conceptos y Definiciones

### Parámetros mínimos de seguridad recomendados 🔴

Estos umbrales son datos numéricos directos que el CEH pregunta con frecuencia:

| Tipo de algoritmo | Tamaño mínimo recomendado | Contexto |
|---|---|---|
| Simétrico (ej. AES) | **256 bits** | Aplicaciones seguras, especialmente transacciones grandes |
| Asimétrico (ej. RSA) | **2048 bits mínimo** | Aplicaciones seguras y altamente protegidas |
| Hash | **256 bits o más** | Aplicaciones seguras (SHA-256, SHA-3) |

Algoritmos débiles que **no deben usarse**: MD5, SHA-1 → sustituir por SHA-256 o SHA-3.

---

### Principios generales de defensa criptográfica

Los principios se agrupan por categoría funcional:

**Gestión de claves:**
- Acceso a claves directamente a aplicación o usuario autorizado.
- IDS para monitorizar intercambio y acceso a claves.
- Las claves no deben estar en código fuente ni binarios (anti-hardcoded keys → contramedida directa contra DUHK).
- No usar la misma clave para múltiples propósitos.
- Rotación periódica de claves para minimizar exposición.
- Límite en el número de operaciones por clave.
- Si se almacenan en disco, proteger con passphrase.
- No permitir transferencia de claves privadas para firma de certificados.

**Generación de claves y derivación:**
- Usar KDF (Key Derivation Function) para derivar claves: cada clave cifrada debe crearse mediante una KDF → evita relaciones simples entre claves (contramedida para related-key attacks).
- Usar hardware-based RNG o recopilar entropía de fuentes diversas para generar claves, nonces e IVs.
- **Key stretching + salting**: incrementa el coste computacional de brute-force sobre claves derivadas.
- Funciones de derivación recomendadas: **PBKDF2**, **bcrypt**, **Argon2**.

**Integridad y autenticación:**
- Implementar message authentication para cifrado de protocolos de clave simétrica.
- Usar firmas digitales para mensajes importantes; verificar antes de procesar.
- Esquemas que combinen confidencialidad e integridad: **GCM (Galois/Counter Mode)** o **Encrypt-then-MAC**.

**Infraestructura:**
- HSM (Hardware Security Module) para almacenamiento y operaciones con claves.
- Usar solo herramientas recomendadas; nunca algoritmos criptográficos "caseros".
- Cifrado probabilístico (sin outputs predecibles para inputs elegidos) → contramedida para chosen-plaintext.
- Sistemas redundantes: cifrar datos múltiples veces.
- Actualizar a los últimos estándares de seguridad.

**Post-quantum:**
- Adoptar algoritmos **quantum-resistant**: lattice-based cryptography, hash-based signatures, code-based cryptography.
- Preparar transición a post-quantum: evaluar y probar candidatos recomendados por NIST.
- Diseño modular de sistemas criptográficos para permitir actualización rápida de algoritmos.
- Usar **zero-knowledge proofs** (ej. zk-SNARKs) para autenticación e integridad sin exponer datos sensibles.

---

### 🔴 Defensa contra ataques a Blockchain

Las contramedidas se asocian directamente con los ataques del Chunk 3:

| Ataque que mitiga | Contramedida |
|---|---|
| **51% / double-spending** | Esperar múltiples confirmaciones antes de aceptar transacciones; monitorización en tiempo real; ML para detectar patrones anómalos; combinar PoW + PoS |
| **Race Attack** | Aumentar velocidad de propagación de transacciones para minimizar ventana de ataque; batch processing y fair sequencing |
| **Finney Attack** | Esperar múltiples confirmaciones; no aceptar zero-confirmation transactions |
| **Eclipse Attack** | Algoritmos de selección de peers aleatorizados; timeouts para conexiones de peers (reconexión periódica); canales de comunicación secundarios de confianza; nodos de bootstrapping de confianza; sistemas de reputación de peers |
| **DeFi Sandwich / Front-running** | Ocultar detalles de transacciones pendientes; randomizar tiempos de envío de transacciones; algoritmos de emparejamiento de órdenes seguros |
| **General blockchain security** | Multisig wallets; HSM para almacenamiento de claves; ZKP para verificar transacciones; DIDs; auditorías de smart contracts; verificación formal |

Contramedidas adicionales relevantes:
- No almacenar claves blockchain en ficheros no seguros (Word, Notepad, notas adhesivas).
- Usar verificación **out-of-band** para validar datos blockchain desde fuentes confiables.
- Atomic swaps para trading cross-chain (reduce riesgo de transacciones incompletas).

---

### 🔴 Defensa contra ataques cuánticos

| Categoría | Contramedida |
|---|---|
| **Algoritmos resistentes** | Lattice-based, hash-based, code-based cryptography |
| **Claves simétricas** | Usar claves más largas (el efecto de Grover's se contrarresta duplicando el tamaño de clave) |
| **Rotación** | Cambiar claves regularmente para limitar el tiempo de exposición |
| **Side-channel** | Protección contra power analysis y emisiones EM (relevante para quantum side-channel) |
| **Transición** | Combinar algoritmos clásicos con quantum-resistant durante el período de transición; diseño modular |
| **Infraestructura** | HSMs con firmware quantum-safe; TPMs que soporten algoritmos quantum-resistant; MFA quantum-enhanced |
| **CI/CD** | Integrar quantum-resistance checks en el SDLC y pipelines CI/CD |
| **Datos almacenados** | Cifrar con algoritmos quantum-resistant para proteger frente a Harvest-Now/Decrypt-Later |
| **Fragmentación** | Dividir datos en fragmentos distribuidos en múltiples ubicaciones (incluso si algunos se comprometen, el original no se puede reconstruir) |
| **Comunicación** | VPNs con cifrado quantum-resistant; certificados digitales basados en algoritmos quantum-resistant |
| **Blockchain cuántico** | Firmas digitales quantum-resistant en protocolos blockchain; DLT quantum-resistant |

---

### Key Stretching

Key stretching convierte una clave débil (ej. password de usuario) en una clave más resistente a brute-force. El proceso: la clave inicial se pasa a un algoritmo que genera una clave mejorada más larga y compleja. El atacante necesita iterar sobre todas las combinaciones posibles de la clave mejorada, no solo de la original.

**Funciones de key stretching:**

| Función | Base algorítmica | Características |
|---|---|---|
| **PBKDF2** | HMAC (hash o HMAC) aplicado junto con Salt | Parte de PKCS #5 v2.01; número configurable de iteraciones |
| **bcrypt** | Variante del algoritmo **Blowfish** convertida en función hash | Diseñada específicamente para passwords; añade Salt automáticamente |
| **Argon2** | Memory-hard function | Ganador del Password Hashing Competition; resistente a ataques de GPU/ASIC |

**Salting**: añadir un valor aleatorio único a cada password **antes** de hashear. Invalida las rainbow tables (precomputadas sin sal) porque cada password tiene un hash único incluso si el password es idéntico en múltiples usuarios.

Combinación habitual en la práctica: **salting + key stretching (PBKDF2/bcrypt/Argon2)**.

---

## 2. Exam Traps ⚠️

⚠️ **[Tamaño mínimo de clave asimétrica]** — La recomendación mínima para asimétrico es **2048 bits**, no 1024. El examen puede presentar 1024 bits como "suficiente para aplicaciones seguras", lo cual es incorrecto según las recomendaciones actuales.

⚠️ **[Tamaño mínimo de clave simétrica]** — La recomendación es **256 bits** para sistemas seguros, especialmente en transacciones grandes. No 128 bits (que se considera suficiente en algunos contextos pero no es el mínimo recomendado por el libro para "seguro").

⚠️ **[Claves en código fuente]** — Las claves **nunca** deben estar en código fuente ni en binarios. Esto es la contramedida directa para DUHK (hardcoded keys). El examen puede presentar "ofuscar la clave en el código" como solución válida; no lo es.

⚠️ **[bcrypt — base algorítmica]** — bcrypt está basado en **Blowfish** (convertido en función hash), no en AES ni SHA. Si el examen pregunta en qué cifrador se basa bcrypt, la respuesta es Blowfish.

⚠️ **[PBKDF2 — estándar]** — PBKDF2 forma parte de **PKCS #5 v2.01**. El examen puede preguntar a qué estándar pertenece.

⚠️ **[Salting vs. Key Stretching — propósito]** — El **salting** previene ataques con rainbow tables (diccionarios precomputados). El **key stretching** incrementa el coste computacional de brute-force. Son complementarios, no equivalentes. El examen puede confundirlos presentando salting como defensa contra brute-force o key stretching como defensa contra rainbow tables.

⚠️ **[GCM — propósito]** — GCM (Galois/Counter Mode) es un modo de cifrado autenticado que proporciona **confidencialidad + integridad** simultáneamente. No es solo cifrado; es AE (Authenticated Encryption). Si el examen pregunta qué modo de cifrado ofrece ambas propiedades en un solo esquema, GCM es la respuesta (junto con Encrypt-then-MAC).

⚠️ **[KDF — contramedida para related-key attacks]** — Usar KDF (Key Derivation Function) para derivar cada clave evita relaciones matemáticas simples entre claves, mitigando los related-key attacks. El examen puede preguntar la contramedida específica para este tipo de ataque.

⚠️ **[Zero-knowledge proofs — contexto]** — ZKPs (ej. zk-SNARKs) permiten autenticación e integridad **sin revelar información sensible**. Aparecen tanto como contramedida para ataques criptográficos como para ataques a blockchain. Si el examen pregunta "¿qué mecanismo permite verificar identidad sin exponer datos?", la respuesta es ZKP.

⚠️ **[Post-quantum transition — diseño modular]** — El diseño modular de sistemas criptográficos permite reemplazar algoritmos rápidamente durante la transición a post-quantum. Si el examen pregunta cómo preparar sistemas para la era cuántica sin reescribir toda la arquitectura, la respuesta es diseño modular + soporte para múltiples algoritmos.

---

## 3. Nemotécnicos

### Tamaños mínimos — regla de los 3 umbrales
**"Simétrico 256, Asimétrico 2048, Hash 256"**  
Simétrico = Hash = 256 bits | Asimétrico = 2048 bits mínimo

### Key stretching functions — "PAB"
**P**BKDF2 (PKCS #5, HMAC+Salt) | **A**rgon2 (Memory-hard, ganador PHC) | **B**crypt (Blowfish-based)

### Salting vs. Key Stretching — propósito diferenciado
**"Salt mata Rainbow Tables; Stretch mata Brute-Force"**

### bcrypt — base
**"bcrypt = Blowfish hashed"** → Blowfish el cifrador de bloque, convertido en función hash.

### Contramedidas blockchain para double-spending
**"Espera Confirmaciones, Monitoriza Patrones, Randomiza Peers"**  
→ Múltiples confirmaciones + ML/tiempo real + selección aleatoria de peers + timeouts

### Quantum resistance — tipos de criptografía
**"Lattice, Hash, Code"** = Los tres tipos de criptografía post-quantum mencionados explícitamente en el libro.

---

## 4. Flashcards

**Q:** ¿Cuál es el tamaño mínimo de clave recomendado para algoritmos simétricos en sistemas seguros, especialmente para transacciones grandes?  
**A:** **256 bits** (ej. AES-256).

---

**Q:** ¿Cuál es el tamaño mínimo de clave recomendado para algoritmos asimétricos en aplicaciones altamente protegidas?  
**A:** **2048 bits mínimo** (ej. RSA-2048). 1024 bits ya no se considera suficiente.

---

**Q:** ¿Cuál es el tamaño mínimo de hash recomendado para aplicaciones seguras?  
**A:** **256 bits o más** → SHA-256 o SHA-3. MD5 y SHA-1 no deben usarse.

---

**Q:** ¿Qué es key stretching y contra qué tipo de ataque defiende principalmente?  
**A:** Key stretching convierte una clave débil (ej. password de usuario) en una clave más larga y resistente usando un algoritmo iterativo. Defiende principalmente contra **ataques de brute-force** al incrementar el coste computacional de probar cada candidato.

---

**Q:** ¿En qué algoritmo de cifrado está basado bcrypt?  
**A:** En **Blowfish**, convertido en una función hash. Añade salt automáticamente y está diseñado específicamente para hashing de passwords.

---

**Q:** ¿A qué estándar pertenece PBKDF2 y cómo funciona a nivel básico?  
**A:** Pertenece a **PKCS #5 v2.01**. Aplica una función hash o HMAC repetidamente junto con un salt al password/passphrase para producir una clave derivada.

---

**Q:** ¿Cuál es la diferencia entre salting y key stretching, y contra qué ataque defiende cada uno?  
**A:** **Salting**: añade un valor aleatorio único al password antes de hashear → invalida **rainbow tables** (tablas precomputadas sin sal). **Key stretching**: aumenta el coste iterativo del proceso de derivación → defiende contra **brute-force**. Son complementarios.

---

**Q:** ¿Qué modo de cifrado proporciona confidencialidad e integridad simultáneamente y es recomendado como alternativa a cifrado + MAC separados?  
**A:** **GCM (Galois/Counter Mode)**. También es válido Encrypt-then-MAC. GCM es un modo de AE (Authenticated Encryption) que integra ambas propiedades en un único esquema.

---

**Q:** ¿Cuál es la contramedida específica para prevenir ataques de tipo related-key?  
**A:** Usar **KDF (Key Derivation Function)** para derivar cada clave — diseñar aplicaciones y protocolos que eviten relaciones matemáticas simples entre claves cifradas. También usar **strong key schedules**.

---

**Q:** ¿Qué contramedida de blockchain mitiga específicamente los Eclipse attacks?  
**A:** Algoritmos de **selección aleatoria de peers** (randomized peer selection); **timeouts para conexiones de peers** que fuerzan reconexión periódica; canales de comunicación secundarios de confianza; nodos de **bootstrapping de confianza**; sistemas de **reputación de peers**.

---

**Q:** ¿Qué mecanismo permite autenticar identidad o verificar integridad sin revelar información sensible, y es recomendado tanto para criptografía general como para blockchain?  
**A:** **Zero-Knowledge Proofs (ZKP)**, por ejemplo zk-SNARKs. Permiten demostrar conocimiento de un secreto sin revelarlo.

---

**Q:** ¿Cómo se contrarresta el efecto de Grover's algorithm sobre claves simétricas?  
**A:** Usando claves simétricas **más largas**. Grover's reduce la seguridad efectiva a la mitad de los bits (AES-128 → ~64 bits de seguridad efectiva contra quantum). AES-256 quedaría con ~128 bits de seguridad efectiva, que sigue siendo aceptable.

---

**Q:** ¿Qué propiedad del diseño de sistemas criptográficos permite reemplazar algoritmos rápidamente durante la transición post-quantum?  
**A:** **Diseño modular** (cryptographic agility). Permite actualizar o reemplazar algoritmos criptográficos con mínima disrupción, sin reescribir toda la arquitectura del sistema.

---

**Q:** ¿Por qué no se debe usar una sola clave criptográfica para múltiples propósitos?  
**A:** Comprometer esa clave en un contexto expone todos los usos. Además, el uso múltiple puede crear relaciones matemáticas entre distintos ciphertexts que faciliten criptoanálisis. Cada clave debe tener un propósito único y ser derivada mediante KDF.

---

**Q:** ¿Qué tres tipos de criptografía post-quantum (quantum-resistant) menciona explícitamente el libro?  
**A:** **Lattice-based cryptography**, **hash-based cryptography** y **code-based cryptography**.

---

**Q:** ¿Qué contramedida específica mitiga los ataques DeFi Sandwich (front-running) en blockchain?  
**A:** Ocultar detalles de transacciones pendientes (no visibles en mempool); randomizar tiempos de envío de transacciones; usar batch processing y **fair sequencing**; algoritmos de emparejamiento de órdenes seguros.

---

## 5. Confusión frecuente

### Key Stretching vs. Salting
- **Key stretching (PBKDF2/bcrypt/Argon2)**: aumenta el coste computacional de cada intento de derivación. Defiende contra brute-force. Se aplica al proceso de derivación de clave.
- **Salting**: añade un valor aleatorio único antes de hashear. Defiende contra rainbow tables (tablas precomputadas). Sin sal, dos usuarios con el mismo password tienen el mismo hash.
- **Criterio**: si la pregunta menciona "rainbow table", "tabla precomputada" o "diccionario" → salting. Si menciona "brute-force", "coste computacional" o "iteraciones" → key stretching.

---

### PBKDF2 vs. bcrypt vs. Argon2
- **PBKDF2**: basado en HMAC + salt, configurable en iteraciones. Estándar PKCS #5 v2.01. Más antiguo; puede paralelizarse (vulnerable a ataques GPU).
- **bcrypt**: basado en **Blowfish**. Diseñado para passwords. Resistente pero limitado en memoria requerida.
- **Argon2**: memory-hard; ganador del Password Hashing Competition (PHC). Más moderno y resistente a ataques GPU/ASIC por requerir gran cantidad de memoria.
- **Criterio**: si la pregunta menciona "PKCS #5" → PBKDF2. Si menciona "Blowfish" → bcrypt. Si menciona "memory-hard" o "PHC" → Argon2.

---

### GCM vs. Encrypt-then-MAC
- **GCM (Galois/Counter Mode)**: modo de cifrado autenticado integrado en un solo esquema (AES-GCM). Proporciona confidencialidad + integridad + autenticación en una sola pasada. Muy eficiente.
- **Encrypt-then-MAC (EtM)**: enfoque de dos pasos: primero cifrar, luego calcular MAC sobre el ciphertext. Más flexible (cualquier combinación de cifrador + MAC). Igual de seguro teóricamente.
- **Criterio**: si la pregunta pide un "modo de cifrado autenticado" de AES → GCM. Si pide un "esquema de AE" general → ambos son válidos, pero EtM es el esquema de mayor seguridad de los tres AE del libro (EtM > E&M > MtE).

---

### Contramedidas para ataques blockchain — Eclipse vs. 51% vs. Race
- **Eclipse**: la contramedida es a nivel de **conectividad de red** (peers aleatorios, timeouts, reputación, bootstrapping).
- **51%**: la contramedida es a nivel de **consenso y validación** (múltiples confirmaciones, monitorización de hash rate, combinar PoW+PoS).
- **Race**: la contramedida es a nivel de **velocidad de propagación** (propagar transacciones más rápido, no aceptar zero-confirmation, batch processing).
- **Criterio**: si la pregunta habla de "nodo aislado" o "peer tables" → Eclipse. Si habla de "mayoría de hash rate" o "double-spending a escala de red" → 51%. Si habla de "zero-confirmation" o "dos transacciones simultáneas" → Race.

---

### Post-Quantum vs. Quantum Cryptography (recordatorio de M20_01)
- **Quantum cryptography**: usa física cuántica (fotones, QKD) para distribuir claves. Requiere hardware cuántico.
- **Post-quantum / quantum-resistant cryptography**: algoritmos matemáticos clásicos (lattice, hash-based, code-based) diseñados para resistir ataques de ordenadores cuánticos. No requiere hardware cuántico.
- **Criterio**: si la pregunta menciona "fotones", "QKD" o "física cuántica" → quantum cryptography. Si menciona "lattice", "resistente a Shor/Grover" o "sin hardware cuántico" → post-quantum cryptography.
