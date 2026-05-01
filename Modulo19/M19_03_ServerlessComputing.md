# M19_03 — Serverless Computing
**Módulo 19 / Subapartado 3 — Serverless Computing**

---

## 1. Conceptos y definiciones

### Qué es Serverless Computing

Serverless computing = **FaaS** = serverless architecture. Tecnología cloud para despliegue de aplicaciones empresariales basadas en contenedores y microservicios. El nombre es engañoso: los servidores existen pero **no están expuestos físicamente al desarrollador**.

Arquitectura: el código de la aplicación corre en infraestructura cloud gestionada por un proveedor tercero. El desarrollador solo sube el código; el proveedor gestiona todo lo demás.

**Responsabilidades del cloud service provider en serverless:**
- Provisioning y scaling
- Load balancing
- Seguridad de la infraestructura
- Patch management del SO y software subyacente

**Responsabilidades del desarrollador:**
- Desarrollar y subir el código únicamente

Modelo de cobro: **pay-per-use** — solo se cobra por los recursos consumidos durante la ejecución. Cuando la función termina, es destruida automáticamente por el entorno cloud.

---

### 🔴 Serverless vs. Containers — tabla comparativa

| Containers | Serverless Computing |
|-----------|---------------------|
| El desarrollador define archivos de configuración del contenedor + SO, librerías, almacenamiento y red | El desarrollador solo desarrolla y sube el código; el CSP gestiona todo el provisioning |
| Crea imagen → la sube a registry → ejecuta contenedor desde la imagen | Tras la ejecución, la función serverless es destruida automáticamente por el entorno cloud |
| El contenedor corre **continuamente** hasta que el desarrollador lo detiene/destruye | El despliegue serverless cobra **solo por los recursos consumidos** |
| Un contenedor **requiere soporte de servidor incluso cuando no ejecuta programas** | Las funciones serverless tienen **timeout** habilitado |
| **Sin restricción de tiempo** para el código dentro del contenedor | La infraestructura del host es **transparente** para el desarrollador |
| Soporta ejecución en cluster de nodos host | Las funciones serverless **no soportan almacenamiento temporal**; los datos se guardan en object storage |
| Almacena datos en almacenamiento temporal o volúmenes mapeados | Las funciones serverless son adecuadas **solo para microservicios** |
| Soporta aplicaciones complejas **y** microservicios ligeros | Adecuadas **solo para microservicios** |
| El desarrollador elige el lenguaje y runtime | La elección de lenguaje está **restringida por el CSP** |

---

### Frameworks serverless

| Framework | Proveedor |
|-----------|----------|
| **Microsoft Azure Functions** | Microsoft |
| **AWS Lambda** | Amazon |
| **Google Cloud Functions** | Google |
| **Serverless Framework** | serverless.com |
| **AWS Fargate** | Amazon |
| **Alibaba Cloud Function Compute** | Alibaba |

---

## 2. Exam Traps ⚠️

⚠️ **[Serverless — "sin servidores"]**
Serverless no significa ausencia de servidores. Los servidores existen pero **no están físicamente expuestos al desarrollador**. El examen puede presentar serverless como una arquitectura que elimina los servidores — incorrecto.

⚠️ **[Serverless — responsabilidad del patch management]**
En serverless, el **cloud service provider** es responsable del patch management del SO y software subyacente, no el desarrollador. El examen puede atribuir esta responsabilidad al desarrollador.

⚠️ **[Serverless — almacenamiento]**
Las funciones serverless **no soportan almacenamiento temporal**. Los datos se almacenan en **object storage**. Los contenedores sí soportan almacenamiento temporal y volúmenes mapeados. El examen puede confundir los modelos de almacenamiento.

⚠️ **[Serverless — restricción de tiempo]**
Las funciones serverless tienen **timeout habilitado**. Los contenedores **no tienen restricción de tiempo**. Un proceso de larga duración es adecuado para contenedor, no para serverless.

⚠️ **[Serverless — restricción de lenguaje]**
En serverless, la elección de lenguaje está **restringida por el CSP**. En contenedores, el desarrollador elige libremente el lenguaje y runtime. El examen puede presentar serverless como agnóstico de lenguaje.

⚠️ **[Serverless — aplicaciones soportadas]**
Serverless es adecuado **únicamente para microservicios**. Los contenedores soportan tanto aplicaciones complejas como microservicios. El examen puede presentar serverless como válido para cualquier tipo de aplicación.

⚠️ **[Serverless — cobro durante inactividad]**
Un contenedor consume recursos de servidor **aunque no esté ejecutando programas**. Serverless cobra **solo cuando la función se ejecuta**; no hay cobro en inactividad. Confundir el modelo de cobro es una trampa recurrente.

⚠️ **[FaaS = Serverless]**
FaaS, serverless computing y serverless architecture son **términos equivalentes** en el contexto CEH. El examen puede presentarlos como conceptos distintos.

---

## 3. Nemotécnicos

### Serverless: qué gestiona el CSP (4 responsabilidades)
**"Pro-Sca-Load-Patch"**
→ **Pro**visioning | **Sca**ling | **Load** balancing | **Patch** management (SO + software)

### Las 4 diferencias críticas Serverless vs. Container para el examen
**"TALL"** → **T**imeout | **A**lmacenamiento (object storage, no temporal) | **L**enguaje restringido | **L**arga duración (no soportada)

Contenedores: sin timeout, almacenamiento temporal/volúmenes, lenguaje libre, soportan procesos largos.
Serverless: timeout activado, object storage, lenguaje restringido por CSP, solo microservicios.

### Frameworks serverless — los 3 grandes clouds
**"AMA"** → **A**WS Lambda | **M**icrosoft Azure Functions | **A**lpha (Google Cloud Functions)
Más: AWS Fargate, Serverless Framework, Alibaba Cloud Function Compute.

---

## 4. Flashcards

**Q:** ¿Serverless computing elimina completamente los servidores?
**A:** No. Los servidores existen pero no están físicamente expuestos al desarrollador. El CSP los gestiona de forma transparente.

---

**Q:** ¿Cuál es la responsabilidad del desarrollador en serverless computing?
**A:** Únicamente desarrollar y subir el código. El CSP gestiona provisioning, scaling, load balancing, seguridad e infraestructura.

---

**Q:** ¿Quién es responsable del patch management del SO en serverless computing?
**A:** El cloud service provider (CSP), no el desarrollador.

---

**Q:** ¿Cómo se cobra el uso en serverless computing?
**A:** Pay-per-use: solo se cobra por los recursos consumidos durante la ejecución. No hay cobro cuando la función no está en ejecución.

---

**Q:** ¿Qué ocurre con una función serverless tras completar su ejecución?
**A:** Es destruida automáticamente por el entorno cloud.

---

**Q:** ¿Tienen timeout las funciones serverless? ¿Y los contenedores?
**A:** Sí, las funciones serverless tienen timeout habilitado. Los contenedores no tienen restricción de tiempo.

---

**Q:** ¿Qué tipo de almacenamiento soportan las funciones serverless?
**A:** No soportan almacenamiento temporal. Los datos se almacenan en object storage.

---

**Q:** ¿Puede el desarrollador elegir libremente el lenguaje en serverless?
**A:** No. La elección de lenguaje está restringida por el cloud service provider.

---

**Q:** ¿Para qué tipo de aplicaciones es adecuado serverless computing?
**A:** Solo para aplicaciones de microservicios. No es adecuado para aplicaciones complejas ni procesos de larga duración.

---

**Q:** ¿Consume recursos un contenedor cuando no está ejecutando programas?
**A:** Sí, el contenedor requiere soporte de servidor incluso cuando no ejecuta programas. En serverless, no hay consumo cuando la función está inactiva.

---

**Q:** Un escenario describe una aplicación con procesos de larga duración que no puede interrumpirse. ¿Serverless o contenedor?
**A:** Contenedor. Serverless tiene timeout habilitado y no soporta procesos de larga duración.

---

**Q:** Nombra 6 frameworks/proveedores de serverless computing.
**A:** AWS Lambda, Microsoft Azure Functions, Google Cloud Functions, Serverless Framework, AWS Fargate, Alibaba Cloud Function Compute.

---

**Q:** ¿Cuáles son las cuatro responsabilidades del CSP en serverless que van más allá del provisioning?
**A:** Scaling, load balancing, seguridad de la infraestructura, y patch management del SO y software subyacente.

---

**Q:** ¿Cuál es la principal desventaja de seguridad de serverless computing?
**A:** Mayor vulnerabilidad de seguridad (increased security vulnerability) y vendor lock-in como restricciones estructurales del modelo.

---

**Q:** ¿Qué diferencia hay entre cómo se crea y gestiona el ciclo de vida en contenedores vs. serverless?
**A:** Contenedor: desarrollador crea imagen → sube a registry → ejecuta; corre continuamente hasta que lo detiene. Serverless: sube código → CSP lo ejecuta bajo demanda → destruido automáticamente al terminar.

---

## 5. Confusión frecuente

**Serverless vs. Contenedores — modelo de ejecución**
- Contenedor: corre **continuamente** una vez iniciado; el desarrollador lo detiene explícitamente.
- Serverless: se ejecuta **bajo demanda**, se destruye al terminar. Sin ejecución = sin coste = sin servidor activo.
- Criterio: si el escenario implica un proceso en ejecución permanente → contenedor. Si implica funciones disparadas por eventos → serverless.

---

**Serverless vs. Contenedores — alcance de aplicaciones**
- Contenedores: soportan aplicaciones complejas **y** microservicios.
- Serverless: adecuado **solo** para microservicios.
- Criterio: aplicación monolítica o compleja → contenedor. Microservicio o función discreta → serverless viable.

---

**Serverless vs. Contenedores — almacenamiento**
- Contenedores: almacenamiento temporal interno o volúmenes mapeados persistentes.
- Serverless: sin almacenamiento temporal; obligatoriamente **object storage** para persistencia.
- Criterio: si la pregunta menciona almacenamiento temporal o volúmenes → contenedor. Si menciona object storage como único medio → serverless.

---

**FaaS vs. SaaS — modelo de cobro**
- FaaS/Serverless: paga **solo cuando la función se ejecuta**; se apaga entre ejecuciones.
- SaaS: modelo de suscripción mensual, por uso, por publicidad o compartido entre usuarios.
- Criterio: ejecución puntual bajo demanda con cobro granular → FaaS. Acceso continuo a una aplicación → SaaS.

---

**Serverless — ventajas vs. desventajas para el examen**
- Ventajas clave: no hay gestión de servidores, pay-per-use, alta escalabilidad, despliegue rápido, latencia reducida.
- Desventajas clave: vendor lock-in, mayor vulnerabilidad de seguridad, no apto para procesos largos, dificultad en testing end-to-end, dificultad con statelessness.
- Criterio: si la pregunta describe el riesgo de dependencia de proveedor en serverless → vendor lock-in. Si describe procesos que no pueden interrumpirse → limitación estructural de serverless.

---

## 6. Preguntas de Práctica — Formato CEH

**P1.** Un desarrollador pregunta a su equipo de seguridad quién es responsable del patch management del sistema operativo en su nueva aplicación serverless en AWS Lambda. ¿Cuál es la respuesta correcta?

A) El desarrollador, ya que sube el código y controla el entorno de ejecución  
B) El cloud service provider (AWS), ya que gestiona toda la infraestructura  
C) El equipo de DevOps, ya que administra el pipeline de CI/CD  
D) Responsabilidad compartida entre el desarrollador y AWS  

**Respuesta correcta: B**
En **serverless computing**, el **cloud service provider** (CSP) es responsable de toda la infraestructura, incluyendo el **patch management del SO y software subyacente**. El desarrollador solo desarrolla y sube el código. Esta es una de las ventajas clave del modelo serverless.

---

**P2.** Una organización tiene una aplicación que procesa transacciones financieras de larga duración (hasta 2 horas por transacción). El CTO propone migrar a serverless. ¿Es esta una arquitectura adecuada?

A) Sí, serverless escala automáticamente para procesos largos  
B) No, las funciones serverless tienen timeout habilitado y no soportan procesos de larga duración  
C) Sí, si se usa AWS Fargate que no tiene restricción de tiempo  
D) Solo si el CSP provee opciones de extended timeout  

**Respuesta correcta: B**
Las **funciones serverless tienen timeout habilitado** y no son adecuadas para procesos de larga duración. Están diseñadas **únicamente para microservicios** con ejecuciones cortas. Para procesos de 2 horas, la arquitectura correcta es usar **contenedores** que no tienen restricción de tiempo.

---

**P3.** Un arquitecto compara el modelo de cobro entre contenedores y serverless. Un contenedor está desplegado pero no recibe tráfico durante 8 horas. ¿Qué costo genera?

A) Cero, porque no procesa peticiones  
B) El costo del tiempo de servidor aunque no ejecute programas  
C) Solo el costo de almacenamiento de la imagen  
D) El mismo que en serverless: pay-per-use  

**Respuesta correcta: B**
Un **contenedor requiere soporte de servidor incluso cuando no ejecuta programas**, por lo que genera costo durante las 8 horas de inactividad. En cambio, **serverless solo cobra cuando la función se ejecuta** — no hay costo en inactividad. Esta es la diferencia fundamental del modelo de facturación.

---

**P4.** Un analista de seguridad evalúa los riesgos de una migración a serverless. Identifica que el equipo de desarrollo usa Python y Node.js, pero el CSP solo soporta Go y Java. ¿Qué limitación describe esto?

A) Vendor lock-in — la organización no puede migrar a otro CSP  
B) Restricción de lenguaje — el CSP limita los lenguajes disponibles  
C) Timeout — ciertos lenguajes no pueden ejecutarse en el tiempo límite  
D) Almacenamiento — Python y Node.js requieren almacenamiento temporal no soportado  

**Respuesta correcta: B**
En serverless, la **elección de lenguaje está restringida por el CSP**. El desarrollador no puede usar libremente cualquier lenguaje o runtime, a diferencia de los contenedores donde tiene control total sobre el entorno de ejecución. Esta es una limitación estructural del modelo serverless.

---

**P5.** ¿Cuál de las siguientes afirmaciones sobre el almacenamiento en serverless computing es correcta según el CEH?

A) Las funciones serverless soportan almacenamiento temporal y volúmenes mapeados  
B) Los datos se almacenan automáticamente en la BBDD del CSP  
C) Las funciones serverless no soportan almacenamiento temporal; los datos se guardan en object storage  
D) El almacenamiento en serverless es equivalente al de los contenedores  

**Respuesta correcta: C**
Las **funciones serverless no soportan almacenamiento temporal**. Los datos se almacenan en **object storage** (ej. AWS S3, Azure Blob Storage). Los contenedores sí soportan almacenamiento temporal interno y volúmenes mapeados persistentes. Esta diferencia es importante para diseñar arquitecturas stateful.

---

**P6.** Un CISO evalúa si serverless es adecuado para un nuevo sistema CRM de la empresa que incluye procesamiento de datos complejos, gestión de sesiones, y múltiples integraciones con sistemas legacy. ¿Qué recomienda el CEH?

A) Serverless es adecuado por su alta escalabilidad automática  
B) Serverless no es adecuado; está diseñado solo para microservicios, no para aplicaciones complejas  
C) Serverless es adecuado si el CRM usa REST APIs  
D) Serverless es adecuado combinado con Docker Swarm  

**Respuesta correcta: B**
Serverless es adecuado **únicamente para microservicios**. Una aplicación CRM compleja con gestión de sesiones, múltiples integraciones y lógica de negocio compleja no es candidata para serverless. Los **contenedores** soportan tanto aplicaciones complejas como microservicios.

---

**P7.** Un desarrollador describe su entorno de ejecución: "El código se ejecuta en infraestructura gestionada por el proveedor; cuando termina, la función se destruye automáticamente; solo pago cuando se ejecuta." ¿Cuál de los siguientes frameworks corresponde a este modelo?

A) Docker con Docker Swarm  
B) Kubernetes con pods  
C) AWS Lambda  
D) Microsoft Azure AKS  

**Respuesta correcta: C**
La descripción corresponde a **serverless computing/FaaS**. **AWS Lambda** es el framework serverless de Amazon. Docker con Swarm y Kubernetes con pods son plataformas de orquestación de contenedores. Azure AKS (Azure Kubernetes Service) es un servicio de orquestación Kubernetes.

---

**P8.** ¿Cuál es la diferencia fundamental en el significado de "serverless" respecto a los servidores físicos?

A) En serverless no existen servidores; todo corre en máquinas virtuales  
B) Serverless elimina todos los servidores; solo existen funciones en la nube  
C) Los servidores existen pero no están físicamente expuestos al desarrollador; los gestiona el CSP de forma transparente  
D) Serverless significa que los servidores son virtuales, no físicos  

**Respuesta correcta: C**
**Serverless no significa ausencia de servidores**. Los servidores existen pero **no están físicamente expuestos al desarrollador** — el CSP los gestiona de forma completamente transparente. El nombre "serverless" se refiere a que el desarrollador no necesita gestionar ni provisionar servidores.
