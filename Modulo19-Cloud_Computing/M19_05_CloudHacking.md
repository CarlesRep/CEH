# M19_05 — Cloud Hacking
**Módulo 19 / Subapartado 5 — Cloud Hacking**

---

## 1. Conceptos y definiciones

### Los tres elementos que los atacantes comprometen en cloud

1. **Web Applications Hacking**: APIs cloud expuestas en Internet → reconocimiento de endpoints → explotación de autenticación insuficiente, flaws de autorización, o injection. Una brecha puede comprometer múltiples tenants.
2. **System Hacking**: VMs, contenedores, y funciones serverless → reconocimiento de interfaces expuestas y protocolos débiles → escalada y persistencia.
3. **Cloud Hacking** (propiamente): vulnerabilidades en las tecnologías cloud mismas → acceso a datos de usuario y bloqueo de servicios. Incluye cloud storage, configuraciones incorrectas, contraseñas débiles, software sin parchear.

> **Nota de ética**: cada CSP (AWS, Azure, GCP) tiene reglas específicas sobre ethical hacking. El proveedor debe ser notificado antes de cualquier operación. El ethical hacking en cloud suele ser factible solo por medios internos.

---

### 🔴 Metodología de Cloud Hacking — 4 fases

| Fase | Objetivo | Herramientas clave |
|------|---------|-------------------|
| **1. Information Gathering** | Recopilar topología de red, IPs, dominios, subdominios, cuentas de usuario, info pública | Nmap, **Shodan**, **Censys**, Reconng |
| **2. Vulnerability Assessment** | Identificar misconfigurations, software sin parchear, flaws en redes/apps/servicios cloud | **Tenable Nessus**, **OpenVAS**, **Qualys** |
| **3. Exploitation** | Explotar vulnerabilidades para acceso no autorizado o control sobre la infraestructura | **Metasploit**, **sqlmap**, **thc-hydra**; scripts custom, injection, bypass de auth |
| **4. Post-Exploitation** | Mantener acceso, cubrir rastros, profundizar en la red, exfiltrar datos | **Cobalt Strike**, **Metasploit**; backdoors, escalada de privilegios, C2 channels |

---

### 🔴 Herramientas de Cloud Hacking — sintaxis y uso

#### Shodan — filtros de búsqueda para infraestructura cloud

| Filtro Shodan | Qué encuentra |
|--------------|--------------|
| `port:443` | Servicios HTTPS (web services en cloud) |
| `ssl.cert.issuer.cn:Amazon` | Certificados SSL emitidos por Amazon → servicios AWS |
| `cloud.region:<Region_code>` | Activos en una región geográfica específica del CSP |
| `org:Microsoft` | Dispositivos y servicios de Microsoft → infraestructura Azure |
| `product:Kubernetes` | Instancias de Kubernetes |
| `org:Amazon` | Servicios en infraestructura Amazon (AWS) |
| `ssl.cert.subject.cn:azure` | Certificados SSL con Azure en el Common Name → servicios Azure |
| `tag:cloud` | Activos etiquetados como parte de infraestructura cloud (**solo usuarios enterprise**) |
| `net:52.0.0.0/8` | Búsqueda en rango IP específico (ej. rango AWS) |
| `http.html:"s3.amazonaws.com"` | Instancias con HTML que menciona S3 → buckets S3 de AWS |
| `Amazon web services Facebook` | Infraestructura AWS usada por Facebook (IPs, puertos, service banners, S3, EC2) |

---

#### Masscan — escaneo de puertos en cloud

Diseñado para escanear **grandes redes e Internet en minutos**.

```bash
# Escanear todos los puertos de una IP
sudo masscan -p0-65535 <target_IP_address> --rate=<rate>

# Guardar resultados en XML
sudo masscan -p0-65535 <target_IP_address> --rate=<rate> -oX <scan_results>.xml

# Guardar resultados en JSON
sudo masscan -p0-65535 <target_IP_address> --rate=<rate> -oJ scan_results.json
```

Opciones clave:
- `-p0-65535`: escanea todos los puertos (0 a 65535)
- `--rate=<rate>`: paquetes por segundo
- `-oX`: output en **XML**
- `-oJ`: output en **JSON**

---

#### Prowler — vulnerability scanning en cloud

Más de **240 controles** que cubren: CIS, NIST 800, NIST CSF, CISA, RBI, FedRAMP, PCI-DSS, GDPR, HIPAA, FFIEC, SOC2, GXP, AWS Well-Architected Framework Security Pillar, AWS FTR, ENS (Esquema Nacional de Seguridad español).

```bash
# Iniciar escaneo especificando el proveedor
prowler <provider>

# Generar reporte en múltiples formatos
prowler <provider> -M csv json-asff json-ocsf html

# Checks específicos
prowler azure --checks storage_blob_public_access_level_is_disabled
prowler aws --services s3 ec2
prowler gcp --services iam compute
prowler kubernetes --services etcd apiserver

# AWS: perfil específico y filtro de región
prowler aws --profile custom-profile --filter-region <region_1> <region_2>

# Azure: suscripción específica
prowler azure --az-cli-auth --subscription-ids <subscription_ID_1> ...

# GCP: proyecto específico
prowler gcp --project-ids <Project_ID_1> <Project_ID_2> ...
```

Formatos de reporte por defecto: **CSV, JSON-OCSF, JSON-ASFF, HTML**.

---

#### CloudSploit — identificación de misconfigurations

Identifica misconfigurations y riesgos de seguridad en recursos cloud (AWS, Azure, GCP). Soporta mapping a frameworks: **HIPAA, CIS Benchmarks, PCI**.

**Configuración de credenciales por proveedor:**

```json
// AWS
{"accessKeyId": "YOURACCESSKEY", "secretAccessKey": "YOURSECRETKEY"}

// Azure
{"ApplicationID": "...", "KeyValue": "...", "DirectoryID": "...", "SubscriptionID": "..."}

// GCP
{"type": "service_account", "project": "...", "client_email": "...", "private_key": "..."}
```

```bash
# Escaneo estándar
./index.js

# Compliance mapping
./index.js --compliance=hipaa
./index.js --compliance=pci
./index.js --compliance=cis

# Output en texto plano
./index.js --console=text

# Output tabla en consola + guardar CSV
./index.js --csv=file.csv --console=table
```

---

### Cleanup y mantenimiento de sigilo (Post-Exploitation)

Tras comprometer el entorno cloud, los atacantes aplican:

| Técnica | Descripción |
|---------|------------|
| **Log Manipulation** | Borrar o modificar logs para eliminar entradas de acciones maliciosas |
| **Removing Credentials and Access Management** | Eliminar credenciales temporales (tokens/keys); crear backdoors ocultos (cuentas o métodos de acceso que se mezclan con actividad legítima) |
| **Manipulating System and Service Configurations** | Eliminar cambios visibles del ataque; deshabilitar o modificar alertas que revelarían su presencia |
| **Implementing Persistence Mechanisms** | Ocultar código malicioso en procesos/servicios legítimos; usar herramientas built-in del cloud para evitar sospechas |

---

## 2. Exam Traps ⚠️

⚠️ **[Shodan — filtro tag:cloud]**
El filtro `tag:cloud` en Shodan está disponible **únicamente para usuarios enterprise**. El examen puede presentarlo como disponible para todos.

⚠️ **[Masscan — rango de puertos]**
La sintaxis correcta para escanear todos los puertos es `-p0-65535`. El examen puede presentar `-p1-65535` (que excluye el puerto 0) como equivalente.

⚠️ **[Masscan — formatos de output]**
`-oX` genera **XML**. `-oJ` genera **JSON**. No confundirlos. El examen puede invertir los flags.

⚠️ **[Prowler — número de controles]**
Prowler cubre más de **240 controles**. El examen puede presentar un número diferente. Dato concreto a memorizar.

⚠️ **[Prowler — frameworks cubiertos]**
Prowler incluye el **ENS (Esquema Nacional de Seguridad español)**. El examen puede preguntar sobre cobertura de frameworks específicos.

⚠️ **[Prowler — formatos de reporte por defecto]**
Los formatos por defecto son: **CSV, JSON-OCSF, JSON-ASFF y HTML**. Se especifican con `-M` o `--output-modes`.

⚠️ **[CloudSploit — compliance]**
CloudSploit soporta mapping a **HIPAA, CIS Benchmarks y PCI**. No todos los frameworks de Prowler. El examen puede mezclar las capacidades de ambas herramientas.

⚠️ **[Fases de cloud hacking — orden]**
El orden es: Information Gathering → Vulnerability Assessment → Exploitation → **Post-Exploitation**. El examen puede omitir Post-Exploitation o invertir fases intermedias.

⚠️ **[Herramientas por fase — no mezclar]**
Information Gathering: Shodan, Censys, Nmap, Reconng. Vulnerability Assessment: Nessus, OpenVAS, Qualys. Exploitation: Metasploit, sqlmap, thc-hydra. Post-Exploitation: Cobalt Strike, Metasploit. El examen puede asignar herramientas a fases incorrectas.

⚠️ **[Ethical hacking en cloud — notificación]**
Antes de cualquier operación de ethical hacking en cloud, el **proveedor debe ser notificado**. Algunas operaciones que causan disrupciones de servicio pueden estar restringidas incluso para pentesters.

---

## 3. Nemotécnicos

### 4 fases del cloud hacking
**"Info-Vuln-Exploit-Post"**
→ **I**nformation Gathering | **V**ulnerability Assessment | **E**xploitation | **P**ost-Exploitation

### Herramientas por fase
- **Fase 1** (Info): **"ShoNaCe"** → **Sho**dan | **N**map | **Ce**nsys | Reconng
- **Fase 2** (Vuln): **"TNQ"** → **T**enable Nessus | **N**essus/OpenVAS | **Q**ualys
- **Fase 3** (Exploit): **"MeSH"** → **Me**tasploit | **S**qlmap | t**H**c-hydra
- **Fase 4** (Post): **"CoMet"** → **Co**balt Strike | **Met**asploit

### Shodan filtros cloud — diferenciadores
- AWS por cert: `ssl.cert.issuer.cn:Amazon`
- Azure por cert: `ssl.cert.subject.cn:azure`
- Azure por org: `org:Microsoft`
- AWS por org: `org:Amazon`
- K8s: `product:Kubernetes`
- S3: `http.html:"s3.amazonaws.com"`
- IP range AWS: `net:52.0.0.0/8`

### Masscan output flags
**"XML = X, JSON = J"** → `-oX` para XML | `-oJ` para JSON

### CloudSploit compliance flags
**"HiPCi"** → `--compliance=hipaa` | `--compliance=pci` | `--compliance=cis`

---

## 4. Flashcards

**Q:** ¿Cuáles son las 4 fases de la metodología de cloud hacking y en qué orden se ejecutan?
**A:** 1) Information Gathering, 2) Vulnerability Assessment, 3) Exploitation, 4) Post-Exploitation.

---

**Q:** ¿Qué herramientas se usan en la fase de Information Gathering de cloud hacking?
**A:** Nmap, Shodan, Censys y Reconng.

---

**Q:** ¿Qué herramientas se usan en la fase de Vulnerability Assessment de cloud hacking?
**A:** Tenable Nessus, OpenVAS y Qualys.

---

**Q:** ¿Qué herramientas se usan en la fase de Exploitation de cloud hacking?
**A:** Metasploit, sqlmap y thc-hydra.

---

**Q:** ¿Qué herramientas se usan en la fase de Post-Exploitation de cloud hacking?
**A:** Cobalt Strike y Metasploit.

---

**Q:** ¿Qué filtro de Shodan permite encontrar instancias de Kubernetes?
**A:** `product:Kubernetes`

---

**Q:** ¿Cómo se diferencia en Shodan una búsqueda de AWS de una búsqueda de Azure usando certificados SSL?
**A:** AWS: `ssl.cert.issuer.cn:Amazon` (emisor del cert). Azure: `ssl.cert.subject.cn:azure` (subject del cert).

---

**Q:** ¿Para qué tipo de usuarios está disponible el filtro `tag:cloud` en Shodan?
**A:** Solo para usuarios enterprise.

---

**Q:** ¿Cuál es la sintaxis de Masscan para escanear todos los puertos de una IP y guardar en XML?
**A:** `sudo masscan -p0-65535 <target_IP> --rate=<rate> -oX <scan_results>.xml`

---

**Q:** ¿Cuántos controles cubre Prowler y qué frameworks incluye?
**A:** Más de 240 controles. Incluye CIS, NIST 800, NIST CSF, CISA, RBI, FedRAMP, PCI-DSS, GDPR, HIPAA, FFIEC, SOC2, GXP, AWS Well-Architected, AWS FTR y ENS (Esquema Nacional de Seguridad español).

---

**Q:** ¿Qué comando de Prowler ejecuta un scan de servicios S3 y EC2 en AWS?
**A:** `prowler aws --services s3 ec2`

---

**Q:** ¿Cuáles son los formatos de reporte que genera Prowler por defecto?
**A:** CSV, JSON-OCSF, JSON-ASFF y HTML. Se configuran con `-M` o `--output-modes`.

---

**Q:** ¿Qué hace CloudSploit y qué frameworks de compliance soporta?
**A:** Identifica misconfigurations y riesgos de seguridad en cloud (AWS, Azure, GCP). Soporta mapping a HIPAA, CIS Benchmarks y PCI.

---

**Q:** ¿Cómo se ejecuta un HIPAA compliance scan con CloudSploit?
**A:** `./index.js --compliance=hipaa`

---

**Q:** ¿Cuáles son las 4 técnicas de cleanup y mantenimiento de sigilo tras comprometer un entorno cloud?
**A:** Log Manipulation, Removing Credentials and Access Management, Manipulating System and Service Configurations, e Implementing Persistence Mechanisms.

---

**Q:** ¿Qué rango IP de Shodan corresponde a un rango AWS conocido?
**A:** `net:52.0.0.0/8` es un rango de IP común para AWS.

---

**Q:** ¿Cómo escanea Prowler un proyecto específico en GCP?
**A:** `prowler gcp --project-ids <Project_ID_1> <Project_ID_2> ...`

---

**Q:** ¿Por qué el ethical hacking en cloud suele realizarse solo por medios internos?
**A:** Para garantizar el cumplimiento de las políticas de seguridad del CSP, evitar accesos no autorizados y respetar las restricciones del proveedor sobre operaciones que puedan causar disrupciones de servicio.

---

## 5. Confusión frecuente

**Shodan vs. Censys — uso en cloud hacking**
- Shodan: motor de búsqueda de dispositivos y servicios expuestos en Internet; con filtros específicos de cloud (org, ssl, product, net, tag).
- Censys: similar a Shodan pero con mayor énfasis en certificados TLS/SSL y datos de escaneo de Internet; ambos se usan en Information Gathering.
- Criterio: si la pregunta menciona filtros cloud específicos o sintaxis de búsqueda → Shodan. Si menciona análisis de certificados de forma general → cualquiera de los dos.

---

**Prowler vs. CloudSploit — alcance**
- Prowler: orientado a vulnerability scanning con más de 240 controles y múltiples frameworks de compliance (incluyendo ENS, NIST, FedRAMP).
- CloudSploit: orientado específicamente a identificación de misconfigurations con mapping a HIPAA, PCI y CIS. Genera reports detallados para análisis posterior.
- Criterio: si la pregunta habla de controles de compliance amplios o ENS → Prowler. Si habla específicamente de misconfigurations y análisis de configuración → CloudSploit.

---

**Masscan vs. Nmap — cuándo usar cada uno**
- Masscan: diseñado para escanear **grandes redes o Internet completa en minutos**. Alta velocidad, control de rate.
- Nmap: escaneo más detallado, con detección de versiones, OS fingerprinting y scripts NSE. Más lento pero más informativo.
- Criterio: si el escenario requiere velocidad y escala (toda Internet, rangos IP grandes de AWS) → Masscan. Si requiere detalle y fingerprinting → Nmap.

---

**Post-Exploitation vs. Cleanup — relación**
- Post-Exploitation es la fase que engloba: mantener acceso, cubrir rastros (cleanup) y profundizar en la red.
- Cleanup es una **técnica dentro de Post-Exploitation**, no una fase separada.
- Criterio: el examen puede preguntar en qué fase se produce el borrado de logs → Post-Exploitation.

---

**Cobalt Strike vs. Metasploit — fase de uso**
- Ambos se usan en **Post-Exploitation** para C2, backdoors y mantenimiento de acceso.
- Metasploit también se usa en **Exploitation** para explotar vulnerabilidades inicialmente.
- Criterio: si la pregunta pregunta qué herramienta se usa para explotar una vulnerabilidad → Metasploit. Si pregunta sobre C2 y persistencia → ambos, pero Cobalt Strike es más específico de post-explotación.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un pentester usa Shodan para descubrir instancias AWS expuestas. Quiere filtrar por el rango de IPs conocido de AWS. ¿Qué filtro de Shodan debe usar?

A) `org:"Amazon"` seguido de `port:22`  
B) `net:52.0.0.0/8`  
C) `cloud:aws tag:cloud`  
D) `ssl.cert.subject.cn:"amazonaws.com"`  

**Respuesta correcta: B**
`net:52.0.0.0/8` es el filtro de Shodan para el **rango de IP conocido de AWS**. El filtro `net:` en Shodan permite buscar dentro de rangos de red CIDR específicos. `org:"Amazon"` también es válido pero menos preciso al incluir toda la organización Amazon.

---

**P2.** Durante la fase de Information Gathering de un cloud pentest, el atacante descubre que la organización usa S3 buckets. Quiere verificar si alguno está mal configurado y es públicamente accesible. ¿En qué fase del cloud hacking methodology se enmarca esta actividad?

A) Fase 3 — Exploitation  
B) Fase 2 — Enumeration  
C) Fase 4 — Post-Exploitation  
D) Fase 1 — Information Gathering  

**Respuesta correcta: D**
La **Fase 1 — Information Gathering** incluye la identificación de servicios cloud expuestos, incluyendo S3 buckets públicos y otros recursos mal configurados. Es la fase de reconocimiento antes de cualquier explotación activa.

---

**P3.** Un auditor usa **Prowler** en un entorno GCP con varios proyectos. ¿Cuál es la sintaxis correcta para escanear proyectos específicos?

A) `prowler gcp scan --projects <project_id>`  
B) `prowler gcp --project-ids <Project_ID_1> <Project_ID_2>`  
C) `prowler --cloud gcp --id <project_id>`  
D) `prowler scan gcp /project/<project_id>`  

**Respuesta correcta: B**
La sintaxis correcta de Prowler para GCP es: `prowler gcp --project-ids <Project_ID_1> <Project_ID_2> ...`. Prowler tiene más de 240 controles y soporta múltiples frameworks de compliance como ENS, NIST y FedRAMP.

---

**P4.** Un pentester necesita escanear rápidamente todo el rango de IPs de AWS (millones de hosts) en busca de puertos abiertos. ¿Qué herramienta es más adecuada por su capacidad de escanear grandes redes a alta velocidad?

A) Nmap  
B) Nikto  
C) Masscan  
D) Burp Suite  

**Respuesta correcta: C**
**Masscan** está diseñado para escanear **grandes redes o Internet completa en minutos** gracias a su alta velocidad y control de rate. Nmap es más detallado pero más lento, adecuado para fingerprinting de hosts específicos, no para escanear rangos de millones de hosts.

---

**P5.** Tras comprometer una instancia EC2, un pentester usa Metasploit y Cobalt Strike para establecer persistencia y un canal de C2. ¿En qué fase de la cloud hacking methodology se encuentran estas actividades?

A) Fase 1 — Information Gathering  
B) Fase 2 — Enumeration  
C) Fase 3 — Exploitation  
D) Fase 4 — Post-Exploitation  

**Respuesta correcta: D**
**Metasploit y Cobalt Strike** se usan en la **Fase 4 — Post-Exploitation** para mantener acceso (backdoors, C2) tras el compromiso inicial. Metasploit también puede usarse en explotación inicial, pero cuando se menciona junto a Cobalt Strike para persistencia y C2, el contexto es post-explotación.

---

**P6.** Un equipo de seguridad realiza un cloud pentest ético. El CISO pregunta si pueden ejecutar un ataque DDoS desde la infraestructura del CSP para probar la resiliencia. ¿Qué respuesta es correcta?

A) Sí, si tienen las credenciales de las instancias cloud  
B) Sí, es una prueba legítima con permiso del cliente  
C) No, el ethical hacking en cloud se realiza solo por medios internos respetando las políticas del CSP  
D) Sí, si el CSP no lo detecta no hay problema  

**Respuesta correcta: C**
El **ethical hacking en cloud** debe realizarse **solo por medios internos**, garantizando el cumplimiento de las políticas de seguridad del CSP, evitando accesos no autorizados y respetando las restricciones del proveedor sobre operaciones que puedan causar disrupciones de servicio a otros clientes.

---

**P7.** Un pentester detecta que una organización tiene S3 buckets con nombres predecibles basados en el nombre de la empresa. Descarga listados públicos de bucket names para encontrar coincidencias. ¿Qué técnica usa?

A) Google Dorking para enumerar S3  
B) DNS Brute Force de subdominios  
C) Bucket Name Enumeration usando listados de buckets conocidos  
D) Cloud API Key stealing  

**Respuesta correcta: C**
La **enumeración de S3 bucket names** es una técnica de reconnaissance en cloud. Los atacantes usan listados de nombres comunes o nombres predecibles basados en la organización objetivo (ej. `empresa-backup`, `empresa-prod`) para descubrir buckets mal configurados con acceso público.

---

**P8.** CloudSploit identifica que una organización tiene múltiples misconfigurations en su entorno cloud mapeadas a HIPAA y CIS. ¿Cuál es la principal diferencia entre CloudSploit y Prowler en este contexto?

A) CloudSploit escanea AWS; Prowler escanea Azure y GCP  
B) CloudSploit está orientado a identificación de misconfigurations con mapping HIPAA/PCI/CIS; Prowler tiene más de 240 controles con soporte a frameworks adicionales como ENS y NIST  
C) CloudSploit usa IA; Prowler usa reglas estáticas  
D) Son herramientas equivalentes sin diferencias significativas  

**Respuesta correcta: B**
**CloudSploit** está orientado específicamente a la identificación de **misconfigurations** con mapping a HIPAA, PCI y CIS. **Prowler** tiene más de **240 controles** y soporta frameworks adicionales como ENS, NIST y FedRAMP, con mayor cobertura de compliance. Ambas son herramientas de cloud security assessment pero con enfoques distintos.
