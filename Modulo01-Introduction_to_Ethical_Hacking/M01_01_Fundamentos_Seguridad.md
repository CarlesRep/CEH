# Módulo 1 — Fundamentos de Seguridad de la Información

## Pilares de la Seguridad (CIA+)

| Pilar | Descripción |
|---|---|
| **Confidencialidad** | Solo los autorizados acceden a la información |
| **Integridad** | La información no ha sido alterada sin autorización |
| **Disponibilidad** | La información es accesible cuando se necesita |
| **Autenticidad** | La información es genuina y proviene de la fuente declarada |
| **No repudio** | El emisor no puede negar haber enviado la información |

---

## Anatomía de un Ataque

**Ataque = Motivo + Método + Vulnerabilidad**

| Componente | Qué es |
|---|---|
| **Motivo** | El *por qué* — objetivo del atacante (robo de datos, daño reputacional, beneficio económico) |
| **Método (TTP)** | El *cómo* — Tactics, Techniques & Procedures |
| **Vulnerabilidad** | El *dónde* — debilidad explotada para ejecutar el método |

---

## TTPs — Diferencia crítica

| Término | Nivel | Descripción | Ejemplo |
|---|---|---|---|
| **Tactic** | Alto | *Qué objetivo táctico* persigue en esa fase | Acceso inicial, persistencia, exfiltración |
| **Technique** | Medio | *Mecanismo concreto* para lograr la táctica | Phishing, pass-the-hash, DLL injection |
| **Procedure** | Bajo | *Implementación exacta* — pasos, herramientas, comandos | Secuencia específica del malware al ejecutarse |

> **Regla:** Tactic = *qué fase* / Technique = *cómo* / Procedure = *exactamente con qué y cómo paso a paso*

| | Táctica | Técnica | Procedimiento |
|---|---|---|---|
| **Nivel** | Alto | Medio | Bajo |
| **Pregunta** | ¿Qué objetivo tiene esta fase? | ¿Qué herramientas/métodos usa? | ¿Qué pasos exactos sigue? |
| **Estabilidad en APTs** | Bastante estable | Similar entre actores | Varía según actor y objetivo |
| **Uso defensivo** | Perfilar al actor | Atribuir el ataque | Forense post-incidente |

---

## Tipos de Ataque

| Tipo | Descripción |
|---|---|
| **Pasivo** | Solo observa/intercepta — no altera el sistema. Difícil de detectar (ej. sniffing, eavesdropping) |
| **Activo** | Modifica, interrumpe o destruye — deja rastro (ej. MitM, DoS, ransomware) |
| **Close-in** | El atacante está físicamente cerca del objetivo (ej. shoulder surfing, dumpster diving) |
| **Insider** | Alguien con acceso legítimo al sistema — empleado, contratista (ej. sabotaje, fuga interna) |
| **Distribution** | Ataque inyectado en la cadena de suministro — hardware/software comprometido *antes* de llegar al usuario |

> **Pasivo vs Activo:** pasivo = *leer sin tocar* / activo = *tocar y modificar*

---

## Information Warfare (InfoWar) — Libicki

**Definición:** Uso de ICT para obtener ventaja competitiva sobre un adversario.

| Categoría | Clave |
|---|---|
| **C2 Warfare** | Control sobre sistemas/redes comprometidos |
| **Intelligence-based** | Superioridad de conocimiento — sensores que corrompen sistemas tecnológicos del adversario |
| **Electronic Warfare** | Degrada comunicaciones — *radio-electrónico* (medio físico) + *criptográfico* (bits/bytes) |
| **Psychological Warfare** | Propaganda y terror para desmoralizar al adversario |
| **Hacker Warfare** | Shutdown, robo, errores de datos, acceso no autorizado — virus, logic bombs, trojans, sniffers |
| **Economic Warfare** | Bloquea flujo de información para dañar economía de empresa o nación |
| **Cyberwarfare** | El más amplio — terrorismo informático + *semantic attacks* + *simula-warfare* |

> **Semantic attack:** toma el control del sistema manteniéndolo aparentemente operativo — la víctima no lo detecta
> **Simula-warfare:** guerra simulada — adquisición de armas para demostración sin uso real

| Estrategia | Descripción |
|---|---|
| **Defensiva** | Proteger activos ICT propios |
| **Ofensiva** | Atacar activos ICT del adversario |

---

## Tipos de Hackers

| Tipo | Perfil clave |
|---|---|
| **Script Kiddie** | Sin habilidades propias — usa tools ajenas, busca cantidad no calidad, sin objetivo concreto |
| **White Hat** | Hacking defensivo/ético — tiene **permiso** del propietario |
| **Black Hat** | Hacking ilegal/malicioso — también llamado *cracker* |
| **Gray Hat** | Opera en ambos lados — reporta vulnerabilidades pero también ayuda a explotarlas |
| **Blue Hat** | Contratado puntualmente para auditar software/sistemas antes de producción |
| **Red Hat** | Agresivo contra black hats — neutraliza amenazas activamente, **sin seguir reglas éticas** |
| **Green Hat** | Novato motivado por aprender — orientado a contribuir, no a dañar |
| **Hacktivist** | Hacking como protesta política/social — defacea webs, filtra datos — targets: gobiernos y corporaciones |
| **State-Sponsored** | Empleado por un gobierno — espionaje y sabotaje a infraestructuras de otros estados |
| **Cyber Terrorist** | Motivación religiosa/política — busca miedo mediante disrupción masiva |
| **Corporate Spy** | Espionaje industrial — roba IP, blueprints, trade secrets — usa **APTs**, puede estar años sin ser detectado |
| **Suicide Hacker** | Ataca infraestructura crítica por una causa — **no le importan las consecuencias legales** |
| **Insider** | Empleado con acceso legítimo que abusa de él — origen: descontento, despedido, mal formado |
| **Hacker Team** | Consorcio organizado con recursos propios — investigan, desarrollan tools y ejecutan ataques planificados. Clasificados como **threat actors** |
| **Criminal Syndicate** | Crimen organizado cibernético — objetivo: dinero y lavado — difíciles de localizar por jurisdicciones |
| **Organized Hackers** | Estructura jerárquica criminal — usan botnets/crimeware ajenos, roban y venden información |

### Distinciones críticas

| Par | Diferencia |
|---|---|
| **White vs Blue** | White = interno/permanente con permiso general · Blue = contratado externo puntual |
| **Red vs White** | Ambos pro-defensa, pero Red **no sigue ética** — puede atacar ilegalmente a black hats |
| **Corporate Spy vs State-Sponsored** | Spy = motivación económica/empresarial · State = motivación geopolítica/militar |
| **Criminal Syndicate vs Organized Hackers** | Syndicate = crimen organizado amplio (también físico) · Organized = puramente cibercriminal jerárquico |
| **Hacker Team** | Clasificado como threat actor — no necesariamente ético pese a investigar vulnerabilidades |
