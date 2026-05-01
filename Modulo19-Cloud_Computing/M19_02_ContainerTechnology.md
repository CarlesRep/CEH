# M19_02 — Container Technology
**Módulo 19 / Subapartado 2 — Container Technology**

---

## 1. Conceptos y definiciones

### Qué es un contenedor

Un contenedor es un paquete de software que incluye **todas sus dependencias** (librerías, archivos de configuración, binarios y otros recursos) y ejecuta de forma independiente del resto de procesos en el entorno cloud. Se provee como CaaS, hereda características de IaaS y PaaS, y se apoya en orquestadores para su gestión.

La diferencia crítica frente a VMs: los contenedores comparten el kernel del SO del host. No necesitan SO propio → son ligeros (megabytes), arrancan en segundos, y el crash de uno no afecta a los demás.

---

### 🔴 Contenedores vs. Máquinas Virtuales

| Virtual Machines | Containers |
|-----------------|------------|
| Heavyweight | Lightweight y portables |
| SO independiente por VM | Comparten un único SO del host |
| Virtualización basada en hardware | Virtualización basada en SO |
| Aprovisionamiento lento | Aprovisionamiento escalable y en tiempo real |
| Rendimiento limitado | Rendimiento nativo |
| Aislamiento completo → más seguras | Aislamiento a nivel de proceso → parcialmente seguras |
| Creación y arranque en **minutos** | Creación y arranque en **segundos** |

---

### 🔴 Arquitectura de contenedores — 5 tiers + 3 fases

**Cinco tiers:**

| Tier | Componente | Función |
|------|-----------|---------|
| Tier-1 | Developer machines | Creación de imágenes, testing y acreditación |
| Tier-2 | Testing and accreditation systems | Verificación/validación de contenido de imágenes, firma y envío a registros |
| Tier-3 | Registries | Almacenamiento y distribución de imágenes a orquestadores |
| Tier-4 | Orchestrators | Transforman imágenes en contenedores y los despliegan en hosts |
| Tier-5 | Hosts | Operan y gestionan contenedores según instrucciones del orquestador |

**Tres fases del ciclo de vida:**
1. **Image Creation, Testing and Accreditation** — desarrolladores crean la imagen; equipos de seguridad la testean y acreditan.
2. **Image Storage and Retrieval** — almacenamiento en registries (Docker Hub, Amazon ECR, Docker Trusted Registry/DTR); incluye tagging, catalogación, control de versiones.
3. **Container Deployment and Management** — orquestadores (Kubernetes, Docker Swarm, Nomad, Mesos) extraen imágenes, despliegan contenedores, monitorizan recursos, reinician en nuevos hosts ante fallos, destruyen y recrean ante actualizaciones.

---

### Docker

Docker es tecnología open-source para desarrollar, empaquetar y ejecutar aplicaciones. Provee PaaS mediante **virtualización a nivel de SO** (OS-level virtualization). Los contenedores Docker son aislados entre sí y se comunican por canales bien definidos.

#### Docker Engine — componentes

| Componente | Descripción |
|-----------|------------|
| **Server (daemon `dockerd`)** | Proceso persistente en background |
| **REST API** | Comunicación y asignación de tareas al daemon |
| **Client CLI** | Interfaz de línea de comandos; inicia comandos Docker |

#### 🔴 Docker Architecture — componentes clave

| Componente | Función |
|-----------|---------|
| **Docker Daemon (`dockerd`)** | Procesa peticiones API; gestiona contenedores, volúmenes, imágenes, redes |
| **Docker Client** | Interfaz primaria del usuario; envía comandos a `dockerd` vía Docker API |
| **Docker Registries** | Almacenan y sirven imágenes; pueden ser privados o públicos. Docker Hub = registro público predeterminado |
| **Images** | Plantillas binarias de solo lectura con instrucciones para crear contenedores |
| **Containers** | Instancia ejecutable de una imagen; se crean/arrancan/detienen/destruyen vía CLI o API |
| **Services** | Escalan contenedores a través de múltiples daemons formando un swarm (managers + workers); se comunican vía Docker API |
| **Networking** | Canal de comunicación entre contenedores aislados |
| **Volumes** | Almacenamiento persistente de datos generados y usados por contenedores |

La comunicación entre Docker client y Docker daemon: **REST API**.

#### Docker Swarm

Modo del Docker engine que gestiona múltiples Docker engines. Permite:
- Comunicar con contenedores y asignarles trabajos
- Expandir/reducir número de contenedores según carga
- Health checks y gestión del ciclo de vida
- Failover y redundancia ante fallos de nodo
- Actualizaciones de software a todos los contenedores

#### 🔴 Docker Networking — Container Network Model (CNM)

CNM: arquitectura de networking de Docker. Tres construcciones:

| Construcción | Función |
|-------------|---------|
| **Sandbox** | Configuración del stack de red del contenedor (interfaces, routing tables, DNS) |
| **Endpoint** | Conecta el contenedor a la red; abstraído de la aplicación para portabilidad |
| **Network** | Colección de endpoints interconectados; sin conexión → sin comunicación |

**Dos interfaces pluggables del CNM:**
- **Network Drivers**: implementan funciones de red; múltiples drivers concurrentes posibles. Tipos: native y remote.
- **IPAM Drivers**: asignan subredes y direcciones IP a endpoints/redes si no están asignadas.

**5 native network drivers del Docker Engine:**

| Driver | Función |
|--------|---------|
| **Host** | El contenedor usa el stack de red del host directamente |
| **Bridge** | Crea un Linux bridge en el host gestionado por Docker |
| **Overlay** | Comunicación entre contenedores a través de infraestructura de red física |
| **MACVLAN** | Conexión entre interfaces de contenedor e interfaz del host usando Linux MACVLAN bridge mode |
| **None** | Stack de red propio; completamente aislado del host |

**3 remote drivers (community/vendors):**

| Driver | Origen | Función |
|--------|--------|---------|
| **Contiv** | Cisco (open-source) | Políticas de seguridad e infraestructura para microservicios multi-tenant |
| **Weave** | Community | Red virtual para conectar contenedores Docker en múltiples clouds |
| **Kuryr** | Community | Implementa remote driver de Docker libnetwork usando Neutron (OpenStack); incluye IPAM driver |

---

### Kubernetes (K8s)

Plataforma open-source de orquestación desarrollada por **Google**. También conocido como **K8s**. Gestiona aplicaciones en contenedores y microservicios en entornos de producción.

**Features clave:**
- **Service discovery**: vía DNS o IP.
- **Load balancing**: distribuye tráfico automáticamente ante alta carga.
- **Storage orchestration**: monta almacenamiento local o cloud.
- **Automated rollouts/rollbacks**: crea/destruye contenedores y mueve recursos automáticamente.
- **Automatic bin packing**: asigna/desasigna recursos (CPU, memoria) según especificación.
- **Self-healing**: health checks; reemplaza contenedores fallidos; no anuncia contenedores no disponibles a clientes.
- **Secret and configuration management**: almacena credenciales, SSH keys, OAuth tokens; despliegue sin reconstruir imágenes.

#### 🔴 Kubernetes Cluster Architecture

Un cluster = mínimo **1 master node + 1 worker node**. Los worker nodes contienen **pods** (grupo de contenedores). El master los gestiona.

**Master Components:**

| Componente | Función |
|-----------|---------|
| **Kube-apiserver** | Front-end del control panel; responde todas las peticiones API; **único componente que interactúa con etcd** |
| **Etcd cluster** | Almacenamiento key-value distribuido y consistente; guarda datos del cluster, service discovery, objetos API |
| **Kube-scheduler** | Escanea pods nuevos y les asigna nodo según recursos, localidad de datos, restricciones hardware/software/política |
| **Kube-controller-manager** | Ejecuta controllers (node, endpoint, replication, service account/token); combinados en un único binario y proceso |
| **Cloud-controller-manager** | Ejecuta controllers que se comunican con proveedores cloud; permite evolución independiente de código K8s y cloud |

**Node (Worker) Components:**

| Componente | Función |
|-----------|---------|
| **Kubelet** | Agente en cada nodo; garantiza que los contenedores del pod estén corriendo y sanos; **no gestiona contenedores no creados por Kubernetes** |
| **Kube-proxy** | Proxy de red en cada worker node; mantiene reglas de red para conexión a pods |
| **Container Runtime** | Software que ejecuta contenedores; K8s soporta Docker, rktlet, containerd, cri-o |

---

### Container Orchestration

Proceso automatizado de gestión del ciclo de vida de contenedores en entornos dinámicos. Tareas que automatiza:
- Provisioning y despliegue
- Failover y redundancia
- Creación/destrucción de contenedores para distribución de carga
- Movimiento de contenedores entre hosts ante agotamiento de recursos o fallo
- Asignación automática de recursos
- Exposición de servicios al exterior
- Load balancing, traffic routing, service discovery
- Health checks de contenedores y hosts
- Configuración de contenedores de aplicación
- Seguridad en comunicación entre contenedores

---

### Tipos de clústeres

| Tipo | Descripción |
|------|------------|
| **HA / Fail-over** | Múltiples nodos simultáneos; si uno falla, otro asume su rol con mínimo downtime |
| **Load Balancing** | Distribuye carga entre nodos; health checks periódicos; redirige tráfico ante fallos; también es HA |
| **High-Performance Computing (HPC)** | Nodos configurados para máximo rendimiento paralelizando tareas |

---

### 🔴 Container Security Challenges

Desafíos de seguridad específicos del examen:

| Challenge | Mecanismo de ataque/riesgo |
|-----------|--------------------------|
| **Inflow of vulnerable source code** | Código no controlado en repositorios open-source con vulnerabilidades |
| **Large attack surface** | Múltiples contenedores, apps, VMs, DBs en el mismo host |
| **Lack of visibility** | El container engine abstrae las acciones; dificulta tracking de actividades |
| **Compromising secrets** | API keys, usuarios, contraseñas accesibles a atacantes con acceso al contenedor |
| **DevOps speed** | Contenedores efímeros: se ejecutan, detienen y eliminan rápidamente → atacantes se ocultan sin instalar malware |
| **Noisy neighboring containers** | Un contenedor agota recursos → DoS sobre otros contenedores |
| **Container breakout to the host** | Contenedores ejecutando como root pueden escalar privilegios al SO del host |
| **Network-based attacks** | Contenedores fallidos con raw sockets y conexiones de red activas → vector de ataque |
| **Bypassing isolation** | Desde un contenedor comprometido → escalada de privilegios a otros contenedores o al host |
| **Kernel exploits** | Los contenedores comparten el kernel del host → una vulnerabilidad afecta a **todos** |
| **Cgroups misconfiguration** | Control groups mal configurados → contención de recursos → DoS |
| **Pod Security Policies (PSPs)** | Configuración compleja y propensa a errores en Kubernetes → brechas de seguridad |
| **Misconfigurations** | Políticas de red excesivamente permisivas o controles de acceso mal configurados |
| **Isolation Breakdowns** | Vulnerabilidades en el container runtime o kernel → escape de contenedor |
| **Insecure Communication** | Sin cifrado → tráfico entre contenedores interceptable |
| **Insufficient Logging/Monitoring** | Dificulta detección y respuesta a incidentes |
| **Patch Management** | Difícil en despliegues a gran escala |
| **Persistent Storage Security** | Requiere cifrado, control de acceso y backup específicos |
| **Compliance Audits Risks** | GDPR, HIPAA, SOX — incumplimiento con impacto reputacional y financiero |

---

## 2. Exam Traps ⚠️

⚠️ **[Contenedores — tiempo de arranque]**
Contenedores arrancan en **segundos**. VMs arrancan en **minutos**. El examen puede invertirlo.

⚠️ **[Contenedores — tipo de virtualización]**
Los contenedores usan **virtualización basada en SO** (OS-level virtualization). Las VMs usan virtualización basada en hardware. Confundirlos es un error clásico.

⚠️ **[Contenedores — nivel de aislamiento]**
Contenedores: aislamiento a **nivel de proceso** → parcialmente seguros. VMs: aislamiento completo → más seguras. El examen puede afirmar que los contenedores son más seguros por su independencia — solo son más seguros entre sí si uno se compromete, pero el kernel compartido es un vector crítico.

⚠️ **[Kube-apiserver — interacción con etcd]**
El kube-apiserver es el **único componente de Kubernetes que interactúa con el etcd cluster**. El examen puede presentar kube-scheduler o kube-controller-manager como los que acceden a etcd — incorrecto.

⚠️ **[Kubelet — contenedores que no gestiona]**
Kubelet **no gestiona contenedores que no hayan sido creados por Kubernetes**. El examen puede presentarlo como gestor universal de contenedores en el nodo.

⚠️ **[Docker — tipo de PaaS]**
Docker provee PaaS mediante **OS-level virtualization**, no mediante hardware virtualization. El examen puede confundir el nivel de virtualización de Docker con el de un hipervisor.

⚠️ **[Docker — comunicación client/daemon]**
La comunicación entre Docker client y Docker daemon es siempre por **REST API**. Aplica tanto en local como en conexión a daemon remoto.

⚠️ **[Noisy neighboring containers — tipo de ataque]**
Un contenedor que agota los recursos del sistema provoca un **DoS** sobre los contenedores vecinos. El examen puede no asociar este escenario con DoS.

⚠️ **[Container breakout — condición]**
El breakout al host ocurre cuando el contenedor se ejecuta como **root**. No es un fallo genérico de aislamiento — requiere esa condición específica.

⚠️ **[Kernel exploits — alcance]**
Dado que los contenedores comparten el kernel del host, una vulnerabilidad del kernel afecta a **todos los contenedores** del host simultáneamente. El examen puede presentarlo como un riesgo limitado al contenedor afectado.

⚠️ **[DevOps speed — vector de ataque]**
La característica de efemeralidad de los contenedores (se ejecutan y eliminan rápido) es un vector de ataque: los atacantes pueden lanzar ataques y **ocultarse sin instalar malware**. No es solo una ventaja operativa.

⚠️ **[Registries — ejemplos]**
Docker Hub, Amazon ECR (Elastic Container Registry) y Docker Trusted Registry (DTR) son registries. El examen puede confundir DTR con Docker Swarm o Docker Engine.

⚠️ **[Orquestadores — acción ante actualización]**
Cuando una app necesita actualizarse, los contenedores existentes son **destruidos** y se crean **nuevos desde las imágenes actualizadas**. No se actualiza el contenedor en ejecución.

---

## 3. Nemotécnicos

### 5 Tiers de la arquitectura de contenedores
**"Developers Test Registries Orchestrate Hosts"**
→ **D**eveloper machines | **T**esting & accreditation | **R**egistries | **O**rchestrators | **H**osts

### 3 Fases del ciclo de vida del contenedor
**"Crear → Guardar → Desplegar"**
→ Image Creation/Testing/Accreditation → Image Storage/Retrieval → Container Deployment/Management

### 5 Native network drivers de Docker
**"H-B-O-M-N"** → *"HBO tiene Mucho Nada"*
→ **H**ost | **B**ridge | **O**verlay | **M**ACVLAN | **N**one

### 3 Remote network drivers Docker (community)
**"CWK"** → *"Cisco Weave Kuryr"*
→ **C**ontiv (Cisco) | **W**eave | **K**uryr (usa Neutron/OpenStack)

### Master components de Kubernetes (5)
**"API Etcd Scheduler Controller Cloud"**
→ kube-**api**server | **etcd** cluster | kube-**scheduler** | kube-**controller**-manager | **cloud**-controller-manager

### Node components de Kubernetes (3)
**"KKC"** → *"Kubelet Kube-proxy Container runtime"*
→ **K**ubelet | **K**ube-proxy | **C**ontainer Runtime

### CNM — 3 construcciones
**"SEN"** → *"Sandbox Endpoint Network"*
→ **S**andbox (config red) | **E**ndpoint (conexión a red) | **N**etwork (colección de endpoints)

### Tipos de clústeres (3)
**"HaLo HPC"** → **HA**/fail-over | **Lo**ad balancing | **HPC**

---

## 4. Flashcards

**Q:** ¿Cuál es la diferencia de arranque entre contenedores y VMs?
**A:** Contenedores arrancan en segundos. VMs arrancan en minutos.

---

**Q:** ¿Qué tipo de virtualización usan los contenedores frente a las VMs?
**A:** Contenedores: virtualización basada en SO (OS-level). VMs: virtualización basada en hardware.

---

**Q:** ¿Cuáles son los tres componentes del Docker Engine?
**A:** Server (daemon `dockerd`), REST API, y Client CLI.

---

**Q:** ¿Mediante qué protocolo se comunican Docker client y Docker daemon?
**A:** REST API, tanto en local como en conexión a daemon remoto.

---

**Q:** ¿Cuáles son los 5 native network drivers de Docker?
**A:** Host, Bridge, Overlay, MACVLAN, None.

---

**Q:** ¿Qué driver de Docker crea una red virtual para conectar contenedores en múltiples clouds?
**A:** Weave (remote driver de community).

---

**Q:** ¿Qué driver de Docker fue creado por Cisco para políticas de seguridad en microservicios multi-tenant?
**A:** Contiv.

---

**Q:** ¿Cuál es el único componente de Kubernetes que interactúa con el etcd cluster?
**A:** Kube-apiserver.

---

**Q:** ¿Qué almacena el etcd cluster en Kubernetes?
**A:** Datos del cluster, detalles de service discovery y objetos API. Es un almacenamiento key-value distribuido y consistente.

---

**Q:** ¿Qué hace kubelet y qué contenedores NO gestiona?
**A:** Garantiza que los contenedores de un pod estén corriendo y sanos. No gestiona contenedores que no hayan sido creados por Kubernetes.

---

**Q:** Un contenedor consume todos los recursos del sistema, impidiendo que otros contenedores funcionen. ¿Qué tipo de ataque/problema representa?
**A:** Noisy neighboring container → genera un ataque de DoS sobre los contenedores vecinos.

---

**Q:** ¿Bajo qué condición puede un contenedor hacer breakout al SO del host?
**A:** Cuando el contenedor se ejecuta como root; puede escalar privilegios y acceder al SO del host.

---

**Q:** ¿Por qué los kernel exploits son especialmente peligrosos en entornos de contenedores?
**A:** Porque todos los contenedores comparten el kernel del host; una vulnerabilidad del kernel afecta a todos los contenedores simultáneamente.

---

**Q:** ¿Qué hace kube-scheduler en el cluster de Kubernetes?
**A:** Escanea pods recién creados y les asigna un nodo según recursos disponibles, localidad de datos, y restricciones de hardware/software/política.

---

**Q:** ¿Cuáles son los tres registries de contenedores mencionados en el libro?
**A:** Docker Hub, Amazon Elastic Container Registry (ECR), Docker Trusted Registry (DTR).

---

**Q:** ¿Qué ocurre con los contenedores en ejecución cuando se actualiza una aplicación en Kubernetes?
**A:** Los contenedores existentes son destruidos y se crean nuevos contenedores desde las imágenes actualizadas.

---

**Q:** ¿Cuáles son las tres fases del ciclo de vida de un contenedor?
**A:** 1) Image Creation, Testing and Accreditation; 2) Image Storage and Retrieval; 3) Container Deployment and Management.

---

**Q:** ¿Qué característica del ciclo de vida de los contenedores aprovechan los atacantes para ocultarse sin instalar malware?
**A:** La efemeralidad (DevOps speed): los contenedores se ejecutan, detienen y eliminan rápidamente, dificultando la detección forense.

---

**Q:** ¿Qué es el CNM en Docker Networking y cuáles son sus tres construcciones?
**A:** Container Network Model. Construcciones: Sandbox (config stack de red), Endpoint (conexión a red), Network (colección de endpoints).

---

**Q:** ¿Qué diferencia un pod de un contenedor en Kubernetes?
**A:** Un pod es un grupo de contenedores gestionados conjuntamente en un nodo worker. El contenedor es la unidad individual dentro del pod.

---

## 5. Confusión frecuente

**Contenedores vs. VMs — seguridad**
- VMs: aislamiento **completo** (cada VM tiene su propio SO) → más seguras en términos de aislamiento.
- Contenedores: aislamiento a **nivel de proceso** → parcialmente seguros. El kernel compartido es el vector crítico.
- Criterio: si la pregunta habla del nivel de aislamiento o del riesgo de kernel exploits → VMs son más seguras. Si habla de que el crash de un contenedor no afecta a otros → contenedores son robustos entre sí, pero no frente al host.

---

**Docker Swarm vs. Kubernetes**
- Docker Swarm: modo del Docker engine para gestionar múltiples Docker engines; más simple, integrado en Docker.
- Kubernetes (K8s): plataforma de orquestación independiente desarrollada por Google; más compleja, con arquitectura master/worker, pods, etcd, y gestión avanzada de microservicios.
- Criterio: si la pregunta menciona orquestación empresarial, self-healing, bin packing automático o gestión de clusters complejos → Kubernetes. Si menciona gestión de múltiples Docker engines de forma nativa → Docker Swarm.

---

**Kube-controller-manager vs. Cloud-controller-manager**
- Kube-controller-manager: ejecuta controllers internos del cluster (node, endpoint, replication, tokens).
- Cloud-controller-manager: ejecuta controllers que interactúan con **proveedores cloud externos**. Permite evolución independiente del código de K8s y el cloud.
- Criterio: si la interacción involucra recursos cloud externos → cloud-controller-manager.

---

**Overlay driver vs. MACVLAN driver**
- Overlay: comunicación entre contenedores **a través de la infraestructura de red física** (multi-host).
- MACVLAN: crea una conexión entre la interfaz del contenedor y la interfaz del host usando Linux MACVLAN bridge mode (nivel de red más bajo, acceso directo a la red física).
- Criterio: para comunicación entre hosts en distintas redes físicas → Overlay. Para conectividad directa a nivel de interfaz de red → MACVLAN.

---

**Registries vs. Orchestrators (roles en el ciclo de vida)**
- Registries (Tier-3): almacenan y distribuyen imágenes. Solo guardan; no ejecutan nada.
- Orchestrators (Tier-4): extraen imágenes de los registries, las transforman en contenedores, los despliegan, monitorizan y gestionan.
- Criterio: si la acción es almacenar/versionar/catalogar → Registry. Si es desplegar/gestionar/monitorizar → Orchestrator.

---

**Cgroups misconfiguration vs. Container breakout**
- Cgroups misconfiguration: afecta a la **asignación de recursos** → DoS por contención.
- Container breakout: afecta al **aislamiento de seguridad** → escalada de privilegios al host (requiere ejecución como root).
- Criterio: si el escenario es agotamiento de CPU/memoria → cgroups. Si es acceso no autorizado al host → breakout.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un analista de seguridad detecta que un contenedor Docker ha sido comprometido y el atacante ha logrado acceder al sistema operativo del host. ¿Cuál fue la condición necesaria para que este container breakout fuera posible?

A) El contenedor usaba el driver de red None  
B) El contenedor se ejecutaba como root  
C) El contenedor usaba almacenamiento en volúmenes  
D) El contenedor no tenía acceso a internet  

**Respuesta correcta: B**
El **container breakout al host** requiere que el contenedor se ejecute como **root**. En ese caso, el proceso dentro del contenedor puede escalar privilegios y acceder al SO del host. Ejecutar contenedores con usuarios no privilegiados es la contramedida principal.

---

**P2.** Una organización observa que varios contenedores en el mismo host han dejado de responder. La investigación revela que un contenedor consume el 100% de CPU y memoria. ¿Qué tipo de ataque/problema describe esto?

A) Container breakout al host  
B) Kernel exploit  
C) Noisy neighboring container → DoS  
D) Bypassing isolation  

**Respuesta correcta: C**
Un **noisy neighboring container** es un contenedor que agota los recursos del sistema (CPU, memoria, red), causando un ataque de **DoS** sobre los demás contenedores del host. Se mitiga configurando correctamente los **cgroups** para limitar los recursos asignados a cada contenedor.

---

**P3.** Un arquitecto de Kubernetes está diseñando el cluster. ¿Cuál es el único componente del master que puede comunicarse directamente con el etcd cluster para leer y escribir el estado del cluster?

A) Kube-scheduler  
B) Kube-controller-manager  
C) Cloud-controller-manager  
D) Kube-apiserver  

**Respuesta correcta: D**
El **kube-apiserver** es el **único componente de Kubernetes que interactúa con el etcd cluster**. Todos los demás componentes (scheduler, controller-manager) acceden al estado del cluster a través del kube-apiserver, nunca directamente a etcd.

---

**P4.** Un pentester analiza la arquitectura de contenedores de una empresa. Identifica que los contenedores se almacenan en Docker Hub y Amazon ECR antes de ser desplegados. ¿En qué tier de la arquitectura de 5 niveles se encuentran estos almacenes?

A) Tier-2 — Testing and Accreditation  
B) Tier-3 — Registries  
C) Tier-4 — Orchestrators  
D) Tier-5 — Hosts  

**Respuesta correcta: B**
**Tier-3 son los Registries** — almacenan y distribuyen imágenes (Docker Hub, Amazon ECR, Docker Trusted Registry/DTR). Tier-2 es testing y acreditación. Tier-4 son los orquestadores que extraen imágenes de los registries y despliegan contenedores.

---

**P5.** Un administrador de red examina la configuración de Docker networking. Necesita un driver que permita que contenedores en diferentes hosts físicos se comuniquen entre sí a través de la infraestructura de red existente. ¿Qué native network driver debe usar?

A) Bridge  
B) Host  
C) MACVLAN  
D) Overlay  

**Respuesta correcta: D**
El driver **Overlay** permite la comunicación entre contenedores **a través de la infraestructura de red física** en entornos multi-host. Bridge crea un puente en el mismo host. Host usa directamente el stack de red del host. MACVLAN conecta interfaces de contenedor con interfaces del host a nivel de red.

---

**P6.** Un investigador forense analiza un ataque a una plataforma de contenedores. El atacante aprovechó que todos los contenedores del host compartían el mismo kernel para comprometer simultáneamente todos los contenedores. ¿Qué tipo de vulnerabilidad explotó?

A) Cgroups misconfiguration  
B) Pod Security Policy misconfiguration  
C) Kernel exploit  
D) Network-based attack via raw sockets  

**Respuesta correcta: C**
Un **kernel exploit** en entornos de contenedores es especialmente peligroso porque todos los contenedores **comparten el kernel del host**. Una vulnerabilidad del kernel afecta simultáneamente a **todos los contenedores** del host. Esta es la diferencia fundamental de seguridad frente a las VMs, que tienen kernels independientes.

---

**P7.** ¿Cuál es la diferencia de tiempo de arranque entre contenedores y máquinas virtuales, y qué tipo de virtualización usa cada uno?

A) Contenedores: minutos, virtualización hardware; VMs: segundos, virtualización OS  
B) Contenedores: segundos, virtualización OS; VMs: minutos, virtualización hardware  
C) Ambos arrancan en segundos; la diferencia es solo el tipo de aislamiento  
D) Contenedores: milisegundos, sin virtualización; VMs: minutos, virtualización hardware  

**Respuesta correcta: B**
**Contenedores** arrancan en **segundos** y usan **virtualización basada en SO** (OS-level virtualization). **VMs** arrancan en **minutos** y usan **virtualización basada en hardware**. Los contenedores son más ligeros (megabytes vs. gigabytes) pero tienen menor aislamiento al compartir el kernel.

---

**P8.** Un equipo DevOps observa que en su pipeline de CI/CD algunos contenedores se ejecutan y eliminan tan rápidamente que el equipo de seguridad no puede analizarlos. ¿Qué security challenge de contenedores describe esta situación y cuál es el riesgo específico?

A) Lack of visibility — el container engine abstrae actividades y dificulta el tracking  
B) DevOps speed — los atacantes pueden lanzar ataques y ocultarse sin instalar malware  
C) Large attack surface — múltiples apps en el mismo host  
D) Noisy neighboring containers — los contenedores efímeros consumen más recursos  

**Respuesta correcta: B**
La **DevOps speed** (efemeralidad de los contenedores) es un security challenge: los contenedores se ejecutan, detienen y eliminan rápidamente. Los atacantes aprovechan esto para **lanzar ataques y ocultarse sin necesidad de instalar malware persistente**, ya que el contenedor desaparece antes de que el equipo de seguridad pueda analizarlo.
