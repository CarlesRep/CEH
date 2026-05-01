# M13_03_WebServerAttackMethodology.md
> Módulo 13 / Subapartado 3 — Web Server Attack Methodology

---

## 1. Conceptos y definiciones

### 🔴 Fases de la metodología de ataque a servidores web

| Fase | Objetivo |
|---|---|
| **1. Information Gathering** | Recopilar información del servidor objetivo (dominio, IP, ASN) |
| **2. Web Server Footprinting / Banner Grabbing** | Extraer datos del sistema: versiones, OS, servicios, nombres de servidor |
| **3. Website Mirroring** | Copiar el sitio web para análisis offline de su estructura |
| **4. Vulnerability Scanning** | Identificar vulnerabilidades y misconfigurations con herramientas automatizadas |
| **5. Session Hijacking** | Capturar o predecir session IDs de sesiones activas para tomar el control |
| **6. Web Server Password Hacking** | Crackear contraseñas mediante brute-force, dictionary, hybrid, rainbow, etc. |

---

### Information Gathering

Primera fase y una de las más críticas. Fuentes: Internet, newsgroups, bulletin boards.

**Herramientas de information gathering:**

| Herramienta | URL | Función principal |
|---|---|---|
| who.is | https://who.is | Whois lookup: domain, IP, WHOIS DB search |
| Whois Lookup | https://whois.domaintools.com | Información de registro de dominio |
| Whois | https://www.whois.com | Consultas whois generales |
| Domain Dossier | https://centralops.net | Información completa de dominio |
| Subdomain Finder | https://pentest-tools.com | Enumeración de subdominios |

**Información obtenida:** domain name, IP address, autonomous system number (ASN).

---

### robots.txt como vector de information gathering

El fichero `robots.txt` lista directorios y ficheros que los web crawlers deben indexar (o excluir). Un `robots.txt` mal escrito puede revelar:
- Ficheros con contraseñas o credenciales
- Directorios ocultos o áreas de membresía
- Enlaces ocultos y estructuras internas

**Acceso:** `URL/robots.txt` en el navegador.
**Descarga:** herramienta `Wget`.

Paradoja de seguridad: incluso si el propietario excluye páginas del indexado, el atacante puede leer el propio `robots.txt` para descubrir exactamente qué está intentando ocultar.

---

### 🔴 Web Server Footprinting / Banner Grabbing — Herramientas

**Objetivo:** obtener server name, server type, OS, versiones de software, running applications, database schema details, account details.

#### Netcat — Banner grabbing
```bash
nc -vv www.moviescope.com 80
GET / HTTP/1.0        # [Enter dos veces]
```
Con AI (HEAD request vía heredoc):
```bash
nc -v 10.10.1.22 80 <<EOF
HEAD / HTTP/1.1
Host: 10.10.1.22
EOF
```

#### Telnet — Banner grabbing
```bash
telnet www.moviescope.com 80
GET / HTTP/1.0        # [Enter dos veces]
```
Limitaciones de seguridad de Telnet: **no cifra datos** + **carece de esquema de autenticación**.
Objetivo: probar el campo `server` en la cabecera HTTP de respuesta (TCP 80).

#### httprecon
Herramienta de fingerprinting avanzado. Realiza los siguientes test cases sobre el servidor:
- GET legítimo a recurso existente
- GET excesivamente largo (URI > 1024 bytes)
- GET a recurso no existente
- HEAD a recurso existente
- Enumeración con OPTIONS (permitido)
- Método DELETE (habitualmente no permitido)
- Método TEST (no definido en el estándar)
- Versión de protocolo HTTP/9.8 (no existe)
- GET con patrones de ataque (`../`, `%%`)

#### Uniscan
Herramienta de fingerprinting versátil: ping, traceroute, nslookup + checks estáticos, dinámicos y de estrés sobre el servidor web. Realiza búsquedas automatizadas en Bing y Google para IPs específicas. Genera informe completo.

#### Footprinting con AI — Comando compuesto
```bash
nmap -sV 10.10.1.22 && whatweb 10.10.1.22 && nikto -h 10.10.1.22
```

| Herramienta | Flag/Opción | Función |
|---|---|---|
| `nmap -sV` | `-sV` = version detection | Detecta puertos abiertos y versiones de servicios |
| `whatweb` | IP target | Identifica tecnologías y frameworks del servidor web |
| `nikto -h` | `-h` = host | Escanea vulnerabilidades: ficheros peligrosos/CGIs, software desactualizado, misconfigurations |

**Herramientas adicionales de footprinting:** Netcraft · ID Serve · Ghost Eye · Skipfish

---

### 🔴 Shodan — IIS Information Gathering

Shodan permite buscar servidores IIS con filtros específicos:

| Filtro Shodan | Propósito |
|---|---|
| `http.title:"IIS"` | Listar todas las instancias IIS |
| `Ssl:"Company Inc." http.title:"IIS"` | IIS con certificados SSL de una organización específica |
| `Ssl.cert.subject.CN:"company.in" http.title:"IIS"` | IIS con CN específico en el certificado → identifica dominio/subdominio |
| `http.title:"IIS Windows Server" country:"US"` | IIS en EEUU (ataques geográficamente dirigidos) |
| `http.title:"IIS7" port:80` | IIS7 en puerto 80 |
| `http.title:"IIS7" net:"<IP>/24"` | IIS7 en un rango de red específico |
| `http.title:"IIS Windows Server"` | IIS en Windows (explotación específica de Windows) |
| `http.title:"Internet Information Services"` | Búsqueda amplia de cualquier IIS |

Datos obtenibles por resultado: IP, puertos abiertos, servicios, versión, HTTP headers, certificados SSL, metadata. Exportación: CSV o JSON.

---

### 🔴 Apache mod_userdir — Enumeración de usuarios con Nmap

`mod_userdir` permite acceder a directorios de usuario con URIs `/~username/`. Si está habilitado, un atacante puede enumerar usernames válidos.

```bash
# Escaneo básico (usa wordlist por defecto: /nselib/data/usernames.lst)
nmap -p80 --script http-userdir-enum <target>

# Escaneo con wordlist personalizada
nmap -p80 --script http-userdir-enum userdir.users=<Wordlist>.txt <target>

# Bypass de detección con user-agent personalizado
nmap -p80 --script http-brute --script-args http.useragent="<User_Agent>" <target>
```

---

### 🔴 Nmap NSE Scripts para web server enumeration

| Comando | Función |
|---|---|
| `nmap --script hostmap-bfk <host>` | Descubrir virtual domains |
| `nmap --script http-trace -p80 localhost` | Detectar servidor vulnerable al método TRACE |
| `nmap --script http-google-email <host>` | Recolectar cuentas de email |
| `nmap -p80 --script http-userdir-enum localhost` | Enumerar usuarios (mod_userdir) |
| `nmap -p80 --script http-waf-detect ...` | Detectar WAF o IPS |
| `nmap --script=http-waf-fingerprint -p80,443 <host>` | Fingerprint de WAF |
| `nmap --script http-enum -p80 <host>` | Enumerar aplicaciones web comunes |
| `nmap -p80 --script http-robots.txt <host>` | Obtener robots.txt |
| `nmap -sV --script http-enum <target_IP>` | Detección de versiones + enumeración web |
| `nmap --script http-passwd --script-args http-passwd.root=/ <target_IP>` | Intentar acceso a /etc/passwd |
| `nmap <target_IP> -p80 --script=http-frontpage-login` | Login en FrontPage |

---

### Default Credentials — Fuentes de consulta

Cuando se identifica la interfaz administrativa del servidor web (mediante port scanning), el atacante busca credenciales por defecto usando:
- Documentación oficial de la interfaz
- Base de datos interna de Metasploit
- Recursos online: **cirt.net** (`https://cirt.net/passwords`), **FortyPoundHead.com**, `defaultpassword.com`, `default-password.info`, `routerpasswords.com`
- Password guessing y brute-force

**cirt.net** = base de datos de contraseñas, credenciales y puertos por defecto.

---

### Default Content — Qué busca el atacante

| Tipo de contenido por defecto | Riesgo |
|---|---|
| Admin debug/test functionality | Contiene info de configuración y estado en tiempo de ejecución del servidor |
| Sample scripts y páginas de demostración | Contienen vulnerabilidades o funcionalidades explotables |
| Funciones administrativas accesibles públicamente | Permiten desplegar web archives, backdoors, command-shell access |
| Manuales de instalación del servidor | Revelan configuración y procedimientos de instalación útiles para el atacante |

**Herramienta para identificar default content:** Nikto2 (`https://cirt.net`).

---

### Directory Brute Forcing

Cuando se solicita un directorio, el servidor responde con: recurso por defecto (ej. `index.html`) / error HTTP 403 / listado del directorio.

Los listados de directorio pueden exponer: improper access controls / acceso no intencionado al web root.

**Herramientas:**
- **Dirhunt** — web crawler para búsqueda y análisis de directorios; detecta false 404 errors, directorios con index vacío creado para ocultar contenido, mode "index of".
- **Sitechecker Website Directory Scanner**

**Directory brute forcing con AI (gobuster):**
```bash
gobuster dir -u https://certifiedhacker.com -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

| Componente | Función |
|---|---|
| `gobuster dir` | Modo brute-force de directorios/ficheros |
| `-u <URL>` | Target URL |
| `-w <wordlist>` | Wordlist para el brute-force |

---

### Vulnerability Scanning

Fase en la que se usan técnicas de sniffing para determinar sistemas activos, servicios de red y aplicaciones, y herramientas automatizadas para identificar vulnerabilidades y misconfigurations.

**Herramienta principal:** Acunetix Web Vulnerability Scanner (WVS).
- Detecta: SQL injection, XSS, y otras vulnerabilidades web.
- Soporta: HTML5, SOAP, AJAX; web forms, password-protected areas, CAPTCHA, SSO, 2FA.
- Detecta: application languages, web server types, smartphone-optimized sites.
- Incluye: port scanning del servidor web.
- Usa: AcuSensor Technology para pen testing manual.

**Herramienta específica para Nginx:** NginxPwner (Python).
```bash
# Crear fichero de rutas
nano /tmp/pathlist
# Ejecutar escaneo
python3 nginxpwner.py <target_URL> /tmp/<pathlist>
```
Detecta: métodos HTTP mal configurados, permisos incorrectos, software desactualizado, configuraciones inseguras por defecto.

**Herramientas adicionales:** OpenText Fortify WebInspect · Tenable.io · ImmuniWeb · Invicti.

---

### Finding Exploitable Vulnerabilities

**Repositorios de exploits:**
- **Exploit Database** (`https://www.exploit-db.com`)
- **Packet Storm** (`https://packetstormsecurity.com`)

**Flujo en Exploit Database:**
1. Buscar por keyword (Apache, IIS, nginx)
2. Filtrar por Type: "Webapps", Platform (Windows/Linux), Date
3. Revisar: descripción, versión afectada, tipo de exploit
4. Ver detalles: exploit code, description, references
5. Verificar versiones/configuraciones del target
6. Descargar y probar exploit en entorno controlado

**Comando AI para finding exploits:**
```bash
sudo apt-get update && sudo apt-get install nmap -y && \
nmap -sV -O 10.10.1.19 -oX nmap_scan.xml && \
sudo apt-get install exploitdb -y && \
searchsploit-nmap nmap_scan.xml
```
- `nmap -sV -O`: service-version detection + OS detection → resultado en XML
- `searchsploit-nmap`: cruza los resultados del scan con la base de datos de exploits

---

### Session Hijacking

Técnicas para capturar/predecir session IDs: session token prediction, session replay, session fixation, sidejacking, XSS.

**Herramientas:**

| Herramienta | Función clave |
|---|---|
| **Burp Suite** | Herramienta de web security testing; el módulo **Sequencer** testea la aleatoriedad de session tokens para predecir el siguiente token válido |
| **JHijack** | Session hijacking automatizado |
| **Ettercap** | Session hijacking automatizado |

---

### 🔴 Web Server Password Hacking — Técnicas completas

Técnicas disponibles en esta fase (más completas que en la sección de ataques):
- Guessing · Dictionary · Brute-force · Hybrid
- **Precomputed hashes**
- **Rule-based attacks**
- **Distributed network attacks**
- **Rainbow attacks**

**Herramientas:**

| Herramienta | Características clave |
|---|---|
| **Hashcat** | Multi-OS, multi-platform; multi-hash (MD4, MD5, SHA-224/256/384/512, RIPEMD-160); modos: straight, combination, brute-force, hybrid dict+mask, hybrid mask+dict |
| **THC Hydra** | Login cracker paralelizado; soporta protocolos: FTP, HTTP-FORM-GET/POST, HTTP-GET/HEAD/POST, HTTPS-*, SSH v1/v2, RDP, SMB, SMTP, SNMP v1+v2+v3, MySQL, MSSQL, LDAP, VNC, Telnet, IMAP, POP3, entre muchos otros |
| **Ncrack** | Cracking de red, complementa a Nmap |
| **Rainbow Crack** | Ataques de rainbow table |
| **Wfuzz** | Fuzzing web |
| **Wireshark** | Captura de credenciales en tráfico de red |

---

### Using Application Server as Proxy

Servidores web configurados como forward proxy o reverse proxy pueden ser usados por atacantes para:
- Atacar sistemas de terceros en Internet
- Conectar a hosts arbitrarios en la red interna de la organización
- Conectar a otros servicios en el propio host proxy

Método: peticiones **GET** y **CONNECT** a través del servidor vulnerable.

---

### Path Traversal vía Nginx Alias Misconfiguration (Kyubi)

Misconfiguration en `nginx.conf`: directiva `alias` **sin trailing slash**.

```nginx
# Configuración vulnerable
location /i {
    alias /data/w3/images/;   # sin trailing slash en /i → vulnerable
}
```

El atacante añade `../` a la URL para navegar fuera del directorio asignado.

**Herramienta de explotación:** Kyubi (Python)
```bash
kyubi -v <target_URL>
```
Kyubi realiza brute force recursivo de rutas URL para detectar ficheros y directorios expuestos por misconfiguración del alias.

**Impacto:** acceso a ficheros de configuración, credenciales, robo de datos, defacement, compromiso total del servidor.

---

## 2. Exam Traps ⚠️

⚠️ **[robots.txt: propósito vs. riesgo]**
El robots.txt está diseñado para decirle a los crawlers qué NO indexar. El examen puede presentarlo como mecanismo de seguridad → incorrecto. Es una lista de exclusión, no un control de acceso. El atacante puede leerlo directamente para descubrir exactamente los directorios que el propietario quiere ocultar.

⚠️ **[Telnet: limitaciones de seguridad exactas]**
Dos limitaciones específicas según el libro: (1) no cifra datos, (2) carece de esquema de autenticación. El examen puede añadir "no soporta compresión" o similares como distractor.

⚠️ **[httprecon: resultado vs. herramienta genérica]**
httprecon es específicamente para **web server fingerprinting avanzado** mediante análisis de cabeceras y comportamiento del servidor. No es un vulnerability scanner genérico. Realiza test cases con métodos HTTP no estándar (TEST, DELETE) y versiones inexistentes (HTTP/9.8).

⚠️ **[Nmap -sV vs. -O]**
`-sV` = service/version detection. `-O` = OS detection. En el comando AI del libro (`nmap -sV 10.10.1.22`) solo aparece `-sV`. El comando de finding exploits usa ambos (`-sV -O`). El examen puede preguntar cuál flag detecta el OS.

⚠️ **[Hashcat: hash types soportados]**
MD4, MD5, SHA-224, SHA-256, SHA-384, SHA-512, RIPEMD-160. El examen puede incluir SHA-1 como distractor (no aparece en la lista del libro) o preguntar cuál NO está soportado.

⚠️ **[THC Hydra: es un login cracker paralelizado, no un port scanner]**
Hydra ataca protocolos de autenticación. El examen puede confundirlo con Ncrack (que también crackea pero es de Nmap project) o con Nmap. Hydra soporta explícitamente SSH v1 Y v2, y SNMP v1+v2+v3.

⚠️ **[Burp Suite Sequencer: función específica]**
El módulo Sequencer de Burp Suite testea la **aleatoriedad** de session tokens para predecir el siguiente token válido. No es un módulo de fuzzing ni de scanning de vulnerabilidades.

⚠️ **[Nginx alias misconfiguration: condición exacta]**
La vulnerabilidad existe cuando la directiva `alias` se usa **sin trailing slash en el bloque location**. No es un fallo de Nginx en general; es una misconfiguration específica del `nginx.conf`. Herramienta de explotación: Kyubi.

⚠️ **[Gobuster: es directory brute forcing, no vulnerability scanning]**
El examen puede categorizar gobuster como vulnerability scanner. Es específicamente una herramienta de directory/file brute-forcing. El vulnerability scanner del módulo es Acunetix.

⚠️ **[Fases de la metodología: orden y nombre exactos]**
Las 6 fases en orden: Information Gathering → Web Server Footprinting → Website Mirroring → Vulnerability Scanning → Session Hijacking → Web Server Password Hacking. El examen puede presentarlas desordenadas o incluir una fase inexistente como distractor.

⚠️ **[cirt.net: función específica]**
cirt.net es una base de datos de **contraseñas por defecto, credenciales y puertos**. No es un repositorio de exploits (eso es Exploit DB / Packet Storm). Nikto2 también está alojado en cirt.net pero es un vulnerability scanner, no una base de datos de contraseñas.

---

## 3. Nemotécnicos

**6 fases de la metodología (en orden)** → **"IG-FP-WM-VS-SH-PH"**:
- **I**nformation **G**athering → **F**oot**P**rinting → **W**ebsite **M**irroring → **V**ulnerability **S**canning → **S**ession **H**ijacking → **P**assword **H**acking

**Modos de ataque de Hashcat (5)** → **"S-C-B-HD-HM"**:
- **S**traight · **C**ombination · **B**rute-force · **H**ybrid **D**ict+mask · **H**ybrid **M**ask+dict

**Hash types de Hashcat** → **"MD(4,5) + SHA(224,256,384,512) + RIPE160"**

**Test cases de httprecon (9)** → **"GET-exist / GET-long / GET-noexist / HEAD / OPTIONS / DELETE / TEST / HTTP9.8 / GET-attack"**

**Herramientas de session hijacking** → **"B-J-E"**: Burp Suite · JHijack · Ettercap

**Filtros Shodan IIS clave** → **"Title / SSL-org / SSL-CN / Country / Port / Net"**

**Password hacking: técnicas ampliadas (8)** → añadir a G-D-B-H las 4 nuevas:
- Guessing · Dictionary · Brute-force · Hybrid · **Precomputed hashes** · **Rule-based** · **Distributed network** · **Rainbow**

**Herramientas footprinting web** → **"NC-T-HR-UN + Netcraft-IDServe-Nmap-Ghost-Skip"**:
- **N**et**C**at · **T**elnet · **H**ttp**R**econ · **UN**iscan (las 4 principales del chunk)

---

## 4. Flashcards

**Q:** ¿Cuáles son las 6 fases de la metodología de ataque a servidores web en orden?
**A:** Information Gathering → Web Server Footprinting → Website Mirroring → Vulnerability Scanning → Session Hijacking → Web Server Password Hacking.

---

**Q:** ¿Qué información puede revelar un robots.txt mal escrito al atacante?
**A:** Contraseñas, direcciones de email, enlaces ocultos, áreas de membresía y la ubicación exacta de directorios y ficheros restringidos.

---

**Q:** ¿Cuáles son los dos comandos para banner grabbing con Netcat en www.moviescope.com?
**A:** `nc -vv www.moviescope.com 80` seguido de `GET / HTTP/1.0` (Enter dos veces).

---

**Q:** ¿Cuáles son las dos limitaciones de seguridad de Telnet según el CEH?
**A:** (1) No cifra los datos enviados por la conexión. (2) Carece de esquema de autenticación.

---

**Q:** ¿Qué hace el módulo Sequencer de Burp Suite?
**A:** Testea la aleatoriedad de session tokens para permitir al atacante predecir el siguiente token válido y tomar el control de una sesión activa.

---

**Q:** ¿Cuál es el comando compuesto generado por AI para web server footprinting?
**A:** `nmap -sV 10.10.1.22 && whatweb 10.10.1.22 && nikto -h 10.10.1.22`

---

**Q:** ¿Qué flag de Nmap activa la detección de versiones de servicios?
**A:** `-sV` (service/version detection).

---

**Q:** ¿Qué filtro de Shodan lista todos los servidores IIS en EEUU?
**A:** `http.title:"IIS Windows Server" country:"US"`

---

**Q:** ¿Qué filtro de Shodan localiza IIS7 en un rango de red específico /24?
**A:** `http.title:"IIS7" net:"<IP_address>/24"`

---

**Q:** ¿Qué herramienta explota misconfigurations de la directiva alias en Nginx?
**A:** Kyubi (`kyubi -v <target_URL>`) — detecta y explota path traversal por alias sin trailing slash.

---

**Q:** ¿Qué misconfiguration exacta de nginx.conf permite path traversal vía alias?
**A:** Uso de la directiva `alias` sin trailing slash en el bloque `location`.

---

**Q:** ¿Cuáles son los 5 modos de ataque de Hashcat?
**A:** Straight, combination, brute-force, hybrid dict+mask, hybrid mask+dict.

---

**Q:** ¿Qué hash types soporta Hashcat?
**A:** MD4, MD5, SHA-224, SHA-256, SHA-384, SHA-512, RIPEMD-160 (entre otros).

---

**Q:** ¿Qué comando Nmap enumera usuarios en Apache con mod_userdir usando una wordlist personalizada?
**A:** `nmap -p80 --script http-userdir-enum userdir.users=<Wordlist>.txt <target>`

---

**Q:** ¿Qué herramienta se usa para gobuster directory brute-forcing y cuál es su sintaxis básica?
**A:** `gobuster dir -u <URL> -w <wordlist>`

---

**Q:** ¿Para qué sirve NginxPwner y cómo se ejecuta?
**A:** Herramienta Python para identificar misconfigurations y vulnerabilidades en Nginx. Se ejecuta con: `python3 nginxpwner.py <target_URL> /tmp/<pathlist>`

---

**Q:** ¿Cuáles son los repositorios de exploits que menciona el CEH para finding exploitable vulnerabilities?
**A:** Exploit Database (`exploit-db.com`) y Packet Storm (`packetstormsecurity.com`).

---

**Q:** ¿Qué test case específico realiza httprecon para forzar comportamiento con un método HTTP no definido?
**A:** Envía el método `TEST` (no definido en el estándar HTTP) para analizar la respuesta del servidor.

---

**Q:** ¿Qué comandos GET y CONNECT usa un atacante cuando emplea un servidor web como proxy?
**A:** Peticiones `GET` y `CONNECT` al servidor proxy vulnerable para conectar a sistemas internos, terceros en Internet, u otros servicios del propio host.

---

**Q:** ¿Cuáles son las 8 técnicas de password hacking en la fase de Web Server Password Hacking?
**A:** Guessing, dictionary, brute-force, hybrid, precomputed hashes, rule-based, distributed network attacks, rainbow attacks.

---

## 5. Confusión frecuente

**Netcat vs. Telnet para banner grabbing**
- Netcat: herramienta de networking genérica (TCP/IP), "back-end tool", usable por scripts. Flag `-vv` para verbose. Puede usarse con heredoc para enviar requests HTTP complejos.
- Telnet: protocolo cliente-servidor; no cifra, sin autenticación. Limitado a conexiones interactivas. El examen puede preguntar cuál tiene limitaciones de seguridad → Telnet (las dos del libro). Netcat no tiene esas limitaciones documentadas en este contexto.

---

**Acunetix vs. NginxPwner**
- Acunetix: vulnerability scanner de propósito general para aplicaciones y servidores web (SQL injection, XSS, HTML5, SOAP, AJAX, port scanning).
- NginxPwner: herramienta específica exclusivamente para vulnerabilidades y misconfigurations de Nginx.
- Criterio: "vulnerability scanner general" → Acunetix. "Nginx específico" → NginxPwner.

---

**cirt.net vs. Exploit Database**
- cirt.net: base de datos de **contraseñas por defecto**, credenciales y puertos. También aloja Nikto.
- Exploit Database: repositorio de **exploits** para vulnerabilidades conocidas. Con filtros por tipo, plataforma y fecha.
- Criterio: "buscar credenciales por defecto de un panel admin" → cirt.net. "Buscar exploit para una versión de Apache" → Exploit Database.

---

**Dirhunt vs. Gobuster**
- Dirhunt: web crawler orientado a análisis de directorios. Detecta false 404 errors, index vacíos. Útil aunque el directory listing esté deshabilitado.
- Gobuster: brute-force puro de directorios/ficheros contra una URL con una wordlist.
- Criterio: "crawling inteligente de directorios con análisis de comportamiento" → Dirhunt. "Brute-force sistemático con wordlist" → Gobuster.

---

**Hashcat vs. THC Hydra**
- Hashcat: crackea **hashes** almacenados (offline). Multi-hash, multi-device. Modos: straight, combination, brute-force, hybrid.
- THC Hydra: crackea credenciales **online** atacando protocolos de autenticación en tiempo real (FTP, SSH, HTTP, RDP, SMB, etc.). Login cracker paralelizado.
- Criterio: "crackear un fichero de hashes robado" → Hashcat. "Ataque de fuerza bruta en tiempo real contra un servicio" → THC Hydra.
