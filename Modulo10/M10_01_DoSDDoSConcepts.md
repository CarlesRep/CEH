# M10_01_DoSDDoSConcepts.md
## CEH v13 — Módulo 10 | DoS/DDoS Concepts

---

## 1. Conceptos y definiciones

### DoS (Denial of Service)

Un ataque DoS reduce, restringe o impide el acceso a recursos del sistema para usuarios legítimos. El atacante satura los recursos de la víctima con solicitudes o tráfico ilegítimo hasta provocar la indisponibilidad total o degradación severa del servicio.

**Objetivo**: impedir que los usuarios legítimos usen el sistema. **No** es robo de datos ni acceso no autorizado.

**Formas de DoS**:
- Saturar el sistema con más tráfico del que puede gestionar.
- Saturar un servicio (p. ej., IRC) con más eventos de los que puede procesar.
- Crashear el stack TCP/IP enviando paquetes corruptos.
- Crashear un servicio con interacciones inesperadas.
- Colgar el sistema induciéndole un bucle infinito.

**Recursos afectados**: ancho de banda · espacio en disco · tiempo de CPU · estructuras de datos del SO · componentes físicos de red · ficheros y programas.

**Dos vectores principales**:
- **Bandwidth attacks**: desbordan la red con alto volumen de tráfico, agotando los recursos de red disponibles.
- **Connectivity attacks**: desbordan el sistema con un número masivo de solicitudes de conexión, consumiendo todos los recursos del SO disponibles para gestión de conexiones.

**Impacto**: pérdida de reputación (goodwill) · cortes de red · pérdidas económicas · disrupciones operativas · en el peor caso, destrucción accidental de ficheros de usuarios conectados en el momento del ataque.

---

### DDoS (Distributed Denial of Service)

Un ataque DDoS es un DoS a gran escala y coordinado, lanzado **indirectamente** a través de múltiples equipos comprometidos (botnets) distribuidos en Internet.

**Definición WWW Security FAQ**: utiliza múltiples equipos para lanzar un DoS coordinado contra uno o más objetivos, multiplicando la efectividad mediante recursos de equipos cómplices involuntarios que actúan como plataformas de ataque.

**Terminología clave**:

| Término | Definición |
|---|---|
| **Primary victim** | El sistema/servicio objetivo real del ataque |
| **Secondary victims** | Los sistemas comprometidos usados para lanzar el ataque (zombies/bots) |
| **Botnet** | Red de equipos comprometidos controlados por el atacante |
| **Zombie agent** | Equipo comprometido mediante malware que ejecuta las órdenes del atacante |
| **Handler** | Sistema intermedio controlado por el atacante que gestiona los zombies |
| **C&C server (Command and Control)** | Infraestructura desde la que el atacante envía comandos a los zombie agents |
| **Reflector systems** | Sistemas legítimos que, sin saberlo, amplifican el ataque al responder a solicitudes con IP de la víctima falsificada |

**Por qué los DDoS son populares**: fácil accesibilidad de exploit kits y escaso esfuerzo técnico requerido para ejecutarlos.

---

### 🔴 Cómo funciona un DDoS — Flujo completo

El diagrama (Figure 10.2) muestra la arquitectura en tres niveles:

```
Attacker
  └─► Handler (×N)          ← Paso 1: el atacante configura handlers
        └─► Zombies (×N)    ← Paso 2: el handler infecta grandes cantidades de PCs
              └─► Target    ← Paso 3: los zombies atacan el servidor objetivo
```

**Flujo detallado con reflectores**:
1. El atacante envía un comando a los **zombie agents** a través del servidor **C&C**.
2. Los zombie agents envían solicitudes de conexión a un gran número de **reflector systems**, con la **IP de la víctima falsificada (spoofed)** como IP de origen.
3. Los reflectores creen que las solicitudes vienen de la víctima y le envían las respuestas directamente.
4. La víctima recibe una avalancha de respuestas no solicitadas de múltiples reflectores simultáneamente.
5. Resultado: degradación severa del rendimiento o shutdown completo del sistema víctima.

**Por qué es difícil de rastrear**: el uso de secondary victims (zombies) hace que el tráfico no provenga directamente del atacante, dificultando la atribución. La IP falsificada añade una capa adicional de ofuscación.

**Objetivo previo al lanzamiento**: obtener acceso administrativo en el mayor número de sistemas posible. El atacante usa scripts de ataque personalizados para identificar sistemas vulnerables, sube el software DDoS y lo ejecuta en el momento elegido.

---

### DoS vs. DDoS — Diferencias clave

| | DoS | DDoS |
|---|---|---|
| **Origen del tráfico** | Un único sistema atacante | Múltiples sistemas comprometidos distribuidos |
| **Escala** | Limitada por los recursos del atacante | Amplificada por botnets; puede consumir los hosts más grandes de Internet |
| **Dificultad de mitigación** | Relativamente fácil (bloquear IP origen) | Muy difícil (tráfico desde miles de IPs distribuidas) |
| **Rastreo** | Posible | Muy difícil por uso de secondary victims y IP spoofing |
| **Intermediarios** | Ninguno | Handlers + Zombies + eventualmente Reflectores |

---

## 2. Exam Traps ⚠️

⚠️ **[DoS — objetivo: disponibilidad, no confidencialidad]** El examen puede presentar opciones que incluyan "robo de datos" o "acceso no autorizado". El objetivo de DoS es exclusivamente la **disponibilidad**. No compromete confidencialidad ni integridad de datos directamente.

⚠️ **[Primary victim vs. Secondary victim]** El examen usa estos términos técnicos directamente. Primary victim = el objetivo real (el servidor atacado). Secondary victims = los zombies/botnets usados para atacar. No confundir.

⚠️ **[Reflector systems — no son atacantes voluntarios]** Los reflectores son sistemas legítimos que responden a solicitudes que aparentemente provienen de la víctima (por IP spoofing). No están comprometidos; simplemente son redirigidos sin saberlo.

⚠️ **[Bandwidth attack vs. Connectivity attack]** Bandwidth attack agota el **ancho de banda de red**. Connectivity attack agota los **recursos del SO** mediante exceso de solicitudes de conexión. El examen los presenta como opciones separadas.

⚠️ **[Handler vs. C&C vs. Zombie]** Son roles distintos en la arquitectura DDoS. Handler: sistema intermedio que gestiona zombies. C&C: infraestructura de mando. Zombie: equipo comprometido que ejecuta el ataque. El examen puede preguntar el rol de cada uno.

⚠️ **[DoS puede destruir datos accidentalmente]** El examen puede afirmar que DoS no daña datos. Incorrecto: en el peor caso puede causar destrucción accidental de ficheros de usuarios conectados en el momento del ataque.

⚠️ **[DDoS — acceso administrativo previo]** El objetivo inicial del atacante antes de lanzar el DDoS es obtener **acceso administrativo** en el mayor número de sistemas posible para instalar el software DDoS.

---

## 3. Nemotécnicos

### Formas de DoS — "F-F-C-C-H"
**"Floods Fire Crashes Crashing Hangs"**
- **F**lood de tráfico
- **F**lood de eventos de servicio (IRC)
- **C**rash de TCP/IP stack (paquetes corruptos)
- **C**rash de servicio (interacción inesperada)
- **H**ang del sistema (bucle infinito)

### Arquitectura DDoS — "A-H-Z-T" (3 pasos del diagrama)
**"Attackers Handle Zombie Targets"**
- **A**ttacker configura Handlers (paso 1)
- **H**andlers infectan Zombies (paso 2)
- **Z**ombies atacan el **T**arget (paso 3)

### Flujo de reflectores — "C-S-R-V"
**"Command → Spoof → Reflect → Victim"**
- Atacante envía **C**omando al C&C
- Zombies envían solicitudes con IP **S**poofed de la víctima a reflectores
- **R**eflectores responden a la víctima
- **V**íctima recibe avalancha de respuestas no solicitadas

### Recursos afectados por DoS — "BDCD"
- **B**andwidth · **D**isk space · **C**PU time · **D**ata structures

---

## 4. Flashcards

**Q:** ¿Cuál es el objetivo principal de un ataque DoS?
**A:** Impedir que los usuarios legítimos accedan al sistema o servicio. No es robo de datos ni acceso no autorizado.

**Q:** ¿Cuál es la diferencia entre un ataque DoS y un DDoS?
**A:** DoS proviene de un único sistema. DDoS es coordinado y lanzado desde múltiples sistemas comprometidos (botnets) distribuidos en Internet, multiplicando la efectividad y dificultando la atribución.

**Q:** ¿Qué son los "secondary victims" en un ataque DDoS?
**A:** Los sistemas comprometidos (zombies/bots) usados para lanzar el ataque contra el objetivo real (primary victim).

**Q:** ¿Qué es un reflector system en el contexto de DDoS?
**A:** Un sistema legítimo que, sin estar comprometido, amplifica el ataque al responder a solicitudes cuya IP de origen ha sido falsificada (spoofed) con la IP de la víctima, enviándole respuestas no solicitadas.

**Q:** ¿Cuáles son los dos vectores principales de un ataque DoS según el CEH?
**A:** Bandwidth attacks (saturan el ancho de banda de red) y Connectivity attacks (saturan los recursos del SO con solicitudes de conexión).

**Q:** ¿Cuáles son los tres pasos del esquema de un ataque DDoS (Figure 10.2)?
**A:** 1) El atacante configura handlers. 2) Los handlers infectan grandes cantidades de PCs (zombies). 3) Los zombies reciben la instrucción de atacar el servidor objetivo.

**Q:** ¿Qué debe hacer el atacante antes de lanzar un ataque DDoS?
**A:** Obtener acceso administrativo en el mayor número de sistemas posible, subir el software DDoS y configurarlo para ejecutarse en el momento elegido.

**Q:** ¿Por qué es difícil rastrear al atacante en un DDoS?
**A:** Porque el tráfico no proviene directamente del atacante sino de secondary victims (zombies) con IPs distribuidas, y el IP spoofing añade una capa adicional de ofuscación.

**Q:** ¿Puede un ataque DoS destruir datos? ¿Por qué?
**A:** Sí, en el peor caso puede causar destrucción accidental de ficheros y programas de usuarios que estaban conectados al sistema víctima en el momento del ataque.

**Q:** ¿Qué rol cumple el servidor C&C (Command and Control) en un ataque DDoS?
**A:** Es la infraestructura desde la que el atacante envía comandos a los zombie agents para iniciar y coordinar el ataque.

**Q:** ¿Cuál es el impacto de un ataque DoS/DDoS? Cita al menos 4 consecuencias.
**A:** Pérdida de reputación (goodwill) · cortes de red · pérdidas económicas · disrupciones operativas · en el peor caso, destrucción accidental de datos de usuarios conectados.

**Q:** ¿Por qué los ataques DDoS se han vuelto populares según el CEH?
**A:** Por la fácil accesibilidad de exploit kits y el escaso esfuerzo técnico (brainwork) requerido para ejecutarlos.

---

## 5. Confusión frecuente

### DoS vs. DDoS
- **DoS**: un único sistema atacante. Más fácil de mitigar (bloquear la IP origen).
- **DDoS**: miles de sistemas distribuidos; IP spoofing; uso de intermediarios (handlers, zombies, reflectores). Muy difícil de mitigar y atribuir.
- **Criterio**: ¿el ataque viene de un único origen? → DoS. ¿Viene coordinado desde múltiples sistemas? → DDoS.

### Bandwidth Attack vs. Connectivity Attack
- **Bandwidth**: el cuello de botella es la **red** (ancho de banda agotado por volumen de tráfico).
- **Connectivity**: el cuello de botella es el **SO** (recursos de gestión de conexiones agotados por número de solicitudes).
- **Criterio**: ¿el recurso agotado es la capacidad de red? → Bandwidth. ¿Es la capacidad del SO para gestionar conexiones? → Connectivity.

### Handler vs. Zombie vs. Reflector
- **Handler**: sistema intermedio controlado por el atacante que coordina a los zombies.
- **Zombie**: equipo comprometido con malware que ejecuta el ataque directamente.
- **Reflector**: sistema legítimo no comprometido que responde a solicitudes con IP spoofed, amplificando inadvertidamente el ataque.
- **Criterio**: ¿está comprometido y recibe órdenes del atacante? → Zombie. ¿Coordina zombies? → Handler. ¿No está comprometido pero amplifica el tráfico? → Reflector.
