# M14_05_WebApplicationSecurity.md
> Módulo 14 / Subapartado 5 — Web Application Security

---

## 1. Conceptos y definiciones

### 🔴 Tipos de Web Application Security Testing

| Tipo | Alias | Conocimiento del tester | Herramientas CEH |
|---|---|---|---|
| **Manual** | — | Variable | SecApps, Selenium, Apache JMeter, Acunetix |
| **Automated** | — | Variable | Ranorex Studio, TestComplete, Leapwork, Katalon, Testsigma |
| **SAST** (Static Application Security Testing) | **Whitebox testing** | Arquitectura + código fuente **conocidos** | Codacy, JFrog, Klocwork, Checkmarx One, PT Application Inspector |
| **DAST** (Dynamic Application Security Testing) | **Blackbox testing** | Arquitectura **desconocida** | Invicti, Acunetix, HCL AppScan, OpenText Fortify On Demand, StackHawk |
| **Fuzz Testing** | **Blackbox testing** | Arquitectura desconocida | WebScarab, Burp Suite, AppScan Standard, Defensics, ffuf |

**SAST:** detecta design flaws en el código fuente; verifica cumplimiento con reglas y estándares. Puede detectar **zero-day vulnerabilities** (AI-powered SAST).

**DAST:** ejecuta código en ejecución; identifica issues en interfaces, requests/responses, sesiones, scripts, autenticación, code injections; usa fuzzing.

---

### 🔴 Web Application Fuzz Testing — Pasos y estrategias

**Pasos (6):**
1. Identify the target system
2. Identify inputs
3. Generate fuzzed data
4. Execute the test using fuzz data
5. Monitor system behavior
6. Log defects

**Estrategias:**

| Estrategia | Descripción |
|---|---|
| **Mutation-Based** | Parte de muestras de datos válidas existentes y las muta para generar datos de prueba nuevos (ciclo iterativo) |
| **Generation-Based** | Genera datos nuevos desde cero; cantidad predefinida según el modelo de testing |
| **Protocol-Based** | El fuzzer envía paquetes forjados; requiere conocimiento detallado del formato del protocolo; escribe especificaciones en el fuzzer y aplica model-based test generation |

**wfuzz para fuzzing con AI:**
```bash
wfuzz -c -z file,/usr/share/wordlists/dirb/common.txt --hc 404 http://www.moviescope.com/FUZZ
```
- `-c`: continuar incluso con respuesta vacía
- `-z file,<wordlist>`: valores desde fichero
- `--hc 404`: ocultar resultados 404 (Not Found)
- `FUZZ`: placeholder donde wfuzz inserta cada ítem de la wordlist

**AI-powered fuzz testing — herramienta:** Prompt Fuzzer — diseñado específicamente para **GenAI applications**; testea prompts contra prompt injection; usa LLMs para analizar respuestas y asignar security scores.

---

### 🔴 SAST vs. DAST — Diferenciadores para el examen

| Aspecto | SAST (Whitebox) | DAST (Blackbox) |
|---|---|---|
| Cuándo se ejecuta | En código estático (sin ejecución) | En aplicación en ejecución |
| Conocimiento del sistema | Arquitectura y código fuente conocidos | Arquitectura desconocida |
| Qué detecta | Design flaws, código fuente | Issues en interfaces, sessions, auth, injections |
| AI-powered ventaja clave | Zero-day vulnerability detection | Reduced false positives; context-aware analysis |
| Herramienta AI-powered DAST | — | ZeroThreat.ai |
| Herramienta AI-powered SAST | Code Genie AI, Codiga | — |

**RASP (Runtime Application Self Protection):** tecnología que protege aplicaciones durante su ejecución. Se sitúa dentro del código de la aplicación (entre server y aplicación). Monitorización continua; detecta ataques en tiempo real incluyendo **zero-day attacks** sin intervención humana. Beneficios: visibility, collaboration/DevOps, penetration testing, incident response.

---

### 🔴 Encoding Schemes

| Encoding | Formato | Ejemplo |
|---|---|---|
| **URL Encoding** | `%` + dos dígitos hexadecimales del ASCII | `%3d` = `=` · `%0a` = newline · `%20` = space |
| **HTML Encoding** | HTML entities | `&amp;` = `&` · `&lt;` = `<` · `&gt;` = `>` |
| **Unicode — 16-bit** | `%u` + codepoint hexadecimal | `%u2215` = `/` |
| **Unicode — UTF-8** | `%` + bytes hexadecimales | `%c2%a9` = `©` |
| **Base64** | Solo caracteres ASCII imprimibles | Usado para email attachments y user credentials |
| **Hex Encoding** | Valor hex de cada carácter | `Hello` = `48 65 6C 6C 6F` |

---

### Whitelisting vs. Blacklisting

| Aspecto | Whitelisting | Blacklisting |
|---|---|---|
| Qué especifica | Lista de entidades **permitidas** | Lista de entidades **no permitidas** |
| Enfoque | Seguridad proactiva | Threat-centric |
| Contra malware moderno | Más eficaz | **No puede detectar amenazas modernas**; requiere actualizaciones constantes |
| Protección contra | Ransomware y otros malware | Aplicaciones maliciosas conocidas |

**ManageEngine Application Control Plus:** herramienta que automatiza la colocación de aplicaciones en whitelists/blacklists basada en control rules; incorpora Endpoint Privilege Management (PoLP + zero trust).

---

### 🔴 Defensa contra ataques de inyección — Puntos clave por tipo

#### SQL Injection
- Usar **prepared statements, parameterized queries, o stored procedures**.
- Validar y sanitizar inputs pasados a la BBDD.
- Evitar SQL dinámico construido con input del usuario.
- Usar **ORM frameworks** para parametrizar queries automáticamente.
- Deshabilitar comandos como **xp_cmdshell**.
- Limitar longitud del input del usuario.
- Conectar a la BBDD con cuenta de **mínimos privilegios**.
- Usar mensajes de error personalizados (no verbose).
- Monitorizar tráfico de BBDD con **IDS y WAF**.
- Usar whitelist lógicamente en lugar de blacklist.

#### Command Injection
- Usar **librerías específicas del lenguaje** que no invoquen el intérprete del OS.
- Validar input; escapar caracteres peligrosos.
- Usar **safe API** que evite el uso del intérprete.
- Estructurar requests para que todos los parámetros sean tratados como datos.
- Usar **least privilege** para ejecutar comandos del OS.
- No ejecutar el servidor web como **root** ni acceder a BBDD como **DBADMIN**.
- Usar **Java sandbox** en entorno J2EE.
- Implementar Python como framework web en lugar de PHP.
- Usar whitelists para caracteres permitidos en input.

#### LDAP Injection
- Validación de tipo, patrón y dominio en todos los inputs.
- Sanitizar inputs; escapar caracteres especiales de LDAP: `*`, `(`, `)`, `\`, `&`, `|`.
- Evitar construir filtros LDAP por concatenación de strings.
- Usar filtro AND para reforzar restricciones en entradas similares.
- Usar **LDAPS** (LDAP over SSL) para cifrar comunicación.
- Configurar LDAP con **bind authentication**.
- Principio de mínimos privilegios para la cuenta de servicio LDAP.

#### File Injection (PHP específico)
- Deshabilitar **allow_url_fopen** y **allow_url_include** en `php.ini`.
- Deshabilitar **register_globals**.
- Implementar **chroot jail**.
- Whitelist de tipos de fichero permitidos y límites de tamaño.
- Almacenar ficheros subidos **fuera del web root**.
- Renombrar ficheros al subir; verificar MIME type.
- Usar **WAF** para monitorizar file injection.
- Comprobar wrappers PHP (`php://filter`, `php://zip`).

#### Server-Side JS Injection
- Evitar función **eval()** para parsear input del usuario.
- No usar `setTimeout()`, `setInterval()`, `Function()` con input del usuario.
- Usar **JSON.parse()** en lugar de eval() para JSON.
- Incluir `"use strict"` al inicio de las funciones.
- No serializar código.
- Ejecutar código del usuario en **sandboxed environment**.

#### Server-Side Template Injection
- No crear templates desde input del usuario.
- Ejecutar templates en **sandboxed environment** (ej. `SandboxedEnvironment` en Jinja2).
- Usar template engines seguros: **Jinja2 (Python)**, **Handlebars (JavaScript)**, **Thymeleaf (Java)**.
- Evitar construir template strings combinando variables y datos del usuario.

#### Log Injection
- Pasar códigos de log en lugar de mensajes.
- Usar formatos estructurados (**JSON o XML**) para logging.
- Usar checksums criptográficos para verificar integridad de logs.
- Usar librerías de logging establecidas: **Log4j (Java)**, **Winston (Node.js)**, **logging module (Python)**.

#### HTML Injection
- Validar todos los inputs; eliminar substrings HTML de texto del usuario.
- Habilitar el flag **HttpOnly** en cookies.
- Usar `.text()` de jQuery en lugar de `.html()`.
- Usar OWASP Java HTML Sanitizer o DOMPurify.

#### CRLF Injection
- Reemplazar `%0d` y `%0a` en datos URL-encoded; `\r` y `\n` en input estándar.
- Codificar datos que se pasen a HTTP headers.
- Deshabilitar headers innecesarios.
- Configurar **XSSUrlFilter** para prevenir CRLF injection.
- Usar Django (Python) o Spring (Java) que manejan CRLF automáticamente.

---

### 🔴 Countermeasures — OWASP categories clave

**XSS:**
- Validar todos los headers, cookies, query strings, form fields, hidden fields.
- Convertir caracteres no alfanuméricos en **HTML character entities** antes de mostrarlos.
- Usar **Content Security Policy (CSP)**.
- Escapar input/output; filtrar metacaracteres.
- Evitar **eval()** e inline JavaScript.
- Usar context-sensitive escaping: HTML encoding, JavaScript escaping, URL encoding, CSS escaping.

**Broken Access Control:**
- Implement **deny by default**.
- Invalidar session IDs en el servidor al hacer logout.
- Usar **RBAC** y **ABAC**.
- Aplicar security headers: **CSP** y **X-Frame-Options**.

**Cryptographic Failures:**
- **AES** para datos almacenados; **TLS con HSTS** para tráfico entrante.
- Cifrado asimétrico: **RSA-2048**; simétrico: **AES-256**.
- **TLS con Perfect Forward Secrecy (PFS)** para datos en tránsito.
- Hash de passwords: **bcrypt, scrypt, Argon2**.
- **NO usar** MD5, SHA-1, PKCS v1, PKCS v1.5 (deprecados).
- Almacenar claves con **HSMs** (Hardware Security Modules) o KMS.

**Insecure Design — principios de secure design:**
- **Least privilege** · **Defense in depth** · **Fail securely** · **Secure by default** · **Minimize attack surface**
- Herramientas de threat modeling: **Microsoft Threat Modeling Tool** o **OWASP Threat Dragon**.

**CSRF:**
- Generar **CSRF tokens únicos por sesión** e incluirlos en forms.
- Usar **SameSite attribute** en cookies.
- Usar atributo **HttpOnly** + header `X-Requested-With`.
- Estrategia de token por petición (per-request), no por sesión.
- Verificar headers **Referer y Origin**.

**Cookie/Session Poisoning:**
- No almacenar passwords en texto plano en cookies.
- Usar **SameSite attribute**.
- Flag **Secure** (solo HTTPS) + flag **HttpOnly** (no accesible via JS).
- Regenerar session IDs tras login y periódicamente.
- Firmar cookies con secret del servidor para verificar integridad.

**Clickjacking:**
- Header **X-Frame-Options** con opciones: **DENY**, **SAMEORIGIN**, **ALLOW-FROM URI**.
- **Nunca** usar métodos client-side (Framebusting/Framebreaking) — son bypasseables fácilmente.
- Usar **CSP** HTTP header.
- Usar **SameSite cookie attribute**.

**Directory Traversal:**
- Eliminar o codificar caracteres `..`, `/`, `\`.
- Normalizar file paths (canonical form).
- Usar **chroot jail** en sistemas Unix.
- Deshabilitar directory listings en el servidor web.
- No usar input del usuario para llamar APIs del filesystem.

**SSRF:**
- Whitelist de schemas URL permitidos (ej. solo `https://`).
- No enviar respuestas raw al cliente.
- Deshabilitar redirecciones HTTP automáticas.
- **Deny by default** para todo tráfico de intranet excepto el esencial.
- Bloquear requests que contengan IPs internas: `127.0.0.1`, `169.254.0.0/16`, `192.168.0.0/16`.

**Session Management:**
- Usar SSL para todas las partes autenticadas.
- Session IDs: largos, aleatorios, generados por plataforma segura.
- Limitar intentos de login; bloquear cuenta tras N intentos fallidos.
- Implementar **MFA**.
- Hash de passwords: **bcrypt, scrypt, Argon2**.
- Mensaje de error genérico: "**invalid username and/or password**" (no indicar cuál es incorrecto).
- Invalidar session tokens al logout en el servidor.

**Password Reset:**
- URLs de reset solo de un uso + tiempo de expiración corto.
- CAPTCHA para prevenir requests automatizados.
- Tokens **criptográficamente seguros y únicos** por cada reset.
- Confirmación adicional (código enviado a email/teléfono).

**Username Enumeration:**
- Mensaje genérico: "The username or password is not valid."
- Tiempo de respuesta uniforme para usernames válidos e inválidos.
- Rate limiting; geo limiting.
- CAPTCHA.
- Usar honeypots.

**Web Service Attacks:**
- Comparar la operación dentro de **SOAPAction y el SOAP body**.
- Deshabilitar WS-Addressing; si se requiere, whitelist de direcciones.
- Usar **digital signatures** para integridad de mensajes.
- Deshabilitar atributo SOAPAction cuando no esté en uso.
- Usar **TLS** para comunicación SOAP.
- Configurar **WSDL Access Control Permissions**.
- Deshabilitar protocolos de documentación (previene generación dinámica de WSDL).

**Watering Hole:**
- Monitorizar tráfico de red.
- Asegurar servidor DNS; implementar **DNSSEC**.
- Usar browser plugins que bloqueen redirecciones HTTP.
- Ejecutar el navegador en entorno virtual.
- Deshabilitar contenido de terceros (advertising services).

**JavaScript Hijacking:**
- Usar `.innerText` en lugar de `.innerHTML`.
- Evitar **eval()**.
- No escribir código de serialización.
- Usar **CORS** en lugar de JSONP.
- Prefix JSON responses con string que los haga JavaScript inválido pero JSON válido: `)]}',\n`.
- Habilitar **sub-resource integrity feature**.

---

### 🔴 WebSocket Security Best Practices

- Usar **wss://** (WebSocket Secure) en lugar de ws:// → cifra con TLS/SSL.
- Validar header **Origin** para aceptar solo dominios de confianza.
- Autenticación con **JWT** antes de establecer la conexión.
- **RBAC** para autorización sobre la conexión WebSocket.
- Límites de tamaño de mensajes (prevenir DoS).
- **Rate limiting** + timeouts de sesión para conexiones inactivas.
- Security headers: **CSP** y **X-Frame-Options**.
- Certificados del servidor válidos y emitidos por **CA de confianza**.

---

### 🔴 N-Stalker — Dato específico

N-Stalker incorpora "N-Stealth HTTP Security Scanner" con base de datos de **39,000 web attack signatures**.

---

## 2. Exam Traps ⚠️

⚠️ **[SAST = whitebox, DAST = blackbox]**
SAST es whitebox (arquitectura y código conocidos). DAST es blackbox (arquitectura desconocida). El examen puede invertirlos. Fuzz testing es también blackbox. El examen puede preguntar cuál modalidad incluye fuzzing → DAST (y también se puede hacer como técnica independiente de blackbox).

⚠️ **[Clickjacking: NUNCA usar Framebusting/Framebreaking]**
El libro especifica explícitamente "Never use client-side methods such as Framebusting or Framebreaking as they can be bypassed easily." La defensa correcta es **X-Frame-Options** header server-side o CSP. El examen puede presentar Framebusting como solución válida → incorrecto.

⚠️ **[X-Frame-Options: opciones exactas]**
Las tres opciones son: **DENY**, **SAMEORIGIN**, **ALLOW-FROM URI**. El examen puede presentar una variante con nombre incorrecto.

⚠️ **[CSRF: per-request tokens, no per-session]**
El libro recomienda explícitamente "per-request token assignment strategy, rather than per-session." El examen puede presentar tokens por sesión como la mejor práctica → es menos seguro.

⚠️ **[Cryptographic Failures: algoritmos de cifrado específicos]**
AES para datos almacenados. AES-256 para cifrado simétrico. RSA-2048 para asimétrico. TLS con HSTS para tráfico entrante. TLS con PFS para datos en tránsito. NO MD5, SHA-1, PKCS v1/v1.5.

⚠️ **[Username Enumeration: mensaje de error exacto]**
El mensaje correcto es **"The username or password is not valid"** (sin indicar cuál campo). El examen puede presentar mensajes como "invalid password" o "user not found" como correctos → revelan información.

⚠️ **[SOAPAction spoofing: contramedida específica]**
La contramedida es comparar la operación dentro de **SOAPAction y el SOAP body** y deshabilitar SOAPAction cuando no está en uso. El examen puede no mencionar esta comparación.

⚠️ **[RASP: dónde se sitúa]**
RASP se sitúa **dentro del código de la aplicación**, entre el servidor y la aplicación. No es un WAF externo. Puede detectar **zero-day attacks** en tiempo real sin intervención humana.

⚠️ **[Mutation-Based vs. Generation-Based fuzzing]**
Mutation-Based: parte de datos **existentes válidos** y los muta. Generation-Based: genera datos **desde cero** (predefinido por el modelo). El examen puede invertir cuál parte de datos existentes.

⚠️ **[N-Stalker: número de signatures]**
N-Stalker incorpora base de datos de **39,000 web attack signatures**. Dato numérico específico preguntable.

⚠️ **[WebSocket: wss:// vs. ws://]**
`wss://` = WebSocket Secure (cifrado con TLS/SSL). `ws://` = WebSocket sin cifrado. La diferencia es análoga a HTTPS vs HTTP. El examen puede preguntar cuál protocolo cifra los datos.

---

## 3. Nemotécnicos

**Testing types** → **"M-A-S-D-F"** con alias:
- **M**anual · **A**utomated · **S**AST=Whitebox · **D**AST=Blackbox · **F**uzz=Blackbox

**Fuzz Testing 6 pasos** → **"I-I-G-E-M-L"**:
**I**dentify target · **I**dentify inputs · **G**enerate fuzz data · **E**xecute · **M**onitor behavior · **L**og defects

**Fuzzing strategies (3)** → **"M-G-P"**: **M**utation (from existing) · **G**eneration (from scratch) · **P**rotocol (forged packets)

**Insecure Design secure principles (5)** → **"L-D-F-S-M"**:
**L**east privilege · **D**efense in depth · **F**ail securely · **S**ecure by default · **M**inimize attack surface

**Crypto defenses** → **"AES-RSA-TLS(PFS)-bcrypt"**:
- Stored data → AES (256) · Asymmetric → RSA-2048 · In transit → TLS+PFS · Passwords → bcrypt/scrypt/Argon2

**Clickjacking X-Frame-Options (3 opciones)** → **"D-S-A"**: **D**ENY · **S**AMEORIGIN · **A**LLOW-FROM URI

**WebSocket security** → **"wss:// + Origin + JWT + RBAC + Rate + Timeout + CSP"**

**CSRF defense core** → **"Token(per-request) + SameSite + HttpOnly + Referer/Origin"**

**Session defense core** → **"SSL + MFA + bcrypt + Invalidate-on-logout + Generic-error-message"**

---

## 4. Flashcards

**Q:** ¿Cuál es la diferencia entre SAST y DAST?
**A:** SAST (whitebox testing): el tester conoce arquitectura y código fuente; analiza código estático. DAST (blackbox testing): arquitectura desconocida; ejecuta código en ejecución para detectar issues en interfaces, sesiones, auth, injections.

---

**Q:** ¿Cuáles son los 6 pasos del fuzz testing?
**A:** Identify target → Identify inputs → Generate fuzzed data → Execute test → Monitor system behavior → Log defects.

---

**Q:** ¿Cuál es la diferencia entre Mutation-Based y Generation-Based fuzzing?
**A:** Mutation-Based: parte de muestras de datos existentes y las muta iterativamente. Generation-Based: genera datos completamente nuevos desde cero según un modelo predefinido.

---

**Q:** ¿Por qué nunca se debe usar Framebusting o Framebreaking contra clickjacking?
**A:** Porque son métodos client-side que pueden ser bypasseados fácilmente. La defensa correcta es el header server-side X-Frame-Options (DENY, SAMEORIGIN, ALLOW-FROM URI) o CSP.

---

**Q:** ¿Cuáles son las tres opciones del header X-Frame-Options para defenderse contra clickjacking?
**A:** DENY (nunca framing), SAMEORIGIN (solo mismo origen), ALLOW-FROM URI (solo URI especificada).

---

**Q:** ¿Qué algoritmos de cifrado recomienda el CEH para datos almacenados y en tránsito?
**A:** Almacenados: AES (AES-256 simétrico), RSA-2048 asimétrico. En tránsito: TLS con HSTS para tráfico entrante; TLS con Perfect Forward Secrecy (PFS) para datos en tránsito.

---

**Q:** ¿Qué mensaje de error debe mostrarse ante un intento de login fallido para evitar username enumeration?
**A:** "The username or password is not valid" — sin indicar cuál de los dos campos es incorrecto.

---

**Q:** ¿Qué es RASP y dónde se sitúa en la arquitectura?
**A:** Runtime Application Self Protection — tecnología que protege aplicaciones durante ejecución. Se sitúa dentro del código de la aplicación (entre server y aplicación). Detecta zero-day attacks en tiempo real sin intervención humana.

---

**Q:** ¿Qué estrategia de CSRF tokens es más segura: per-session o per-request?
**A:** Per-request — cada petición recibe un token único, reduciendo la ventana de tiempo para explotación. El libro recomienda explícitamente per-request en lugar de per-session.

---

**Q:** ¿Qué hace wfuzz con el flag --hc 404?
**A:** Oculta de los resultados las respuestas HTTP 404 (Not Found), filtrando rutas inválidas y mostrando solo las que devuelven otros códigos (rutas existentes).

---

**Q:** ¿Para qué se usa Prompt Fuzzer y qué aplicaciones objetivo tiene?
**A:** Es un tool AI-powered de fuzz testing diseñado específicamente para aplicaciones GenAI. Simula prompt injection attacks y usa LLMs para analizar respuestas y asignar security scores.

---

**Q:** ¿Cuál es la contramedida específica para SOAPAction spoofing?
**A:** Comparar la operación dentro del SOAPAction header y el SOAP body; deshabilitar el atributo SOAPAction cuando no esté en uso; deshabilitar WS-Addressing o usar whitelist de direcciones.

---

**Q:** ¿Cuántas web attack signatures tiene la base de datos de N-Stalker?
**A:** 39,000 web attack signatures (incorpora N-Stealth HTTP Security Scanner).

---

**Q:** ¿Qué protocolo WebSocket cifra los datos y con qué tecnología?
**A:** `wss://` (WebSocket Secure) — cifra datos en tránsito usando TLS/SSL. `ws://` es la versión sin cifrado.

---

**Q:** ¿Cuáles son los cinco principios de secure design según el CEH?
**A:** Least privilege, Defense in depth, Fail securely, Secure by default, Minimize attack surface.

---

**Q:** ¿Qué herramientas de threat modeling menciona el libro para Insecure Design?
**A:** Microsoft Threat Modeling Tool y OWASP Threat Dragon.

---

**Q:** ¿Cuáles son los dos comandos PHP en php.ini que deben deshabilitarse para prevenir file injection?
**A:** `allow_url_fopen` y `allow_url_include`.

---

**Q:** ¿Qué encoding debe reemplazarse específicamente para prevenir CRLF injection?
**A:** `%0d` (CR) y `%0a` (LF) en datos URL-encoded; `\r` y `\n` en input estándar.

---

**Q:** ¿Qué atributo de cookie previene que las cookies se envíen con cross-site requests?
**A:** El atributo **SameSite** — relevante para CSRF, cookie poisoning y clickjacking.

---

**Q:** ¿Qué es ManageEngine Application Control Plus y qué principios de seguridad implementa?
**A:** Herramienta que automatiza whitelisting/blacklisting basada en control rules; implementa Endpoint Privilege Management para establecer PoLP (Principle of Least Privilege) y zero trust.

---

## 5. Confusión frecuente

**SAST vs. DAST — whitebox vs. blackbox**
- SAST = Static = Whitebox: tester conoce el código y la arquitectura; analiza sin ejecutar.
- DAST = Dynamic = Blackbox: tester no conoce la arquitectura; ejecuta la app y la ataca.
- Fuzz testing es una técnica blackbox que se puede usar en el contexto de DAST.
- Criterio: "tester tiene acceso al código fuente" → SAST. "Tester prueba la app en ejecución sin conocer su arquitectura" → DAST.

---

**X-Frame-Options DENY vs. SAMEORIGIN**
- DENY: nunca puede cargarse en un frame/iframe, independientemente del origen.
- SAMEORIGIN: puede cargarse en frame solo si el origen es el mismo.
- ALLOW-FROM URI: solo puede cargarse en frame desde la URI especificada.
- Criterio: "bloquear completamente el framing" → DENY. "Permitir solo framing propio" → SAMEORIGIN.

---

**CSRF SameSite vs. HttpOnly — atributos de cookie distintos**
- SameSite: previene que la cookie se envíe con peticiones cross-site → defensa CSRF.
- HttpOnly: previene que la cookie sea accesible via JavaScript → defensa contra XSS que roba cookies.
- Criterio: "evitar que el cookie se use en peticiones desde otro origen" → SameSite. "Evitar que JavaScript acceda al cookie" → HttpOnly.

---

**Mutation-Based vs. Protocol-Based fuzzing**
- Mutation-Based: modifica datos válidos existentes → no requiere conocimiento del protocolo.
- Protocol-Based: forja paquetes del protocolo → **requiere conocimiento detallado del formato del protocolo**.
- Criterio: "necesita conocimiento del protocolo" → Protocol-Based. "Parte de datos válidos y los corrompe" → Mutation-Based.

---

**RASP vs. WAF**
- RASP: se sitúa **dentro** del código de la aplicación; detecta ataques desde dentro; monitorización en tiempo real; puede detectar zero-day attacks.
- WAF: se sitúa **externamente** entre cliente y servidor; filtra tráfico HTTP antes de llegar a la aplicación; basado en reglas y firmas.
- Criterio: "protección integrada en el código de la app que detecta zero-day" → RASP. "Filtrado de tráfico HTTP entre cliente y servidor" → WAF.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un equipo de seguridad está evaluando qué tipo de testing usar para una aplicación web en producción. No tienen acceso al código fuente y necesitan identificar vulnerabilidades en interfaces, sesiones y autenticación. ¿Qué tipo de testing deben usar?

A) SAST — Whitebox testing  
B) DAST — Blackbox testing  
C) Manual testing con acceso al repositorio  
D) RASP — Runtime monitoring  

**Respuesta correcta: B**
**DAST (Dynamic Application Security Testing)** es **blackbox testing** donde el tester no conoce la arquitectura interna; ejecuta la aplicación y la ataca para detectar vulnerabilidades en interfaces, requests/responses, sesiones, autenticación e injections. SAST requiere acceso al código fuente (whitebox). RASP es protección, no testing.

---

**P2.** Un desarrollador de seguridad implementa RASP en su aplicación web para protegerla contra ataques zero-day. ¿Dónde se sitúa RASP en la arquitectura y cuál es su ventaja clave?

A) Externamente entre cliente y servidor; filtra tráfico HTTP basado en firmas conocidas  
B) En el firewall perimetral; bloquea IPs maliciosas en tiempo real  
C) Dentro del código de la aplicación; detecta ataques en tiempo real incluyendo zero-day sin intervención humana  
D) En el proxy reverso; inspecciona headers HTTP antes de llegar al servidor  

**Respuesta correcta: C**
**RASP (Runtime Application Self Protection)** se sitúa **dentro del código de la aplicación** (entre servidor y aplicación). Su ventaja clave es la monitorización continua que detecta ataques en tiempo real incluyendo **zero-day attacks** sin intervención humana. Un WAF es externo (opción A). RASP conoce el contexto interno de la aplicación.

---

**P3.** Durante un pentest, el atacante intenta clickjacking embebiendo una página bancaria en un iframe de su sitio malicioso. El banco intenta defenderse implementando Framebusting JavaScript. ¿Es esta una defensa adecuada según el CEH?

A) Sí, Framebusting es la contramedida más efectiva contra clickjacking  
B) No, los métodos client-side como Framebusting son bypasseables fácilmente  
C) Sí, Framebusting combinado con Framebreaking es una defensa robusta  
D) No, pero solo falla si el atacante usa HTTP en lugar de HTTPS  

**Respuesta correcta: B**
El CEH especifica explícitamente: **"Never use client-side methods such as Framebusting or Framebreaking as they can be bypassed easily."** La defensa correcta es el header server-side **X-Frame-Options** (DENY, SAMEORIGIN, ALLOW-FROM URI) o Content Security Policy (CSP).

---

**P4.** Un ingeniero de seguridad configura el header X-Frame-Options para permitir que la aplicación solo sea embebida en iframes desde el mismo dominio corporativo, bloqueando cualquier otro origen. ¿Qué valor debe usar?

A) DENY  
B) SAMEORIGIN  
C) ALLOW-FROM URI  
D) BLOCK-FRAME  

**Respuesta correcta: B**
**SAMEORIGIN** permite que la página sea embebida en un frame **solo si el origen es el mismo** que la propia página. DENY bloquea completamente el framing de cualquier origen (incluido el propio). ALLOW-FROM URI permite especificar una URI externa concreta. BLOCK-FRAME no existe en la especificación.

---

**P5.** Una aplicación web almacena contraseñas de usuarios en la BBDD. El CISO requiere que el algoritmo de hashing sea resistente a ataques de fuerza bruta modernos. ¿Qué algoritmos recomienda el CEH para este propósito?

A) MD5 o SHA-1  
B) AES-256 o RSA-2048  
C) bcrypt, scrypt o Argon2  
D) SHA-256 o SHA-3  

**Respuesta correcta: C**
El CEH recomienda **bcrypt, scrypt o Argon2** para hashing de contraseñas. Son algoritmos diseñados específicamente para ser lentos (aumentando el coste computacional de ataques de fuerza bruta). MD5 y SHA-1 están deprecados. AES y RSA son cifrado, no hashing. SHA-256/SHA-3 son rápidos y no adecuados para passwords.

---

**P6.** Un equipo de QA está seleccionando una estrategia de fuzz testing para una API. Tienen un conjunto de requests válidos existentes y quieren generar variaciones de esos datos para descubrir vulnerabilidades. ¿Qué estrategia deben usar?

A) Protocol-Based Fuzzing  
B) Generation-Based Fuzzing  
C) Mutation-Based Fuzzing  
D) Model-Based Fuzzing  

**Respuesta correcta: C**
**Mutation-Based Fuzzing** parte de **muestras de datos válidas existentes** y las muta iterativamente para generar nuevos datos de prueba. Generation-Based genera datos completamente desde cero según un modelo. Protocol-Based forja paquetes del protocolo (requiere conocimiento detallado del protocolo).

---

**P7.** Un atacante explota una vulnerabilidad CSRF en una aplicación bancaria. El CISO implementa tokens CSRF. Un junior developer propone usar un token por sesión. ¿Qué recomienda el CEH al respecto?

A) Un token por sesión es suficiente y más eficiente  
B) Los tokens CSRF no son necesarios si se usa HTTPS  
C) La estrategia per-request (un token por petición) es más segura que per-session  
D) El atributo HttpOnly en cookies reemplaza la necesidad de tokens CSRF  

**Respuesta correcta: C**
El CEH especifica explícitamente: **"per-request token assignment strategy, rather than per-session."** Un token por petición tiene una ventana de tiempo de explotación mucho menor. HTTPS no previene CSRF. HttpOnly previene robo de cookies via XSS, pero no CSRF.

---

**P8.** Después de un login fallido, una aplicación muestra el mensaje "Incorrect password for user john@company.com". ¿Qué vulnerabilidad presenta este comportamiento y cuál es el mensaje correcto?

A) Broken Authentication — debe mostrar "Invalid credentials"  
B) Username Enumeration — debe mostrar "The username or password is not valid"  
C) Verbose Errors — debe mostrar "Error 403: Unauthorized"  
D) Information Disclosure — debe redirigir a la página de registro  

**Respuesta correcta: B**
El mensaje revela que el **username existe** (solo la contraseña es incorrecta), facilitando **Username Enumeration**. El mensaje correcto según el CEH es **"The username or password is not valid"** — sin indicar cuál de los dos campos es incorrecto. Este es un design flaw de autenticación explotable para enumeración.

---

**P9.** Un desarrollador PHP quiere prevenir ataques de file injection. El auditor le recomienda deshabilitar dos directivas en `php.ini`. ¿Cuáles son?

A) `register_globals` y `display_errors`  
B) `allow_url_fopen` y `allow_url_include`  
C) `safe_mode` y `open_basedir`  
D) `disable_functions` y `exec`  

**Respuesta correcta: B**
Las dos directivas clave para prevenir file injection PHP son **`allow_url_fopen`** (deshabilita la apertura de ficheros remotos) y **`allow_url_include`** (deshabilita el include de ficheros desde URLs remotas). Ambas, si están habilitadas, permiten incluir ficheros maliciosos remotos.

---

**P10.** Un CISO evalúa el cifrado para los datos en reposo de su aplicación. Necesita cifrado simétrico para datos almacenados y asimétrico para intercambio de claves. ¿Qué combinación recomienda el CEH?

A) AES-128 para datos almacenados + RSA-1024 para intercambio  
B) MD5 para datos almacenados + SHA-1 para intercambio  
C) AES-256 para datos almacenados + RSA-2048 para asimétrico  
D) 3DES para datos almacenados + ECDSA-256 para asimétrico  

**Respuesta correcta: C**
El CEH recomienda **AES-256** para cifrado simétrico de datos almacenados y **RSA-2048** para cifrado asimétrico. MD5 y SHA-1 están deprecados. RSA-1024 es insuficiente por las capacidades computacionales actuales. 3DES es legacy y no recomendado.

---

**P11.** Un analista de seguridad encuentra que una aplicación construye templates usando directamente el input del usuario: `template = "Hello " + user_input`. ¿Qué vulnerabilidad presenta y qué engine de templates seguro recomienda el CEH para Python?

A) SQL Injection — usar SQLAlchemy  
B) Server-Side Template Injection (SSTI) — usar Jinja2 con SandboxedEnvironment  
C) XSS Stored — usar DOMPurify  
D) SSRF — usar allowlist de URLs  

**Respuesta correcta: B**
Construir templates con input del usuario directamente es **Server-Side Template Injection (SSTI)**. El CEH recomienda para Python el uso de **Jinja2 con `SandboxedEnvironment`** para ejecutar templates en un entorno aislado. Para JavaScript: Handlebars. Para Java: Thymeleaf.

---

**P12.** N-Stalker es un web application security scanner que incorpora "N-Stealth HTTP Security Scanner". ¿Cuántas web attack signatures contiene su base de datos según el CEH?

A) 15.000 signatures  
B) 25.000 signatures  
C) 39.000 signatures  
D) 50.000 signatures  

**Respuesta correcta: C**
N-Stalker incorpora N-Stealth HTTP Security Scanner con una base de datos de **39.000 web attack signatures**. Este dato numérico específico aparece frecuentemente en preguntas de identificación de herramientas del examen CEH. Los otros valores son distractores.
