# M13_05_PatchManagement.md
> Módulo 13 / Subapartado 5 — Patch Management

---

## 1. Conceptos y definiciones

### Patch vs. Hotfix — Distinción core

| Concepto | Definición | Características clave |
|---|---|---|
| **Patch** | Pequeña pieza de software diseñada para corregir problemas, vulnerabilidades de seguridad y bugs, o mejorar la usabilidad/rendimiento de un programa | Corrección de **múltiples** bugs/problemas conocidos; disponible públicamente para **todos los clientes**; actualización de carácter general |
| **Hotfix** | Paquete que aborda un defecto crítico en un entorno en producción (live environment) | Corrección de **un único** problema; actualiza una versión específica del producto; no siempre se distribuye fuera de la organización del cliente; puede entregarse como **combined hotfix** o **service pack** |

**Software vulnerability:** debilidad de un programa software que lo hace susceptible a ataques de malware. Los patches de los vendors reducen la probabilidad de que una amenaza explote una vulnerabilidad específica.

Regla: un sistema sin patches es mucho más vulnerable que un sistema parcheado regularmente. Si un atacante identifica la vulnerabilidad antes de que se corrija → el sistema es susceptible a ataques de malware.

---

### 🔴 Patch Management — Definición y tareas

Patch management: área de systems management que implica adquirir, probar e instalar múltiples patches en un sistema administrado. Es un método de defensa contra vulnerabilidades que causan debilidades de seguridad o corrupción de datos.

**Tareas del proceso:**
- Elegir, verificar, probar y aplicar patches
- Actualizar patches previamente aplicados con los actuales
- Listar patches aplicados al software actual
- Registrar repositorios/depósitos de patches para fácil selección
- Asignar y desplegar los patches aplicados

---

### 🔴 Proceso automatizado de Patch Management (6 pasos)

| Paso | Acción |
|---|---|
| **1. Detect** | Usar herramientas para detectar patches de seguridad ausentes |
| **2. Assess** | Evaluar el problema y su severidad; considerar los factores que influyen en la decisión |
| **3. Acquire** | Descargar el patch para pruebas |
| **4. Test** | Instalar el patch primero en una máquina de pruebas para verificar las consecuencias de la actualización |
| **5. Deploy** | Desplegar el patch en los equipos y verificar que las aplicaciones no se ven afectadas |
| **6. Maintain** | Suscribirse para recibir notificaciones sobre vulnerabilidades cuando se notifican |

---

### Instalación de un Patch

**Fuentes apropiadas para updates y patches:**
- Crear un plan de patch management adaptado al entorno operacional y objetivos de negocio.
- Encontrar updates y patches en los **home sites** de los vendors de aplicaciones u OS.
- Método recomendado para tracking proactivo: **registrarse en los home sites para recibir alertas**.

**Métodos de instalación:**
- **Manual:** el usuario descarga el patch del vendor e instala.
- **Automático:** las aplicaciones usan la función de auto-update para actualizarse.

**Verificación antes de desplegar:**
- Verificar la fuente del patch antes de instalarlo.
- Usar un programa de patch management para **validar versiones de ficheros y checksums** antes de desplegar.
- La herramienta de patch management debe poder monitorizar los sistemas parcheados.
- El equipo de patch management debe comprobar updates y patches regularmente.

---

### 🔴 Patch Management Best Practices

Agrupadas por categoría para el examen:

**Gobernanza y política:**
- Definir roles, responsabilidades y procedimientos, incluyendo plazos para aplicar patches y gestión de emergencias.
- Desarrollar una política completa de evaluación, testing, aprobación y despliegue.
- Incluir el patch management como parte de las operaciones IT estándar.
- Integrar aplicaciones de terceros en la estrategia de patch management.

**Inventario y alcance:**
- Definir qué sistemas, aplicaciones y dispositivos (servidores, workstations, equipos de red) requieren parcheado.
- Mantener un inventario actualizado de todos los activos hardware y software para no omitir ninguno.

**Priorización:**
- Evaluar y priorizar patches según la severidad de las vulnerabilidades que abordan.
- Agrupar activos por criticidad, tipo de aplicación u otros criterios relevantes.

**Testing y despliegue:**
- Usar herramientas para automatizar el descubrimiento de patches aplicables.
- Implementar un proceso de testing para verificar que los patches no causan problemas en sistemas existentes.
- Usar entorno de pruebas controlado **antes** de desplegar en producción.
- Asegurar compatibilidad con aplicaciones y configuraciones existentes.
- Hacer backup de los sistemas antes de aplicar patches.
- Implementar patching programado para minimizar interrupciones (alineado con maintenance windows).

**Respuesta a emergencias:**
- Crear procedimiento para despliegue fast-track de patches para vulnerabilidades críticas activamente explotadas.
- Tener un plan de **rollback rápido** si el patch causa problemas imprevistos.

**Monitorización y mantenimiento:**
- Suscribirse a security advisories y feeds para estar informado sobre nuevas vulnerabilidades.
- Verificar que los patches se han aplicado correctamente tras el despliegue.
- Usar herramientas para rastrear el estado de los patches en todos los activos.
- Realizar vulnerability scanning regularmente para identificar vulnerabilidades sin parchear y verificar la efectividad de los patches.
- Proteger el propio sistema de patch management contra ataques.
- Revisar y refinar periódicamente los procesos de patch management.
- Limitar el acceso a las herramientas de patch management al personal autorizado.

---

### 🔴 Herramientas de Patch Management

| Herramienta | URL | Característica destacada |
|---|---|---|
| **GFI LanGuard** | gfi.com | Escanea la red automáticamente; instala y gestiona patches de seguridad y no seguridad; soporta Windows, Mac OS X, Linux y aplicaciones de terceros; auto-download de patches ausentes; **patch rollback** |
| **SolarWinds Patch Manager** | solarwinds.com | Automatización del proceso de patch management |
| **Symantec Client Management Suite** | broadcom.com | Gestión de clientes y patches |
| **Kaseya Patch Management** | kaseya.com | Patch management automatizado |
| **Software Vulnerability Manager** | flexera.com | Gestión de vulnerabilidades de software |
| **Ivanti Patch for Endpoint Manager** | ivanti.com | Patch management para endpoints |

---

## 2. Exam Traps ⚠️

⚠️ **[Patch vs. Hotfix: número de problemas que corrige]**
Patch = múltiples bugs/problemas conocidos. Hotfix = **un único** problema crítico en entorno live. El examen puede presentar un hotfix como "paquete que corrige múltiples problemas" → incorrecto. O un patch como "fix para un único defecto crítico" → también incorrecto.

⚠️ **[Hotfix: distribución]**
Los hotfixes **no siempre se distribuyen fuera de la organización del cliente**. Son updates para un problema específico de un cliente. Pueden entregarse como combined hotfix o service pack. El examen puede afirmar que los hotfixes siempre son públicos → incorrecto.

⚠️ **[Patch: disponibilidad]**
Un patch es una actualización **pública disponible para todos los clientes**. A diferencia del hotfix, no es específico de un cliente.

⚠️ **[Proceso automatizado: orden exacto de los 6 pasos]**
Detect → Assess → Acquire → Test → Deploy → Maintain. El examen puede reordenar los pasos o intercambiar Acquire y Test. La lógica es: primero detectar, evaluar y descargar; luego probar; luego desplegar; por último mantener suscripciones.

⚠️ **[Test: dónde se instala el patch]**
El patch se instala primero en una **máquina de pruebas** (test machine), no directamente en producción. El examen puede omitir este paso o indicar que se instala directamente en producción en la fase de Acquire.

⚠️ **[Fuente de patches: método recomendado de tracking]**
El método recomendado es **registrarse en los home sites de los vendors para recibir alertas**. No es suficiente con visitar el sitio periódicamente; la suscripción activa es la práctica recomendada.

⚠️ **[Verificación antes del despliegue: checksums]**
Antes de desplegar, hay que validar **versiones de ficheros y checksums** usando el programa de patch management. El examen puede omitir los checksums y presentar solo la validación de versiones como suficiente.

⚠️ **[GFI LanGuard: sistemas operativos soportados]**
Soporta Windows, **Mac OS X** y **Linux**, además de muchas aplicaciones de terceros. El examen puede preguntar si soporta solo Windows → incorrecto.

⚠️ **[Patch rollback]**
GFI LanGuard incluye explícitamente la función de **patch rollback**. No todas las herramientas de patch management tienen esta función documentada en el libro. El examen puede preguntar qué herramienta permite revertir patches.

⚠️ **[Patch management como parte del SDLC]**
El chunk anterior ya mencionaba integrar patch management en el SDLC. Este chunk lo refuerza como best practice. El examen puede preguntar en qué etapa o proceso debe integrarse → SDLC y operaciones IT estándar.

---

## 3. Nemotécnicos

**Patch vs. Hotfix** → regla directa:
- **P**atch = **P**úblico + **P**luriproblemas (múltiples bugs)
- **H**otfix = **H**asta uno solo (un único defecto) + **H**ot (entorno live/producción)

**6 pasos del proceso automatizado** → **"D-A-A-T-D-M"**:
- **D**etect · **A**ssess · **A**cquire · **T**est · **D**eploy · **M**aintain
- Nemotécnico: "**D**os **A**viones **A**terrizan, **T**res **D**espegan, **M**añana"

**Best practices — 5 bloques** → **"G-I-P-T-M"**:
- **G**obernanza/política → **I**nventario → **P**riorización → **T**esting/despliegue → **M**onitorización

**Herramientas de patch management (6)** → **"GFI · Solar · Symantec · Kaseya · Flexera · Ivanti"**

**Instalación de patch: verificación** → "Fuente → Checksums → Monitor → Regular check"

---

## 4. Flashcards

**Q:** ¿Cuál es la diferencia principal entre un patch y un hotfix?
**A:** Patch: corrige múltiples bugs/problemas conocidos; disponible públicamente para todos los clientes. Hotfix: corrige un único defecto crítico en entorno live; actualiza una versión específica; no siempre se distribuye fuera de la organización del cliente.

---

**Q:** ¿Cuáles son los 6 pasos del proceso automatizado de patch management en orden?
**A:** Detect → Assess → Acquire → Test → Deploy → Maintain.

---

**Q:** ¿En qué consiste el paso "Test" del proceso de patch management?
**A:** Instalar el patch primero en una máquina de pruebas para verificar las consecuencias de la actualización antes de desplegarlo en producción.

---

**Q:** ¿Qué se debe validar antes de desplegar un security patch según las mejores prácticas?
**A:** Verificar la fuente del patch y validar versiones de ficheros y checksums usando el programa de patch management.

---

**Q:** ¿Cuál es el método recomendado por el CEH para el tracking proactivo de patches?
**A:** Registrarse en los home sites de los vendors de aplicaciones/OS para recibir alertas sobre nuevas vulnerabilidades y patches.

---

**Q:** ¿Qué es un hotfix y cómo puede distribuirse agrupado?
**A:** Paquete que aborda un único defecto crítico en entorno live. Puede entregarse agrupado como combined hotfix o service pack.

---

**Q:** ¿Qué herramienta de patch management permite auto-download de patches ausentes y patch rollback?
**A:** GFI LanGuard.

---

**Q:** ¿Qué sistemas operativos soporta GFI LanGuard?
**A:** Windows (Microsoft), Mac OS X y Linux, además de muchas aplicaciones de terceros.

---

**Q:** ¿Qué debe hacerse antes de aplicar patches para facilitar la recuperación ante problemas?
**A:** Hacer backup de los sistemas antes de aplicar patches.

---

**Q:** ¿Qué procedimiento especial debe existir para vulnerabilidades críticas activamente explotadas?
**A:** Un procedimiento de fast-track para despliegue urgente de patches, al margen del ciclo normal de patching.

---

**Q:** ¿Dónde deben encontrarse las fuentes apropiadas de updates y patches?
**A:** En los home sites de los vendors de las aplicaciones o del OS.

---

**Q:** ¿Cuál es la definición de patch management como proceso?
**A:** Área de systems management que implica adquirir, probar e instalar múltiples patches. Incluye: escanear vulnerabilidades de red, detectar patches y hotfixes ausentes y desplegar los patches relevantes en cuanto están disponibles.

---

**Q:** ¿Qué debe protegerse explícitamente como parte de las best practices de patch management?
**A:** El propio sistema de patch management contra ataques, para evitar su compromiso.

---

**Q:** ¿Qué dos métodos de instalación de un patch existen?
**A:** Manual (el usuario descarga e instala desde el vendor) y automático (la aplicación usa la función de auto-update).

---

**Q:** ¿Cuáles son las 6 herramientas de patch management que menciona el CEH?
**A:** GFI LanGuard, SolarWinds Patch Manager, Symantec Client Management Suite, Kaseya Patch Management, Software Vulnerability Manager (Flexera), Ivanti Patch for Endpoint Manager.

---

**Q:** ¿Cuándo se deben incluir las aplicaciones de terceros en la estrategia de patch management?
**A:** Siempre — las best practices indican explícitamente incluir las aplicaciones de terceros en la estrategia de patch management de la organización.

---

**Q:** ¿Qué es un service pack en el contexto de patch management?
**A:** Un conjunto de hotfixes agrupados que el vendor entrega como paquete combinado (combined hotfix).

---

**Q:** ¿Para qué sirve el vulnerability scanning en el contexto de patch management?
**A:** Para identificar vulnerabilidades sin parchear y verificar la efectividad de los patches ya aplicados.

---

**Q:** ¿Qué criterio se usa para priorizar el orden de parcheado entre activos?
**A:** Severidad de las vulnerabilidades que los patches abordan + agrupación de activos por criticidad, tipo de aplicación u otros criterios relevantes.

---

**Q:** ¿Qué debe verificarse después del despliegue de un patch?
**A:** Que el patch se ha aplicado correctamente y está funcionando según lo previsto. Se usan herramientas para rastrear el estado de los patches en todos los activos.

---

## 5. Confusión frecuente

**Patch vs. Hotfix**
- Patch: múltiples problemas, público para todos los clientes, actualización general.
- Hotfix: un único problema crítico, específico de un cliente o versión, entorno live, no siempre público.
- Criterio: "paquete público con múltiples correcciones" → patch. "Fix urgente para un único defecto crítico en producción" → hotfix.

---

**Combined Hotfix vs. Service Pack**
- Son lo mismo desde la perspectiva del libro: un vendor puede entregar un conjunto de hotfixes agrupados llamado combined hotfix o service pack.
- Criterio: si el examen pregunta cómo se pueden distribuir los hotfixes de forma agrupada → combined hotfix o service pack (ambas respuestas son equivalentes según el texto).

---

**Acquire vs. Test en el proceso de patch management**
- Acquire: descargar el patch para pruebas (solo descarga).
- Test: instalar el patch en una máquina de pruebas para verificar consecuencias.
- Criterio: "descarga del patch" → Acquire. "Verificación del impacto en entorno no productivo" → Test. Son pasos consecutivos; el examen puede fusionarlos o intercambiarlos.

---

**Patch rollback vs. Back-out plan**
- Patch rollback: función de una herramienta de patch management (ej. GFI LanGuard) que revierte un patch instalado.
- Back-out plan: plan documentado que define cómo revertir el sistema al estado previo ante una implementación fallida (concepto de gestión, no función de herramienta).
- Criterio: "función de herramienta que revierte un patch" → patch rollback. "Plan documentado de reversión" → back-out plan.

---

**Manual Installation vs. Automatic Installation**
- Manual: el usuario descarga el patch del vendor e instala activamente.
- Automatic: la aplicación usa su función de auto-update sin intervención del usuario.
- Criterio: "intervención activa del usuario para descargar e instalar" → manual. "La aplicación se actualiza sola" → automático. No son mejores prácticas opuestas; ambas son métodos válidos según el contexto operacional.
