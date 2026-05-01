# M02 — Footprinting: Overview y Mapa de Técnicas

## ¿Qué es el Footprinting?

Recopilación de información sobre el target de **todas las fuentes disponibles** — siempre es **pasivo o semi-pasivo** (fase previa a interacción directa).

## 7 Categorías de Técnicas

```
Footprinting
├── 1. Search Engines        → Google Dorks, GHDB, SHODAN
├── 2. Internet Research     → People Search, Job Sites, Archive.org, Dark Web
├── 3. Social Networking     → RRSS, LinkedIn, email harvesting
├── 4. Whois Footprinting    → Whois Lookup, IP Geolocation
├── 5. DNS Footprinting      → DNS Interrogation, Reverse DNS
├── 6. Network & Email       → Traceroute, Email Header Analysis
└── 7. Social Engineering    → Eavesdropping, Shoulder Surfing, Dumpster Diving
```

## Conceptos clave — flashcard rápida

| Concepto | Una línea |
|---|---|
| **SHODAN** | Google de dispositivos conectados — NO páginas web |
| **GHDB** | Subconjunto de Exploit-DB — colección de Google Dorks |
| **Reverse DNS** | IP → hostname (inverso al DNS normal) |
| **Traceroute** | Mapea saltos de red entre atacante y objetivo |
| **Dumpster Diving** | Buscar info en basura física — también: *trashing* |
| **Shoulder Surfing** | Observar pantalla/teclado desde detrás — close-in attack |
| **archive.org** | Recupera páginas eliminadas — requiere solicitud activa para borrar |

## ⚠️ Distinción crítica

> **SHODAN vs Google:** Google = páginas web indexadas · SHODAN = dispositivos conectados (IoT, cámaras, SCADA, routers)
