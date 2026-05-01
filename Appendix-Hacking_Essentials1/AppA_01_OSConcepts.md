# AppA_01_OSConcepts.md
# CEH v13 — Appendix A: Operating System Concepts

---

## 1. Conceptos y definiciones

### Windows OS

Windows, desarrollado por Microsoft Corporation, domina el entorno corporativo y gubernamental. Su evolución sigue dos líneas arquitectónicas distintas que el examen explota:

- **Línea MS-DOS/9x**: sistemas heredados sin soporte de memoria protegida real (MS-DOS 1.0 → Windows ME). Relevante solo históricamente.
- **Línea NT Kernel**: arquitectura moderna con separación estricta User Mode / Kernel Mode. Ramificada en versiones para PC (NT 3.1 → Windows 11) y para servidor (Server 2003 → Server 2022).

La arquitectura Windows se divide en **dos modos de operación del procesador**:

| Modo | Componentes | Acceso |
|---|---|---|
| **User Mode** | Subsistemas integrales (Workstation Service, Server Service, Security), subsistemas de entorno (Win32, POSIX, OS/2), aplicaciones | **Limitado** a recursos del sistema |
| **Kernel Mode** | HAL, Microkernel, Executive Services (I/O Manager, VMM, Security Reference Monitor, Process Manager, IPC Manager, PnP Manager, Power Manager, Window Manager/GDI), Object Manager, drivers | **Irrestricto**: acceso directo a memoria y dispositivos hardware |

La lógica del diseño: el User Mode actúa como barrera de protección. Si una aplicación falla, no colapsa el sistema. Los drivers en Kernel Mode, en cambio, tienen acceso total —de ahí que un driver malicioso sea un vector de ataque crítico.

**Executive Services** son el puente entre User Mode y hardware: gestionan E/S, memoria virtual, procesos, seguridad de referencia y comunicación entre procesos (IPC). El **HAL (Hardware Abstraction Layer)** aísla al kernel de las particularidades del hardware físico, permitiendo portabilidad entre plataformas.

---

### UNIX OS

UNIX (desarrollado en los años 60) fue diseñado para ser portable entre cualquier tipo de sistema de cómputo. Arquitectura de **tres componentes**:

- **Kernel**: el "cerebro" del SO. Asigna tiempo de CPU y memoria a los programas, gestiona el sistema de ficheros y atiende las llamadas al sistema (system calls).
- **Shell**: interfaz entre usuario y kernel. Recibe input del usuario, lo envía al kernel y devuelve el output.
- **Programs/Processes**: procesos en ejecución sobre la máquina.

La jerarquía de directorios UNIX sigue una **estructura arbórea invertida** cuya raíz es `/` (root). Todos los ficheros, sin excepción, cuelgan de esta raíz.

---

### Linux OS

Linux es open source, derivado conceptualmente de UNIX, ampliamente usado en empresas y organismos gubernamentales. Sus componentes:

| Componente | Función |
|---|---|
| **Hardware** | Dispositivos físicos (monitor, RAM, HDD, CPU) |
| **Kernel** | Control total sobre recursos del sistema |
| **Shell** | Interfaz usuario → kernel → output |
| **Applications/Utilities** | Programas lanzados desde la shell; proveen funcionalidad al usuario |
| **System Libraries** | Funciones especiales que **no requieren derechos de acceso al kernel** para implementar funcionalidad del SO |
| **Daemons** | Servicios en background (impresión, scheduling, etc.) |
| **Graphical Server** | Subsistema de visualización gráfica, denominado **X** |

Arquitectura en tres capas: **User Space** (aplicaciones/herramientas) → **Linux Kernel** (gestión de procesos, memoria, ficheros, drivers, red) → **Hardware** (CPU, RAM, disco, red).

**Características clave de Linux** 🔴:
- **Portabilidad**: el kernel y las apps se instalan en distintas plataformas hardware.
- **Open Source**: código fuente libre, desarrollo comunitario.
- **Multiuser**: múltiples usuarios acceden a recursos (RAM, memoria) simultáneamente.
- **Multiprogramming**: múltiples aplicaciones en ejecución simultánea.
- **Hierarchical File System**: estructura jerárquica estándar para ficheros de usuario y sistema.
- **Shell**: programa intérprete especial para ejecutar programas/aplicaciones.
- **Security**: autenticación, control de acceso a ficheros con contraseñas, cifrado de datos.

---

### macOS

Desarrollado por Apple Inc., es **closed-source** (propietario). Es el SO primario de los Mac. Soporta **multitarea apropiativa (preemptive multitasking)** y **protección de memoria**. Arquitectura en **5 capas** (de más alta a más baja):

| Capa | Contenido clave |
|---|---|
| **Cocoa Application** | AppKit — tecnologías para la UI de apps |
| **Media** | AV Foundation, Core Animation, Core Audio, Core Image, Core Text, OpenAL, OpenGL, Quartz |
| **Core Services** | Address Book, Core Data, Core Foundation, Foundation, Quick Look, Social, Security, WebKit |
| **Core OS** | Accelerate, Directory Services, Disk Arbitration, OpenCL, System Configuration |
| **Kernel and Device Drivers** | BSD, File System, Mach, Networking — soporte para ficheros, red, seguridad, IPC, drivers, lenguajes de programación |

La capa **Kernel and Device Drivers** es la más baja y contiene el microkernel **Mach** (base de macOS) junto a componentes **BSD**.

---

## 2. Exam Traps ⚠️

⚠️ **[Windows User Mode vs Kernel Mode]**
El examen pregunta qué componente tiene "acceso irrestricto a memoria y dispositivos". La respuesta es **Kernel Mode**, no User Mode. User Mode tiene acceso *limitado*. Los subsistemas de entorno (Win32, POSIX, OS/2) están en **User Mode**, no en Kernel Mode.

⚠️ **[HAL — dónde vive]**
El HAL (Hardware Abstraction Layer) está en **Kernel Mode**, no en User Mode. El examen puede presentarlo como componente de alto nivel para despistar.

⚠️ **[Executive Services]**
Son parte de **Kernel Mode**. El examen puede intentar ubicarlos en User Mode por su nombre ("servicios"). Incluyen el VMM, I/O Manager, Security Reference Monitor, Process Manager, IPC Manager.

⚠️ **[Security Reference Monitor]**
Es un componente de **Executive Services en Kernel Mode**. No confundir con el subsistema "Security" de los Integral Subsystems en User Mode. Hay uno en cada modo.

⚠️ **[Linux: System Libraries]**
La clave definitoria es que **no requieren derechos de acceso al kernel** para implementar funcionalidad del SO. Si el examen pregunta qué componente Linux actúa sin permisos de kernel, la respuesta son las System Libraries.

⚠️ **[Linux: Graphical Server = "X"]**
El subsistema gráfico de Linux se llama **X** (o X Window System). No es "GUI" ni "Desktop". El nombre exacto importa.

⚠️ **[macOS: open source vs closed source]**
macOS es **closed-source** (propietario de Apple). Linux es open source. El examen puede invertirlos en una pregunta de características.

⚠️ **[macOS: número de capas y orden]**
Son **5 capas**. El orden correcto de arriba a abajo: Cocoa Application → Media → Core Services → Core OS → Kernel and Device Drivers. El examen puede alterar el orden o cambiar nombres de capas.

⚠️ **[macOS: Mach]**
El microkernel de macOS se llama **Mach**, ubicado en la capa **Kernel and Device Drivers** (la más baja). No confundir con "Kernel Mode" de Windows ni con el kernel Linux.

⚠️ **[UNIX: shell como interfaz]**
La shell UNIX es la **interfaz entre usuario y kernel**, no entre aplicaciones y hardware. La confusión típica es ubicarla como capa de hardware o como componente del kernel.

⚠️ **[Directorios UNIX: root = "/"]**
La raíz del sistema de ficheros UNIX se denota con `/`, no con `C:\` (eso es Windows). El examen puede mezclar notaciones.

---

## 3. Nemotécnicos

### Windows: capas Kernel Mode (de abajo a arriba)
**"Hay Más KDE Grandes Esperando"**
- **H**AL
- **M**icrokernel
- **K**ernel Mode Drivers
- **E**xecutive Services (+ Object Manager)
- → (arriba: User Mode)

### Windows Executive Services — componentes 🔴
**"I-S-I-V-P-P-P-W"** → "In Secure IPC, VMM Processes Plug Power Windows"
- **I**/**O** Manager
- **S**ecurity Reference Monitor
- **I**PC Manager
- **V**MM (Virtual Memory Manager)
- **P**rocess Manager
- **P**nP Manager
- **P**ower Manager
- **W**indow Manager / GDI

### Linux: 7 características (POMUHS + S)
**"Para Obtener Máxima Utilidad, Hay Seguridad Siempre"**
- **P**ortabilidad
- **O**pen Source
- **M**ultiuser
- **M**ultiprogramming → (doble M: Multi-Multi)
- **H**ierarchical File System
- **S**hell
- **S**ecurity

### macOS: 5 capas (arriba → abajo)
**"Cocina Merece Comer Con Kilo"**
- **Co**coa Application
- **M**edia
- **C**ore Services
- **C**ore OS
- **K**ernel and Device Drivers

### UNIX: 3 componentes
**"K-S-P"** → Kernel, Shell, Programs

---

## 4. Flashcards

**Q:** ¿Qué modo del procesador Windows tiene acceso irrestricto a memoria del sistema y dispositivos externos?
**A:** Kernel Mode.

---

**Q:** ¿En qué modo de Windows se ejecutan Win32, POSIX y OS/2?
**A:** User Mode (son subsistemas de entorno dentro de User Mode).

---

**Q:** ¿Qué componente de Windows aísla al kernel de las particularidades del hardware físico?
**A:** HAL (Hardware Abstraction Layer), en Kernel Mode.

---

**Q:** Nombra los 8 componentes de Executive Services en Windows.
**A:** I/O Manager, Security Reference Monitor, IPC Manager, VMM, Process Manager, PnP Manager, Power Manager, Window Manager/GDI.

---

**Q:** ¿Cuáles son los 3 componentes principales de la arquitectura UNIX?
**A:** Kernel, Shell, Programs (procesos).

---

**Q:** En UNIX, ¿qué componente asigna tiempo de CPU y memoria a los programas y gestiona las llamadas al sistema?
**A:** El Kernel.

---

**Q:** ¿Con qué símbolo se denota el directorio raíz en UNIX/Linux?
**A:** `/` (slash).

---

**Q:** ¿Qué componente Linux gestiona servicios en background como impresión o scheduling?
**A:** Los Daemons.

---

**Q:** ¿Cómo se denomina el subsistema gráfico de Linux?
**A:** X (Graphical Server / X Window System).

---

**Q:** ¿Qué componente Linux implementa funcionalidad del SO sin requerir derechos de acceso al kernel?
**A:** System Libraries.

---

**Q:** ¿Es macOS open source o closed source?
**A:** Closed source (propietario de Apple).

---

**Q:** ¿Qué característica de macOS garantiza que un proceso no interfiera con la memoria de otro?
**A:** Memory protection (protección de memoria). Junto a preemptive multitasking.

---

**Q:** ¿Cuántas capas tiene la arquitectura de macOS? Nómbralas de arriba a abajo.
**A:** 5 capas: Cocoa Application → Media → Core Services → Core OS → Kernel and Device Drivers.

---

**Q:** ¿En qué capa de macOS se encuentran BSD, Mach, File System y Networking?
**A:** Kernel and Device Drivers (la capa más baja).

---

**Q:** ¿Qué microkernel usa macOS como base?
**A:** Mach.

---

**Q:** ¿Qué comando Windows muestra estadísticas de protocolo y conexiones TCP/IP actuales, incluyendo NetBIOS?
**A:** `nbtstat`

---

**Q:** ¿Qué diferencia hay entre `netstat` y `nbtstat` en Windows?
**A:** `netstat` muestra todas las conexiones de red activas y puertos. `nbtstat` muestra estadísticas de protocolo y conexiones TCP/IP actuales (foco en NetBIOS).

---

**Q:** ¿Qué comando Windows muestra información de configuración DNS de la infraestructura?
**A:** `nslookup`

---

**Q:** ¿Qué comando Linux/UNIX busca una cadena de texto dentro de un fichero?
**A:** `grep string file`

---

**Q:** ¿Qué comando Linux/UNIX compara dos ficheros y muestra las diferencias?
**A:** `diff file1 file2`

---

## 5. Confusión frecuente

### `netstat` vs `nbtstat` (Windows)
- **netstat**: muestra **todas** las conexiones de red activas y puertos del sistema.
- **nbtstat**: muestra estadísticas de protocolo y conexiones TCP/IP (orientado a NetBIOS).
- **Criterio de decisión**: si la pregunta menciona NetBIOS o estadísticas de protocolo → `nbtstat`. Si es visión general de conexiones y puertos → `netstat`.

---

### Windows User Mode vs Kernel Mode
- **User Mode**: subsistemas de entorno (Win32, POSIX, OS/2), subsistemas integrales, aplicaciones. Acceso **limitado**.
- **Kernel Mode**: HAL, Microkernel, Executive Services, drivers. Acceso **irrestricto**.
- **Criterio de decisión**: si el concepto tiene acceso directo a hardware/memoria → Kernel Mode. Si es una aplicación o subsistema de compatibilidad → User Mode.

---

### Security Reference Monitor vs subsistema Security (User Mode)
- **Security Reference Monitor**: componente de **Executive Services en Kernel Mode**. Implementa la política de control de acceso.
- **Security (Integral Subsystem, User Mode)**: subsistema en User Mode que gestiona autenticación y políticas de seguridad a nivel de usuario.
- **Criterio de decisión**: si la pregunta habla de control de acceso a nivel de kernel/objetos del sistema → Security Reference Monitor. Si habla de autenticación de usuario → Security Subsystem (User Mode).

---

### Linux Daemons vs Shell
- **Shell**: interfaz interactiva entre usuario y kernel. El usuario escribe comandos, la shell los envía al kernel.
- **Daemons**: procesos en background sin interacción directa del usuario. Realizan tareas como printing o scheduling.
- **Criterio de decisión**: interacción → Shell. Tarea autónoma en background → Daemon.

---

### macOS Core Services vs Core OS
- **Core Services**: servicios fundamentales de alto nivel (Core Data, Foundation, WebKit, Security, Quick Look). Orientado a abstracciones de datos y servicios de app.
- **Core OS**: interfaces de bajo nivel relacionadas con hardware y red (Accelerate, OpenCL, Directory Services, System Configuration).
- **Criterio de decisión**: si la pregunta menciona hardware o red a bajo nivel → Core OS. Si menciona servicios de datos/app → Core Services.

---

### UNIX Shell vs Linux Shell
Funcionalmente idénticos: ambos son la interfaz entre usuario y kernel. La distinción del examen es de contexto (qué SO se describe), no de función. Si la pregunta es sobre arquitectura UNIX de 3 componentes → Kernel/Shell/Programs. Si es sobre Linux → los 7 componentes incluyen Shell como uno más entre Hardware, Kernel, Applications, System Libraries, Daemons y Graphical Server.
