# M13_02_WebServerAttacks.md
> Módulo 13 / Subapartado 2 — Web Server Attacks

---

## 1. Conceptos y definiciones

### DNS Server Hijacking

El atacante compromete un servidor DNS y modifica su tabla de mapeo para redirigir las consultas hacia un servidor DNS falso controlado por él. Cuando la víctima introduce una URL legítima, es redirigida al sitio del atacante. La diferencia con otros ataques DNS es que la modificación ocurre **en el servidor DNS en sí**, no en la resolución recursiva.

---

### 🔴 DNS Amplification Attack — Mecanismo paso a paso

Explota las consultas DNS recursivas para generar un DDoS contra el servidor DNS de la víctima. El vector de amplificación es el IP spoofing: los bots envían consultas con la IP de la víctima → las respuestas van a la víctima.

**Flujo del ataque (9 pasos):**
1. El atacante instruye a los hosts comprometidos (bots) para que lancen consultas DNS.
2. Los bots suplantan (*spoof*) la IP de la víctima y envían consultas al servidor DNS primario configurado en los ajustes TCP/IP de la víctima.
3–8. Si el mapeo no existe en el DNS primario, la solicitud se reenvía al root server → a los namespaces TLD (`.com`, etc.) → proceso recursivo hasta resolver.
9. El DNS primario envía la respuesta de mapeo a la IP de la víctima (porque los bots usaron su IP). El volumen de respuestas procedentes de múltiples bots colapsa el servidor DNS de la víctima → **DDoS**.

**Clave del mecanismo:** la amplificación ocurre porque la respuesta DNS es mayor que la consulta y se envía a la víctima, no al bot que la generó.

---

### Directory Traversal

Explotación de HTTP que permite acceder a directorios restringidos fuera del root directory del servidor web manipulando la URL. El atacante usa la secuencia **`../`** (dot-dot-slash) para navegar fuera del directorio raíz. Se apoya en el método de prueba y error para mapear la estructura de directorios.

**Condiciones de vulnerabilidad:** código de la aplicación web con validación insuficiente del input recibido desde el navegador; software del servidor mal parcheado o mal configurado.

---

### Website Defacement

Modificación no autorizada del contenido visual de una o varias páginas web. El atacante inyecta código para añadir imágenes, popups o texto que cambian la apariencia del sitio, o reemplaza el sitio completo. Métodos comunes: **MySQL injection**. Doble objetivo: dañar la reputación de la organización objetivo **y** infectar a los visitantes haciendo el sitio vulnerable a ataques víricos.

---

### 🔴 Web Server Misconfiguration — Lista de tipos

| Tipo de mala configuración | Riesgo específico |
|---|---|
| Verbose debug/error messages | Revela versión del servidor, rutas, estructura interna |
| Anonymous or default users/passwords | Acceso no autenticado o con credenciales conocidas |
| Sample configuration and script files | Ficheros con información o permisos no destinados a producción |
| Remote administration functions | Superficie de ataque remota innecesaria |
| Unnecessary services enabled | Amplían superficie de ataque |
| Misconfigured/default SSL certificates | Comunicaciones inseguras, MITM |
| Directory browsing habilitado (IIS `web.config`) | Expone el árbol de directorios y ficheros sensibles |
| Unrestricted file uploads (IIS) | Permite subir web shells → RCE |
| Root location (`location / {}`) ausente en Nginx | Comportamiento indefinido: listado de directorios, exposición de configuración por defecto |
| User input sin sanitizar en variables | Ataques de inyección |

---

### HTTP Response-Splitting Attack

El atacante inyecta **nuevas líneas (CRLF)** en los campos de cabecera de la respuesta HTTP, junto con código arbitrario. El servidor interpreta el input manipulado como dos respuestas HTTP separadas.

**Mecanismo:**
- El atacante modifica una única petición para que parezca dos añadiendo datos de cabecera de respuesta en el campo de input.
- El servidor responde a cada "petición" por separado.
- El atacante controla la primera respuesta (p. ej., redirige a sitio malicioso); el navegador descarta las demás.

**Ataques derivados de HTTP Response Splitting:** XSS, CSRF, SQL injection, **web cache poisoning**.

**Ejemplo del libro:** el servidor divide la respuesta en dos → primera respuesta al atacante, segunda a la víctima → la víctima proporciona credenciales creyendo estar en el servidor legítimo → el servidor envía la respuesta de la víctima al atacante.

---

### Web Cache Poisoning

El atacante fuerza al servidor web a vaciar su caché real y almacena contenido infectado para una URL específica. Todos los usuarios que accedan a esa URL a través de la caché recibirán contenido malicioso hasta que la caché sea purgada.

**Prerequisito:** el servidor web y la aplicación deben tener vulnerabilidades de HTTP response-splitting.

**Diferencia con Response-Splitting:** Response-splitting manipula la sesión de un usuario concreto; cache poisoning compromete la caché compartida que sirve a **todos** los usuarios.

---

### SSH Brute Force Attack

SSH crea un túnel cifrado entre dos hosts para transferir datos no cifrados por una red insegura. Puerto: **TCP 22**.

**Flujo del ataque:**
1. El atacante escanea servidores SSH con bots (port scan en TCP 22) para identificar vulnerabilidades.
2. Ataque de fuerza bruta para obtener credenciales de acceso.
3. Con las credenciales obtenidas, usa los túneles SSH para transmitir malware o exploits **sin ser detectado** (tráfico cifrado).

**Herramientas:** **Nmap** (descubrimiento/escaneo) + **Ncrack** (brute force) en plataforma Linux.

---

### FTP Brute Force con AI (Hydra)

El examen incluye el comando exacto generado por AI (ChatGPT) para brute force FTP:

```bash
hydra -L /usr/share/wordlists/ftp-usernames.txt -P /usr/share/wordlists/ftp-passwords.txt ftp://10.10.1.11
```

| Flag | Significado |
|---|---|
| `-L <fichero>` | Lista de **usernames** (mayúscula L = fichero de lista) |
| `-P <fichero>` | Lista de **passwords** (mayúscula P = fichero de lista) |
| `ftp://IP` | Protocolo objetivo + IP del target |

*Nota: `-l` (minúscula) = username único; `-p` (minúscula) = password única. El examen puede explotar esta distinción.*

---

### 🔴 HTTP/2 Continuation Flood Attack — Mecanismo paso a paso

Explota el mecanismo de fragmentación de cabeceras HTTP/2 mediante frames CONTINUATION para agotar los recursos de Apache.

**Contexto:** En HTTP/2, las cabeceras demasiado grandes se fragmentan: primera parte en frame `HEADERS`, resto en frames `CONTINUATION`. La secuencia termina cuando se establece el flag `END_HEADERS`.

**Flujo del ataque:**
1. El atacante establece una conexión TCP con el servidor Apache objetivo.
2. Envía un frame `HEADERS` legítimo.
3. El servidor asigna memoria y recursos para procesar el frame.
4. En lugar de completar la secuencia con el flag `END_HEADERS`, el atacante envía frames `CONTINUATION` indefinidamente.
5. Por cada frame recibido, el servidor asigna más memoria y CPU esperando el fin de la secuencia.
6. Los recursos (memoria + CPU) se agotan progresivamente.
7. El servidor se ralentiza, se bloquea o deja de responder → **DoS**.

---

### Frontjacking Attack

Ataque que inyecta o manipula componentes front-end (scripts, HTML) para secuestrar la interfaz de usuario o las interacciones del usuario. Objetivo principal: **servidores Nginx reverse proxy mal configurados** en entornos de hosting compartido.

**Técnicas combinadas:** CRLF injection + HTTP request header injection + XSS.

**Vector específico:** explotación de variables `$uri` y `$document_uri` mal sanitizadas en la configuración del reverse proxy → inyección de un nuevo `Host` header.

**Flujo:**
1. El atacante construye una petición HTTP con caracteres CRLF en la URI para inyectar un `Host` header malicioso.
2. El reverse proxy acepta la petición con el header inyectado.
3. El reverse proxy enruta la petición al servidor controlado por el atacante en lugar del backend legítimo.
4. El servidor del atacante responde con contenido malicioso (phishing, malware, XSS reflejado).

---

### 🔴 Web Server Password Cracking

**Objetivos principales del atacante:**
- Servidores SMTP y FTP
- Web shares
- Túneles SSH
- Autenticación en formularios web

**Técnicas de cracking (orden de potencia creciente):**

| Técnica | Descripción | Limitación |
|---|---|---|
| **Guessing** | Manual o con diccionario; explota contraseñas predecibles | Solo eficaz con contraseñas débiles |
| **Dictionary attack** | Fichero predefinido con palabras; programa las prueba una a una | No eficaz con contraseñas con caracteres especiales |
| **Brute force** | Prueba todas las combinaciones posibles (A-Z, a-z, 0-9, especiales) | Puede tardar meses/años con contraseñas complejas |
| **Hybrid attack** | Combina dictionary + brute force + símbolos + números | El más potente de los cuatro |

**Herramientas de cracking:** **THC Hydra**, **Ncrack**.
**Métodos auxiliares:** social engineering, spoofing, phishing, troyanos/virus, wiretapping, keystroke logging.

---

### DoS/DDoS en Web Servers

Flood de peticiones falsas que hace el servidor no disponible para usuarios legítimos. Objetivos preferentes: servidores bancarios, pasarelas de pago, root name servers.

**Recursos objetivo del ataque:**
- Ancho de banda de red
- Memoria del servidor
- Mecanismo de gestión de excepciones de la aplicación
- Uso de CPU
- Espacio en disco
- Espacio en base de datos

---

### MITM / Sniffing

El atacante intercepta y/o modifica comunicaciones entre usuario y servidor web. El vector: el atacante se hace pasar por un proxy ante la víctima. Si la víctima acepta, todo el tráfico pasa por el atacante → robo de credenciales bancarias, usuarios, contraseñas.

---

### 🔴 Web Application Attacks — Catálogo completo

| Ataque | Mecanismo resumido |
|---|---|
| **SSRF** | Peticiones forjadas desde el servidor web hacia servidores internos/backend |
| **Parameter/Form Tampering** | Manipulación de parámetros cliente-servidor (credenciales, precios, permisos) |
| **Cookie Tampering** | Modificación de cookies persistentes y no persistentes |
| **Unvalidated Input / File Injection** | Input no validado o inyección de ficheros en la aplicación |
| **Session Hijacking** | Robo/predicción del mecanismo de control de sesión → acceso a partes autenticadas |
| **SQL Injection** | Inyección de código malicioso en strings enviados al servidor SQL |
| **Directory Traversal** | Manipulación de URL con `../` para salir del root directory |
| **DoS** | Terminación de operaciones del servidor por saturación |
| **XSS** | Inyección de tags HTML o scripts en el sitio objetivo |
| **Buffer Overflow** | Envío de datos que superan el espacio de almacenamiento → crash o comportamiento vulnerable |
| **CSRF** | Uso de la confianza de un usuario autenticado para enviar código/comandos maliciosos al servidor |
| **Command Injection** | Modificación del contenido de la página web mediante HTML; explotación de campos de formulario sin restricciones |
| **Source Code Disclosure** | Errores tipográficos en scripts o mala configuración (permisos ejecutables) → exposición de credenciales de BBDD y claves secretas |

---

## 2. Exam Traps ⚠️

⚠️ **[DNS Amplification vs. DNS Hijacking]**
Amplification = los bots usan la IP de la víctima para inundar su servidor DNS con respuestas (DDoS por volumen). Hijacking = el atacante modifica la configuración del servidor DNS para redirigir a un servidor falso. El examen presenta los dos como si fueran el mismo ataque. Criterio: ¿el objetivo es inundar con tráfico? → Amplification. ¿El objetivo es redirigir a sitio falso? → Hijacking.

⚠️ **[HTTP/2 Continuation Flood vs. HTTP/2 endless continuation frames (Apache vulnerability)]**
Son el mismo concepto descrito en dos contextos del libro. La vulnerabilidad en la tabla de Apache (chunk 1) y el ataque Continuation Flood (chunk 2) describen el mismo mecanismo: frames CONTINUATION sin `END_HEADERS`. No son ataques distintos.

⚠️ **[Hydra flags: -L/-P vs. -l/-p]**
`-L` (mayúscula) = fichero con lista de usernames. `-P` (mayúscula) = fichero con lista de passwords. `-l` (minúscula) = username único. `-p` (minúscula) = password única. El examen puede mostrar el comando con minúscula y preguntar qué cambia.

⚠️ **[Directory Traversal: secuencia correcta]**
La secuencia es `../` (dot-dot-slash). No es `./` (directorio actual) ni `//` (doble barra). El examen puede presentar variantes incorrectas.

⚠️ **[Web Cache Poisoning: prerequisito obligatorio]**
Cache poisoning **requiere** que existan vulnerabilidades de HTTP response-splitting en el servidor y la aplicación. No es un ataque independiente de response-splitting; es una consecuencia posible de él.

⚠️ **[SSH Brute Force: puerto y herramientas]**
SSH corre en **TCP 22**. Las herramientas del examen son **Nmap** (escaneo) + **Ncrack** (brute force), **en Linux**. No confundir Ncrack con THC Hydra; Hydra aparece para FTP brute force, Ncrack para SSH.

⚠️ **[Frontjacking: servidor objetivo]**
Frontjacking apunta específicamente a **Nginx reverse proxy** mal configurados en hosting compartido. No es un ataque genérico a cualquier servidor web. El vector son las variables `$uri` y `$document_uri`.

⚠️ **[Hybrid attack = dictionary + brute force, no al revés]**
Hybrid attack **combina** dictionary attack y brute force. Si el examen pregunta cuál es el más potente de los cuatro métodos de cracking, la respuesta es **hybrid**, no brute force.

⚠️ **[Defacement: doble objetivo]**
El examen puede presentar defacement únicamente como un ataque reputacional. El libro especifica que también tiene un segundo objetivo: **infectar a los visitantes** haciendo el sitio vulnerable a ataques víricos. Ambos objetivos son correctos.

⚠️ **[SSRF: quién confía en quién]**
En SSRF, el servidor backend responde porque cree que la petición viene del servidor web (mismo segmento de red). El cliente no tiene acceso directo al backend; el abuso es que el servidor web actúa como intermediario involuntario.

---

## 3. Nemotécnicos

**Tipos de mala configuración de servidor web** → **"VASRUNS"**:
- **V**erbose debug/error messages
- **A**nonymous/default users/passwords
- **S**ample config/script files
- **R**emote administration functions
- **U**nnecessary services enabled
- **N**o SSL adecuado (misconfigured/default SSL certificates)
- *(más: directory browsing, unrestricted uploads, missing root location en Nginx)*

**Técnicas de password cracking (orden potencia):** **"G-D-B-H"**
- **G**uessing → **D**ictionary → **B**rute force → **H**ybrid
- Frase: "**Grandes Detectives Buscan Hackers**"

**Recursos objetivo de un DoS en servidor web** → **"BMACHD"**:
- **B**andwidth (network)
- **M**emory
- **A**pplication exception handling
- **C**PU
- **H**ard-disk space
- **D**atabase space

**Flujo DNS Amplification (simplificado):** Bots → IP víctima suplantada → DNS primario → recursión hasta TLD → respuestas van a IP víctima → DDoS

**HTTP/2 Continuation Flood (6 fases):** TCP connect → HEADERS frame → servidor asigna memoria → CONTINUATION frames sin END_HEADERS → servidor acumula recursos → agotamiento → DoS

**Hydra FTP flags:** "**L**ista **U**sernames = **L** mayúscula; **P**asswords = **P** mayúscula"

---

## 4. Flashcards

**Q:** ¿Qué secuencia de caracteres usa un atacante en un directory traversal attack?
**A:** `../` (dot-dot-slash) para navegar fuera del root directory del servidor web.

---

**Q:** ¿En qué puerto TCP corre SSH y qué herramientas usa el CEH para SSH brute force?
**A:** TCP puerto 22. Herramientas: Nmap (escaneo) + Ncrack (brute force), en plataforma Linux.

---

**Q:** ¿Cuál es el prerequisito para que un web cache poisoning attack sea posible?
**A:** El servidor web y la aplicación deben tener vulnerabilidades de HTTP response-splitting.

---

**Q:** En un DNS amplification attack, ¿por qué las respuestas DNS llegan a la víctima y no a los bots?
**A:** Porque los bots suplantan (spoof) la IP de la víctima en las consultas DNS. Las respuestas se envían a la IP origen de la consulta, que es la de la víctima.

---

**Q:** ¿Qué flag de Hydra especifica un fichero de lista de usernames vs. un username único?
**A:** `-L` (mayúscula) = fichero de lista de usernames. `-l` (minúscula) = username único.

---

**Q:** ¿Qué flag final distingue al HTTP/2 Continuation Flood de un intercambio HTTP/2 normal?
**A:** La ausencia del flag `END_HEADERS`. En un flujo normal, el último frame CONTINUATION lleva `END_HEADERS`. El atacante nunca lo envía.

---

**Q:** ¿Qué tipo de servidor es el objetivo principal del frontjacking attack?
**A:** Nginx reverse proxy mal configurados en entornos de hosting compartido.

---

**Q:** ¿Qué variables de configuración de Nginx explota el frontjacking?
**A:** `$uri` y `$document_uri`, cuando no están correctamente sanitizadas.

---

**Q:** ¿Cuál es la técnica de password cracking más potente según el CEH?
**A:** Hybrid attack — combina dictionary attack + brute force + símbolos + números.

---

**Q:** ¿Cuáles son los seis recursos que un DoS/DDoS web server attack intenta consumir?
**A:** Ancho de banda de red, memoria del servidor, mecanismo de gestión de excepciones, CPU, espacio en disco, espacio en base de datos.

---

**Q:** ¿Cuál es la diferencia entre DNS server hijacking y DNS amplification?
**A:** Hijacking: el atacante modifica la configuración del DNS para redirigir a un servidor falso. Amplification: los bots usan la IP de la víctima para generar un flood de respuestas DNS → DDoS.

---

**Q:** ¿Qué dos objetivos tiene el website defacement según el CEH?
**A:** (1) Dañar la reputación de la organización cambiando la apariencia del sitio. (2) Infectar a los visitantes haciendo el sitio vulnerable a ataques víricos.

---

**Q:** En un HTTP response-splitting attack, ¿cuántas respuestas genera el servidor y a quién van?
**A:** Dos respuestas. El atacante controla la primera (puede redirigir al usuario a un sitio malicioso); el navegador descarta las demás.

---

**Q:** ¿Qué ataque web explota la confianza del servidor backend en el servidor web para acceder a recursos internos?
**A:** SSRF (Server-Side Request Forgery) — el backend cree que la petición proviene del servidor web porque están en la misma red.

---

**Q:** ¿Qué misconfiguration de IIS permite subir web shells y derivar en RCE?
**A:** Unrestricted file uploads — el servidor permite subir ficheros sin validación ni controles de seguridad.

---

**Q:** ¿Qué ataque usa las credenciales robadas de la víctima para establecer una sesión legítima con el sitio real?
**A:** Phishing — tras capturar credenciales en el sitio falso, el atacante las usa para autenticarse en el sitio legítimo y realizar operaciones maliciosas.

---

**Q:** ¿Qué misconfiguration de Nginx provoca comportamiento indefinido como listado de directorios?
**A:** La ausencia del bloque `location / {}` (root location missing).

---

**Q:** ¿Qué método de cracking no es efectivo cuando la contraseña contiene caracteres especiales?
**A:** Dictionary attack — solo trabaja con palabras del diccionario sin variaciones de caracteres especiales.

---

## 5. Confusión frecuente

**HTTP Response-Splitting vs. Web Cache Poisoning**
- Response-Splitting: el atacante inyecta CRLF en cabeceras para dividir la respuesta del servidor en dos → afecta a la sesión de un usuario concreto.
- Cache Poisoning: el atacante fuerza al servidor a almacenar contenido malicioso en su caché compartida → afecta a **todos** los usuarios hasta que se purgue la caché.
- Relación: Cache Poisoning **requiere** Response-Splitting como vector. No son independientes.
- Criterio: "un usuario redirigido a sitio malicioso" → Response-Splitting. "Todos los usuarios reciben contenido malicioso desde caché" → Cache Poisoning.

---

**DNS Hijacking vs. DNS Amplification**
- Hijacking: modificación de la configuración del servidor DNS → objetivo = redirección a sitio falso. Ataque dirigido.
- Amplification: bots con IP suplantada inundan el DNS de la víctima con respuestas → objetivo = DDoS. Ataque volumétrico.
- Criterio: "usuario redirigido a URL falsa" → Hijacking. "Servidor DNS inoperativo por flood" → Amplification.

---

**Brute Force vs. Hybrid Attack**
- Brute force: prueba todas las combinaciones posibles de caracteres. Exhaustivo pero lento; puede tardar meses con contraseñas complejas.
- Hybrid: combina diccionario + brute force + símbolos + números. Más potente y eficiente que brute force puro.
- Criterio: "todas las combinaciones posibles sin diccionario" → Brute force. "Combinación de métodos con diccionario de base" → Hybrid.

---

**SSRF vs. CSRF**
- SSRF: el atacante explota el servidor web para que haga peticiones a servidores internos. La víctima es el backend.
- CSRF: el atacante usa la sesión autenticada de un usuario para enviar peticiones maliciosas al servidor en nombre del usuario. La víctima es el usuario autenticado.
- Criterio: "servidor → servidor interno" → SSRF. "Usuario autenticado → acción no deseada en servidor" → CSRF.

---

**Session Hijacking vs. Cookie Tampering**
- Session Hijacking: robo, predicción o negociación del mecanismo de control de sesión para acceder a partes autenticadas. Concepto amplio.
- Cookie Tampering: modificación específica del contenido de cookies (persistentes o no persistentes). Es un vector posible dentro del session hijacking, pero no son equivalentes.
- Criterio: "acceso a sesión autenticada" → Session Hijacking. "Modificación del valor de una cookie concreta" → Cookie Tampering.
