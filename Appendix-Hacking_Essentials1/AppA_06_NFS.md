# AppA_06_NFS.md
# CEH v13 — Appendix A: Network File System (NFS)

---

## 1. Conceptos y definiciones

### NFS — qué es y cómo funciona

NFS (Network File System) es un protocolo de sistema de ficheros distribuido que permite a los usuarios **leer, escribir, almacenar y acceder a ficheros** en dispositivos conectados a través de una red. Funciona sobre todas las redes basadas en IP y usa **TCP/UDP** para el acceso y entrega de datos.

---

### Seguridad en NFS 🔴

NFS ofrece **dos niveles de seguridad**:

| Nivel | Tipo | Descripción |
|---|---|---|
| **Host level** | Control de acceso | Restringe operaciones cuando el usuario remoto no proporciona credenciales correctas |
| **File level** | Operacional | Limita las acciones sobre los ficheros en un sistema de ficheros montado |

---

### Métodos de control de acceso en NFS 🔴

**Root Squashing**
- Limita los privilegios de superusuario mediante autenticación de identidad.
- El administrador mapea el **UID de root al usuario anónimo** en la estructura de credenciales NFS RPC.
- Resultado: el root remoto no tiene privilegios de root en el servidor NFS.

**nosuid**
- Impide que los bits **SUID o SGID** tengan efecto en el sistema de ficheros montado.
- Usa la opción `nosuid` para **prevenir la ejecución de ejecutables NFS** que cambien la identidad del usuario en el host.

**noexec**
- Impide la ejecución de ficheros desde la partición montada.
- Usa la opción `noexec` para **prevenir que un usuario ejecute binarios** desde el sistema de ficheros NFS.

---

## 2. Exam Traps ⚠️

⚠️ **[NFS — protocolo de transporte]**
NFS usa **TCP y UDP** (ambos). El examen puede presentar solo uno como respuesta correcta. La respuesta completa incluye los dos.

⚠️ **[Host level vs File level — qué restringe cada uno]**
Host level restringe **operaciones** cuando las credenciales son incorrectas (quién puede conectarse). File level limita **acciones sobre los ficheros** en el sistema montado (qué puede hacerse). El examen puede invertir las definiciones.

⚠️ **[Root Squashing — mecanismo exacto]**
No elimina al usuario root: **mapea su UID al usuario anónimo** en las credenciales NFS RPC. La trampa es presentar "elimina el acceso root" como correcto — lo que hace es neutralizar sus privilegios reasignando su identidad.

⚠️ **[nosuid vs noexec — distinción fina]**
nosuid bloquea el efecto de los bits SUID/SGID (cambio de identidad al ejecutar). noexec bloquea directamente la **ejecución de cualquier binario** desde la partición. El examen puede presentar un escenario de escalada por SUID y preguntar qué opción lo previene: si el problema es SUID → nosuid; si es ejecución arbitraria → noexec.

---

## 3. Nemotécnicos

### Los 3 métodos NFS — "RSN"
Root Squashing → nosuid → noexec

Regla de alcance (de más específico a más amplio):
- **Root Squashing**: neutraliza al root reasignando su UID al anónimo.
- **nosuid**: bloquea solo el cambio de identidad por SUID/SGID.
- **noexec**: bloquea toda ejecución de binarios desde la partición.

### Niveles de seguridad NFS
- **Host level** = control de **acceso** (credenciales, identidad).
- **File level** = control **operacional** (qué hacer con los ficheros).

---

## 4. Flashcards

**Q:** ¿Qué protocolos de transporte usa NFS para el acceso y entrega de datos?
**A:** TCP y UDP (ambos).

---

**Q:** ¿Cuáles son los dos tipos de seguridad que ofrece NFS?
**A:** Host level (control de acceso) y File level (operacional).

---

**Q:** ¿Qué hace exactamente Root Squashing en NFS?
**A:** Mapea el UID de root al usuario anónimo en la estructura de credenciales NFS RPC, neutralizando así los privilegios de superusuario del usuario root remoto.

---

**Q:** ¿Qué impide la opción `nosuid` en NFS?
**A:** Que los bits SUID o SGID tengan efecto en el sistema de ficheros montado, previniendo que ejecutables cambien la identidad del usuario.

---

**Q:** ¿Qué impide la opción `noexec` en NFS?
**A:** La ejecución de cualquier fichero o binario desde la partición montada.

---

**Q:** Un atacante monta un sistema de ficheros NFS y ejecuta un binario con bit SUID para escalar privilegios. ¿Qué opción NFS lo hubiera prevenido?
**A:** `nosuid` — impide que el bit SUID tenga efecto en ficheros del sistema de ficheros NFS montado.

---

**Q:** ¿Cuál es la diferencia de alcance entre `nosuid` y `noexec`?
**A:** `nosuid` bloquea solo el cambio de identidad por SUID/SGID (la ejecución en sí sigue permitida). `noexec` bloquea la ejecución de cualquier binario desde la partición — es más restrictivo.

---

**Q:** ¿A qué nivel de seguridad NFS pertenece Root Squashing?
**A:** Host level — restringe los privilegios del superusuario remoto basándose en su identidad/credenciales.

---

## 5. Confusión frecuente

### nosuid vs noexec
- **nosuid**: permite ejecutar binarios pero **neutraliza el SUID/SGID** (el binario no puede cambiar la identidad del proceso).
- **noexec**: impide **ejecutar cualquier binario** desde la partición, independientemente de si tiene o no SUID.
- **Criterio de decisión**: escalada de privilegios por SUID → nosuid. Ejecución de código arbitrario desde NFS → noexec.

---

### Host level vs File level
- **Host level**: controla **quién** puede conectarse y qué operaciones puede realizar según sus credenciales. Mecanismo representativo: Root Squashing.
- **File level**: controla **qué** puede hacerse con los ficheros del sistema montado. Mecanismos representativos: nosuid, noexec.
- **Criterio de decisión**: credenciales o identidad del usuario → Host level. Permisos sobre ficheros o ejecución → File level.
