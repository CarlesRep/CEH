# M17_05 — Mobile Security Guidelines and Tools
**Módulo 17 / Subapartado 5 — Mobile Security Guidelines & Tools**

---

## 1. Conceptos y definiciones

### OWASP Top 10 Mobile — Soluciones por categoría 🔴

El examen presenta escenarios de mitigación y pregunta qué riesgo OWASP abordan. La lógica interna de cada solución es la que determina la respuesta correcta.

| Riesgo OWASP | Soluciones clave (orientadas al examen) |
|---|---|
| **M1 — Improper Credential Usage** | No hardcodear credenciales; cifrar en transmisión; usar access tokens revocables en lugar de credenciales en el dispositivo; rotar API keys/tokens regularmente |
| **M2 — Inadequate Supply Chain Security** | Solo librerías de terceros validadas y de confianza; app signing y distribución seguros; monitorizar incidentes de supply chain mediante scanning |
| **M3 — Insecure Auth/Authorization** | No usar patrones débiles de autenticación; reforzar autenticación server-side; usar Face ID/Touch ID; validar roles y permisos; integrity checks locales para detectar alteraciones de código |
| **M4 — Insufficient Input/Output Validation** | Validación estricta de input/output; validación específica por contexto de datos; data integrity checks; secure coding practices |
| **M5 — Insecure Communication** | SSL/TLS en canales de transporte de la app; cipher suites fuertes con longitud de clave adecuada; certificados firmados por CA de confianza; **nunca certificados self-signed, caducados, revocados, de host incorrecto**; certificate pinning; verificar siempre SSL chain; capa de cifrado adicional antes de enviar al canal SSL; no enviar datos sensibles por SMS/MMS/notificaciones |
| **M6 — Inadequate Privacy Controls** | Minimizar PII procesada; autenticación y autorización para acceso a datos; herramientas de análisis estático/dinámico para detectar logging de datos sensibles; ofuscación de código y anti-tampering |
| **M7 — Insufficient Binary Protections** | Integrity checks en el arranque para detectar redistribución/modificación no autorizada; backend enforcement; local security checks |
| **M8 — Security Misconfiguration** | Configuraciones por defecto aseguradas; no hardcodear credenciales por defecto; no permisos excesivos (world-readable/world-writable); solicitar solo permisos necesarios; deshabilitar cleartext traffic; certificate pinning; deshabilitar debugging en producción; **deshabilitar backup mode en Android**; limitar superficie de ataque exportando solo activities/content providers/services necesarios |
| **M9 — Insecure Data Storage** | Cifrado robusto; HTTPS/SSL-TLS para transmisión; almacenamiento en ubicaciones con acceso restringido; controles de acceso robustos; input validation y sanitización; gestión segura de sesiones (tokens aleatorios, timeouts, almacenamiento seguro cliente y servidor); actualizar librerías y dependencias |
| **M10 — Insufficient Cryptography** | Algoritmos de cifrado fuertes con longitud de clave suficiente; key management seguro con **key vaults o HSMs** (Hardware Security Modules); evitar implementaciones de cifrado personalizadas; **SHA-256 o bcrypt** para hash; **salting** al hashear contraseñas; **PBKDF2, bcrypt o scrypt** para key derivation functions (KDF) para hashing de contraseñas; validar certificados y firmas digitales |

---

### Buenas prácticas específicas por plataforma para M5 🔴

**iOS específico:**
- Certificados válidos y fail closed
- Usar Secure Transport API para designar certificados de cliente de confianza en **CFNetwork**
- Asegurar que las llamadas NSURL no permitan certificados self-signed o inválidos (clase `setAllowsAnyHTTPSCertificate`)
- Implementar certificate pinning usando método NSURL `connection:willSendRequestForAuthenticationChallenge`

**Android específico:**
- Eliminar código que acepte certificados indiscriminadamente como `org.apache.http.conn.ssl.AllowAllHostnameVerifier` o `SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER`
- Si se extiende SSLSocketFactory, asegurarse de que `checkServerTrusted` esté correctamente implementado
- No sobreescribir `onReceivedSslError` para permitir certificados SSL inválidos

---

### Almacenamiento seguro — Android KeyStore vs iOS Keychain 🔴

#### Android KeyStore
- Mecanismos de autenticación: patrones, PINs, contraseñas, huellas dactilares
- Usar **hardware-backed Android KeyStore**
- Almacenar master key y otras claves en **ubicaciones diferentes**
- El master key puede almacenarse en la implementación **software** del Android KeyStore
- Derivar claves usando la passphrase del usuario
- Al usar SharedPreferences: **cifrar los datos** para añadir capa adicional de seguridad
- Principio de **least privilege** para acceso a datos sensibles
- No hardcodear datos sensibles (API keys, tokens, credenciales) en el código fuente
- **Ofuscar código y datos** para dificultar reverse engineering

#### iOS Keychain
- Mecanismos: Touch ID, Face ID, passcodes, contraseñas
- **Cifrado AES-256-bit hardware-backed**
- Usar **ACLs (Access Control Lists)** para especificar accesibilidad al keychain por apps
- Almacenar solo **pequeños fragmentos de datos** directamente en el keychain
- Especificar `AccessControlFlags` para autenticar la clave
- Implementar mecanismo para **borrar los datos del keychain al desinstalar la app**
- Datos compartidos entre app principal y extensiones deben estar **cifrados y asegurados**

**Diferencia clave:** iOS usa **AES-256-bit** específicamente; Android usa hardware-backed KeyStore (sin cifrado de bit fijo mencionado explícitamente). iOS borra el keychain al desinstalar si se implementa correctamente; Android almacena master key en ubicación software separada del resto.

---

### Countermeasures — SMiShing 🔴

- Nunca responder a SMS sospechosos sin verificar la fuente
- No hacer clic en enlaces de SMS
- Nunca responder a SMS que soliciten información personal o financiera
- No llamar a números dejados en un SMS
- Activar la función "block texts from the internet" del operador
- No responder a SMS que transmitan urgencia o soliciten acción rápida
- Evitar mensajes de números no telefónicos (relay services de internet)
- Verificar errores ortográficos, gramaticales o inconsistencias de idioma
- Instalar software anti-phishing o herramientas de filtrado de SMS
- MFA como capa adicional de seguridad
- Las organizaciones deben usar **short codes oficiales** para sus comunicaciones
- Ejecutar **phishing simulations** para evaluar la concienciación de los usuarios
- Usar plataformas de mensajería autorizadas como **Signal o WhatsApp** para comunicaciones internas
- Plan claro para responder a incidentes de smishing, especialmente para dispositivos BYOD

---

### Countermeasures — OTP Hijacking 🔴

**Para usuarios:**
- Política de contraseñas: únicas, fuertes, sin reutilización, rotación periódica, password manager cifrado
- Actualizar software y OS periódicamente
- Habilitar **SIM locking mediante PIN** para evitar acceso no autorizado a la SIM
- **Deshabilitar visualización de notificaciones sensibles en la pantalla de bloqueo**
- Evitar apps que se autentiquen vía SMS
- Minimizar el uso de métodos de recuperación vía SMS o email
- Nunca reenviar OTPs a terceros
- Introducir el OTP **manualmente en el navegador** (no copy-paste)

**Para desarrolladores:**
- OTPs transmitidos por canales cifrados (SMS cifrado o push notifications seguros)
- OTPs con **cifrado end-to-end**
- Combinar OTP con otros factores de autenticación (biométrico o hardware)
- **Limitar el número de solicitudes OTP** desde un único usuario (anti-brute-force)
- **Tiempos de expiración cortos** para OTPs
- Behavioral analytics para detectar múltiples solicitudes OTP en poco tiempo
- Considerar **hardware-based OTP generators o security keys**
- Algoritmos seguros para generación OTP: **HOTP (HMAC-based OTP) o TOTP (Time-based OTP)**
- OTPs únicos por evento de autenticación; **nunca reutilizados**

---

### General Guidelines — Mobile Platform Security (puntos clave de examen)

- **No conectar dos redes separadas simultáneamente** (ej. Wi-Fi y Bluetooth al mismo tiempo)
- Bluetooth **desactivado por defecto**; activar solo cuando sea necesario
- Passcode de **8 caracteres complejos** recomendado
- Timeout de bloqueo automático: **1 minuto**
- Permitir solo **aplicaciones firmadas** para instalar o ejecutar
- **Sandbox** de aplicaciones y datos
- Deshabilitar el logging excesivo en el dispositivo
- Usar red de datos **celular** en lugar de Wi-Fi público
- Siempre cerrar sesión en apps móviles tras el uso, especialmente apps enlazadas entre sí
- Usar datos sensibles con **follow-me-data y ShareFile** como solución gestionada por la empresa
- Usar Citrix para mantener datos en el data center y dispositivos personales separados
- Verificar la ubicación de impresoras antes de imprimir documentos sensibles
- Deshabilitar la colección de datos de diagnóstico y uso

---

### Mobile Device Security — Guidelines para Administrador (datos específicos)

- Publicar política de empresa para dispositivos consumer-grade y BYOD
- Publicar política de empresa para la nube
- Especificar **session timeout** a través de Access Gateway
- Especificar si el domain password puede cachearse en el dispositivo o debe introducirse en cada acceso
- Métodos de autenticación en Access Gateway permitidos:
  - No authentication
  - Domain only
  - SMS authentication
  - RSA SecurID only
  - **Domain + RSA SecurID**
- Mantener relojes de dispositivos móviles **sincronizados con una fuente de tiempo común**
- Emplear **UEM (Unified Endpoint Management)** que extiende EMM y MAM a todos los endpoints
- Utilizar plataformas **MTD (Mobile Threat Defense)** con análisis de comportamiento
- Emplear biométricos: huella dactilar, voz, facial, iris
- Utilizar **CASB (Cloud Access Security Broker)** como capa adicional entre usuarios cloud y proveedores de servicios

---

### Reverse Engineering de aplicaciones móviles

**Definición:** análisis y extracción del código fuente de una app para comprenderla y, si es necesario, regenerarla con modificaciones.

**Usos legítimos y maliciosos:**
- Leer y comprender código fuente
- Detectar vulnerabilidades subyacentes
- Escanear información sensible embebida (hardcoded credentials)
- Análisis de malware
- Regenerar la app con modificaciones (clone apps)
- Verificar cumplimiento de estándares (GDPR, HIPAA)
- Comprender compatibilidad multiplataforma
- Debug y troubleshooting

**Por qué es efectivo en testing:**
- Superar controles que bloquean análisis dinámico (end-to-end encryption, SSL, root detection)
- Análisis estático en black-box testing: binario y bytecode para descubrir vulnerabilidades de hardcoded credentials
- Resilience assessment: verificar que las protecciones anti-reversing (MASVS-R) funcionan intentando vulnerarlas

---

### Herramientas — Mapa completo por categoría

| Categoría | Herramienta principal | Función clave |
|-----------|----------------------|---------------|
| Source Code Analysis | Syhunt Mobile | >350 checks Java/Android; >240 checks iOS (19 categorías); escanea OWASP Mobile Top 10 |
| Reverse Engineering | Apktool | Desensamblar/reconstruir APKs; smali debugging; recursos near-original |
| Reverse Engineering | Androguard, Frida, JEB, Bytecode Viewer | Análisis adicional |
| App Repackaging Detection | Appdome | RASP para Android/iOS; verifica integridad mediante checksums del binario; anti-tampering, anti-hooking, anti-repackaging; **sin modificar el código fuente** |
| Mobile Protection | Avast, Comodo Mobile Security, AVG Mobile Security | Antivirus, VPN, ID protection, Wi-Fi security |
| Mobile Anti-Spyware | TotalAV | Detecta spyware, malware, hardware (adware agresivo) |
| Mobile Pen Testing | ImmuniWeb MobileSuite | ML para pentesting iOS/Android; testing estático, dinámico e interactivo (SCA); reportes CVE/CWE/CVSSv3; integración CI/CD |

**App Repackaging:** proceso de extraer detalles de una app legítima de tiendas oficiales, modificarla inyectando código malicioso y redistribuirla como app auténtica.

---

### Algoritmos y estándares criptográficos mencionados explícitamente 🔴

| Uso | Algoritmo/Estándar recomendado |
|-----|-------------------------------|
| Hash de contraseñas | **SHA-256** o **bcrypt** |
| KDF (Key Derivation Function) | **PBKDF2**, **bcrypt**, **scrypt** |
| Salting | Obligatorio al hashear contraseñas |
| Almacenamiento seguro en iOS | **AES-256-bit** (hardware-backed) |
| Generación de OTP | **HOTP** (HMAC-based) o **TOTP** (Time-based) |
| Key storage empresarial | **Key vaults** o **HSMs** (Hardware Security Modules) |
| Transporte | **TLS / HTTPS** |

---

## 2. Exam Traps ⚠️ 🔴

⚠️ **[M8 — deshabilitar backup mode]** La solución específica de Android para M8 (Security Misconfiguration) incluye **deshabilitar el backup mode en Android**. Dato específico de plataforma que el examen puede presentar en opciones.

⚠️ **[M10 — KDF recomendadas]** Para hashing de contraseñas, las KDF recomendadas son **PBKDF2, bcrypt o scrypt**, no SHA-256 directamente. SHA-256 es para hashing genérico. BCrypt aparece en ambas listas; el contexto determina si es hash o KDF.

⚠️ **[M10 — evitar implementaciones propias de cifrado]** Una trampa clásica: el examen puede presentar "implementar tu propio algoritmo de cifrado" como opción de mejora. La recomendación es exactamente la contraria: **evitar implementaciones de cifrado personalizadas**.

⚠️ **[Certificate pinning — cuándo aplicar]** Certificate pinning aparece como mitigación para M5 (Insecure Communication) y también en M8 (Security Misconfiguration). El examen puede preguntar a qué riesgo pertenece: es mitigación de ambos, pero principalmente de M5.

⚠️ **[iOS Keychain — tamaño de datos]** El keychain iOS está diseñado para almacenar solo **pequeños fragmentos de datos**. No es un almacenamiento de propósito general. Si el examen pregunta qué almacenar en el keychain, la respuesta es datos pequeños y críticos (tokens, claves, credenciales), no ficheros grandes.

⚠️ **[OTP — HOTP vs TOTP]** HOTP = HMAC-based (basado en contador). TOTP = Time-based (basado en tiempo). Ambos son algoritmos seguros para generación de OTP. El examen puede preguntar cuál es "time-based": TOTP.

⚠️ **[Passcode — longitud recomendada]** La recomendación general es un passcode **complejo de 8 caracteres**. El timeout de auto-lock recomendado es **1 minuto**. Datos numéricos directamente preguntables.

⚠️ **[SMiShing — short codes]** Las organizaciones deben usar **official short codes** para sus comunicaciones, no números normales. Esto mejora la legitimidad y facilita que los usuarios distingan comunicaciones reales de ataques.

⚠️ **[CASB — posición en la arquitectura]** CASB se posiciona como **capa de seguridad adicional entre usuarios cloud y proveedores de servicios**. No es un MDM ni un EMM; es específico del acceso a servicios cloud.

⚠️ **[UEM vs EMM]** UEM (Unified Endpoint Management) **extiende** las capacidades de EMM y MAM a **todos los endpoints** (no solo móviles). EMM es específico de movilidad. Si el examen pregunta qué gestiona todos los endpoints → UEM.

⚠️ **[Appdome — sin modificar código fuente]** Appdome automatiza la seguridad de apps **sin requerir cambios en el código fuente**. Característica diferencial que el examen puede usar para distinguirlo de otras soluciones.

⚠️ **[Android: `checkServerTrusted` vs `onReceivedSslError`]** Son dos puntos distintos de bypass de SSL en Android. `checkServerTrusted` debe estar correctamente implementado si se extiende SSLSocketFactory. `onReceivedSslError` no debe sobreescribirse para aceptar certificados inválidos. El examen puede preguntar cuál de los dos es la vulnerabilidad cuando una app acepta cualquier certificado.

---

## 3. Nemotécnicos

### M10 — Algoritmos criptográficos recomendados
**"SHA-B para hash, PBS para KDF"**:
- Hash: **SHA**-256, **B**crypt
- KDF: **P**BKDF2, **B**crypt, **S**crypt

Regla adicional: bcrypt aparece en ambos grupos (es KDF y también hash).

### iOS Keychain — cifrado específico
**"iK-256"**: iOS Keychain = **256**-bit AES hardware-backed

### OTP algorithms
- **H**OTP = **H**MAC (HMAC-based, contador)
- **T**OTP = **T**empo/Time (time-based)
Regla: la T de TOTP = Time.

### Access Gateway — métodos de autenticación (5)
**"N-D-S-R-DR"**:
- **N**o auth → **D**omain only → **S**MS auth → **R**SA SecurID → **D**omain + **R**SA SecurID

### Passcode guidelines — datos numéricos
- Longitud: **8** caracteres complejos
- Auto-lock: **1** minuto
- Erase Data iOS: **10** intentos

### Countermeasures SMiShing (las 3 "NUNCA")
**"NUNCA"**: Nunca responder / Nunca hacer clic en links / Nunca llamar al número del SMS

### Android SSL bypass — 3 puntos críticos
**"AAO"**: **A**llowAllHostnameVerifier → **A**LLOW_ALL_HOSTNAME_VERIFIER → **O**nReceivedSslError → todos deben eliminarse/no implementarse.

---

## 4. Flashcards

**Q:** ¿Qué KDFs recomienda OWASP Mobile para el hashing de contraseñas?
**A:** **PBKDF2, bcrypt o scrypt**. Además, es obligatorio implementar **salting** al hashear contraseñas. SHA-256 es para hashing genérico, no específicamente para contraseñas.

---

**Q:** ¿Qué tipo de cifrado usa el iOS Keychain y qué mecanismo de autenticación lo protege?
**A:** **AES-256-bit hardware-backed**. Se protege mediante Touch ID, Face ID, passcodes o contraseñas. Solo debe almacenar **pequeños fragmentos de datos**.

---

**Q:** ¿Qué hace Appdome y cuál es su característica diferencial frente a otras soluciones de protección de apps?
**A:** Ofrece solución **RASP** (Runtime Application Self-Protection) para Android e iOS. Su característica diferencial: automatiza la seguridad de apps (anti-tampering, anti-hooking, anti-repackaging) **sin requerir cambios en el código fuente**.

---

**Q:** ¿Qué es CASB y dónde se posiciona en la arquitectura de seguridad móvil?
**A:** **Cloud Access Security Broker** — capa de seguridad adicional posicionada entre usuarios cloud y proveedores de servicios. Proporciona visibilidad y control sobre el uso de servicios cloud.

---

**Q:** ¿Cuál es la diferencia entre UEM y EMM?
**A:** **EMM** (Enterprise Mobility Management) gestiona dispositivos móviles. **UEM** (Unified Endpoint Management) extiende las capacidades de EMM y MAM a **todos los endpoints** (no solo móviles). UEM tiene alcance más amplio.

---

**Q:** ¿Cuáles son los algoritmos de generación de OTP seguros mencionados en el CEH y qué significa cada acrónimo?
**A:** **HOTP** (HMAC-based One-Time Password) — basado en contador. **TOTP** (Time-based One-Time Password) — basado en tiempo. Los OTPs deben ser únicos por evento y nunca reutilizados.

---

**Q:** ¿Qué medida específica de Android para M8 (Security Misconfiguration) menciona el CEH?
**A:** **Deshabilitar el backup mode en Android**. También: deshabilitar debugging en producción, no usar permisos world-readable/world-writable, limitar activities/content providers/services exportados.

---

**Q:** ¿Qué deben hacer los desarrolladores para mitigar M5 (Insecure Communication) según las buenas prácticas Android?
**A:** Eliminar código como `AllowAllHostnameVerifier` o `SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER`; implementar `checkServerTrusted` correctamente si se extiende SSLSocketFactory; no sobreescribir `onReceivedSslError` para aceptar certificados inválidos.

---

**Q:** ¿Qué contramedidas específicas para OTP hijacking se recomiendan para los desarrolladores?
**A:** Tiempos de expiración cortos, limitar solicitudes OTP por usuario, OTPs únicos y nunca reutilizados, cifrado end-to-end, behavioral analytics para detectar múltiples solicitudes, considerar hardware-based OTP generators, usar HOTP o TOTP.

---

**Q:** ¿Cuál es la longitud de passcode recomendada en las guidelines generales de seguridad móvil?
**A:** **8 caracteres complejos**. Timeout de auto-lock: **1 minuto**. Activar borrado de datos (Erase Data/wipe) tras cierto número de intentos fallidos.

---

**Q:** ¿Por qué no se deben conectar dos redes separadas simultáneamente en un dispositivo móvil?
**A:** Conectar Wi-Fi y Bluetooth simultáneamente crea un vector de ataque: un atacante puede usar un canal para pivotar al otro, comprometiendo datos que circulan por ambas redes.

---

**Q:** ¿Qué es app repackaging y cómo lo detecta Appdome?
**A:** Proceso de extraer una app legítima de tiendas oficiales, inyectar código malicioso y redistribuirla como auténtica. Appdome verifica la integridad mediante **checksums del binario y la estructura** de la app, detectando modificaciones no autorizadas.

---

**Q:** ¿Qué función tiene Syhunt Mobile y cuántos checks realiza para Android e iOS?
**A:** Analiza código fuente de apps móviles. Para Android (Java): más de **350 vulnerability checks**. Para iOS (Swift, Objective-C, C): más de **240 checks** en 19 categorías de vulnerabilidad. Escanea OWASP Mobile Top 10.

---

**Q:** ¿Qué contramedida de OTP hijacking se recomienda a nivel de SIM para los usuarios?
**A:** Habilitar **SIM locking mediante PIN** para evitar el acceso no autorizado a la SIM. También: deshabilitar la visualización de notificaciones sensibles en la pantalla de bloqueo.

---

**Q:** ¿Qué métodos de autenticación soporta Access Gateway según las guidelines para administradores?
**A:** No authentication, Domain only, SMS authentication, RSA SecurID only, **Domain + RSA SecurID** (combinación). El administrador debe especificar cuál está permitido en la política.

---

**Q:** ¿Qué es MASVS-R y para qué se usa en el contexto del CEH?
**A:** **Mobile Application Security Verification Standard Anti-Reversing Controls**. Estándar para proteger apps contra reverse engineering. Se verifica su eficacia mediante **resilience assessment** (intentar hacer reverse engineering para comprobar que las protecciones funcionan).

---

**Q:** ¿Qué recomienda el CEH para el almacenamiento de claves de cifrado en entorno empresarial?
**A:** Usar **key vaults o HSMs (Hardware Security Modules)** para almacenar claves de cifrado de forma segura. Evitar almacenar claves directamente en el código o en ubicaciones no protegidas.

---

**Q:** ¿Qué debe especificarse respecto al domain password en la política de Access Gateway para dispositivos móviles?
**A:** Si el domain password puede **cachearse en el dispositivo** o si los usuarios deben **introducirlo en cada solicitud de acceso**. Es una decisión de política del administrador.

---

## 5. Confusión frecuente

### SHA-256 (hash) vs PBKDF2/bcrypt/scrypt (KDF para contraseñas)
- **SHA-256:** función hash criptográfica para propósito general (integridad de datos, firmas digitales).
- **PBKDF2/bcrypt/scrypt:** KDFs diseñadas específicamente para hashing de contraseñas; son computacionalmente costosas por diseño para resistir ataques de fuerza bruta.
- **Criterio:** ¿hashing de contraseñas? → PBKDF2, bcrypt o scrypt. ¿Hash general de datos/ficheros? → SHA-256.

---

### HOTP vs TOTP
- **HOTP:** basado en contador (HMAC-based); el OTP cambia con cada uso según un contador compartido.
- **TOTP:** basado en tiempo; el OTP cambia cada N segundos (ej. 30s); requiere sincronización de tiempo entre cliente y servidor.
- **Criterio:** ¿el enunciado dice "time-based"? → TOTP. ¿"HMAC-based" o "counter-based"? → HOTP.

---

### UEM vs EMM vs MDM
- **MDM:** gestión básica de dispositivos móviles (passcode, remote lock/wipe, jailbreak detection).
- **EMM:** extiende MDM + gestión de aplicaciones + gestión de contenido.
- **UEM:** extiende EMM y MAM a **todos los endpoints** (móviles, portátiles, escritorios, IoT).
- **Criterio:** cuanto más amplio el alcance → UEM > EMM > MDM.

---

### Certificate Pinning — mitigación de M5 vs M8
- El examen puede asignar certificate pinning a M5 (Insecure Communication) o a M8 (Security Misconfiguration) dependiendo del contexto.
- **M5:** pinning como medida para asegurar el canal de comunicación.
- **M8:** "Disallow cleartext traffic y usar certificate pinning cuando sea posible" aparece explícitamente en las soluciones de M8.
- **Criterio:** si el contexto es comunicación insegura → M5. Si es misconfiguration de la app → M8. En la práctica es mitigación de ambos.

---

### Appdome (RASP) vs MobSF (análisis)
- **Appdome:** solución de **protección en runtime** (RASP); protege apps en producción contra tampering, hooking y repackaging; no modifica el código fuente.
- **MobSF:** herramienta de **análisis estático y dinámico** de APKs/IPAs; identifica vulnerabilidades; se usa en pentesting y revisión de seguridad, no como protección en producción.
- **Criterio:** ¿proteger la app en producción? → Appdome. ¿Analizar una app en busca de vulnerabilidades? → MobSF.
