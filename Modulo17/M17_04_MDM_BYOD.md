# M17_04 — Mobile Device Management (MDM) y BYOD
**Módulo 17 / Subapartado 4 — MDM & BYOD**

---

## 1. Conceptos y definiciones

### MDM — Definición y capacidades fundamentales

MDM (Mobile Device Management) proporciona plataformas para la distribución **over-the-air o por cable** de aplicaciones, datos y ajustes de configuración para todo tipo de dispositivos móviles. Gestiona tanto dispositivos corporativos como personales (BYOD) en la empresa.

**Funcionalidades básicas del software MDM:**
- Uso de passcode para el dispositivo
- Bloqueo remoto en caso de pérdida
- Borrado remoto (remote wipe) de datos en dispositivo perdido/robado
- Detección de dispositivos rooteados o con jailbreak
- Aplicación de políticas e inventario de activos
- Monitorización y reporting en tiempo real

**Objetivo:** reducir costes de soporte, discontinuidad del negocio y riesgos de seguridad mediante la gestión centralizada de apps en todos los dispositivos móviles de la empresa.

**Soluciones MDM destacadas:** Scalefusion MDM, ManageEngine Mobile Device Manager Plus, Microsoft Intune, SOTI MobiControl, AppTec360, Jamf Pro.

---

### BYOD — Definición, beneficios y riesgos 🔴

**BYOD (Bring Your Own Device):** política que permite a los empleados utilizar sus dispositivos personales (portátiles, smartphones, tablets) en el lugar de trabajo para acceder a recursos de la organización según sus privilegios de acceso.

#### Beneficios BYOD (4 principales)

| Beneficio | Lógica |
|-----------|--------|
| Mayor productividad | Los empleados dominan sus propios dispositivos; se actualizan a tecnologías más recientes sin coste para la empresa |
| Satisfacción del empleado | El empleado elige y costea su dispositivo; integra datos personales y corporativos en uno solo |
| Flexibilidad laboral | Un único dispositivo para necesidades personales y profesionales; trabajo desde cualquier lugar; menos restricciones que con dispositivos corporativos; sustituye el modelo cliente-servidor por una estrategia móvil y cloud-céntrica |
| Reducción de costes | La empresa no compra dispositivos; el coste del servicio de datos recae en el empleado |

#### Riesgos BYOD 🔴 (14 categorías)

| Riesgo | Descripción clave |
|--------|-------------------|
| Redes no seguras | Acceso a datos corporativos desde redes públicas no cifradas → data leakage |
| Data leakage y endpoint security | Dispositivos con conectividad cloud son endpoints inseguros; pérdida del dispositivo → exposición de datos corporativos |
| Eliminación inadecuada del dispositivo | Dispositivos desechados incorrectamente pueden contener datos sensibles (financieros, tarjetas de crédito, corporativos) |
| Soporte multi-dispositivo | Diferentes plataformas y OS → dificultad de gestión y control por el departamento IT |
| Mezcla de datos personales y corporativos | Riesgo de privacidad y seguridad; dificulta el borrado selectivo al finalizar el empleo |
| Dispositivos perdidos o robados | Tamaño reducido → alta probabilidad de pérdida; compromiso de datos corporativos almacenados |
| Falta de concienciación | Empleados no formados en BYOD → compromiso de datos corporativos |
| Bypass de políticas de red | Dispositivos BYOD en redes wireless pueden saltarse las políticas de red que solo aplican en LAN cableada |
| Problemas de infraestructura | Distintos OS, plataformas y programas con sus propias vulnerabilidades; complejidad para IT |
| Empleados descontentos | Uso indebido o filtración de datos corporativos a competidores |
| Jailbreaking/Rooting | Bypass de medidas de seguridad del fabricante → riesgos adicionales |
| Backup inadecuado | Dispositivos personales sin prácticas de backup apropiadas → riesgo de pérdida/corrupción |
| Software desactualizado / Gestión de parches | Falta de actualizaciones → vulnerabilidades sin parchear |
| Shadow IT y servicios cloud no autorizados | Uso de cloud storage/file sharing no autorizados → limita la supervisión IT de los datos corporativos |

---

### BYOD Policy Implementation — 5 principios 🔴

| Principio | Acciones clave |
|-----------|----------------|
| 1. Definir requisitos | Segmentar empleados por criticidad, sensibilidad temporal, acceso a datos/sistemas (ej. teletrabajo completo, part-time, day extender); realizar **Privacy Impact Assessment (PIA)** al inicio de cada proyecto BYOD con todos los equipos relevantes; el PIA lo realiza el **mobile governance committee** |
| 2. Seleccionar dispositivos y construir portfolio tecnológico | Decidir cómo gestionar usuarios y acceso a datos; además del MDM, considerar escritorios virtuales o software on-device; asegurar soporte de WLAN corporativa |
| 3. Desarrollar políticas | Involucrar a HR, legal, seguridad y privacidad; política debe cubrir: seguridad de la información, protección de datos, confidencialidad/propiedad, tracking/monitorización, terminación del empleo, evaluación de redes Wi-Fi, comportamiento aceptable/inaceptable |
| 4. Seguridad | Gestión de activos e identidad, controles de almacenamiento local, controles de medios extraíbles, niveles de acceso de red, controles de apps, seguridad web y mensajería, gestión de salud del dispositivo, DLP; evaluar: seguridad de información, seguridad de operaciones, seguridad de transmisión |
| 5. Soporte | Establecer capacidades de soporte desde el principio; el mobile committee debe reevaluar periódicamente los niveles de soporte |

---

### BYOD Security Guidelines — Administrador 🔴

Las siguientes medidas son las más probables en preguntas de examen por su especificidad:

- Proteger centros de datos con sistemas de protección **multi-capa**
- Usar canal cifrado para transferencia de datos
- **No permitir dispositivos con jailbreak o root**
- Aplicar políticas de autenticación de sesión y timeout en gateways de acceso
- Imponer acceso WLAN corporativo cuando se esté en las instalaciones
- Exigir passcodes complejos y rotación frecuente
- Registrar y autenticar el dispositivo antes de permitir acceso a la red corporativa
- Considerar **autenticación multifactor (MFA)** para acceso remoto a sistemas de información
- Requerir firma del acuerdo BYOD antes de acceder a los sistemas
- Al finalizar el empleo: decidir entre **borrado total del dispositivo o borrado selectivo** de apps/datos corporativos; mantener datos corporativos separados de los personales
- Cifrar todos los datos corporativos con algoritmos fuertes + canal cifrado
- En caso de pérdida/robo: reset o wipe remoto de contraseñas
- Implementar **VPN SSL** para acceso remoto seguro
- No proporcionar acceso offline a información sensible (solo accesible vía red corporativa)
- Habilitar mecanismo de **re-autenticación periódica**
- Monitorización en tiempo real con sistema **EMM (Enterprise Mobility Management)**
- Desarrollar **blacklist** de aplicaciones restringidas en dispositivos BYOD
- Backup de datos del dispositivo en servidores offsite o cloud
- Auditorías de seguridad y evaluaciones de vulnerabilidades periódicas
- Implementar **containerización o sandboxing** para separar datos corporativos de personales
- Habilitar remote wipe y lock
- Utilizar **application whitelisting/blacklisting** para controlar qué apps se instalan/ejecutan
- Imponer cifrado del dispositivo con herramientas como **BitLocker o FileVault**
- Crear estrategia de **offboarding** para garantizar eliminación de datos sensibles y restricción de acceso

---

### BYOD Security Guidelines — Empleado

- Usar mecanismos de cifrado para almacenar datos
- Mantener separación clara entre datos personales y corporativos
- Actualizar regularmente el dispositivo con el último OS y parches
- Usar soluciones antivirus y DLP
- Establecer passcode fuerte y cambiarlo frecuentemente
- No descargar ficheros de fuentes no confiables
- Borrar todos los datos, credenciales de acceso y apps corporativas antes de abandonar la organización
- Acudir solo a distribuidores y tiendas autorizadas para reparaciones
- No subir/hacer backup de datos corporativos en cloud storage personal no autorizado
- Reportar al equipo IT en caso de robo o pérdida
- Usar conexión VPN segura en redes Wi-Fi públicas
- **No sincronizar el dispositivo móvil con otros dispositivos personales** (TV, escritorio, Bluetooth)
- No hacer jailbreak ni root
- Revisar permisos solicitados por apps antes de instalarlas
- Implementar bloqueo automático o autenticación biométrica

---

### MDM vs EMM — Distinción

**MDM** = gestión básica de dispositivos (hardware + OS).
**EMM (Enterprise Mobility Management)** = alcance más amplio; incluye MDM + gestión de aplicaciones + gestión de contenido + seguridad. En el contexto del CEH, el término EMM aparece en el contexto de monitorización en tiempo real.

---

## 2. Exam Traps ⚠️ 🔴

⚠️ **[MDM — distribución over-the-air]** El examen puede preguntar cómo distribuye MDM las aplicaciones y configuraciones: **over-the-air o por cable**. No solo inalámbricamente. La capacidad OTA es la más destacada, pero el cable también es válido.

⚠️ **[MDM — detección de jailbreak/root]** Una funcionalidad básica de MDM es **detectar si el dispositivo está rooteado o con jailbreak**. Si el examen pregunta qué herramienta detecta jailbreak en entorno empresarial → MDM.

⚠️ **[BYOD — beneficio vs riesgo de "flexibilidad"]** La flexibilidad laboral y el trabajo desde cualquier lugar son **beneficios** BYOD. Sin embargo, esa misma conectividad desde cualquier lugar es también un **riesgo** (redes no seguras). El examen puede presentar el mismo concepto en ambos lados; el contexto determina si es beneficio o riesgo.

⚠️ **[PIA — cuándo se realiza]** El Privacy Impact Assessment se realiza **al inicio de cada proyecto BYOD**, no al finalizar ni durante la fase de soporte. Lo realiza el **mobile governance committee**.

⚠️ **[BYOD política — quién la desarrolla]** No solo IT. Debe incluir una delegación de recursos de la empresa: **HR, legal, seguridad y privacidad**. Si el examen pregunta qué departamentos participan en el desarrollo de la política BYOD → no solo IT.

⚠️ **[Borrado total vs selectivo al finalizar empleo]** La guía establece que al salir un empleado se debe decidir entre **borrado total del dispositivo o borrado selectivo** de apps/datos corporativos. El objetivo es mantener los datos corporativos separados de los personales precisamente para facilitar el borrado selectivo sin afectar los datos personales.

⚠️ **[VPN para BYOD — tipo específico]** La guía de administrador especifica implementar **VPN SSL** (no genéricamente "una VPN"). Dato concreto que puede aparecer en opciones de examen.

⚠️ **[EMM vs MDM]** La monitorización en tiempo real de dispositivos en entorno BYOD se realiza con sistema **EMM (Enterprise Mobility Management)**, no solo MDM. El examen puede preguntar qué sistema proporciona monitorización en tiempo real en el contexto BYOD.

⚠️ **[Shadow IT — riesgo específico]** El uso de cloud storage/file-sharing no autorizados en dispositivos personales crea un escenario de **shadow IT** que **limita la supervisión IT** de los datos corporativos. No es simplemente "data leakage"; es pérdida de visibilidad y control.

⚠️ **[Cifrado de dispositivos — herramientas mencionadas]** Las herramientas específicas mencionadas para cifrado en reposo en BYOD son **BitLocker** (Windows) y **FileVault** (macOS). Si el examen presenta estas herramientas en contexto BYOD, la respuesta es cifrado de datos en reposo.

---

## 3. Nemotécnicos

### MDM — 6 funcionalidades básicas
**"PRWDEP"**: **P**asscode → **R**emote lock → **W**ipe remoto → **D**etección jailbreak/root → **E**nforcement de políticas + inventario → **P**erformance monitoring (tiempo real)

### BYOD — 4 beneficios
**"PEFL"**: **P**roductividad → **E**mployee satisfaction → **F**lexibilidad laboral → **L**ower costs (costes reducidos)

### BYOD — Riesgos más preguntados (7 de 14)
**"RDSMJBS"**: **R**edes no seguras → **D**ata leakage → **S**oporte multi-dispositivo → **M**ezcla personal/corporativo → **J**ailbreak/root → **B**ypass de políticas de red → **S**hadow IT

### BYOD Policy — 5 principios
**"DeSDP S"**: **De**finir requisitos → **S**eleccionar dispositivos → **D**esarrollar políticas → **P**olicies security → **S**upport

Alternativa más mnemónica: "**D**os **S**oldados **D**efendiendo **P**equeñas **S**endas" (Definir → Seleccionar → Desarrollar → [Security] → Soporte)

### Herramientas de cifrado BYOD
- **B**itLocker = **B**ig Windows (Windows)
- **F**ileVault = **F**ruta (Apple/macOS)

---

## 4. Flashcards

**Q:** ¿Qué es MDM y cuál es su método de distribución principal?
**A:** Mobile Device Management; plataforma para distribución **over-the-air o por cable** de aplicaciones, datos y configuraciones para todo tipo de dispositivos móviles. Gestiona dispositivos corporativos y BYOD.

---

**Q:** ¿Cuáles son las 6 funcionalidades básicas del software MDM?
**A:** Passcode del dispositivo, bloqueo remoto, borrado remoto de datos, detección de jailbreak/root, aplicación de políticas e inventario, monitorización y reporting en tiempo real.

---

**Q:** ¿Qué es BYOD y qué modelo de negocio sustituye?
**A:** Política que permite a empleados usar sus dispositivos personales para acceder a recursos corporativos. Sustituye el modelo cliente-servidor tradicional por una estrategia **móvil y cloud-céntrica**.

---

**Q:** ¿Cuáles son los 4 beneficios principales de BYOD para una organización?
**A:** Mayor productividad, satisfacción del empleado, flexibilidad laboral (trabajo desde cualquier lugar), reducción de costes (la empresa no compra dispositivos).

---

**Q:** ¿Qué es el Privacy Impact Assessment (PIA) en el contexto BYOD y quién lo realiza?
**A:** Procedimiento organizado para documentar hechos, objetivos, riesgos de privacidad y mitigaciones a lo largo del ciclo de vida del proyecto. Se realiza **al inicio de cada proyecto BYOD** por el **mobile governance committee** (usuarios finales de cada segmento + gestión IT).

---

**Q:** ¿Qué departamentos deben participar en el desarrollo de la política BYOD?
**A:** No solo IT. Debe incluir: **HR, legal, seguridad y privacidad**, además de IT. Es una delegación de recursos de la empresa.

---

**Q:** ¿Qué tipo de VPN especifica la guía de seguridad BYOD para acceso remoto seguro?
**A:** **VPN SSL** (SSL-based VPN).

---

**Q:** ¿Cuál es la diferencia entre borrado total y borrado selectivo en el offboarding BYOD?
**A:** Borrado total = wipe completo del dispositivo. Borrado selectivo = eliminación solo de apps y datos corporativos, preservando los datos personales del empleado. Mantener datos separados facilita el borrado selectivo.

---

**Q:** ¿Qué sistema se usa para la monitorización en tiempo real de dispositivos en entornos BYOD?
**A:** **EMM (Enterprise Mobility Management)**, mencionado específicamente en las guías de seguridad para administradores BYOD. EMM tiene un alcance más amplio que MDM.

---

**Q:** ¿Qué es Shadow IT en el contexto BYOD y por qué es un riesgo?
**A:** Uso de servicios cloud de almacenamiento y compartición de ficheros no autorizados en dispositivos personales. Riesgo: **limita la supervisión IT y el control** sobre los datos corporativos, creando puntos ciegos en la gestión de la seguridad.

---

**Q:** ¿Qué riesgo BYOD implica que un empleado accede a datos corporativos desde una red pública?
**A:** **Sharing confidential data on unsecured networks** → las conexiones pueden no estar cifradas → riesgo de data leakage.

---

**Q:** ¿Qué herramientas específicas menciona el CEH para cifrado de datos en reposo en dispositivos BYOD?
**A:** **BitLocker** (Windows) y **FileVault** (macOS).

---

**Q:** ¿Qué riesgo BYOD describe la situación en que un empleado con resentimientos puede causar daño?
**A:** **Disgruntled employees** (empleados descontentos) — pueden hacer uso indebido de datos corporativos almacenados en sus dispositivos o filtrar información sensible a competidores.

---

**Q:** Según las guías BYOD para empleados, ¿qué debe hacer antes de abandonar la organización?
**A:** Borrar todos los datos corporativos, credenciales de acceso y aplicaciones relacionadas con la organización de todos los dispositivos, antes de irse en cualquier circunstancia.

---

**Q:** ¿Qué riesgo BYOD describe la incapacidad del departamento IT para controlar todos los dispositivos debido a la diversidad de plataformas?
**A:** **Infrastructure issues** (problemas de infraestructura) — distintos OS, plataformas y programas con sus propias vulnerabilidades; complejidad para IT en la gestión, backup y compatibilidad.

---

**Q:** ¿Qué medida de seguridad BYOD separa físicamente los datos corporativos de los personales en el dispositivo?
**A:** **Containerización o sandboxing** — aísla los datos corporativos de los personales, mejorando el control y la protección de información sensible y facilitando el borrado selectivo.

---

**Q:** ¿Cuál es la diferencia entre application whitelisting y blacklisting en BYOD?
**A:** Whitelisting = lista de apps **permitidas** (solo se pueden instalar/ejecutar las que están en la lista). Blacklisting = lista de apps **prohibidas** (se bloquean las de la lista; el resto están permitidas). Ambas son medidas de control de apps en dispositivos BYOD.

---

**Q:** ¿Por qué BYOD puede llevar a bypass de las políticas de red corporativas?
**A:** Los dispositivos BYOD conectados a redes **wireless** pueden saltarse las políticas de red que solo se aplican en **LAN cableada**, ya que las políticas pueden diferir entre ambos tipos de red.

---

**Q:** ¿Qué componentes debe cubrir obligatoriamente una política BYOD general?
**A:** Seguridad de la información, protección de datos, confidencialidad y propiedad, información sobre tracking/monitorización, consideraciones sobre terminación del empleo, guía para evaluar seguridad de redes Wi-Fi, comportamiento aceptable e inaceptable.

---

## 5. Confusión frecuente

### MDM vs EMM
- **MDM:** gestión de dispositivos móviles a nivel de hardware y OS; funcionalidades básicas (passcode, remote lock/wipe, detección jailbreak, políticas, monitorización).
- **EMM:** alcance más amplio; engloba MDM + gestión de aplicaciones + gestión de contenido + seguridad. La guía BYOD menciona EMM específicamente para **monitorización en tiempo real**.
- **Criterio:** ¿gestión básica de dispositivo? → MDM. ¿Monitorización en tiempo real en entorno BYOD? → EMM.

---

### Remote Wipe total vs Selective Wipe (borrado selectivo)
- **Remote Wipe total:** borra todo el contenido del dispositivo; se usa cuando el dispositivo es irrecuperable o cuando el empleado ha salido.
- **Selective Wipe:** borra solo datos y apps corporativas; preserva los datos personales del empleado. Posible solo si los datos corporativos y personales están **separados** (containerización).
- **Criterio:** ¿dispositivo perdido/robado sin posibilidad de recuperación? → Remote wipe total. ¿Empleado que sale de la organización? → Puede usarse borrado selectivo si los datos están separados.

---

### Blacklist vs Whitelist de aplicaciones
- **Blacklist:** prohíbe apps específicas; más permisiva (lo no listado está permitido).
- **Whitelist:** permite solo apps aprobadas; más restrictiva (lo no listado está bloqueado).
- **Criterio:** ¿control más estricto sobre qué apps pueden ejecutarse? → Whitelist. ¿Bloquear apps específicas problemáticas? → Blacklist.

---

### BYOD riesgo "Mezcla de datos" vs "Data leakage"
- **Mezcla de datos personales y corporativos:** riesgo de privacidad y dificultad para aplicar medidas de seguridad específicas al dato corporativo; dificulta el borrado selectivo.
- **Data leakage:** exposición activa de datos corporativos (por pérdida del dispositivo, redes no seguras, etc.).
- **Criterio:** ¿dificultad para separar/proteger/borrar datos? → Mezcla de datos. ¿Exposición/filtración activa? → Data leakage.

---

### PIA vs Auditoría de seguridad en BYOD
- **PIA (Privacy Impact Assessment):** se realiza **al inicio** de cada proyecto BYOD; documenta riesgos de privacidad y mitigaciones; lo realiza el mobile governance committee.
- **Auditoría de seguridad:** se realiza **periódicamente** durante la operación del programa BYOD; identifica y mitiga riesgos de seguridad técnicos.
- **Criterio:** ¿inicio del proyecto, privacidad? → PIA. ¿Operación continua, técnico? → Auditoría de seguridad.
