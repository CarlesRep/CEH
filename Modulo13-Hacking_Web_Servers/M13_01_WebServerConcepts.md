# M13_01_WebServerConcepts.md
> Módulo 13 / Subapartado 1 — Web Server Concepts

---

## 1. Conceptos y definiciones

### Funcionamiento básico de un servidor web

Un servidor web recibe peticiones HTTP de clientes (navegadores), localiza el recurso solicitado en su almacenamiento o en servidores de aplicación, y devuelve una respuesta HTTP. Si el recurso no existe, genera un mensaje de error. El modelo es estrictamente cliente-servidor: el navegador actúa como cliente, el servidor web como servidor.

Servidores web más comunes: **Apache, Microsoft IIS, Nginx, Google Web Server, Tomcat**.

---

### 🔴 Componentes de un servidor web

| Componente | Función | Detalle clave |
|---|---|---|
| **Document Root** | Directorio raíz que almacena los ficheros HTML de un dominio | Ejemplo: dominio `www.certifiedhacker.com`, document root `certroot` en `/admin/web` → ruta física `/admin/web/certroot/` |
| **Server Root** | Directorio de nivel superior con configuración, logs y ejecutables | Subdirectorios: `conf/` (configuración), `logs/` (access + error logs), `cgi-bin/` (scripts CGI) |
| **Virtual Document Tree** | Almacenamiento en disco/máquina diferente cuando el disco original está lleno | Es **case-sensitive**; permite seguridad a nivel de objeto |
| **Virtual Hosting** | Hospedar múltiples dominios/sitios en el mismo servidor | Tipos: **name-based**, **IP-based**, **port-based** |
| **Web Proxy** | Intermediario entre cliente y servidor | Previene bloqueos de IP y mantiene anonimato |

---

### Arquitectura Apache

Apache es open-source y usa arquitectura modular. Sus módulos principales:

| Módulo | Función |
|---|---|
| `mod_auth` | Autenticación de usuarios |
| `mod_ssl` | Cifrado SSL/TLS |
| `mod_rewrite` | Reescritura y redirección de URLs |
| `mod_proxy` | Proxy, gateway y balanceo de carga |

Componente adicional: **BMMTM Extensible Agent** — intercepta peticiones/respuestas HTTP(S) para monitorización y análisis de rendimiento.

---

### 🔴 Arquitectura IIS (Internet Information Services)

Desarrollado por Microsoft. Protocolos soportados: **HTTP, HTTPS, FTP, FTPS, SMTP, NNTP**.

Componentes principales:

| Componente | Rol |
|---|---|
| **HTTP.sys** | Protocol listener — intercepta peticiones HTTP entrantes |
| **WWW Service** | Gestiona procesos y configura HTTP.sys con datos de configuración |
| **WAS (Windows Process Activation Service)** | Lee `ApplicationHost.config` y levanta worker processes |

**Flujo de una petición IIS:**
1. Cliente envía petición HTTP → interceptada por `HTTP.sys`
2. `HTTP.sys` consulta a `WAS`
3. `WAS` lee `ApplicationHost.config` (sitio + application pool) → envía config a `WWW Service`
4. `WWW Service` configura `HTTP.sys`
5. `WAS` inicia el worker process correspondiente al application pool
6. Worker process procesa la petición y devuelve respuesta a `HTTP.sys`
7. Cliente recibe respuesta

Proceso principal: `inetinfo.exe`. Funciones delegadas en DLLs: content indexing, server-side scripting, web-based printing.

---

### 🔴 Arquitectura Nginx

Modelo: **master-worker, single-threaded, event-driven, asíncrono, non-blocking**.
Cada worker puede gestionar **más de 1000 conexiones simultáneas**.

| Componente | Función |
|---|---|
| **Master Process** | Lee/valida configuración, gestiona sockets y worker processes |
| **Worker Processes** | Manejan peticiones de clientes; usan non-blocking I/O |
| **Proxy Cache** | Almacena copias de contenido; reduce carga en backend |
| **Cache Loader** | Carga metadatos de caché en memoria al inicio de Nginx |
| **Cache Manager** | Elimina periódicamente contenido expirado o no usado |
| **Memcache** | Capa de caché en memoria para consultas frecuentes; reduce queries a BBDD |

---

### 🔴 Vulnerabilidades Apache (tabla examen)

| Vulnerabilidad | Mecanismo de ataque |
|---|---|
| HTTP Response Splitting | Inyección de cabeceras maliciosas en respuestas HTTP → XSS, cache poisoning |
| HTTP/2 DoS (endless continuation frames) | Envío continuo de cabeceras HTTP/2 → agotamiento de memoria |
| `mod_macro` buffer over-read | Solicitudes especialmente construidas → lectura más allá del buffer → info disclosure |
| HTTP/2 DoS (window size 0) | Establece `initial window size = 0` → servidor espera indefinidamente → DoS |
| HTTP/2 stream memory not reclaimed on RST | Crear y resetear streams repetidamente → agotamiento de memoria |
| Insecure default configuration | Credenciales admin por defecto → RCE |
| Improper authorization | Controles de autorización defectuosos → escalada de privilegios |
| DNS rebinding (Apache Allura) | Validación de input insuficiente en funcionalidad de importación → acceso a servicios internos |
| Environment variable injection (Apache Zeppelin) | Inyección en `ZEPPELIN_INTP_CLASSPATH_OVERRIDES` → ejecución arbitraria de código |
| Code injection (Apache Zeppelin/JDBC) | Inyección durante configuración de conexión MySQL → RCE |
| Improper certificate validation (Apache Airflow) | Intercepción de tráfico FTP_TLS → MITM |
| XSS (Apache Airflow) | Inyección de scripts maliciosos en logs de task instances |
| Path traversal (Apache OFBiz) | Acceso a directorios fuera del directorio restringido → RCE o info disclosure |
| SQL injection (Apache Submarine) | Neutralización inadecuada de elementos SQL → acceso/modificación no autorizada de BBDD |

---

### 🔴 Vulnerabilidades IIS (tabla examen)

| Vulnerabilidad | Mecanismo |
|---|---|
| Trust boundary violation (Telerik Report Server) | Separación de privilegios insuficiente → acceso no autenticado a funcionalidad restringida |
| Authentication bypass | Endpoint inseguro → acceso no autenticado + ejecución arbitraria de código |
| CRLF XSS (SiteMinder Web Agent) | Mala configuración → ejecución de JavaScript arbitrario en el navegador del cliente |
| CCURE passwords exposed | IIS registra en logs credenciales Windows del C•CURE 9000 → acceso a logs = robo de credenciales |
| Arbitrary file path access (Aquaforest TIFF Server) | Configuración por defecto no restringe rutas → directory traversal |
| Windows IIS elevation of privilege | Manejo incorrecto de peticiones de usuario → toma de control del sistema |
| File/directory permissions (Hitachi JP1) | Permisos por defecto incorrectos → manipulación no autorizada de ficheros |
| TYPO3 XSS (PATH_INFO) | `GeneralUtility::getIndpEnv()` no filtra `PATH_INFO` → inyección de HTML malicioso |
| MailEnable vulnerability | Usuarios autenticados añaden ficheros con contenido no saneado en carpetas públicas → RCE |
| XSS en PasswordManager (`/isapi/PasswordManager.dll`) | Parámetro `ResultURL` no neutralizado → robo de información |

---

### 🔴 Vulnerabilidades Nginx (tabla examen)

| Vulnerabilidad | Mecanismo |
|---|---|
| NULL pointer dereference (HTTP/3 / QUIC) | Solicitudes construidas → terminación de worker processes → DoS o RCE potencial |
| SSRF (mintplex-labs/anything-llm) | Upload link feature → port scanning interno, acceso a apps no públicas, file inclusion, file deletion |
| RCE (Nginx-UI config exposure) | Configuración expuesta → RCE, escalada de privilegios, info disclosure |
| Improper certificate validation (Nginx-UI Import Certificate) | Validación de input deficiente → escritura arbitraria de ficheros |
| SQL injection | Parámetros sin sanitizar en queries → acceso o modificación no autorizada de BBDD |
| Unauthenticated private keys access | `.htaccess` no funciona en Nginx → claves privadas accesibles sin autenticación |
| Excessive memory/CPU (HTTP/2) | Flood de peticiones HTTP/2 → consumo de memoria y CPU → DoS |
| OS command injection (nginxWebUI) | Manipulación de argumentos de fichero en upload feature → ejecución remota de comandos |
| Default file permissions (Nginx Management Suite) | Permisos por defecto permiten modificar ficheros sensibles → afecta Instance Manager y API Connectivity Manager |

---

### Perspectivas de seguridad: por qué se comprometen servidores web

**Webmaster:** El mayor riesgo es la exposición del intranet/LAN a amenazas de Internet. Los scripts CGI pueden contener bugs explotables.

**Network administrator:** Un servidor mal configurado crea agujeros en la seguridad de la LAN. El exceso de control puede inutilizar el acceso legítimo.

**End user:** El usuario percibe la navegación como segura. Sin embargo, JavaScript y WebAssembly pueden ejecutar código malicioso en el navegador y actuar como vector para saltarse firewalls y acceder a la LAN.

---

### 🔴 Errores de configuración que comprometen un servidor web

- Permisos de ficheros y directorios incorrectos
- Instalación con configuración por defecto
- Servicios innecesarios habilitados (content management, administración remota)
- Conflictos entre seguridad y usabilidad del negocio
- Ausencia de política de seguridad, procedimientos y mantenimiento
- Autenticación incorrecta con sistemas externos
- Cuentas por defecto con contraseñas por defecto o sin contraseña
- Ficheros de backup, muestra o por defecto innecesarios
- Bugs en el software del servidor, OS y aplicaciones web
- Certificados SSL mal configurados y configuración de cifrado incorrecta
- Funciones administrativas o de depuración habilitadas o accesibles
- Uso de certificados autofirmados y certificados por defecto
- No dedicar un servidor exclusivamente a servicios web
- Privilegios excesivos / incumplimiento del principio de mínimo privilegio

---

## 2. Exam Traps ⚠️

⚠️ **[Virtual Document Tree vs. Virtual Hosting]**
El examen puede presentarlos como equivalentes. **Virtual Document Tree** resuelve un problema de almacenamiento (disco lleno → otro disco/máquina) y es case-sensitive. **Virtual Hosting** resuelve un problema de infraestructura (múltiples dominios en un mismo servidor). Son conceptos distintos.

⚠️ **[Tipos de Virtual Hosting]**
El examen enumera exactamente tres tipos: **name-based, IP-based, port-based**. No confundir con tipos de hosting (shared, VPS, dedicated). La pregunta puede plantear un cuarto tipo inexistente.

⚠️ **[HTTP.sys vs. WAS vs. WWW Service]**
El examen puede preguntar cuál componente intercepta la petición (HTTP.sys), cuál lee ApplicationHost.config (WAS), o cuál configura HTTP.sys (WWW Service). Son tres roles distintos. HTTP.sys no lee configuración directamente; lo hace WAS.

⚠️ **[Nginx worker processes: número de conexiones]**
Dato específico: cada worker process gestiona **más de 1000 conexiones simultáneas** con un modelo single-threaded non-blocking. No confundir "single-threaded" con "baja capacidad de concurrencia".

⚠️ **[HTTP/2 DoS: window size 0 vs. endless continuation frames]**
Son dos vulnerabilidades distintas de Apache con mecanismos diferentes. Window size 0 → el servidor espera indefinidamente actualizaciones. Endless continuation frames → agotamiento de memoria por cabeceras continuas. El examen puede presentar el síntoma (DoS) y pedir el mecanismo correcto.

⚠️ **[mod_macro buffer over-read → resultado]**
No es DoS. El resultado es **information disclosure** (revela datos de posiciones de memoria adyacentes). Confundirlo con DoS es el error típico.

⚠️ **[Nginx: .htaccess y claves privadas]**
El vector de ataque no es un fallo de Nginx en sí, sino que Nginx no soporta `.htaccess`. Si un desarrollador depende de `.htaccess` para proteger claves privadas y el servidor es Nginx, la protección no aplica. La respuesta correcta no es "fallo de configuración de Nginx" sino "dependencia incorrecta de .htaccess en un servidor que no lo soporta".

⚠️ **[CGI scripts y seguridad]**
Desde la perspectiva del webmaster, los scripts CGI instalados en el servidor pueden contener bugs que son agujeros de seguridad. No son seguros por defecto; su presencia amplía la superficie de ataque.

⚠️ **[Website defacement vs. Data tampering]**
Defacement = cambio visual del sitio (apariencia, mensajes). Data tampering = alteración o borrado de datos del servidor, que puede incluir reemplazar datos con malware. El defacement es estético; el data tampering afecta la integridad de los datos.

⚠️ **[IIS: proceso principal]**
El proceso principal de IIS es `inetinfo.exe`. Las funciones se delegan en DLLs. No confundir con `HTTP.sys` (driver de kernel, listener de protocolo).

---

## 3. Nemotécnicos

**Componentes Server Root** → **"CLC"**
- **C**onf → configuración
- **L**ogs → logs de acceso y error
- **C**gi-bin → scripts CGI

**Módulos Apache** → **"ASRP"**
- **A**uth → autenticación
- **S**SL → cifrado
- **R**ewrite → URLs
- **P**roxy → proxy/balanceo

**Componentes IIS** → **"H-W-WAS"**
- **H**TTP.sys → listener
- **W**WW Service → configura HTTP.sys
- **W**AS → lee ApplicationHost.config, levanta workers

**Flujo IIS (7 pasos):** Cliente → HTTP.sys → WAS → ApplicationHost.config → WWW Service → HTTP.sys configurado → Worker process → Respuesta al cliente

**Tipos de Virtual Hosting** → **"NIP"**: Name-based · IP-based · Port-based

**Impactos de ataques a servidores web** → **"CSRF-RDT"**:
- **C**ompromise of user accounts
- **S**econdary attacks from the website
- **R**oot access to other apps/servers
- **I** → deFacement (website **I**mage change — mnemónico forzado, recuerda que es visual)
- **F** → Data tampering (**F**alsificación de datos)
- **R**eputation damage
- **D**ata theft

*(Alternativa directa: **"Comp-Defa-Second-Root-Tamp-Theft-Rep"** — 7 impactos)*

**Perspectivas de compromiso** → **"WNE"**: **W**ebmaster (CGI, LAN exposure) · **N**etwork admin (configuración LAN) · **E**nd user (JavaScript/WebAssembly, bypass firewall)

---

## 4. Flashcards

**Q:** ¿Qué componente de IIS intercepta las peticiones HTTP entrantes?
**A:** `HTTP.sys` (protocol listener).

---

**Q:** ¿Qué fichero de configuración lee WAS en IIS?
**A:** `ApplicationHost.config` — el fichero raíz del sistema de configuración de IIS.

---

**Q:** ¿Cuántas conexiones simultáneas puede gestionar un worker process de Nginx?
**A:** Más de 1000, gracias al modelo single-threaded, non-blocking I/O, event-driven.

---

**Q:** ¿Cuáles son los tres tipos de virtual hosting?
**A:** Name-based, IP-based, port-based.

---

**Q:** ¿Qué módulo Apache gestiona el balanceo de carga y las funciones de proxy?
**A:** `mod_proxy`.

---

**Q:** ¿Qué vulnerabilidad de Apache consiste en establecer el HTTP/2 initial window size a 0?
**A:** DoS: el servidor queda esperando indefinidamente actualizaciones del window size, resultando en denegación de servicio.

---

**Q:** ¿Qué hace el Cache Manager de Nginx?
**A:** Comprueba periódicamente la caché en busca de contenido expirado y elimina entradas antiguas para que la caché no supere sus límites configurados.

---

**Q:** ¿Qué diferencia hay entre document root y server root?
**A:** Document root almacena los ficheros HTML de las páginas web del dominio. Server root es el directorio de nivel superior con configuración (`conf/`), logs (`logs/`) y scripts CGI (`cgi-bin/`).

---

**Q:** ¿Qué es una Virtual Document Tree y qué característica de seguridad tiene?
**A:** Almacenamiento en una máquina o disco diferente cuando el original está lleno. Es case-sensitive y permite seguridad a nivel de objeto.

---

**Q:** ¿Cuál es el proceso principal de IIS y qué función cumple?
**A:** `inetinfo.exe` — proceso principal del servidor; delega funciones como content indexing, server-side scripting y web-based printing en DLLs.

---

**Q:** ¿Qué vulnerabilidad de Apache permite revelar datos de posiciones de memoria adyacentes?
**A:** `mod_macro` buffer over-read — solicitudes especialmente construidas provocan lectura más allá del límite del buffer.

---

**Q:** ¿Por qué la vulnerabilidad "Unauthenticated private keys access" de Nginx no es un fallo de Nginx en sí?
**A:** Porque la protección dependía de `.htaccess`, que Nginx no soporta. Si el servidor es Nginx, `.htaccess` no tiene efecto y las claves privadas quedan expuestas.

---

**Q:** ¿Qué protocolos soporta IIS además de HTTP/HTTPS?
**A:** FTP, FTPS, SMTP, NNTP.

---

**Q:** ¿Cuál es el objetivo técnico de mod_rewrite en Apache?
**A:** Reescritura de URLs, personalización de rutas y redirección basada en reglas configurables.

---

**Q:** ¿Cuál es el riesgo de seguridad específico que identifica el CEH desde la perspectiva del end user?
**A:** JavaScript y WebAssembly permiten ejecutar código malicioso en el navegador; pueden actuar como vector para saltarse firewalls y acceder a la LAN local.

---

**Q:** ¿Qué componente de Apache intercepta peticiones HTTP(S) para monitorización y análisis de rendimiento?
**A:** BMMTM Extensible Agent.

---

**Q:** ¿Qué subdirectorio del server root contiene los scripts CGI?
**A:** `cgi-bin/`.

---

**Q:** ¿Qué impacto de un ataque web implica reemplazar el contenido de un servidor con malware?
**A:** Data tampering — el atacante altera o borra datos y puede sustituirlos por malware.

---

**Q:** ¿Qué componente Nginx almacena en memoria datos de consultas frecuentes para reducir queries a la base de datos?
**A:** Memcache.

---

**Q:** ¿Cuál es el mecanismo de la vulnerabilidad DNS rebinding en Apache Allura?
**A:** Validación de input insuficiente en la funcionalidad de importación permite manipular respuestas DNS para acceder a servicios internos y exponer información sensible.

---

## 5. Confusión frecuente

**Virtual Document Tree vs. Virtual Hosting**
- Virtual Document Tree: solución de almacenamiento (disco lleno → otro disco/máquina). Propiedad destacada: case-sensitive, seguridad a nivel de objeto.
- Virtual Hosting: solución de infraestructura (múltiples dominios en un mismo servidor). Tipos: name-based, IP-based, port-based.
- Criterio de decisión: si la pregunta habla de espacio en disco o de case-sensitivity → Virtual Document Tree. Si habla de dominios compartiendo servidor → Virtual Hosting.

---

**Website Defacement vs. Data Tampering**
- Defacement: ataque visual/reputacional. El atacante cambia la apariencia del sitio y pone sus propios mensajes. No afecta necesariamente a los datos internos.
- Data Tampering: ataque a la integridad de los datos. El atacante modifica, borra o sustituye datos (puede incluir inyección de malware). No implica necesariamente cambio visual.
- Criterio: "aspecto visual / apariencia del sitio" → defacement. "Datos alterados / malware inyectado en servidor" → data tampering.

---

**HTTP.sys vs. WAS (IIS)**
- HTTP.sys: escucha peticiones (protocol listener). No lee configuración directamente.
- WAS: lee `ApplicationHost.config`, gestiona application pools y levanta worker processes.
- Criterio: "intercepta / escucha petición" → HTTP.sys. "Lee configuración / gestiona workers" → WAS.

---

**Apache mod_proxy vs. mod_rewrite**
- mod_proxy: reenvía peticiones a otros servidores; funciones de proxy y balanceo de carga.
- mod_rewrite: transforma URLs internamente; redirecciones y rutas personalizadas.
- Criterio: "balanceo de carga / reenvío a backend" → mod_proxy. "URL personalizada / redirección basada en reglas" → mod_rewrite.

---

**HTTP/2 DoS: window size 0 vs. endless continuation frames (Apache)**
- Window size 0: el atacante bloquea el envío de datos → servidor espera indefinidamente → DoS por bloqueo.
- Endless continuation frames: el atacante envía cabeceras HTTP/2 sin parar → agotamiento de memoria → DoS por consumo.
- Criterio: "servidor espera / no puede enviar datos" → window size 0. "Memoria agotada / consumo excesivo" → endless continuation frames.
