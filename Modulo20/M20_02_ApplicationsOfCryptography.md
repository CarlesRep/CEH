# M20_02 — Applications of Cryptography

---

## 1. Conceptos y Definiciones

### Public Key Infrastructure (PKI) — arquitectura y flujo

PKI es la arquitectura completa que permite usar cifrado de clave pública y firmas digitales a escala. No es solo software: incluye hardware, personas, políticas y procedimientos. Su función central es **vincular claves públicas con identidades** mediante certificados digitales firmados por una CA.

**Componentes de PKI y sus roles:**

| Componente | Función |
|---|---|
| CA (Certification Authority) | Emite y verifica certificados digitales |
| RA (Registration Authority) | Verifica la identidad del solicitante antes de que la CA emita el certificado. Actúa como verificador para la CA |
| VA (Validation Authority) | Almacena certificados con sus claves públicas; verifica la validez de un certificado cuando se consulta |
| Certificate Management System | Genera, distribuye, almacena y verifica certificados |
| Digital Certificate | Acredita la identidad del sujeto en transacciones online |
| End User | Solicita, gestiona y usa certificados |

**Flujo completo del proceso PKI:**
1. El sujeto solicita certificado a la **RA**.
2. La RA verifica la identidad y solicita a la **CA** que emita el certificado.
3. La CA emite el certificado (clave pública + identidad del sujeto) y actualiza la **VA**.
4. El sujeto firma digitalmente con su clave privada e incluye el certificado en el mensaje.
5. El receptor verifica el certificado consultando a la **VA**.
6. La VA compara el certificado con la información actualizada por la CA y determina si es válido.

La distinción crítica: la **RA verifica identidad** pero **no emite certificados** (eso lo hace la CA). La **VA almacena y valida** certificados pero no los emite.

---

### Certificado firmado (CA) vs. certificado autofirmado

| Aspecto | CA-Signed | Self-Signed |
|---|---|---|
| Quién lo emite | CA de confianza externa | El propio usuario |
| Verificación | El receptor consulta a la VA | El receptor consulta al propio usuario |
| Herramientas | CA comercial (Comodo, DigiCert, etc.) | Adobe Acrobat, Java keytool, Apple Keychain |
| Contenido | Nombre, nº de serie, expiración, clave pública, **firma de la CA** | Nombre, clave pública, **firma del propio usuario** |
| Confianza | Confianza transitiva a través de la CA | Confianza directa manual |

---

### Firma digital — mecanismo interno

Una firma digital simula propiedades de la firma manuscrita mediante criptografía asimétrica. El flujo es:

1. Se aplica una **función hash** al mensaje → se obtiene el *message digest* (huella).
2. El emisor cifra ese hash con su **clave privada** → esto es la firma.
3. El receptor descifra la firma con la **clave pública del emisor** → recupera el hash original.
4. El receptor calcula independientemente el hash del mensaje recibido.
5. Si ambos hashes coinciden → integridad y autenticidad verificadas.

Si el atacante modifica el mensaje, el hash cambia → la verificación falla. La firma no cifra el mensaje completo (eso lo hace el cifrado); solo cifra el hash. Para confidencialidad adicional, el mensaje firmado puede cifrarse por separado.

---

### SSL — Secure Sockets Layer

SSL es un protocolo de capa de aplicación desarrollado por Netscape para asegurar transmisiones en Internet. Requiere TCP como transporte subyacente. Usa **RSA para cifrado asimétrico** inicial y luego **claves simétricas** para la sesión.

**Tres propiedades de canal SSL:**
- **Canal privado**: mensajes cifrados tras el handshake con clave secreta.
- **Canal autenticado**: el servidor siempre se autentica; el cliente opcionalmente.
- **Canal fiable**: comprobación de integridad en cada transferencia.

**Flujo del SSL Handshake (simplificado):**
1. Client Hello → atributos: versión de protocolo, session ID, cipher suite, método de compresión.
2. Server Hello + Certificate (+ Server Key Exchange opcional) + (Certificate Request opcional) + Server Hello Done.
3. Client Certificate (si se solicitó) + Client Key Exchange + Certificate Verify (si aplica).
4. Change Cipher Spec (cliente) + Finished (cliente).
5. Change Cipher Spec (servidor) + Finished (servidor).
6. Inicio de intercambio de datos de aplicación.

**Reanudación de sesión SSL**: el cliente envía el session ID de una sesión anterior; si el servidor lo reconoce, evita el handshake completo y restablece directamente con ese estado.

---

### 🔴 TLS — Transport Layer Security

TLS es el sucesor de SSL. Usa **RSA con 1024 y 2048 bits**. Combina:
- **Cifrado simétrico** para datos bulk.
- **Cifrado asimétrico** para autenticación e intercambio de claves.
- **MAC** para integridad de mensajes.

TLS es **independiente del protocolo de aplicación**: HTTP, FTP, etc. pueden operar sobre TLS de forma transparente.

**Dos capas de TLS:**

| Capa | Función |
|---|---|
| TLS Record Protocol | Fragmenta, comprime, aplica MAC, cifra y envía datos. Usa **DES** (simétrico). Dos propiedades: privacidad (cifrado simétrico con clave única por conexión) e integridad (MAC con hash seguro: SHA o MD5) |
| TLS Handshake Protocol | Autentica cliente/servidor, negocia algoritmos y genera claves. Usa criptografía asimétrica |

**Flujo del TLS Handshake:**
1. Client Hello (valor aleatorio del cliente + cipher suites soportados).
2. Server Hello (valor aleatorio del servidor) + Certificate + (Certificate Request) + Server Hello Done.
3. Client Certificate (si se solicitó) + Client Key Exchange (**pre-master secret cifrado con clave pública del servidor**).
4. Cliente y servidor derivan independientemente el **master secret** y las **session keys** a partir del pre-master secret.
5. Change Cipher Spec (cliente) + Client Finished.
6. Change Cipher Spec (servidor) + Server Finished.
7. Intercambio de datos cifrados con session keys.

Diferencia clave frente a SSL: en TLS el pre-master secret se cifra con la **clave pública del servidor** (asimétrico); las session keys resultantes se usan simétricamente.

---

### PGP — Pretty Good Privacy

PGP es un protocolo híbrido: combina la velocidad del cifrado simétrico con la gestión de claves del asimétrico.

**Algoritmos usados:**
- Transporte de clave: **RSA**
- Cifrado bulk del mensaje: **IDEA**
- Firmas digitales: **RSA**
- Message digest: **MD5**
- Compresión: ZIP/ZLIB (antes del cifrado)

**Flujo de cifrado PGP:**
1. PGP **comprime** el plaintext (reduce patrones explotables por criptoanálisis).
2. PGP genera una **clave aleatoria de sesión** (one-time secret key).
3. Cifra el mensaje con esa clave de sesión (cifrado simétrico).
4. Cifra la clave de sesión con la **clave pública del receptor** (RSA).
5. Envía: ciphertext + clave de sesión cifrada.

**Flujo de descifrado PGP:**
1. El receptor usa su **clave privada** para descifrar la clave de sesión.
2. Usa la clave de sesión para descifrar el mensaje.

PGP se describe como "hybrid cryptosystem": convencional (~1000x más rápido que público) + público (gestión de claves).

---

### GPG — GNU Privacy Guard

GPG es la implementación libre del estándar **OpenPGP**. También es un sistema de cifrado híbrido. Soporta **S/MIME, SSH, ECC (ECDSA, ECDH, EdDSA)** y usa la biblioteca criptográfica **Libgcrypt**.

**Diferencia operativa GPG vs. PGP:**
- GPG firma con la **clave privada del emisor** y cifra con la **clave pública del receptor**.
- La verificación de firma se realiza automáticamente al descifrar usando la **clave pública del emisor**.

---

### Web of Trust (WoT)

WoT es el modelo de confianza de PGP/OpenPGP/GnuPG. Frente a PKI (donde solo la CA centralizada firma certificados), en WoT **cada usuario actúa como CA** y puede firmar certificados de otros usuarios en los que confía.

La confianza se extiende por cadenas transitivas: si A confía en B y B confía en C, A puede confiar indirectamente en C. Es un modelo **descentralizado** de distribución de claves.

---

### S/MIME — Secure/Multipurpose Internet Mail Extensions

S/MIME permite cifrar correos electrónicos en clientes como Outlook. Requiere un certificado de firma vinculado al keychain del sistema. El icono de **cerrojo** indica mensaje cifrado; el **tick azul/verde** indica mensaje firmado digitalmente.

---

### 🔴 Blockchain — estructura y tipos

Un blockchain es un **distributed ledger technology (DLT)** donde los datos se almacenan en bloques encadenados criptográficamente. Cada bloque contiene:
1. **Data** (detalles de la transacción).
2. **Hash** (del bloque actual).
3. **Hash del bloque anterior** (encadenamiento).

El primer bloque se llama **genesis block** y se representa con 0s.

**Mecanismos criptográficos:**
- **Hash functions**: principalmente **SHA-256**.
- **Algoritmos de clave asimétrica** para validación.

**Proof of Work**: proceso por el que los *crypto miners* validan bloques usando algoritmos criptográficos complejos. Los mineros reciben compensación por este trabajo. Si se altera un bloque, el hash ya no coincide con el almacenado en el bloque siguiente → la cadena lo invalida automáticamente. Aun así, un atacante podría recalcular todos los hashes → el Proof of Work hace esto computacionalmente inviable.

**Cuatro variantes de blockchain:**

| Tipo | Control | Quién participa | Permisos | Ejemplos | Uso |
|---|---|---|---|---|---|
| Public | Descentralizado, sin autoridad central | Cualquiera | Sin permisos | Bitcoin, Ethereum | B2C |
| Private | Supervisor/autoridad central | Solo miembros autorizados | Controlados por administrador | Hyperledger, Ripple (XRP) | B2B; defensa, banca |
| Federated/Consortium | Grupo de organizaciones (no una sola entidad) | Nodos predeterminados de confianza | Compartidos entre el grupo | EWF (energía), R3 (bancos) | Gobierno, bancos centrales |
| Hybrid | Mixto (privado + público) | Variable | Datos selectivamente públicos | IBM Food Trust | Organizaciones que combinan necesidades |

---

### Disk Encryption

Cifra cada bit almacenado en un disco o volumen. Protege datos cuando el SO no está activo. Funciona **on-the-fly**: los datos se cifran/descifran automáticamente en memoria (RAM) sin intervención del usuario.

**Herramientas clave:**
- **VeraCrypt**: volúmenes cifrados on-the-fly; cifra todo el sistema de ficheros (nombres, metadata, espacio libre).
- **BitLocker**: Microsoft; usa **TPM** para protección del volumen Windows completo; protección offline.
- **FileVault** (macOS): usa **XTS-AES-128 con clave de 256 bits**; disponible desde macOS Lion.
- **Rohos**: usa AES-256 (aprobado por NIST); soporta particiones ocultas en USB, Google Drive, OneDrive, Dropbox.
- **Cryptsetup** (Linux): basado en el módulo del kernel **dm-crypt**; soporta LUKS, loop-AES, TrueCrypt/VeraCrypt, BitLocker.

---

## 2. Exam Traps ⚠️

⚠️ **[RA vs. CA — roles en PKI]** — La RA **verifica la identidad** del solicitante pero **no emite certificados**. La CA es quien emite. El examen presenta escenarios donde se confunde quién hace qué. Regla: RA = verificar identidad; CA = emitir certificado.

⚠️ **[VA — función en PKI]** — La VA **almacena** certificados y **valida** su vigencia cuando se consulta. No emite ni verifica identidades. Confusión habitual: asignar a la VA el rol de la CA o de la RA.

⚠️ **[Firma digital — qué se cifra con qué clave]** — En una firma digital, el emisor cifra el **hash del mensaje** con su **clave privada**. El receptor lo verifica con la **clave pública del emisor**. El examen invierte frecuentemente las claves. Regla: privada → firma; pública → verifica.

⚠️ **[PGP — algoritmos específicos]** — PGP usa **RSA** para transporte de clave y firmas, **IDEA** para cifrado bulk, y **MD5** para message digest. El examen puede sustituir IDEA por AES o MD5 por SHA para generar respuestas incorrectas.

⚠️ **[PGP — orden de operaciones]** — PGP **comprime antes de cifrar**, no después. La compresión previa reduce patrones explotables por criptoanálisis. Si el examen ofrece "cifrar luego comprimir" como opción, es incorrecta.

⚠️ **[SSL — autenticación de extremos]** — En SSL, la autenticación del **servidor es obligatoria**; la del **cliente es opcional**. Si el examen pregunta "¿qué extremo siempre se autentica?", la respuesta es el servidor.

⚠️ **[TLS — pre-master secret]** — El pre-master secret lo genera el **cliente** y lo cifra con la **clave pública del servidor**. No es el servidor quien lo genera. Ambos derivan independientemente las session keys a partir de él.

⚠️ **[TLS Record Protocol — algoritmo]** — El TLS Record Protocol usa **DES** para cifrado simétrico. El TLS Handshake usa asimétrico (RSA). Confusión habitual: asignar RSA al Record Protocol.

⚠️ **[WoT vs. PKI — modelo de confianza]** — En PKI, la CA centralizada emite y firma certificados. En WoT, **cada usuario actúa como CA** y puede firmar los certificados de otros. El examen puede preguntar cuál es "descentralizado" → WoT.

⚠️ **[Blockchain — genesis block]** — El primer bloque de un blockchain se llama **genesis block** y su campo de hash del bloque anterior se representa con **0s** (porque no hay bloque previo). El examen puede preguntar sobre esta particularidad.

⚠️ **[Blockchain — tipos B2B vs. B2C]** — Public blockchain → **B2C** (cualquiera puede participar). Private blockchain → **B2B** (acceso controlado). El examen puede presentar escenarios corporativos y preguntar qué tipo de blockchain corresponde.

⚠️ **[FileVault — especificaciones]** — FileVault usa **XTS-AES-128 con clave de 256 bits**. El examen puede confundir el tamaño del bloque (128 bits) con el tamaño de la clave (256 bits), o atribuir AES-256 como nombre del algoritmo cuando el nombre correcto es XTS-AES-128.

⚠️ **[BitLocker — hardware requerido]** — BitLocker usa **TPM** (chip en placa base), no HSM. Si el escenario menciona cifrado de disco completo en Windows con chip de placa base, la respuesta es BitLocker + TPM.

⚠️ **[Proof of Work — blockchain]** — El Proof of Work es el mecanismo que hace computacionalmente inviable que un atacante recalcule todos los hashes de la cadena tras alterar un bloque. No es solo la validación por hash; los hashes solos no son suficientes → el Proof of Work añade el coste computacional.

---

## 3. Nemotécnicos

### PKI components — roles
**"RA Verifica, CA Emite, VA Almacena"**  
→ RA = Verifica identidad | CA = Emite certificado | VA = Almacena y valida

### Flujo firma digital — claves
**"Privada firma, Pública verifica"**  
Cifras con tu privada → cualquiera verifica con tu pública.  
Inverso al cifrado de mensajes (pública cifra, privada descifra).

### PGP — algoritmos
**"RSA transporta, IDEA cifra, MD5 digiere"**  
- RSA → key transport + firmas  
- IDEA → bulk encryption  
- MD5 → message digest

### PGP — orden de operaciones
**"Comprime → Cifra → Empaqueta"**  
1. Compresión (ZIP) → 2. Cifrado simétrico (IDEA) → 3. Cifrado de clave sesión con RSA → 4. Envío

### Blockchain types — acceso
**"Pub=todos, Priv=admin, Fed=grupo, Hyb=mezcla"**  
B2C → Public | B2B → Private | Gobierno/Bancos → Federated | Ambas necesidades → Hybrid

### Blockchain genesis block
"El primer bloque nació de ceros" → hash del bloque anterior = 0s.

### TLS capas — recordar qué hace cada una
**"Record = Cifra datos (DES) | Handshake = Negocia claves (RSA)"**

---

## 4. Flashcards

**Q:** En PKI, ¿qué componente verifica la identidad del solicitante antes de que se emita el certificado?  
**A:** La RA (Registration Authority). Verifica la identidad y solicita a la CA que emita el certificado. La CA no verifica identidades directamente.

---

**Q:** ¿Cuál es el rol de la VA en PKI y cómo interactúa con el receptor de un mensaje firmado?  
**A:** La VA (Validation Authority) almacena certificados con sus claves públicas. Cuando un receptor quiere verificar la autenticidad de un certificado, consulta a la VA, que lo compara con la información actualizada por la CA.

---

**Q:** En una firma digital, ¿con qué clave se genera la firma y con qué clave se verifica?  
**A:** La firma se genera cifrando el hash del mensaje con la **clave privada del emisor**. Se verifica descifrando la firma con la **clave pública del emisor** y comparando el hash resultante con el del mensaje recibido.

---

**Q:** ¿Qué algoritmos usa PGP para cifrado de mensaje, transporte de clave y message digest?  
**A:** Cifrado bulk del mensaje: **IDEA**. Transporte de clave y firmas digitales: **RSA**. Message digest: **MD5**.

---

**Q:** ¿Por qué PGP comprime el mensaje antes de cifrarlo?  
**A:** La compresión previa reduce los patrones estadísticos del plaintext que podrían ser explotados por técnicas de criptoanálisis, aumentando la resistencia al análisis criptográfico.

---

**Q:** ¿Cuál es la diferencia fundamental entre el modelo de confianza PKI y el Web of Trust?  
**A:** En PKI, la confianza es centralizada: solo la CA firma certificados. En WoT, es descentralizada: cada usuario actúa como CA y puede firmar los certificados de otros usuarios en los que confía, creando cadenas transitivas de confianza.

---

**Q:** En SSL, ¿qué extremo de la comunicación siempre se autentica y cuál es opcional?  
**A:** El **servidor siempre se autentica**. La autenticación del **cliente es opcional**.

---

**Q:** Durante el TLS Handshake, ¿quién genera el pre-master secret y cómo se transmite de forma segura?  
**A:** Lo genera el **cliente**. Lo cifra con la **clave pública del servidor** y lo envía. El servidor lo descifra con su clave privada. Ambos derivan independientemente el master secret y las session keys a partir de él.

---

**Q:** ¿Qué algoritmo de cifrado usa el TLS Record Protocol y qué propiedades de seguridad proporciona?  
**A:** Usa **DES** (cifrado simétrico). Proporciona privacidad (cifrado con clave única por conexión) e integridad (MAC usando SHA o MD5).

---

**Q:** Un analista necesita implementar cifrado de disco completo en Windows usando el chip de seguridad de la placa base. ¿Qué solución corresponde?  
**A:** **BitLocker** con **TPM** (Trusted Platform Module). BitLocker usa el TPM para cifrar el volumen completo de Windows y proteger los datos cuando el SO está offline.

---

**Q:** ¿Qué tecnología de cifrado usa FileVault en macOS y cuáles son sus especificaciones?  
**A:** **XTS-AES-128** con una clave de **256 bits**. Disponible desde macOS Lion. El cifrado se inicia automáticamente en segundo plano tras activarlo.

---

**Q:** ¿Qué contiene cada bloque de un blockchain y cómo funciona la invalidación ante una manipulación?  
**A:** Cada bloque contiene: datos de la transacción, hash del bloque actual y hash del bloque anterior. Si se altera un bloque, su hash cambia y ya no coincide con el hash almacenado en el siguiente bloque, invalidando la cadena desde ese punto.

---

**Q:** ¿Qué es el "genesis block" en un blockchain?  
**A:** Es el primer bloque de la cadena. Su campo de "hash del bloque anterior" se representa con 0s porque no existe bloque previo.

---

**Q:** ¿Qué tipo de blockchain usa Bitcoin y Ethereum, y para qué modelo de negocio es adecuado?  
**A:** **Public blockchain** (descentralizado, sin autoridad central, sin permisos de acceso). Es adecuado para servicios **B2C**.

---

**Q:** ¿Qué diferencia a un federated/consortium blockchain de un private blockchain?  
**A:** En un private blockchain, una sola autoridad central controla el acceso. En un federated blockchain, el control lo ejerce un **grupo de organizaciones o individuos** (no una sola entidad). Ejemplos: EWF (energía), R3 (bancos).

---

**Q:** ¿Qué herramienta de Linux se usa para cifrado de disco basado en dm-crypt y qué formatos soporta?  
**A:** **Cryptsetup**. Soporta: dm-crypt plano, LUKS, loop-AES, TrueCrypt/VeraCrypt y BitLocker.

---

**Q:** ¿Por qué el Proof of Work hace seguro un blockchain aunque un atacante pueda recalcular hashes?  
**A:** Recalcular los hashes de todos los bloques posteriores al alterado es computacionalmente factible. El Proof of Work añade un coste computacional elevado a cada bloque, haciendo inviable en la práctica recalcular toda la cadena.

---

**Q:** ¿Qué es GPG y en qué se diferencia de PGP a nivel de licencia y estándar?  
**A:** GPG (GNU Privacy Guard) es una implementación **libre y open-source** del estándar **OpenPGP**. PGP es el protocolo original propietario. GPG soporta además S/MIME, SSH y ECC (ECDSA, ECDH, EdDSA), y usa la biblioteca Libgcrypt.

---

## 5. Confusión frecuente

### CA vs. RA en PKI
- **CA**: emite y firma certificados digitales. Es la autoridad de certificación real.
- **RA**: verifica la identidad del solicitante y actúa como intermediario antes de que la CA emita el certificado. No emite certificados.
- **Criterio**: si la pregunta es "¿quién emite el certificado?" → CA. Si es "¿quién verifica la identidad del solicitante?" → RA.

---

### Firma digital vs. Cifrado de mensaje
- **Firma digital**: se cifra el **hash** del mensaje con la **clave privada** del emisor → proporciona autenticación y no repudio.
- **Cifrado de mensaje**: se cifra el **mensaje completo** con la **clave pública** del receptor → proporciona confidencialidad.
- **Criterio**: si la pregunta menciona "no repudio" o "autenticidad del emisor" → firma digital (clave privada). Si menciona "solo el receptor puede leer" → cifrado (clave pública del receptor).

---

### SSL vs. TLS
- **SSL**: protocolo desarrollado por Netscape; usa RSA; requiere TCP; el servidor siempre se autentica, el cliente opcionalmente. Tecnológicamente obsoleto.
- **TLS**: sucesor de SSL; dos capas (Record + Handshake); RSA 1024/2048 bits; independiente del protocolo de aplicación; más seguro y moderno.
- **Criterio**: si la pregunta menciona "desarrollado por Netscape" o es sobre el protocolo histórico → SSL. Para implementaciones modernas y seguras → TLS.

---

### PGP vs. GPG
- **PGP**: protocolo original; usa IDEA para bulk encryption, RSA para claves, MD5 para digest. Propietario en origen.
- **GPG**: implementación libre de OpenPGP; más moderna; soporta ECC, S/MIME, SSH; usa Libgcrypt. El cifrador por defecto es CAST-128.
- **Criterio**: si la pregunta especifica "IDEA" o "MD5" como algoritmo interno → PGP clásico. Si menciona "OpenPGP", "ECC" o "libre/open-source" → GPG.

---

### Public vs. Private Blockchain
- **Public**: sin autoridad central, cualquiera puede unirse, permissionless, B2C. Ejemplos: Bitcoin, Ethereum.
- **Private**: autoridad central que controla el acceso, solo miembros autorizados, B2B. Ejemplos: Hyperledger, Ripple.
- **Criterio**: si el escenario menciona "banco", "defensa" o control corporativo → private. Si menciona "cualquiera puede participar" o aplicaciones de consumo → public.

---

### Federated vs. Hybrid Blockchain
- **Federated/Consortium**: control compartido por un **grupo de organizaciones** (no pública, no de una sola empresa). Rápido y escalable. EWF, R3.
- **Hybrid**: combina elementos públicos y privados en una **misma cadena**, con datos seleccionables entre públicos y privados. IBM Food Trust.
- **Criterio**: si la pregunta menciona "varios bancos/organizaciones comparten control" → federated. Si menciona "algunos datos públicos, otros privados en la misma organización" → hybrid.
