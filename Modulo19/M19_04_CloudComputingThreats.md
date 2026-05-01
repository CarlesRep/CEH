# M19_04 — Cloud Computing Threats
**Módulo 19 / Subapartado 4 — Cloud Computing Threats**

---

## 1. Conceptos y definiciones

### 🔴 OWASP Top 10 Cloud Security Risks

| # | Riesgo | Núcleo del problema |
|---|--------|---------------------|
| R1 | Accountability and Data Ownership | Cloud pública → pérdida de control y recuperabilidad de datos |
| R2 | User Identity Federation | Múltiples identidades entre proveedores → gestión compleja; menos control sobre offboarding |
| R3 | Regulatory Compliance | Datos seguros en un país pueden no serlo en otro; leyes divergentes |
| R4 | Business Continuity and Resiliency | Si el CSP gestiona mal la continuidad → pérdida económica en desastre |
| R5 | User Privacy and Secondary Usage of Data | Redes sociales minan datos de usuario para uso secundario; "default share" compromete privacidad |
| R6 | Service and Data Integration | Datos en tránsito sin protección → escuchas e interceptación |
| R7 | Multi Tenancy and Physical Security | Segregación lógica insuficiente → tenants interfieren con seguridad de otros |
| R8 | Incidence Analysis and Forensic Support | Logs distribuidos en múltiples hosts/países → forensics difícil; distintas leyes aplican |
| R9 | Infrastructure Security | Mala configuración → escaneo de puertos activos, contraseñas y configuraciones por defecto |
| R10 | Non-Production Environment Exposure | Entornos dev/test → acceso no autorizado, divulgación y modificación de información |

---

### 🔴 OWASP Top 10 Kubernetes Risks

| # | Riesgo | Mecanismo clave |
|---|--------|----------------|
| K01 | Insecure Workload Configurations | Contenedores con root, sin límites de recursos, acceso de red excesivo |
| K02 | Supply Chain Vulnerabilities | Imágenes de terceros con vulnerabilidades en CI/CD pipelines |
| K03 | Overly Permissive RBAC Configurations | Permisos excesivos → escalada de privilegios; viola principio de least privilege |
| K04 | Lack of Centralized Policy Enforcement | Políticas inconsistentes entre clusters → brechas; mitigar con **OPA (Open Policy Agent)** |
| K05 | Inadequate Logging and Monitoring | Sin logs → ciegos ante ataques; requiere alertas en tiempo real |
| K06 | Broken Authentication Mechanisms | Autenticación débil, sin MFA → acceso no autorizado al cluster |
| K07 | Missing Network Segmentation Controls | Sin políticas de red → movimiento lateral libre; atacantes se propagan por el cluster |
| K08 | Secrets Management Failures | Secretos hardcodeados o en ubicaciones no seguras → credenciales comprometidas |
| K09 | Misconfigured Cluster Components | API server, etcd, kubelet mal configurados → control del cluster |
| K10 | Outdated and Vulnerable Kubernetes Components | K8s y dependencias desactualizadas con CVEs conocidos |

---

### 🔴 OWASP Top 10 Serverless Security Risks

| # | Riesgo | Vector específico de serverless | Impacto |
|---|--------|---------------------------------|---------|
| A1 | Injection | Eventos desde S3, Kinesis, DynamoDB, CosmoDB, CodeCommit, SMS, email, IoT; **el firewall no filtra eventos de email/DB** | Borrado/corrupción de datos en cloud storage |
| A2 | Broken Authentication | Funciones stateless disparadas por eventos → spoofed emails para invocar funciones sin autenticación | Fuga de datos, ruptura de lógica de negocio |
| A3 | Sensitive Data Exposure | Datos en /tmp sin limpiar; cifrado débil; ataques MiTM, cracking de claves | Exposición de PII, credenciales, datos médicos/financieros |
| A4 | XML External Entities (XXE) | Solo afecta al contenedor de la función si está en VPN interna | Fuga de código de función y archivos sensibles (env vars, /tmp) |
| A5 | Broken Access Control | Funciones sobre-privilegiadas; arquitectura stateless facilita explotación | Fuga de cloud storage y DBs |
| A6 | Security Misconfiguration | Funciones con **timeout largo y bajo límite de concurrencia** → DoS | Fuga de info, pérdida de dinero, DoS |
| A7 | Cross-Site Scripting (XSS) | En serverless: vectores desde emails, logs, cloud storage, IoT (no solo DB/inputs) | Impersonación, robo de API keys |
| A8 | Insecure Deserialization | Lenguajes dinámicos (Python, NodeJS) con JSON | Código arbitrario, fuga de datos |
| A9 | Using Components with Known Vulnerabilities | Librerías de terceros en microservicios | Impacto según especificidad de CVE |
| A10 | Insufficient Logging and Monitoring | Auditoría compleja en serverless → respuesta tardía | Impacto significativo por detección tardía |

---

### 🔴 Ataques específicos de cloud — mecánica y contramedidas

#### Side-Channel Attack (Cross-guest VM Breach)
El atacante despliega una VM maliciosa en el **mismo host físico** que la VM objetivo. Explota los **recursos físicos compartidos** (caché del procesador). Tipos: timing attack, data remanence, acoustic cryptanalysis, power monitoring attack, differential fault analysis. Objetivo: extraer claves criptográficas o secretos en texto plano.

#### Wrapping Attack
Ataque durante la **traducción del mensaje SOAP en la capa TLS**. El atacante duplica el cuerpo del mensaje SOAP (y su firma) y lo envía al servidor como usuario legítimo. El servidor verifica la autenticación mediante el valor de firma (también duplicado) y lo acepta. Resultado: el adversario puede ejecutar código malicioso en los servidores cloud.
Contramedidas clave: XML schema validation, authenticated encryption en XML encryption specification, WS-SecurityPolicy "SignedParts", CryptoCoverageChecker interceptor.

#### Man-in-the-Cloud (MITC) Attack
Versión avanzada de MITM. Abusa de **servicios de sincronización de ficheros cloud** (Google Drive, Dropbox) para C&C, exfiltración y acceso remoto. Los **synchronization tokens** no distinguen tráfico malicioso de legítimo. Mecanismo: el atacante planta su token en el Drive de la víctima → la app sincroniza con la cuenta del atacante → roba el token legítimo → restaura el token original para evitar detección.
Contramedida clave: **CASB** (Cloud Access Security Broker) para monitorizar tráfico cloud.

#### Cloud Hopper Attack
Dirigido a **MSPs (Managed Service Providers)** y sus clientes. El atacante infiltra el MSP → accede a perfiles de clientes desde la cuenta del MSP → comprime y exfiltra datos. Vectores: spear-phishing con malware customizado, PowerShell/PowerSploit para reconocimiento, C&C con dominios spoofing legítimos, fileless malware.
Contramedida clave: **jump servers**.

#### Cloud Cryptojacking
Uso no autorizado de recursos de la víctima para minar criptomoneda. Vectores: misconfiguraciones cloud, sitios comprometidos, vulnerabilidades cliente/servidor. Herramientas: **CoinHive, Cryptoloot** (JavaScript-based crypto-miners). Pasos: compromete servicio cloud → embebe script de mining → víctima visita → script ejecuta automáticamente en el navegador → recompensa va al atacante por cada bloque añadido al blockchain.
Señal de alerta: **subidas repentinas en la factura de recursos cloud**.

#### Cloudborne Attack
Vulnerabilidad en servidores bare-metal cloud. El atacante implanta un **backdoor en el firmware del servidor** (específicamente en el **BMC — Baseboard Management Controller**) mediante vulnerabilidades en hardware super-micro. El backdoor persiste aunque el servidor sea reasignado a nuevos clientes (si el **firmware re-flash no se realiza correctamente**). El BMC gestiona el servidor remotamente vía **IPMI** (Intelligent Platform Management Interface).

#### IMDS Attack (Instance Metadata Service)
IMDS provee información de la instancia y genera credenciales para roles asociados. El atacante explota una vulnerabilidad zero-day o un reverse proxy mal configurado → se conecta a la instancia cloud → obtiene metadatos → usa credenciales para acceder a recursos cloud.
Contramedida clave: usar **IMDSv2** en lugar de IMDSv1; deshabilitar IMDS cuando no sea necesario.

#### CPDoS / CDN Cache Poisoning
El atacante envía peticiones HTTP malformadas o sobredimensionadas → el servidor de origen responde con error → la **CDN cachea la respuesta de error** → usuarios legítimos reciben el error cacheado (ej. "404 Not Found") → DoS efectivo sin atacar directamente el servidor.

#### Cloud Snooper Attack
Dirigido a **AWS Security Groups (SGs)**. El atacante instala rootkits (explotando filtros de tráfico débiles, supply-chain attacks o brute-force SSH). Envía paquetes C2 camuflados como tráfico legítimo en **puertos 80 y 443**. El rootkit recrea los paquetes con **source ports 1010, 2020, 6060, 7070, 8080, 9999** y los redirige al backdoor Trojan. Al exfiltrar, el rootkit vuelve a usar puertos 80/443 para pasar el firewall.

#### Golden SAML Attack
Dirigido a **identity providers** con protocolo SAML (ej. **ADFS — Active Directory Federation Service**). El atacante obtiene acceso administrativo al IdP → roba el **certificado y clave de firma de assertions** → genera tokens/respuestas SAML falsificadas → el service provider acepta el token falsificado → acceso completo a servicios federados.

#### Living Off the Cloud (LotC)
Evolución de "living off the land". El atacante usa **servicios SaaS/IaaS legítimos** (Dropbox, Google Drive, Ngrok) como vectores de C2, exfiltración y distribución de malware. Difícil de detectar porque el tráfico se mezcla con tráfico cloud legítimo.

#### EDoS (Economic Denial of Sustainability)
El atacante ejecuta código malicioso que consume elevados recursos computacionales/almacenamiento → el **titular legítimo recibe la factura** hasta que se detecta la causa. En el peor caso: quiebra del cliente.
Contramedida: servicio de mitigación EDoS reactivo/on-demand **(scrubber service)** con enfoque client-puzzle.

---

### 🔴 Vulnerabilidades de contenedores (13)

| # | Vulnerabilidad | Riesgo concreto |
|---|---------------|----------------|
| 1 | Impetuous Image Creation | Imágenes sin controles de seguridad |
| 2 | Insecure Image Configurations | Base image con software obsoleto/innecesario → aumenta superficie de ataque |
| 3 | Unreliable Third-Party Resources | Recursos no confiables → ataques maliciosos |
| 4 | Unauthorized Access | Acceso a cuentas → privilege escalation |
| 5 | Insecure Container Runtime Configurations | Montaje de directorios sensibles del host |
| 6 | Data Exposure in Docker Files | Contraseñas y SSH keys expuestas en imágenes Docker |
| 7 | Embedded Malware | Malware en imágenes o descargado post-despliegue |
| 8 | Non-Updated Images | Imágenes desactualizadas con bugs de seguridad |
| 9 | Hijacked Repository and Infected Resources | Misconfiguraciones → envenenamiento de repositorio |
| 10 | Hijacked Image Registry | Compromiso del registry → imágenes maliciosas |
| 11 | Exposed Services due to Open Ports | Misconfiguration → exposición de puertos → info leakage |
| 12 | Exploited Applications | SQLi, XSS, RFI en apps vulnerables |
| 13 | Mixing of Workload Sensitivity Levels | Orquestadores mezclan cargas de distinta sensibilidad en el mismo host → contaminación cruzada |

---

### 🔴 Vulnerabilidades de Kubernetes (10)

| # | Vulnerabilidad | Detalle técnico |
|---|---------------|----------------|
| 1 | No Certificate Revocation | K8s no soporta revocación de certificados → hay que regenerar toda la cadena |
| 2 | Unauthenticated HTTPS Connections | Conexiones entre componentes sin TLS adecuado → acceso no autorizado a kubelet-managed Pods |
| 3 | Exposed Bearer Tokens in Logs | Tokens en logs de hyperkube kube-apiserver → impersonación de usuarios legítimos |
| 4 | Exposure of Sensitive Data via Environment Variables | Valores accesibles via environment logging |
| 5 | Secrets at Rest not Encrypted by Default | Secretos sin cifrar en etcd → recuperables con acceso al etcd server |
| 6 | Non-constant Time Password Comparison | Kube-apiserver vulnerable a **timing attacks** en autenticación básica |
| 7 | Hardcoded Credential Paths | Si cluster token y root CA están en ubicaciones distintas → inserción de token/CA maliciosos |
| 8 | Log Rotation is not Atomic | Reinicio de kubelet durante rotación de logs → borrado de todos los logs |
| 9 | No Back-off Process for Scheduling | El scheduler entra en bucle infinito programando pods rechazados |
| 10 | No Non-repudiation | Kube-apiserver sin auditoría central; sin debug mode → no registra acciones de usuario |

---

### Amenazas cloud adicionales — resumen ejecutivo

| Amenaza | Mecanismo clave | Contramedida prioritaria |
|---------|----------------|-------------------------|
| Data Breach/Loss | Fallo en auth/authz/access controls; CSP abusa datos | DLP software, CASBs, micro-segmentation, MFA |
| Malicious Insiders | Acceso autorizado abusado (empleados/ex-empleados/contratistas) | Supply chain management, transparencia en IS |
| Loss of Encryption Keys | Mala gestión de claves | No almacenar claves junto a datos cifrados; usar HSM; AES/RSA |
| Isolation Failure | Multi-tenancy → fallo de aislamiento entre tenants | Mantener memoria, almacenamiento y red aislados |
| EDoS | Consumo de recursos → facturación al legítimo | Scrubber service con client-puzzle approach |
| Lock-in | Sin portabilidad entre CSPs | Multi-cloud strategy, APIs estandarizadas, exit strategy previa |
| Golden SAML | Robo de cert de firma SAML en ADFS | MFA, monitorización de actividades, rotación de certificados |
| Cloudborne | Backdoor en firmware BMC de bare-metal | Firmware re-flash correcto; validar antes de reasignar servidor |
| Wrapping Attack | Duplicación del cuerpo SOAP en TLS | XML schema validation, WS-SecurityPolicy "SignedParts" |
| Unsynchronized Clocks | Timestamps incorrectos → análisis de logs imposible | **NTP** (Network Time Protocol) |
| Abuse/Nefarious Use | Registro débil → anonymous access para DDoS, rainbow tables, botnets | Proceso robusto de registro y validación; per-tenant firewalls |

---

## 2. Exam Traps ⚠️

⚠️ **[Wrapping Attack — capa de operación]**
El wrapping attack opera en la **capa TLS durante la traducción del mensaje SOAP**. No es un ataque de red genérico. El examen puede presentarlo como un ataque de replay simple.

⚠️ **[MITC vs. MITM]**
MITM intercepta comunicaciones directamente. MITC abusa de **servicios de sincronización cloud** (Drive, Dropbox) mediante tokens de sincronización. El examen puede presentarlos como equivalentes.

⚠️ **[Cloudborne — persistencia]**
El backdoor de Cloudborne persiste si el **firmware re-flash no se realiza correctamente** durante la reasignación del servidor. La vulnerabilidad reside en el **BMC** (Baseboard Management Controller), gestionado via **IPMI**. El examen puede confundir BMC con hypervisor.

⚠️ **[IMDS — contramedida principal]**
La contramedida principal es usar **IMDSv2** (no IMDSv1). Además: deshabilitar IMDS cuando no se necesite y asignar mínimos privilegios a los roles de instancia.

⚠️ **[EDoS — mecanismo]**
EDoS no es un DoS técnico que interrumpe el servicio directamente; su impacto es **financiero**: el legítimo titular es facturado por el consumo fraudulento. La contramedida es el **scrubber service** (on-demand, reactivo).

⚠️ **[Golden SAML — objetivo]**
Golden SAML ataca al **identity provider** (ADFS), no al service provider. El atacante roba el certificado y la clave de firma de assertions SAML. El examen puede presentarlo como un ataque al SP.

⚠️ **[Cloud Snooper — puertos usados]**
El rootkit reconstituye paquetes con **source ports específicos** (1010, 2020, 6060, 7070, 8080, 9999) internamente, pero usa **puertos 80 y 443** para pasar el firewall tanto al entrar como al exfiltrar.

⚠️ **[Kubernetes — revocación de certificados]**
K8s **no soporta revocación de certificados**. Para eliminar un certificado hay que regenerar **toda la cadena**. El examen puede afirmar que K8s soporta CRL (Certificate Revocation List).

⚠️ **[Kubernetes — Secrets cifrados por defecto]**
Los Secrets de Kubernetes **no están cifrados por defecto** en reposo. Se almacenan sin cifrar en etcd. El examen puede afirmar lo contrario.

⚠️ **[Serverless A1 — fuentes de injection]**
En serverless, las fuentes de injection no son solo APIs: incluyen **S3, Kinesis, DynamoDB, CosmoDB, CodeCommit, SMS, email, IoT**. Y el firewall **no puede filtrar** eventos de email o base de datos. El examen puede limitar los vectores a la API.

⚠️ **[Serverless A6 — configuración de DoS]**
La misconfiguration que permite DoS en serverless es específica: **timeout largo + bajo límite de concurrencia**. El examen puede presentar cualquier misconfiguration como causa de DoS en serverless.

⚠️ **[Loss of Encryption Keys — almacenamiento]**
La contramedida principal es **no almacenar las claves junto a los datos cifrados**. Usar HSM (Hardware Security Module). El examen puede presentar el almacenamiento co-localizado como práctica aceptable.

⚠️ **[Cloud Hopper — objetivo primario]**
Cloud Hopper ataca primero al **MSP**, no directamente al cliente final. Desde el MSP, accede a los clientes. El examen puede presentarlo como un ataque directo a la organización objetivo.

⚠️ **[Cryptojacking — señal de alerta en cloud]**
La señal de alerta específica en cloud cryptojacking es el **aumento repentino en la factura de recursos cloud**. El examen puede no asociar cryptojacking con indicadores financieros.

---

## 3. Nemotécnicos

### OWASP Cloud Top 10 (R1–R10)
**"A-User-Reg-Biz-Privacy-Serv-Multi-Inc-Infra-Non"**
→ **A**ccountability | **U**ser Identity | **Reg**ulatory | **Biz** Continuity | **Privacy** | **Serv**ice Integration | **Multi**-tenancy | **Inc**ident Forensics | **Infra** Security | **Non**-Production

### OWASP Kubernetes Top 10 (K01–K10)
**"Workloads Supply RBAC Policy Logs Auth Network Secrets Cluster Old"**
→ K01 Workloads | K02 Supply Chain | K03 RBAC | K04 Policy | K05 Logs | K06 Auth | K07 Network | K08 Secrets | K09 Cluster | K10 Old components

### Ataques cloud — diferenciación por vector único
- **Wrapping**: SOAP + TLS → duplicación de body
- **MITC**: tokens de sincronización cloud (Drive/Dropbox)
- **Side-Channel**: mismo host físico + caché compartida
- **Cloudborne**: firmware BMC de bare-metal
- **Golden SAML**: certificado de firma SAML en ADFS
- **Cloud Snooper**: rootkit + puertos 80/443 como camuflaje
- **EDoS**: facturación financiera al legítimo
- **CPDoS**: CDN cachea respuesta de error

### Kubernetes vulnerabilidades — las 3 más técnicas
**"No-Cert, No-TLS, No-Audit"**
→ **No** Certificate Revocation | Unauthenticated **TLS** connections | **No** Non-repudiation (no auditoría central)

### HSM, NTP, DNSSEC — contramedidas específicas
- **HSM** → protección de encryption keys
- **NTP** → sincronización de relojes (Unsynchronized Clocks)
- **DNSSEC** → ataques DNS (poisoning, cybersquatting, hijacking)

---

## 4. Flashcards

**Q:** ¿Qué ataque cloud duplica el cuerpo de un mensaje SOAP durante la traducción en la capa TLS?
**A:** Wrapping Attack. El atacante duplica el cuerpo del mensaje (y su firma) y lo envía al servidor como usuario legítimo.

---

**Q:** ¿En qué se diferencia un ataque MITC de un MITM?
**A:** MITM intercepta comunicaciones directamente. MITC abusa de servicios de sincronización cloud (Google Drive, Dropbox) mediante tokens de sincronización para exfiltración, C&C y acceso remoto.

---

**Q:** ¿Qué componente específico del servidor bare-metal explota un ataque Cloudborne?
**A:** El BMC (Baseboard Management Controller), gestionado vía IPMI, que permite el control remoto del servidor.

---

**Q:** ¿Cuál es la contramedida principal contra un ataque IMDS?
**A:** Usar IMDSv2 en lugar de IMDSv1. Además, deshabilitar IMDS cuando no sea necesario y asignar mínimos privilegios a los roles de instancia.

---

**Q:** ¿Qué es EDoS y cómo difiere de un DDoS?
**A:** EDoS (Economic Denial of Sustainability) destruye recursos financieros: un atacante consume recursos cloud fraudulentamente y el titular legítimo recibe la factura. DDoS interrumpe el servicio. La contramedida de EDoS es el scrubber service con enfoque client-puzzle.

---

**Q:** ¿Qué vulnerability de Kubernetes permite un timing attack?
**A:** Non-constant Time Password Comparison en kube-apiserver, que no realiza comparación segura de valores secretos en autenticación básica.

---

**Q:** ¿Están cifrados por defecto los Secrets de Kubernetes en reposo?
**A:** No. Los Secrets no están cifrados por defecto; se almacenan en etcd sin cifrar. Un atacante con acceso al etcd puede recuperarlos.

---

**Q:** ¿Qué herramientas JavaScript se usan en ataques de cloud cryptojacking?
**A:** CoinHive y Cryptoloot. Se embeben como scripts en páginas web y ejecutan en el navegador de la víctima.

---

**Q:** ¿Cuál es la señal de alerta específica de cryptojacking en entornos cloud?
**A:** Aumento repentino e inesperado en la factura de recursos cloud.

---

**Q:** ¿Qué fuentes de eventos (además de APIs) son vectores de injection en serverless según OWASP A1?
**A:** Cloud storage events (S3, Blob), stream data processing (AWS Kinesis), modificaciones de DB (DynamoDB, CosmoDB), modificaciones de código (AWS CodeCommit), notificaciones (SMS, email, IoT).

---

**Q:** ¿Por qué el firewall no protege contra injection en serverless cuando el vector es email?
**A:** El firewall no puede filtrar los eventos generados mediante email o base de datos; no tiene visibilidad sobre esos canales de invocación.

---

**Q:** ¿Qué herramienta OWASP se recomienda para K04 (Lack of Centralized Policy Enforcement) en Kubernetes?
**A:** OPA (Open Policy Agent) para aplicar políticas de seguridad consistentes en todos los clusters.

---

**Q:** ¿Cuáles son los 5 pasos del ataque Cloud Snooper?
**A:** 1) Envía paquetes C2 con destino 80/443 → 2) Firewall los pasa → 3) Rootkit recrea paquetes con source ports 1010/2020/6060/7070/8080/9999 → 4) Backdoor ejecuta comandos C2 → 5) Rootkit exfiltra con source ports 80/443.

---

**Q:** ¿Cuál es la contramedida principal para el ataque Wrapping?
**A:** XML schema validation para detectar mensajes SOAP manipulados + WS-SecurityPolicy "SignedParts" para que el usuario especifique el body y headers a firmar.

---

**Q:** ¿Qué contramedida específica se usa para Unsynchronized System Clocks en cloud?
**A:** NTP (Network Time Protocol) + servidor de tiempo dentro del firewall de la organización.

---

**Q:** ¿Cuál es la contramedida principal para la pérdida de encryption keys?
**A:** No almacenar las claves junto a los datos cifrados; usar HSM (Hardware Security Module) para almacenarlas de forma segura.

---

**Q:** ¿Cuáles son los pasos del Golden SAML Attack?
**A:** 1) Accede al servidor ADFS → 2) Roba certificado y clave de firma → 3) Usuario solicita acceso → SP redirige al IdP → 4) Atacante intercepta y envía respuesta SAML falsificada con las claves robadas → 5) SP concede acceso a servicios federados.

---

**Q:** ¿Cuántos pasos tiene un ataque CPDoS y cuál es el resultado final?
**A:** 5 pasos: 1) Petición HTTP maliciosa → 2) CDN no tiene caché → reenvía al origen → 3) Origen devuelve error → CDN cachea el error → 4) Usuarios reciben error (ej. 404) → 5) CDN difunde el error al resto de usuarios.

---

**Q:** ¿Qué vulnerability de K8s impide la revocación de certificados individuales?
**A:** K8s no soporta certificate revocation; para retirar un certificado hay que regenerar toda la cadena de certificados.

---

**Q:** ¿Qué diferencia a Living Off the Cloud (LotC) de un ataque de malware convencional?
**A:** LotC usa servicios cloud legítimos (Dropbox, Google Drive, Ngrok) como infraestructura de C2, distribución de malware y exfiltración, mezclándose con tráfico cloud normal. No necesita infraestructura maliciosa propia visible.

---

## 5. Confusión frecuente

**Wrapping Attack vs. Replay Attack**
- Replay: reenvía un mensaje capturado íntegramente.
- Wrapping: duplica específicamente el **cuerpo del mensaje SOAP** (manteniendo la firma válida del original) durante la traducción en TLS. El servidor valida la firma duplicada y acepta el mensaje fraudulento.
- Criterio: si menciona SOAP + TLS + duplicación de body → Wrapping.

---

**Side-Channel Attack vs. Isolation Failure**
- Side-Channel: el atacante **activamente explota** los recursos físicos compartidos (caché de procesador) desde una VM co-residente para extraer claves criptográficas.
- Isolation Failure: fallo **pasivo** de aislamiento entre tenants que permite acceso accidental o deliberado a datos de otro tenant.
- Criterio: si el escenario describe extracción de claves desde otra VM en el mismo host físico → Side-Channel. Si describe acceso accidental a datos de otro tenant → Isolation Failure.

---

**Cloud Hopper vs. Supply Chain Attack**
- Cloud Hopper: ataca primero al **MSP** como punto de entrada hacia sus clientes; implica movimiento lateral entre sistemas de clientes desde la cuenta MSP.
- Supply Chain: compromete componentes software de terceros (librerías, imágenes, CI/CD) antes de que lleguen al entorno objetivo.
- Criterio: si el vector es el MSP como intermediario de servicios → Cloud Hopper. Si el vector es software/componentes de terceros → Supply Chain.

---

**EDoS vs. DDoS**
- DDoS: objetivo es la **disponibilidad del servicio** (interrumpirlo).
- EDoS: objetivo es el **impacto financiero** (facturar al legítimo por recursos consumidos fraudulentamente). El servicio puede seguir disponible.
- Criterio: si la pregunta habla de coste/facturación anormal o ruina económica → EDoS.

---

**Golden SAML vs. Pass-the-Ticket (Kerberos)**
- Golden SAML: falsifica tokens **SAML** robando el certificado de firma del **IdP (ADFS)**; afecta a servicios federados cloud.
- Pass-the-Ticket: forja tickets **Kerberos** (TGT/TGS) robando el hash KRBTGT; afecta a entornos AD on-premises.
- Criterio: si el escenario menciona ADFS, SAML assertions, cloud federation → Golden SAML. Si menciona Kerberos, AD, TGT → Pass-the-Ticket.

---

**CPDoS vs. DDoS convencional**
- DDoS: múltiples sistemas atacan directamente el servidor destino con flood de tráfico.
- CPDoS: un único atacante envía peticiones HTTP malformadas → la **CDN amplifica el ataque** cacheando el error → el servidor de origen ya no necesita ser atacado directamente.
- Criterio: si el escenario menciona CDN y errores cacheados distribuyéndose a usuarios → CPDoS.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un pentester observa que el servidor víctima está en AWS y el IMDS está accesible. Explota una vulnerabilidad en un reverse proxy mal configurado para conectarse a `http://169.254.169.254/latest/meta-data/iam/security-credentials/`. ¿Qué ataque ejecuta?

A) Side-Channel Attack  
B) IMDS Attack  
C) Cloud Snooper Attack  
D) MITC Attack  

**Respuesta correcta: B**
El atacante ejecuta un **IMDS Attack (Instance Metadata Service Attack)**. Accede al endpoint de metadatos de AWS (`169.254.169.254`) explotando el proxy mal configurado para obtener credenciales IAM temporales asociadas al rol de la instancia. La contramedida es usar **IMDSv2** y deshabilitar IMDS cuando no sea necesario.

---

**P2.** Un analista de seguridad investiga una brecha en la que el atacante instaló un rootkit en el firmware de un servidor bare-metal. Cuando el servidor fue reasignado a otro cliente, el backdoor persistió. ¿Qué tipo de ataque describe esto?

A) Golden SAML Attack  
B) Cloud Hopper Attack  
C) Cloudborne Attack  
D) Side-Channel Attack  

**Respuesta correcta: C**
El **Cloudborne Attack** explota vulnerabilidades en servidores bare-metal cloud para implantar un **backdoor en el firmware del BMC (Baseboard Management Controller)**. El backdoor persiste aunque el servidor sea reasignado a nuevos clientes si el **firmware re-flash no se realiza correctamente**. El BMC se gestiona vía IPMI.

---

**P3.** Un equipo de respuesta a incidentes detecta que los usuarios de un sitio web empresarial reciben "404 Not Found" aunque la web está operativa. La CDN está devolviendo una respuesta cacheada. ¿Qué ataque ha ocurrido?

A) DDoS volumétrico  
B) CPDoS — CDN Cache Poisoning  
C) Cloud Cryptojacking  
D) Living Off the Cloud (LotC)  

**Respuesta correcta: B**
**CPDoS (Cache Poisoned Denial of Service)**: el atacante envía una petición HTTP malformada o sobredimensionada → el servidor devuelve un error → la **CDN cachea esa respuesta de error** → los usuarios legítimos reciben el error cacheado. El ataque es efectivo sin atacar directamente el servidor de origen.

---

**P4.** Un investigador descubre que las credenciales de una cuenta de AWS fueron comprometidas porque un atacante con acceso al MSP corporativo se movió lateralmente a los clientes del MSP. ¿Qué tipo de ataque cloud describe esto?

A) MITC Attack  
B) Cloud Hopper Attack  
C) Side-Channel Attack  
D) EDoS Attack  

**Respuesta correcta: B**
El **Cloud Hopper Attack** se dirige a los **MSPs (Managed Service Providers)** como vector de entrada para acceder a sus clientes. El atacante infiltra el MSP → desde la cuenta del MSP accede a los perfiles de clientes → exfiltra datos. La contramedida clave son los **jump servers**.

---

**P5.** Un CISO recibe una factura de AWS tres veces mayor de lo esperado. La investigación revela que alguien embebió scripts JavaScript (CoinHive) en las páginas web de la empresa para usar los recursos de las visitas. ¿Qué ataque cloud es este y cuál es la señal de alerta específica?

A) EDoS — señal: interrupción del servicio  
B) Cloud Cryptojacking — señal: aumento repentino en la factura de recursos cloud  
C) Living Off the Cloud — señal: tráfico DNS sospechoso  
D) MITC Attack — señal: tokens de sincronización alterados  

**Respuesta correcta: B**
El **Cloud Cryptojacking** usa recursos no autorizados para minar criptomoneda. Las herramientas **CoinHive y Cryptoloot** son JavaScript que se ejecutan en el navegador de la víctima. La señal de alerta específica en cloud es el **aumento repentino e inesperado en la factura de recursos cloud**.

---

**P6.** Durante un pentest a un identity provider ADFS, el atacante obtiene acceso administrativo y roba el certificado y la clave privada de firma de assertions SAML. Puede ahora generar tokens SAML falsificados aceptados por todos los service providers federados. ¿Qué ataque es este?

A) Pass-the-Ticket (Kerberos)  
B) Wrapping Attack  
C) Golden SAML Attack  
D) MITC Attack  

**Respuesta correcta: C**
El **Golden SAML Attack** roba el certificado y clave de firma de assertions SAML del **identity provider (ADFS)**. Con esas claves, el atacante puede forjar tokens SAML que cualquier service provider federado aceptará como legítimos, obteniendo acceso a todos los servicios federados. Es el equivalente SAML del Golden Ticket de Kerberos.

---

**P7.** Un equipo de seguridad detecta una vulnerabilidad de Kubernetes: los Secrets almacenados en etcd pueden ser recuperados con solo acceder al servidor etcd. ¿Qué vulnerabilidad específica de K8s describe esto?

A) No Certificate Revocation  
B) Secrets at Rest not Encrypted by Default  
C) Non-constant Time Password Comparison  
D) Exposed Bearer Tokens in Logs  

**Respuesta correcta: B**
**Secrets at Rest not Encrypted by Default** en Kubernetes: los Secrets se almacenan en **etcd sin cifrado por defecto**. Un atacante con acceso al servidor etcd puede recuperar todos los Secrets (credenciales, API keys, tokens) en texto plano. La contramedida es habilitar el cifrado en reposo para etcd.

---

**P8.** Un atacante despliega una VM maliciosa en el mismo proveedor cloud que su objetivo. Desde la VM maliciosa, analiza patrones de timing en la caché compartida del procesador del host físico para extraer claves criptográficas de la VM objetivo. ¿Qué tipo de ataque cloud es este?

A) MITC Attack  
B) CPDoS  
C) Side-Channel Attack  
D) Cloudborne Attack  

**Respuesta correcta: C**
El **Side-Channel Attack (Cross-guest VM Breach)** explota los **recursos físicos compartidos** (caché del procesador) entre VMs co-residentes en el mismo host físico. El atacante despliega una VM maliciosa en el mismo host y usa timing attacks para extraer claves criptográficas de la VM objetivo. No requiere compromiso directo de la VM víctima.
