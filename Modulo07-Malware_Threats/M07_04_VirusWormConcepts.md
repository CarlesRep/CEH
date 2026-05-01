# M07 — Malware Threats
### Parte 4: Virus and Worm Concepts

---

## 1. ¿Qué es un Virus?

Un virus informático es un **programa autorreplicante** que produce su código adjuntando copias de sí mismo a otro código ejecutable, operando sin conocimiento ni consentimiento del usuario. Como un virus biológico, es contagioso y puede contaminar otros ficheros, pero solo puede infectar máquinas externas **con la asistencia de los usuarios**.

Características de los virus:
- Infects other programs.
- Se transforma a sí mismo.
- Se cifra a sí mismo.
- Altera datos.
- Corrompe ficheros y programas.
- Se replica a sí mismo.

**Diferencia clave con gusanos:** los virus necesitan un host y asistencia humana para propagarse. Los gusanos se propagan solos por la red.

---

## 2. Fases del Ciclo de Vida del Virus

1. **Design:** desarrollo del código del virus usando lenguajes de programación o kits de construcción.
2. **Replication:** el virus se replica durante un período en el sistema objetivo y luego se propaga.
3. **Launch:** el virus se activa cuando el usuario realiza acciones específicas como ejecutar un programa infectado.

---

## 3. Fases de Funcionamiento de un Virus 🔴

### Infection Phase
El virus se carga en memoria, busca un ejecutable en disco, adjunta código malicioso a un programa legítimo sin conocimiento del usuario. El usuario lanza el programa infectado, lo que infecta otros programas del sistema. El ciclo continúa hasta que el usuario detecta la anomalía.

Secuencia de infección:
1. El virus se carga en memoria y busca un ejecutable en disco.
2. Adjunta código malicioso a un programa legítimo.
3. El usuario ejecuta el programa infectado sin saberlo.
4. La ejecución infecta otros programas del sistema.
5. El ciclo continúa.

Los boot sector viruses ejecutan su código **antes de que el PC arranque**.

Métodos de propagación: ficheros infectados, servicios de file-sharing, USBs y medios de almacenamiento, adjuntos maliciosos y descargas.

### Attack Phase
Una vez extendidos por el sistema, los virus corrompen ficheros y programas. Algunos solo se activan tras un evento trigger. Los más avanzados **ocultan su presencia** y atacan solo después de haberse extendido completamente.

Acciones comunes: borrar ficheros, alterar contenido de ficheros de datos, ejecutar tareas no relacionadas con aplicaciones (reproducir música, crear animaciones).

---

## 4. Tipos de Virus (23 tipos) 🔴

### 1. System / Boot Sector Virus
Ataca el **Master Boot Record (MBR)** y el **DOS Boot Record**. El MBR es la zona más propensa a virus porque si se corrompe, se pierden todos los datos. El sector del sistema solo tiene 512 bytes, por lo que estos virus ocultan su código en otro espacio del disco.

**Funcionamiento:** mueve el MBR a otra ubicación del disco duro y copia su código en la ubicación original del MBR. Al arrancar el sistema, primero se ejecuta el código del virus y luego se pasa el control al MBR original.

Portadores principales: adjuntos de email y medios extraíbles (USB). Residen en memoria.

### 2. File Virus
Infecta ficheros ejecutables o interpretados: COM, EXE, SYS, OVL, OBJ, PRG, MNU, BAT. Puede ser direct-action (no residente) o memory-resident. Inserta su código en el fichero original. No muestra aumento de tamaño en el listado de directorio (técnica stealth).

### 3. Multipartite Virus (Hybrid Virus)
Combina el enfoque de file infectors y boot record infectors. Ataca simultáneamente el **boot sector Y los ficheros ejecutables/de programa**. Si infecta el boot sector, afecta a los ficheros del sistema y viceversa. Reinfecta el sistema repetidamente si no se elimina completamente.

### 4. Macro Virus
Infecta aplicaciones Microsoft Word o similares ejecutando automáticamente una secuencia de acciones. Escritos en **Visual Basic for Applications (VBA)**. Infectan plantillas o convierten documentos infectados en ficheros de plantilla manteniendo su apariencia de documentos normales. Se propagan principalmente via email. Aprovechan aplicaciones con macro capability: Word, Excel, otros programas Office, ficheros Windows Help.

### 5. Cluster Virus
Infecta ficheros **sin cambiar el fichero ni plantar ficheros adicionales**. Guarda el código del virus en el disco duro y sobreescribe el puntero en la entrada de directorio, redirigiendo el punto de lectura del disco al código del virus en lugar del programa real. Solo existe **una copia del virus en el disco** aunque afecte a todos los programas.

### 6. Stealth / Tunneling Virus
Se ocultan de los programas antivirus **alterando y corrompiendo activamente las interrupciones de service call** mientras se ejecutan. Dan información falsa para ocultar su presencia. Ocultan el tamaño original del fichero o colocan temporalmente una copia en otro drive, reemplazando el fichero infectado por el fichero no infectado almacenado en disco. Residen en memoria. Uno de sus carriers es el rootkit.

### 7. Encryption Virus (Cryptolocker)
Consta de una copia cifrada del virus y un **módulo de descifrado**. El módulo de descifrado permanece constante; la encriptación usa diferentes claves. Usa XOR en cada byte con una clave aleatoria. Bloquea el acceso a la máquina o da acceso limitado. El escáner de virus no puede detectarlo por firmas, pero **sí puede detectar el módulo de descifrado**.

### 8. Sparse Infector Virus
Infecta **menos frecuentemente** para minimizar su probabilidad de ser descubierto. Infecta solo ocasionalmente (ej. cada décimo programa ejecutado) o solo ficheros que cumplan ciertas condiciones (ej. tamaño máximo de 128 KB). Estrategia de "wakeup call" para días específicos del mes.

### 9. Polymorphic Virus 🔴
**Modifica su código en cada replicación** para evitar la detección, cambiando el módulo de cifrado y la secuencia de instrucciones. Usa generadores de números aleatorios.

Tres componentes:
- **Encrypted virus code:** el código malicioso cifrado.
- **Decryptor routine:** descifra el código del virus y toma control. Varía en cada infección.
- **Mutation engine:** genera rutinas de descifrado aleatorias para cada nueva infección.

Proceso: al ejecutar un programa infectado, la rutina de descifrado toma control → descifra el virus y el mutation engine → el virus localiza un nuevo programa a infectar → crea una réplica con nueva rutina de descifrado generada por el mutation engine → cifra las nuevas copias → adjunta al nuevo programa.

**Ninguna infección parece igual.** Los escáneres de virus los detectan con dificultad.

### 10. Metamorphic Virus 🔴
**Se reescribe completamente** cada vez que infecta un nuevo fichero ejecutable. Más efectivo que el polimórfico. El algoritmo original permanece intacto pero el código cambia totalmente. Usa un metamorphic engine.

Técnicas de transformación: disassembler, expander, permutator, assembler.

Pasos de transformación del cuerpo:
1. Inserta dead code.
2. Reshapes expressions.
3. Reorders instructions.
4. Modifies variable names.
5. Encrypts program code.
6. Modifies program control structure.

**Diferencia clave con Polymorphic:** Polymorphic cifra el virus y cambia la rutina de descifrado; el código del virus es siempre el mismo, solo varía cómo se descifra. Metamorphic reescribe completamente el código del virus en cada infección; no necesita cifrado.

### 11. Overwriting File / Cavity Virus (Space Filler)
Sobreescribe parte del fichero host con una constante (generalmente nulls) **sin aumentar la longitud del fichero** mientras preserva su funcionalidad. Mantener el tamaño constante permite evitar la detección. Raramente encontrado por la falta de hosts adecuados y complejidad del código.

### 12. Companion / Camouflage Virus
Se almacena con el **mismo nombre de fichero** que el programa objetivo. Crea un fichero `.COM` idéntico al ejecutable `.EXE` objetivo. DOS ejecuta ficheros COM antes que EXE con el mismo nombre raíz. El virus se ejecuta primero, infecta más ficheros, y luego carga y ejecuta el `.EXE` legítimo. El usuario probablemente no nota nada anómalo. Fácil de detectar por la presencia del fichero `.COM` extra.

### 13. Shell Virus
El código del virus forma una **shell alrededor del código del programa host**, convirtiéndose en el programa original con el código host como subrutina. Casi todos los boot program viruses son shell viruses.

### 14. File Extension Virus
Cambia las extensiones de ficheros. Aprovecha que Windows **oculta extensiones conocidas por defecto**. Un fichero llamado `BAD.TXT.VBS` aparece como `BAD.TXT`. El usuario cree que es texto y lo abre; en realidad es un script Visual Basic ejecutable.

**Mitigación:** desactivar "Hide file extensions" en Windows (Control Panel → Appearance and Personalization → View tab → Uncheck "Hide extensions for known file types").

### 15. FAT Virus
Ataca el **File Allocation Table (FAT)**, la tabla usada para localizar ficheros en disco. Puede destruir el índice haciendo imposible que el ordenador localice ficheros. El virus puede propagarse a ficheros cuando el FAT intenta acceder a ellos, corrompiendo eventualmente todo el sistema. Manifestaciones: ficheros corruptos, desaparecidos o inaccesibles; cambios en la arquitectura FAT (ej. el sistema dice usar FAT12 cuando debería usar FAT32).

### 16. Logic Bomb Virus
Virus **activado por respuesta a un evento**: lanzamiento de una aplicación, o cuando se alcanza una fecha/hora específica. Cuando está programado para ejecutarse en una fecha específica se denomina **Time Bomb**. Los time bombs suelen programarse para fechas importantes (Navidad, San Valentín). Ejemplo: un keylogger que espera hasta que el usuario visita un sitio bancario para activarse y robar credenciales.

### 17. Web Scripting Virus
Viola la seguridad del navegador a través de un sitio web, **inyectando scripting del lado del cliente** en la página web. Bypasa controles de acceso y roba información del navegador. Ataca sitios con grandes poblaciones: redes sociales, reseñas de usuarios, email. Se propaga más rápido que otros virus.

Dos tipos:
- **Non-persistent:** ataca sin conocimiento del usuario.
- **Persistent:** roba cookies directamente; el atacante puede secuestrar la sesión e impersonar al usuario.

### 18. Email Virus
Código enviado como adjunto de email que, si se activa, produce efectos dañinos: destruir ficheros, enviar el adjunto a todos los contactos del libro de direcciones. El remitente puede ser desconocido, la línea de asunto puede tener contenido sin sentido, o el hacker puede disfrazar el email para que parezca de un remitente de confianza.

### 19. Armored Virus
Diseñado para **confundir o engañar a los sistemas antivirus** para evitar que detecten la fuente real de la infección. Muestra otras ubicaciones aunque esté en el propio sistema.

Técnicas que usa:
- **Anti-disassembly:** código especialmente diseñado para producir listados incorrectos en herramientas de análisis de desensamblado.
- **Anti-debugging:** asegura que el programa no está ejecutándose bajo un debugger. Ralentiza el reverse engineering.
- **Anti-heuristics:** código máquina para prevenir el análisis heurístico.
- **Anti-emulation:** evita el análisis dinámico mediante fingerprinting del entorno emulado.
- **Anti-goat:** usa reglas heurísticas para detectar posibles ficheros señuelo (goat files).

### 20. Add-on Virus
Adjunta su código al código host **sin hacer cambios al mismo** o reubica el código host para insertar su código al principio.

### 21. Intrusive Virus
**Sobreescribe el código host completamente o parcialmente** con código viral.

### 22. Direct Action / Transient Virus
Transfiere todos los controles del código host a donde reside en memoria. Selecciona el programa objetivo, lo modifica y corrompe. La vida del virus es **directamente proporcional a la vida de su host**: solo se ejecuta cuando se ejecuta el programa adjunto y termina cuando éste termina. Opera solo por un período corto y va directamente al disco a buscar programas para infectar.

### 23. TSR (Terminate and Stay Resident) Virus
Permanece permanentemente en la memoria de la máquina objetivo durante toda la sesión de trabajo, **incluso después de que el programa host se ejecute y termine**. Incorpora vectores de interrupción en su código para que cuando ocurra una interrupción, el vector dirija la ejecución al código TSR.

Pasos de infección:
1. Toma control del sistema.
2. Asigna una porción de memoria para su código.
3. Se transfiere y activa en la porción de memoria asignada.
4. Hookea el flujo de ejecución a sí mismo.
5. Empieza a replicarse para infectar ficheros.

---

## 5. Virus Hoaxes y Fake Antivirus

### Virus Hoaxes
Falsas alarmas sobre virus inexistentes. Pueden ser casi tan dañinas como los virus reales en términos de pérdida de productividad y ancho de banda cuando usuarios ingenuos las reenvían.

Características: mensajes de advertencia que se propagan rápidamente afirmando que cierto email no debe abrirse porque dañaría el sistema. En algunos casos, **los propios mensajes de advertencia contienen adjuntos con virus**.

Ejemplo: Google Critical Security Alert Scam (email falso que imita alertas reales de Google, engañando al usuario para hacer clic en links maliciosos).

### Fake Antivirus (Rogue Antivirus)
Forma de fraude en Internet basada en malware. Parece y funciona como un antivirus real. Se distribuye en banner ads, pop-ups, links en emails y resultados de motores de búsqueda. Al hacer clic, redirige a una página donde se pide comprar/suscribirse introduciendo datos de pago.

Efectos: infecta el sistema con malware, roba información sensible (contraseñas, datos bancarios, tarjetas de crédito), corrompe ficheros.

**Dato relevante para el examen:** según AV-Comparatives, **dos tercios de todas las aplicaciones antivirus presentes en el Android Play Store son falsas**.

Ejemplo: Antivirus 10 (simula escaneo del sistema, muestra amenazas falsas, crea alertas de sistema falsas y pop-ups de antivirus, incita a comprar la versión completa).

---

## 6. Ransomware

El ransomware es un tipo de malware que **restringe el acceso al sistema infectado o a los ficheros** y exige un pago de rescate online para eliminar las restricciones. Puede cifrar ficheros o simplemente bloquear el sistema mostrando mensajes de extorsión.

Se propaga como troyano via: adjuntos de email, sitios web hackeados, programas infectados, descargas desde sitios no confiables, vulnerabilidades en servicios de red.

**Mecanismos de engaño:** los mensajes aparentan ser de empresas o fuerzas del orden, acusando falsamente a la víctima de actividades ilegales (pornografía, software pirata) o de usar software sin licencia (ej. aviso falso de activación de Microsoft Office).

**Vectores psicológicos:** miedo, confianza, sorpresa y vergüenza.

Pasos de infección con ransomware:
1. Crear ransomware con herramientas como Chaos Ransomware Builder v4.
2. Transferir a la víctima via email o medios físicos.
3. La víctima descarga y abre el fichero malicioso → el ransomware cifra los ficheros del sistema.
4. Aparece una ventana con instrucciones para pagar el rescate.

Ejemplos relevantes:
- **Mallox:** ataca sistemas Windows via servidores MS-SQL vulnerables. Añade extensión `.mallox`. Nota de rescate: "RECOVERY INFORMATION.txt".
- **STOP/Djvu:** más de 600 variantes, usa RSA encryption, spread via spam emails y sitios de torrents piratas.
- **WannaCry/Petya** (puerto 445).

---

## 7. Computer Worms

Los gusanos son programas maliciosos independientes que **se replican, ejecutan y propagan por conexiones de red de forma autónoma sin intervención humana**. Son un subtipo de virus. No requieren un host para replicarse.

**Objetivos:** consumir recursos de computación causando sobrecarga en servidores y sistemas, instalar backdoors en computadoras infectadas convirtiéndolas en zombies para botnets.

Proceso de infección con gusano (6 pasos):
1. Crear el gusano con herramientas como Internet Worm Maker Thing o Batch Worm Generator.
2. Desplegar via phishing emails, webs maliciosas, comparticiones de red, USBs infectados. Usar packers/crypters (BitCrypter, H-Crypt) para evadir detección.
3. Al ejecutarse en el sistema, el gusano ejecuta su payload usando código de exploit y script automatizado.
4. El gusano escanea la red en busca de otros dispositivos vulnerables (puertos abiertos, vulnerabilidades conocidas).
5. Se copia a sí mismo en los dispositivos vulnerables encontrados.
6. Instala backdoors y altera configuraciones del sistema para permanecer activo y exfiltrar datos continuamente.

---

## 8. Virus vs Worm: Diferencias Clave 🔴

| Aspecto | Virus | Worm |
|---|---|---|
| Infección | Inserción en un fichero o programa ejecutable | Explota vulnerabilidades del OS o aplicación, se replica |
| Daño | Puede borrar o alterar ficheros | Generalmente no modifica programas almacenados; consume CPU y memoria |
| Funcionamiento | Altera el funcionamiento sin conocimiento del usuario | Consume ancho de banda, memoria y recursos, sobrecargando servidores |
| Propagación | No se propaga a otros ordenadores sin que se replique y envíe un fichero infectado | Puede replicarse y propagarse usando IRC, Outlook u otros programas de correo |
| Velocidad | Se propaga a tasa uniforme según programación | **Se propaga más rápido** que un virus |
| Eliminación | Difíciles de eliminar | **Más fáciles de eliminar** en comparación con un virus |

---

## 3. Exam Traps ⚠️

⚠️ **[Polymorphic vs Metamorphic: qué cambia]** — Polymorphic: **cifra el virus y cambia la rutina de descifrado** en cada infección (el virus en sí es siempre el mismo, solo varía cómo se descifra). Metamorphic: **reescribe completamente el código del virus** en cada infección sin necesitar cifrado. Metamorphic es más efectivo que polymorphic.

⚠️ **[Polymorphic: 3 componentes]** — Encrypted virus code + Decryptor routine + **Mutation engine**. El mutation engine es el componente que distingue al polymorphic virus.

⚠️ **[Stealth Virus: oculta tamaño]** — El stealth virus oculta el tamaño original del fichero o coloca una copia del fichero no infectado para mostrar al leer el fichero. No muestra aumento de tamaño en el listado de directorio.

⚠️ **[Boot Sector Virus: qué hace con el MBR]** — Mueve el MBR a otra ubicación del disco y copia su código en la ubicación original del MBR. Al arrancar, **primero se ejecuta el código del virus, luego el MBR original**. El examen puede invertir el orden.

⚠️ **[Companion Virus: COM antes que EXE]** — DOS ejecuta ficheros COM **antes** que EXE con el mismo nombre raíz. El companion virus crea un `.COM` con el mismo nombre que el `.EXE` objetivo. El virus se ejecuta primero, luego ejecuta el `.EXE` legítimo.

⚠️ **[Logic Bomb vs Time Bomb]** — La Logic Bomb se activa por un evento (lanzar una aplicación, visitar un sitio). Cuando el evento es una fecha/hora específica, se llama **Time Bomb**. El time bomb es un subtipo de logic bomb, no un concepto separado.

⚠️ **[Cluster Virus: una sola copia en disco]** — A pesar de afectar aparentemente a todos los programas, **solo existe una copia del virus en el disco**. El virus sobreescribe punteros en el directorio, no el fichero en sí.

⚠️ **[Encryption Virus: el escáner detecta el módulo de descifrado]** — El escáner de virus no puede detectar el encryption virus por sus firmas, pero **sí puede detectar el módulo de descifrado** (que permanece constante).

⚠️ **[Virus vs Worm: propagación autónoma]** — El virus necesita asistencia humana para propagarse a otras máquinas (enviar un fichero infectado). El gusano se propaga autónomamente por la red sin intervención humana. Esta es la diferencia fundamental.

⚠️ **[Worm: más fácil de eliminar que un virus]** — Contraintuitivo pero correcto: los gusanos son **más fáciles de eliminar** que los virus. Los virus están incrustados en ficheros del sistema; los gusanos son programas independientes.

⚠️ **[Fake Antivirus en Android Play Store]** — Según AV-Comparatives, **dos tercios** de todas las aplicaciones antivirus en el Android Play Store son falsas. Dato específico que puede aparecer en el examen.

⚠️ **[Ransomware: spread como Trojan]** — El ransomware normalmente se propaga **como un troyano**, no como un virus o gusano. Entra via adjuntos de email, webs hackeadas, programas infectados, etc.

⚠️ **[Multipartite vs Boot Sector]** — Boot Sector: ataca solo el boot sector. Multipartite: ataca simultáneamente boot sector **Y** ficheros ejecutables. Los sector viruses que también se propagan a través de ficheros infectados se llaman multipartite.

⚠️ **[TSR vs Direct Action]** — TSR: permanece en memoria **después** de que el programa host termine. Direct Action: solo existe mientras existe el host; su vida es proporcional a la del host.

---

## 4. Nemotécnicos

**23 tipos de virus (grupos conceptuales):**

Grupo BOOT: **System/Boot Sector, Multipartite** (ambos atacan sectores de arranque).

Grupo FILE: **File Virus, Companion, Cluster, Add-on, Intrusive, Shell, File Extension, FAT, Overwriting/Cavity**.

Grupo EVASIÓN: **Stealth, Armored, Encryption, Sparse Infector, Polymorphic, Metamorphic**.

Grupo COMPORTAMIENTO: **Macro, Web Scripting, Email, Logic Bomb, Direct Action/Transient, TSR**.

**Polymorphic vs Metamorphic:**
> **Poly = mismo virus, distinta rutina de descifrado**
> **Meta = virus completamente reescrito cada vez**

**Virus lifecycle (3 etapas):**
> **"Design → Replicate → Launch"**

**Diferencias Virus vs Worm (las más examinadas):**
> Worm = autónomo, sin host, más rápido, más fácil de eliminar, no modifica ficheros almacenados.

---

## 5. Flashcards

**Q:** ¿Qué diferencia fundamental hay entre virus y gusano?
**A:** El virus necesita asistencia humana para propagarse; el gusano se propaga autónomamente por la red sin intervención humana.

**Q:** ¿Cuál se propaga más rápido: virus o gusano?
**A:** El gusano se propaga más rápido.

**Q:** ¿Cuál es más fácil de eliminar: virus o gusano?
**A:** El gusano es más fácil de eliminar.

**Q:** ¿Qué hace un boot sector virus con el MBR?
**A:** Mueve el MBR a otra ubicación del disco y copia su código en la ubicación original del MBR. Al arrancar, el código del virus se ejecuta primero y luego pasa el control al MBR original.

**Q:** ¿Qué diferencia a un virus polymorphic de uno metamorphic?
**A:** Polymorphic: cifra el virus y cambia la rutina de descifrado en cada infección (el virus en sí no cambia). Metamorphic: reescribe completamente el código del virus en cada infección. Metamorphic es más efectivo.

**Q:** ¿Cuáles son los 3 componentes de un polymorphic virus?
**A:** Encrypted virus code, Decryptor routine, y Mutation engine.

**Q:** ¿Qué tipo de virus se activa en una fecha/hora específica?
**A:** Time Bomb (subtipo de Logic Bomb).

**Q:** ¿Qué virus crea un fichero .COM con el mismo nombre que el .EXE objetivo?
**A:** Companion / Camouflage Virus. DOS ejecuta COM antes que EXE.

**Q:** ¿Por qué el Encryption Virus no puede detectarse por firmas pero sí puede ser detectado?
**A:** Porque el módulo de descifrado permanece constante y el escáner puede detectarlo aunque no pueda detectar el cuerpo cifrado del virus.

**Q:** ¿Qué tipo de virus infecta ficheros sin cambiar el fichero ni plantar ficheros adicionales?
**A:** Cluster Virus. Sobreescribe punteros en la entrada de directorio.

**Q:** ¿Cuántas copias del virus existen en disco en un Cluster Virus?
**A:** Solo una, aunque parezca afectar a todos los programas.

**Q:** ¿Cómo oculta el Stealth Virus su presencia?
**A:** Ocultando el tamaño original del fichero, o colocando temporalmente una copia del fichero no infectado para mostrar al leer el fichero. No muestra aumento de tamaño en listados de directorio.

**Q:** ¿Qué tipo de virus permanece en memoria incluso después de que el programa host termina?
**A:** TSR (Terminate and Stay Resident) Virus.

**Q:** ¿Qué tipo de virus solo existe mientras existe su programa host?
**A:** Direct Action / Transient Virus.

**Q:** ¿Cómo se propaga principalmente el ransomware?
**A:** Como un troyano, via adjuntos de email, sitios web hackeados, programas infectados y descargas desde sitios no confiables.

**Q:** ¿Qué porcentaje de apps antivirus en el Android Play Store son falsas según AV-Comparatives?
**A:** Dos tercios (2/3).

**Q:** ¿Qué tipo de virus infecta simultáneamente boot sector y ficheros ejecutables?
**A:** Multipartite Virus (también llamado hybrid virus).

**Q:** ¿Qué técnica de transformación usa el Metamorphic Virus para evitar detección por patrones?
**A:** Se reescribe completamente a sí mismo en cada infección usando un metamorphic engine. Pasos: insertar dead code, reshape expressions, reordenar instrucciones, modificar nombres de variables, cifrar código, modificar la estructura de control del programa.

---

## 6. Confusión frecuente

**Polymorphic vs Metamorphic** → Polymorphic: el cuerpo del virus es siempre el mismo, solo cambia la rutina de descifrado. La detección del módulo de descifrado puede revelar el polymorphic. Metamorphic: el código del virus completo se reescribe, sin depender de cifrado. Metamorphic es más sofisticado y difícil de detectar.

**Boot Sector vs Multipartite** → Boot Sector: solo ataca el MBR/boot record. Multipartite: ataca boot sector Y ficheros ejecutables simultáneamente. Los sector viruses que también se propagan por ficheros infectados son multipartite.

**Stealth vs Armored** → Stealth: oculta su presencia ocultando el tamaño del fichero o intercambiando ficheros infectados/limpios al leer. Armored: confunde las herramientas de análisis (anti-disassembly, anti-debugging, anti-emulation, anti-heuristics).

**Logic Bomb vs Time Bomb** → Time Bomb es un subtipo de Logic Bomb donde el evento trigger es una fecha/hora específica. Toda Time Bomb es una Logic Bomb, pero no toda Logic Bomb es una Time Bomb.

**Cluster Virus vs File Virus** → File Virus: inserta su código directamente en el fichero. Cluster Virus: no modifica el fichero, solo sobreescribe punteros en el directorio para redirigir la ejecución.

**TSR vs Direct Action** → TSR: permanece en memoria después de que el host termina, hookea interrupciones. Direct Action: vive y muere con su host, opera solo mientras el host ejecuta.

**Encryption Virus vs Polymorphic Virus** → Encryption Virus: usa cifrado para ocultar su código del escáner; el módulo de descifrado es constante. Polymorphic: usa mutation engine para cambiar tanto el código cifrado como la rutina de descifrado en cada infección. El polymorphic es una evolución del encryption virus.

**Virus vs Worm: propagación** → Virus: no puede propagarse a otras máquinas sin que un fichero infectado sea replicado y enviado manualmente. Worm: se propaga autónomamente explotando vulnerabilidades en el OS o aplicaciones, sin intervención humana.

**Ransomware vs Destructive Trojan** → Ransomware: restringe el acceso o cifra ficheros y exige rescate; no necesariamente destruye datos permanentemente. Destructive Trojan: borra aleatoriamente ficheros, carpetas y entradas de registro sin posibilidad de recuperación ni demanda de rescate.

**Companion Virus vs File Extension Virus** → Companion: crea un fichero COM con el mismo nombre que el EXE objetivo (explota el orden COM→EXE→BAT de DOS). File Extension: cambia la extensión del fichero para engañar al usuario sobre el tipo de fichero (BAD.TXT.VBS se muestra como BAD.TXT).

---

## 7. Preguntas de Práctica — Formato CEH

**P1.** Un antivirus detecta un virus por su rutina de descifrado, pero no por el código del cuerpo del virus en sí. El cuerpo está siempre cifrado con diferentes claves. ¿Qué tipo de virus describe esto?

A) Metamorphic Virus  
B) Polymorphic Virus  
C) Stealth Virus  
D) Armored Virus  

**Respuesta correcta: B**
Un **Polymorphic Virus** cifra su cuerpo con diferentes claves en cada infección pero mantiene una **rutina de descifrado constante**. El antivirus puede detectarlo por esa rutina de descifrado. Un Metamorphic virus reescribe completamente su código (sin cifrado fijo) — es más difícil de detectar porque no tiene componente constante.

---

**P2.** Un worm se propaga a través de una red corporativa explotando una vulnerabilidad en el protocolo de red, sin necesidad de interacción del usuario. ¿En qué se diferencia fundamentalmente de un virus?

A) Los worms son más pequeños que los virus  
B) Los worms se replican y propagan de forma autónoma por la red; los virus requieren un host/fichero para propagarse  
C) Los worms solo afectan a Windows; los virus son multiplataforma  
D) Los worms cifran los ficheros; los virus los corrompen  

**Respuesta correcta: B**
La diferencia fundamental es que los **worms se replican y propagan de forma autónoma** a través de la red sin necesitar un host o fichero (son standalone). Los **virus necesitan un programa host o fichero ejecutable** para propagarse y requieren que el usuario ejecute el fichero infectado.

---

**P3.** Un malware no modifica el código de los ficheros ejecutables, pero modifica las entradas del directorio del disco para apuntar al código malicioso. ¿Qué tipo de virus es?

A) File Virus  
B) Boot Sector Virus  
C) Cluster Virus  
D) Stealth Virus  

**Respuesta correcta: C**
Un **Cluster Virus** (también llamado File System/Directory Virus) **no modifica directamente los ficheros ejecutables**, sino que sobreescribe los **punteros en el directorio del disco** para redirigir la ejecución al código malicioso. El fichero original parece intacto pero la ejecución es desviada.

---

**P4.** Un virus infecta el MBR del disco duro y el sector de arranque de los ficheros ejecutables simultáneamente. ¿Qué tipo de virus es?

A) Boot Sector Virus  
B) Multipartite Virus  
C) Cluster Virus  
D) Macro Virus  

**Respuesta correcta: B**
Un **Multipartite Virus** infecta **tanto el boot sector/MBR como los ficheros ejecutables** simultáneamente. Un Boot Sector Virus solo ataca el MBR/sector de arranque. La capacidad de atacar múltiples vectores a la vez lo hace especialmente difícil de erradicar — si limpias los ficheros pero no el boot sector, se reinfecta.

---

**P5.** Una organización recibe emails masivos advirtiendo de un "virus extremadamente peligroso que borrará todos los datos". El email pide reenviar el aviso a todos los contactos. ¿Qué es esto?

A) Spear-phishing con malware adjunto  
B) Virus Hoax  
C) Ransomware  
D) Worm de email  

**Respuesta correcta: B**
Un **Virus Hoax** es un aviso falso sobre un virus inexistente que se propaga por email, creando pánico y consumiendo tiempo/recursos de los receptores. No contiene malware real. Se propaga por la acción del propio usuario que lo reenvía (ingeniería social), no por código malicioso autónomo.

---

**P6.** Un ransomware cifra todos los ficheros de una red corporativa usando AES-256 y solicita pago en criptomoneda. ¿Qué componente del malware es responsable específicamente del cifrado de los ficheros?

A) Dropper  
B) Downloader  
C) Payload  
D) Crypter  

**Respuesta correcta: C**
El **Payload** es el componente que ejecuta la acción maliciosa principal — en este caso, el cifrado de ficheros. El Crypter cifra el malware en sí mismo para evadir detección (no los datos de la víctima). El Dropper instala el malware. El Downloader descarga el malware.

---

**P7.** Un virus infecta ficheros ejecutables (.exe, .com) pero permanece en memoria RAM después de que el programa host termina, hookando interrupciones del sistema para infectar nuevos ficheros cuando se ejecutan. ¿Qué tipo de virus es?

A) Direct Action Virus  
B) TSR (Terminate and Stay Resident) Virus  
C) Stealth Virus  
D) Sparse Infector Virus  

**Respuesta correcta: B**
Un **TSR (Terminate and Stay Resident) Virus** permanece en **memoria RAM después de que el programa host termina**, hookando interrupciones del sistema para infectar programas cuando se ejecutan. Un Direct Action Virus solo actúa mientras el host ejecuta y "muere" con él.

---

**P8.** Un ransomware se entrega a través de un email con un archivo adjunto. Cuando la víctima lo abre, el ransomware descarga el módulo de cifrado desde un servidor externo. ¿Qué componente inicial del malware describe el adjunto del email?

A) Payload  
B) Dropper  
C) Downloader  
D) Injector  

**Respuesta correcta: C**
El adjunto del email es un **Downloader**: contiene solo el código para descargar el componente real del malware (módulo de cifrado) desde un servidor externo después de la ejecución. Es más sigiloso que un Dropper (que lleva el malware consigo) porque el ejecutable inicial es muy pequeño y puede evadir análisis estático.
