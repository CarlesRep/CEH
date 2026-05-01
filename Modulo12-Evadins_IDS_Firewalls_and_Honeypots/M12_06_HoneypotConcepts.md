# M12_06_HoneypotConcepts

## 1. Conceptos y definiciones

### Honeypot: qué es y para qué existe
Un honeypot es un sistema señuelo diseñado para atraer actividad no autorizada, registrar el comportamiento del atacante y extraer inteligencia útil sobre técnicas, herramientas, intenciones y patrones de ataque. Su lógica no es prestar un servicio de negocio, sino **convertir cualquier interacción en señal de alto valor**. Como no tiene valor productivo real ni debería recibir tráfico legítimo, cualquier conexión, escaneo, autenticación o interacción con él se interpreta como sospechosa o directamente maliciosa.

La ventaja conceptual del honeypot frente a otros controles es que **reduce drásticamente el ruido**. En un IDS/IPS o en un SIEM convencional, una parte del tráfico puede ser legítima y otra maliciosa; en un honeypot, la mera interacción ya es un indicador fuerte. Por eso se usa para tres fines principales: **prevención**, **detección** e **investigación**. No evita por sí mismo todas las intrusiones, pero sí permite detectar reconocimiento temprano, capturar TTPs, observar herramientas de post-explotación y comprender cómo opera un adversario en condiciones controladas.

También tiene una contrapartida operativa: cuanto más realista sea, más información capta, pero mayor es el riesgo. Un honeypot requiere mantenimiento, control del tráfico y una estrategia clara de aislamiento. Esa tensión entre **realismo**, **valor analítico** y **riesgo de compromiso** es la base de casi todas sus clasificaciones.

### Tipos de honeypots según el nivel de interacción 🔴
La clasificación más importante para examen es por **nivel de interacción**, porque determina cuánto se emula, qué calidad de datos se obtiene y cuánto riesgo asume el defensor.

| Tipo | Qué simula | Información capturada | Riesgo | Rasgo distintivo de examen | Ejemplos del chunk |
|---|---|---:|---:|---|---|
| Low-interaction | Un conjunto limitado de servicios y respuestas | Baja-media, principalmente transaccional | Bajo | Emula parcialmente; no suele poder ser comprometido por completo | tiny-ssh-honeypot, KFSensor, Honeytrap |
| Medium-interaction | Sistema/servicios más creíbles, pero con comandos preconfigurados | Media-alta | Medio | Permite interacciones más complejas, pero sigue sin ser totalmente real | Cowrie, Honeygrove, Kippo |
| High-interaction | Servicios y SO reales | Muy alta, incluyendo herramientas, técnicas e intención del atacante | Alto | No emula: expone sistemas reales controlados | Honeynet |
| Pure honeypot | Entorno que replica la red/producción real | Alta | Alto | Engaña al atacante haciéndole creer que interactúa con infraestructura crítica real | Producción simulada realista |

#### Low-interaction honeypots
Emulan solo una parte del comportamiento de un sistema. Son útiles para detectar sondeos, worms, port scanning y actividad automatizada porque ofrecen poco coste y bajo riesgo. El inconveniente es que un atacante con cierto nivel puede detectar inconsistencias rápidamente si intenta usar funcionalidades que el emulador no soporta.

**KFSensor** implementa servicios vulnerables y troyanos para atraer atacantes; puede monitorizar puertos y servicios TCP, UDP e ICMP, y generar alertas sobre port scanning y DoS. La lógica es sencilla: exponer superficie aparente suficiente para recibir interacción y registrar patrones de sondeo.

**Honeytrap** actúa como daemon de bajo nivel que arranca procesos servidor dinámicamente en los puertos solicitados. Recibe datos, los concatena en una *attack string* y los almacena. Esa cadena se analiza para ver si el atacante intenta descargar ficheros desde otros hosts. En el chunk se destaca que soporta **FTP y TFTP**, e identifica y registra **HTTP_URIs**. Esto suele ser material de memorización directa en CEH.

#### Medium-interaction honeypots 🔴
Buscan un engaño más creíble: simulan mejor el sistema operativo, las aplicaciones y el flujo de interacción. Capturan más contexto que los low-interaction, pero como solo responden a comandos predefinidos o a una lógica controlada, siguen pudiendo delatarse por comportamientos anómalos.

**Cowrie** es especialmente preguntable porque puede operar en dos modos. En modo de interacción media, emula una shell UNIX en Python para registrar fuerza bruta y comandos SSH/Telnet. En modo de interacción alta, actúa como proxy SSH/Telnet hacia otro sistema y observa el comportamiento del atacante sobre una máquina distinta. La clave de examen es no reducir Cowrie a “solo SSH honeypot”: el chunk lo presenta como **SSH/Telnet** y además **medium-to-high interaction**.

#### High-interaction honeypots 🔴
No emulan: exponen sistemas reales con sistemas operativos y aplicaciones reales. Eso permite observar la intrusión completa: explotación, escalada, persistencia, exfiltración, movimiento lateral y TTPs reales. El precio es que el atacante puede comprometer completamente el sistema si no está bien contenido.

El ejemplo nuclear es la **honeynet**, que no es una única máquina ni un producto específico, sino una **arquitectura**: una red completa de sistemas reales, monitorizada y aislada, diseñada para ser atacada. La pieza crítica de control es el **honeywall gateway**, que permite tráfico entrante hacia los sistemas trampa pero controla el tráfico saliente usando tecnologías de prevención de intrusiones. La idea no es solo capturar al atacante, sino **dejarle operar en un recinto controlado sin permitirle pivotar hacia terceros**.

#### Pure honeypots
El chunk los describe como entornos que emulan la red de producción real de la organización para desviar tiempo y recursos del atacante hacia lo que parece ser infraestructura crítica. Su valor está en generar alertas tempranas y descubrir vulnerabilidades explotadas por el atacante. En examen puede aparecer como categoría separada de high-interaction; la diferencia práctica es que el *pure honeypot* se enfatiza como **réplica de producción**, mientras que high-interaction describe el **nivel de realismo/interacción**.

### Tipos de honeypots según el despliegue
Aquí el criterio ya no es el grado de realismo, sino **para qué y dónde se despliegan**.

| Tipo | Objetivo principal | Interacción típica | Dónde se usa | Limitación principal |
|---|---|---|---|---|
| Production honeypot | Mejorar seguridad operativa y detectar actividad interna/externa | Normalmente low-interaction | Dentro de la red corporativa | Recoge menos inteligencia profunda |
| Research honeypot | Investigar TTPs, explotación y comportamiento del adversario | High-interaction | Institutos, gobiernos, militar, investigación | No mejora directamente la seguridad productiva de la empresa |

#### Production honeypots
Se despliegan dentro de la red real de la organización, junto al resto de sistemas. Están orientados a **detección operativa**, no tanto a investigación profunda. Por eso suelen ser low-interaction: menos riesgo, menos coste, más sencillos de mantener. También sirven para identificar amenazas internas, movimientos laterales y accesos impropios dentro del perímetro.

#### Research honeypots
Se usan para recolectar el máximo detalle sobre cómo se ejecuta un ataque, qué vulnerabilidades se explotan y qué herramientas se emplean. Son típicamente high-interaction y están asociados a entidades con interés analítico o estratégico, como gobiernos, ejército o centros de investigación. El punto de examen es que **no están pensados para reforzar directamente la infraestructura de producción**, sino para generar conocimiento.

### Tipos de honeypots según la tecnología de engaño 🔴
Esta clasificación se centra en **qué tipo de actividad atraen**.

| Tipo | Qué atrae | Qué busca obtener el defensor |
|---|---|---|
| Malware honeypot | Malware, worms, explotación automatizada | Patrones de malware, firmas, TTPs, actores |
| Database honeypot | SQLi, enumeración y ataques contra BBDD | TTPs específicas de bases de datos |
| Spam honeypot | Spammers, abuso de open relays/open proxies | Inteligencia sobre campañas y remitentes |
| Email honeypot / email trap | Correos maliciosos a direcciones señuelo | Técnicas de phishing, spoofing, distribución |
| Spider honeypot / spider trap | Crawlers y spidering malicioso | Identificación de scraping, enumeración web, recolección de URLs |
| Honeynet | Ataques de red completos | Capacidad integral del atacante |

#### Malware honeypots
Se configuran con vulnerabilidades conocidas, APIs obsoletas o servicios débiles —el chunk cita **SMBv1**, APIs desactualizadas, troyanos, virus y backdoors simulados— para atraer malware y explotación automatizada. Son útiles para identificar patrones de campaña y firmas, pero más importante aún, para analizar la secuencia de acciones del malware y su infraestructura asociada.

#### Database honeypots
Presentan bases de datos falsas que parecen contener información sensible, por ejemplo datos de tarjetas o empleados. Su valor está en que inducen al atacante a realizar **SQL injection**, **enumeración** o movimiento lateral relacionado con bases de datos. A partir de esas acciones se extraen TTPs concretas del atacante frente a entornos DB.

#### Spam honeypots y email honeypots
El examen puede mezclarlos. El **spam honeypot** se centra en infraestructura que acepta correo abusivo, como open relays y open proxies. El **email honeypot** o **email trap** consiste en direcciones de correo falsas diseminadas para recibir mensajes maliciosos. Uno atrapa la **campaña e infraestructura de spam**; el otro atrapa el **correo malicioso dirigido** hacia identidades trampa.

#### Spider honeypots
Son trampas para recolectores automáticos de información web. Se montan sitios falsos que parecen legítimos y permiten detectar a actores que realizan crawling o spidering para extraer URLs, contactos, directorios u otros metadatos valiosos.

### Honeynets y honeywall gateway 🔴
Una honeynet es una red de honeypots en una arquitectura controlada. La pieza clave es el **honeywall gateway**, que media entre el atacante y los sistemas trampa. Debe dejar pasar suficiente tráfico como para no romper la ilusión, pero limitar el tráfico saliente para impedir que el entorno se use como plataforma de ataque. Este equilibrio entre observación y contención es fundamental: una honeynet útil debe permitir interacción rica, pero nunca convertirse en un punto de pivote real.

### Herramientas de honeypot mencionadas
El chunk destaca **HoneyBOT** como herramienta principal. Se describe como un **honeypot de interacción media para Windows**, orientado a crear un entorno seguro que capture tráfico no solicitado y funcione como componente de investigación o sistema de alerta temprana similar a un early-warning IDS.

Otras herramientas listadas: **Blumira honeypot software**, **NeroSwarm Honeypot**, **Valhala Honeypot**, **Cowrie** y **StingBox**. En CEH suelen aparecer como reconocimiento nominal, no tanto por detalle técnico profundo salvo Cowrie.

### Detección de honeypots: lógica general
Desde la perspectiva ofensiva, detectar un honeypot consiste en buscar **inconsistencias entre lo que aparenta ser y cómo se comporta realmente**. Todo se resume en esto: el honeypot quiere parecer un sistema real; el atacante busca fallos en esa simulación.

Los métodos principales del chunk son:

#### Fingerprinting del servicio en ejecución
Se compara el banner, la versión declarada y el comportamiento real del servicio. Si el servicio dice ser una versión concreta pero responde como otra, o no implementa funcionalidades esperables, puede tratarse de un honeypot. El comando dado es:

```bash
nmap -sV -p 80 <target_ip>
```

La idea no es solo identificar el servidor web, sino detectar discrepancias entre firma y comportamiento.

#### Análisis del tiempo de respuesta 🔴
Los honeypots a menudo añaden capas de logging, instrumentación o emulación que introducen **latencias altas o inconsistentes**. El atacante compara RTTs y patrones temporales con lo normal para ese servicio.

```bash
nmap -p 80 --scan-delay 1s --max-retries 5 <target_ip>
```

Indicadores: respuesta lenta constante o variabilidad anómala.

#### Análisis de direcciones MAC
Se revisa el **OUI** de la MAC para ver si corresponde a fabricantes conocidos o si delata virtualización/honeypot. El comando dado es:

```bash
arp-scan --interface=eth0 --localnet
```

Esto también conecta con la detección de VMware: si la MAC cae en un rango asignado a VMware, el atacante sospecha de virtualización y, por extensión, de posible honeypot.

#### Enumeración de puertos inesperados
Si un host que parece ser un servidor web expone más servicios de los razonables, el atacante sospecha. El chunk pone el ejemplo de un supuesto servidor web con **80/443** abiertos, pero además **22, 21 y puertos altos** sin una justificación clara.

```bash
nmap -p- <target_ip>
```

La clave no es “muchos puertos = honeypot” sin más, sino “**superficie expuesta incoherente con el rol aparente del sistema**”.

#### Análisis de configuración y metadatos
Banners por defecto, versiones obsoletas, cabeceras HTTP incoherentes, *welcome messages* SSH genéricos o metadatos inconsistentes suelen revelar despliegues artificiales o mal terminados. El atacante intenta correlacionar banner, stack, rol del host y comportamiento observado.

### Tar pits y su detección 🔴
Un **tar pit** no es exactamente un honeypot clásico: su propósito principal no es atraer y observar, sino **ralentizar** o atascar la actividad maliciosa. El examen puede confundirlos deliberadamente.

#### Layer 7 tar pits
Operan a nivel de aplicación, por ejemplo respondiendo lentamente a comandos SMTP para frustrar spammers. Se detectan observando **latencia inusualmente alta** en el servicio.

#### Layer 4 tar pits
Manipulan el stack TCP/IP. El ejemplo del chunk es que aceptan conexiones TCP y luego anuncian **TCP window size = 0**, impidiendo el envío posterior de datos. Siguen reconociendo paquetes aunque el flujo está efectivamente bloqueado. El atacante los detecta viendo ACKs continuos combinados con ventana cero.

#### Layer 2 tar pits
Actúan en la misma red local para frenar al atacante una vez dentro. El chunk indica un identificador muy memorizable: la MAC **00:00:0F:FF:FF:FF**. También pueden detectarse revisando respuestas ARP anómalas.

### Detecciones específicas de infraestructuras de honeypot

#### Honeypots sobre VMware
La virtualización no implica automáticamente honeypot, pero un rango OUI asignado a VMware puede levantar sospechas. En CEH la asociación clave es **VMware ↔ análisis de MAC address**.

#### Honeyd
Honeyd es un daemon de honeypot muy usado para crear miles de honeypots simulados. Se detecta mediante **time-based TCP fingerprinting** y observando su comportamiento como **SYN proxy**. El rasgo no es tanto un banner concreto como un patrón temporal/anómalo de respuesta TCP.

#### User-Mode Linux (UML) Honeypot
Puede delatarse por información específica en:

- `/proc/mounts`
- `/proc/interrupts`
- `/proc/cmdline`

Aquí el examen suele pedir “qué ficheros revisar” más que el detalle interno de UML.

#### Snort_inline Honeypot
Snort_inline es una variante de Snort con capacidad de manipular paquetes. Se usó en honeynets GenII para bloquear ataques conocidos y evitar que el atacante rebote desde el entorno trampa. El atacante puede detectarlo observando **modificaciones o drops en paquetes salientes** comparando tráfico desde otro host.

#### Fake AP
Un **fake access point** emite *beacon frames* 802.11b con **ESSID y BSSID aleatorios**, pero no genera tráfico real asociado al punto de acceso. La ausencia de tráfico coherente con esos beacons es la señal de alerta.

#### Bait and switch honeypots
Redirigen tráfico malicioso hacia un honeypot una vez detectado el intento de intrusión. El atacante puede detectarlos comparando parámetros TCP/IP como:

- **RTT**
- **TTL**
- **TCP timestamp**

Si estos cambian de forma inconsistente durante la sesión, puede haber redirección encubierta.

### Herramientas de detección de honeypots
El chunk menciona **Send-Safe Honeypot Hunter**, orientada a comprobar listas de proxies **HTTPS, SOCKS4 y SOCKS5** para identificar proxies que realmente sean honeypots. Funcionalidades destacadas:

- comprobar proxylists locales o remotas
- trabajar con varios listados a la vez
- subir resultados por FTP
- automatizar el procesado periódico
- validar proxylists convencionales además de detectar honeypots

En CEH interesa más la asociación **herramienta → propósito** que la operativa profunda.

## 2. Exam Traps ⚠️

⚠️ **[Honeypot vs Honeynet]** El examen puede presentar ambos como sinónimos. No lo son. Un **honeypot** es un sistema señuelo; una **honeynet** es una **arquitectura/red de sistemas trampa** monitorizada y controlada, normalmente con un **honeywall gateway**.

⚠️ **[Low-interaction vs High-interaction]** La trampa habitual es pensar que ambos solo cambian en “cantidad de logs”. La diferencia real es que el **low-interaction emula servicios limitados** y no suele poder ser comprometido por completo, mientras que el **high-interaction usa servicios y SO reales** y sí puede comprometerse en un entorno controlado.

⚠️ **[Production honeypot vs Research honeypot]** El examen puede hacer creer que ambos mejoran la seguridad operativa por igual. El **production honeypot** sí busca detección dentro de la red real; el **research honeypot** busca **inteligencia profunda sobre el atacante**, no protección productiva directa.

⚠️ **[Spam honeypot vs Email honeypot]** No son idénticos. El **spam honeypot** se centra en capturar abuso de **mail relays/proxies/infraestructura de spam**. El **email honeypot** o **email trap** son **direcciones falsas** para atraer correos maliciosos.

⚠️ **[Spider honeypot]** Puede aparecer como si protegiera usuarios finales o correo. No: está diseñado para **atrapar crawlers/spiders** que extraen URLs, directorios, contactos y metadatos web.

⚠️ **[Cowrie]** Es fácil que lo simplifiquen como “SSH honeypot”. En el chunk es **SSH y Telnet**, y además **medium-to-high interaction** según el modo de funcionamiento.

⚠️ **[Honeytrap]** La pregunta puede centrarse en qué protocolos usa para descargar ficheros detectados en la attack string. La respuesta correcta del chunk es **FTP y TFTP**, no HTTP.

⚠️ **[KFSensor]** La confusión típica es reducirlo a un detector de puertos TCP. El chunk dice que monitoriza **TCP, UDP e ICMP** y alerta de **port scanning y DoS**.

⚠️ **[Fingerprinting del servicio]** No consiste solo en leer banners. La clave es buscar **discrepancias entre banner, versión declarada y comportamiento real del servicio**.

⚠️ **[Response time analysis]** El examen puede presentarlo como “latencia alta = sistema saturado”. En contexto de honeypot detection, la latencia anómala o inconsistente puede delatar **logging/monitorización/emulación** adicionales.

⚠️ **[Unexpected open ports]** No basta con “hay muchos puertos abiertos”. El criterio correcto es que los puertos abiertos sean **incoherentes con el rol aparente** del sistema.

⚠️ **[VMware detection]** La prueba típica no es un comando de virtualización complejo, sino el análisis del **OUI/MAC address range asignado a VMware**.

⚠️ **[Layer 7 tar pit vs Layer 4 tar pit]** Layer 7 ralentiza protocolos de aplicación como **SMTP**. Layer 4 manipula **TCP/IP**, por ejemplo anunciando **window size = 0**. El examen suele mezclarlos porque ambos “ralentizan” al atacante.

⚠️ **[Layer 2 tar pit]** El dato distintivo memorizable es la MAC **00:00:0F:FF:FF:FF**. Si aparece esa opción, probablemente buscan la respuesta de layer 2 tar pit.

⚠️ **[Honeyd detection]** La trampa es pensar que se detecta por banner. El chunk destaca **time-based TCP fingerprinting** y **SYN proxy behavior**.

⚠️ **[Snort_inline]** No se identifica solo porque “bloquea ataques”. Lo distintivo es que **puede modificar paquetes**; el atacante puede detectarlo observando **drops o cambios en tráfico saliente** desde otro host.

⚠️ **[Fake AP]** La clave no es solo la emisión de beacons falsos, sino que **no produce tráfico real asociado al AP**. Beacon sin tráfico coherente = indicador fuerte.

⚠️ **[Bait and switch honeypot]** No se detecta buscando puertos falsos, sino variaciones sospechosas en **RTT, TTL y TCP timestamp** por la redirección del tráfico.

⚠️ **[Send-Safe Honeypot Hunter]** El examen puede ponerla como herramienta de cracking o proxy chaining. En este chunk se usa para **comprobar listas de proxies HTTPS/SOCKS4/SOCKS5 y detectar honeypots**.

## 3. Nemotécnicos

### Clasificación por interacción: **“Lo emula poco, Medio finge mejor, High es real”**
- **Low**: poco realismo, bajo riesgo, pocos datos
- **Medium**: engaño intermedio, más datos, más riesgo
- **High**: real, máximo dato, máximo riesgo

### Despliegue: **“Production protege, Research aprende”**
- **Production** = mejora detección operativa en la empresa
- **Research** = estudia al atacante en profundidad

### Honeytrap: **“Ataque en cadena: guarda string, mira FTP/TFTP, registra URI”**
- concatena datos en **attack string**
- busca descargas por **FTP/TFTP**
- registra **HTTP_URIs**

### Cowrie: **“Cowrie = Credenciales + Consola”**
Te recuerda que registra **brute force** y **shell interactions** en **SSH/Telnet**.

### Tar pits por capa: **“7 tarda, 4 cierra ventana, 2 cae al agujero MAC”**
- **L7** → responde lento
- **L4** → **TCP window size = 0**
- **L2** → MAC especial **00:00:0F:FF:FF:FF**

### Detección de honeypots: **“Servicio, Tiempo, MAC, Puertos, Metadatos”**
Orden útil para recordar el bloque principal:
1. **Servicio** fingerprinting
2. **Tiempo** de respuesta
3. **MAC** / OUI
4. **Puertos** inesperados
5. **Metadatos** y configuración

### Bait and switch: **“RTT-TTL-Timestamp”**
Si ves redirección encubierta tras detectar intrusión, piensa en:
- **RTT**
- **TTL**
- **TCP timestamp**

### UML detection: **“Mounts, Interrupts, Cmdline”**
Ficheros a revisar:
- `/proc/mounts`
- `/proc/interrupts`
- `/proc/cmdline`

## 4. Flashcards

**Q:** ¿Qué característica convierte a un honeypot en una fuente de alta señal para detección?
**A:** Que no tiene valor productivo ni actividad autorizada; cualquier interacción suele ser un probe, ataque o compromiso.

**Q:** ¿Cuál es la diferencia principal entre un honeypot y una honeynet?
**A:** El honeypot es un sistema señuelo; la honeynet es una arquitectura o red completa de sistemas trampa monitorizada y controlada.

**Q:** ¿Qué tipo de honeypot emula solo un número limitado de servicios y aplicaciones?
**A:** El low-interaction honeypot.

**Q:** ¿Qué tipo de honeypot proporciona la mayor cantidad de información sobre técnicas, herramientas e intención del atacante?
**A:** El high-interaction honeypot.

**Q:** ¿Qué componente controla el tráfico saliente en una honeynet para evitar que el atacante dañe otros sistemas?
**A:** El honeywall gateway.

**Q:** ¿Qué tipo de honeypot se despliega normalmente dentro de la red corporativa para mejorar la seguridad operativa?
**A:** El production honeypot.

**Q:** ¿Qué tipo de honeypot usan normalmente gobiernos o centros de investigación para obtener inteligencia detallada del atacante?
**A:** El research honeypot.

**Q:** ¿Qué herramienta del chunk se describe como honeypot de interacción media para Windows?
**A:** HoneyBOT.

**Q:** ¿Qué honeypot del chunk es SSH/Telnet y puede operar en modo medium o high interaction?
**A:** Cowrie.

**Q:** ¿Qué honeypot concatena los datos recibidos en una attack string y puede intentar descargar archivos por FTP o TFTP?
**A:** Honeytrap.

**Q:** ¿Qué herramienta se describe como capaz de monitorizar puertos TCP, UDP e ICMP y alertar sobre port scanning y DoS?
**A:** KFSensor.

**Q:** ¿Qué comando se usa en el chunk para fingerprinting del servicio web en el puerto 80?
**A:** `nmap -sV -p 80 <target_ip>`

**Q:** ¿Qué comando se usa para medir tiempos de respuesta en un servicio HTTP y buscar latencias anómalas?
**A:** `nmap -p 80 --scan-delay 1s --max-retries 5 <target_ip>`

**Q:** ¿Qué comando se usa para enumerar MAC addresses en la red local y analizar OUIs?
**A:** `arp-scan --interface=eth0 --localnet`

**Q:** ¿Qué comando del chunk escanea todos los puertos del objetivo para detectar puertos inesperados?
**A:** `nmap -p- <target_ip>`

**Q:** ¿Qué dos indicadores delatan con frecuencia un honeypot al hacer fingerprinting del servicio?
**A:** Discrepancias entre la versión declarada y el comportamiento real, o ausencia de funciones esperadas del servicio.

**Q:** ¿Qué característica temporal puede delatar un honeypot durante el análisis de response time?
**A:** Respuestas consistentemente lentas o con variabilidad anómala.

**Q:** ¿Qué dato específico puede sugerir la presencia de una VM de VMware usada como honeypot?
**A:** Un rango OUI/MAC address asignado a VMware.

**Q:** ¿Qué valor de MAC se asocia en el chunk a la detección de layer 2 tar pits?
**A:** `00:00:0F:FF:FF:FF`

**Q:** ¿Qué rasgo de red caracteriza a un layer 4 tar pit?
**A:** Que sigue reconociendo paquetes mientras mantiene el TCP window size reducido a cero.

**Q:** ¿Qué protocolo se usa como ejemplo típico de servicio ralentizado por un layer 7 tar pit?
**A:** SMTP.

**Q:** ¿Qué técnica se usa para detectar Honeyd según el chunk?
**A:** Time-based TCP fingerprinting observando el comportamiento tipo SYN proxy.

**Q:** ¿Qué archivos puede revisar un atacante para detectar un honeypot basado en User-Mode Linux?
**A:** `/proc/mounts`, `/proc/interrupts` y `/proc/cmdline`.

**Q:** ¿Cómo puede un atacante detectar un entorno con Snort_inline en una honeynet?
**A:** Observando paquetes salientes modificados o descartados desde otro host.

**Q:** ¿Qué delata un fake AP según el chunk?
**A:** Que emite beacon frames falsos con ESSID/BSSID aleatorios, pero no genera tráfico real asociado al AP.

**Q:** ¿Qué parámetros TCP/IP pueden delatar un bait and switch honeypot?
**A:** RTT, TTL y TCP timestamp.

**Q:** ¿Qué tipo de honeypot está orientado a atrapar crawlers y spidering malicioso?
**A:** El spider honeypot o spider trap.

**Q:** ¿Qué diferencia esencial hay entre spam honeypot y email honeypot?
**A:** El spam honeypot atrae abuso de infraestructura de correo; el email honeypot usa direcciones falsas para atraer emails maliciosos.

**Q:** ¿Qué herramienta del chunk comprueba proxies HTTPS, SOCKS4 y SOCKS5 para identificar honeypots?
**A:** Send-Safe Honeypot Hunter.

## 5. Confusión frecuente

### Honeypot vs Honeynet
**Diferencia memorable:** uno es una **trampa individual**; la otra es una **arquitectura completa**.

**Criterio de decisión:** si la pregunta habla de una **red de sistemas reales**, monitorización integral y control del tráfico con **honeywall**, la respuesta es **honeynet**.

### Low-interaction vs Medium-interaction
**Diferencia memorable:** el low “responde poco”; el medium “finge mejor”.

**Criterio de decisión:** si el sistema solo emula servicios básicos y falla ante interacciones inesperadas, es **low**. Si simula OS/servicios con más profundidad pero sigue limitado a comandos preconfigurados, es **medium**.

### Medium-interaction vs High-interaction
**Diferencia memorable:** el medium **simula**, el high **es real**.

**Criterio de decisión:** si el atacante puede comprometer un sistema real en un entorno controlado, es **high-interaction**.

### Production honeypot vs Research honeypot
**Diferencia memorable:** uno **protege la operación**, el otro **estudia al atacante**.

**Criterio de decisión:** si el objetivo es mejorar detección interna con menor riesgo, **production**. Si el objetivo es obtener inteligencia detallada de TTPs y explotación, **research**.

### Spam honeypot vs Email honeypot
**Diferencia memorable:** uno atrapa **infraestructura y campañas**, el otro atrapa **mensajes dirigidos a identidades trampa**.

**Criterio de decisión:** si la pregunta menciona open relays/open proxies, es **spam honeypot**. Si menciona direcciones falsas distribuidas para atraer correo malicioso, es **email honeypot**.

### Honeypot detection vs Tar pit detection
**Diferencia memorable:** el honeypot quiere **engañarte**; el tar pit quiere **atascarte**.

**Criterio de decisión:** si el foco está en banners, servicios, puertos, OUI o metadatos, probablemente es **honeypot detection**. Si el foco está en **latencia artificial**, **ventana TCP a cero** o MAC especial a nivel 2, es **tar pit**.

### Honeyd vs Fake AP
**Diferencia memorable:** Honeyd engaña en **servicios de red**; el fake AP engaña en **Wi-Fi/beacons**.

**Criterio de decisión:** si el indicador es **SYN proxy/time-based TCP fingerprinting**, piensa en **Honeyd**. Si el indicador es **beacon frames con ESSID/BSSID aleatorios sin tráfico real**, piensa en **fake AP**.

### VMware detection vs Honeypot certainty
**Diferencia memorable:** VMware no significa automáticamente honeypot.

**Criterio de decisión:** la MAC/OUI de VMware solo sugiere **virtualización**. Se vuelve indicio de honeypot cuando se combina con otros artefactos de simulación o incoherencias del sistema.
