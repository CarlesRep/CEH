# M19_01 — Cloud Computing Concepts
**Módulo 19 / Subapartado 1 — Cloud Computing Concepts**

---

## 1. Conceptos y definiciones

### Características esenciales del cloud computing (NIST)

El cloud no es simplemente "hosting remoto". Sus características fundamentales definen por qué es un modelo distinto:

- **On-demand self-service**: el consumidor aprovisiona recursos sin intervención humana del proveedor.
- **Broad network access**: acceso a través de mecanismos estándar desde cualquier plataforma (laptop, móvil, PDA).
- **Resource pooling**: modelo multi-tenant; recursos físicos/virtuales reasignados dinámicamente. El consumidor no controla ni conoce la ubicación exacta.
- **Rapid elasticity**: escalado instantáneo, percibido como ilimitado por el consumidor.
- **Measured service**: pay-per-use con monitorización transparente (almacenamiento, CPU, ancho de banda).

Las características adicionales del libro (distributed storage, automated management, virtualization technology) son extensiones prácticas de las anteriores, no parte de la definición NIST canónica.

---

### 🔴 Modelos de servicio: IaaS / PaaS / SaaS + extensiones

La lógica fundamental es el nivel de control del suscriptor vs. responsabilidad del proveedor:

| Modelo | El suscriptor gestiona | El proveedor gestiona | Ejemplos clave |
|--------|----------------------|----------------------|----------------|
| **IaaS** | SO, middleware, apps, datos | Virtualización, servidores, almacenamiento, red | Amazon EC2, Microsoft OneDrive, Rackspace |
| **PaaS** | Apps desplegadas, config. del entorno | Todo lo anterior + SO + middleware | Google App Engine, Salesforce, Microsoft Azure |
| **SaaS** | Solo los datos de uso | Todo | Google Docs/Calendar, Salesforce CRM, Freshbooks |

**Extensiones del modelo as-a-Service:**

| Modelo | Clave conceptual | Ejemplos |
|--------|-----------------|---------|
| **IDaaS** | SSO, MFA, IGA, access management como servicio | OneLogin, Okta, Azure AD |
| **SECaaS** | Seguridad (pentest, IDS, anti-malware, SIEM) como servicio; basado en SaaS | eSentire MDR, Foundstone |
| **CaaS** | Contenedores y clusters como servicio; hereda IaaS + PaaS | Amazon EC2, Google Kubernetes Engine (GKE) |
| **FaaS** | Serverless; paga solo cuando la función se ejecuta; microservicios | AWS Lambda, Google Cloud Functions, Azure Functions, Oracle Functions |
| **XaaS** | Todo como servicio; engloba NaaS, STaaS, TaaS, MaaS, DRaaS | NetApp, AWS Elastic Beanstalk, Heroku, Apache Stratos |
| **FWaaS** | Firewall como servicio; filtrado, packet filtering, network analyzing, IPsec, detección malware | Zscaler Cloud Firewall, Fortinet, Cisco, Sophos |
| **DaaS** | Escritorios virtuales y apps on-demand; multi-tenancy, pay-as-you-need | Amazon WorkSpaces, Citrix Managed Desktops, Azure Windows Virtual Desktop |
| **MBaaS** | Backend para apps móviles vía API/SDK; push notifications, user management, geolocation | Firebase, AWS Amplify, Apple CloudKit, Backendless |
| **MaaS (Machines)** | También llamado EaaS (Equipment-as-a-Service); fabricantes venden/arriendan máquinas y reciben % de beneficios | — |

**FaaS crítico para el examen**: la función *se apaga* cuando no se usa → sin cargos. Latencia alta como contrapartida.

---

### Responsabilidades compartidas 🔴

La separación de responsabilidades es proporcional al nivel de abstracción:
- **IaaS**: el suscriptor asume SO, middleware, apps, datos. Mayor superficie de ataque para el suscriptor.
- **PaaS**: el suscriptor solo gestiona la app y opcionalmente la config del entorno.
- **SaaS**: el proveedor gestiona prácticamente todo; el suscriptor solo usa.

La separación de funciones **previene conflictos de interés, fraude, abuso y error**, y restringe la influencia individual.

---

### 🔴 Modelos de despliegue

| Modelo | Tenants | Control | Seguridad | Coste | Ejemplos |
|--------|---------|---------|-----------|-------|---------|
| **Public** | Múltiples (cualquiera) | Proveedor | Menor | Menor | Amazon EC2, Google App Engine, Windows Azure, IBM Bluemix |
| **Private** | Uno (organización) | Organización | Mayor | Mayor | BMC Software, VMware vRealize Suite, SAP Cloud Platform |
| **Community** | Múltiples (comunidad específica) | Organizaciones participantes o MSP | Media | Medio (< private) | Cisco Cloud Solutions, Salesforce Health Cloud |
| **Hybrid** | Mixto | Compartido | Alta (componente privada) | Variable | Microsoft Azure, Logicalis |
| **Multi Cloud** | Múltiples proveedores distintos | Un interfaz propietario | Compleja | Variable | Microsoft Azure Arc, Google Cloud Anthos |
| **Distributed** | Geográficamente distribuido, un plano de control | Centralizado | Estricto | Alto (infraestructura red) | Google Distributed Cloud, Cloudflare CDN |
| **Poly Cloud** | Varios servicios de distintos clouds en una plataforma | Usuario elige feature por cloud | — | Alto R&D | GCP + AWS |

**Diferencia Multi Cloud vs. Poly Cloud**: Multi Cloud usa distintos proveedores para distintas cargas pero con un interfaz común. Poly Cloud toma *características específicas* de cada cloud en una sola plataforma.

**Hybrid Cloud**: dos o más clouds (privada/pública/community) que siguen siendo entidades únicas pero enlazadas. Ejemplo canónico: datos operacionales del cliente en privada, actividades no críticas en pública.

**Compliance en Private Cloud**: Sarbanes-Oxley, PCI DSS y HIPAA son más fáciles de cumplir en cloud privada.

---

### Arquitectura de almacenamiento cloud

Tres capas:
- **Front-end**: acceso del usuario final + APIs de gestión.
- **Middleware**: deduplicación y replicación de datos.
- **Back-end**: hardware físico.

Características: fault-tolerant por redundancia, durable por replicación. Servicios de object storage: Amazon S3, Oracle Cloud Storage, Microsoft Azure Storage, OpenStack Swift.

---

### NIST Reference Architecture — 5 actores

| Actor | Rol |
|-------|-----|
| **Cloud Consumer** | Usa los servicios; negocia SLA con el proveedor |
| **Cloud Provider** | Adquiere y gestiona la infraestructura; provee servicios |
| **Cloud Carrier** | Intermediario de conectividad y transporte entre proveedor y consumidor |
| **Cloud Auditor** | Examen independiente de controles de seguridad, privacidad y rendimiento |
| **Cloud Broker** | Gestiona uso, rendimiento y entrega; intermedia entre consumidor y proveedor |

**Servicios del Cloud Broker**:
1. **Service Intermediation**: mejora una función con capacidades adicionales (valor añadido).
2. **Service Aggregation**: combina múltiples servicios en uno nuevo.
3. **Service Arbitrage**: como aggregation pero sin servicios fijos; el broker elige entre múltiples agencias.

**Servicios disponibles por modelo** (según NIST):
- **PaaS**: DB, BI, despliegue de aplicaciones, desarrollo y testing, integración.
- **IaaS**: almacenamiento, gestión de servicios, CDN, hosting de plataformas, backup/recovery, cómputo.
- **SaaS**: RRHH, ERP, ventas, CRM, colaboración, gestión documental, email, ofimática, gestión de contenidos, servicios financieros, redes sociales.

---

### Fog Computing vs. Edge Computing vs. Cloud

**Fog computing**: capa distribuida e independiente entre IoT y cloud. Nodos en el borde de red; analítica a corto plazo. También llamado *intelligent gateway*. Procesa en LAN.

**Edge computing**: subconjunto de fog. La inteligencia está en dispositivos como *programmable automation controllers*. Procesa en el propio dispositivo edge (no en LAN). Para operaciones urgentes en **milisegundos**.

| Feature | Cloud | Fog | Edge |
|---------|-------|-----|------|
| Velocidad | Alta (depende VM) | Mayor que cloud | Mayor que fog |
| Latencia | Alta | Baja | Baja |
| Integración datos | Múltiples fuentes | Múltiples fuentes y dispositivos | Fuentes limitadas |
| Capacidad | Sin reducción de datos | Reduce datos enviados a cloud | Reduce datos enviados a fog |
| Responsividad | Baja | Alta | Alta |
| Seguridad | Menos seguro que fog | Muy seguro | Seguridad customizada |
| Escala (nodos) | Miles de servidores | Millones de nodos | Miles de millones de endpoints |

---

### Cloud vs. Grid Computing

| Cloud Computing | Grid Computing |
|----------------|---------------|
| Arquitectura cliente-servidor | Arquitectura distribuida |
| Mayor escalabilidad | Escalabilidad estándar |
| Recursos centralizados | Recursos colaborativos |
| Más flexible | Menos flexible |
| El proveedor posee los servidores | La organización posee y gestiona los grids |
| IaaS, PaaS, SaaS | Distributed information/computing/pervasive systems |
| Protocolos web estándar | Grid middleware |
| Pay-as-you-go | Gratuito para el usuario |
| Orientado a servicio | Orientado a aplicación |
| Soporta interoperabilidad: NO (vendor lock-in) | Soporta interoperabilidad: SÍ |

---

## 2. Exam Traps ⚠️

⚠️ **[IaaS vs. PaaS — seguridad]**
El examen puede afirmar que IaaS tiene *menor* riesgo de seguridad. Incorrecto: **PaaS tiene menor riesgo que IaaS** porque el proveedor gestiona más capas. El suscriptor de IaaS es responsable del SO y apps.

⚠️ **[CaaS — herencia de modelos]**
CaaS no es solo IaaS ni solo PaaS. El examen puede preguntar de qué hereda: hereda características de **ambos, IaaS y PaaS**.

⚠️ **[FaaS — cobro]**
FaaS incurre en cargos **únicamente cuando la función se ejecuta**. Cuando no está en uso, la infraestructura se apaga y **no se cobra**. El examen puede confundirlo con un modelo de suscripción mensual.

⚠️ **[MaaS — ambigüedad]**
Existen dos "MaaS" en el libro: **Malware-as-a-Service** (dentro de XaaS) y **Machines-as-a-Service** (también llamado EaaS). Si el examen pregunta por MaaS en contexto de cloud model de fabricantes, es EaaS/Machines.

⚠️ **[Multi Cloud vs. Poly Cloud]**
Multi Cloud: múltiples proveedores gestionados desde **un único interfaz**. Poly Cloud: características de distintos clouds en **una sola plataforma**, el usuario elige la feature por cloud. El examen puede tratarlos como sinónimos — no lo son.

⚠️ **[Distributed Cloud vs. Multi Cloud]**
Distributed Cloud: geograficamente distribuido pero con **un solo plano de control centralizado**. Multi Cloud: varios proveedores. Son modelos distintos.

⚠️ **[Fog vs. Edge — dónde procesa]**
Fog: procesa en la **LAN** (intelligent gateway). Edge: procesa en el **propio dispositivo** (programmable automation controllers). El examen puede invertir la relación.

⚠️ **[Edge computing — operaciones]**
Edge está diseñado para operaciones urgentes en **milisegundos**. No es un modelo de propósito general; si el escenario habla de latencia ultrabaja y dispositivos IoT, edge es la respuesta.

⚠️ **[Community Cloud — seguridad]**
Community cloud tiene seguridad **moderada**, no alta. Otros tenants de la comunidad pueden acceder a datos. El examen puede presentarlo como más seguro que private cloud — es falso.

⚠️ **[Cloud Broker — Service Arbitrage vs. Aggregation]**
Service Aggregation combina servicios fijos. Service Arbitrage también combina pero **sin fijar los servicios agregados**; el broker elige dinámicamente entre múltiples agencias. La diferencia clave es la flexibilidad.

⚠️ **[IDaaS — fallo de servidor]**
La desventaja principal de IDaaS es que un **fallo en un solo servidor puede interrumpir el servicio** o crear redundancia en otros servidores de autenticación. El examen puede confundirlo con un fallo total del sistema.

⚠️ **[Grid Computing — pago]**
Grid computing es **gratuito para el usuario** (no hay modelo pay-as-you-go). Cloud usa pay-as-you-go. El examen puede presentar grid como de pago.

⚠️ **[Private Cloud — compliance]**
Sarbanes-Oxley, PCI DSS y HIPAA son **más fáciles de cumplir en private cloud**. El examen puede sugerir que hybrid o community son equivalentes — no lo son para compliance regulatoria.

---

## 3. Nemotécnicos

### Las 8 características del cloud (NIST + libro)
**"On-Demand Rapid Automation Broad Resource Measures Virtual Distribution"**
→ **O**n-demand self-service | **R**apid elasticity | **A**utomated management | **B**road network access | **R**esource pooling | **M**easured service | **V**irtualization technology | **D**istributed storage

### Los 5 actores NIST
**"Consumers Pay Crazy Amounts Briskly"**
→ **C**onsumer | **P**rovider | **C**arrier | **A**uditor | **B**roker

### Modelos de despliegue (orden de control ascendente)
**Public → Community → Hybrid → Private** (de menos a más control/coste/seguridad)
Multi Cloud, Distributed y Poly Cloud son extensiones fuera de los 4 estándar.

### Servicios del broker (3)
**"I-A-A"** → Intermediation / Aggregation / Arbitrage
- Intermediation = **mejora** una función
- Aggregation = **combina** (fijo)
- Arbitrage = **combina** (flexible, elige entre agencias)

### FaaS vs. SaaS — cuándo pagar
- **SaaS**: pago por uso, suscripción, publicidad, o uso compartido
- **FaaS**: paga **solo cuando se ejecuta**; se apaga cuando no se usa

### Cloud vs. Fog vs. Edge — escala de nodos
**"Miles → Millones → Miles de millones"**
→ Cloud (miles de servidores) | Fog (millones de nodos) | Edge (miles de millones de endpoints)

---

## 4. Flashcards

**Q:** ¿Cuál es la diferencia entre IaaS, PaaS y SaaS en términos de responsabilidad del suscriptor?
**A:** IaaS: el suscriptor gestiona SO, middleware, apps y datos. PaaS: solo apps y config del entorno. SaaS: solo el uso de la aplicación.

---

**Q:** ¿Qué servicio cloud proporciona SSO, MFA e IGA gestionados por terceros?
**A:** IDaaS (Identity-as-a-Service). Ejemplos: OneLogin, Okta, Microsoft Azure AD.

---

**Q:** ¿Qué característica diferencia FaaS de otros modelos de servicio cloud?
**A:** Serverless: la infraestructura se apaga cuando la función no se ejecuta y no genera cargos. Se paga únicamente por ejecución.

---

**Q:** ¿De qué modelos hereda características CaaS?
**A:** De IaaS y PaaS simultáneamente.

---

**Q:** Un escenario describe un entorno cloud donde múltiples organizaciones del sector sanitario comparten infraestructura con requisitos de compliance común. ¿Qué modelo es?
**A:** Community Cloud.

---

**Q:** ¿Cuáles son los cinco actores del NIST Cloud Reference Architecture?
**A:** Cloud Consumer, Cloud Provider, Cloud Carrier, Cloud Auditor, Cloud Broker.

---

**Q:** ¿Qué actor NIST proporciona conectividad y transporte entre proveedor y consumidor?
**A:** Cloud Carrier.

---

**Q:** ¿Cuáles son los tres tipos de servicios que ofrece un Cloud Broker?
**A:** Service Intermediation (mejora funciones), Service Aggregation (combina servicios fijos), Service Arbitrage (combina servicios de múltiples agencias sin fijación).

---

**Q:** ¿En qué se diferencia Service Arbitrage de Service Aggregation?
**A:** Arbitrage combina servicios de múltiples agencias sin que los servicios agregados estén fijados; el broker elige dinámicamente. Aggregation combina servicios predefinidos.

---

**Q:** ¿En qué capa de la arquitectura de almacenamiento cloud se realiza la deduplicación y replicación de datos?
**A:** En la capa middleware.

---

**Q:** ¿Cuál es la principal ventaja del modelo Multi Cloud frente al vendor lock-in?
**A:** Baja probabilidad de vendor lock-in al distribuir cargas entre múltiples proveedores.

---

**Q:** ¿Qué modelo cloud usan los fabricantes para vender o arrendar máquinas y recibir un porcentaje de los beneficios generados?
**A:** MaaS (Machines-as-a-Service), también denominado EaaS (Equipment-as-a-Service).

---

**Q:** ¿Dónde procesa los datos fog computing versus edge computing?
**A:** Fog: en la LAN mediante un intelligent gateway. Edge: en el propio dispositivo (programmable automation controllers).

---

**Q:** ¿Para qué tipo de operaciones está diseñado edge computing?
**A:** Para operaciones urgentes y de baja latencia con tiempos de respuesta en el orden de milisegundos.

---

**Q:** ¿Qué compliance regulatoria es más fácil de cumplir en cloud privada?
**A:** Sarbanes-Oxley, PCI DSS e HIPAA.

---

**Q:** ¿Cuál es la diferencia entre Multi Cloud y Distributed Cloud?
**A:** Multi Cloud: múltiples proveedores gestionados desde un único interfaz propietario. Distributed Cloud: un solo entorno cloud centralizado distribuido geográficamente con un único plano de control.

---

**Q:** ¿Qué modelo cloud tiene mayor riesgo de seguridad para el suscriptor entre IaaS y PaaS?
**A:** IaaS, porque el suscriptor es responsable del SO, middleware y aplicaciones, mientras que en PaaS el proveedor gestiona esas capas.

---

**Q:** ¿Qué tecnología usa FWaaS además de packet filtering?
**A:** Network analyzing, IPsec, detección de malware y análisis de datos avanzados.

---

**Q:** ¿Cuál es la escala de nodos en Cloud, Fog y Edge respectivamente?
**A:** Cloud: miles de servidores. Fog: millones de nodos. Edge: miles de millones de endpoints.

---

**Q:** ¿Qué modelo cloud distribuye activos y aplicaciones entre distintos proveedores mediante un interfaz propietario único?
**A:** Multi Cloud. Ejemplos: Microsoft Azure Arc, Google Cloud Anthos.

---

## 5. Confusión frecuente

**Fog Computing vs. Edge Computing**
- Fog: la inteligencia y el procesamiento están en un nodo/gateway en la **LAN**, entre los dispositivos IoT y el cloud. Millones de nodos.
- Edge: la inteligencia está **en el propio dispositivo** (programmable automation controllers). Miles de millones de endpoints. Es un subconjunto de fog.
- Criterio de decisión: si el escenario menciona procesamiento en el dispositivo mismo o latencia de milisegundos → Edge. Si menciona gateway inteligente o LAN → Fog.

---

**Multi Cloud vs. Poly Cloud**
- Multi Cloud: distintos proveedores para distintas cargas, gestionados desde **un único interfaz**.
- Poly Cloud: distintos clouds para distintas **características/features** en una sola plataforma; el usuario elige la feature específica de cada cloud.
- Criterio: si el escenario habla de features específicas de cada cloud combinadas → Poly. Si habla de distribución de cargas entre proveedores → Multi.

---

**Service Aggregation vs. Service Arbitrage (Cloud Broker)**
- Aggregation: los servicios combinados están **predefinidos y fijos**.
- Arbitrage: similar pero el broker puede **elegir entre múltiples agencias** sin compromiso fijo.
- Criterio: si hay flexibilidad dinámica en la elección del servicio → Arbitrage.

---

**IaaS vs. PaaS — quién asume el riesgo de seguridad del software**
- IaaS: el suscriptor. El software que instala y ejecuta es su responsabilidad.
- PaaS: el proveedor gestiona el middleware y el SO; el suscriptor tiene menor riesgo.
- Criterio: a más capas gestionadas por el proveedor → menor riesgo de seguridad de software para el suscriptor.

---

**Public Cloud vs. Community Cloud**
- Public: disponible para cualquier usuario; el proveedor controla todo.
- Community: solo para organizaciones de una comunidad específica con requisitos comunes (compliance, seguridad, regulación); puede ser gestionado por las propias organizaciones o un MSP.
- Criterio: si el escenario menciona un sector específico o requisitos regulatorios comunes entre varias organizaciones → Community.

---

**IDaaS vs. SECaaS**
- IDaaS: identidad y acceso (SSO, MFA, IGA). Gestión de cuentas de usuario.
- SECaaS: servicios de seguridad más amplios (pentest, IDS, anti-malware, SIEM). Basado en SaaS.
- Criterio: si la pregunta menciona autenticación o gestión de identidades → IDaaS. Si menciona detección de intrusos, malware o SIEM → SECaaS.

---

## 6. Preguntas de Práctica — Formato CEH

---

### Pregunta 1 — Modelos de servicio: responsabilidad
Una empresa contrata un servicio cloud y solo gestiona sus propias aplicaciones desplegadas y la configuración del entorno de ejecución. El proveedor gestiona el sistema operativo, middleware, servidores, almacenamiento y virtualización. ¿Qué modelo de servicio describe esto?

A) IaaS  
B) SaaS  
C) PaaS  
D) FaaS  

> **Respuesta correcta: C** — En **PaaS (Platform-as-a-Service)**, el suscriptor solo gestiona las **aplicaciones desplegadas y la configuración del entorno**, mientras el proveedor gestiona SO, middleware, servidores, almacenamiento y virtualización. En IaaS el suscriptor también gestiona el SO. En SaaS solo usa la aplicación. En FaaS paga solo por ejecución de funciones.

---

### Pregunta 2 — FaaS y modelo de cobro
Un equipo de desarrollo usa funciones cloud que se ejecutan solo cuando llegan peticiones y permanecen inactivas el resto del tiempo. En períodos de baja demanda (ej. noches), el consumo de recursos cae a cero. ¿Cuál es el modelo de servicio y cuánto se cobra durante la inactividad?

A) SaaS; se cobra suscripción mensual fija  
B) PaaS; se cobra por los recursos reservados aunque no se usen  
C) FaaS (Serverless); no se cobra durante la inactividad porque la infraestructura se apaga  
D) IaaS; se cobra por las instancias EC2 aunque estén paradas  

> **Respuesta correcta: C** — **FaaS (Function-as-a-Service) / Serverless**: la infraestructura **se apaga cuando la función no se ejecuta** y **no se generan cargos**. Solo se paga cuando la función está en ejecución. La contrapartida es mayor latencia al arrancar ("cold start"). Ejemplos: AWS Lambda, Google Cloud Functions, Azure Functions.

---

### Pregunta 3 — Modelos de despliegue
Una organización del sector sanitario necesita compartir infraestructura cloud con otras organizaciones hospitalarias que tienen requisitos de compliance HIPAA comunes, con costes compartidos y seguridad moderada-alta. ¿Qué modelo de despliegue es el más adecuado?

A) Public Cloud  
B) Private Cloud  
C) Community Cloud  
D) Distributed Cloud  

> **Respuesta correcta: C** — **Community Cloud**: múltiples organizaciones de una **comunidad específica** (sector sanitario, financiero, gubernamental) comparten infraestructura con requisitos comunes (compliance, seguridad, regulación). Costes menores que private cloud pero mayor seguridad que public cloud. Puede ser gestionado por las propias organizaciones o un MSP.

---

### Pregunta 4 — NIST Reference Architecture
Una empresa contrata a una organización independiente para evaluar los controles de seguridad, privacidad y rendimiento de su proveedor cloud. ¿Cuál de los cinco actores NIST describe esta organización?

A) Cloud Carrier  
B) Cloud Broker  
C) Cloud Consumer  
D) Cloud Auditor  

> **Respuesta correcta: D** — El **Cloud Auditor** realiza un **examen independiente** de los controles de seguridad, privacidad y rendimiento del servicio cloud. El Cloud Broker intermedia entre proveedor y consumidor. El Cloud Carrier proporciona conectividad. El Cloud Consumer usa los servicios.

---

### Pregunta 5 — Fog vs Edge Computing
Un fabricante necesita tomar decisiones de control en sus máquinas de producción con latencia de milisegundos, procesando los datos directamente en los equipos industriales (programmable automation controllers) sin enviarlos a ningún servidor remoto ni gateway. ¿Qué paradigma computacional debe usar?

A) Cloud Computing  
B) Fog Computing  
C) Edge Computing  
D) Grid Computing  

> **Respuesta correcta: C** — **Edge Computing**: la inteligencia y el procesamiento están **en el propio dispositivo** (programmable automation controllers). Diseñado para operaciones urgentes en **milisegundos**. No envía datos a ningún nodo o gateway intermedio. Fog Computing procesa en un gateway/nodo de la LAN (no en el dispositivo). Cloud tiene latencia alta.

---

### Pregunta 6 — Service Arbitrage vs Service Aggregation
Un Cloud Broker gestiona servicios cloud para sus clientes. En un contrato combina servicios de 5 proveedores diferentes que están predefinidos. En otro contrato, puede elegir dinámicamente entre 12 proveedores según el rendimiento y precio en tiempo real, sin compromisos fijos. ¿Cómo se denominan estos dos tipos de servicio del broker?

A) Primer contrato = Service Arbitrage; Segundo contrato = Service Aggregation  
B) Primer contrato = Service Aggregation; Segundo contrato = Service Arbitrage  
C) Ambos son Service Aggregation, solo difieren en el número de proveedores  
D) Primer contrato = Service Intermediation; Segundo contrato = Service Aggregation  

> **Respuesta correcta: B** — **Service Aggregation** (primer contrato): combina servicios **predefinidos y fijos** de varios proveedores. **Service Arbitrage** (segundo contrato): similar a Aggregation pero **sin servicios fijos**; el broker puede **elegir dinámicamente entre múltiples agencias** según criterios en tiempo real (rendimiento, precio). La flexibilidad dinámica es la diferencia clave.

---

### Pregunta 7 — Multi Cloud vs Poly Cloud
Una empresa usa AWS para almacenamiento S3, Azure para Machine Learning y GCP para BigQuery, aprovechando la característica más destacada de cada proveedor en un sistema integrado en una sola plataforma. Los equipos eligen el servicio específico de cada cloud para cada caso de uso. ¿Qué modelo cloud describe esto?

A) Multi Cloud; usa múltiples proveedores con un interfaz único  
B) Hybrid Cloud; combina cloud público y privado  
C) Poly Cloud; usa características específicas de distintos clouds en una sola plataforma  
D) Distributed Cloud; distribuye la infraestructura geográficamente  

> **Respuesta correcta: C** — **Poly Cloud**: toma **características específicas** de cada cloud en **una sola plataforma integrada**, eligiendo la mejor feature de cada proveedor para cada caso. Multi Cloud usa distintos proveedores para distintas **cargas** desde un interfaz único, pero no necesariamente explota las características específicas de cada uno. Poly Cloud tiene mayor complejidad y coste de I+D.

---

### Pregunta 8 — Compliance en modelos de despliegue
Un director de cumplimiento normativo pregunta qué modelo de cloud computing facilita más el cumplimiento de regulaciones como PCI DSS, HIPAA y Sarbanes-Oxley. ¿Cuál es la respuesta correcta?

A) Public Cloud, porque los proveedores ya tienen todas las certificaciones  
B) Hybrid Cloud, porque combina lo mejor de private y public  
C) Private Cloud, porque la organización mantiene control total sobre los datos y la infraestructura  
D) Community Cloud, porque los costes de compliance se comparten entre organizaciones  

> **Respuesta correcta: C** — El **Private Cloud** facilita más el cumplimiento de **PCI DSS, HIPAA y Sarbanes-Oxley** porque la organización tiene **control total** sobre los datos, infraestructura, auditorías y configuraciones de seguridad. El libro CEH especifica explícitamente estas tres regulaciones como más fáciles de cumplir en cloud privada.

