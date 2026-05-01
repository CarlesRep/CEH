# M07 — Malware Threats
### Parte 2: APT Concepts

---

## 1. Definición de APT

Un **Advanced Persistent Threat (APT)** es un tipo de ataque de red en el que el atacante obtiene acceso no autorizado a una red objetivo y **permanece sin ser detectado durante un tiempo prolongado**. Las tres palabras del nombre tienen un significado técnico preciso:

- **Advanced:** uso de técnicas para explotar vulnerabilidades subyacentes del sistema. Múltiples zero-day exploits y malware personalizado.
- **Persistent:** sistema externo de **C&C (Command & Control)** que extrae datos continuamente y monitoriza la red de la víctima.
- **Threat:** involucración humana en la coordinación del ataque.

Los APT son ataques altamente sofisticados, planificados y coordinados. Los atacantes **borran evidencias** de sus actividades maliciosas tras cumplir sus objetivos. Se ejecutan principalmente contra organizaciones con información valiosa: financieras, sanidad, defensa y aeroespacial, manufactura y negocio.

El objetivo principal es **obtener información sensible**, no sabotear. La información obtenida incluye: documentos clasificados, credenciales, información personal de empleados/clientes, información de red, transacciones, tarjetas de crédito, estrategia de negocio, acceso a sistemas de control, propiedad intelectual, registros financieros, información técnica e infraestructura, emails y comunicaciones internas, datos operacionales (procesos de fabricación, cadena de suministro).

---

## 2. Características de los APT 🔴

El examen puede pedir cualquiera de estas características. Se presentan con su nombre y descripción concisa.

**Objectives:** obtener información sensible de forma repetida para beneficio económico ilegal. Secundariamente: espionaje por objetivos políticos o estratégicos.

**Timeliness:** tiempo que tarda el atacante desde la evaluación de vulnerabilidades hasta su explotación y mantenimiento del acceso.

**Resources:** cantidad de conocimiento, herramientas y técnicas necesarias. Los APT son ataques sofisticados que requieren recursos considerables y cibercriminales altamente cualificados.

**Risk Tolerance:** nivel hasta el que el ataque permanece sin ser detectado. Los APT están bien planificados con conocimiento profundo de la red objetivo, lo que les permite permanecer sin ser detectados durante mucho tiempo.

**Skills and Methods:** técnicas de ingeniería social para recopilar información, técnicas para evadir detección por mecanismos de seguridad, y técnicas para mantener acceso durante largo tiempo.

**Actions:** acciones técnicas que diferencian los APT de otros ciberataques. El objetivo principal es mantener presencia en la red de la víctima durante mucho tiempo y extraer el máximo de datos posible.

**Attack Origination Points:** múltiples intentos de entrada a la red objetivo. Requieren investigación exhaustiva para identificar vulnerabilidades y funciones de gatekeeper.

**Numbers Involved:** número de sistemas host involucrados. Los APT generalmente son ejecutados por grupos organizados de crimen.

**Knowledge Source:** recopilación de información sobre amenazas específicas mediante fuentes online para planificar la explotación.

**Multi-phased:** los APT siguen múltiples fases: **reconocimiento, acceso, descubrimiento, captura y exfiltración de datos**.

**Tailored to the Vulnerabilities:** el código malicioso se diseña específicamente para atacar las vulnerabilidades concretas presentes en la red de la víctima.

**Multiple Points of Entries:** el adversario crea múltiples puntos de entrada en la fase inicial. Si uno es descubierto y parcheado, usa otro alternativo.

**Evading Signature-Based Detection Systems:** los APT están estrechamente relacionados con exploits zero-day que contienen malware nunca descubierto previamente. Por eso pueden bypassar firewalls, antivirus, IDS/IPS y filtros anti-spam.

**Specific Warning Signs:** aunque son casi imposibles de detectar, hay indicios: actividad inusual de cuentas de usuario, presencia de backdoor Trojans, transferencias y uploads de ficheros inusuales, actividad inusual en bases de datos.

**Highly Targeted:** no son ataques aleatorios. Son meticulosamente planificados y ejecutados contra objetivos específicos: agencias gubernamentales, instituciones militares, corporaciones e infraestructura crítica.

**Long-term Engagement:** a diferencia de otras amenazas que buscan beneficio rápido, los APT buscan presencia a largo plazo. Los atacantes estudian el objetivo durante meses o años para personalizar sus métodos y evadir la detección.

**Use of Advanced Techniques:** spear-phishing, zero-day vulnerabilities, rootkits y malware multi-stage.

**Complex C2 Infrastructure:** infraestructura sofisticada de servidores C2 para comunicarse con sistemas comprometidos, recibir datos robados y enviar instrucciones adicionales. Esta infraestructura es **redundante y ofuscada** para resistir intentos de desmantelamiento.

---

## 3. APT Lifecycle: Las 6 Fases 🔴

El APT lifecycle describe el proceso sistemático que sigue un atacante para comprometer y explotar una red objetivo. Cada fase debe completarse antes de pasar a la siguiente.

### Fase 1: Preparation

El adversario define el objetivo, realiza investigación exhaustiva sobre él, organiza un equipo, construye o adquiere herramientas, y realiza tests de detección. Los APT requieren alto nivel de preparación porque el adversario no puede arriesgarse a ser detectado. Puede ser necesario obtener recursos y datos adicionales antes de ejecutar el ataque.

### Fase 2: Initial Intrusion

Primer intento de entrar en la red objetivo. Técnicas comunes:
- **Spear-phishing emails:** parecen legítimos pero contienen links o adjuntos con malware ejecutable. Los links redirigen al target a sitios donde el navegador y software son comprometidos.
- **Explotación de vulnerabilidades en servidores públicamente accesibles.**
- **Ingeniería social:** para recopilar información del objetivo.

En esta fase se despliega código malicioso o malware en el sistema objetivo para iniciar una **conexión saliente (outbound connection)**.

### Fase 3: Expansion

Objetivos: expandir acceso a la red y obtener credenciales. Si el objetivo era solo un sistema, no hay expansión. En la mayoría de casos, el atacante quiere acceder a múltiples sistemas desde el sistema inicialmente comprometido.

Acciones clave:
- Obtener credenciales de login de administrador de las **credenciales cacheadas** del sistema inicial.
- Escalar privilegios y ganar acceso a otros sistemas de la red.
- Cuando no se obtienen credenciales válidas: ingeniería social, explotar vulnerabilidades, distribuir USBs infectados.

**Tras obtener credenciales legítimas, el movimiento del atacante en la red es difícil de rastrear** porque usa nombre de usuario y contraseña válidos. Esta fase soporta la fase de Search and Exfiltration.

### Fase 4: Persistence

Mantener acceso al sistema objetivo. Empieza evadiendo dispositivos de seguridad de endpoint (IDS, firewalls), estableciendo acceso al sistema, y continuando hasta que no haya más utilidad para los datos y activos.

Técnicas:
- Uso de **malware personalizado y herramientas reempaquetadas** que no pueden ser detectadas por antivirus ni herramientas de seguridad del objetivo.
- Malware que incluye servicios, ejecutables y drivers instalados en múltiples sistemas de la red.
- Instalación del malware en ubicaciones **raramente examinadas**: routers, servidores, firewalls, impresoras.

### Fase 5: Search and Exfiltration

El atacante logra el objetivo último de la explotación: acceder a un recurso para ataques adicionales o beneficio financiero. Los atacantes pueden conocer la existencia de datos valiosos en la red sin saber su ubicación exacta.

Método común: robar **todos los datos** incluyendo documentos importantes, emails, drives compartidos y otros tipos de datos presentes en la red objetivo.

Los datos también pueden recopilarse mediante herramientas automatizadas como **network sniffers**. Los atacantes usan técnicas de cifrado para **evadir tecnologías DLP (Data Loss Prevention)** de la red objetivo.

### Fase 6: Cleanup

El atacante realiza acciones para prevenir la detección y eliminar evidencias del compromiso.

Técnicas usadas:
- Evadir detección.
- Eliminar evidencias de intrusión.
- Ocultar el objetivo del ataque y detalles del atacante.
- **Manipular datos en el entorno objetivo** para desorientar a los analistas de seguridad.

El atacante debe hacer que el sistema aparezca tal como estaba antes de que ganara acceso. Puede cambiar los atributos de cualquier fichero a su estado original (tamaño, fecha, etc.).

---

## 3. Exam Traps ⚠️

⚠️ **[APT: objetivo principal = información, no sabotaje]** — El examen puede presentar escenarios donde el objetivo del APT parece ser destruir la infraestructura. El objetivo principal de un APT es **obtener información sensible**, no sabotear. El sabotaje puede ocurrir pero no es el fin principal.

⚠️ **[Advanced = zero-day exploits, no solo "técnicas avanzadas" genéricas]** — "Advanced" en APT tiene un significado técnico específico: uso de zero-day exploits y malware personalizado para explotar vulnerabilidades subyacentes. No es simplemente "un ataque difícil".

⚠️ **[Persistent = C2, no solo "dura mucho tiempo"]** — "Persistent" hace referencia específicamente al sistema externo de C&C que extrae datos continuamente y monitoriza la red. No es solo que el ataque dure mucho tiempo.

⚠️ **[Fases del APT: 6, no 5]** — Preparation → Initial Intrusion → Expansion → Persistence → Search and Exfiltration → Cleanup. El examen puede omitir Preparation o Cleanup o presentar solo 5 fases. Son **6**.

⚠️ **[Multi-phased: las 5 fases mencionadas en características]** — En la sección de características, el libro menciona las fases como: reconocimiento, acceso, descubrimiento, captura y exfiltración de datos. Estas 5 son las fases mencionadas en la característica "Multi-phased", distintas de las 6 fases del lifecycle completo.

⚠️ **[Initial Intrusion: outbound connection]** — Tras desplegar el malware en el sistema objetivo, se inicia una **conexión saliente** (outbound), no entrante. El malware llama a casa. El examen puede preguntar la dirección de la conexión inicial.

⚠️ **[Expansion: credenciales cacheadas]** — En la fase de Expansion, el primer recurso para obtener credenciales de administrador son las **credenciales cacheadas** en el sistema inicial comprometido. El examen puede preguntar la fuente primaria de credenciales en esta fase.

⚠️ **[Persistence: ubicaciones raramente examinadas]** — Los routers, firewalls, impresoras y servidores son las ubicaciones preferidas para instalar malware de persistencia porque raramente se examinan para buscar malware. El examen puede preguntar qué hace especialmente efectiva la persistencia de un APT.

⚠️ **[Search and Exfiltration: evadir DLP con cifrado]** — Los atacantes usan cifrado específicamente para evadir las tecnologías DLP de la red objetivo durante la exfiltración. El examen puede preguntar qué técnica permite exfiltrar datos eludiendo DLP.

⚠️ **[Cleanup: manipulación de datos para desorientar]** — Además de borrar evidencias, el atacante puede **manipular datos** en el entorno para desorientar a los analistas de seguridad. No es solo borrar; también es falsificar.

⚠️ **[APT vs ataques convencionales: tiempo y detección]** — Los ataques convencionales buscan beneficio rápido; los APT buscan presencia a largo plazo (meses o años). La C2 infrastructure de los APT es redundante y ofuscada. El examen puede preguntar qué característica distingue más claramente un APT de un ataque estándar.

---

## 4. Nemotécnicos

**APT Lifecycle: 6 fases en orden:**
> **"Prep-Intro-Expand-Persist-Search-Clean"**
> **Prep**aration → **Intro**duction (Initial Intrusion) → **Expand** (Expansion) → **Persist**ence → **Search** and Exfiltration → **Clean**up

**Las 3 palabras de APT con su significado técnico:**
> **A** = zero-day exploits + malware personalizado
> **P** = sistema C&C que extrae datos continuamente
> **T** = involucración humana coordinada

**Características APT (16):** grupos de memorización:
> **Objetivos y timing:** Objectives, Timeliness, Risk Tolerance
> **Recursos y métodos:** Resources, Skills and Methods, Actions
> **Técnica:** Multi-phased, Tailored vulnerabilities, Multiple entries, Evades signatures
> **Perfil del ataque:** Highly targeted, Long-term, Advanced techniques, Complex C2
> **Detección:** Specific warning signs, Knowledge source

---

## 5. Flashcards

**Q:** ¿Qué significa "Persistent" en APT desde el punto de vista técnico?
**A:** La existencia de un sistema externo de C&C que extrae datos continuamente y monitoriza la red de la víctima.

**Q:** ¿Cuál es el objetivo principal de un APT?
**A:** Obtener información sensible, no sabotear. La presencia a largo plazo para exfiltrar datos es el fin.

**Q:** ¿Cuántas fases tiene el APT lifecycle?
**A:** 6: Preparation, Initial Intrusion, Expansion, Persistence, Search and Exfiltration, Cleanup.

**Q:** ¿Qué tipo de conexión inicia el malware en la fase de Initial Intrusion?
**A:** Una conexión saliente (outbound connection) hacia el servidor del atacante.

**Q:** ¿Cuál es el primer recurso que usa el atacante para obtener credenciales en la fase Expansion?
**A:** Las credenciales cacheadas del sistema inicialmente comprometido.

**Q:** ¿Por qué es difícil rastrear al atacante una vez obtiene credenciales válidas?
**A:** Porque usa nombre de usuario y contraseña legítimos, por lo que su movimiento en la red parece tráfico normal.

**Q:** ¿Qué técnica usan los APT para evadir las tecnologías DLP durante la exfiltración?
**A:** Cifrado de los datos antes de exfiltrarlos.

**Q:** ¿En qué ubicaciones instalan preferentemente el malware de persistencia los APT?
**A:** Routers, servidores, firewalls e impresoras, porque son ubicaciones raramente examinadas.

**Q:** ¿Qué diferencia el Cleanup de un APT de un simple borrado de logs?
**A:** El Cleanup también puede incluir la manipulación de datos en el entorno objetivo para desorientar a los analistas de seguridad.

**Q:** ¿Por qué los APT pueden bypassar firewalls, antivirus e IDS/IPS?
**A:** Porque están relacionados con zero-day exploits que contienen malware nunca descubierto previamente, por lo que no hay firmas de detección disponibles.

**Q:** ¿Cuáles son los indicios específicos de un APT en una red?
**A:** Actividad inusual de cuentas de usuario, presencia de backdoor Trojans, transferencias y uploads de ficheros inusuales, actividad inusual en bases de datos.

**Q:** ¿Qué caracteriza la infraestructura C2 de un APT?
**A:** Es redundante y ofuscada para resistir intentos de desmantelamiento.

**Q:** ¿Contra qué tipos de organizaciones se ejecutan principalmente los APT?
**A:** Organizaciones con información valiosa: financieras, sanidad, defensa y aeroespacial, manufactura y negocio.

**Q:** ¿Qué técnicas usa el atacante cuando no puede obtener credenciales válidas en la fase Expansion?
**A:** Ingeniería social, explotación de vulnerabilidades y distribución de USBs infectados.

**Q:** ¿Qué hace especialmente peligrosa la característica "Multiple Points of Entries" de un APT?
**A:** Si el analista descubre y parchea un punto de entrada, el atacante puede usar un punto de entrada alternativo para mantener el acceso.

---

## 6. Confusión frecuente

**APT objetivo principal: información vs sabotaje** → El objetivo principal es extraer información sensible (espionaje económico, político, estratégico). El sabotaje o daño a infraestructura puede ocurrir pero no es el fin primario de un APT clásico.

**Advanced (APT) vs "ataque avanzado" genérico** → En APT, "Advanced" se refiere específicamente al uso de zero-day exploits y malware personalizado adaptado a las vulnerabilidades concretas del objetivo, no simplemente a que sea un ataque técnicamente sofisticado.

**Persistent (APT) vs "ataque que dura mucho"** → "Persistent" en APT hace referencia al sistema C&C externo que mantiene comunicación continua con la red comprometida. No es simplemente que el ataque dure tiempo: es la infraestructura de control continuo.

**Fases del lifecycle (6) vs fases en la característica Multi-phased (5)** → El lifecycle tiene 6 fases: Preparation, Initial Intrusion, Expansion, Persistence, Search & Exfiltration, Cleanup. La característica "Multi-phased" menciona 5: reconocimiento, acceso, descubrimiento, captura y exfiltración. Son marcos de referencia distintos dentro del mismo libro.

**Expansion vs Initial Intrusion** → Initial Intrusion: primer punto de entrada al sistema objetivo, despliega malware, inicia outbound connection. Expansion: una vez dentro, se busca obtener credenciales de admin y acceder a múltiples sistemas adicionales.

**Persistence vs Cleanup** → Persistence: mantener acceso activo mediante malware en ubicaciones no examinadas. Cleanup: borrar/manipular evidencias para no ser detectado. Son fases distintas con objetivos opuestos: una mantiene presencia; la otra oculta rastros.

**APT vs ataque convencional** → APT: objetivo = información, largo plazo (meses/años), altamente dirigido, C2 compleja y redundante, múltiples puntos de entrada, evade firmas. Ataque convencional: objetivo = beneficio rápido, corto plazo, oportunista, infraestructura simple.

**Search and Exfiltration: método vs evasión** → El método de exfiltración puede ser robar todos los datos o usar network sniffers. La evasión de DLP se logra mediante **cifrado** de los datos exfiltrados. El examen puede preguntar separadamente el método y la técnica de evasión.

---

## 7. Preguntas de Práctica — Formato CEH

**P1.** Un analista de seguridad detecta que un atacante lleva 8 meses dentro de la red de una empresa financiera sin ser detectado, extrayendo datos periódicamente mediante un servidor de C2 externo. ¿Qué tipo de amenaza describe esta situación?

A) Ataque de Ingeniería Social  
B) Advanced Persistent Threat (APT)  
C) Ransomware con persistencia  
D) Insider Threat  

**Respuesta correcta: B**
La situación describe un **APT**: presencia prolongada sin detección (8 meses), extracción continua de datos y uso de servidor C2 externo. Las tres características de APT están presentes: Advanced (técnicas sofisticadas), Persistent (C&C continuo), Threat (involucración humana coordinada).

---

**P2.** En el APT lifecycle, tras comprometer el sistema inicial, el atacante necesita credenciales de administrador para moverse lateralmente. ¿Cuál es la primera fuente que aprovecha?

A) Ataques de brute-force contra el Active Directory  
B) Phishing a otros usuarios del dominio  
C) Credenciales cacheadas del sistema inicialmente comprometido  
D) Explotación de vulnerabilidades en servicios web internos  

**Respuesta correcta: C**
En la **Fase 3 — Expansion**, el primer recurso para obtener credenciales de administrador son las **credenciales cacheadas** del sistema inicial comprometido. Solo si éstas no están disponibles el atacante recurre a ingeniería social, explotación de vulnerabilidades o USBs infectados.

---

**P3.** Un SOC detecta que un APT exfiltra documentos clasificados hacia servidores externos. Tienen tecnologías DLP implementadas pero no logran bloquear la exfiltración. ¿Qué técnica usa el APT para evadir el DLP?

A) Fragmentación de paquetes IP  
B) Cifrado de los datos antes de exfiltrarlos  
C) Exfiltración mediante DNS tunneling solo  
D) Uso de protocolo ICMP para ocultar los datos  

**Respuesta correcta: B**
Los APT usan **cifrado** para evadir tecnologías DLP durante la exfiltración (Fase 5 — Search and Exfiltration). Si los datos están cifrados, el DLP no puede inspeccionar el contenido y verificar si contiene información sensible.

---

**P4.** ¿En qué fase del APT lifecycle el malware establece una conexión hacia el servidor del atacante por primera vez?

A) Preparation  
B) Initial Intrusion  
C) Expansion  
D) Persistence  

**Respuesta correcta: B**
En la **Fase 2 — Initial Intrusion**, tras desplegar el malware en el sistema objetivo (via spear-phishing u otras técnicas), éste inicia una **conexión saliente (outbound)** hacia el servidor del atacante. Esta conexión outbound es característica — el malware "llama a casa".

---

**P5.** Un equipo forense analiza un sistema comprometido por un APT y no encuentra evidencias del ataque en los logs. Sin embargo, nota que varios ficheros del sistema tienen fechas de modificación inconsistentes. ¿En qué fase del APT se realizaron estas acciones?

A) Persistence  
B) Search and Exfiltration  
C) Expansion  
D) Cleanup  

**Respuesta correcta: D**
La **Fase 6 — Cleanup** incluye no solo el borrado de evidencias sino también la **manipulación de datos y atributos de ficheros** (tamaño, fecha, etc.) para que aparezcan en su estado original. El atacante también puede manipular datos para desorientar a los analistas de seguridad.

---

**P6.** Un APT instala malware de persistencia en la red de la víctima. ¿En qué tipo de dispositivos instala preferentemente el malware para maximizar la persistencia?

A) Workstations de usuarios y laptops corporativas  
B) Routers, servidores, firewalls e impresoras  
C) Dispositivos móviles y tablets  
D) Bases de datos y servidores de aplicación  

**Respuesta correcta: B**
Los APT instalan malware de persistencia en **routers, servidores, firewalls e impresoras** porque son **ubicaciones raramente examinadas** en busca de malware. Esto maximiza la persistencia ya que los equipos de seguridad raramente escanean firmwares de dispositivos de red o impresoras.

---

**P7.** ¿Cuál de las siguientes afirmaciones describe correctamente el significado técnico de las tres palabras de "APT"?

A) Advanced = ataque complejo; Persistent = dura más de 30 días; Threat = requiere más de 5 atacantes  
B) Advanced = zero-day exploits y malware personalizado; Persistent = sistema C2 externo que extrae datos continuamente; Threat = involucración humana coordinada  
C) Advanced = usa más de 10 herramientas; Persistent = bypassa el firewall constantemente; Threat = causa daño financiero  
D) Advanced = explotación de vulnerabilidades públicas; Persistent = instalación de backdoors; Threat = grupo patrocinado por un estado  

**Respuesta correcta: B**
Los significados técnicos exactos son: **Advanced** = uso de zero-day exploits y malware personalizado para explotar vulnerabilidades subyacentes; **Persistent** = sistema externo de C&C que extrae datos continuamente y monitoriza la red; **Threat** = involucración humana coordinada en el ataque. Las otras opciones mezclan conceptos incorrectos.

---

**P8.** Un APT opera durante meses usando credenciales legítimas obtenidas en la fase de Expansion. ¿Por qué es esto especialmente difícil de detectar?

A) Las credenciales legítimas no generan alertas en IDS/IPS  
B) El atacante usa VPNs para ocultar su IP  
C) El movimiento del atacante parece tráfico normal porque usa nombre de usuario y contraseña válidos  
D) Los SIEM no monitorizan accesos con credenciales válidas  

**Respuesta correcta: C**
Una vez el atacante obtiene credenciales válidas, su **movimiento en la red es difícil de rastrear** porque usa un nombre de usuario y contraseña legítimos, haciendo que su actividad aparezca como tráfico normal de usuario autorizado. Esto es una característica fundamental de la fase Expansion que lo diferencia de ataques que usan técnicas de explotación evidentes.
