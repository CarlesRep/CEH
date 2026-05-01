# AppA_02_FileSystems.md
# CEH v13 — Appendix A: File Systems

---

## 1. Conceptos y definiciones

### Qué es un sistema de ficheros

Un file system es un **conjunto de tipos de datos** que gestiona almacenamiento, categorización jerárquica, navegación, acceso y recuperación de datos. Proporciona una representación lógica en jerarquía de ficheros y directorios, especifica el formato de las rutas y organiza el todo en **directorios con estructura de árbol** que requieren autorización de acceso.

Sistemas de ficheros principales: FAT, NTFS, HFS, HFS+, APFS, Ext2, Ext3, Ext4.

---

### Tipos de sistemas de ficheros 🔴

| Tipo | Característica definitoria |
|---|---|
| **Disk File System** | Almacenamiento/recuperación en dispositivo de almacenamiento (disco duro) |
| **Shared Disk File System** | Múltiples servidores acceden al mismo **subsistema de disco externo** |
| **Network File System** | Acceso a ficheros en otros equipos conectados por **red** |
| **Database File System** | Los ficheros se identifican por sus **características/metadatos** (tipo, tema, autor...) en lugar de (o además de) jerarquía |
| **Flash File System** | Diseñado para almacenamiento/recuperación en **dispositivos de memoria flash** |
| **Tape File System** | Almacenamiento/recuperación en cinta en **formato autodescriptivo** |
| **Special Purpose File System** | Ficheros organizados dinámicamente por software para **comunicación entre procesos o espacio temporal** |

---

### Windows: FAT (File Allocation Table) 🔴

FAT fue el **primer sistema de ficheros usado con Windows/DOS**. La tabla de asignación se ubica al **principio del volumen**. Las tres versiones se diferencian por el **tamaño de las entradas en la estructura FAT**:

| Versión | Bytes por clúster | Límite de clústeres |
|---|---|---|
| **FAT12** | 1,5 bytes | < 4.087 clústeres |
| **FAT16** | 2 bytes | 4.087 – 65.526 clústeres (inclusive) |
| **FAT32** | 4 bytes | 65.526 – 268.435.456 clústeres (inclusive) |

**FAT32** — extensión de FAT que soporta unidades de hasta **2 TB**. Usa clústeres pequeños para mayor eficiencia. Crea **copias de seguridad de la tabla FAT** en lugar de usar la copia por defecto.

**MBR de FAT32** — estructura del Master Boot Record:

| Offset | Descripción | Tamaño |
|---|---|---|
| 000h | Código ejecutable (arranca el equipo) | 446 bytes |
| 1BEh | 1ª entrada de partición | 16 bytes |
| 1CEh | 2ª entrada de partición | 16 bytes |
| 1DEh | 3ª entrada de partición | 16 bytes |
| 1EEh | 4ª entrada de partición | 16 bytes |
| 1FEh | Firma del registro de arranque | 2 bytes |

---

### Windows: NTFS (New Technology File System) 🔴

NTFS es el **sistema de ficheros estándar de Windows NT** y todos sus descendientes (XP, Vista, 7, 8.1, 10, 11, Server 2003/2008/2012/2016/2019/2022). Por defecto desde **Windows NT 3.1**.

Mejoras sobre FAT: soporte mejorado para **metadatos**, estructuras de datos avanzadas (rendimiento, fiabilidad, uso de disco), **ACLs (Access Control Lists)** de seguridad y **journaling** (registro de cambios para recuperación).

**Archivos de sistema NTFS** 🔴 — todos con prefijo `$`:

| Fichero | Función |
|---|---|
| `$mft` | Registro para **cada fichero** del volumen (Master File Table) |
| `$mftmirr` | **Espejo del MFT** para recuperación de ficheros |
| `$logfile` | Usado para **recuperación** (journaling) |
| `$boot` | Contiene el **bootstrap** del volumen |
| `$bitmap` | **Bitmap** de todo el volumen |
| `$badclus` | Contiene todos los **clústeres defectuosos** |
| `$attrdef` | Definiciones de atributos del sistema y de usuario |
| `$quota` | Cuota de disco por usuario |
| `$upcase` | Convierte caracteres a **Unicode en mayúsculas** |
| `$volume` | Nombre y número de versión del volumen |

**Arquitectura NTFS**: Hard Disk → MBR → Boot Sector → Ntldr → NTFS.sys → Ntoskrnl.exe → OS. Todo en **Kernel Mode**, salvo la aplicación final en User Mode.

---

### Windows: EFS (Encrypting File System)

EFS se introdujo en **NTFS versión 3.0**. Proporciona cifrado a nivel de sistema de ficheros. Principios de funcionamiento:

- **Transparencia**: el propietario del fichero no necesita descifrarlo para acceder o modificarlo — el descifrado es transparente.
- Cuando el usuario termina con el fichero, la **política de cifrado se restaura automáticamente**.
- Los usuarios no autorizados reciben **denegación de acceso** al intentar abrir el fichero cifrado.
- La activación requiere establecer los **atributos de cifrado** sobre ficheros y carpetas.

**Estructura interna de un fichero EFS**:
- Header
- File Encryption Key cifrada con la **clave pública del propietario** (Data Decryption Field)
- File Encryption Key cifrada con la **clave pública del agente de recuperación 1** (Data Recovery Field)
- File Encryption Key cifrada con la **clave pública del agente de recuperación 2** (opcional)
- Datos cifrados

**Componentes EFS**:
- *User Mode*: Application → Win32 Layer → EFS Service → CryptoAPI → RSA Base Provider
- *Kernel Mode*: I/O Manager → EFS Driver y FSRTL → NTFS → Hard Disk
- Comunicación entre modos: **LPC (Local Procedure Call)** para toda la gestión de claves.

---

### Windows: Sparse Files

Permiten **ahorrar espacio en disco** asignando clústeres de disco duro **solo para datos significativos (no cero)**. Los datos no definidos (ceros) se representan como **espacio no asignado** en disco.

Ejemplo cuantitativo del libro:
- Sin atributo sparse: 17 GB usados (7 GB reales + 10 GB de ceros).
- Con atributo sparse: 7 GB usados (solo los datos significativos).

---

### Linux: Arquitectura del sistema de ficheros

**User Space**: aplicaciones de usuario + GNU C Library.
**Kernel Space**: System call interface → Inode cache + Virtual file system + Directory cache → Individual file systems → Buffer cache → Device drivers.

El **Virtual File System (VFS)** actúa como capa de abstracción: permite que las aplicaciones interactúen con cualquier sistema de ficheros concreto de forma uniforme.

---

### Linux: FHS (Filesystem Hierarchy Standard) 🔴

Define la estructura de directorios en Linux y sistemas tipo Unix. Todos los ficheros y directorios cuelgan de la raíz `/`.

| Directorio | Contenido |
|---|---|
| `/bin` | Binarios esenciales (cat, ls, cp) |
| `/boot` | Ficheros estáticos del cargador de arranque (kernels, initrd) |
| `/dev` | Ficheros de dispositivos esenciales (ej. /dev/null) |
| `/etc` | Ficheros de configuración específicos del host |
| `/home` | Directorios home de usuarios |
| `/lib` | Librerías esenciales para `/bin` y `/sbin` |
| `/media` | Puntos de montaje para medios extraíbles |
| `/mnt` | Sistemas de ficheros montados temporalmente |
| `/opt` | Paquetes de software adicionales (add-on) |
| `/root` | Directorio home del usuario root |
| `/proc` | Sistema de ficheros virtual: info de procesos y kernel |
| `/run` | Información sobre procesos en ejecución (daemons, usuarios activos) |
| `/sbin` | Binarios necesarios para el funcionamiento del sistema |
| `/srv` | Datos específicos del sitio para servicios del sistema |
| `/tmp` | Ficheros temporales |
| `/usr` | Jerarquía secundaria para datos de usuario de solo lectura |
| `/var` | Datos variables (logs, spool files, etc.) |
| `/sys` | Información sobre dispositivos conectados |

---

### Linux: Familia Extended File System 🔴

**EXT (Extended)** — primer FS de Linux. Superó limitaciones de Minix (64 MB de partición, nombres cortos). Límites: partición máxima **2 GB**, nombre de fichero máximo **255 caracteres**. Sin timestamps separados de acceso/modificación. Sustituido por EXT2.

**EXT2** — usa algoritmos mejorados (más velocidad). Mantiene timestamps adicionales. El superbloque registra estado del FS (clean/dirty). **No es journaling** → riesgo de corrupción al escribir.

**EXT3** — versión **journaling** de EXT2. Mismas utilidades de mantenimiento (fsck). Conversión desde EXT2: `#/sbin/tune2fs -j`. Ventajas sobre EXT2:
- **Data Integrity**: mayor integridad en apagados abruptos.
- **Speed**: mayor throughput por el journaling.
- **Easy Transition**: migración directa desde EXT2 sin reformatear.

**EXT4** — reemplaza a EXT3. Requiere **Linux Kernel v2.6.19+**. Características clave:

| Característica | Detalle |
|---|---|
| Tamaño máximo de fichero | **16 TB** |
| Tamaño máximo del FS | **1 EB (exabyte)** |
| Extents | Reemplaza el esquema de block mapping de EXT2/3 |
| Delayed allocation | Asigna bloques más grandes de una vez → menos fragmentación |
| Multi-block allocation | Asigna ficheros de forma contigua en disco |
| Journal checksumming | Checksums en el journal → mayor fiabilidad |
| Persistent preallocation | Preasigna espacio en disco para un fichero |
| Improved timestamps | Timestamps en **nanosegundos** |
| Backward compatibility | Permite montar EXT3 y EXT2 como EXT4 |

---

### macOS: Sistemas de ficheros

| Sistema | Descripción |
|---|---|
| **HFS** | Desarrollado por Apple para soportar macOS en Mac |
| **HFS+** | Sucesor de HFS. **Sistema de ficheros primario en Macintosh** |
| **UFS** | Derivado del **Berkeley Fast File System (FFS)**, originado en Bell Laboratories. Usado en FreeBSD, NetBSD, OpenBSD, NeXTStep, Solaris. Actúa como **sustituto de HFS en macOS** |

---

## 2. Exam Traps ⚠️

⚠️ **[FAT — bytes por clúster]**
FAT12 = 1,5 bytes; FAT16 = 2 bytes; FAT32 = 4 bytes. El examen puede dar los valores invertidos o mezclados. El patrón es progresivo: más bits → más bytes por entrada → más clústeres soportados.

⚠️ **[FAT32 — capacidad máxima]**
FAT32 soporta unidades de hasta **2 TB**. No confundir con los ~4 GB de tamaño máximo de fichero individual de FAT32 (dato no en el chunk pero frecuente en examen). El dato del libro es el tamaño de **unidad**: 2 TB.

⚠️ **[EFS — introducido en NTFS 3.0]**
El examen puede preguntar en qué versión de NTFS se introdujo EFS: **versión 3.0**. No es NTFS 4.0 ni NTFS 5.0.

⚠️ **[EFS — transparencia para el propietario]**
EFS es transparente para el propietario: **no necesita descifrarlo para usarlo**. La pregunta trampa presenta "el usuario debe descifrar antes de editar" como opción correcta. Es falso.

⚠️ **[EFS — claves en el fichero]**
La File Encryption Key se cifra con la **clave pública** del propietario (y de los agentes de recuperación), no con la privada. El cifrado asimétrico usa pública para cifrar, privada para descifrar.

⚠️ **[Sparse Files — qué se asigna]**
Solo se asigna clúster de disco para datos **no cero** (significativos). Los ceros no ocupan espacio real. El examen puede presentar "se comprimen los datos" como respuesta alternativa incorrecta.

⚠️ **[EXT — primer FS Linux vs EXT2]**
EXT (sin número) fue el primero. EXT2 lo reemplazó. El examen puede preguntar cuál fue el primer FS de Linux: **EXT**, no EXT2.

⚠️ **[EXT2 — no es journaling]**
EXT2 **no tiene journaling**. EXT3 sí. Si el examen pregunta cuál es journaling entre EXT2 y EXT3, la respuesta es EXT3.

⚠️ **[EXT4 — límites de tamaño]**
Fichero individual: **16 TB**. Sistema de ficheros completo: **1 EB**. El examen puede invertirlos o proponer cifras incorrectas como "1 PB" o "2 TB".

⚠️ **[EXT4 — kernel mínimo]**
Requiere **Linux Kernel v2.6.19** o superior. Dato numérico concreto que el examen puede preguntar directamente.

⚠️ **[Comando conversión EXT2 → EXT3]**
`/sbin/tune2fs -j`. El examen puede presentar variantes incorrectas como `tune2fs -e3` o `fsck -j`. La flag es **-j** (de journaling).

⚠️ **[UFS en macOS — origen]**
UFS deriva del **Berkeley Fast File System (FFS)**, originado en **Bell Laboratories**. No confundir origen (Bell Labs) con los sistemas que lo usan (BSD derivados).

⚠️ **[HFS vs HFS+]**
HFS fue el original. HFS+ es el **sucesor** y el **sistema de ficheros primario en Macintosh**. El examen puede preguntar cuál es el primario actual: HFS+.

⚠️ **[Database File System — criterio de identificación]**
El Database FS identifica ficheros por **características/metadatos** (tipo, autor, tema). Si la pregunta menciona "metadata" como método de identificación en lugar de jerarquía de directorios, la respuesta es Database File System.

⚠️ **[Special Purpose File System — función]**
Diseñado para **comunicación entre procesos o espacio temporal**. No confundir con Network FS (que accede por red) ni con Flash FS.

---

## 3. Nemotécnicos

### FAT: bytes por clúster (en orden creciente)
**"1,5 — 2 — 4"** → FAT12, FAT16, FAT32
Regla: el número de la versión crece → los bytes por entrada también crecen.

### NTFS System Files — los 10 ficheros `$` 🔴
**"MiMirror Log Boot Bitmap Bad Attr Quota Up Vol"**
- **$mft** / **$mftmirr** — MFT y su espejo
- **$logfile** — log de recuperación
- **$boot** — bootstrap
- **$bitmap** — mapa de volumen
- **$badclus** — clústeres malos
- **$attrdef** — definición de atributos
- **$quota** — cuotas de usuario
- **$upcase** — mayúsculas Unicode
- **$volume** — nombre y versión

### EXT1→4: progresión de características
- EXT: primero, sin timestamps separados, max 2 GB / 255 chars
- EXT2: velocidad + timestamps, **sin journaling**
- EXT3: EXT2 + **journaling** (`tune2fs -j`)
- EXT4: EXT3 + 16TB/1EB + extents + delayed alloc + nanosecond timestamps + backward compat

### EXT4 límites numéricos
**"16 para uno, 1 para todos"**
- Fichero individual: **16 TB**
- FS completo: **1 EB**

### FHS — directorios clave del examen (los que más confunden)
- `/proc` → procesos y kernel como **ficheros virtuales**
- `/sys` → dispositivos **conectados**
- `/run` → procesos **en ejecución ahora**
- `/opt` → software **adicional (add-on)**
- `/srv` → datos de **servicios del sistema**
- `/mnt` → montaje **temporal**
- `/media` → medios **extraíbles**

### MBR FAT32 — estructura de offsets
**"446 + 4×16 + 2"** = 512 bytes totales
- 446 bytes: código ejecutable
- 4 entradas × 16 bytes = 64 bytes
- 2 bytes: firma de arranque

---

## 4. Flashcards

**Q:** ¿Qué tamaño de entrada en la FAT tiene FAT16?
**A:** 2 bytes por clúster. (FAT12 = 1,5 bytes; FAT32 = 4 bytes)

---

**Q:** ¿Cuál fue el primer sistema de ficheros usado con Windows/DOS?
**A:** FAT (File Allocation Table).

---

**Q:** ¿Dónde se ubica la tabla FAT dentro del volumen?
**A:** Al principio del volumen (beginning of the volume).

---

**Q:** ¿Cuál es el tamaño máximo de unidad que soporta FAT32?
**A:** 2 TB.

---

**Q:** ¿Desde qué versión de Windows NT es NTFS el sistema de ficheros por defecto?
**A:** Desde Windows NT 3.1.

---

**Q:** ¿Qué mejoras aporta NTFS sobre FAT? (3 principales)
**A:** Soporte mejorado para metadatos, estructuras de datos avanzadas para rendimiento/fiabilidad/uso de disco, ACLs de seguridad y journaling.

---

**Q:** ¿Cuál es la función del fichero NTFS `$mftmirr`?
**A:** Espejo del MFT usado para recuperar ficheros.

---

**Q:** ¿Cuál es la función del fichero NTFS `$logfile`?
**A:** Usado para recuperación (journaling).

---

**Q:** ¿En qué versión de NTFS se introdujo EFS?
**A:** NTFS versión 3.0.

---

**Q:** ¿Necesita el propietario de un fichero EFS descifrarlo para editarlo?
**A:** No. EFS es transparente para el propietario; el cifrado/descifrado es automático.

---

**Q:** ¿Qué ahorra el atributo Sparse File en NTFS?
**A:** Espacio en disco, asignando clústeres solo para datos no cero (significativos). Los datos cero no ocupan espacio real.

---

**Q:** ¿Cuál fue el primer sistema de ficheros de Linux y a qué sistema superó?
**A:** EXT, que superó las limitaciones del sistema de ficheros Minix (64 MB de partición, nombres cortos).

---

**Q:** ¿EXT2 es un sistema de ficheros journaling?
**A:** No. EXT2 no tiene journaling. Su mayor riesgo es la corrupción al escribir.

---

**Q:** ¿Cuál es el comando para convertir EXT2 a EXT3?
**A:** `#/sbin/tune2fs -j`

---

**Q:** ¿Cuál es el tamaño máximo de fichero individual en EXT4?
**A:** 16 TB.

---

**Q:** ¿Cuál es el tamaño máximo del sistema de ficheros EXT4 completo?
**A:** 1 EB (exabyte).

---

**Q:** ¿Qué versión mínima del kernel Linux requiere EXT4?
**A:** Linux Kernel v2.6.19.

---

**Q:** ¿Qué característica de EXT4 permite montar particiones EXT2 y EXT3 sin reformatear?
**A:** Backward compatibility.

---

**Q:** ¿Cuál es el sistema de ficheros primario en Macintosh?
**A:** HFS+ (HFS Plus).

---

**Q:** ¿De qué sistema deriva UFS y dónde fue desarrollado originalmente?
**A:** Del Berkeley Fast File System (FFS), desarrollado originalmente en Bell Laboratories.

---

**Q:** ¿Qué directorio Linux contiene información sobre procesos y el kernel expuesta como ficheros virtuales?
**A:** `/proc`

---

**Q:** ¿Qué tipo de sistema de ficheros identifica los ficheros por sus características/metadatos en lugar de por jerarquía?
**A:** Database File System.

---

## 5. Confusión frecuente

### FAT12 vs FAT16 vs FAT32 — bytes por clúster
- **FAT12**: 1,5 bytes. El más antiguo, el de menor capacidad.
- **FAT16**: 2 bytes. Intermedio.
- **FAT32**: 4 bytes. El más moderno de los FAT, soporta hasta 2 TB.
- **Criterio de decisión**: ordenarlos por número de versión = ordenarlos por bytes por clúster (ascendente). Si el examen da los bytes y pide la versión, el más pequeño siempre es FAT12.

---

### EXT2 vs EXT3 — journaling
- **EXT2**: sin journaling. Riesgo de corrupción en escritura. Más rápido en operaciones simples.
- **EXT3**: con journaling. Mayor integridad. Se convierte desde EXT2 con `tune2fs -j`.
- **Criterio de decisión**: si la pregunta menciona "journaling", "integridad ante fallos" o "apagados abruptos" → EXT3. Si menciona "sin journaling" o "riesgo de corrupción" → EXT2.

---

### `$mft` vs `$mftmirr`
- **`$mft`**: Master File Table. Contiene un registro para **cada fichero** del volumen.
- **`$mftmirr`**: espejo del MFT. Se usa para **recuperar ficheros** cuando el MFT principal está dañado.
- **Criterio de decisión**: si la pregunta habla de "recuperación" → `$mftmirr`. Si habla de "registro de todos los ficheros" → `$mft`.

---

### Sparse Files vs EFS — mecanismos NTFS de gestión de datos
- **Sparse Files**: ahorran **espacio en disco** no asignando clústeres para datos cero.
- **EFS**: cifran el contenido del fichero para **control de acceso**.
- **Criterio de decisión**: si la pregunta menciona "espacio", "ceros", "datos no significativos" → Sparse. Si menciona "cifrado", "acceso no autorizado", "transparencia" → EFS.

---

### `/proc` vs `/sys` vs `/run` — directorios virtuales Linux
- **`/proc`**: información de **procesos y kernel** expuesta como ficheros virtuales.
- **`/sys`**: información de **dispositivos conectados**.
- **`/run`**: información sobre procesos **actualmente en ejecución** (daemons activos, usuarios logados).
- **Criterio de decisión**: kernel/procesos → `/proc`; hardware/dispositivos → `/sys`; estado runtime actual → `/run`.

---

### HFS vs HFS+
- **HFS**: el original de Apple para Mac.
- **HFS+**: sucesor de HFS. **Sistema de ficheros primario actual en Macintosh**.
- **Criterio de decisión**: si la pregunta pregunta cuál es el primario en Mac → HFS+. Si pregunta cuál vino primero → HFS.

---

### Shared Disk FS vs Network FS
- **Shared Disk FS**: múltiples servidores acceden al mismo **subsistema de disco externo** (SAN-like). El disco es compartido directamente.
- **Network FS**: acceso a ficheros en otros equipos a través de **red**. El disco no es local; se accede vía protocolo de red.
- **Criterio de decisión**: "disco externo compartido" → Shared Disk. "red" o "otro equipo" → Network FS.
