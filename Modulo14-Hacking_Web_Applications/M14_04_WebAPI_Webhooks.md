# M14_04_WebAPI_Webhooks.md
> Módulo 14 / Subapartado 4 — Web API and Webhooks

---

## 1. Conceptos y definiciones

### Web API — Definición y tipos

Web API = interfaz de programación que proporciona servicios web online a aplicaciones cliente para recuperar y actualizar datos desde múltiples fuentes online via HTTP.

**Tipos de Web Service APIs:**

| API | Descripción | Diferenciador |
|---|---|---|
| **SOAP API** | Protocolo de comunicación web via XML y HTTP; multiplataforma (Windows, macOS, Linux) | Genera, recupera, modifica y borra logs/profiles/credentials |
| **REST API** | Estilo arquitectónico (no protocolo, no framework) para comunicación entre sistemas web | Stateless; usa HTTP methods (PUT, POST, GET, DELETE) |
| **RESTful API** | Implementación de REST principles + HTTP; diseñada para independencia de aplicaciones | 6 características clave (ver abajo) |
| **XML-RPC** | Usa formato XML específico para transferir datos | Más simple que SOAP; usa menos bandwidth |
| **JSON-RPC** | Como XML-RPC pero usa formato JSON | Más ligero que XML |

**RESTful API — 6 características:**

| Característica | Descripción |
|---|---|
| **Stateless** | El cliente almacena el estado de la sesión; el servidor NO guarda datos durante el procesamiento |
| **Cacheable** | El cliente debe cachear las respuestas (representations) para mejorar rendimiento |
| **Client-server** | Cliente y servidor son independientes; servidor = backend, cliente = frontend |
| **Uniform Interface** | Recursos identificados via URL única con métodos HTTP (PUT, POST, GET, DELETE) |
| **Layered System** | Arquitectura multicapa; servidores intermediarios proveen caché compartida; el cliente nunca notifica directamente al servidor principal |
| **Code on Demand** | Opcional: el servidor puede enviar código ejecutable temporal al cliente |

---

### 🔴 Webhooks vs. APIs

| Aspecto | Webhooks | APIs |
|---|---|---|
| Dirección | Automated messages desde sitios web al servidor | Comunicación servidor → sitio web |
| Trigger | Solo cuando hay nueva actualización (HTTP POST) | Hace calls independientemente de actualizaciones |
| Real-time | Actualiza apps con información en tiempo real | Requiere implementaciones adicionales para tiempo real |
| Control de datos | Menos control sobre el flujo de datos | Control fácil sobre el flujo de datos |

Webhooks = "Reverse APIs" — proveen lo necesario para la especificación API, pero el developer debe crear una API para usar un webhook.

**Webhook security risks:**

| Riesgo | Mitigación |
|---|---|
| Data Exposure in Transit | Usar HTTPS (cifrar datos en tránsito) |
| Lack of Verification | Implementar **HMAC** (autenticación e integridad) |
| Replay Attacks | Incluir timestamp en el payload; verificar que esté dentro de la ventana de tiempo aceptable |
| Unrestricted Sources | **IP whitelisting** + mutual TLS |
| Duplication and Forgery | **Mutual TLS** para autenticar sender y receiver |
| Endpoint Configuration Errors | Doble verificación de URLs y validación de endpoints |
| Forged Requests | Usar **secret token** compartido entre sender y receiver |
| Unvalidated Redirects | Validar redirecciones contra whitelist de URLs permitidas |

---

### 🔴 OWASP Top 10 API Security Risks

| ID | Riesgo | Vector principal |
|---|---|---|
| **API1** | Broken Object-Level Authorization | Manipulación de IDs en peticiones para acceder a objetos de otros usuarios |
| **API2** | Broken Authentication | Tokens comprometidos; impersonación de otros usuarios |
| **API3** | Broken Object Property Level Authorization | Exposición de todas las propiedades del objeto; privilege escalation o account takeover |
| **API4** | Unrestricted Resource Consumption | Peticiones concurrentes masivas → DoS por alta carga |
| **API5** | Broken Function-Level Authorization | Acceso a funciones administrativas por usuarios no privilegiados |
| **API6** | Unrestricted Access to Sensitive Business Flows | Automatización abusiva de flujos de negocio (compra de tickets, posts) |
| **API7** | Server-Side Request Forgery | Endpoint que acepta URI del cliente → acceso a servicios internos, port scanning, bypass de firewall |
| **API8** | Security Misconfiguration | Servicios innecesarios; opciones legacy; exposición de datos y detalles del sistema |
| **API9** | Improper Inventory Management | Acceso a versiones antiguas/no parcheadas de la API |
| **API10** | Unsafe Consumption of APIs | Atacar third-party APIs integradas en lugar del objetivo directamente |

---

### 🔴 API Vulnerabilities comunes

| Vulnerabilidad | Descripción |
|---|---|
| **Enumerated Resources** | Flaws de diseño revelan info mediante APIs públicas no autenticadas; permite adivinar user IDs |
| **Sharing Resources via Unsigned URLs** | URLs a recursos hipermedia (imagen, audio, video) vulnerables a hotlinking; solución: signed URLs |
| **Vulnerabilities in Third-Party Libraries** | Librerías open-source sin actualizar → security flaws |
| **Improper Use of CORS** | `Access-Control-Allow-Origin` para todos los orígenes en APIs privadas → hotlinking |
| **Code Injections** | SQLi, XSS en campos de la API → robo de cookies de sesión y credenciales |
| **RBAC Privilege Escalation** | Cambios en endpoints sin atención adecuada en APIs con RBAC |
| **No ABAC Validation** | Sin validación de Attribute-Based Access Control → acceso no autorizado a objetos de la API |
| **Business Logic Flaws** | Explotación de workflows legítimos para fines maliciosos |

---

### 🔴 Web API Hacking Methodology (4 fases)

| Fase | Contenido |
|---|---|
| **1. Identify the target** | HTTP headers en plaintext (HTTP/HTTPS); message formats (JSON para REST, XML para SOAP) |
| **2. Detect security standards** | OAuth, OpenID Connect, SAML, OAuth 1.X/2.X, WS-Security, SSL |
| **3. Identify the attack surface** | API Metadata (Swagger/RAML/WSDL), API Discovery, Brute Force de paths, análisis de requests/responses |
| **4. Launch attacks** | Fuzzing, invalid input, malicious input, injection, insecure configs, credential stuffing, OAuth attacks |

**API Enumeration — Kiterunner:**
```bash
# Single target
kr scan https://target.com:8443/ -w routes.kite -A=apiroutes-210228:20000 -x 10 --ignore-length=34

# Probar HTTP y HTTPS
kr scan target.com -w routes.kite -A=apiroutes-210228:20000 -x 10 --ignore-length=34

# Lista de targets
kr scan targets.txt -w routes.kite -A=apiroutes-210228:20000 -x 10 --ignore-length=34
```

**API Metadata formats:**
- REST API: **Swagger, RAML, API-Blueprint, I/O Docs**
- SOAP API: **WSDL/XML-Schema**

**Herramienta de análisis de requests/responses:** Postman — captura API traffic (requests, responses, cookies) usando built-in proxy.

---

### 🔴 Ataques contra APIs

#### Fuzzing
Envío repetido de input aleatorio → genera mensajes de error que revelan información crítica. Herramienta: **Fuzzapi**.

#### XML Bomb Attack (Billion Laughs)
Documento XML con entidades anidadas exponencialmente (lol1→lol2→…→lol9). Al procesar, el parser intenta expandir lol9 → memory-out-of-bound error → servidor caído o vulnerable. Es una variante de recursive payload attack.

#### Injection en APIs
```
# Normal
http://billpay.com/api/v1/cust/459
SELECT * FROM Customers where custID='459'

# SQL injection
http://billpay.com/api/v1/cust/'%20or%20'1'='1
SELECT * FROM Customers where custID='' or '1' = '1'
```
Las APIs son también vulnerables a JSON injection, JavaScript injection, XPath injection, XSLT injection.

#### Credential Stuffing
No es password guessing ni brute-force. El atacante reutiliza pares de credenciales previamente robados de otras brechas en múltiples servicios. Herramientas: **Sentry MBA**, **PhantomJS**.

#### Insecure Direct Object References (IDOR)
Direct object references usados como argumentos de API calls sin imposición de access rights. Identificables mediante API metadata.

#### IDOR Bypass via Parameter Pollution
El atacante envía dos parámetros `user_id` en la misma petición: uno del atacante + uno de la víctima.
```
# Normal (solo atacante — genera 401)
api.xyz.com/profile/user_id=654

# Parameter pollution (bypassa IDOR)
api.xyz.com/profile/user_id=654&user_id=321
```
La aplicación verifica el primer user_id (atacante → válido) y accede a los datos del segundo (víctima).

---

### 🔴 OAuth Attacks

OAuth = protocolo de autorización que permite acceso limitado a recursos de un usuario en un sitio, a otro sitio, sin exponer credenciales.

**Actores de OAuth:**

| Actor | Rol |
|---|---|
| **Resource Owner** | Usuario que concede permisos (read/write) a una aplicación |
| **Authorization/Resource Server (API)** | Resource server = cuenta de usuario protegida; Authorization server = valida identidad y emite access token |
| **Client/Application** | Aplicación que solicita acceso a la cuenta del usuario |

**4 pasos del Authorization Code Grant:**
1. Usuario envía GET request al cliente via user agent (botón "Login with / Connect")
2. User agent redirigido al authorization server con: `response_type`, `client_id`, `redirect_uri`, `scope`, `state`
3. Authorization server redirige a `redirect_uri` con: `code` (authorization code) + `state`
4. Cliente solicita access token con: `grant_type=authorization_code`, `code`, `redirect_uri`

**Tipos de ataques OAuth:**

| Ataque | Mecanismo |
|---|---|
| **Attack on Connect Request** | CSRF para conectar cuenta falsa del atacante con cuenta de la víctima en el cliente |
| **Attack on redirect_uri** | Explotar XSS en el dominio del cliente para exfiltrar el authorization code a través del redirect_uri |
| **CSRF on Authorization Response** | CSRF para conectar cuenta falsa del atacante con cuenta de la víctima; el atacante almacena el authorization_code |
| **Access Token Reusage** | Access token de "ClientA" funciona en "ClientB" en implicit grant → el atacante reutiliza el token |
| **SSRF via Dynamic Client Registration** | Parámetros vulnerables en el endpoint `/register`: `logo_uri`, `jwks_uri`, `request_uris` → SSRF |
| **WebFinger User Enumeration** | `/.well-known/webfinger` permite validar usernames que no existen → enumeración de usuarios |
| **Exploiting Flawed Scope Validation** | Añade scope adicional (ej. `openid email profile`) al exchange del authorization code → acceso a más datos |

**SSRF via Dynamic Client Registration — parámetros vulnerables:**
- `logo_uri`: el servidor fetcha el logo → SSRF
- `jwks_uri`: URL maliciosa en lugar de JWT key → el atacante obtiene authorization code de todos los usuarios
- `request_uris`: array de URLs con JWT → SSRF mediante request_uri

---

### 🔴 OWASP API Security Risks — Soluciones clave

| API | Solución clave |
|---|---|
| API1 | GUIDs como IDs; mecanismo de autorización basado en políticas de usuario |
| API2 | Account lockout/CAPTCHA; weak-password checks; MFA |
| API3 | Evitar métodos genéricos como `to_json()`; mínimo de datos en respuesta |
| API4 | Limitar rate de interacciones cliente-API; throttling |
| API5 | Evitar function-level authorization; default deny |
| API6 | Protección para flujos de negocio; bloquear IPs de Tor/proxies conocidos; CAPTCHA/biometría |
| API7 | Aislar mecanismos de fetch de recursos remotos; deshabilitar HTTP redirections |
| API8 | Solo los HTTP verbs necesarios; CORS policy correcta; security headers |
| API9 | Inventario de todos los entornos API (prod, staging, testing, dev); security review de todas las versiones |
| API10 | Validar y sanitizar datos de APIs integradas; allowlist de redirecciones |

---

### 🔴 Best Practices API Security (puntos clave para examen)

- HTTPS con TLS/SSL
- Server-generated tokens en hidden fields HTML para validar requests
- Sanitizar y validar input **en el servidor** (no client-side para evitar bypass)
- IP whitelisting
- **Rate limiting** para limitar calls por cliente en un tiempo determinado
- **Pagination** para evitar payloads oversized
- Parameterized statements en SQL (prevenir SQL injection)
- Implementar **API gateways** para autenticar tráfico y controlar/analizar uso
- **MFA** + OAuth2 + OpenID Connect para autenticación
- Security headers: `X-Frame-Options`, `X-XSS-Protection`, `Content-Security-Policy`
- Usar SOAP APIs (con security features built-in) en lugar de REST para escenarios de alta seguridad
- Tokens con expiración y revocación
- Quotas y throttling para controlar y rastrear uso de API
- Log auditoría antes y después de cada security event; sanitizar logs (prevenir log injection)
- Service mesh technology para gestión de autenticación multi-servicio

### 🔴 Best Practices Webhook Security (puntos clave)

- HTTPS (no HTTP)
- **HMAC-based signatures** para verificación de mensajes
- Tracking de `event_id` para evitar replay attacks (doble procesamiento)
- Firewall rechaza llamadas de fuentes no autorizadas que no sean IPs del ESP
- Rate limiting en webhook calls
- Comparar `X-Cld-Timestamp` con timestamp actual (prevenir timing attacks)
- Idempotency en el procesamiento de eventos (evitar replicación)
- **Mutual TLS** para verificar clientes
- No enviar información confidencial por webhooks → usar APIs autorizadas
- Responder siempre con 200 OK (no 4xx/5xx) para evitar desactivación del webhook

---

## 2. Exam Traps ⚠️

⚠️ **[REST vs. RESTful — no son sinónimos exactos]**
REST es un estilo arquitectónico. RESTful API es la implementación de ese estilo usando HTTP. El examen puede usar ambos términos indistintamente; REST no es un protocolo (SOAP sí es protocolo).

⚠️ **[XML-RPC vs. JSON-RPC vs. SOAP — diferenciadores]**
SOAP usa XML propietario. XML-RPC usa formato XML específico (más simple, menos bandwidth). JSON-RPC = como XML-RPC pero con JSON. El examen puede mezclar cuál usa XML y cuál JSON.

⚠️ **[Webhooks = "Reverse APIs" — dirección del flujo]**
Las APIs comunican servidor → sitio. Los webhooks comunican sitio web → servidor (HTTP POST cuando hay evento). El examen puede invertir la dirección. "Reverse" se refiere a que es el servidor el que notifica al cliente proactivamente.

⚠️ **[Webhook Replay Attacks: mitigación específica]**
La mitigación es incluir un **timestamp** en el payload y verificar que esté dentro de una ventana de tiempo aceptable. No es HMAC (eso es para Lack of Verification). No es mutual TLS (eso es para Duplication/Forgery y Unrestricted Sources).

⚠️ **[Credential Stuffing vs. Brute-Force]**
Credential stuffing NO implica adivinar contraseñas. Usa pares de credenciales ya robadas de otras brechas. No es volumétrico por fuerza bruta; es automatización de credenciales conocidas. Herramientas: Sentry MBA, PhantomJS.

⚠️ **[IDOR Bypass via Parameter Pollution: orden de los IDs]**
El primer user_id es del **atacante** (para que la verificación pase) y el segundo es de la **víctima** (para acceder a sus datos). El orden importa: `user_id=atacante&user_id=víctima`. Inverso = no funciona.

⚠️ **[OAuth actores: Resource Server vs. Authorization Server]**
Son dos roles distintos que pueden estar en el mismo servidor o separados. Resource Server = cuenta protegida del usuario. Authorization Server = valida identidad y emite access tokens. El examen puede presentarlos como un único componente.

⚠️ **[OAuth Attack on redirect_uri: XSS en el dominio del cliente]**
El ataque requiere una vulnerabilidad **XSS** en el dominio del cliente (no del provider). El authorization code se exfiltra a través de esa página vulnerable. No es un ataque directo sobre el redirect_uri del provider.

⚠️ **[SSRF via Dynamic Client Registration: parámetros vulnerables exactos]**
Los tres parámetros: `logo_uri`, `jwks_uri`, `request_uris`. El endpoint mapeado es `/register`. El examen puede preguntar cuáles parámetros del registration request son vulnerables a SSRF.

⚠️ **[API9: Improper Inventory Management — el riesgo específico]**
El riesgo es el acceso a versiones antiguas/no parcheadas de la API conectadas a la misma BBDD de producción. No es sobre la gestión de usuarios. Es sobre el ciclo de vida de las versiones de la API.

⚠️ **[API Security: input validation server-side no client-side]**
El libro especifica explícitamente: "Perform input validation **on the server side** instead of the client side to prevent bypassing attacks." El examen puede presentar client-side como válido.

---

## 3. Nemotécnicos

**RESTful API 6 características** → **"S-C-CS-U-L-CoD"**:
**S**tateless · **C**acheable · **C**lient-**S**erver · **U**niform Interface · **L**ayered System · **C**ode **o**n **D**emand

**Webhook risks (8)** → **"D-V-R-U-DF-E-F-R"**:
**D**ata exposure · **V**erification lack · **R**eplay · **U**nrestricted sources · **D**uplication/**F**orgery · **E**ndpoint errors · **F**orged requests · **R**edirects unvalidated

**OWASP API Top 10** → **"BOA-AUA-BFA-USS-SSRF-SM-IM-UCAPI"**:
- API1: **B**roken **O**bject-Level **A**uth
- API2: Broken **A**uth
- API3: Broken **O**bject **P**roperty-Level Auth *(A3)*
- API4: **U**nrestricted Resource Consumption
- API5: Broken **F**unction-Level **A**uth
- API6: **U**nrestricted **S**ensitive Business Flows
- API7: **SSRF**
- API8: **S**ecurity **M**isconfiguration
- API9: **I**mproper Inventory **M**anagement
- API10: **U**nsafe **C**onsumption of **API**s

**OAuth actors** → **"O-A-C"**: **O**wner (user) · **A**uthorization/Resource Server · **C**lient (application)

**OAuth Authorization Code Grant (4 pasos)** → **"GET → Redirect(code) → Redirect(uri+code) → Token"**

**OAuth attacks (6)** → **"Connect-Redirect-CSRF-Token-SSRF-Scope"**

**API Hacking Methodology (4 fases)** → **"I-D-I-L"**: **I**dentify · **D**etect standards · **I**dentify attack surface · **L**aunch attacks

**Webhook mitigaciones clave** → **"HMAC → Replay=Timestamp · Source=IPwhitelist+mTLS · Forgery=SecretToken"**

---

## 4. Flashcards

**Q:** ¿Cuál es la diferencia entre REST API y RESTful API?
**A:** REST es un estilo arquitectónico (no protocolo, no framework). RESTful API es la implementación de ese estilo usando principios REST y protocolos HTTP (PUT, POST, GET, DELETE).

---

**Q:** ¿Cuáles son las 6 características de una RESTful API?
**A:** Stateless, Cacheable, Client-Server, Uniform Interface, Layered System, Code on Demand.

---

**Q:** ¿Qué hace que los webhooks sean llamados "Reverse APIs"?
**A:** Porque la dirección de comunicación es inversa: los webhooks envían mensajes automáticos desde sitios web al servidor (HTTP POST cuando ocurre un evento), mientras las APIs son llamadas del servidor al sitio.

---

**Q:** ¿Qué mitigación específica protege contra replay attacks en webhooks?
**A:** Incluir un timestamp en el payload del webhook y verificar al recibirlo que esté dentro de una ventana de tiempo aceptable.

---

**Q:** ¿Cuáles son los riesgos API1 y API5 de OWASP y en qué se diferencian?
**A:** API1 (Broken Object-Level Authorization): acceso a objetos de otros usuarios manipulando IDs. API5 (Broken Function-Level Authorization): acceso a funciones administrativas por usuarios no privilegiados. API1 = acceso a datos de objetos. API5 = acceso a funciones de nivel admin.

---

**Q:** ¿Qué es credential stuffing y en qué se diferencia de brute-force?
**A:** Credential stuffing automatiza el uso de pares de credenciales robados de otras brechas en múltiples servicios. NO adivina contraseñas; usa las ya conocidas. Herramientas: Sentry MBA, PhantomJS.

---

**Q:** ¿Cuál es el mecanismo del IDOR bypass via parameter pollution?
**A:** El atacante envía dos parámetros user_id: `user_id=atacante&user_id=víctima`. La aplicación verifica el primero (atacante, válido) y procesa el segundo (datos de la víctima).

---

**Q:** ¿Cuáles son los tres actores de OAuth?
**A:** Resource Owner (usuario que concede permisos), Authorization/Resource Server (valida identidad y emite tokens), Client/Application (solicita acceso a la cuenta).

---

**Q:** ¿Cuáles son los tres parámetros vulnerables a SSRF en el Dynamic Client Registration endpoint de OAuth?
**A:** `logo_uri`, `jwks_uri` y `request_uris`. El endpoint está mapeado a `/register`.

---

**Q:** ¿Qué hace el XML Bomb attack y cómo funciona?
**A:** Crea entidades XML anidadas exponencialmente (lol1→lol9). Al procesar lol9, el parser intenta expandir miles de millones de caracteres → memory-out-of-bound error → servidor caído. Es un tipo de Recursive Payload attack.

---

**Q:** ¿Qué tool usa el CEH para API enumeration y cuál es su ventaja frente a herramientas convencionales?
**A:** Kiterunner — no solo hace brute-force de directorios sino que descubre y comprende estructuras complejas de endpoints de API mediante context-aware scanning.

---

**Q:** ¿Qué metadata formats usa REST API vs. SOAP API para describir sus APIs?
**A:** REST: Swagger, RAML, API-Blueprint, I/O Docs. SOAP: WSDL/XML-Schema.

---

**Q:** ¿Por qué el libro recomienda input validation server-side en lugar de client-side?
**A:** Para prevenir ataques de bypass: si la validación es solo client-side, el atacante puede manipular el HTTP request directamente y eludirla.

---

**Q:** ¿Qué riesgo API corresponde a versiones antiguas de una API conectadas a la misma BBDD de producción?
**A:** API9 — Improper Inventory Management. El atacante accede a versiones antiguas/no parcheadas para obtener datos de producción.

---

**Q:** ¿Qué herramienta REST API vulnerability scanner menciona el libro que puede también integrarse en un CICD pipeline?
**A:** Astra — puede descubrir y testear autenticaciones (login/logout) y se integra fácilmente en pipelines CICD.

---

**Q:** ¿Cuál es la diferencia entre HMAC y mutual TLS en seguridad de webhooks?
**A:** HMAC: verifica autenticidad e integridad del mensaje (Lack of Verification). Mutual TLS: autentica tanto sender como receiver en la comunicación (Duplication/Forgery y Unrestricted Sources).

---

**Q:** ¿Qué best practice de API security previene SQL injection?
**A:** Usar parameterized statements (declaraciones parametrizadas) en queries SQL.

---

**Q:** ¿Qué security headers específicos menciona el libro para APIs?
**A:** `X-Frame-Options`, `X-XSS-Protection`, `Content-Security-Policy`.

---

**Q:** ¿En qué consiste el ataque OAuth "Exploiting Flawed Scope Validation"?
**A:** El atacante añade scopes adicionales (ej. `profile`) al exchange del authorization code o access token → el OAuth server (con scope validation defectuosa) concede acceso a más datos del resource owner de los que debería.

---

**Q:** ¿Qué hace la vulnerabilidad CORS `Access-Control-Allow-Origin: *` en una API privada?
**A:** Permite a cualquier origen acceder a los recursos → vulnerable a hotlinking. En APIs privadas, solo deben permitirse orígenes específicos de confianza.

---

## 5. Confusión frecuente

**SOAP vs. XML-RPC vs. JSON-RPC**
- SOAP: protocolo completo que usa XML propietario; más complejo; soporta WS-Security, WSDL.
- XML-RPC: usa formato XML específico estándar; más simple que SOAP; menos bandwidth.
- JSON-RPC: como XML-RPC pero con JSON; más ligero que XML-RPC.
- Criterio: "protocolo completo con seguridad built-in y WSDL" → SOAP. "RPC simple con XML, menos bandwidth" → XML-RPC. "RPC simple con JSON" → JSON-RPC.

---

**API1 vs. API3 (ambas son sobre autorización de objetos)**
- API1 (Broken Object-Level Authorization): el atacante accede a objetos de OTROS usuarios manipulando IDs en la petición → exposición de datos de otros usuarios.
- API3 (Broken Object Property Level Authorization): el API expone TODAS las propiedades del objeto al cliente sin filtrar por sensibilidad → puede llevar a privilege escalation.
- Criterio: "acceder a los datos del objeto de otro usuario" → API1. "Ver propiedades sensibles de un objeto que no deberían ser accesibles" → API3.

---

**Webhook Lack of Verification vs. Replay Attack — mitigaciones distintas**
- Lack of Verification: sin mecanismo para verificar autenticidad/integridad → solución: **HMAC**.
- Replay Attack: reenvío de webhook legítimo para causar acciones no deseadas → solución: **timestamp** en payload + ventana de tiempo.
- Criterio: "verificar que el remitente es quien dice ser y el mensaje no fue alterado" → HMAC. "Prevenir que el mismo webhook se procese dos veces" → timestamp + event_id.

---

**OAuth Resource Server vs. Authorization Server**
- Resource Server: aloja y protege la cuenta/recursos del usuario.
- Authorization Server: valida la identidad del usuario y emite access tokens.
- En la práctica pueden ser el mismo servidor, pero el examen los trata como roles distintos.
- Criterio: "emite el access token" → Authorization Server. "Proporciona acceso a los recursos protegidos con el token" → Resource Server.

---

**API Fuzzing vs. Invalid Input vs. Malicious Input**
- Fuzzing: input aleatorio masivo para generar errores que revelan información. Herramienta: Fuzzapi.
- Invalid Input: cuando fuzzing es difícil por estructura; input inválido (texto en lugar de número, null chars) para obtener info de errores inesperados.
- Malicious Input: más peligroso; inyecta input malicioso directo (XML bomb, shell scripts) para comprometer la infraestructura.
- Criterio: "input aleatorio para revelar info" → Fuzzing. "Input tipo incorrecto para explorar comportamiento" → Invalid Input. "Input diseñado para comprometer el servidor" → Malicious Input.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un auditor de seguridad necesita evaluar si una API REST cumple con los principios de diseño RESTful. Encuentra que el servidor mantiene el estado de la sesión del usuario entre peticiones. ¿Qué principio RESTful viola esta implementación?

A) Cacheable  
B) Uniform Interface  
C) Stateless  
D) Layered System  

**Respuesta correcta: C**
La característica **Stateless** de REST establece que el **servidor NO guarda datos del estado de la sesión** entre peticiones — el cliente debe almacenar toda la información de estado. Si el servidor mantiene estado de sesión, viola este principio fundamental de RESTful APIs.

---

**P2.** Un equipo de desarrollo integra webhooks de un tercero en su plataforma. El CISO pregunta cómo se puede garantizar que los mensajes recibidos provienen realmente del remitente esperado y no han sido modificados. ¿Qué mecanismo resuelve este problema?

A) IP whitelisting  
B) Timestamps en el payload  
C) HMAC-based signatures  
D) Mutual TLS  

**Respuesta correcta: C**
**HMAC** (Hash-based Message Authentication Code) resuelve el problema de **Lack of Verification** en webhooks — verifica tanto la **autenticidad** (que el remitente es quien dice ser) como la **integridad** (que el mensaje no fue alterado). IP whitelisting limita las fuentes. Timestamps previenen replay attacks. Mutual TLS autentica la conexión pero no el contenido del mensaje.

---

**P3.** Un atacante descubre que la API de una aplicación acepta el parámetro `user_id=654` para devolver el perfil del usuario. Al cambiar el valor a `user_id=321` obtiene el perfil de otro usuario. ¿Qué vulnerabilidad de la OWASP API Top 10 explota?

A) API4 — Unrestricted Resource Consumption  
B) API5 — Broken Function-Level Authorization  
C) API1 — Broken Object-Level Authorization  
D) API3 — Broken Object Property Level Authorization  

**Respuesta correcta: C**
**API1 — Broken Object-Level Authorization** ocurre cuando la API no valida que el usuario tiene derecho de acceso al objeto específico que solicita, permitiendo manipular IDs para acceder a objetos de otros usuarios. API5 es sobre funciones admin. API3 expone propiedades del objeto, no acceso a objetos de otros usuarios.

---

**P4.** Un atacante registra una aplicación OAuth maliciosa usando el endpoint `/register` con el parámetro `logo_uri` apuntando a un servidor interno de la red: `http://169.254.169.254/latest/meta-data/`. ¿Qué tipo de ataque ejecuta?

A) WebFinger User Enumeration  
B) Access Token Reusage  
C) SSRF via Dynamic Client Registration  
D) CSRF on Authorization Response  

**Respuesta correcta: C**
El parámetro `logo_uri` del endpoint OAuth `/register` es vulnerable a **SSRF (Server-Side Request Forgery) via Dynamic Client Registration**. El servidor intenta fetchear el logo desde la URI proporcionada, resultando en una petición interna al servidor de metadata de AWS. Los tres parámetros vulnerables son: `logo_uri`, `jwks_uri` y `request_uris`.

---

**P5.** Un pentester prueba la resiliencia de una API ante ataques de denegación de servicio XML. Construye un documento SOAP con entidades anidadas: lol1 que contiene 10 "lol", lol2 con 10 "lol1", hasta lol9. ¿Qué tipo de ataque es este?

A) SOAP Injection  
B) Oversize Payload  
C) XML Bomb (Billion Laughs)  
D) SOAPAction Spoofing  

**Respuesta correcta: C**
El **XML Bomb (Billion Laughs)** usa entidades XML anidadas exponencialmente. Al procesar lol9, el parser intenta expandir potencialmente miles de millones de caracteres → memory-out-of-bound error → servidor caído. Es gramaticalmente válido pero genera DoS. Oversize Payload envía un payload enorme directamente, sin el mecanismo de expansión de entidades.

---

**P6.** Un analista estudia la API de un e-commerce y encuentra que un endpoint de compra de entradas (`/api/v1/buy-ticket`) no tiene límites de velocidad. Un bot compra masivamente entradas antes de que los usuarios reales puedan hacerlo. ¿Qué riesgo OWASP API identifica esta situación?

A) API7 — SSRF  
B) API6 — Unrestricted Access to Sensitive Business Flows  
C) API4 — Unrestricted Resource Consumption  
D) API9 — Improper Inventory Management  

**Respuesta correcta: B**
**API6 — Unrestricted Access to Sensitive Business Flows** cubre la automatización abusiva de flujos de negocio legítimos (compra masiva de tickets, creación masiva de posts). La mitigación incluye bloquear IPs de Tor/proxies conocidos y usar CAPTCHA/biometría. API4 es sobre recursos del sistema (CPU, memoria), no flujos de negocio.

---

**P7.** Un auditor evalúa una API y descubre que responde con la representación completa del objeto usuario incluyendo campos como `is_admin: false`, `internal_score: 82`, `api_secret: "abc123"` que no deberían ser visibles. ¿Qué riesgo OWASP API identifica esta situación?

A) API1 — Broken Object-Level Authorization  
B) API2 — Broken Authentication  
C) API3 — Broken Object Property Level Authorization  
D) API5 — Broken Function-Level Authorization  

**Respuesta correcta: C**
**API3 — Broken Object Property Level Authorization** ocurre cuando la API expone **todas las propiedades del objeto** sin filtrar por sensibilidad/visibilidad. Un atacante podría usar el campo `is_admin: false` para intentar cambiarlo a `true` (privilege escalation). La solución es evitar métodos genéricos como `to_json()` y devolver mínimo de datos.

---

**P8.** Un pentester usa **Kiterunner** para descubrir endpoints de API. ¿Qué ventaja específica tiene Kiterunner frente a herramientas convencionales de brute-force de directorios como dirb o gobuster?

A) Kiterunner usa autenticación OAuth para acceder a APIs protegidas  
B) Kiterunner descubre y comprende estructuras complejas de endpoints con context-aware scanning  
C) Kiterunner solo funciona con SOAP APIs usando WSDL  
D) Kiterunner ejecuta fuzzing de payloads SQL en cada endpoint descubierto  

**Respuesta correcta: B**
**Kiterunner** no solo hace brute-force de directorios sino que realiza **context-aware scanning** que le permite descubrir y comprender estructuras complejas de endpoints de API (parámetros, métodos HTTP, formatos de respuesta). Esto lo diferencia de herramientas simples como dirb o gobuster.

---

**P9.** Durante un pentest OAuth, el atacante observa que el access token obtenido para "ClientA" también funciona cuando se presenta a "ClientB" (implicit grant flow). ¿Qué ataque OAuth es este?

A) CSRF on Authorization Response  
B) Attack on redirect_uri  
C) Exploiting Flawed Scope Validation  
D) Access Token Reusage  

**Respuesta correcta: D**
**Access Token Reusage** ocurre cuando un access token emitido para una aplicación específica ("ClientA") también es aceptado por otra ("ClientB") en el implicit grant flow. El token debería estar vinculado al client_id que lo solicitó. Scope Validation sería añadir permisos no concedidos originalmente.

---

**P10.** Un atacante envía la siguiente petición a una API: `api.xyz.com/profile/user_id=654&user_id=321`. Su propio ID es 654 y el ID de la víctima es 321. La petición devuelve los datos de la víctima. ¿Qué técnica usa el atacante?

A) SQL Injection mediante parameter concatenation  
B) IDOR Bypass via Parameter Pollution  
C) API5 — Broken Function-Level Authorization  
D) Access Token Reusage  

**Respuesta correcta: B**
El atacante usa **IDOR Bypass via Parameter Pollution**: envía dos parámetros `user_id` en la misma petición. La aplicación verifica el primero (ID del atacante → autorizado) y procesa el segundo (ID de la víctima → accede a sus datos). El orden es crítico: atacante primero, víctima segundo.

---

**P11.** Un equipo de seguridad revisa las best practices de API security y debate si la validación de inputs debe hacerse en el cliente o en el servidor. ¿Qué recomienda el CEH y por qué?

A) Client-side; es más eficiente y reduce la carga del servidor  
B) Ambos lados; la redundancia aumenta la seguridad  
C) Server-side; para prevenir ataques de bypass que eludan la validación client-side  
D) Solo mediante WAF externo; no el código de la aplicación  

**Respuesta correcta: C**
El CEH especifica explícitamente: **"Perform input validation on the server side instead of the client side to prevent bypassing attacks."** La validación client-side puede ser eludida manipulando directamente el HTTP request (Burp Suite). La validación server-side es la única que no puede ser bypasseada por el atacante.

---

**P12.** Un desarrollador pregunta qué formato de API metadata debe usar para documentar su nueva API RESTful que necesita ser consumida por servicios externos. ¿Qué formatos están disponibles para REST APIs según el CEH?

A) WSDL y XML-Schema exclusivamente  
B) Swagger, RAML, API-Blueprint e I/O Docs  
C) SOAP, UDDI y WSDL  
D) JSON-Schema y OpenAPI exclusivamente  

**Respuesta correcta: B**
Para **REST APIs**, los formatos de metadata son: **Swagger, RAML, API-Blueprint e I/O Docs**. Para SOAP APIs se usa WSDL/XML-Schema. UDDI es para discovery de servicios, no descripción. Esta distinción REST vs. SOAP para metadata aparece frecuentemente en el examen CEH.
