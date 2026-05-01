# M09_05_SocialEngineeringCountermeasures.md
## CEH v13 — Módulo 09 | Social Engineering Countermeasures

---

## 1. Conceptos y definiciones

### Marco general

Las contramedidas de ingeniería social operan en tres niveles: **políticas y procedimientos**, **formación y concienciación**, y **controles técnicos**. Ningún control técnico aislado es suficiente — la defensa siempre pasa por la combinación de los tres niveles, con la formación como elemento central.

---

### 🔴 Estructura de la defensa — Tres capas

| Capa | Elementos principales |
|---|---|
| **Políticas y procedimientos** | Password policy · Physical security policy · Access privilege policy · Change management · Hardware/software policy |
| **Formación y concienciación** | Security awareness training · Social engineering campaigns · Gap analysis · Remediation strategies |
| **Controles técnicos** | 2FA/MFA · Spam filters · Anti-phishing toolbars · Anti-virus · Encrypted communications · Background checks |

Para que las políticas sean efectivas, la organización debe:
1. Distribuir las políticas entre los empleados con formación adecuada.
2. Obtener **firma del empleado** reconociendo que comprende las políticas.
3. Definir las **consecuencias de las violaciones** de política.

---

### 🔴 Password Policy — Requisitos del CEH

- Cambiar contraseñas regularmente + política de expiración.
- Mínimo **6–8 caracteres**, combinando alfanuméricos y caracteres especiales.
- No usar respuestas de preguntas sociales como contraseña (nombre de mascota, ciudad de nacimiento, película favorita).
- Bloquear cuenta tras un número determinado de intentos fallidos.
- No compartir contraseñas ni comunicarlas por teléfono, email o SMS.
- No reutilizar la misma contraseña en cuentas distintas.
- No almacenar contraseñas en medios físicos (notas adhesivas, libretas).
- No compartir cuentas de ordenador.
- Bloquear o apagar el equipo antes de abandonar el puesto.

---

### Physical Security Policy

- Emitir tarjetas de identificación, uniformes y otros controles de acceso.
- Escoltar a los visitantes a salas designadas.
- Restringir el acceso a zonas sensibles.
- Destruir documentos con trituradora de papel o burn bins (previene dumpster diving).
- Desplegar personal de seguridad complementado con alarmas y cámaras de vigilancia.
- Borrar dispositivos al retirarlos: sobreescribir el contenido del disco con 0s, 1s y caracteres aleatorios.

---

### 🔴 Defense Strategy — Tres fases secuenciales

| Fase | Descripción |
|---|---|
| **Social Engineering Campaign** | Simulación de ataques reales con técnicas diversas sobre grupos variados de empleados para evaluar su reacción |
| **Gap Analysis** | Evaluación de la organización contra mejores prácticas del sector, amenazas emergentes y estrategias de mitigación, usando los resultados de la campaña |
| **Remediation Strategies** | Plan detallado de mitigación basado en las brechas identificadas; foco en educación por roles y mitigación de amenazas específicas |

---

### 🔴 Contramedidas adicionales — Lista consolidada

- Formación en políticas de seguridad (conceptos básicos de SE, políticas y métodos de concienciación).
- Implementar privilegios de acceso apropiados: cuentas de administrador, usuario y guest con niveles de autorización respectivos.
- Definir procedimientos de respuesta a incidentes de ingeniería social.
- Restringir recursos solo a usuarios autorizados.
- Categorizar la información: top secret · proprietary · internal use only · public use.
- Realizar verificación de antecedentes y proceso de baja controlado (los ex-empleados y empleados con antecedentes son objetivos fáciles).
- Capas múltiples de antivirus en endpoints y mail gateway.
- Implementar **2FA/MFA** para servicios de alto riesgo (VPN, modem pools).
- Adoptar gestión de cambios documentada (más seguro que procesos ad-hoc).
- Mantener sistemas y software actualizados y parcheados.
- Política de hardware: prohibir USBs no autorizados.
- Política de software: solo software legítimo; definir quién puede instalar.
- Verificar cabeceras de email y enlaces antes de acceder.
- Verificar la identidad de quien solicita información.
- Filtros de spam.
- Usar canales de comunicación seguros y cifrados para información sensible.

---

### Defensa contra Phishing — Checklist

- Campañas de phishing educativas.
- Filtros de spam que detecten fuentes sospechosas.
- No responder a emails que soliciten información sensible.
- Hover sobre enlaces para verificar la URL real antes de hacer clic.
- Nunca proporcionar credenciales por teléfono.
- Comprobar saludo genérico, errores ortográficos y gramaticales.
- Confirmar el remitente antes de proporcionar información.
- Usar solo sitios HTTPS.
- Implementar MFA para prevenir ataques de Whaling.
- Contactar a la organización solo a través de canales oficiales (web corporativa).
- Realizar búsqueda inversa de imágenes en perfiles sospechosos de redes sociales.
- Reportar inmediatamente cuentas falsas en redes sociales.
- Denunciar en la oficina de cibercrimen si una cuenta de redes sociales extorsiona dinero.
- Instalar extensiones de seguridad anti-phishing en el navegador.
- Realizar copias de seguridad periódicas para minimizar el impacto de ransomware post-phishing.

---

### 🔴 Detección de emails de phishing — 13 indicadores

1. Adjuntos inesperados de usuarios, clientes, proveedores o compañeros desconocidos.
2. Adjuntos con formatos inusuales o no reconocidos.
3. Diferencias entre el email ID del remitente y el nombre de display.
4. Emails de IDs con nombres de organización incompletos/incorrectos o con números en lugar de letras.
5. Saludo genérico: "Dear users", "Dear customers", "Hello".
6. Sensación de urgencia + solicitud de transferencia inmediata de fondos (supuestamente de conocidos).
7. Enlace que muestra una URL diferente al hacer hover, o con nombre/dominio incorrecto.
8. Ofertas demasiado atractivas (lotería, suscripción gratuita, vacaciones, oferta de trabajo).
9. Email supuestamente del banco/proveedor/organización solicitando credenciales o instalación de actualizaciones.
10. Solicitudes de donaciones benéficas que requieren verificación.
11. Errores ortográficos evidentes y uso extraño de puntuación.
12. Solicitudes de información personal.
13. Ausencia de firma completa y datos de contacto del remitente.

**Técnica de hover**: pasar el ratón sobre el nombre en el campo "From" para ver el dominio real vinculado al remitente. Si no coincide con el dominio legítimo → posible phishing. Mismo procedimiento para URLs en el cuerpo del email.

---

### Herramientas anti-phishing

| Herramienta | Función |
|---|---|
| **Netcraft** | Toolbar; bloquea sitios peligrosos; proporciona información de reputación de webs visitadas; sistema de comunidad de alerta |
| **PhishTank** | Clearinghouse colaborativo de datos sobre phishing; API abierta para integración en aplicaciones; permite verificar si una URL es phishing |
| **OhPhish** | Portal web de simulación de campañas de phishing sobre empleados; genera informes MIS y tendencias en tiempo real por usuario/departamento/cargo; soporta: Entice to Click · Credential Harvesting · Send Attachment · Training · Vishing · Smishing |
| **Scanurl / Isitphishing / Threatcop / e.Veritas / Virustotal** | Herramientas adicionales de detección de phishing |

---

### 🔴 Tabla de objetivos, técnicas y defensas por área

| Área objetivo | Técnicas de ataque | Estrategia de defensa |
|---|---|---|
| **Front office / Help desk** | Eavesdropping, shoulder surfing, impersonation, persuasion, intimidation | Formación: nunca revelar contraseñas por teléfono; aplicar políticas específicas |
| **Technical support / SysAdmins** | Impersonation, persuasion, intimidation, fake SMS, llamadas, emails | Formación: nunca revelar contraseñas por teléfono o email |
| **Perimeter security / Office** | Impersonation, reverse SE, piggybacking, tailgating | Autenticación estricta (badge/token/biometría), formación, guardias de seguridad |
| **Vendors** | Shoulder surfing, eavesdropping, ingratiation | Formación de vendedores en ingeniería social; acompañar siempre a visitantes |
| **Mail room** | Impersonation, persuasion, intimidation | Cerrar y monitorizar el mail room; formación de empleados |
| **Machine room / Phone closet** | Robo/daño/falsificación de correo | Mantener cerradas las salas de servidores y armarios telefónicos; inventario actualizado de equipos |
| **Executives** | Fake SMS, llamadas, emails para obtener datos confidenciales | Formación: nunca revelar identidad, contraseñas o información confidencial por teléfono o email |
| **Dumpsters** | Dumpster diving | Basura en zonas seguras y monitorizadas; triturar documentos; borrar medios magnéticos |

---

### Contramedidas de Voice Cloning

- Escepticismo ante llamadas no solicitadas que pidan información sensible o acciones inusuales.
- Verificar la identidad del llamante con métodos adicionales (contraseñas, preguntas de seguridad).
- Formación sobre riesgos de voice cloning.
- Implementar **voice biometrics** u otras tecnologías avanzadas de autenticación.
- Usar tecnologías **anti-spoofing** para detectar voces sintéticas.
- Usar canales de comunicación de voz cifrados.
- Establecer procedimientos de verificación de identidad para interacciones por voz.

---

### Contramedidas de Deepfake

- **Digital watermarking**: incrustar códigos invisibles en vídeos auténticos en el momento de su creación.
- **Blockchain**: registros inmutables de contenido digital original para verificar autenticidad y rastrear origen.
- Mejorar tecnologías de reconocimiento facial para distinguir caras reales de generadas.
- Medidas de privacidad para proteger datos biométricos de uso no consentido.
- Mecanismos de reporte en redes sociales para marcar posibles deepfakes.
- Formación al público y profesionales de medios para evaluar críticamente el contenido digital.
- Herramientas de **IA/ML para detectar inconsistencias**: movimientos oculares antinaturales, expresiones faciales inconsistentes, iluminación irregular, lip-syncing defectuoso.
- Directrices éticas para desarrolladores de IA.
- **Técnicas forenses avanzadas**: análisis de artefactos de compresión, detalles a nivel de píxel, inconsistencias en streams de audio y vídeo.

---

### Contramedidas de Identity Theft — Consolidado

- Triturar o proteger todos los documentos con información privada.
- No aparecer en listas de marketing.
- Revisar extractos de tarjeta de crédito regularmente.
- No dar información personal por teléfono.
- Vaciar el buzón postal rápidamente; usar buzón con cerradura.
- Verificar todas las solicitudes de datos personales.
- No publicar identificadores personales en redes sociales (nombre del padre, mascota, dirección, ciudad de nacimiento).
- Habilitar 2FA en todas las cuentas online.
- No usar Wi-Fi público para acceder o compartir información sensible.
- Instalar firewall y antivirus.
- No almacenar información financiera en el sistema; usar contraseñas fuertes para cuentas financieras.
- Revisar facturas de teléfono en busca de llamadas no realizadas.
- Guardar en lugar seguro: tarjeta de la Seguridad Social, pasaporte, carné de conducir.
- Leer políticas de privacidad de webs.
- Entrar información personal solo en páginas HTTPS.
- Activar alertas de fraude.
- No permitir que terceros abran cuentas personales.
- Usar **credit freeze** con las principales agencias de crédito (Equifax, Experian, TransUnion).
- Optar por facturación y extractos electrónicos para reducir riesgo de robo de correo postal.

---

### OhPhish — Herramienta de auditoría de phishing

- Portal web para simular campañas de phishing sobre empleados.
- Captura respuestas y genera informes MIS y tendencias en tiempo real.
- Seguimiento por usuario, departamento o cargo.
- Métodos soportados: **Entice to Click · Credential Harvesting · Send Attachment · Training · Vishing · Smishing**.

---

## 2. Exam Traps ⚠️

⚠️ **[Longitud mínima de contraseña]** El CEH especifica **6–8 caracteres** como mínimo. El examen puede ofrecer valores distintos; la respuesta correcta según el libro es 6–8.

⚠️ **[Firma del empleado — requisito explícito]** No basta con distribuir las políticas. El examen puede preguntar qué paso adicional es necesario: obtener la **firma del empleado** reconociendo que ha entendido las políticas.

⚠️ **[Defense Strategy — orden de las tres fases]** El orden es estricto: Campaign → Gap Analysis → Remediation. El examen puede desordénarlas.

⚠️ **[OhPhish — qué hace y qué métodos soporta]** OhPhish es una herramienta de **simulación** (auditoría), no de ataque real. Sus 6 métodos (Entice to Click, Credential Harvesting, Send Attachment, Training, Vishing, Smishing) pueden preguntarse directamente.

⚠️ **[Netcraft vs. PhishTank]** Netcraft es una **toolbar** que bloquea sitios y da reputación de webs. PhishTank es un **clearinghouse colaborativo** con API abierta para verificar URLs de phishing. No son intercambiables.

⚠️ **[Deepfake — digital watermarking como contramedida]** El examen puede preguntar cuál es la contramedida preventiva más efectiva en el momento de crear el vídeo → digital watermarking (incrustado en el momento de creación).

⚠️ **[Deepfake — blockchain]** El uso de blockchain para deepfakes no es autenticación de usuarios: es para crear **registros inmutables de contenido original** que permitan verificar la autenticidad del vídeo.

⚠️ **[Credit freeze — agencias de referencia]** El examen puede preguntar las tres agencias de crédito: **Equifax, Experian y TransUnion**.

⚠️ **[Contramedidas de Voice Cloning — voice biometrics]** La contramedida técnica específica para voice cloning es implementar **voice biometrics** (no MFA genérico).

⚠️ **[2FA en servicios de alto riesgo]** El libro especifica VPNs y modem pools como ejemplos de servicios de alto riesgo que requieren 2FA. El examen puede preguntar para qué tipo de servicios se recomienda 2FA en el contexto de ingeniería social.

⚠️ **[Dumpster diving — contramedida de dispositivos]** Para deshacerse de dispositivos hardware, la contramedida es sobrescribir el disco con **0s, 1s y caracteres aleatorios** (no simplemente borrar los ficheros).

---

## 3. Nemotécnicos

### Las 3 fases de Defense Strategy — "C-G-R"
**"Campaña Genera Remedio"**
- **C**ampaign → **G**ap Analysis → **R**emediation

### Requisitos de Password Policy — "C-A-B-L-N-E-S"
**"Cada Año Bloquean Las Nuevas Entradas Sospechosas"**
- **C**ambiar regularmente
- **A**lfa-numérico + caracteres especiales
- **B**loquear tras intentos fallidos
- **L**ongitud mínima 6–8 caracteres
- **N**o comunicar por teléfono/email/SMS
- **E**xpiración configurada
- **S**in reutilización entre cuentas

### 13 indicadores de phishing — Agrupados en 4 bloques
- **Remitente**: diferencia ID/display · dominio incorrecto · ausencia de firma
- **Contenido**: saludo genérico · errores ortográficos · urgencia · solicitud de datos/credenciales
- **Adjuntos**: formatos inusuales · adjuntos inesperados
- **Enlaces**: hover muestra URL diferente · solo HTTPS válidos

### Contramedidas deepfake — "W-B-IA-F"
- **W**atermarking digital
- **B**lockchain para registros de contenido
- **IA/ML** para detectar inconsistencias
- **F**orensics avanzado (compresión, píxeles, audio/vídeo)

### Agencias de credit freeze — "E-E-T"
**Equifax · Experian · TransUnion**

---

## 4. Flashcards

**Q:** ¿Qué tres pasos debe seguir una organización para que sus políticas de seguridad sean efectivas contra la ingeniería social?
**A:** 1) Distribuir las políticas con formación adecuada. 2) Obtener la firma del empleado reconociendo que las ha entendido. 3) Definir las consecuencias de su incumplimiento.

---

**Q:** ¿Cuál es la longitud mínima de contraseña que especifica el CEH?
**A:** Mínimo 6–8 caracteres, combinando alfanuméricos y caracteres especiales.

---

**Q:** ¿Cuáles son las tres fases de la Defense Strategy contra ingeniería social en orden?
**A:** Social Engineering Campaign → Gap Analysis → Remediation Strategies.

---

**Q:** ¿Para qué tipos de servicios recomienda el CEH implementar 2FA en el contexto de ingeniería social?
**A:** Servicios de alto riesgo como VPNs y modem pools.

---

**Q:** ¿Cómo se debe destruir la información de un dispositivo hardware antes de retirarlo, según el CEH?
**A:** Sobrescribiendo el contenido del disco con 0s, 1s y caracteres aleatorios.

---

**Q:** ¿Qué es OhPhish y qué métodos de simulación soporta?
**A:** Portal web de simulación de campañas de phishing sobre empleados. Métodos: Entice to Click · Credential Harvesting · Send Attachment · Training · Vishing · Smishing.

---

**Q:** ¿Cuál es la diferencia funcional entre Netcraft y PhishTank?
**A:** Netcraft es una toolbar que bloquea sitios peligrosos y proporciona información de reputación de webs. PhishTank es un clearinghouse colaborativo con API abierta para verificar si una URL es un sitio de phishing.

---

**Q:** ¿Cuál es la contramedida técnica específica contra Voice Cloning mencionada en el CEH?
**A:** Implementar voice biometrics u otras tecnologías avanzadas de autenticación para verificar la autenticidad de comunicaciones por voz.

---

**Q:** ¿Cuál es la contramedida preventiva para deepfakes que se aplica en el momento de creación del vídeo?
**A:** Digital watermarking — incrustar códigos invisibles en vídeos auténticos en el momento de su creación.

---

**Q:** ¿Para qué se usa blockchain como contramedida contra deepfakes?
**A:** Para crear registros inmutables de contenido digital original que permitan verificar la autenticidad del vídeo y rastrear su origen.

---

**Q:** ¿Qué indicador de phishing se detecta haciendo hover con el ratón sobre el nombre del remitente?
**A:** Si el dominio real vinculado al remitente no coincide con el dominio legítimo esperado (p. ej., el campo "From" muestra "Gmail" pero el dominio real no es gmail.com).

---

**Q:** ¿Cuáles son las tres agencias de crédito con las que se recomienda hacer credit freeze para prevenir el robo de identidad?
**A:** Equifax, Experian y TransUnion.

---

**Q:** ¿Cómo se categoriza la información según las políticas de seguridad contra ingeniería social?
**A:** Top secret · Proprietary · For internal use only · For public use.

---

**Q:** ¿Qué técnica de ataque corresponde a la zona "Machine room / Phone closet" y cuál es su contramedida?
**A:** Ataque: intentar acceder, retirar equipos o conectar un analizador de protocolos. Contramedida: mantener salas de servidores y armarios telefónicos cerrados con llave en todo momento y llevar un inventario actualizado de equipos.

---

**Q:** ¿Por qué los ex-empleados y empleados con antecedentes son considerados objetivos fáciles en ingeniería social?
**A:** Porque poseen conocimiento interno de la organización y pueden tener motivaciones para colaborar con atacantes o ser más susceptibles a manipulación.

---

## 5. Confusión frecuente

### Netcraft vs. PhishTank
- **Netcraft**: herramienta del usuario final (toolbar de navegador); bloquea sitios activamente; da información de reputación; funciona como red de alerta comunitaria.
- **PhishTank**: base de datos colaborativa; permite verificar URLs; proporciona API para desarrolladores e investigadores.
- **Criterio**: ¿el uso es navegación protegida del usuario? → Netcraft. ¿Es verificación de URL o integración en aplicación? → PhishTank.

---

### 2FA vs. MFA
- **2FA (Two-Factor Authentication)**: exactamente dos factores de prueba de identidad (algo que tienes + algo que sabes).
- **MFA (Multi-Factor Authentication)**: dos o más factores. 2FA es un subconjunto de MFA.
- En el CEH, 2FA se menciona específicamente para VPN y modem pools. MFA se menciona para prevenir ataques de Whaling.
- **Criterio**: cuando el examen pregunta el mecanismo específico anti-Whaling → MFA. Para servicios de alto riesgo (VPN) → 2FA/MFA.

---

### Social Engineering Campaign vs. Phishing Campaign (OhPhish)
- **Social Engineering Campaign** (defense strategy): simulación interna para evaluar reacción de empleados ante técnicas diversas; parte de un ciclo Campaign → Gap → Remediation.
- **OhPhish Phishing Campaign**: herramienta específica de simulación de phishing; genera métricas por usuario/departamento; soporta múltiples métodos (Vishing, Smishing incluidos).
- **Criterio**: ¿el contexto es el marco estratégico de defensa? → Social Engineering Campaign. ¿Es una herramienta concreta de auditoría? → OhPhish.

---

### Digital Watermarking vs. Blockchain (contramedidas deepfake)
- **Digital Watermarking**: se aplica **en el momento de creación** del vídeo; incrusta un código invisible en el contenido auténtico.
- **Blockchain**: crea un **registro posterior** del contenido original para verificar autenticidad y trazabilidad.
- **Criterio**: ¿la contramedida actúa en el momento de grabar? → Watermarking. ¿Actúa creando un registro verificable? → Blockchain.
