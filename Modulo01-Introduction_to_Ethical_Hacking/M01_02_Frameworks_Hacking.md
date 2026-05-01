# Módulo 1 — Frameworks de Hacking y Análisis de Intrusiones

## CEH Ethical Hacking Framework — 5 Fases

```
Reconnaissance → Vulnerability Scanning → Gaining Access → Maintaining Access → Clearing Tracks
```

| Fase | Sub-fases / Acciones | Clave |
|---|---|---|
| **1. Reconnaissance** | Footprinting · Scanning · Enumeration | Recopilación de información — pasiva y activa |
| **2. Vulnerability Scanning** | Examina sistemas, apps y controles existentes | Output: lista de loopholes para explotación |
| **3. Gaining Access** | Password cracking · buffer overflow · exploits | **Aquí ocurre el hacking real** — escalada de privilegios a admin/root |
| **4. Maintaining Access** | Instalar malware · robar credenciales · manipular datos | Cerrar vulnerabilidades para evitar otros atacantes — launchpad |
| **5. Clearing Tracks** | Modificar/borrar logs con log-wiping utilities | Permanecer indetectado |

### Fase 1 en detalle — Reconnaissance

| Sub-fase | Qué hace |
|---|---|
| **Footprinting** | Perfil del target: IP range, namespace, empleados, dominio (Whois) |
| **Scanning** | Identifica hosts activos, puertos abiertos, servicios — probing más profundo que recon |
| **Enumeration** | Conexión activa/directa — extrae usuarios, routing tables, shares, banners, grupos |

> **Recon pasivo** = sin contacto directo (OSINT, news) · **Recon activo** = interacción directa con el sistema

### Distinciones críticas

| Par | Diferencia |
|---|---|
| **Scanning vs Enumeration** | Scanning = detecta qué hay abierto · Enumeration = extrae info concreta mediante conexión activa |
| **Recon pasivo vs activo** | Pasivo = sin tocar el target · Activo = interacción directa (se solapa con scanning) |
| **Gaining vs Maintaining Access** | Gaining = entrar · Maintaining = quedarse y controlar |

---

## Cyber Kill Chain — Lockheed Martin — 7 Fases

```
Reconnaissance → Weaponization → Delivery → Exploitation → Installation → C2 → Actions on Objectives
```

| Fase | Qué hace el adversario |
|---|---|
| **1. Reconnaissance** | Recopila info del target: OSINT, Whois, DNS, scanning de puertos/servicios |
| **2. Weaponization** | Crea o adapta el payload malicioso (malware + exploit + backdoor) según vulnerabilidades identificadas |
| **3. Delivery** | Transmite el payload — email adjunto, link malicioso, USB, watering hole |
| **4. Exploitation** | El código malicioso se ejecuta explotando una vulnerabilidad de OS/app/servidor |
| **5. Installation** | Instala backdoor/malware adicional para persistencia — oculta actividad (cifrado) |
| **6. Command & Control (C2)** | Establece canal bidireccional víctima↔servidor atacante — usa web, email, DNS para camuflarlo |
| **7. Actions on Objectives** | Ejecuta el objetivo final: robo de datos, disrupción, destrucción, launchpad para nuevos ataques |

### Distinciones críticas

| Par | Diferencia |
|---|---|
| **Weaponization vs Exploitation** | Weaponization = *crear* el arma · Exploitation = *detonarla* en el sistema víctima |
| **Delivery vs Exploitation** | Delivery = el payload *llega* · Exploitation = el payload *se ejecuta* |
| **Installation vs C2** | Installation = persistencia local · C2 = control remoto bidireccional |

---

## MITRE ATT&CK Framework

**Base de conocimiento global** de tácticas y técnicas de adversarios — basada en observaciones del mundo real.

### 3 Matrices

| Matriz | Ámbito |
|---|---|
| **Enterprise** | Sistemas corporativos — 14 tácticas |
| **Mobile** | Dispositivos móviles |
| **PRE-ATT&CK** | Fases previas al ataque (recon, weaponization) |

> ATT&CK Enterprise cubre las **fases tardías** del Kill Chain (exploit, control, maintain, execute) con mayor granularidad

### 14 Tácticas Enterprise (en orden lógico del ataque)

1. Reconnaissance
2. Resource Development
3. Initial Access
4. Execution
5. Persistence
6. Privilege Escalation
7. Defense Evasion
8. Credential Access
9. Discovery
10. Lateral Movement
11. Collection
12. Command and Control
13. Exfiltration
14. Impact

### Usos clave

- Priorizar desarrollo de capacidades defensivas
- Medir **cobertura** de controles de seguridad existentes
- Describir una intrusión completa con referencia común
- Identificar patrones comunes entre actores (*tradecraft*)
- Conectar mitigaciones ↔ debilidades ↔ adversarios

---

## Diamond Model of Intrusion Analysis

**Elemento central:** el **Diamond Event** — unidad atómica de cualquier intrusión.

```
          Adversary
             /\
            /  \
           /    \
   Infrastructure ---- Capability
           \    /
            \  /
             \/
           Victim
```

### 4 Features del Diamond Event

| Feature | Qué es |
|---|---|
| **Adversary** | El atacante — explota una capability contra la víctima (insider, competidor, APT) |
| **Victim** | El target — persona, organización, IP, dominio, email |
| **Capability** | Las herramientas/métodos usados — malware, brute force, ransomware |
| **Infrastructure** | Hardware/software usado para llegar a la víctima — *el puente adversario→víctima* |

### Meta-features del evento

| Meta-feature | Descripción |
|---|---|
| **Timestamp** | Fecha/hora — inicio, fin y periodicidad del evento |
| **Phase** | Fase del Kill Chain en que ocurre (recon, delivery, exploitation…) |
| **Result** | Éxito / Fallo / Desconocido — o en términos CIA: C/I/A comprometida |
| **Direction** | Ruta: victim→infra · adversary→infra · infra→infra · bidireccional |
| **Methodology** | Clase de técnica: spear-phishing, DDoS, drive-by-compromise… |
| **Resource** | Recursos externos: hardware, software, conocimiento, datos, acceso |

### Extended Diamond Model

| Meta-feature adicional | Relación que describe |
|---|---|
| **Socio-political** | Adversary ↔ Victim — motivación: lucro, espionaje, hacktivismo |
| **Technology** | Infrastructure ↔ Capability — cómo la tecnología habilita operación y comunicación |

---

## Comparativa de los 4 Frameworks

| | CEH Framework | Cyber Kill Chain | MITRE ATT&CK | Diamond Model |
|---|---|---|---|---|
| **Foco** | Proceso del ethical hacker | Flujo lineal del ataque | Técnicas reales granulares | Análisis e investigación de intrusión |
| **Fases/Elementos** | 5 fases | 7 fases | 14 tácticas | 4 features + meta-features |
| **Autor** | EC-Council | Lockheed Martin | MITRE | Analistas expertos |
| **Uso principal** | Guía metodológica pentest | Detección y prevención | Cobertura defensiva y profiling | Atribución y correlación de eventos |

### Equivalencia CEH ↔ Kill Chain

| CEH (5 fases) | Kill Chain (7 fases) |
|---|---|
| Reconnaissance | Reconnaissance |
| Vulnerability Scanning | Weaponization |
| Gaining Access | Delivery + Exploitation |
| Maintaining Access | Installation + C2 |
| Clearing Tracks | — (implícito en C2/Installation) |

### Cómo se usan conjuntamente

```
Kill Chain  →  "esto es fase Delivery"
    ↓
MITRE ATT&CK  →  "técnica T1566 - Spear Phishing Attachment"
    ↓
Diamond Model  →  "adversary X, infrastructure Y, capability Z, victim W"
```
