# M12_07_IDSFirewallEvasionCountermeasures

## 1. Conceptos y definiciones

### Contramedidas frente a evasión de IDS 🔴
El objetivo de un IDS no es solo detectar firmas conocidas, sino interpretar correctamente tráfico ambiguo, fragmentado, ofuscado o manipulado para que el atacante no consiga que el sensor vea una realidad distinta de la que ve el host final. La evasión de IDS suele explotar precisamente esa diferencia de interpretación: fragmentación IP, solapamiento de fragmentos, túneles, ofuscación, shellcode polimórfico, rutas no monitorizadas o reglas mal ajustadas.

Por eso, una defensa sólida contra evasión de IDS se apoya en cuatro pilares.

**Normalización de tráfico.** Un traffic normalizer elimina ambigüedad antes de que el tráfico llegue al IDS. Su misión es convertir variantes semánticamente equivalentes en una representación uniforme. Esto reduce la capacidad del atacante para esconder payloads en fragmentación anómala, campos inconsistentes o técnicas de manipulación del flujo.

**Reensamblado correcto.** Si el IDS no reensambla fragmentos y sesiones igual que el host destino, el atacante puede esconder el payload repartido entre fragmentos o usando solapamientos. Por eso el libro insiste en normalizar paquetes fragmentados, permitir el reensamblado en orden correcto y usar soluciones capaces de reensamblar sesiones fragmentadas.

**Correlación topológica.** Un IDS solo ve lo que atraviesa su punto de inspección. Si un paquete llega por una ruta no protegida por IDS, la confianza en ese tráfico debe caer y debe aplicarse análisis más profundo. Esta idea enlaza con el diseño del despliegue: colocar el IDS tras estudiar topología, naturaleza del tráfico y número de hosts.

**Detección de técnicas de ocultación.** No basta con firmas estáticas. El texto recomienda reglas para detectar túneles y ofuscación, análisis heurístico y de comportamiento para polimorfismo, y protección híbrida basada en firmas más analítica estadística y behavioral analysis para reducir la ceguera ante zero-days.

La referencia al **NOP opcode distinto de `0x90`** apunta a shellcode polimórfico: un IDS demasiado literal buscará sleds clásicos con `0x90`, mientras que un atacante puede emplear instrucciones equivalentes funcionalmente para romper firmas ingenuas.

La recomendación de usar **TCP FIN o RST** busca terminación activa de sesiones maliciosas. No elimina la intrusión ya ocurrida, pero corta la comunicación y limita explotación, exfiltración o movimiento lateral.

La mención a **definir DNS server para client resolver** y a **bloquear ICMP TTL expired** se relaciona con limitar canales auxiliares usados en evasión o fingerprinting, y reducir respuestas útiles para manipulación o reconocimiento.

### Contramedidas frente a evasión de firewall 🔴
Un firewall resiste la evasión cuando su política, visibilidad y gobierno operativo impiden que el tráfico malicioso encuentre excepciones, rutas no inspeccionadas o reglas demasiado amplias. El principio central es **default deny**: denegar todo y abrir únicamente lo estrictamente necesario. Todo lo demás son controles complementarios que endurecen ese modelo.

Las contramedidas del libro se agrupan en varias capas:

**Política restrictiva.** Deny all, especificar IPs origen/destino y puertos, deshabilitar FTP por defecto y revisar todo tráfico entrante y saliente permitido. Esto reduce superficie de exposición y evita reglas implícitas demasiado generales.

**Inspección profunda.** La evasión moderna se apoya en HTTPS/TLS, encapsulación y protocolos de capa de aplicación. Por eso aparecen **HTTPS/TLS inspection** y **deep packet inspection (DPI)**. Sin inspección en capa de aplicación, el firewall ve metadatos pero no el contenido real del intercambio.

**Control de abuso y anomalía.** Limitar conexiones por IP, restringir por geografía y crear reglas basadas en comportamiento de usuario elevan la dificultad para campañas automatizadas, bots distribuidos o uso anómalo de cuentas legítimas.

**Gobierno y trazabilidad.** Backups del ruleset, auditorías periódicas, control de acceso administrativo, registro de cambios y envío a syslog remoto protegen contra error humano, sabotaje interno y alteraciones no autorizadas del perímetro.

**Aislamiento operativo.** Ejecutar servicios del firewall con una cuenta dedicada en lugar de root/administrator limita el impacto de una explotación del propio firewall.

**Validación ofensiva controlada.** El libro menciona **HTTP Evader** para probar evasiones sospechosas. Conceptualmente, esto refleja una idea importante: un firewall no debe darse por seguro por configuración teórica; hay que probar cómo se comporta frente a tráfico malicioso realista.

**Zero trust.** La mención a un modelo zero-trust corrige un fallo clásico: asumir que el tráfico interno es fiable. Bajo zero trust, tanto tráfico interno como externo deben validarse, lo que dificulta evasiones desde hosts ya comprometidos dentro de la red.

### Contramedidas frente a evasión de endpoint security 🔴
La evasión de endpoint security se dirige contra AV, EDR, listas blancas, AMSI, memoria de procesos y telemetría local. La defensa no puede descansar en una sola firma o en un único agente; debe combinar prevención, restricción, observabilidad y hardening del endpoint.

**Prevención y reducción de superficie.** Mantener sistema operativo y aplicaciones actualizados, aplicar whitelisting de aplicaciones, validar binarios y scripts con code signing, restringir USB y aplicar DLP. El objetivo es reducir oportunidades de ejecución, persistencia y exfiltración.

**Contención.** La segmentación de red evita que una intrusión local se convierta en brote lateral. Least privilege y roles con privilegios específicos impiden que un payload obtenga permisos administrativos con facilidad.

**Visibilidad.** Logs detallados y correlación con SIEM permiten ver patrones que un endpoint aislado puede no interpretar bien. La monitorización en tiempo real ayuda a detectar anomalías de ejecución, comunicación o acceso a recursos.

**Resistencia técnica del sistema.** El libro destaca **ASLR, DEP y CFI**. Las tres son defensas de memoria, pero no hacen lo mismo:

| Control | Qué dificulta | Idea clave para examen |
|---|---|---|
| ASLR | Predicción de direcciones | Rompe payloads que dependen de direcciones estables |
| DEP | Ejecución en zonas de datos | Impide ejecutar código en memoria marcada como no ejecutable |
| CFI | Desviación ilegítima del flujo de control | Restringe saltos/calls a destinos válidos |

**Endurecimiento de acceso.** MFA para sistemas críticos y VPN para acceso remoto reducen la utilidad de credenciales robadas y la exposición de endpoints fuera del perímetro controlado.

**Engaño defensivo.** El uso de decoys o honeypots en endpoint security no bloquea por sí mismo, pero mejora detección temprana y desvío del atacante.

### Contramedidas frente a evasión de NAC 🔴
La evasión de NAC intenta introducir dispositivos no autorizados, aprovechar equipos ya autenticados o romper el modelo de segmentación y control de acceso. La defensa efectiva exige que NAC no sea solo una puerta inicial, sino un control dinámico, contextual y correlacionado con otras capas.

**Política y cumplimiento.** Deben existir políticas claras de autenticación, autorización y remediación para dispositivos no conformes. Sin esta capa, el NAC puede autenticar pero no gobernar correctamente excepciones o incumplimientos.

**Identidad del dispositivo.** Device profiling identifica y clasifica qué tipo de equipo intenta conectarse. Esto es importante porque un atacante puede reutilizar credenciales válidas desde un dispositivo distinto; el contexto del dispositivo ayuda a detectar anomalías.

**Aplicación dinámica.** El NAC debe ajustar políticas en tiempo real según amenazas detectadas o cambios de contexto. Un NAC estático falla frente a ataques que modifican comportamiento tras obtener acceso inicial.

**Segmentación y RBAC.** VLANs, separación de recursos sensibles y RBAC limitan el alcance de una evasión. Incluso si el atacante entra, no debería poder moverse libremente.

**Autenticación reforzada.** MFA, autenticación por certificados o biometría aumentan el coste de suplantación. Aquí el criterio de examen es claro: el control no debe depender solo de contraseña o de presencia física en la red.

**Correlación y threat hunting.** Integrar NAC con SIEM permite unir eventos NAC con firewall, IDS/IPS y endpoint. Además, el libro remarca caza proactiva de evasiones de NAC: no basta con esperar alertas.

**Zero trust aplicado a acceso.** Cada acceso debe verificarse continuamente. Esto choca directamente con el error clásico de “una vez dentro, confianza permanente”.

### Contramedidas frente a evasión de antivirus 🔴
La evasión de antivirus explota la debilidad de motores puramente basados en firmas mediante ofuscación, plantillas mutadas, scripts dinámicos, carga en memoria y técnicas que retrasan o fragmentan el comportamiento malicioso. La defensa consiste en ampliar el modelo de detección y reforzar el entorno alrededor del AV.

**Detección avanzada.** Behavioral analysis, heurística, machine learning y threat intelligence permiten detectar actividad sospechosa sin depender de una firma exacta previa. La idea central del examen es que **signature-less detection** complementa, no reemplaza totalmente, el motor clásico.

**Protección en tiempo real y EPP/EDR.** El texto diferencia el antivirus tradicional de una protección de endpoint más amplia. EPP añade capas preventivas; EDR añade visibilidad, telemetría y respuesta.

**Sandboxing.** Ejecutar archivos sospechosos en entornos aislados sirve para observar comportamiento. Su debilidad está en las evasiones temporales o dependientes de contexto; por eso se combina con otras capas.

**Control de scripts y documentos.** El libro destaca análisis en tiempo real de scripts y adjuntos maliciosos. Esto apunta directamente a ataques fileless, macros y payloads de scripting que pueden esquivar firmas binarias clásicas.

**Defense in depth.** Antivirus aislado no basta. Debe operar junto con IDS, IPS, firewall, SIEM, segmentación y mínimos privilegios.

### Tabla comparativa de contramedidas por dominio 🔴

| Dominio | Problema principal de evasión | Contramedida nuclear | Señales clave de examen |
|---|---|---|---|
| IDS | Ambigüedad de tráfico, fragmentación, ofuscación, polimorfismo | Normalización + reensamblado + heurística | Overlapping fragments, shellcode polimórfico, reglas Snort |
| Firewall | Reglas permisivas, cifrado, túneles, abuso de servicios permitidos | Default deny + DPI + TLS inspection | HTTPS inspection, limitación por IP, auditoría de reglas |
| Endpoint | EDR bypass, living-off-the-land, AMSI bypass, inyección, memoria | Hardening + monitorización + control de ejecución | ASLR/DEP/CFI, whitelisting, SIEM, FIM |
| NAC | Dispositivo no autorizado, uso de host autenticado, segmentación rota | Device profiling + RBAC + MFA + segmentación | VLAN, NAC dinámico, integración con SIEM |
| Antivirus | Ofuscación, polimorfismo, scripts, carga en memoria | Heurística + ML + sandbox + tiempo real | Signature-less detection, EPP/EDR, análisis de scripts |

## 2. Exam Traps ⚠️
⚠️ **[Traffic normalization vs packet filtering]** El examen puede presentar la normalización como si fuese simplemente “bloquear tráfico”. No es lo mismo. La normalización no se limita a negar; su función es **eliminar ambigüedad** del flujo antes de que el IDS lo analice.

⚠️ **[Fragment reassembly]** Puede insinuar que basta con que el host final reensamble los fragmentos. La respuesta correcta es que **el IDS también debe normalizar y reensamblar** de forma coherente con el host, o el atacante puede esconder el payload.

⚠️ **[NOP sled polymorphism]** El examen puede asociar NOP sled exclusivamente con `0x90`. La trampa es pensar que buscar `0x90` resuelve el problema. La respuesta correcta es que el shellcode polimórfico puede usar **otros opcodes funcionalmente equivalentes**.

⚠️ **[RST/FIN packets]** Puede vender FIN/RST como mecanismo preventivo general. No lo son. Sirven para **terminar sesiones maliciosas activas**, no sustituyen endurecimiento ni detección previa.

⚠️ **[Zero trust]** El examen puede presentarlo como contramedida exclusiva de firewall o NAC. La respuesta correcta es que **zero trust es un modelo transversal**, aplicable especialmente a firewall y NAC, pero con implicaciones en todo el acceso a recursos.

⚠️ **[Application whitelisting]** Puede aparecer tanto como defensa de endpoint como defensa de antivirus. La respuesta correcta es que es válida en ambos contextos, pero en endpoint security se orienta a **control de ejecución**, mientras que en anti-virus evasion aparece como medida para **reducir superficie de ejecución de malware**.

⚠️ **[EPP vs EDR]** Trampa clásica: tratarlos como sinónimos. **EPP** prioriza prevención; **EDR** prioriza telemetría, hunting y respuesta.

⚠️ **[SIEM]** Puede aparecer como si el SIEM detuviera directamente el ataque. No. **Centraliza, correlaciona y detecta**; no sustituye firewall, IDS, AV o NAC.

⚠️ **[FIM vs antivirus]** El examen puede mezclar File Integrity Monitoring con análisis antimalware. La respuesta correcta es que **FIM detecta cambios en ficheros críticos**, no clasifica malware como haría un AV/EDR.

⚠️ **[MAC filtering]** Puede plantearse como control fuerte de NAC. La respuesta correcta es que ayuda, pero **no es suficiente por sí solo**, porque las MAC pueden suplantarse; por eso el libro lo combina con MFA, profiling y autenticación avanzada.

⚠️ **[HTTPS/TLS inspection vs DPI]** Pueden aparecer como equivalentes. No lo son. **TLS inspection** descifra/inspecciona tráfico cifrado; **DPI** inspecciona contenido a nivel de aplicación. TLS inspection puede ser necesaria para que la DPI vea el contenido.

⚠️ **[Honeypots as countermeasure]** El examen puede sugerir que un honeypot bloquea la evasión. La respuesta correcta es que **detecta, engaña y aporta inteligencia**, pero no sustituye controles preventivos.

⚠️ **[Behavior-based vs signature-based]** Puede presentar ambos modelos como excluyentes. La respuesta correcta es que la defensa recomendada es **multicapa**, combinando firmas, heurística, comportamiento, ML y threat intelligence.

## 3. Nemotécnicos

### IDS: **NORA-HRDT** 🔴
Para recordar el núcleo defensivo de IDS:
- **N**ormalización
- **O**verlapping fragments controlados
- **R**eensamblado
- **A**nálisis heurístico
- **H**ybrid detection
- **R**ST/FIN
- **D**eploy según topología
- **T**unneling/obfuscation rules

Idea mental: si el atacante intenta **confundir** al IDS, tú primero **normalizas** y luego **reensamblas**.

### Firewall: **DDD-LOG-AUDIT** 🔴
- **D**efault deny
- **D**PI
- **D**ecrypt/inspect TLS
- **LOG** centralizado y protegido
- **AUDIT** periódico

Traducción rápida: un firewall se rompe menos cuando **niega por defecto, inspecciona de verdad y deja trazabilidad**.

### Endpoint memory protections: **A-D-C**
- **A**SLR = **A**leatoriza direcciones
- **D**EP = **D**atos no ejecutables
- **C**FI = **C**ontrola el flujo

### NAC: **P-D-A-S-C**
- **P**olicies claras
- **D**evice profiling
- **A**utenticación fuerte
- **S**egmentación
- **C**orrelación con SIEM

### Anti-virus moderno: **H-M-S-R**
- **H**eurística
- **M**achine learning
- **S**andbox
- **R**eal-time protection

### Regla de decisión rápida
Si la técnica ofensiva intenta:
- **confundir paquetes** → piensa en **IDS normalization/reassembly**
- **aprovechar tráfico permitido** → piensa en **firewall default deny + DPI/TLS inspection**
- **ejecutar binarios/scripts en host** → piensa en **endpoint/AV whitelisting + EDR + script analysis**
- **colar un dispositivo o moverse por la red** → piensa en **NAC + RBAC + segmentación**

## 4. Flashcards

**Q:** ¿Cuál es la contramedida más directa contra evasión de IDS basada en ambigüedad de paquetes?
**A:** Usar un traffic normalizer para eliminar ambigüedad del flujo antes de que llegue al IDS.

**Q:** ¿Qué debe hacer un IDS frente a paquetes fragmentados para evitar evasión?
**A:** Normalizarlos y reensamblarlos correctamente, incluyendo manejo de fragmentos solapados.

**Q:** ¿Qué problema intenta mitigar la búsqueda de NOPs distintos de `0x90`?
**A:** El shellcode polimórfico que evita firmas basadas solo en el NOP sled clásico `0x90`.

**Q:** ¿Para qué se usan TCP FIN o RST como contramedida?
**A:** Para terminar sesiones TCP maliciosas activas.

**Q:** ¿Qué debe evaluarse antes de desplegar un IDS según el libro?
**A:** Topología de red, naturaleza del tráfico y número de hosts a monitorizar.

**Q:** ¿Qué contramedida del IDS ayuda a detectar técnicas como tunneling y obfuscation?
**A:** Configurar reglas específicas del IDS para detectar túneles y ofuscación.

**Q:** ¿Qué modelo de política es la base de endurecimiento de firewall?
**A:** Deny all por defecto y habilitar solo los servicios necesarios.

**Q:** ¿Qué inspección del firewall ayuda frente a evasiones sobre tráfico cifrado?
**A:** Integrated HTTPS/TLS inspection.

**Q:** ¿Qué técnica del firewall inspecciona tráfico a nivel de aplicación?
**A:** Deep Packet Inspection (DPI).

**Q:** ¿Qué herramienta menciona el libro para probar evasiones sospechosas de firewall?
**A:** HTTP Evader.

**Q:** ¿Qué control reduce la posibilidad de que una evasión de endpoint se propague lateralmente?
**A:** Segmentación de red.

**Q:** ¿Qué principio reduce el impacto de malware en endpoints al limitar permisos?
**A:** Principle of least privilege.

**Q:** ¿Qué tres técnicas de protección de memoria aparecen como contramedidas en endpoints?
**A:** ASLR, DEP y CFI.

**Q:** ¿Qué control verifica la autenticidad de ejecutables y scripts en endpoints?
**A:** Code signing certificates.

**Q:** ¿Qué solución monitoriza cambios en archivos críticos y directorios?
**A:** File Integrity Monitoring (FIM).

**Q:** ¿Qué control del NAC clasifica dispositivos que intentan conectarse a la red?
**A:** Device profiling.

**Q:** ¿Qué modelo de autorización limita acceso según el rol del usuario en NAC?
**A:** Role-Based Access Control (RBAC).

**Q:** ¿Qué integración permite correlacionar eventos de NAC con otras telemetrías de seguridad?
**A:** Integración del NAC con SIEM.

**Q:** ¿Qué mecanismo fortalece NAC más allá de usuario y contraseña?
**A:** Certificate-based authentication o biometric authentication, además de MFA.

**Q:** ¿Qué tipo de detección debe usar un antivirus moderno para identificar amenazas desconocidas?
**A:** Detección basada en comportamiento, heurística, machine learning y threat intelligence.

**Q:** ¿Qué añade un EPP frente a un antivirus clásico según el enfoque del libro?
**A:** Capas adicionales de protección de endpoint más allá del motor antivirus tradicional.

**Q:** ¿Qué añade un EDR frente a prevención pura?
**A:** Telemetría, hunting y capacidades de respuesta sobre el endpoint.

**Q:** ¿Qué contramedida ayuda a analizar adjuntos y documentos maliciosos en tiempo real?
**A:** Herramientas de análisis de scripts en tiempo real.

**Q:** ¿Qué enfoque defensivo se recomienda contra evasiones de antivirus basadas en firmas estáticas?
**A:** Signature-less detection como anomaly detection y AI.

**Q:** ¿Qué objetivo tiene un syslog remoto en la defensa de firewalls?
**A:** Centralizar logs y dificultar que un atacante local los manipule o destruya.

## 5. Confusión frecuente

### Traffic normalization vs Packet filtering
**Diferencia:** el packet filtering decide permitir o bloquear; la traffic normalization **reescribe o sanea** el flujo para quitar ambigüedad antes del análisis.

**Criterio de decisión:** si la pregunta habla de **eliminar ambigüedad de fragmentación, campos inconsistentes o evasiones de parsing**, la opción correcta es **normalization**, no simple filtering.

### DPI vs HTTPS/TLS inspection
**Diferencia:** DPI inspecciona contenido de aplicación; TLS inspection permite ver dentro del tráfico cifrado para que esa inspección tenga sentido.

**Criterio de decisión:** si el tráfico está cifrado y la pregunta pide ver contenido real, prioriza **TLS inspection**. Si habla de inspección semántica de protocolos/aplicaciones, piensa en **DPI**.

### Antivirus vs EPP vs EDR 🔴
**Diferencia:** antivirus clásico detecta y bloquea malware; **EPP** amplía prevención en endpoint; **EDR** añade observación, hunting y respuesta.

**Criterio de decisión:** si la pregunta trata de **investigar actividad, correlacionar comportamiento o responder**, la respuesta suele ser **EDR**. Si trata de **prevenir ejecución o endurecer el host**, suele encajar mejor **EPP/AV**.

### MAC filtering vs Device profiling
**Diferencia:** MAC filtering verifica una dirección permitida; device profiling intenta identificar **qué tipo de dispositivo** es y si su comportamiento encaja con lo esperado.

**Criterio de decisión:** si el enunciado habla de **clasificación contextual del dispositivo**, elige **device profiling**. Si se centra en permitir/bloquear por dirección MAC, elige **MAC filtering**.

### Least privilege vs RBAC
**Diferencia:** least privilege es el principio general de dar el mínimo acceso necesario; RBAC es un mecanismo concreto para implementarlo mediante roles.

**Criterio de decisión:** si la pregunta pide el **principio**, responde **least privilege**. Si pide el **modelo de asignación de permisos por función/puesto**, responde **RBAC**.

### Sandbox vs Real-time protection
**Diferencia:** sandbox ejecuta en aislamiento para observar comportamiento; real-time protection inspecciona continuamente durante la operación normal del sistema.

**Criterio de decisión:** si el escenario habla de **detonar o analizar de forma aislada un archivo sospechoso**, es **sandbox**. Si habla de protección continua durante uso normal, es **real-time protection**.

### Honeypot vs Preventive control
**Diferencia:** el honeypot engaña, detecta y recopila inteligencia; no sustituye un control preventivo que bloquee o restrinja acceso.

**Criterio de decisión:** si el enunciado busca **desvío del atacante y obtención de información**, la respuesta es **honeypot**. Si busca **impedir directamente la acción**, no.
