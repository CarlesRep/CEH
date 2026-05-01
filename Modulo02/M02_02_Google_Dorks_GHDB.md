# M02 — Google Dorks, GHDB y Aplicaciones

## Operadores de Google — Resumen completo

### Por dónde busca

| Operador | Busca en | Ejemplo |
|---|---|---|
| **site:** | Dominio/sitio | `site:microsoft.com` |
| **inurl:** | URL (1 palabra) | `inurl:admin` |
| **allinurl:** | URL (todas las palabras) | `allinurl:google career` |
| **intitle:** | Título (1 palabra) | `intitle:login` |
| **allintitle:** | Título (todas las palabras) | `allintitle:detect malware` |
| **intext:** | Cuerpo de la página | `intext:"vpn configuration"` |
| **inanchor:** | Texto ancla de enlaces (1) | `inanchor:Norton` |
| **allinanchor:** | Texto ancla (todas) | `allinanchor:best cloud service` |

### Por tipo de información

| Operador | Función |
|---|---|
| **filetype:** | Extensión de archivo (pdf, xls, docx…) |
| **cache:** | Versión en caché de Google |
| **link:** | Páginas que enlazan al URL dado |
| **related:** | Sitios similares al URL |
| **info:** | Información sobre esa página |
| **source:** | Solo en Google News |
| **phonebook:** | Números de teléfono de persona/org |
| **before:** / **after:** | Filtro por fecha de publicación |
| **location:** | Resultados de una ubicación específica |

> ⚠️ **site: vs source:** site = cualquier búsqueda web · source = solo Google News
> ⚠️ **info: vs related:** info = datos sobre esa página · related = páginas similares

### Google + AI (ShellGPT)

```bash
# Encontrar PDFs en eccouncil.org → guardar en recon1.txt
lynx --dump "http://www.google.com/search?q=site:eccouncil.org+filetype:pdf" \
| grep "http" | cut -d "=" -f2 | grep -o "http[^&]*" > recon1.txt
```

| Parte | Función |
|---|---|
| `lynx --dump` | Navegador CLI — vuelca resultado de Google |
| `grep "http"` | Filtra líneas con URLs |
| `cut -d "=" -f2` | Extrae segundo campo separado por `=` |
| `grep -o "http[^&]*"` | URL limpia sin parámetros tras `&` |
| `> recon1.txt` | Guarda en fichero |

---

## GHDB — Google Hacking Database

**Fuente:** exploit-db.com/google-hacking-database
**GHDB = subconjunto de Exploit-DB** — solo Google Dorks

### 14 Categorías (memorizar lista completa)

1. Footholds
2. Files Containing Usernames
3. Sensitive Directories
4. Web Server Detection
5. Vulnerable Files
6. Vulnerable Servers
7. Error Messages
8. Files Containing Juicy Info
9. Files Containing Passwords
10. Sensitive Online Shopping Info
11. Network or Vulnerability Data
12. Pages Containing Login Portals
13. Various Online Devices
14. Advisories and Vulnerabilities

### Usos ofensivos

| Uso | Clave |
|---|---|
| **Reconnaissance** | Ficheros, directorios y dispositivos expuestos |
| **Exploiting Misconfigs** | Servidores mal configurados → acceso no autorizado |
| **Finding Vulnerable Systems** | Software desactualizado → punto de entrada |
| **Credential Harvesting** | Users + passwords → brute force / credential stuffing |
| **Open Ports & Services** | Mapa de entry points |

### SearchSploit

- CLI para buscar en Exploit-DB **offline**
- Útil en redes **air-gapped**
- Copia local del repositorio

> ⚠️ **GHDB vs SearchSploit:** GHDB = online · SearchSploit = CLI offline sobre copia local

---

## VPN Footprinting — Dorks clave

| Objetivo | Dork |
|---|---|
| **Login portals genéricos** | `inurl:"/sslvpn_logon.shtml" intitle:"User Authentication"` |
| **VPN login** | `inurl:/sslvpn/Login/Login` |
| **Fortinet** | `intext:Please Login SSL VPN inurl:remote/login intext:FortiClient` |
| **Cisco ASA** | `intitle:"SSL VPN Service" + intext:"Your system administrator..."` |
| **Citrix/Netscaler** | `inurl:"/vpn/tmindex.html" vpn` |
| **Directorios OpenVPN** | `intitle:"index of" /etc/openvpn/` |
| **Claves OpenVPN** | `"-----BEGIN OpenVPN Static key V1-----" ext:key` |
| **Ficheros .ovpn** | `Index of / *.ovpn` |

> 🧠 **Patrón:** `intitle:"index of"` = directorios expuestos · `ext:key` = ficheros de claves

---

## FTP Footprinting — Dorks clave

| Dork | Objetivo |
|---|---|
| `intitle:"Index of ftp passwords"` | Ficheros con passwords |
| `intitle:"Index of" ws_ftp.ini` | Usernames + passwords FTP |
| `"ws_ftp.log" ext:log` | Sensitive directories |
| `allintitle:"CrushFTP WebInterface"` | Login portals + password reset |
| `intitle:"index of" inurl:ftp intext:admin` | Admin folders en FTP |
| `"index of" /ftp/logs` | Log files |

---

## Dark Web Footprinting — Dorks clave

| Objetivo | Dork |
|---|---|
| **PDFs confidenciales** | `filetype:pdf site:onion confidential` |
| **Passwords en configs** | `inurl:config filetype:txt password` |
| **Database dumps** | `filetype:sql site:onion dump` |
| **Claves privadas** | `filetype:key site:onion private` |
| **Código fuente** | `filetype:py site:onion "def "` |
| **Vulnerabilidades** | `filetype:txt inurl:exploit "security vulnerability"` |

> 🧠 **Patrón dark web:** `filetype:X site:onion keyword`

### Surface Web vs Deep Web vs Dark Web

| | Surface Web | Deep Web | Dark Web |
|---|---|---|---|
| **Indexado** | Sí | No | No |
| **Acceso** | Chrome, Firefox | Tor, WWW Virtual Library | Tor, ExoneraTor |
| **Relación** | — | Incluye Dark Web | Subconjunto del Deep Web |

**Herramientas Dark Web:** Tor Browser · ExoneraTor · OnionLand Search
