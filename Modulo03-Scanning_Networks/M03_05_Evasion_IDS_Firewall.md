# M03 — Evasión de IDS y Firewall

## 11 Técnicas de Evasión

> 🧠 **Mnemotécnia:** **P**ackets **S**ource **S**poofed **I**P **I**P **M**AC **C**ustom **R**andom **B**ad **P**roxy **A**nonymizer

1. **Packet Fragmentation**
2. **Source Routing**
3. **Source Port Manipulation**
4. **IP Address Decoy**
5. **IP Address Spoofing**
6. **MAC Address Spoofing**
7. **Creating Custom Packets**
8. **Randomizing Host Order**
9. **Sending Bad Checksums**
10. **Proxy Servers**
11. **Anonymizers**

---

## 1. Packet Fragmentation

- Divide probe packets en **fragmentos pequeños**
- IDS: más CPU procesando → muchos los **omiten**
- Fragmentos se reasamblan en el destino
- **SYN/FIN scanning con IP fragments** = técnica derivada — evita signature-based IDS

---

## 2. Source Routing

- Manipula campo **IP options** para definir ruta del paquete
- El atacante fuerza ruta que **evita routers con firewall/IDS**
- Tipos: **Loose** (ruta aproximada) y **Strict** (ruta exacta)

---

## 3. Source Port Manipulation

- Reemplaza puerto origen con **puerto bien conocido** (HTTP=80, DNS=53, FTP=21)
- Firewalls mal configurados permiten tráfico de puertos de confianza

```bash
# Nmap source port manipulation
nmap -g 80 <target>
nmap --source-port 80 <target>
```

---

## 4. IP Address Decoy

- Genera múltiples IPs señuelo — IDS no puede distinguir IP real

```bash
nmap -D RND:10 <target>                          # 10 decoys aleatorios
nmap -D decoy1,decoy2,ME,decoy4 <target>         # Manual — ME = posición IP real
```

> ⚠️ No efectivo si target usa route tracing activo o response dropping

---

## 5. IP Address Spoofing

- Altera headers → usa IP falsa — target responde a IP spoofada
- Principal uso: **DoS attacks**

```bash
hping3 www.certifiedhacker.com -a 7.7.7.7        # Spoofar IP origen
```

> ⚠️ No se puede completar three-way handshake con IP spoofada

---

## 6. MAC Address Spoofing

```bash
nmap -sT -Pn --spoof-mac 0 <target>              # MAC aleatoria automática
nmap -sT -Pn --spoof-mac [Vendor] <target>       # MAC de vendor específico
nmap -sT -Pn --spoof-mac [new MAC] <target>      # MAC manual específica
```

---

## 7. Custom Packets

**Herramientas:** Colasoft Packet Builder · NetScanTools Pro

**Colasoft Packet Builder:** 3 vistas — Packet List · Decode Editor · Hex Editor
- Crea, edita y envía paquetes personalizados
- Soporta intervalos, loops y delays
- Crea paquetes fragmentados para bypassear firewall/IDS

---

## 8. Randomizing Host Order

```bash
nmap --randomize-hosts <target>    # Mezcla grupos de 16384 hosts
```
Menos detectable por sistemas de monitorización

---

## 9. Sending Bad Checksums

```bash
nmap --badsum <target>
```
- Respuesta → IDS/firewall **no verifica checksums** (mal configurado)
- Sin respuesta → sistema correctamente configurado

---

## 10. Proxy Servers

**Función:** intermediario entre atacante y target — oculta IP real

**Proxy Chaining:** mayor anonimato con múltiples proxies encadenados
```
Atacante → Proxy1 → Proxy2 → Proxy3 → Target
```
Cada proxy elimina info de identificación antes de pasar al siguiente.

**Herramientas:** Proxy Switcher · CyberGhost VPN · Burp Suite · Tor · Hotspot Shield · Proxifier

---

## 11. Anonymizers

- Servidor intermedio que accede webs en nombre del usuario
- Elimina IP e información identificativa

| Tipo | Descripción | Ventaja | Desventaja |
|---|---|---|---|
| **Networked** | Múltiples nodos intermedios | Análisis de tráfico muy complejo | Riesgo en cada nodo |
| **Single-Point** | Un único intermediario | Oculta IP efectivamente | Menos resistente a análisis sofisticado |

**Herramientas:** Whonix · Psiphon · TunnelBear · I2P · Bright Data
**Circumvention tools:** AstrillVPN · Tails (live OS en USB/SD — no deja rastro)

---

## Detección de IP Spoofing — 3 técnicas

| Técnica | Cómo funciona | Efectiva cuando |
|---|---|---|
| **Direct TTL Probes** | Comparar TTL recibido con TTL esperado | Atacante en subred **diferente** |
| **IP Identification Number (IPID)** | Monitorizar IPID incremental — si respuesta no es próxima al probe = spoofado | Incluso en **misma subred** |
| **TCP Flow Control** | Datos enviados después de window=0 = spoofados | Durante **handshake** principalmente |

---

## Countermeasures contra IP Spoofing

- No autenticación **solo por IP** — añadir contraseñas
- **Ingress filtering** — bloquea paquetes con IP fuera del rango definido
- **Egress filtering** — bloquea salida de paquetes con IP externa
- **ISNs aleatorios** — evitar secuencias predecibles
- **IPSec** — autenticación, integridad, confidencialidad
- Migrar a **IPv6** · Usar **VPN** en redes públicas

### 📝 Pregunta típica CEH

> *"Un atacante envía paquetes SYN con IPs de múltiples fuentes para confundir al IDS. ¿Qué técnica es?"*
> → **IP Address Decoy**

> *"¿Qué técnica envía paquetes con checksums incorrectos para identificar sistemas mal configurados?"*
> → **Sending Bad Checksums**

> *"¿Qué técnica de evasión manipula el campo IP options para definir la ruta del paquete?"*
> → **Source Routing**
