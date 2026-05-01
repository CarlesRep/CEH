# M01 — Risk, CTI, Threat Modeling e Incident Management

## Risk — Fórmulas (memorizar las 3)

```
RISK = Threats × Vulnerabilities × Impact
RISK = Threat × Vulnerability × Asset Value
Level of Risk = Consequence × Likelihood
```

| Nivel | Consecuencia |
|---|---|
| **Extreme/High** | Peligro grave o inminente |
| **Medium** | Peligro moderado |
| **Low** | Peligro menor |

> ⚠️ Los controles **reducen** el riesgo — **no siempre lo eliminan**
> **Risk Matrix** = gráfico probabilidad × impacto — cada organización crea la suya

### Risk Management — 4 Fases

> 🧠 **Mnemotécnia:** **I**dentificamos **A**menazas, **T**ratamos **T**odo
> → **I**dentification · **A**ssessment · **T**reatment · **T**racking & Review

| Fase | Qué hace | Dato clave |
|---|---|---|
| **1. Risk Identification** | Identifica fuentes, causas y consecuencias | **Objetivo principal** del risk management |
| **2. Risk Assessment** | Evalúa probabilidad e impacto — prioriza | Proceso iterativo continuo |
| **3. Risk Treatment** | Selecciona e implementa controles | Requiere: método, responsables, costes, beneficios |
| **4. Risk Tracking & Review** | Verifica eficacia de controles | Identifica mejoras |

---

## Cyber Threat Intelligence (CTI)

**Objetivo:** Convertir amenazas *desconocidas* en *conocidas* → postura proactiva

### 4 Tipos de CTI

> 🧠 **Regla:** Strategic=*boardroom* · Tactical=*TTPs* · Operational=*ops concretas* · Technical=*IoCs*

| Tipo | Nivel | Consumidor | Foco | Característica única |
|---|---|---|---|---|
| **Strategic** | Alto | CISO, directivos | Tendencias, geopolítica, largo plazo | Fuentes: OSINT, CTI vendors, ISAOs/ISACs |
| **Tactical** | Medio | SOC managers, NOC, admins | TTPs del atacante | Actualiza productos de seguridad y parchea |
| **Operational** | Medio-bajo | IR managers, forenses | Amenazas *contra esta org* | A menudo solo accesible para gobiernos |
| **Technical** | Bajo | SOC staff, IR teams | IoCs: IPs, hashes, dominios | **Vida útil corta** — directo a IDS/IPS/FW |

### ⚠️ Pares que confunden en el examen

| Par | Diferencia |
|---|---|
| **Tactical vs Technical** | Tactical = *cómo ataca* (TTPs) · Technical = *con qué exactamente* (IoCs) |
| **Operational vs Tactical** | Operational = amenaza *contra esta org* · Tactical = metodología general |
| **Strategic vs Operational** | Strategic = largo plazo/negocio · Operational = incidente próximo concreto |

---

## Threat Intelligence Lifecycle — 5 Fases

> 🧠 **Mnemotécnia:** **P**lanificamos **C**olectando **P**rocesamos **A**nalizamos **D**iseminamos
> → **P**lanning · **C**ollection · **P**rocessing · **A**nalysis · **D**issemination

```
Planning → Collection → Processing → Analysis → Dissemination → (feedback → Planning)
```

| Fase | Clave |
|---|---|
| **1. Planning & Direction** | Define *qué* se necesita — base de todo el ciclo |
| **2. Collection** | HUMINT, IMINT, MASINT, SIGINT, OSINT, IoCs, third parties |
| **3. Processing & Exploitation** | Datos brutos → formato usable (parsing, filtrado, correlación) |
| **4. Analysis & Production** | Formato usable → inteligencia accionable (deducción, inducción, abducción) |
| **5. Dissemination & Integration** | Distribuye al consumidor adecuado → genera **feedback** → reinicia ciclo |

> ⚠️ **Processing vs Analysis:** Processing = estructurar datos · Analysis = convertir en inteligencia

---

## Threat Modeling — 5 Pasos

> 🧠 **Mnemotécnia:** **S**iempre **O**bservamos **D**escomposición **T**omando **V**ulnerabilidades
> → **S**ecurity Objectives · **O**verview · **D**ecompose · **T**hreats · **V**ulnerabilities

| Paso | Qué hace | Clave |
|---|---|---|
| **1. Identify Security Objectives** | Metas CIA de la app | ¿Qué proteger? ¿Compliance? ¿QoS? |
| **2. Application Overview** | Componentes, flujos, trust boundaries | Roles, tecnologías, mecanismos de seguridad |
| **3. Decompose the Application** | Trust boundaries, data flows, entry/exit points | Facilita hallar amenazas específicas |
| **4. Identify Threats** | Amenazas relevantes al contexto | Equipos de dev + testing |
| **5. Identify Vulnerabilities** | Debilidades asociadas a amenazas | Corregir *antes* de que sean explotadas |

> ⚠️ Si un paso bloquea → saltar al **paso 4**

**Paso 3 — Decompose en detalle:**

| Elemento | Una línea |
|---|---|
| **Trust Boundaries** | Dónde cambia el nivel de confianza |
| **Data Flows** | Entrada→salida — especial atención en cruces de trust boundaries |
| **Entry Points** | Donde usuarios/atacantes interactúan — foco en acceso crítico |
| **Exit Points** | Donde la app transfiere datos — priorizar salidas con datos no confiables |

---

## Incident Management & IH&R

### Jerarquía

```
Incident Management
    └── Incident Handling
            └── Incident Response
```

### IH&R — 9 Pasos

> 🧠 **Mnemotécnia:** **P**ronto **R**egistramos **T**riage, **N**otificamos · **C**ontenemos, **E**videnciamos, **E**rradicamos, **R**ecuperamos · **P**ublicamos
> → **P**reparation · **R**ecording · **T**riage · **N**otification · **C**ontainment · **E**vidence · **E**radication · **R**ecovery · **P**ost-incident

| # | Paso | Qué hace |
|---|---|---|
| 1 | **Preparation** | Auditoría de activos, políticas, equipo IR, herramientas |
| 2 | **Incident Recording & Assignment** | Registro inicial — comunicación y ticketing |
| 3 | **Incident Triage** | Analiza, valida, categoriza y **prioriza** — tipo, severidad, impacto |
| 4 | **Notification** | Informa a stakeholders: management, vendors, clientes |
| 5 | **Containment** | Evita propagación — limita el daño |
| 6 | **Evidence Gathering & Forensic Analysis** | Recopila evidencias — forense completo |
| 7 | **Eradication** | Elimina **causa raíz** y cierra vectores de ataque |
| 8 | **Recovery** | Restaura sistemas, servicios y datos |
| 9 | **Post-Incident Activities** | Documentación · impact assessment · políticas · cierre · **disclosure** |

### ⚠️ Pares que confunden en el examen

| Par | Diferencia |
|---|---|
| **Containment vs Eradication** | Containment = *detener propagación* · Eradication = *eliminar causa raíz* |
| **Triage vs Notification** | Triage = análisis interno · Notification = comunicar hacia fuera |
| **Recovery vs Post-Incident** | Recovery = restaurar · Post-Incident = revisar, documentar, cerrar |

### 📝 Pregunta típica CEH

> *"¿Qué paso del IH&R incluye el disclosure del incidente?"*
> → **Post-Incident Activities (paso 9)**

> *"Un atacante ha comprometido varios hosts. El equipo decide aislar los sistemas afectados. ¿En qué paso están?"*
> → **Containment (paso 5)** — no Eradication, porque no han eliminado la causa raíz
