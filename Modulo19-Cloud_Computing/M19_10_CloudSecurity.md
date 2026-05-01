# M19_10 — Cloud Security
**Módulo 19 / Subapartado 10 — Cloud Security**

---

## 1. Conceptos y definiciones

### 🔴 Cloud Security Control Layers (7 capas)

| Capa | Controles representativos |
|------|--------------------------|
| **Application** | SDLC, binary analysis, scanners, WAF, transactional security |
| **Information** | DLP, content monitoring/filtering, database activity monitoring, encryption |
| **Management** | GRC, IAM, VA/VM, patch management, configuration management, monitoring |
| **Network** | IPS/IDS, firewalls, deep packet inspection, anti-DDoS, QoS, DNSSEC, OAuth |
| **Trusted Computing** | Hardware y software RoT (Root of Trust), API |
| **Computation and Storage** | Host-based firewalls, HIDS/HIPS, integrity/file/log management, encryption, masking |
| **Physical** | Fences, guards, gates, CCTV, biometría, alarmas, patrullas de seguridad |

---

### 🔴 Categorías de controles de seguridad cloud

| Categoría | Función | Ejemplo |
|-----------|---------|---------|
| **Deterrent** | Reducen ataques mediante advertencias | Cartel en la valla informando de consecuencias adversas |
| **Preventive** | Fortalecen el sistema eliminando vulnerabilidades | Mecanismo de autenticación fuerte para prevenir acceso no autorizado |
| **Detective** | Detectan y reaccionan ante incidentes en curso | IDS/IPS detectando ataques |
| **Corrective** | Minimizan consecuencias limitando el daño | Restaurar backups del sistema |

---

### 🔴 SAML — Security Assertion Markup Language

**Tres entidades:**
- **Client/User**: entidad con cuenta válida que solicita servicio/recurso via navegador
- **Service Provider (SP)**: servidor que aloja apps/servicios para usuarios
- **Identity Provider (IdP)**: almacena directorios de usuarios y mecanismos de validación

**Flujo SAML:**
1. Usuario solicita acceso al SP
2. SP envía SAML request al IdP para validar al usuario
3. IdP crea tres assertions en XML:
   - **Authentication assertion**: tipo de login (contraseña, 2FA, etc.)
   - **Attribute assertion**: detalles específicos del usuario
   - **Authorization assertion**: si el usuario puede ser admitido o denegado
4. IdP reenvía assertions XML al SP
5. SP concede acceso al recurso

SAML proporciona **SSO** (Single Sign-On) → un solo conjunto de credenciales para múltiples apps/servicios.

---

### 🔴 CASB — Cloud Access Security Broker

**Definición**: solución on-premise o cloud-hosted ubicada **entre la infraestructura on-premises de la organización y la del CSP**. Actúa como gatekeeper extendiendo las políticas de seguridad más allá de la infraestructura propia.

**4 características principales:**
1. **Visibility**: descubre shadow IT y monitoriza actividad del usuario en apps cloud permitidas
2. **Data Security**: cifrado, tokenización, control de acceso, information rights management
3. **Threat Protection**: detecta y responde a amenazas internas maliciosas, usuarios privilegiados comprometidos y cuentas comprometidas
4. **Compliance**: descubre datos críticos en cloud; aplica políticas DLP para cumplimiento de data residency

**Qué ofrece un CASB:**
- Firewalls (identificación de malware)
- Autenticación de credenciales
- WAF (protección a nivel de aplicación)
- DLP (evita transferencia de información crítica)
- Credential mapping cuando SSO no está disponible

**Cómo funciona:**
- Garantiza que el tráfico entre dispositivos on-premises y el CSP cumple las políticas de seguridad
- Auto-discovery de cloud apps en uso, apps de alto riesgo y usuarios de alto riesgo
- Encripta y aplica device profiling

**CASB soluciones:** Forcepoint ONE CASB, CloudCodes, Cisco Cloudlock, Zscaler CASB, Proofpoint CASB, FortiCASB.

---

### Zero Trust Networks

**Principio**: "Trust no one and validate before providing a cloud service or granting access permission." Por defecto, ningún usuario o dispositivo es confiable.

**Componentes:**
- **Control plane**: coordina y gestiona el data plane; aprueba peticiones solo de usuarios/dispositivos verificados; aplica **fine-grain policies** (rol, hora del día, tipo de dispositivo)
- **Data plane**: configurado para aceptar tráfico solo del cliente aprobado por el control plane

**Técnicas integradas**: cifrado, MFA, PAM (Privileged Access Management).
**Método de segmentación**: **micro-segmentation** → divide la red en zonas menores con acceso separado. Si hay brecha en un perímetro, micro-segmentation previene explotación adicional.

---

### Cloud Network Security — componentes clave

| Componente | Descripción |
|-----------|------------|
| **VPC (Virtual Private Cloud)** | Entorno cloud privado dentro de la cloud pública; independiente de otros VPCs; combina escalabilidad de cloud pública con segregación de cloud privada |
| **Public Subnet** | VMs transmiten paquetes directamente a Internet via **Internet Gateway (IGW)**; IGW permite tráfico IPv4 e IPv6 sin restricciones de ancho de banda |
| **Private Subnet** | VMs no acceden directamente a Internet; usan **NAT gateway** (en public subnet) para salida; NAT no permite tráfico entrante directo desde Internet |
| **Transit Gateway** | Solución de routing que gestiona comunicación entre red on-premises y VPCs via unidad centralizada; elimina peerings complejos; puede ser filtrado por ACLs cloud |
| **VPC Endpoint** | Conexión privada entre VPC y otro servicio cloud **sin acceso a Internet**, gateways externos, NAT, VPN ni IPs públicas; el tráfico no sale de la red organizacional |

**Dos tipos de VPC Endpoint:**
- **Interface endpoint**: ENI (Elastic Network Interface) con IP privada en subnet definida; punto de entrada para tráfico hacia VPC o servicios cloud
- **Gateway-load-balancer endpoint**: ENI que desvía tráfico a un servicio configurado via gateway load balancer para inspección de seguridad

---

### Scout Suite — security assessment multi-cloud

Herramienta de auditoría de seguridad multi-cloud; usa APIs de los CSPs para recopilar datos de configuración; resalta áreas de riesgo con **códigos de color** por nivel de riesgo. Output: **reporte HTML**.

```bash
# AWS
scout aws --profile <your-aws-profile>

# Azure
scout azure --tenant-id <tid> --subscription-id <sid> --client-id <cid> --client-secret <cs>

# GCP
scout gcp --service-account-file path/to/your-service-account-key.json
```

---

### 🔴 Best Practices Docker Security — datos concretos

| Práctica | Comando/Detalle |
|---------|----------------|
| Eliminar todas las capabilities y asignar solo necesarias | `--cap-drop all` |
| Prevenir privilege escalation con setuid/setgid | `--security-opt=no-new-privileges` |
| Deshabilitar inter-container communication | `--icc=false`; para comunicar: `--link=CONTAINER:ALIAS` |
| Modo read-only en filesystem | `--read-only` flag |
| Log level recomendado | `info` (no usar `debug` en daemon) |
| Usuario por defecto Docker | root → configurar usuario no privilegiado con `USER` directive |
| Módulos Linux de seguridad | seccomp, AppArmor, SELinux |
| Verificar integridad de imágenes de registry | Docker Content Trust (DCT) |
| Secretos: NO variables de entorno | Usar **Docker secrets management** (cifra en tránsito) |
| Almacenamiento de datos sensibles | Docker **volumes** (no bind mounts) |
| TLS entre client y daemon | Establecer autenticación básica con TLS sobre HTTPS |
| Herramientas de auditoría Docker | InSpec y dive |
| Namespaces para aislamiento | PID, IPC, network, user namespaces |
| Health monitoring | Directiva `HEALTHCHECK` en Dockerfiles |

---

### 🔴 Kubernetes Security Best Practices — datos concretos

| Práctica | Herramienta/Detalle |
|---------|-------------------|
| Revocación de certificados | Mantener CRL; usar **OCSP (Online Certificate Status Protocol) stapling** |
| Conexiones HTTPS autenticadas | Two-way TLS por defecto; todos los componentes deben usar CA del kube-apiserver |
| Bearer tokens en logs | Log filtering para eliminar tokens y credenciales de autenticación |
| Secrets en variables de entorno | Usar **Kubernetes Secrets** en todos los componentes |
| Secrets cifrados en reposo | KMS para cifrado; evitar **AES-GCM (AES-Galois/Counter Mode)** y CBC |
| Passwords comparison | Función `crypto.subtle.ConstantTimeCompare`; rechazar basic auth |
| Rutas de credenciales hardcoded | Definir método de configuración; path generalization para cross-platform |
| Log rotation | Técnica **copy-then-rename**; o logs persistentes lineales |
| Scheduler sin back-off | Implementar **back-off process** para kube-scheduler |
| No non-repudiation | Logging secundario; todos los eventos de auth deben ser recuperables centralmente |
| Policy enforcement | **OPA (Open Policy Agent)** o **Kyverno** |
| Monitorización | Prometheus, Grafana, ELK Stack |
| Secretos K8s | Revisar y rotar regularmente |

---

### NIST Access Control — IaaS / PaaS / SaaS (tablas)

**IaaS** — sujetos → objetos (selección clave):
- IaaS end user → Hypervisor: Login, Read, Write, Create
- VM → Hypervisor: Write
- VM → Other VMs (mismo host): Read, Write
- Hypervisor → Hardware resources: Read, Write
- Hypervisor → VMs: Read, Write, Create

**PaaS** — sujetos → objetos (selección clave):
- Application user → Memory data: **Read** (solo lectura)
- VM of hosted app → Other apps' data (mismo host): Read, Write
- Application developer → Middleware data, memory data: Create, Read, Write
- CSP → Application-related data: **Replicate**

**SaaS** — sujetos → objetos (selección clave):
- Application user → Application-related data: Read, Write
- Application user → Application: Execute
- VM of hosted app → Other application code (mismo host): Execute

---

### Herramientas de seguridad — clasificación

| Categoría | Herramientas principales |
|-----------|------------------------|
| **Cloud security general** | Qualys Cloud Platform, Prisma Cloud, Netskope One, Trend Micro Deep Security |
| **Shadow cloud discovery** | Securiti (AI-powered), CloudEagle, Microsoft Defender for Cloud Apps, FireCompass |
| **Container security** | Aqua, Sysdig Falco, Anchore, Snyk Container, Lacework |
| **Kubernetes security** | ACS for Kubernetes (Red Hat), Aqua K8s Security, Kyverno, Kubeaudit, Kubescape |
| **Serverless security** | Dashbird, CloudGuard, Datadog Serverless Monitoring, Prisma Cloud, lumigo |
| **CASB** | Forcepoint ONE CASB, Cisco Cloudlock, Zscaler CASB, Proofpoint CASB, FortiCASB |
| **NG SWG** | Netskope Next Gen SWG, Cloudflare Gateway, Skyhigh SWG |
| **Multi-cloud audit** | Scout Suite |
| **Org de referencia** | **Cloud Security Alliance (CSA)** — nonprofit, promueve best practices |

**Aqua**: escanea imágenes, VMs y funciones serverless buscando vulnerabilidades conocidas, secretos, issues de configuración/permisos, malware y licencias open-source.

**Next-Generation Secure Web Gateway (NG SWG)** — features:
- URL filtering
- Decriptado de certificados TLS/SSL
- Operaciones CASB
- Advanced Threat Protection (ATP) con sandboxing y ML
- DLP para tráfico web y apps cloud
- Contextos de metadata cualitativos

---

## 2. Exam Traps ⚠️

⚠️ **[SAML — tres assertions]**
El IdP crea **tres** assertions: authentication assertion, **attribute assertion** (detalles del usuario) y authorization assertion. El examen puede presentar solo dos o confundir sus funciones.

⚠️ **[CASB — ubicación]**
CASB se sitúa **entre la infraestructura on-premises de la organización y la del CSP**. No es un componente interno del CSP ni solo on-premises. El examen puede presentarlo como solo cloud-hosted.

⚠️ **[Zero Trust — micro-segmentation]**
El mecanismo de contención de Zero Trust es **micro-segmentation**: divide la red en zonas menores con acceso separado. Si hay brecha, la micro-segmentation evita explotación lateral. El examen puede confundirla con VLAN segmentation genérica.

⚠️ **[Private Subnet — salida a Internet]**
Las VMs en subnet privada usan **NAT gateway** (en la public subnet) para salida, no IGW. El NAT **no permite tráfico entrante** desde Internet directamente. El examen puede afirmar que las subnets privadas usan IGW.

⚠️ **[VPC Endpoint — Internet]**
Un VPC Endpoint establece conexión privada entre VPC y servicio cloud **sin usar Internet, gateways externos, NAT ni IPs públicas**. El tráfico permanece en la red organizacional. El examen puede presentarlo como una conexión que atraviesa Internet cifrada.

⚠️ **[Docker — usuario por defecto]**
El usuario por defecto de Docker images es **root**. Best practice: usar la directiva `USER` para correr como usuario no privilegiado. El examen puede presentar "sin usuario" como el default.

⚠️ **[Docker — inter-container communication]**
Para deshabilitar ICC: `--icc=false`. Para comunicar contenedores específicos con ICC deshabilitado: `--link=CONTAINER_NAME_or_ID:ALIAS`. El examen puede omitir la opción `--link` como solución alternativa.

⚠️ **[Docker — secretos en variables de entorno]**
Best practice: **NO usar variables de entorno para secretos**. Usar Docker secrets management (cifra en tránsito). El examen puede presentar variables de entorno como método aceptable para secrets.

⚠️ **[Kubernetes — cifrado NO recomendado]**
Para cifrado de secrets K8s: evitar **AES-GCM (AES-Galois/Counter Mode)** y **CBC (Cipher Block Chaining)**. Usar KMS. El examen puede presentar AES-GCM como recomendado.

⚠️ **[Scout Suite — output]**
Scout Suite genera un **reporte HTML** (no CSV ni JSON por defecto). Usa **códigos de color** para priorizar riesgos. El examen puede describir un output diferente.

⚠️ **[Control de seguridad — Deterrent vs. Preventive]**
Deterrent: reduce la probabilidad de ataque mediante advertencias (no bloquea técnicamente). Preventive: elimina o minimiza la vulnerabilidad (bloquea técnicamente). El examen puede confundir un cartel de advertencia (deterrent) con un mecanismo de autenticación (preventive).

⚠️ **[NIST PaaS — CSP replicates data]**
En el modelo de acceso NIST PaaS, el **CSP** puede hacer **Replicate** sobre "application-related data". El examen puede afirmar que el CSP tiene operaciones de Write en ese contexto.

---

## 3. Nemotécnicos

### 7 capas de seguridad cloud
**"App-Info-Mgmt-Net-Trust-Comp-Phys"**
→ **App**lication | **Info**rmation | **M**anagement | **Net**work | **Tr**usted Computing | **Comp**utation/Storage | **Phys**ical

### 4 categorías de controles
**"D-P-D-C"** → **D**eterrent (avisa) | **P**reventive (bloquea) | **D**etective (detecta) | **C**orrective (repara)
Analogía: Cartel → Candado → Cámara → Backup

### SAML — 3 assertions del IdP
**"Auth-Att-Authz"** → **Auth**entication (tipo login) | **Att**ribute (detalles usuario) | **Authz** (autorización: admit/deny)

### CASB — 4 características
**"V-D-T-C"** → **V**isibility | **D**ata security | **T**hreat protection | **C**ompliance

### Zero Trust — 2 planos
**"Control autoriza → Data ejecuta"**
→ Control plane (verifica, fine-grain policies) → Data plane (acepta tráfico solo del autorizado)

### Docker security — 4 flags críticos
**"Cap-No-New-icc-Read"** → `--cap-drop all` | `--security-opt=no-new-privileges` | `--icc=false` | `--read-only`

### K8s secrets cifrado — qué EVITAR
**"No GCM, No CBC"** → Evitar AES-Galois/Counter Mode y Cipher Block Chaining → usar KMS

### Cloud Network — VPC componentes
**"VPC → IGW (público) → NAT (privado) → Transit GW (on-prem) → VPC Endpoint (sin Internet)"**

---

## 4. Flashcards

**Q:** ¿Cuáles son las 4 características principales de un CASB?
**A:** Visibility (shadow IT + actividad en apps), Data Security (cifrado, tokenización, DLP), Threat Protection (amenazas internas, cuentas comprometidas), Compliance (data residency, políticas DLP).

---

**Q:** ¿Cuáles son las tres entidades del protocolo SAML?
**A:** Client/User, Service Provider (SP) e Identity Provider (IdP).

---

**Q:** ¿Qué tres assertions XML genera el IdP en SAML?
**A:** Authentication assertion (tipo de login), Attribute assertion (detalles del usuario), Authorization assertion (admitir o denegar acceso).

---

**Q:** ¿Cuál es la diferencia entre un control Deterrent y un control Preventive?
**A:** Deterrent: reduce la probabilidad de ataque mediante advertencias (cartel). Preventive: elimina o minimiza la vulnerabilidad técnicamente (autenticación fuerte). Deterrent advierte; Preventive bloquea.

---

**Q:** ¿Qué método de segmentación usa Zero Trust para limitar la propagación de brechas?
**A:** Micro-segmentation: divide la red en zonas más pequeñas con acceso separado. Si hay brecha en un perímetro, la micro-segmentation previene explotación lateral.

---

**Q:** ¿Cómo acceden a Internet las VMs en una subnet privada de AWS?
**A:** Via NAT gateway (ubicado en la public subnet). El NAT no permite tráfico entrante directo desde Internet, lo que mantiene la subnet privada.

---

**Q:** ¿Qué diferencia hay entre un Interface Endpoint y un Gateway-load-balancer Endpoint en VPC?
**A:** Interface endpoint: ENI con IP privada; punto de entrada para tráfico hacia VPC/servicios cloud. Gateway-load-balancer endpoint: ENI que desvía tráfico a un servicio de inspección de seguridad configurado via gateway load balancer.

---

**Q:** ¿Qué hace el flag `--icc=false` en Docker y cómo se habilita la comunicación entre contenedores específicos con este flag activo?
**A:** `--icc=false` deshabilita la comunicación entre contenedores (inter-container communication). Para permitir comunicación entre contenedores específicos: `--link=CONTAINER_NAME_or_ID:ALIAS`.

---

**Q:** ¿Cuál es el usuario por defecto de las imágenes Docker y cuál es la best practice?
**A:** Por defecto: root. Best practice: usar la directiva `USER` en el Dockerfile para especificar un usuario no privilegiado.

---

**Q:** ¿Qué modos de cifrado deben evitarse para cifrar Kubernetes Secrets en reposo?
**A:** AES-Galois/Counter Mode (AES-GCM) y Cipher Block Chaining (CBC). Usar KMS (Key Management Service).

---

**Q:** ¿Qué herramienta multi-cloud de auditoría de seguridad usa APIs de los CSPs, genera reporte HTML y usa códigos de color para priorizar riesgos?
**A:** Scout Suite.

---

**Q:** ¿Qué organización internacional sin ánimo de lucro promueve best practices y estándares de seguridad en cloud?
**A:** Cloud Security Alliance (CSA).

---

**Q:** ¿Qué herramienta de seguridad de contenedores escanea imágenes, VMs y funciones serverless buscando vulnerabilidades, secretos, malware y licencias open-source?
**A:** Aqua.

---

**Q:** ¿Cuáles son las features principales de NG SWG (Next-Generation Secure Web Gateway)?
**A:** URL filtering, decriptado TLS/SSL, operaciones CASB, Advanced Threat Protection (ATP) con sandboxing y ML, DLP para web y apps cloud, contextos de metadata cualitativos.

---

**Q:** ¿Qué herramienta de observabilidad y monitorización se usa para seguridad de aplicaciones serverless?
**A:** Dashbird (monitorización en tiempo real, detección de errores, visibilidad end-to-end para serverless).

---

**Q:** En el modelo de acceso control NIST para PaaS, ¿qué operación puede realizar el CSP sobre "application-related data"?
**A:** Replicate. El CSP puede replicar datos de la aplicación (no Write directo en este contexto).

---

**Q:** ¿Qué función de comparación debe usarse en K8s para evitar timing attacks en comparación de contraseñas?
**A:** `crypto.subtle.ConstantTimeCompare` (función de comparación en tiempo constante).

---

**Q:** ¿Dónde se sitúa un CASB en la arquitectura cloud?
**A:** Entre la infraestructura on-premises de la organización y la infraestructura del cloud provider. Actúa como gatekeeper extendiendo las políticas de seguridad más allá de la infraestructura propia.

---

## 5. Confusión frecuente

**CASB vs. NG SWG — funciones**
- CASB: específicamente para cloud apps; discovery de shadow IT; aplica políticas de seguridad, DLP y compliance entre on-premises y cloud. Ubicado entre infraestructura org y CSP.
- NG SWG: protege la red de amenazas cloud en general; incluye URL filtering, decriptado TLS, ATP con ML. Más orientado a protección de tráfico web/cloud saliente.
- Criterio: si la pregunta menciona shadow IT, cloud app discovery y compliance → CASB. Si menciona URL filtering, ATP, sandboxing → NG SWG.

---

**Deterrent vs. Preventive vs. Detective vs. Corrective**
- Deterrent: **advierte** antes del ataque (cartel). No bloquea técnicamente.
- Preventive: **bloquea** antes del ataque (autenticación fuerte, firewall).
- Detective: **detecta** durante el ataque (IDS/IPS).
- Corrective: **repara** después del ataque (restaurar backup).
- Criterio: si la pregunta describe una acción post-incidente → Corrective. Si describe monitorización activa → Detective. Si describe bloqueo técnico → Preventive. Si describe advertencia → Deterrent.

---

**SAML Authentication Assertion vs. Authorization Assertion**
- Authentication assertion: describe **cómo** se autenticó el usuario (contraseña, 2FA, etc.).
- Authorization assertion: describe **si** el usuario puede acceder al servicio (admit/deny).
- Attribute assertion: describe **quién** es el usuario (atributos específicos).
- Criterio: si la pregunta habla del método de login → authentication. Si habla de acceso permitido/denegado → authorization.

---

**VPC Endpoint vs. Transit Gateway — qué evitan**
- VPC Endpoint: evita que el tráfico entre VPC y servicio cloud salga a Internet. Conexión privada directa.
- Transit Gateway: evita las conexiones de peering complejas entre múltiples VPCs y la red on-premises. Centraliza el routing.
- Criterio: si el objetivo es comunicación privada con un servicio cloud sin Internet → VPC Endpoint. Si el objetivo es simplificar la topología de red entre VPCs y on-premises → Transit Gateway.

---

**Scout Suite vs. CloudSploit vs. Prowler — herramientas de auditoría cloud**
- Scout Suite: multi-cloud (AWS/Azure/GCP); auditoría de postura de seguridad; reporte HTML con color-coding.
- CloudSploit: identificación de misconfigurations; mapping a HIPAA/PCI/CIS; output tabular/CSV.
- Prowler: 240+ controles; frameworks amplios (NIST, FedRAMP, ENS); línea de comandos con múltiples formatos.
- Criterio: multi-cloud + reporte HTML visual → Scout Suite. Misconfigurations + compliance mapping específico → CloudSploit. 240+ controles + ENS español → Prowler.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un CISO necesita implementar una solución que monitorice y controle todo el tráfico entre usuarios y servicios cloud (SaaS, IaaS, PaaS), aplicando políticas de seguridad, DLP y visibilidad en tiempo real. ¿Qué tecnología cubre estos requisitos?

A) WAF (Web Application Firewall)  
B) CASB (Cloud Access Security Broker)  
C) VPC Endpoint  
D) SIEM  

**Respuesta correcta: B**
Un **CASB (Cloud Access Security Broker)** actúa como punto de control entre usuarios y servicios cloud, aplicando: visibilidad, DLP, control de acceso, detección de amenazas, y compliance. Es la solución de seguridad cloud que cubre monitorización de tráfico cloud en tiempo real. La contramedida de MITC Attack también es CASB.

---

**P2.** Una organización quiere que el tráfico entre sus instancias EC2 y el servicio S3 no salga a Internet pública. ¿Qué servicio de AWS implementa para garantizar una conexión privada?

A) Transit Gateway  
B) Security Group  
C) VPC Endpoint  
D) NAT Gateway  

**Respuesta correcta: C**
**VPC Endpoint** permite una **conexión privada directa** entre la VPC y servicios AWS (como S3) sin que el tráfico salga a Internet. Un NAT Gateway sí envía tráfico a Internet. Transit Gateway simplifica la conectividad entre VPCs. Security Groups controlan el tráfico pero no evitan el tránsito por Internet.

---

**P3.** Un arquitecto de seguridad cloud implementa el principio de **Defense in Depth** para proteger una aplicación en AWS. ¿Qué combinación de controles cubre múltiples capas correctamente?

A) Solo WAF en la capa de aplicación  
B) Security Groups + NACLs + WAF + VPC + IAM Roles con least privilege  
C) Solo IAM con MFA habilitado  
D) Solo cifrado AES-256 para todos los datos  

**Respuesta correcta: B**
**Defense in Depth** en cloud requiere controles en múltiples capas: **Security Groups** (control de red a nivel de instancia), **NACLs** (control de red a nivel de subred), **WAF** (capa de aplicación), **VPC** (aislamiento de red), y **IAM Roles con least privilege** (control de identidad y acceso). Un único control no es Defense in Depth.

---

**P4.** Un equipo de seguridad implementa cifrado para datos en reposo en S3 y DynamoDB. El CISO requiere que las claves de cifrado estén almacenadas de forma segura y sean gestionadas por la organización, no por AWS. ¿Qué servicio deben usar?

A) AWS Secrets Manager  
B) AWS KMS con Customer Managed Keys (CMK)  
C) AWS Parameter Store  
D) IAM con encryption policies  

**Respuesta correcta: B**
**AWS KMS con Customer Managed Keys (CMK)** permite que la organización gestione sus propias claves de cifrado, con control total sobre rotación, auditoría y revocación. AWS Secrets Manager almacena secrets (credenciales, API keys), no claves de cifrado de datos. Parameter Store es para configuración, no gestión de claves de cifrado.

---

**P5.** Un CISO evalúa el modelo de responsabilidad compartida de cloud. En un servicio **IaaS**, ¿cuál es la responsabilidad del cliente respecto a la seguridad del SO de las VMs?

A) El CSP gestiona completamente el SO, incluyendo patches  
B) El cliente es responsable del patch management y configuración del SO guest  
C) La responsabilidad es compartida 50/50 entre CSP y cliente  
D) Solo el CSP tiene acceso al SO en IaaS  

**Respuesta correcta: B**
En el modelo de **responsabilidad compartida IaaS**, el cliente es **responsable del SO guest** (incluyendo patch management, hardening y configuración), las aplicaciones y los datos. El CSP gestiona la infraestructura física, la red de la plataforma y el hypervisor. Esta distinción es fundamental: IaaS → el cliente gestiona el SO.

---

**P6.** Un cloud security engineer implementa logging centralizado en AWS. Quiere capturar todas las llamadas API realizadas en la cuenta AWS para auditoría y detección de amenazas. ¿Qué servicio debe habilitar?

A) AWS Config  
B) AWS CloudTrail  
C) VPC Flow Logs  
D) AWS GuardDuty  

**Respuesta correcta: B**
**AWS CloudTrail** registra todas las **llamadas API realizadas en la cuenta AWS** (quién hizo qué, cuándo y desde dónde). Es el servicio de auditoría principal para cumplimiento e investigación de incidentes. VPC Flow Logs registra tráfico de red. Config rastrea cambios en configuración. GuardDuty es threat detection (usa CloudTrail entre sus fuentes).

---

**P7.** Una organización implementa una **Zero Trust Architecture** para su entorno cloud. ¿Cuál es el principio core de Zero Trust?

A) Confiar en todo el tráfico dentro de la VPC corporativa  
B) "Never trust, always verify" — ningún usuario o dispositivo es confiable por defecto, independientemente de su ubicación  
C) Aplicar MFA solo para accesos desde redes externas  
D) Cifrar solo los datos clasificados como sensibles  

**Respuesta correcta: B**
El principio core de **Zero Trust** es **"Never trust, always verify"**: ningún usuario, dispositivo o conexión es confiable por defecto, independientemente de si está dentro o fuera de la red corporativa. Cada acceso debe ser verificado explícitamente, con mínimos privilegios y monitorización continua.

---

**P8.** Un CISO quiere implementar controles de seguridad para prevenir el **Cloud Cryptojacking**. ¿Cuál es la señal de alerta más específica y cuál es la contramedida principal?

A) Señal: tráfico DNS sospechoso; contramedida: DNSSEC  
B) Señal: aumento en los intentos de login fallidos; contramedida: MFA  
C) Señal: aumento repentino en la factura de recursos cloud; contramedida: límites de presupuesto + alertas de gasto anómalo  
D) Señal: latencia elevada en las APIs; contramedida: CDN  

**Respuesta correcta: C**
La señal específica de **Cloud Cryptojacking** es el **aumento repentino e inesperado en la factura de recursos cloud** (CPU/GPU, red). Las contramedidas incluyen: límites de presupuesto en el CSP, alertas de gasto anómalo, monitorización de CPU/GPU, Content Security Policy para bloquear scripts de minería, y auditorías regulares de permisos cloud.
