# M02 — Internet Research Services, Dark Web y Competitive Intelligence

## People Search Services

**Herramientas:** Spokeo · Intelius · pipl · BeenVerified · Whitepages · Instant Checkmate · PeekYou

**Qué revelan:** nombre · dirección · teléfono · email · fecha de nacimiento · fotos · familia · RRSS · registros criminales · propiedades · empresas propias

**Uso ofensivo:** base para ingeniería social → datos bancarios, tarjetas de crédito, historial

---

## Job Sites Footprinting

**Por qué son valiosas:** revelan involuntariamente la infraestructura técnica

| Tipo de oferta | Info técnica expuesta |
|---|---|
| Network Administrator | Firewalls, appliances, protocolos |
| DBA / Developer | OS, versiones de software, esquema de BD |
| SysAdmin | Hipervisores, VMs, servidores internos |

**Fuentes adicionales:** CVs de empleados → expertise técnico, tecnologías usadas

**Herramientas:** Dice · LinkedIn · Glassdoor · Simply Hired

> ⚠️ **People Search vs Job Sites:** People Search = info personal · Job Sites = info técnica de infraestructura
> ⚠️ **Ofertas vs CVs:** Ofertas = tecnologías de la empresa · CVs = expertise del empleado

---

## Competitive Intelligence

**Definición:** Recopilación **ética y legal** de info sobre competidores — NO espionaje industrial

| Enfoque | Técnicas |
|---|---|
| **Directo** | Trade shows, social engineering de empleados/clientes |
| **Indirecto** | Webs, job postings, RRSS, press releases, patentes, informes financieros |

### Herramientas por objetivo

| Objetivo | Herramienta | Foco |
|---|---|---|
| **Historial/identidad** | EDGAR (sec.gov) | Filings ante la SEC — info financiera corporativa |
| | D&B Hoovers | 120M registros de empresas |
| | LexisNexis | Registros legales y públicos |
| | Business Wire | Press releases y contenido multimedia |
| | Factiva | +33.000 fuentes globales |
| **Planes/estrategia** | MarketWatch | Noticias financieras en tiempo real |
| | Wall Street Transcript | Informes de industria + entrevistas CEOs |
| | Euromonitor | Investigación de mercados de consumo |
| | USPTO | Patentes y marcas registradas |
| | Experian | Estrategias de marketing digital de competidores |
| **Tráfico/SEO** | SEMRush | Keywords, AdWords, competidores en búsqueda |
| | SimilarWeb | Tráfico web, geografía, fuentes de referencia |
| | SERanking | Tráfico, competidores, PPC |

---

## Geolocalización del Target

**Uso ofensivo:** dumpster diving · surveillance · social engineering físico · acceso a edificios

| Herramienta | Qué permite |
|---|---|
| **Google Earth** | Imágenes 3D, street view, altitud, coordenadas |
| **Google Maps / Waze** | Direcciones, tráfico, landmarks |
| **Wikimapia** | Mapas colaborativos con info adicional |

**Objetivo:** localizar entradas · cámaras · puntos débiles en perímetro · recursos eléctricos

---

## Financial Services

**Herramientas:** Google Finance · MSN Money · Yahoo Finance · Investing.com

**Info:** valor de acciones · perfil de empresa · competidores · press releases · informes financieros

---

## Alerts y ORM

| Tipo | Herramientas | Qué hace |
|---|---|---|
| **Alertas** | Google Alerts · X Alerts · Giga Alerts | Notifica menciones del target en web/blogs/noticias |
| **ORM Tracking** | Mention · ReviewPush · Reputology | Monitoriza reputación online en tiempo real |

---

## Groups, Forums y Blogs

**Info obtenible de empleados:** nombre completo · lugar trabajo/residencia · teléfonos · emails · fotos

**Táctica:** registro con perfil falso en grupos de empleados (LinkedIn Groups, Google Groups)

**Búsqueda por:** FQDNs · IPs · usernames

---

## Public Source-Code Repositories

**Plataformas:** GitHub · GitLab · SourceForge · BitBucket

**Info sensible expuesta:**
- Ficheros de configuración
- Claves SSH y SSL privadas
- Código fuente con vulnerabilidades
- Librerías dinámicas

**Uso ofensivo:** spear phishing · social engineering · ataques a infraestructura

**Herramienta:** **Recon-ng** — framework CLI de reconocimiento web
