# M02 — Search Engines Avanzadas, IoT y OS Fingerprinting

## SHODAN vs Censys vs ZoomEye — IoT Search Engines

| Herramienta | Diferencial clave |
|---|---|
| **Shodan** | El más usado — indexa puertos, servicios, dispositivos — integra CVE, Metasploit, Exploit-DB, OSVDB, Packetstorm |
| **Censys** | Monitorización continua — descubre **activos desconocidos** — análisis de certificados |
| **ZoomEye** | Similar a Shodan — orientado al mercado chino |

> ⚠️ **Shodan vs Censys:** Shodan = búsqueda + integra exploits · Censys = monitorización continua + activos desconocidos
> ⚠️ **Shodan** = único que integra directamente búsqueda de exploits conocidos desde la misma interfaz

**Dispositivos objetivo IoT:** SCADA · control de tráfico · CCTV · electrodomésticos · appliances industriales

**Info obtenible:** fabricante · IP · hostname · ciudad · país · lat/long · puertos abiertos · servicios

**Uso ofensivo:** establecer backdoor → usar como launchpad

---

## OS Fingerprinting — Herramientas online

**Objetivo:** identificar OS de la infraestructura del target

| Herramienta | Qué obtiene | Diferencial |
|---|---|---|
| **Netcraft** | OS, sites asociados | "What's that site running?" — enfocado en webs |
| **Shodan** | OS, IP, hostname, geodatos | Integra exploits conocidos |
| **Censys** | OS, IP, protocolos, geolocalización | Monitoriza infraestructura completa |

---

## Otras Técnicas de Search Engine Footprinting

| Técnica | Herramientas | Qué obtiene |
|---|---|---|
| **Google Advanced Search** | google.com/advanced_search | Sin operadores — partners, vendors, clientes |
| **Google Advanced Image Search** | google.com/advanced_image_search | Imágenes del target, ubicaciones, empleados |
| **Reverse Image Search** | TinEye · Google Images · Bing · Pinterest | Origen y ubicaciones online de una imagen |
| **Video Search Engines** | YouTube · Google Videos · Bing Videos | Info oculta: fecha, thumbnail, metadata |
| **Video Analysis Tools** | YouTube Metadata · EZGif · VideoReverser.com | Metadata, inversión de vídeo, conversión a texto |
| **Meta Search Engines** | Startpage · MetaGer · eTools.ch | Múltiples motores simultáneos — **oculta la IP** |
| **FTP Search Engines** | NAPALM FTP Indexer · Mamont · Globalfilesearch.com | Ficheros en servidores FTP |

### ⚠️ Distinciones clave

| Par | Diferencia |
|---|---|
| **Meta Search vs normal** | Meta = sin índice propio — agrega resultados — más privacidad |
| **Reverse Image vs Image Search** | Image = buscar imágenes · Reverse = usar imagen como query |
| **FTP Engines vs Google Dorks FTP** | FTP engines = indexan servidores FTP directamente · Dorks = encuentran info FTP en web |

---

## Subdominios — Herramientas

**¿Por qué son útiles?** → suelen estar en pruebas → **menos seguros y más vulnerables**

**Dork para subdominios:** `site:microsoft.com -inurl:www`

| Herramienta | Qué obtiene |
|---|---|
| **Netcraft** | Subdominios, web servers, OS, SSL authorities |
| **DNSdumpster** | Subdominios, IPs, DNS servers |
| **Pentest-Tools Find Subdomains** | Subdominios, IPs, OS, tecnología, plataforma, page titles |

> ⚠️ **DNSdumpster vs Pentest-Tools:** DNS = enfocado en DNS · Pentest-Tools = más detalle técnico (OS, tecnología)

---

## archive.org — Wayback Machine

- Recupera versiones archivadas **desde la creación** del sitio
- Recupera info **eliminada**: páginas, imágenes, audio, vídeo, software
- Útil para phishing y web application attacks

```bash
# Recuperar URLs archivadas con wayback
photon.py -u <URL> -l 3 -t 200 --wayback

# Solo URLs
python photon.py -u <URL> -l 3 -t 200 --only-urls
```

> ⚠️ Eliminar historial de archive.org requiere **solicitud activa** — no es automático
