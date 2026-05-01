# M14_03_WebAppHackingMethodology.md
> Módulo 14 / Subapartado 3 — Web Application Hacking Methodology

---

## 1. Conceptos y definiciones

### 🔴 Fases de la metodología (13 fases en orden)

| # | Fase |
|---|---|
| 1 | Footprint web infrastructure |
| 2 | Analyze web applications |
| 3 | Bypass client-side controls |
| 4 | Attack authentication mechanisms |
| 5 | Attack authorization schemes |
| 6 | Attack access controls |
| 7 | Attack session management mechanisms |
| 8 | Perform injection attacks |
| 9 | Attack application logic flaws |
| 10 | Attack shared environments |
| 11 | Attack database connectivity |
| 12 | Attack web application clients |
| 13 | Attack web services |

---

### 🔴 Fase 1: Footprint Web Infrastructure — Tareas y herramientas

**Tareas principales:**

| Tarea | Técnicas / Herramientas |
|---|---|
| **Server discovery** | Whois lookup, DNS interrogation, port scanning |
| **Service discovery** | Nmap, NetScanTools Pro |
| **Server identification (banner grabbing)** | Telnet, Netcat, OpenSSL (para SSL/HTTPS) |
| **WAF and proxy detection** | WAFW00F, WhatWaf, Nmap (`http-waf-detect`) |
| **Hidden content discovery** | Web spidering (OWASP ZAP, Burp Suite, WebScarab) |
| **Load balancers detection** | `dig`, `host`, `lbd` (load balancing detector) |
| **Web app technologies detection** | Wappalyzer, BuiltWith |
| **WebSocket enumeration** | STEWS (`python3 STEWS-fingerprint.py -1 -k -u <URL>`) |

**Banner grabbing sobre SSL (OpenSSL):**
```bash
s_client -host <target> -port 443
GET / HTTP/1.0    # [Enter]
```
Puerto SSL por defecto: **443**. Telnet y Netcat solo funcionan sobre HTTP; para HTTPS se requiere OpenSSL.

**Detección de proxies:** comando `TRACE` en HTTP/1.1 → si hay proxy, éste añade headers al request al reenviarlo; el servidor devuelve el request modificado → el atacante compara ambas versiones.

**WAF detection:** WAFW00F detecta WAFs buscando: cookies, server cloaking, response codes, drop action, pre-built-in rules.

**Comandos para detectar load balancers:**
```bash
host <target domain>          # resolución a múltiples IPs → load balancer
dig <target domain>           # más detallado que host
lbd <target domain>           # analiza Server: y Date: headers
```
`lbd` detecta load balancing via DNS y/o HTTP analizando diffs entre respuestas del servidor.

---

### 🔴 Fase 1 — Puertos HTTP comunes (tabla examen)

| Puerto | Servicio |
|---|---|
| **80** | WWW standard (HTTP) |
| **81** | Alternate WWW |
| **88** | Kerberos |
| **443** | SSL (HTTPS) |
| **1433** | MSSQL Server |
| **1434** | MSSQL Monitor |
| **1527** | Oracle Net Services |
| **7001** | BEA WebLogic |
| **7002** | BEA WebLogic over SSL |
| **8005** | Apache Tomcat |
| **8000/8001** | Alternate web server / management |
| **9090** | Sun Java Web Server admin module |
| **10000** | Netscape Administrator interface |

---

### Fase 1 — Website Mirroring

Crea réplica local del sitio para análisis offline sin múltiples requests al servidor original.

**Herramientas y sintaxis:**
```bash
# wget
wget --mirror --convert-links --adjust-extension --page-requisites --no-parent http://certifiedhacker.com

# HTTrack
httrack https://certifiedhacker.com -O ~/Desktop/certifiedhacker mirror
```

| Flag wget | Función |
|---|---|
| `--mirror` | Descarga recursiva de todas las páginas |
| `--convert-links` | Convierte links en HTML para que apunten a ficheros locales |
| `--adjust-extension` | Añade extensiones correctas a ficheros HTML descargados |
| `--page-requisites` | Descarga imágenes y stylesheets necesarios |
| `--no-parent` | No sube al directorio padre |

---

### Fase 2: Analyze Web Applications

Información obtenible al analizar una aplicación web:
- Software y versión; OS; subdirectorios y parámetros; filename/path/DB field names
- Scripting platform (extensiones: `.php`, `.asp`, `.jsp`)
- Tecnologías usadas (`.NET`, `J2EE`, `PHP` en URLs)
- Contact details y CMS details (útiles para social engineering)
- SSL certificate details; cookies; error messages; publicly accessible files

**Técnicas de análisis:**

**Gather wordlist — CeWL:**
```bash
cewl --help
cewl https://www.certifiedhacker.com     # devuelve lista de palabras únicas de la URL
```

**Metadata extraction — ExifTool:**
Extrae: usernames, OS, email addresses, software usado (versión y tipo), servidores, fechas de creación/modificación, autores de documentos.

**Identificar server-side technologies:**
- Session tokens delatan la tecnología:
  - `JSESSIONID` → Java
  - `ASPSESSIONID` → IIS server
  - `ASP.NET_SessionId` → ASP.NET
  - `PHPSESSID` → PHP
- Herramientas: **httprint**, **WhatWeb** (`whatweb -v <URL> | tee output.txt`)
- WhatWeb tiene **más de 1800 plugins**; identifica también versiones, emails, account IDs, SQL errors.

**Identificar ficheros y directorios:**
```bash
gobuster -u <URL> -w common.txt                    # enumera directorios
gobuster -u <URL> -w common.txt -s 200             # filtra por status code 200
nmap -sV --script=http-enum <target>               # NSE script
dirb http://www.moviescope.com                     # dirb
```

**Vulnerability scanning con AI:**
```bash
nikto -h www.moviescope.com
nmap --script vuln www.moviescope.com
```

---

### Fase 3: Bypass Client-Side Controls

**Atacar hidden form fields:**
1. Identificar aplicación vulnerable
2. Guardar el HTML de la página
3. Localizar el hidden field
4. Modificar el valor del campo precio
5. Guardar y recargar en el navegador
6. Click en Buy → request enviado al servidor con precio modificado

Alternativa con Burp Suite: trapear el request y modificar el campo precio. Se pueden introducir **valores negativos** para forzar un reembolso.

**Browser Extensions:**
- **Intercepting:** capturar y modificar request/response (limitación: obfuscación/cifrado).
- **Decompiling:** descompilar el bytecode del componente → permite modificar datos independientemente de obfuscación o cifrado (ventaja principal).

**Google Chrome Sync Attack:** los atacantes añaden extensiones maliciosas falsas que parecen legítimas → exfiltran datos sincronizados: contraseñas, historial, bookmarks, autofill, configuración, datos de geolocalización, sensores.

---

### 🔴 Fase 4: Attack Authentication Mechanism

#### Username Enumeration — Métodos
- **Verbose failure messages:** el mensaje de error indica cuál campo (username o password) es incorrecto → el atacante sabe si el username existe.
  - Ejemplos: "Account not found" (username incorrecto) vs "Incorrect password" (username válido).
- **Predictable usernames:** secuencias como `user101`, `user102` permiten enumerar.

#### 🔴 Design Flaws en Authentication
| Flaw | Descripción |
|---|---|
| Bad Passwords | Sin control mínimo de complejidad; acepta blancos, nombres, palabras de diccionario |
| Brute-Forcible Login | Sin lockout tras intentos fallidos |
| Verbose Failure Messages | Indica qué campo es incorrecto |
| Insecure Transmission | HTTP no cifrado → susceptible a MITM; incluso HTTPS puede ser inseguro si las credenciales van en query string o cookies |
| Password Reset Mechanism | Genera verbose error sobre si el username es válido; permite guessing del campo "Existing password" sin restricciones |
| Forgotten Password | Permite brute-force de respuestas de recuperación; no limita intentos |
| "Remember Me" | Persistent cookies simples (`RememberUser=jason`) fácilmente predecibles |
| User Impersonation | Puede llevar a vertical privilege escalation si la lógica de impersonation tiene fallos |
| Improper Validation | Trunca contraseñas, chequea solo los primeros caracteres, es case-insensitive |
| Predictable Usernames/Passwords | Generación automática por secuencia; initial passwords distribuidas por fuentes conocidas |
| Insecure Distribution of Credentials | Activation URLs enviadas por email pueden revelar patrón de tokens |

#### Implementation Flaws en Authentication
- **Fail-Open Login:** excepción (null pointer) en `db.getUser()` que bypassa la validación → la función retorna "Login successful!" en el bloque catch.
- **Flaws in Multistage Login:** 3 stages (username+password → challenge → token físico); cada stage puede tener logic defects.
- **Insecure Storage of Credentials:** contraseñas en texto plano o cifrado débil en BBDD.

#### Password Attacks
- **Password Functionality Exploits:** explotar password change, password recovery, "Remember Me".
- **Brute-force:** Burp Suite Intruder; password lists (birthdate, nickname, phone…); password dictionaries.
- **Password Reset Poisoning:** el atacante modifica el header `Host` en el request de reset → el link de reset generado apunta a `badhost.com` → cuando la víctima hace click, el atacante captura el token.

**Pasos del Password Reset Poisoning:**
1. Atacante obtiene email de la víctima (OSINT, social engineering)
2. Envía request de reset con Host header manipulado (`Host: badhost.com`)
3. El link enviado a la víctima contiene la URL del servidor del atacante
4. Víctima hace click → atacante captura el token → clona la aplicación o actúa como proxy

#### Cookie Exploitation
| Técnica | Mecanismo |
|---|---|
| **Cookie Poisoning** | Modifica contenido del cookie para acceso no autorizado o identity theft |
| **Cookie Sniffing** | Captura el cookie con session ID de la víctima → bypassa autenticación |
| **Cookie Replay** | Reutiliza el session/cookie de un usuario mientras está logado; deja de funcionar cuando el usuario cierra sesión |

**Cookie poisoning pasos:** robar cookie (script injection/eavesdropping) → replay del cookie con valores alterados → Burp Suite o OWASP ZAP para trapear cookies.

#### Bypass Authentication
**Bypass SAML-based SSO:**
- SAML = Security Assertion Markup Language, XML-based, entre IdP y SP.
- Los mensajes SAML se cifran en **Base64** (fácilmente reversible).
- Campos vulnerables: **signature** y **assertion**.
- Herramienta: **SAML Raider** (extensión de Burp Suite) — modifica mensajes SAML y gestiona certificados X.509.

**Bypass Rate Limit — Técnicas:**
- Explorar endpoints alternativos: `/api/v3/sign-up`, `/SignUp`, `/api/v1/sign-up`…
- Incluir **blank bytes**: `%00`, `%0d%0a`, `%0d`, `%0a`, `%09`, `%0C`, `%20`
- Alterar IP origin mediante headers: `X-Originating-IP`, `X-Forwarded-For`, `X-Remote-IP`, `X-Remote-Addr`, `X-Client-IP`, `X-Host`, `X-Forwarded-Host`

**Bypass MFA:**
- **HTTP Response Body Manipulation:** cambiar `"success":false` por `"success":true` con el mismo OTP incorrecto.
- **Status Code Manipulation:** cambiar código 403 "Invalid Token" por 200 OK con `"success":true`.

---

### 🔴 Fase 5-6: Attack Authorization Schemes & Access Controls

**Fuentes para authorization attacks:** URIs, parameter tampering, POST data, HTTP headers, query strings, cookies, hidden tags.

**HTTP Authorization header:** contiene username + password codificados en **Base-64**. El atacante puede comprometer el header enviando dos HTTP requests en el mismo header → el proxy ejecuta el primero, el sistema objetivo ejecuta el segundo.

**Parameter-Based Access Control:** parámetros que determinan el acceso (cookies, query strings); difieren entre user normal y administrador → identificar parámetros de admin y replicarlos.

**Referer-Based Access Control:** el HTTP referer se considera inseguro; el atacante puede manipularlo a cualquier valor.

**Location-Based Access Control:** bypasseable mediante web proxy, VPN, dispositivo móvil con data roaming, manipulación directa de mecanismos client-side.

**Access Controls Attack Methods:**
- Usar diferentes cuentas de usuario (Burp Suite para comparar dos contextos de usuario)
- Atacar multistage processes: capturar y probar cada request; cambiar session token por uno de usuario menos privilegiado
- Atacar static resources: solicitar URLs directamente
- Modificar HTTP methods: GET, POST, PUT, DELETE, TRACE, OPTIONS

---

### 🔴 Fase 7: Attack Session Management

**Session Management Attack — 2 stages:**
1. **Session token generation:** prediction + tampering
2. **Exploitation of session token handling:** MITM, session hijacking, session replay

**Session Token Prediction:**
- Patrón secuencial → predecir siguiente token cambiando parámetros como fecha o username.
- Ejemplo hex encoding: `user=jason;app=admin;date=08/01/2020` → cambiar la fecha → nuevo token válido.
- Pasos: sniffar tokens → analizar estructura/encoding/patrón → predecir/adivinar tokens recientes → brute-force si necesario.

**Session Token Sniffing:**
- Herramientas: **Wireshark** o **Burp Suite** (intercepting proxy).
- Si cookies HTTP se usan como mecanismo y el flag `secure` no está activo → replay posible.

**Manipulating WebSocket Traffic con Burp Suite:** Proxy → WebSockets history → Send to Repeater → clonar conexión → manipular raw request → Send.

**JSESSIONID = "user01"** → variable predecible (username). El atacante cambia a "user02" → acceso no autorizado.

---

### 🔴 Fase 8: Perform Injection Attacks

**Tipos de injection attacks:**

| Tipo | Vector |
|---|---|
| Web Scripts Injection | Input en código ejecutado dinámicamente |
| OS Commands Injection | Input en comandos a nivel de sistema |
| SMTP Injection | Comandos SMTP arbitrarios → spam masivo |
| SQL Injection | Queries maliciosas en campos de entrada |
| LDAP Injection | Filtros LDAP para acceso a directorios |
| XPath Injection | Strings maliciosos en queries XPath |
| Buffer Overflow | Datos masivos más allá de la capacidad del campo |
| File Injection | Explotación de "dynamic file include" |
| Canonicalization | `../` para acceder a directorios restringidos |

**Local File Inclusion (LFI):**
```
# Básico
http://xyz.com/page=../../../../../../etc/passwd

# Evadir .php añadido
http://xyz.com/page=../../../../../../etc/passwd%00    # null byte
http://xyz.com/page=../../../../../../etc/passwd?      # question mark

# Bypass .php execution con PHP filter
http://xyz.com/index.php?page=php://filter/convert.base64-encode/resource=index
# Luego decodificar: base64 -d savefile.php
```
LFI frecuente en PHP por uso extensivo de `require()` con `$_GET['page']` sin validación.

---

### Fase 9: Attack Application Logic Flaws

**Forced browsing:** saltarse una fase de un proceso de varios pasos (ej. ir de step 2 directamente a step 4 en checkout para evitar pago). Herramienta: Burp Suite para controlar requests.

---

### Fase 10: Attack Shared Environments

Múltiples clientes en la misma infraestructura → ataques entre aplicaciones y contra el mecanismo de acceso remoto del provider.

Script Perl explotable en shared environments:
```perl
my $command = param("cmmd"); $command=`$command`;
```
Permite ejecutar comandos OS remotamente (ej. `whoami`) a través de Internet.

---

### 🔴 Fase 11: Attack Database Connectivity

**Tipos de data connectivity attacks:**

| Tipo | Mecanismo |
|---|---|
| **Connection String Injection** | Inyección de parámetros con `;` en strings dinámicos → override de parámetros (ej. `Encryption=off`) |
| **CSPP (Connection String Parameter Pollution)** | Sobrescribe valores de parámetros en el connection string |
| **Connection Pool DoS** | Construye queries SQL masivas → ejecuta múltiples simultáneamente → agota el pool de conexiones |

**Connection Pool DoS en ASP.NET:** max connections por defecto = **100**, timeout = **30 segundos**. El atacante ejecuta 100 queries de 30+ segundos dentro de 30 segundos → nadie más puede usar la BBDD.

**CSPP — Hash Stealing:**
```
User_Value: ; Data Source = Rogue_Server
Password_Value: ; Integrated Security = true
```
→ La app se conecta al servidor falso → el sniffer del servidor falso captura credenciales Windows.

**CSPP — Port Scanning:**
```
User_Value: ; Data Source = Target_Server, Target_Port
Password_Value: ; Integrated Security = true
```
→ La app intenta conectar al puerto → diferentes mensajes de error revelan si el puerto está abierto.

**CSPP — Hijacking Web Credentials:**
Sobreescribe `IntegratedSecurity=true` → la app se conecta a la BBDD usando la cuenta del sistema (no credenciales del usuario) → acceso con privilegios del sistema.

---

### Fase 12: Attack Web Application Clients

Métodos principales de ataque a clientes web:

| Método | Descripción |
|---|---|
| XSS | Inyección de scripts que se ejecutan en el navegador de la víctima |
| HTTP Header Injection | División de respuesta HTTP → defacement, cache poisoning, XSS |
| Request Forgery | CSRF — explota confianza del sitio en el navegador del usuario |
| Privacy Attacks | Tracking mediante estado persistente del navegador filtrado |
| Redirection Attacks | Links que aparentan ser legítimos pero redirigen a sitio malicioso |
| Frame Injection | Inyección de código en frames cuando no se valida el input |
| Session Fixation | Atacante autentica con session ID conocido → hijackea sesión del usuario |
| ActiveX Attacks | Luring via email/link → explotación de vulnerabilidades de ejecución remota |

---

### 🔴 Fase 13: Attack Web Services

#### WSDL Probing
1. Trapear documento WSDL del tráfico del web service
2. Analizar: propósito de la app, entry points, tipos de mensaje
3. Crear set de requests válidos con operaciones y schemas XML
4. Incluir contenido malicioso en requests SOAP → analizar errores

#### SOAP Injection
Inyección de caracteres especiales (comillas simples, dobles, semicolons) en campos de input → bypassa autenticación del web service → acceso a BBDD backend. Funciona como SQL injection.

#### SOAPAction Spoofing
El atacante modifica el header SOAPAction (que indica la operación a ejecutar) mientras el SOAP body contiene una operación diferente → el gateway valida el body (operación legítima) pero el servidor ejecuta la operación del header (maliciosa, ej. `deleteAllUsers`).

#### WS-Address Spoofing
El atacante modifica los headers `<ReplyTo>` y `<FaultTo>` de WS-Address para redirigir respuestas a un endpoint bajo su control → también puede generar tráfico masivo → DoS. Herramienta: **WS-Attacker**.

#### XML Injection
Inyección de datos y tags XML en campos de input → manipula schemas XML o popula BBDD XML con entradas falsas. Usos: bypass de autorización, escalada de privilegios, DoS.

#### Web Services Parsing Attacks
- **Recursive Payloads:** documento SOAP gramaticalmente correcto con loops infinitos → agota CPU y XML parser.
- **Oversize Payloads:** payload excesivamente grande → consume todos los recursos del sistema.

**Herramientas web service attacks:** SoapUI (soporta SOAP, REST, HTTP, JMS, AMF, JDBC) · XMLSpy · WS-Attacker.

---

## 2. Exam Traps ⚠️

⚠️ **[13 fases de la metodología — orden exacto]**
Las fases 5 y 6 son "Attack Authorization Schemes" y "Attack Access Controls" (en ese orden), no al revés. La fase 4 es Authentication ANTES que Authorization. El examen puede reordenar fases adyacentes o incluir una fase inexistente.

⚠️ **[OpenSSL para banner grabbing SSL: puerto y comando exactos]**
Puerto SSL = **443**. Comando: `s_client -host <target> -port 443`. Luego `GET / HTTP/1.0`. Telnet y Netcat NO pueden hacer banner grabbing sobre SSL. El examen puede presentar Netcat para HTTPS → incorrecto.

⚠️ **[lbd: qué analiza]**
lbd detecta load balancing via **DNS y/o HTTP** analizando los headers `Server:` y `Date:` y diffs entre respuestas. No es un port scanner ni un vulnerability scanner.

⚠️ **[Session tokens: lenguaje → token]**
JSESSIONID = Java. ASPSESSIONID = IIS. ASP.NET_SessionId = ASP.NET. PHPSESSID = PHP. El examen puede intercambiar los mapeos. Regla: JSP/Java → J; ASP → A; PHP → P.

⚠️ **[WhatWeb: número de plugins]**
WhatWeb tiene **más de 1800 plugins**. Dato numérico específico que aparece en preguntas de identificación de herramienta.

⚠️ **[Password Reset Poisoning: el header manipulado]**
El header manipulado es **Host**, no Referer ni X-Forwarded-For. El atacante sobreescribe el valor esperado (`certifiedhacker.com`) por su servidor (`badhost.com`). El token de reset apunta al servidor del atacante.

⚠️ **[SAML: campos vulnerables y encoding]**
Los mensajes SAML usan **Base64** (fácilmente reversible). Los campos vulnerables son **signature** y **assertion**. SAML Raider es extensión de **Burp Suite**, no herramienta independiente.

⚠️ **[Bypass MFA: body manipulation vs. status code]**
Son dos técnicas distintas. Body manipulation: cambiar `success:false` → `success:true`. Status code: cambiar 403 → 200 OK. El examen puede presentar solo una como "bypass MFA" completo.

⚠️ **[Connection Pool DoS: valores exactos ASP.NET]**
Max connections por defecto = **100**; timeout = **30 segundos**. El ataque: 100 queries de 30+ segundos en 30 segundos. El examen puede usar valores diferentes como distractores.

⚠️ **[SOAPAction Spoofing: qué valida el gateway y qué ejecuta el servidor]**
El gateway valida el **SOAP body** (operación legítima). El servidor web service ejecuta la operación del **SOAPAction header** (maliciosa). La discrepancia entre header y body es el mecanismo del ataque.

⚠️ **[LFI: null byte %00 para evadir .php]**
`%00` (null byte) al final del string evade la extensión `.php` añadida automáticamente. También funciona `?` al final. El PHP filter (`php://filter/convert.base64-encode/resource=index`) evita la ejecución directa del PHP.

⚠️ **[lbd vs. dig vs. host para detectar load balancers]**
`host` = resolución básica (una o varias IPs). `dig` = más detallado que host. `lbd` = análisis específico de load balancing via headers y DNS diffs. El examen puede presentar `dig` como la herramienta de load balancing → es válida pero `lbd` es la específica.

---

## 3. Nemotécnicos

**13 fases de la metodología** → **"F-A-BC-Auth-Authz-AC-SM-Inj-Logic-Shared-DB-Client-WS"**:
Foot · Analyze · Bypass Client · Auth · Authz · Access · Session · Injection · Logic · Shared · DB · Client · WebServices

**Puertos web clave** → **"80-443-1433-7001-8005"**:
HTTP · HTTPS · MSSQL · BEA WebLogic · Apache Tomcat

**Session tokens por tecnología** → **"J-A-AN-P"**:
**J**SESSIONID=Java · **A**SPSESSIONID=IIS · **A**SP.**N**ET_SessionId=ASP.NET · **P**HPSESSID=PHP

**wget mirror flags** → **"M-C-A-P-N"**: **M**irror · **C**onvert-links · **A**djust-extension · **P**age-requisites · **N**o-parent

**Design flaws authentication (8)** → **"B-B-V-I-Rst-Frgt-Rem-Imp"**:
**B**ad passwords · **B**rute-forcible · **V**erbose messages · **I**nsecure transmission · **R**e**s**e**t** mechanism · **F**orgotten password · **R**emember me · **I**mpersonation

**CSPP attacks (3)** → **"H-P-HJ"**: **H**ash stealing · **P**ort scanning · **H**i**j**acking credentials

**Web Service Attacks (5)** → **"W-SOAP-SOAPAction-WSAddr-XML-Parsing"**:
WSDL Probing · SOAP Injection · SOAPAction Spoofing · WS-Address Spoofing · XML Injection · Parsing Attacks

---

## 4. Flashcards

**Q:** ¿Cuáles son las 13 fases de la metodología de hacking de aplicaciones web en orden?
**A:** Footprint Infrastructure → Analyze Web App → Bypass Client-Side Controls → Attack Authentication → Attack Authorization → Attack Access Controls → Attack Session Management → Perform Injection Attacks → Attack Logic Flaws → Attack Shared Environments → Attack DB Connectivity → Attack Web App Clients → Attack Web Services.

---

**Q:** ¿Qué herramienta y comando se usa para banner grabbing sobre SSL/HTTPS?
**A:** OpenSSL: `s_client -host <target> -port 443`, luego `GET / HTTP/1.0`. Puerto por defecto: 443.

---

**Q:** ¿Qué detecta WAFW00F y qué elementos analiza?
**A:** Detecta y fingerprinta WAFs buscando: cookies, server cloaking, response codes, drop action y pre-built-in rules.

---

**Q:** ¿Cómo se detecta si un servidor está detrás de un proxy usando HTTP?
**A:** Usando el comando TRACE en HTTP/1.1 — el proxy añade headers al request al reenviarlo; el atacante compara el request original con el que devuelve el servidor para identificar las modificaciones del proxy.

---

**Q:** ¿Qué tool es específica para detectar load balancers y qué analiza?
**A:** `lbd` (load balancing detector) — detecta load balancing via DNS y/o HTTP analizando los headers `Server:` y `Date:` y las diferencias entre respuestas del servidor.

---

**Q:** ¿Qué session token indica que la aplicación usa Java?
**A:** `JSESSIONID`. (ASPSESSIONID = IIS; ASP.NET_SessionId = ASP.NET; PHPSESSID = PHP)

---

**Q:** ¿Cuántos plugins tiene WhatWeb?
**A:** Más de 1800 plugins. Identifica CMS, plataformas de blogging, librerías JS, web servers, dispositivos embebidos, versiones, emails, account IDs, SQL errors.

---

**Q:** ¿Cuáles son los flags de wget para mirroring completo de un sitio web?
**A:** `--mirror --convert-links --adjust-extension --page-requisites --no-parent`

---

**Q:** ¿Cuál es el mecanismo del Password Reset Poisoning?
**A:** El atacante modifica el header `Host` en el request de reset de contraseña → el link generado apunta al servidor del atacante → cuando la víctima hace click, el atacante captura el token de reset.

---

**Q:** ¿Qué encoding usan los mensajes SAML y cuáles son sus dos campos vulnerables?
**A:** Base64. Campos vulnerables: signature y assertion.

---

**Q:** ¿Cómo se bypassa MFA mediante HTTP Response Body Manipulation?
**A:** Se cambia el parámetro `"success":false` a `"success":true` en la respuesta con el mismo OTP incorrecto.

---

**Q:** ¿Cuáles son los valores por defecto de max connections y timeout del connection pool en ASP.NET?
**A:** Max connections: 100. Timeout: 30 segundos.

---

**Q:** ¿Cuál es el mecanismo del SOAPAction Spoofing?
**A:** El atacante modifica el header SOAPAction (a una operación maliciosa, ej. `deleteAllUsers`) mientras el SOAP body contiene una operación legítima → el gateway valida el body y permite el paso → el servidor ejecuta la operación del header.

---

**Q:** ¿Cómo se evita que PHP ejecute un fichero en LFI usando el PHP filter?
**A:** `php://filter/convert.base64-encode/resource=<fichero>` → convierte el PHP a Base64 sin ejecutarlo; luego se decodifica con `base64 -d`.

---

**Q:** ¿Qué hace CeWL y cuál es su uso en web application hacking?
**A:** Genera una lista de palabras únicas presentes en la URL objetivo. Se usa para crear wordlists para ataques de brute-force.

---

**Q:** ¿Cuáles son las tres técnicas de CSPP y qué hace cada una?
**A:** Hash Stealing (redirige al servidor falso para capturar credenciales Windows), Port Scanning (conecta a diferentes puertos y analiza mensajes de error), Hijacking Web Credentials (sobreescribe IntegratedSecurity=true para conectar con la cuenta del sistema).

---

**Q:** ¿Qué herramienta se usa para WebSocket enumeration y cuál es la sintaxis?
**A:** STEWS: `python3 STEWS-fingerprint.py -1 -k -u <Target URL>`

---

**Q:** ¿Cuáles son las dos técnicas para atacar browser extensions?
**A:** Intercepting Traffic (capturar y modificar request/response — limitado por obfuscación/cifrado) y Decompiling (descompilar bytecode — permite modificar datos independientemente de cifrado u obfuscación).

---

**Q:** ¿Cuáles son los headers HTTP que el atacante puede modificar para bypass de rate limiting por IP?
**A:** X-Originating-IP, X-Forwarded-For, X-Remote-IP, X-Remote-Addr, X-Client-IP, X-Host, X-Forwarded-Host.

---

**Q:** ¿Qué es un Recursive Payload en web service parsing attacks?
**A:** Documento SOAP gramaticalmente correcto que contiene loops de procesamiento infinitos → agota el XML parser y los recursos de CPU del servidor.

---

## 5. Confusión frecuente

**Intercepting vs. Decompiling Browser Extensions**
- Intercepting: captura y modifica tráfico en tránsito. Limitación: si los datos están obfuscados o cifrados, puede ser ineficaz.
- Decompiling: analiza el bytecode del componente offline. Ventaja: funciona independientemente de obfuscación o cifrado porque se ve el código fuente.
- Criterio: "modificar datos en tránsito" → intercepting. "Análisis del código del componente, supera cifrado/obfuscación" → decompiling.

---

**Session Hijacking vs. Session Replay vs. Session Fixation**
- Session Hijacking: roba el session ID de un usuario de un sitio confiable para actividades maliciosas.
- Session Replay: obtiene el session ID y lo reutiliza para acceder a la cuenta (mientras el usuario esté logado).
- Session Fixation: el atacante establece un session ID conocido de antemano → trick al usuario para que se autentique con ese ID → hijackea la sesión.
- Criterio: "robo del ID activo" → hijacking. "Reutilización del ID obtenido" → replay. "ID predefinido por el atacante antes de la autenticación" → fixation.

---

**SOAP Injection vs. SOAPAction Spoofing**
- SOAP Injection: inyecta caracteres especiales en campos de input de mensajes SOAP → manipula queries al backend (como SQL injection en web services).
- SOAPAction Spoofing: el header SOAPAction declara una operación diferente a la del SOAP body → el gateway valida el body pero el servidor ejecuta la operación del header.
- Criterio: "inyección de caracteres en el body del mensaje SOAP" → SOAP Injection. "Discrepancia entre header SOAPAction y body" → SOAPAction Spoofing.

---

**Connection String Injection vs. CSPP**
- Connection String Injection: inyección básica de parámetros usando `;` en strings construidos dinámicamente → añade o modifica parámetros (ej. `Encryption=off`).
- CSPP (Connection String Parameter Pollution): técnica más específica que sobreescribe parámetros existentes del connection string usando el algoritmo "last one wins" para comprometer credenciales, hacer port scanning o hijackear credenciales.
- Criterio: "añadir un parámetro nuevo como Encryption=off" → Connection String Injection. "Sobreescribir DataSource o IntegratedSecurity para objetivos específicos" → CSPP.

---

**Authorization vs. Access Control**
- Authorization (Fase 5): mecanismo que determina qué puede hacer un usuario autenticado; controla el acceso a recursos o funcionalidades específicas.
- Access Control (Fase 6): implementación lógica basada en autenticación y gestión de sesión; controla datos individuales, niveles de acceso por roles, funciones de admin.
- Criterio en el examen: ambos son adyacentes; "escalada de privilegios mediante manipulación de parámetros de usuario/rol" → authorization. "Control de acceso a subconjuntos de datos por roles (empleados, managers, CEO)" → access control.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un pentester ejecuta el comando `lbd certifiedhacker.com` y observa respuestas diferentes en los headers `Server:` y `Date:` entre peticiones consecutivas. ¿Qué está detectando y mediante qué técnica?

A) Detección de WAF mediante análisis de response codes  
B) Detección de load balancer mediante análisis de headers HTTP y DNS  
C) Detección de proxy mediante comando TRACE  
D) Detección de versión del servidor mediante banner grabbing  

**Respuesta correcta: B**
`lbd` detecta **load balancers** analizando diferencias en headers `Server:` y `Date:` entre respuestas, además de via DNS (múltiples IPs para el mismo dominio). WAFW00F detecta WAFs. El comando TRACE detecta proxies. Nmap/Telnet/OpenSSL hacen banner grabbing.

---

**P2.** Un atacante intercepta un request de password reset, modifica el header `Host: certifiedhacker.com` por `Host: malicious.com` y lo reenvía. La víctima recibe el email de reset, hace click en el link, y el atacante captura el token. ¿Qué tipo de ataque es este?

A) Cookie Poisoning  
B) CSRF  
C) Password Reset Poisoning  
D) Session Fixation  

**Respuesta correcta: C**
Este es un **Password Reset Poisoning** — el atacante modifica el header **Host** en el request de reset para que el link generado apunte a su servidor malicioso. La víctima hace click sin sospechar (el email parece legítimo) y el atacante captura el token de reset.

---

**P3.** Un pentester usa SAML Raider durante un pentest de SSO y quiere identificar los dos campos más vulnerables a manipulación en los mensajes SAML interceptados. ¿Cuáles son?

A) username y password  
B) token y session_id  
C) signature y assertion  
D) redirect_uri y client_id  

**Respuesta correcta: C**
Los dos campos vulnerables en mensajes SAML son **signature** y **assertion**. Los mensajes SAML usan Base64 (fácilmente reversible). SAML Raider es una extensión de Burp Suite para modificar mensajes SAML y gestionar certificados X.509.

---

**P4.** Durante una evaluación, el equipo red intenta bypassar la autenticación multifactor interceptando la respuesta del servidor tras introducir un OTP incorrecto. Cambian `"success":false` a `"success":true` en el response body. ¿Qué técnica de bypass de MFA están usando?

A) Status Code Manipulation  
B) HTTP Response Body Manipulation  
C) Session Fixation  
D) Cookie Replay  

**Respuesta correcta: B**
La técnica es **HTTP Response Body Manipulation** — el atacante modifica el parámetro `success` en el body de la respuesta con el mismo OTP incorrecto. Status Code Manipulation es diferente: cambia el código HTTP de 403 a 200 OK directamente.

---

**P5.** Un pentester analiza una aplicación ASP.NET y quiere explotar el connection pool mediante queries masivas. Sabe que ASP.NET tiene valores por defecto para max connections y timeout. ¿Qué valores son correctos y qué número de queries debe ejecutar en qué tiempo?

A) Max 50 conexiones, timeout 60 segundos; ejecutar 50 queries de 60+ segundos  
B) Max 100 conexiones, timeout 30 segundos; ejecutar 100 queries de 30+ segundos en 30 segundos  
C) Max 200 conexiones, timeout 45 segundos; ejecutar 200 queries simultáneas  
D) Max 100 conexiones, timeout 60 segundos; ejecutar 100 queries de 60+ segundos  

**Respuesta correcta: B**
Los valores por defecto de ASP.NET son: **max 100 conexiones** y **timeout de 30 segundos**. El ataque ejecuta **100 queries de más de 30 segundos dentro de un periodo de 30 segundos** para agotar completamente el connection pool e impedir que otros usuarios accedan a la BBDD.

---

**P6.** Un SOC analista revisa logs de acceso y encuentra que un cliente accedió a la aplicación sin pasar por la página de pago, yendo directamente del step 2 al step 4 del proceso de checkout. ¿Qué fase de la metodología de web application hacking corresponde a esta técnica?

A) Fase 7 — Attack Session Management  
B) Fase 9 — Attack Application Logic Flaws  
C) Fase 5 — Attack Authorization Schemes  
D) Fase 8 — Perform Injection Attacks  

**Respuesta correcta: B**
Saltarse una fase de un proceso de varios pasos (ej. ir de step 2 a step 4 en checkout para evitar el pago) se conoce como **forced browsing** y corresponde a la **Fase 9 — Attack Application Logic Flaws**.

---

**P7.** Un pentester identifica que una web application PHP incluye ficheros mediante `require($_ GET['page'])` sin validación. Intenta leer `/etc/passwd` pero el servidor añade `.php` al final automáticamente. ¿Qué payload puede usar para evadir esta restricción?

A) `../../etc/passwd#`  
B) `../../etc/passwd%00`  
C) `../../etc/passwd+`  
D) `../../etc/passwd;`  

**Respuesta correcta: B**
El **null byte `%00`** (URL-encoded) trunca el string en C/PHP, evitando que se añada la extensión `.php`. Esto es un ataque de **Local File Inclusion (LFI)**. La opción `?` también funciona como alternativa, pero `%00` es la técnica clásica mencionada en el CEH.

---

**P8.** Un atacante modifica el header SOAPAction de un mensaje a `deleteAllUsers` mientras mantiene en el SOAP body una operación legítima (`getUsers`). El gateway de seguridad permite el paso. ¿Qué mecanismo explota este ataque?

A) El gateway valida el SOAPAction header y el servidor ejecuta el SOAP body  
B) El gateway valida el SOAP body y el servidor ejecuta la operación del SOAPAction header  
C) El gateway no puede ver el contenido SOAP cifrado  
D) El servidor ejecuta ambas operaciones en secuencia  

**Respuesta correcta: B**
En **SOAPAction Spoofing**, el gateway valida el **SOAP body** (operación legítima → permite el paso) pero el servidor web service ejecuta la operación del **SOAPAction header** (maliciosa: `deleteAllUsers`). La discrepancia entre lo que el gateway valida y lo que el servidor ejecuta es el mecanismo del ataque.

---

**P9.** Un pentester usa WhatWeb contra una aplicación web objetivo. La herramienta identifica CMS, librerías JS, versiones y emails. ¿Cuántos plugins tiene WhatWeb que hacen posible esta identificación exhaustiva?

A) 800 plugins  
B) 1.200 plugins  
C) 1.800+ plugins  
D) 2.500 plugins  

**Respuesta correcta: C**
WhatWeb tiene **más de 1800 plugins**, lo que le permite identificar CMS, plataformas de blogging, librerías JavaScript, servidores web, dispositivos embebidos, versiones, emails, account IDs y errores SQL. Este dato numérico específico aparece frecuentemente en el examen CEH.

---

**P10.** Un analista usa la herramienta `dirb http://www.target.com` y también `gobuster -u http://www.target.com -w common.txt -s 200`. ¿En qué fase de la metodología de web app hacking se enmarcan estas actividades?

A) Fase 1 — Footprint Web Infrastructure  
B) Fase 2 — Analyze Web Applications  
C) Fase 4 — Attack Authentication Mechanisms  
D) Fase 8 — Perform Injection Attacks  

**Respuesta correcta: B**
La enumeración de ficheros y directorios con `dirb` y `gobuster` forma parte de la **Fase 2 — Analyze Web Applications**. El objetivo es identificar subdirectorios, ficheros y parámetros de la aplicación. La Fase 1 es footprinting de infraestructura (Nmap, whois).

---

**P11.** Durante un pentest, el atacante intercepta una cookie con valor `JSESSIONID=user01` y la modifica a `JSESSIONID=user02`. Consigue acceder a la cuenta del segundo usuario. ¿Qué vulnerabilidades combinadas hacen posible este ataque?

A) SQL Injection + Privilege Escalation  
B) Session Token Prediction + Session Token Tampering  
C) CSRF + Session Fixation  
D) Cookie Sniffing + Brute-Force  

**Respuesta correcta: B**
El ataque combina **Session Token Prediction** (el token `JSESSIONID` contiene el username directamente, haciendo el patrón predecible) y **Session Token Tampering** (modificar el valor del token para acceder como otro usuario). JSESSIONID = Java. El nombre de usuario como token es una design flaw de gestión de sesiones.

---

**P12.** Un cliente ha desplegado su aplicación web en un entorno compartido (shared hosting). Un pentester descubre un script Perl con el código: `my $command = param("cmmd"); $command=`$command`;`. ¿Qué tipo de vulnerabilidad es esta y en qué fase de la metodología se explota?

A) SSRF — Fase 7 (Attack Session Management)  
B) OS Command Injection — Fase 10 (Attack Shared Environments)  
C) SQL Injection — Fase 11 (Attack Database Connectivity)  
D) File Injection — Fase 8 (Perform Injection Attacks)  

**Respuesta correcta: B**
El script Perl pasa el parámetro `cmmd` directamente a la shell del OS sin validación, permitiendo **OS Command Injection** (ejecutar `whoami`, `cat /etc/passwd`, etc.). Esto se explota en la **Fase 10 — Attack Shared Environments**, donde múltiples clientes en la misma infraestructura son vulnerables entre sí.
