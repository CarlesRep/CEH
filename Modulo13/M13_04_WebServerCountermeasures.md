# M13_04_WebServerCountermeasures.md
> Módulo 13 / Subapartado 4 — Web Server Attack Countermeasures

---

## 1. Conceptos y definiciones

### Arquitectura de red segura para servidores web

La arquitectura ideal tiene **tres segmentos**:

| Segmento | Contenido | Función |
|---|---|---|
| **Internet segment** | Red pública externa | Tráfico entrante de usuarios |
| **DMZ** (Demilitarized Zone / Secure Server Security Segment) | Servidores web | Zona aislada entre Internet e intranet |
| **Internal network** | Aplicaciones, BBDD, sistemas internos | Red corporativa protegida |

La colocación del servidor web en la DMZ añade barreras entre el servidor y la red interna y entre el servidor y la red pública. Los firewalls aplican reglas de acceso en ambas fronteras. La segmentación de red asegura que el compromiso de un segmento no implica el compromiso de otros.

---

### 🔴 Countermeasures: Patches y Updates

Lista completa para patch management seguro:
- Escanear vulnerabilidades existentes; parchear y actualizar regularmente.
- Revisar y hacer peer review de toda la documentación antes de aplicar hotfix o security patch.
- Aplicar todas las actualizaciones "as-needed" (según necesidad).
- Probar service packs y hotfixes en entorno no productivo representativo antes de producción.
- Asegurar consistencia de hotfixes y niveles de security patch en todos los Domain Controllers (DCs).
- Planificar interrupciones del servicio; tener backup tapes y emergency repair disks disponibles.
- Mantener un **back-out plan** para revertir al estado previo ante una implementación fallida.
- Deshabilitar todos los mapeos de extensiones de script no utilizados.
- Evitar configuraciones por defecto.
- Usar **virtual patches** para capacidades adicionales de identificación/logging.
- Establecer un plan de disaster recovery para fallos en patch management.
- Realizar risk assessment para determinar qué segmentos parchear primero.
- Inventario detallado de todos los endpoints, servicios y dependencias.
- Desplegar siempre en entorno de pruebas antes del sistema completo.
- Desplegar sistema de alertas para patches.
- Usar herramientas de patch management automatizado (ej. **SolarWinds Patch Manager**).
- Monitorización y reporting periódico del proceso de patch management.
- Limitar el número de versiones de software para reducir exposición a riesgos de terceros.
- Validar y documentar todas las operaciones de patch y actualización.
- Integrar el patch management como parte del **SDLC**.

---

### 🔴 Countermeasures: Protocolos

- Bloquear todos los puertos innecesarios, tráfico ICMP y protocolos innecesarios.
- Hardening del TCP/IP stack; aplicar últimos patches.
- Para protocolos inseguros (Telnet, SMTP, FTP): usar políticas **IPsec**.
- Acceso remoto: asegurar con protocolos de tunneling y cifrado.
- Usar **TLS/SSL** para comunicación con el servidor web.
- Servidores FTP anónimos: deben operar en una parte "inocua" del árbol de directorios, separada del árbol del servidor web.
- Configurar correctamente el **HTTP service banner** para ocultar OS version y tipo.
- Aislar servidores de soporte (ej. LDAP) de la subred local; filtrar tráfico mediante firewall.
- Usar **FTPS** para transferencias de ficheros (no FTP).
- Redirigir todo el tráfico HTTP a **HTTPS**.
- Usar cabeceras **HSTS** para forzar conexiones seguras y prevenir downgrade attacks.
- Automatizar la renovación de certificados SSL/TLS para evitar certificados expirados.
- Implementar **rate-limiting** para mitigar DDoS que ataquen el SSL/TLS handshake.

---

### 🔴 Countermeasures: Cuentas

- Eliminar módulos y extensiones de aplicación no utilizados.
- Deshabilitar cuentas de usuario por defecto creadas durante la instalación del OS.
- Conceder permisos NTFS mínimos (least privilege) a usuarios anónimos de IIS.
- Eliminar usuarios de BBDD y stored procedures innecesarios; aplicar principio de least privilege en la aplicación de BBDD para defenderse contra SQL query poisoning.
- Usar NTFS permissions, web permissions y .NET Framework access control (URL authorization).
- Política de contraseñas fuertes + audits y alertas para fallos de login (ralentiza brute-force y dictionary attacks).
- Ejecutar procesos con cuentas de mínimos privilegios.
- Limitar el acceso admin/root al mínimo número de usuarios; mantener registro.
- Logs de actividad de usuarios en forma cifrada en el servidor o en una máquina separada en la intranet.
- Deshabilitar cuentas no interactivas que no requieren login interactivo.
- Usar VPN seguras (ej. **OpenVPN**) para acceso a plataformas multi-servidor.
- Usar gestores de contraseñas (ej. **KeePass**).
- Habilitar **Separation of Duties (SoD)** en la configuración del servidor.
- Política de expiración de contraseñas periódica.
- Habilitar bloqueo de cuenta tras un número límite de intentos fallidos de login.
- Implementar **2FA/MFA**.
- Usar **CAPTCHA** en páginas de login y registro para prevenir ataques de bots automatizados.
- Preguntas de seguridad con respuestas impredecibles como factor adicional.
- Usar algoritmos de hashing **one-way** fuertes: **bcrypt, scrypt, Argon2** para almacenar contraseñas.
- Diseñar procesos de recuperación de cuentas seguros que verifiquen identidad sin exponer la cuenta a takeover.

---

### 🔴 Countermeasures: Ficheros y Directorios

- Eliminar ficheros innecesarios dentro de ficheros `.jar`.
- Eliminar información de configuración sensible del byte code.
- Evitar mapear directorios virtuales entre dos servidores diferentes o sobre una red.
- Monitorizar y verificar frecuentemente: network services logs, website access logs, database server logs (SQL Server, MySQL, Oracle) y OS logs.
- Deshabilitar el servicio de directory listings.
- Eliminar ficheros no web: archivos, backups, ficheros de texto, headers/include files.
- Deshabilitar el servicio de ciertos tipos de fichero mediante resource map.
- Almacenar ficheros y scripts de aplicaciones web en una partición o unidad separada del OS, logs y ficheros de sistema.
- Ejecutar el servidor web dentro de un **sandbox directory**.
- Excluir metacaracteres al procesar inputs de usuarios.
- Emplear **file integrity checkers** para verificar contenido web y detección de intrusiones.
- Si la aplicación permite file uploads: escanear para malware y almacenar fuera del web root.
- Usar **WAF** contra SQL injection y otros ataques web que puedan llevar a acceso no autorizado a ficheros.
- Usar **SFTP** en lugar de FTP para cifrar transferencias.
- Asegurar que los ficheros de configuración (`.htaccess`, `web.config`) no sean accesibles desde la web.
- Implementar **version control** para ficheros de aplicaciones web.

---

### Detección de intentos de hacking — WDS

Cuando un atacante instala un backdoor, el tamaño de los ficheros infectados aumenta automáticamente.

**Website Change Detection System (WDS):** script que corre en el servidor, compara periódicamente los **hash values** de los ficheros con sus **master hash values** de referencia. Detecta cambios en: HTML, JavaScript, PHP, ASP, Perl, Python. Si detecta cambio → alerta al usuario.

Herramienta ejemplo: **DirectoryMonitor** — recorre carpetas web, detecta cambios en el codebase, alerta por email.

---

### 🔴 Defensa contra ataques — Áreas específicas

#### Puertos
- Monitorizar todos los puertos regularmente.
- No permitir acceso público a **puerto 80 (HTTP)** ni **puerto 443 (HTTPS)** sin restricciones.
- Puerto 80 abierto sin control → vulnerable a DoS.
- Tráfico de intranet: cifrado o restringido.
- Defensa contra IP spoofing: regla "deny this IP" en firewall o comando **"routed blackhole"**.

#### Server Certificates (contra MITM)
- Validación directa de certificados.
- Usar protocolo que no dependa de terceros para validación.
- Verificar que los rangos de fechas del certificado son válidos y que el certificado se usa para su propósito previsto.
- Verificar que el certificado **no ha sido revocado** y que la clave pública es válida hasta una root authority de confianza.

#### machine.config (.NET)
Mecanismo de seguridad a nivel de máquina para aplicaciones .NET:
- Mapear recursos protegidos a `HttpForbiddenHandler`; eliminar `HttpModules` no utilizados.
- Deshabilitar tracing: `<trace enable="false"/>`.
- Desactivar debug compiles.
- Verificar que los errores de ASP.NET no se reenvían al cliente.
- Verificar la configuración del estado de sesión.

#### Code Access Security (IIS)
- Implementar secure coding practices (evita source-code disclosure e input validation attacks).
- Restringir la política de code access security: sin permisos para ejecutar código descargado de Internet o intranet.
- Configurar IIS para rechazar URLs con `../` (previene path traversal).
- Lockdown de comandos del sistema con ACLs restrictivas.
- Instalar nuevos patches y actualizaciones.

---

### 🔴 Defensa contra HTTP Response Splitting y Web Cache Poisoning

**Regla core:** al establecer cookies, **eliminar CRs y LFs** antes de insertar datos en cabeceras HTTP de respuesta.

| Rol | Medidas |
|---|---|
| **Server Admin** | Usar la última versión del software web; actualizar/parchear OS y servidor regularmente; ejecutar vulnerability scanner |
| **Application Developers** | Restringir acceso de la aplicación web a IPs únicas; **bloquear CR (`%0d` o `\r`) y LF (`%0a` o `\n`)**; cumplir RFC 2616 (HTTP/1.1); parsear todos los inputs antes de usarlos en cabeceras HTTP |
| **Proxy Servers** | Evitar compartir conexiones TCP entrantes entre diferentes clientes; usar diferentes conexiones TCP para diferentes virtual hosts; implementar correctamente "maintain request host header" |

Técnica adicional: **UDP source port randomization** contra blind response forgery. Limitar número de recursive queries simultáneas; aumentar TTLs de registros legítimos.

---

### 🔴 Defensa contra DNS Hijacking

Medidas clave:
- Elegir registrar acreditado por **ICANN**; solicitar **REGISTRAR-LOCK** en el dominio.
- Proteger la información de la cuenta del registrant.
- Incluir DNS hijacking en incident response y business continuity planning.
- Usar DNS monitoring tools/services con alertas sobre cambios de IP.
- No descargar codecs y downloaders de sitios no confiables.
- Instalar y actualizar antivirus.
- **Cambiar la contraseña por defecto del router**.
- Restringir zone transfers; usar script blockers en el navegador.
- **DNSSEC**: capa adicional de seguridad sobre DNS que previene su compromiso.
- Configurar arquitectura **master–slave DNS**: master sin acceso a Internet; dos servidores slave (si uno es comprometido, solo se actualiza cuando recibe actualización del master).
- Monitorización constante de servidores DNS.
- Cambiar username y password por defecto del router; mantener firmware actualizado.
- Usar servicios **VPN** seguros (no VPN gratuitos que puedan rastrear actividad).
- Usar **DoH (DNS over HTTPS)** o **DoT (DNS over TLS)** para cifrar consultas DNS.
- Usar filtros DNS o proveedores DNS seguros (Cloudflare, Google Public DNS).
- Implementar **MFA y RBAC** para el acceso a interfaces de gestión DNS.
- Restringir acceso a servidores DNS mediante lista de IPs de confianza.
- Usar **ACLs** para restringir quién puede hacer consultas DNS.
- Verificación por **geolocalización** para detectar patrones de acceso inusuales a interfaces de gestión DNS.

---

### 🔴 Herramientas de seguridad — Clasificación por función

#### Web Application Security Scanners
| Herramienta | Función destacada |
|---|---|
| **Syhunt Hybrid** | Automatiza security testing; detecta XSS, directory traversal, fault injection, SQL injection, command execution; analiza JS; firma vulnerabilidades de aplicación |
| **N-Stalker X** | Busca clickjacking, SQL injection, XSS; spider crawling; web macros para form authentication; capacidades de proxy |
| **OWASP ZAP** | Proxy de ataque web open-source |
| **Burp Suite** | Web security testing; session hijacking, fuzzing |
| **SonarQube** | Análisis de calidad y seguridad de código |
| Invicti, Wapiti, WebScarab, WPSec, Skipfish, Detectify, Arachni, Tinfoil, OpenText Fortify On Demand | Scanners adicionales |

#### Web Server Security Scanners
| Herramienta | Función destacada |
|---|---|
| **Qualys Community Edition** | Descubrimiento de IT assets, vulnerability management, web app scanning, cloud assets inventory |
| **Nikto** | Vulnerability scanner para servidores web (hosted en cirt.net) |
| Observatory, WordPress Security Scan, Web Vulnerability Scanner, ImmuniWeb | Scanners adicionales |

#### Web Server Malware Infection Monitoring
| Herramienta | Función destacada |
|---|---|
| **QualysGuard Malware Detection** | Escaneo proactivo de sitios web para malware; alertas automatizadas e informes |
| Sucuri Site Check, SiteLock, Quttera, Web Inspector, SiteGuarding | Herramientas adicionales |

#### Web Server Security Tools
| Herramienta | Función destacada |
|---|---|
| **OpenText Fortify WebInspect** | DAST automatizado; descubre misconfigurations; identifica y prioriza vulnerabilidades en aplicaciones en ejecución; imita técnicas de hacking reales |
| Acunetix, NetIQ Secure Configuration Manager, SAINT Security Suite, Sophos Intercept X for Server, UpGuard | Herramientas adicionales |

#### Web Server Pen Testing Tools
| Herramienta | Función destacada |
|---|---|
| **CORE Impact** | Evalúa postura de seguridad con técnicas reales de ciberdelincuentes; importa resultados de scan; ejecuta exploits; cubre servidores de red, workstations, firewalls, routers y aplicaciones |
| **Cobalt Strike** | Framework de pen testing y C2 |
| Fuxploider, Mitmproxy | Herramientas adicionales |

---

## 2. Exam Traps ⚠️

⚠️ **[DMZ: nombre alternativo]**
La DMZ se denomina también en el libro **"secure server security segment"**. El examen puede usar ambos términos. El servidor web va en la DMZ, no en la red interna ni directamente en el segmento de Internet.

⚠️ **[Back-out plan: cuándo se necesita]**
El back-out plan permite revertir al estado previo ante una **implementación fallida**, no ante un ataque. El examen puede confundirlo con el disaster recovery plan (que se activa ante fallos en patch management) o con el business continuity plan.

⚠️ **[SolarWinds Patch Manager: categoría]**
Es una herramienta de **patch management automatizado**, no un vulnerability scanner ni un SIEM. El examen puede presentarla en contextos de scanning.

⚠️ **[FTPS vs. SFTP]**
En countermeasures de protocolos: usar **FTPS** para transferencias de ficheros en general (FTP con TLS). En countermeasures de ficheros y directorios: usar **SFTP** para cifrar transferencias. Son dos recomendaciones distintas en secciones distintas del mismo capítulo. FTPS = FTP + TLS/SSL. SFTP = SSH File Transfer Protocol (distinto protocolo).

⚠️ **[Hashing passwords: algoritmos correctos]**
Algoritmos recomendados por el libro: **bcrypt, scrypt, Argon2**. No MD5, no SHA-1 (son rápidos y no adecuados para contraseñas). El examen puede presentar SHA-256 como opción válida para almacenamiento de contraseñas → incorrecto según este módulo.

⚠️ **[WDS: mecanismo de detección]**
El WDS detecta backdoors comparando **hash values** de los ficheros con sus **master hash values**. No es un IDS de red, no analiza tráfico. Detecta cambios en ficheros ejecutables y presencia de ficheros nuevos (HTML, JS, PHP, ASP, Perl, Python).

⚠️ **[machine.config: scope]**
El fichero `machine.config` afecta a **todas las aplicaciones** en la máquina (machine-level settings .NET). No es equivalente a `web.config` (que afecta a una aplicación concreta). El examen puede confundirlos.

⚠️ **[Defensa HTTP Response Splitting: caracteres a bloquear]**
Los caracteres exactos: **CR (`%0d` o `\r`) y LF (`%0a` o `\n`)**. El examen puede presentar solo uno de los dos o usar codificaciones diferentes. Hay que conocer ambas representaciones (URL-encoded y literal).

⚠️ **[DNS Hijacking: REGISTRAR-LOCK]**
El REGISTRAR-LOCK lo proporciona el registrar acreditado por ICANN, no el administrador directamente. El examen puede preguntar a quién se solicita → al registrar.

⚠️ **[DNSSEC vs. DoH/DoT]**
DNSSEC añade una capa de seguridad sobre DNS que previene su compromiso (autenticación de registros DNS). DoH y DoT cifran las **consultas DNS** para prevenir interceptación/redirección. Son mecanismos complementarios con propósitos distintos.

⚠️ **[master–slave DNS: configuración exacta]**
Master **sin acceso a Internet** + **dos** servidores slave. Si un slave es comprometido, solo se actualiza cuando recibe actualización del master. El examen puede cambiar el número de slaves (un slave) o el acceso del master (con Internet).

⚠️ **[Fortify WebInspect: categoría correcta]**
WebInspect es una herramienta de **DAST (Dynamic Application Security Testing)** automatizado. No es un SAST ni un vulnerability scanner de red. Aparece tanto en "Web Server Security Tools" como mencionado previamente en el módulo en "vulnerability scanning".

---

## 3. Nemotécnicos

**3 segmentos de red segura** → **"I-D-N"**: **I**nternet → **D**MZ → **N**etwork (interna)

**Algoritmos hash seguros para contraseñas** → **"B-S-A"**: **B**crypt · **S**crypt · **A**rgon2
(regla: los tres empiezan por consonante, son lentos por diseño → seguros para contraseñas)

**Countermeasures de protocolos — puntos clave** → **"B-H-T-F-H-R"**:
- **B**loquear puertos/ICMP innecesarios
- **H**ardening TCP/IP stack
- **T**LS/SSL para comunicación
- **F**TPS para transferencias
- **H**TTP → HTTPS redirect
- **H**STS headers + **R**ate-limiting

**Defensa HTTP Response Splitting por rol** → **"S-A-P"**:
- **S**erver admin (software actualizado, vulnerability scanner)
- **A**pplication developers (bloquear CR/LF, cumplir RFC 2616, parsear inputs)
- **P**roxy servers (no compartir TCP connections entre clientes)

**Defensa DNS Hijacking — puntos core** → **"ICANN-LOCK + DNSSEC + master-slave(2) + DoH/DoT + MFA/RBAC"**

**WDS: cómo funciona** → "Compara **hashes** actuales con **master hashes** → detecta cambios → alerta"

**Clasificación de herramientas (4 categorías)**:
- Web **App** Security Scanners → Syhunt, N-Stalker, OWASP ZAP
- Web **Server** Security Scanners → Qualys, Nikto
- **Malware** Monitoring → QualysGuard Malware Detection
- **Pen Testing** → CORE Impact, Cobalt Strike

---

## 4. Flashcards

**Q:** ¿Cuáles son los tres segmentos de una arquitectura de red segura para servidores web?
**A:** Internet segment, DMZ (Secure Server Security Segment) y Internal network. El servidor web se ubica en la DMZ.

---

**Q:** ¿Qué es un back-out plan y cuándo se activa?
**A:** Plan que permite revertir el sistema a su estado previo ante una implementación fallida de un patch o actualización.

---

**Q:** ¿Qué herramienta de patch management automatizado menciona el CEH?
**A:** SolarWinds Patch Manager.

---

**Q:** ¿Cuáles son los tres algoritmos de hashing recomendados por el CEH para almacenar contraseñas de forma segura?
**A:** bcrypt, scrypt y Argon2 (one-way hashing algorithms fuertes).

---

**Q:** ¿Cómo funciona un Website Change Detection System (WDS)?
**A:** Compara periódicamente los hash values de los ficheros del servidor con sus master hash values. Si detecta un cambio o un fichero nuevo (HTML, JS, PHP, ASP, Perl, Python), alerta al usuario.

---

**Q:** ¿Qué herramienta WDS de ejemplo menciona el CEH y cómo alerta?
**A:** DirectoryMonitor — recorre carpetas web, detecta cambios en el codebase y alerta por email.

---

**Q:** ¿Qué hace la configuración machine.config y a qué nivel actúa?
**A:** Proporciona un mecanismo de seguridad a nivel de máquina (.NET framework) que afecta a todas las aplicaciones. Permite deshabilitar tracing, debug compiles, evitar que errores ASP.NET lleguen al cliente y gestionar HttpModules.

---

**Q:** ¿Qué caracteres deben bloquearse para defenderse contra HTTP response splitting y en qué codificaciones?
**A:** CR (`%0d` o `\r`) y LF (`%0a` o `\n`). Deben eliminarse antes de insertar datos en cabeceras HTTP de respuesta.

---

**Q:** ¿Cuáles son las tres medidas que deben adoptar los Application Developers para defenderse contra HTTP response splitting?
**A:** (1) Restringir acceso de la aplicación a IPs únicas. (2) Bloquear CR (`%0d`/`\r`) y LF (`%0a`/`\n`). (3) Cumplir RFC 2616 (HTTP/1.1). (4) Parsear todos los inputs antes de usarlos en cabeceras HTTP.

---

**Q:** ¿Qué es DNSSEC y cómo se diferencia de DoH/DoT en la defensa contra DNS hijacking?
**A:** DNSSEC añade una capa de autenticación sobre DNS para prevenir su compromiso (firma criptográfica de registros). DoH/DoT cifran las consultas DNS para evitar interceptación y redirección. DNSSEC = integridad de registros. DoH/DoT = confidencialidad de consultas.

---

**Q:** ¿Cuál es la configuración recomendada de master–slave DNS para defenderse contra DNS hijacking?
**A:** Master sin acceso a Internet + dos servidores slave. Si un slave es comprometido, solo se actualiza cuando recibe actualización del master.

---

**Q:** ¿Qué técnica de red defiende contra IP spoofing en servidores web?
**A:** Regla "deny this IP address" en el ruleset del firewall o comando "routed blackhole" procesando el fichero de log de seguridad.

---

**Q:** ¿Qué debe hacerse con el HTTP service banner para mejorar la seguridad?
**A:** Configurarlo correctamente para ocultar detalles del dispositivo host como versión y tipo de OS.

---

**Q:** ¿Cuál es la función principal de CORE Impact?
**A:** Herramienta de pen testing que evalúa la postura de seguridad del servidor web usando técnicas reales de ciberdelincuentes; escanea vulnerabilidades, importa resultados de scan y ejecuta exploits para verificarlos.

---

**Q:** ¿Cuál es la diferencia entre Syhunt Hybrid y N-Stalker X?
**A:** Syhunt Hybrid: automatiza security testing, detecta XSS/directory traversal/SQL injection/command execution, analiza JS. N-Stalker X: busca clickjacking/SQL injection/XSS, permite spider crawling, crea web macros para form authentication y ofrece capacidades de proxy.

---

**Q:** ¿Qué hace QualysGuard Malware Detection?
**A:** Escanea proactivamente sitios web en busca de malware, proporciona alertas automatizadas e informes detallados para identificación y resolución rápida de infecciones.

---

**Q:** ¿Qué debe hacerse con los ficheros subidos por usuarios (file uploads) según las countermeasures?
**A:** Escanearlos para malware y almacenarlos fuera del web root.

---

**Q:** ¿Cuál es la diferencia de uso entre FTPS y SFTP según este módulo?
**A:** FTPS (FTP + TLS/SSL): recomendado en countermeasures de protocolos para transferencias de ficheros en general. SFTP (SSH File Transfer Protocol): recomendado en countermeasures de ficheros y directorios para cifrar transferencias.

---

**Q:** ¿Qué configuración IIS debe aplicarse para prevenir path traversal según code access security?
**A:** Configurar IIS para rechazar URLs con `../`, lockdown de comandos del sistema con ACLs restrictivas e instalar nuevos patches.

---

**Q:** ¿Para qué sirve el sandbox directory en la defensa de ficheros de un servidor web?
**A:** Ejecutar el servidor web dentro de un sandbox directory para prevenir el acceso a ficheros del sistema.

---

## 5. Confusión frecuente

**DMZ vs. Internal Network**
- DMZ: zona aislada entre Internet e intranet donde se ubican los servidores web. Tiene firewalls en ambas fronteras.
- Internal network: red corporativa con aplicaciones, BBDD y sistemas internos. Los servidores web NO deben estar aquí.
- Criterio: "servidor web accesible desde Internet" → DMZ. "BBDD o aplicaciones internas" → internal network.

---

**machine.config vs. web.config**
- machine.config: configuración a nivel de máquina (.NET); afecta a **todas** las aplicaciones en el servidor. Controla tracing, debug, HttpModules, HttpForbiddenHandler, session state.
- web.config: configuración a nivel de aplicación individual. Afecta solo a esa aplicación.
- Criterio: "configuración que afecta a todo el servidor .NET" → machine.config. "Configuración de una aplicación concreta" → web.config.

---

**DNSSEC vs. DoH/DoT**
- DNSSEC: autentica la integridad de los registros DNS mediante firmas criptográficas. Previene que los registros sean falsificados.
- DoH/DoT: cifra el canal de comunicación de las consultas DNS. Previene interceptación y redirección de consultas.
- Criterio: "autenticidad de registros DNS" → DNSSEC. "Cifrado de consultas DNS en tránsito" → DoH/DoT.

---

**Syhunt Hybrid vs. Qualys Community Edition**
- Syhunt Hybrid: web **application** security scanner. Foco en vulnerabilidades de aplicación (XSS, SQL injection, directory traversal, command execution).
- Qualys Community Edition: web **server** security scanner + asset discovery + vulnerability management + cloud inventory.
- Criterio: "vulnerabilidades de la aplicación web" → Syhunt. "Asset discovery y gestión de vulnerabilidades del servidor" → Qualys.

---

**Back-out plan vs. Disaster Recovery Plan**
- Back-out plan: plan para revertir al estado previo ante una **implementación fallida** de patch/actualización.
- Disaster Recovery Plan: plan para gestionar fallos en el proceso de patch management en general (más amplio).
- Criterio: "fallo al aplicar un hotfix específico y necesidad de revertir" → back-out plan. "Fallos sistémicos en el proceso de patch management" → disaster recovery plan.
