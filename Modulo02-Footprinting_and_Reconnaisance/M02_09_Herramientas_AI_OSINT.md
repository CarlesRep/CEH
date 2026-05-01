# M02 — Herramientas Automatizadas y AI-Powered OSINT

## Herramientas de Footprinting Automatizado

| Herramienta | Función diferencial |
|---|---|
| **Maltego** | Mapea **relaciones** entre personas, orgs, dominios, IPs, documentos — **visualización gráfica** |
| **Recon-ng** | Framework modular CLI — módulo `recon/domains-hosts/brute_hosts` para hosts |
| **FOCA** | Extrae **metadata** de documentos (Office, PDF, ODF) + DNS, PTR, IPs |
| **subfinder** | Subdominios pasivos — outputs: JSON, file, stdout |
| **OSINT Framework** | **Directorio categorizado** de herramientas OSINT gratuitas — árbol web |
| **Recon-Dog** | All-in-one vía APIs — Censys, Whois, port scan, CMS detection, honeypot, tecnologías |
| **BillCipher** | Info gathering web/IP — Python 2/3 y Ruby — DNS, Whois, port scan, zone transfer |

### ⚠️ Distinciones clave

| Par | Diferencia |
|---|---|
| **Maltego vs Recon-ng** | Maltego = visualización gráfica de relaciones · Recon-ng = framework CLI modular |
| **FOCA vs subfinder** | FOCA = metadata documentos + DNS/IP · subfinder = solo subdominios pasivos |
| **OSINT Framework vs resto** | **No ejecuta ataques** — es un directorio categorizado |
| **Recon-Dog vs BillCipher** | Recon-Dog = APIs externas (Censys, Shodan, Wappalyzer) · BillCipher = Python/Ruby multiplataforma |

---

## FOCA — Funciones específicas

| Función | Descripción |
|---|---|
| **Web Search** | Hosts y dominios en URLs del dominio principal |
| **DNS Search** | NS, MX, SPF → nuevos hosts |
| **IP Resolution** | Hostname → IP (contra DNS interno) |
| **PTR Scanning** | Más servidores en el mismo segmento IP |
| **Bing IP** | Dominios asociados a cada IP descubierta |
| **Common Names** | Ataques de diccionario contra DNS |

---

## OSINT Framework — Indicadores

| Indicador | Significado |
|---|---|
| **(T)** | Tool — se instala y ejecuta localmente |
| **(D)** | Google Dork |
| **(R)** | Requires registration |
| **(M)** | URL con término de búsqueda — debe editarse manualmente |

---

## AI-Powered OSINT Tools

| Herramienta | Función diferencial |
|---|---|
| **Taranis AI** | NLP + AI sobre noticias → reportes e informes PDF publicables |
| **OSS Insight** | +5 mil millones eventos GitHub — queries en lenguaje natural |
| **DorkGPT** | Genera y **refina** Google Dorks mediante GPT |
| **DorkGenius** | **Automatiza** Google Dorking completo |
| **Cylect.io** | Integra múltiples DBs en una interfaz |
| **ChatPDF** | Extrae info de PDFs mediante interfaz conversacional |
| **Bardeen.ai** | Automatiza recopilación de datos online |
| **DarkGPT** | GPT-4-200K para consultar **bases de datos filtradas/comprometidas** |
| **Explore AI** | Motor de búsqueda AI para **YouTube** |
| **AnyPicker** | Web scraper visual sin código |

### ⚠️ Distinciones clave

| Par | Diferencia |
|---|---|
| **DorkGPT vs DorkGenius** | DorkGPT = genera y *refina* · DorkGenius = *automatiza* el dorking completo |
| **DarkGPT** | Único para **DBs de datos filtrados/comprometidos** |
| **Explore AI** | Único especializado en **YouTube** como fuente OSINT |
| **Taranis AI vs OSS Insight** | Taranis = noticias → reportes · OSS Insight = GitHub → inteligencia técnica |

---

## Herramientas adicionales

Sudomy · theHarvester · whatweb · Raccoon · Orb · Web Check · OSINT.SH
