# M14_01_WebApplicationConcepts.md
> Módulo 14 / Subapartado 1 — Web Application Concepts

---

## 1. Conceptos y definiciones

### Web Applications — Definición y características

Programas software que se ejecutan en navegadores web y actúan como interfaz entre usuarios y servidores web a través de páginas web. Permiten solicitar, enviar y recuperar datos desde/hacia una base de datos sobre Internet mediante una GUI.

**Tecnologías de base:**
- Client-side: JavaScript, HTML, CSS
- Server-side: PHP, ASP, CFM, Java, C#, Ruby, .NET, SQL

**Ventajas clave para el examen:**
- Independientes del sistema operativo → desarrollo y troubleshooting fácil y económico
- Accesibles en cualquier momento y lugar con conexión a Internet
- Ejecutables en cualquier dispositivo con navegador (PDAs, smartphones, etc.)
- Tecnologías core flexibles y escalables: JSP, Servlets, Active Server Pages, SQL Server, .NET
- Datos almacenados en servidores dedicados gestionados por administradores especializados

---

### 🔴 Cómo funcionan las web applications — Flujo de petición

1. Usuario introduce URL en el navegador → petición enviada al **web server**
2. El web server comprueba la **extensión del fichero**:
   - Extensión **HTM/HTML** → el propio web server procesa y sirve el fichero directamente
   - Extensión que requiere procesamiento server-side (**php, asp, cfm**, etc.) → el web server pasa la petición al **web application server**
3. El web application server procesa la petición
4. El web application server accede a la **base de datos** para actualizar o recuperar información
5. El web application server devuelve los resultados al **web server**
6. El web server envía los resultados al **navegador del usuario**

---

### 🔴 Arquitectura de Web Applications — 3 capas

| Capa | Nombre | Contenido |
|---|---|---|
| **Capa 1** | Client / Presentation layer | Dispositivos físicos del cliente (laptops, smartphones, PCs), OS y navegadores |
| **Capa 2** | Business logic layer | Dos subcapas: web-server logic layer + business logic layer |
| **Capa 3** | Database layer | Cloud services, B2B layer (transacciones comerciales), database server (MS SQL Server, MySQL) |

**Detalle de la capa 2 — Business logic layer:**

*Web-server logic layer* — componentes:
- Firewall (seguridad del contenido)
- HTTP request parser (gestiona peticiones entrantes y reenvía respuestas)
- Proxy caching server
- Authentication and login handler
- Resource handler (capaz de gestionar múltiples peticiones simultáneamente)
- Hardware (servidor)
- Ejemplos: IIS Web Server, Apache Web Server

*Business logic layer* — componentes:
- Lógica funcional de la aplicación (implementada con .NET, Java, "middleware")
- Define el flujo de datos
- Almacena datos de la aplicación
- Integra aplicaciones legacy con la funcionalidad más reciente

---

### 🔴 Web Services — Arquitectura y roles

Un web service es una aplicación o software desplegado sobre Internet que usa un **protocolo de mensajería estándar** (como SOAP) para permitir la comunicación entre aplicaciones desarrolladas en plataformas diferentes.

Integración estándar con: **SOAP, UDDI, WSDL, REST**.

**Tres roles:**

| Rol | Función |
|---|---|
| **Service Provider** | Plataforma desde donde se ofrecen los servicios; despliega y publica las service descriptions en el registry |
| **Service Requester** | Aplicación/cliente que busca un servicio; habitualmente el navegador; invoca el servicio en nombre del usuario |
| **Service Registry** | Lugar donde el provider carga las service descriptions; el requester descubre el servicio y obtiene binding data desde aquí |

**Tres operaciones:**

| Operación | Descripción |
|---|---|
| **Publish** | El provider publica las service descriptions para que el requester las descubra |
| **Find** | El requester obtiene las service descriptions: en desarrollo (service interface description) y en runtime (binding + location description) |
| **Bind** | El requester llama y establece comunicación con el servicio en runtime usando los binding data de las service descriptions |

**Dos artefactos:**

| Artefacto | Descripción |
|---|---|
| **Service** | Módulo software ofrecido por el provider; puede actuar también como requester invocando otros servicios |
| **Service Description** | Contiene todos los detalles: operaciones, ubicaciones de red, binding, datatypes. Almacenable en registry |

---

### 🔴 Características de los Web Services

| Característica | Descripción |
|---|---|
| **XML-based** | Usa XML para representación y transporte de datos; evita dependencias de OS, red o plataforma → alta interoperabilidad |
| **Coarse-grained** | Un objeto contiene gran cantidad de información y funcionalidad; es una combinación de múltiples fine-grained services |
| **Loosely coupled** | Interconexión mediante web API enviando mensajes XML; la API añade una capa de abstracción para hacer la conexión flexible y adaptable |
| **Asynchronous + synchronous support** | Sync: el usuario espera respuesta. Async: el usuario no espera. RPC-based messages → sync; document-based messages → async. Implementados con servlets, SOAP/XML, HTTP |
| **RPC support** | Soporta Remote Procedure Calls igual que las aplicaciones tradicionales |

---

### 🔴 Tipos de Web Services

| Tipo | Descripción |
|---|---|
| **SOAP** | Simple Object Access Protocol; define el formato XML para transferencia de datos entre provider y requester; determina el procedimiento para construir web services; habilita el intercambio entre diferentes lenguajes de programación |
| **RESTful** | REpresentational State Transfer; diseñado para mayor productividad; usa conceptos HTTP; es un **enfoque arquitectónico**, no un protocolo como SOAP |

---

### 🔴 Componentes de la arquitectura de Web Services

| Componente | Nombre completo | Función |
|---|---|---|
| **UDDI** | Universal Description, Discovery, and Integration | Servicio de directorio que lista todos los servicios disponibles |
| **WSDL** | Web Services Description Language | Lenguaje basado en XML que describe y rastrea web services |
| **WS-Security** | Web Services Security | Extensión de SOAP; mantiene la integridad y confidencialidad de mensajes SOAP; autentifica usuarios |

Otros componentes: WS-Work Processes, WS-Policy, WS Security Policy.

---

### 🔴 Vulnerability Stack — 7 capas

El vulnerability stack muestra las capas de una web application y los vectores de ataque en cada una.

| Capa | Elemento | Vector de ataque |
|---|---|---|
| **Layer 7** | Custom web application (business logic: .NET, Java) | Input validation attacks: **XSS**, SQL injection |
| **Layer 6** | Third-party components (ej. payment gateways: citrix.com) | Explotación de redirección entre sitio principal y third-party como vía de entrada |
| **Layer 5** | Web server (software) | Footprinting + banner grabbing (nombre/versión) → búsqueda en **CVE database** + explotación; herramienta: **Nmap** |
| **Layer 4** | Database | Explotación de vulnerabilidades de BBDD; herramienta: **sqlmap** |
| **Layer 3** | Operating system | Escaneo de puertos abiertos + envío de malware/backdoors a través de ellos → compromiso del equipo |
| **Layer 2** | Network (routers/switches) | Flood del **CAM table** → switch actúa como hub → sniffing de credenciales y datos |
| **Layer 1** | Security (IDS/IPS) | Técnicas de **evasión** para no activar alarmas del IDS/IPS |

**Dirección del ataque:** el atacante explota vulnerabilidades en una o más capas para ganar acceso irrestricto a la aplicación o a toda la red.

---

## 2. Exam Traps ⚠️

⚠️ **[Extensiones HTM/HTML vs. extensiones server-side]**
Si la extensión es HTM o HTML, el **web server** sirve el fichero directamente sin pasar al web application server. Solo con extensiones que requieren procesamiento server-side (php, asp, cfm) se invoca el web application server. El examen puede presentar HTM como si requiriera procesamiento de aplicación → incorrecto.

⚠️ **[3 capas de arquitectura: la capa 2 tiene dos subcapas]**
La "business logic layer" tiene dentro una *web-server logic layer* y una *business logic layer*. No es una arquitectura de 4 capas; son 3 capas con la capa 2 subdividida. El examen puede presentar 4 capas como opción correcta.

⚠️ **[SOAP vs. REST: protocolo vs. enfoque arquitectónico]**
SOAP es un **protocolo**. REST es un **enfoque arquitectónico** (architectural approach), no un protocolo. El examen puede presentar REST como protocolo o SOAP como "enfoque" → ambos incorrectos.

⚠️ **[UDDI vs. WSDL — funciones distintas]**
UDDI = directorio de servicios disponibles (discovery). WSDL = lenguaje de descripción de servicios (descripción e introspección). El examen los intercambia frecuentemente. Regla: UDDI = directorio/lista. WSDL = descripción/XML.

⚠️ **[WS-Security: extensión de qué]**
WS-Security es una extensión de **SOAP** (no de HTTP, no de REST). Su objetivo: integridad + confidencialidad de mensajes SOAP + autenticación de usuarios. El examen puede vincularlo a REST → incorrecto.

⚠️ **[Coarse-grained vs. fine-grained service]**
Coarse-grained: gran cantidad de información y funcionalidad, combinación de múltiples fine-grained services. El examen puede invertir las definiciones. Regla: "coarse" = grueso = grande = más funcionalidad.

⚠️ **[Vulnerability Stack: Layer 1 = IDS/IPS, Layer 7 = web app]**
La numeración es de abajo (Layer 1 = seguridad/IDS) a arriba (Layer 7 = web app/business logic). El examen puede presentar Layer 1 como la capa más alta o confundir la numeración. La capa más próxima al atacante externo es Layer 1 (seguridad perimetral que debe evadir primero).

⚠️ **[Layer 2: CAM table flood → switch actúa como hub]**
El ataque en Layer 2 no es directamente sniffing; es primero agotar la CAM table del switch para que actúe como hub, y luego sniffear. El examen puede presentar sniffing directo en Layer 2 omitiendo el paso del CAM table flood.

⚠️ **[Layer 4: herramienta es sqlmap]**
Para explotar vulnerabilidades de BBDD (Layer 4), la herramienta mencionada es **sqlmap**. No confundir con Nmap (Layer 5 web server). El examen puede intercambiar las herramientas por capa.

⚠️ **[Service Description: dos fases del Find]**
La operación Find tiene dos fases: en **development time** (service interface description) y en **runtime** (binding + location description). El examen puede presentar Find como una operación única sin distinguir las dos fases.

---

## 3. Nemotécnicos

**Flujo de petición web (6 pasos)** → **"URL → Server → Extensión? → AppServer → DB → Browser"**
- Si HTM/HTML: Server sirve directamente (salta AppServer y DB)
- Si php/asp/cfm: Server → AppServer → DB → Server → Browser

**3 capas de arquitectura** → **"C-B-D"**:
- **C**lient/Presentation · **B**usiness logic (web-server logic + business logic) · **D**atabase

**Componentes web-server logic layer (6)** → **"F-H-P-A-R-HW"**:
- **F**irewall · **H**TTP parser · **P**roxy cache · **A**uth handler · **R**esource handler · **H**ard**w**are

**3 roles de web service** → **"P-R-Re"**: **P**rovider · **R**equester · **Re**gistry

**3 operaciones de web service** → **"P-F-B"**: **P**ublish · **F**ind · **B**ind

**Componentes web service** → **"U-W-WS"**: **U**DDI (directorio) · **W**SDL (descripción XML) · **W**S-**S**ecurity (extensión SOAP)

**Vulnerability Stack (7 capas, de arriba a abajo)** → **"WebApp-3P-WebServer-DB-OS-Network-IDS"**:
- L7: Web app (XSS) · L6: 3rd party (redirección) · L5: Web server (CVE+Nmap) · L4: DB (sqlmap) · L3: OS (malware+puertos) · L2: Network (CAM flood) · L1: IDS/IPS (evasión)

**SOAP vs. REST** → "SOAP = **proto**col · REST = **archi**tecture" (P antes que A en el alfabeto → SOAP es más "rígido", primero)

**Características web services (5)** → **"X-C-L-AS-R"**:
- **X**ML-based · **C**oarse-grained · **L**oosely coupled · **A**sync/**S**ync support · **R**PC support

---

## 4. Flashcards

**Q:** ¿Cuándo el web server pasa la petición al web application server?
**A:** Cuando el fichero solicitado tiene una extensión que requiere procesamiento server-side (php, asp, cfm). Si la extensión es HTM o HTML, el web server lo sirve directamente.

---

**Q:** ¿Cuáles son las 3 capas de la arquitectura de web applications?
**A:** (1) Client/Presentation layer, (2) Business logic layer (con subcapas web-server logic y business logic), (3) Database layer.

---

**Q:** ¿Qué tecnologías de ejemplo contiene la database layer?
**A:** Cloud services, B2B layer (transacciones comerciales) y database servers como MS SQL Server y MySQL.

---

**Q:** ¿Cuáles son los 3 roles en una arquitectura de web service?
**A:** Service Provider (ofrece el servicio), Service Requester (cliente que busca el servicio; habitualmente el navegador), Service Registry (almacena las service descriptions).

---

**Q:** ¿Cuáles son las 3 operaciones de una arquitectura de web service?
**A:** Publish (el provider publica service descriptions), Find (el requester obtiene descriptions), Bind (el requester establece comunicación con el servicio en runtime).

---

**Q:** ¿Cuál es la diferencia entre SOAP y REST?
**A:** SOAP es un protocolo que define el formato XML para transferencia de datos. REST (RESTful) es un enfoque arquitectónico, no un protocolo; usa conceptos HTTP subyacentes.

---

**Q:** ¿Qué es UDDI y para qué sirve?
**A:** Universal Description, Discovery, and Integration — servicio de directorio que lista todos los servicios web disponibles.

---

**Q:** ¿Qué es WSDL y qué tecnología usa?
**A:** Web Services Description Language — lenguaje basado en XML que describe y rastrea web services.

---

**Q:** ¿De qué es extensión WS-Security y cuál es su función?
**A:** Es una extensión de SOAP. Mantiene la integridad y confidencialidad de los mensajes SOAP y autentifica usuarios.

---

**Q:** ¿Qué es un coarse-grained service en el contexto de web services?
**A:** Un servicio que contiene gran cantidad de información y funcionalidad; es una combinación de múltiples fine-grained services.

---

**Q:** En el vulnerability stack, ¿qué capa corresponde al IDS/IPS y qué técnica usa el atacante contra ella?
**A:** Layer 1. El atacante usa técnicas de evasión para no activar alarmas del IDS/IPS.

---

**Q:** ¿Qué ataque realiza el atacante en Layer 2 del vulnerability stack y cuál es el mecanismo?
**A:** Flood del CAM table del switch → el switch actúa como hub → el atacante puede sniffear credenciales y datos de la red.

---

**Q:** ¿Qué herramienta usa el atacante en Layer 4 (Database) del vulnerability stack?
**A:** sqlmap — para explotar vulnerabilidades de la base de datos.

---

**Q:** ¿Qué hace el atacante en Layer 5 (Web Server) del vulnerability stack?
**A:** Footprinting + banner grabbing (obtiene nombre y versión del web server con Nmap) → busca vulnerabilidades publicadas en la CVE database para esa versión → las explota.

---

**Q:** ¿Qué tipo de ataque realiza el atacante en Layer 7 (Web Application) del vulnerability stack?
**A:** Input validation attacks como XSS, explotando vulnerabilidades en la business logic implementada con .NET o Java.

---

**Q:** ¿Cuáles son los dos artefactos en una arquitectura de web service?
**A:** Service (módulo software ofrecido por el provider) y Service Description (contiene operaciones, ubicaciones de red, binding, datatypes; almacenable en el registry).

---

**Q:** ¿Cuáles son los lenguajes server-side y client-side que usa una web application?
**A:** Server-side: Java, C#, Ruby, PHP, etc. Client-side: HTML, JavaScript, CSS.

---

**Q:** ¿En qué dos fases se puede procesar la operación Find de un web service?
**A:** Development time (service interface description) y runtime (binding + location description).

---

**Q:** ¿Qué componentes contiene la web-server logic layer?
**A:** Firewall, HTTP request parser, proxy caching server, authentication and login handler, resource handler y componente hardware (servidor). Ejemplos: IIS, Apache.

---

**Q:** ¿Cuál es la vulnerabilidad de Layer 6 (Third-party components) según el vulnerability stack?
**A:** El atacante explota la redirección entre el sitio principal (ej. Amazon) y el third-party (ej. payment gateway como citrix.com) como vía de entrada al sitio objetivo.

---

## 5. Confusión frecuente

**SOAP vs. REST**
- SOAP: protocolo; define formato XML estricto; usa WSDL para descripción; más rígido; soporta WS-Security.
- REST: enfoque arquitectónico; usa conceptos HTTP (GET, POST, PUT, DELETE); más ligero y flexible; no es un protocolo.
- Criterio: "protocolo con formato XML definido" → SOAP. "Architectural approach sobre HTTP" → REST.

---

**UDDI vs. WSDL**
- UDDI: directorio de servicios (discovery). Responde a "¿qué servicios existen?".
- WSDL: descripción técnica de un servicio específico en XML. Responde a "¿cómo uso este servicio?".
- Criterio: "descubrir/listar servicios disponibles" → UDDI. "Describir la interfaz técnica de un servicio" → WSDL.

---

**Layer 1 vs. Layer 7 del vulnerability stack**
- Layer 1 (más baja) = IDS/IPS → seguridad perimetral. El atacante la enfrenta intentando evadir alarmas.
- Layer 7 (más alta) = Web application / business logic. El atacante la explota con input validation attacks (XSS).
- Criterio: pregunta sobre "evasión de alarmas" → Layer 1. Pregunta sobre "XSS o input validation" → Layer 7.

---

**Web server vs. Web application server**
- Web server: recibe peticiones HTTP; sirve ficheros estáticos (HTM/HTML) directamente; reenvía peticiones de ficheros dinámicos al app server. Ejemplos: Apache, IIS, Nginx.
- Web application server: procesa la lógica de negocio de ficheros dinámicos (php, asp, cfm); accede a la BBDD. Ejemplo: Tomcat.
- Criterio: "sirve ficheros HTM/HTML" → web server. "Procesa PHP/ASP/CFM y accede a BBDD" → web application server.

---

**Service Provider vs. Service Requester vs. Service Registry**
- Provider: publica el servicio y sus descriptions en el registry.
- Registry: almacén de descriptions; el requester las consulta aquí.
- Requester: usa las descriptions del registry para hacer bind con el provider y consumir el servicio.
- Criterio de decisión: el que publica → Provider. El que lista → Registry. El que consume → Requester.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un pentester necesita identificar si el servidor web objetivo delega el procesamiento de ciertos ficheros a un servidor de aplicaciones separado. ¿Qué extensión de fichero confirmaría que el web server NO invoca el web application server?

A) .asp  
B) .php  
C) .html  
D) .cfm  

**Respuesta correcta: C**
Los ficheros con extensión HTM o HTML son servidos directamente por el web server sin pasar al web application server. Las extensiones .asp, .php y .cfm requieren procesamiento server-side y activan la invocación del web application server.

---

**P2.** Un auditor de seguridad analiza la arquitectura de una web application y detecta tres capas. La capa intermedia contiene un firewall, un HTTP request parser, un proxy caching server y un authentication handler. ¿Cómo se denomina correctamente esta subcapa?

A) Business logic layer  
B) Database layer  
C) Web-server logic layer  
D) Presentation layer  

**Respuesta correcta: C**
Los componentes descritos (firewall, HTTP request parser, proxy caching server, authentication handler) pertenecen a la **web-server logic layer**, que es una subcapa dentro de la capa 2 (Business logic layer). La business logic layer propiamente dicha implementa la lógica funcional con .NET o Java.

---

**P3.** Un desarrollador describe su sistema como: "el cliente almacena los datos de la sesión, el servidor no guarda estado entre peticiones, los recursos se identifican mediante URL únicas con métodos HTTP estándar." ¿Cuál es el estilo arquitectónico descrito?

A) SOAP  
B) XML-RPC  
C) WSDL  
D) REST  

**Respuesta correcta: D**
Las características descritas (stateless, identificación de recursos por URL, métodos HTTP) corresponden a **REST** (REpresentational State Transfer). REST es un enfoque arquitectónico, no un protocolo. SOAP es un protocolo; XML-RPC usa formato XML específico; WSDL es un lenguaje de descripción, no un estilo arquitectónico.

---

**P4.** Durante un pentest de aplicación web, el atacante quiere descubrir todos los servicios web disponibles en la organización objetivo. ¿Qué componente debe consultar para obtener un directorio de todos los servicios disponibles?

A) WSDL  
B) WS-Security  
C) UDDI  
D) SOAP  

**Respuesta correcta: C**
**UDDI** (Universal Description, Discovery, and Integration) es el servicio de directorio que lista todos los servicios web disponibles. WSDL describe la interfaz técnica de un servicio específico. WS-Security es una extensión de SOAP para autenticación. SOAP es el protocolo de mensajería.

---

**P5.** Un atacante realiza footprinting de una web application y quiere explotar la capa más cercana a la lógica de negocio de la aplicación usando inputs maliciosos como XSS y SQL injection. ¿A qué capa del vulnerability stack apunta?

A) Layer 1 (IDS/IPS)  
B) Layer 4 (Database)  
C) Layer 5 (Web Server)  
D) Layer 7 (Web Application)  

**Respuesta correcta: D**
La **Layer 7** corresponde a la web application (business logic, .NET, Java) y es el objetivo de input validation attacks como **XSS** y SQL injection. Layer 1 es IDS/IPS (evasión). Layer 4 es Database (sqlmap). Layer 5 es Web Server (banner grabbing + CVE).

---

**P6.** Un pen tester quiere capturar el banner del servidor HTTPS objetivo para identificar su versión. Ha intentado con Telnet y Netcat sin éxito. ¿Qué herramienta y puerto debe usar?

A) Nmap en puerto 80  
B) OpenSSL en puerto 443  
C) Netcat en puerto 8443  
D) Curl en puerto 443  

**Respuesta correcta: B**
Para **banner grabbing sobre SSL/HTTPS** se requiere **OpenSSL** en el **puerto 443**. Telnet y Netcat solo funcionan sobre HTTP plano. Aunque Curl puede acceder a HTTPS, OpenSSL con `s_client -host <target> -port 443` es la herramienta específica mencionada en el CEH.

---

**P7.** En la arquitectura de web services, el Service Provider publica descripciones en el registry, el Service Requester obtiene esas descripciones y luego establece comunicación directa con el proveedor. ¿Qué operación describe el proceso de establecer esa comunicación directa en runtime?

A) Find  
B) Publish  
C) Bind  
D) Register  

**Respuesta correcta: C**
**Bind** es la operación por la que el Service Requester llama e invoca el servicio usando los binding data obtenidos de las service descriptions. Publish es la operación del provider para publicar descriptions. Find es la operación del requester para obtener las descriptions. Register no es una operación estándar del modelo de web services.

---

**P8.** Una organización ha sufrido un ataque en el que el switch de red comenzó a actuar como un hub, permitiendo al atacante capturar todo el tráfico de la red incluyendo credenciales en texto plano. ¿Qué capa del vulnerability stack fue atacada y qué técnica se usó?

A) Layer 1 — Evasión de IDS  
B) Layer 2 — CAM table flood  
C) Layer 3 — Port scanning  
D) Layer 4 — SQL injection  

**Respuesta correcta: B**
El ataque describe un **CAM table flood** en **Layer 2** (Network). Al agotar la CAM table del switch, éste actúa como hub y reenvía el tráfico a todos los puertos, permitiendo el sniffing. Layer 1 es IDS/IPS. Layer 3 es el OS. Layer 4 es la BBDD.

---

**P9.** Un web service utiliza un componente que contiene todas las operaciones disponibles, ubicaciones de red, binding data y datatypes del servicio. Este componente puede ser almacenado en un registry. ¿Cómo se denomina?

A) Service  
B) Service Description  
C) Service Registry  
D) UDDI  

**Respuesta correcta: B**
La **Service Description** contiene todos los detalles: operaciones, ubicaciones de red, binding data y datatypes. Puede almacenarse en el Service Registry. Service es el módulo software en sí. UDDI es el registry donde se almacenan las descriptions (no la description misma).

---

**P10.** Un auditor analiza una web application y encuentra que la operación Find del web service se comporta diferente en development time versus runtime. ¿Qué información proporciona específicamente la operación Find en runtime?

A) Service interface description  
B) Binding + location description  
C) Datatypes del servicio  
D) Lista de todos los servicios disponibles  

**Respuesta correcta: B**
En **runtime**, la operación Find devuelve la **binding + location description** (cómo conectar y dónde está el servicio). En **development time**, Find devuelve la **service interface description** (qué operaciones ofrece el servicio). Los datatypes forman parte de la Service Description. La lista de servicios es el UDDI registry.

---

**P11.** Durante un pentest, un analista ejecuta sqlmap contra la capa de base de datos de una web application a la vez que Nmap con banner grabbing contra el servidor web. ¿A qué capas del vulnerability stack corresponden estos ataques respectivamente?

A) Layer 3 y Layer 5  
B) Layer 4 y Layer 3  
C) Layer 4 y Layer 5  
D) Layer 5 y Layer 7  

**Respuesta correcta: C**
**sqlmap** ataca la **Layer 4 (Database)**. **Nmap con banner grabbing** ataca la **Layer 5 (Web Server)** para identificar nombre y versión del servidor y buscar CVEs asociados. Layer 3 es el OS. Layer 7 es la Web Application.

---

**P12.** Un arquitecto de seguridad está evaluando si usar WS-Security para proteger los mensajes intercambiados entre servicios web. ¿De qué es extensión WS-Security y qué tres objetivos de seguridad cubre?

A) Extensión de REST; cubre autenticación, autorización y auditoría  
B) Extensión de SOAP; cubre integridad, confidencialidad y autenticación  
C) Extensión de WSDL; cubre cifrado, firma digital y no repudio  
D) Extensión de UDDI; cubre disponibilidad, integridad y confidencialidad  

**Respuesta correcta: B**
**WS-Security** es una extensión de **SOAP** (no de REST, WSDL ni UDDI). Sus tres objetivos son: mantener la **integridad** de los mensajes SOAP, mantener la **confidencialidad** de los mensajes SOAP y **autenticar** usuarios. No está relacionado con REST, WSDL ni UDDI.
