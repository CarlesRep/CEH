# AppA_05_VirtualizationConcepts.md
# CEH v13 — Appendix A: Virtualization Concepts

---

## 1. Conceptos y definiciones

### Qué es virtualización

Virtualización es la creación de una versión virtual de recursos de **hardware o software** en un sistema. El cambio fundamental: antes de virtualización, una plataforma hardware ejecuta **un solo SO** y sus aplicaciones; después, la capa de virtualización (ej. VMware) permite ejecutar **múltiples SOs y sus aplicaciones** sobre el mismo hardware físico.

---

### Características de la virtualización 🔴

| Característica | Descripción |
|---|---|
| **Partitioning** | Ejecutar múltiples SOs y apps en un único sistema físico particionando virtualmente los recursos hardware |
| **Isolation** | Cada VM está aislada de su host físico y del resto de VMs |
| **Encapsulation** | Una VM se representa como un único fichero identificable por sus servicios; protege la VM de interferencias de otras VMs |

La lógica: **Partitioning** hace posible la coexistencia; **Isolation** garantiza que un fallo en una VM no se propague; **Encapsulation** permite portabilidad y protección.

---

### Beneficios de la virtualización 🔴

| Beneficio | Mecanismo |
|---|---|
| **Resource Efficiency** | Mayor utilización del hardware → mayor ROI |
| **Increase in Uptime** | Recursos redundantes e interconexiones en un único sistema físico |
| **Reduced Disk Space** | Utilización eficiente del espacio de disco disponible |
| **Increased Flexibility** | Mayor flexibilidad en despliegue; multiplexación de recursos de red |
| **Business Continuity** | Facilita continuidad de negocio y recuperación ante desastres |
| **Improved QoS** | Distribuye la carga de red entre VMs |
| **Migration** | Permite mover datos, apps, SOs, procesos y recursos entre máquinas |
| **Environmental Benefits** | Menos emisiones de CO2 y ahorro energético |

---

### Vendors de virtualización

| Vendor | Especialidad |
|---|---|
| **VMware** | Virtualización de networking, storage y seguridad; data centers virtuales |
| **Citrix** | Virtualización de apps y escritorios Windows como servicio seguro bajo demanda |
| **Oracle** | Virtualización completa e integrada desde escritorios hasta data centers (hardware + software stacks) |
| **Microsoft** | Rango desde data center hasta escritorio; gestión de activos físicos y virtuales desde plataforma única |

---

### Seguridad en virtualización

El proceso típico de seguridad en virtualización incluye tres niveles:
1. Asegurar el **entorno virtual**.
2. Asegurar cada VM a **nivel de sistema**.
3. Asegurar la **red virtual**.

**Preocupaciones de seguridad**:
- La complejidad adicional de infraestructura dificulta la monitorización de eventos anómalos.
- Una VM offline puede usarse como **pasarela** para acceder a los sistemas de la empresa.
- La naturaleza dinámica de las VMs permite mover cargas de trabajo a VMs con **menor nivel de seguridad**.

---

### Virtual Firewall

Software firewall que monitoriza y controla paquetes transmitidos **entre VMs**. Corre completamente en el entorno virtual. Dos modos de funcionamiento:

| Modo | Ubicación | Alcance |
|---|---|---|
| **Bridge-mode** | Reside en el virtual switch inter-red | Filtra el tráfico entre redes virtuales |
| **Hypervisor-mode** | Reside en el Virtual Machine Monitor (VMM) | Monitoriza **toda la actividad de las VMs**: hardware, software, storage, servicios y memoria |

---

### Virtual Operating Systems

Instalación lógica de un SO en software de virtualización sobre un host OS ya instalado. Permite ejecutar múltiples SOs en un único hardware.

**Ventajas**: no requiere hardware adicional; uso eficiente de recursos; replica servicios del host OS (backup, recovery, gestión de seguridad).

**Limitaciones**: consume muchos recursos del host (CPU y memoria); las llamadas al sistema del SO virtual deben pasar por el hardware del host OS → **penalización de rendimiento**.

---

### Virtual Databases

Sistema de gestión de bases de datos que permite a los usuarios **consultar varias bases de datos simultáneamente** tratándolas como una única entidad.

**Ventajas**: reparte la carga de BDs grandes; simplifica migraciones; despliegue dinámico de nuevas instancias; mayor disponibilidad mediante aislamiento y failover entre BDs virtuales.

**Desventajas**: alto consumo de recursos; mayor complejidad para los DBAs; dificultad para diagnosticar errores (origen en la VM o en la BD).

---

## 2. Exam Traps ⚠️

⚠️ **[Características: Partitioning vs Isolation vs Encapsulation]**
El examen puede dar definiciones y pedir el término. Clave de distinción: "múltiples SOs en un solo hardware" → Partitioning. "cada VM no afecta a otras" → Isolation. "VM como fichero único que se protege de interferencias" → Encapsulation.

⚠️ **[Encapsulation — doble función]**
Encapsulation no solo protege; también permite **identificar** la VM por sus servicios (fichero único). El examen puede preguntar solo por la función de protección y omitir la de identificación, o viceversa.

⚠️ **[Virtual Firewall: bridge-mode vs hypervisor-mode]**
Bridge-mode opera en el **virtual switch** (filtra tráfico de red). Hypervisor-mode opera en el **VMM** y monitoriza TODO (hardware, software, storage, memoria). El examen puede confundir el nivel de visibilidad de cada modo.

⚠️ **[Virtual OS — limitación de rendimiento]**
Las system calls del SO virtual deben pasar por el hardware del host OS, lo que **penaliza el rendimiento**. No es una limitación de funcionalidad sino de velocidad. El examen puede presentar "no puede usar hardware directamente" como formulación alternativa correcta.

⚠️ **[Seguridad: VMs offline como vector de ataque]**
Una VM **offline** (apagada) puede usarse como pasarela para acceder a los sistemas de la empresa. El examen puede presentar "solo las VMs activas son un riesgo de seguridad" como trampa — es falso.

⚠️ **[Business Continuity como beneficio de virtualización]**
El examen puede no asociar virtualización con recuperación ante desastres. Business Continuity y disaster recovery son beneficios explícitos de la virtualización según el libro.

---

## 3. Nemotécnicos

### Las 3 características de virtualización
**"PIE"**
- **P**artitioning: múltiples SOs en un hardware
- **I**solation: VMs aisladas entre sí
- **E**ncapsulation: VM como fichero único + protección

### Virtual Firewall: modos
- **Bridge** → switch (entre redes)
- **Hypervisor** → VMM (todo: hardware, software, storage, memoria)
Regla: el que está más abajo en la pila (VMM) ve más.

### Beneficios: los 8 con iniciales
**"RE-UP-RD-FL-BC-QS-MI-EN"**
Resource Efficiency, Uptime, Reduced Disk, Flexibility, Business Continuity, QoS, Migration, Environmental.

---

## 4. Flashcards

**Q:** ¿Cuál es la diferencia fundamental entre un entorno sin virtualización y uno con virtualización?
**A:** Sin virtualización, el hardware ejecuta un solo SO. Con virtualización, la capa de virtualización permite ejecutar múltiples SOs y sus aplicaciones sobre el mismo hardware físico.

---

**Q:** ¿Qué característica de la virtualización permite ejecutar múltiples SOs en un único sistema físico?
**A:** **Partitioning** — particiona virtualmente los recursos hardware.

---

**Q:** ¿Qué característica de la virtualización garantiza que un fallo en una VM no afecte a otras?
**A:** **Isolation** — cada VM está aislada del host físico y del resto de VMs.

---

**Q:** ¿Qué representa Encapsulation en el contexto de virtualización?
**A:** La VM se representa como un único fichero identificable por sus servicios, lo que la protege de interferencias de otras VMs.

---

**Q:** ¿En qué modo del virtual firewall reside éste en el Virtual Machine Monitor y monitoriza toda la actividad de las VMs?
**A:** **Hypervisor-mode**.

---

**Q:** ¿En qué modo del virtual firewall reside éste en el virtual switch inter-red?
**A:** **Bridge-mode**.

---

**Q:** ¿Por qué el rendimiento de un SO virtual es inferior al de un SO nativo?
**A:** Porque las llamadas al sistema del SO virtual deben pasar por el hardware del host OS, lo que introduce una penalización de rendimiento.

---

**Q:** ¿Qué beneficio de virtualización está directamente relacionado con disaster recovery?
**A:** **Business Continuity** — la virtualización facilita la continuidad de negocio y la recuperación ante desastres.

---

**Q:** ¿Cómo mejora la virtualización la QoS?
**A:** Distribuyendo la carga de red entre las máquinas virtuales.

---

**Q:** ¿Cuál es la principal preocupación de seguridad relacionada con la naturaleza dinámica de las VMs?
**A:** Que la carga de trabajo puede moverse fácilmente a una VM con **menor nivel de seguridad**.

---

**Q:** ¿Qué permite una base de datos virtual que no permite una BD convencional?
**A:** Consultar varias bases de datos simultáneamente tratándolas como una única entidad.

---

**Q:** ¿Qué vendor de virtualización se especializa en virtualización de apps y escritorios Windows como servicio seguro bajo demanda?
**A:** **Citrix**.

---

**Q:** ¿Qué vendor de virtualización cubre desde escritorios hasta data centers con una solución completa e integrada de hardware y software stacks?
**A:** **Oracle**.

---

**Q:** ¿Cuál es una limitación de seguridad de las VMs offline según el libro?
**A:** Una VM offline puede usarse como **pasarela** para acceder a los sistemas de la empresa.

---

**Q:** ¿Qué beneficio de virtualización permite trasladar datos, apps, SOs y procesos entre máquinas?
**A:** **Migration**.

---

## 5. Confusión frecuente

### Partitioning vs Isolation
- **Partitioning**: hace posible que **coexistan múltiples SOs** en el mismo hardware (es el mecanismo de división de recursos).
- **Isolation**: garantiza que las VMs **no se interfieran entre sí** (es el mecanismo de separación y contención).
- **Criterio de decisión**: "varios SOs en un hardware" → Partitioning. "fallo contenido en una VM" o "no afecta a otras" → Isolation.

---

### Bridge-mode vs Hypervisor-mode (Virtual Firewall)
- **Bridge-mode**: actúa en el **virtual switch**. Filtra tráfico de red entre redes virtuales. Visibilidad: tráfico de red.
- **Hypervisor-mode**: actúa en el **VMM**. Monitoriza hardware, software, storage, servicios y memoria. Visibilidad: todo.
- **Criterio de decisión**: "filtrar tráfico de red entre VMs" → Bridge. "monitorizar toda la actividad de las VMs incluyendo hardware y memoria" → Hypervisor.

---

### Virtual OS vs Virtual Database
- **Virtual OS**: instalación lógica de un SO sobre un host OS existente. Problema: rendimiento (system calls pasan por el host).
- **Virtual Database**: DBMS que permite consultar múltiples BDs como si fueran una. Problema: complejidad para DBAs y alto consumo de recursos.
- **Criterio de decisión**: si la pregunta menciona "múltiples SOs" o "sistema operativo" → Virtual OS. Si menciona "múltiples bases de datos" o "consulta unificada" → Virtual Database.

---

### VMware vs Citrix vs Oracle vs Microsoft — especialidades
- **VMware**: data centers, networking, storage, seguridad.
- **Citrix**: apps y escritorios Windows como servicio.
- **Oracle**: solución completa (escritorio → data center), hardware + software stacks.
- **Microsoft**: físico + virtual desde data center hasta escritorio, plataforma única.
- **Criterio de decisión**: "Windows apps/desktops como servicio" → Citrix. "networking y storage virtualizados" → VMware. "hardware y software stacks integrados" → Oracle.
