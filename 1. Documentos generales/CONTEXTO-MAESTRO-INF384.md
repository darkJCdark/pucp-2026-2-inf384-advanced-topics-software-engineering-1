# INF384 · Contexto maestro para preparar la evaluación grupal

Documento de contexto para que Claude (o cualquier asistente) arranque con todo lo necesario sobre el curso **INF384 Temas Avanzados en Ingeniería de Software 1** (PUCP, 2026-2), a cargo del prof. José Felipe del Castillo Salazar. Estudiante: Jorge Chamorro (código 20206234).
Elaborado el 12/11/2026 a partir de los resúmenes de clase ya generados (`contexto-semana-1.md`, `contexto-semanas-2-3.md`, `contexto-semanas-4-5.md`, `S6`, `S7`, `S8`, `S10`, `S11`, `S12-contexto-clase.md`), `Lab4-caso-contexto.md`, `RESPUESTAS.md` del Lab 4, el sílabo y la memoria guardada del curso.

Convenciones: `[inferido]` = lectura mía, no dictada ni escrita en la fuente. `[no sé]` = dato que no tengo. "Diap. N" = lámina del profesor. Donde las fuentes se contradicen lo señalo en la sección 9.

## 1. Reglas de trabajo con Jorge (prioridad máxima)

1. Responder siempre en **español neutro**, tono profesional, directo y conciso. Sin saludos, cortesías, disculpas ni frases de relleno ("Claro", "Con gusto"). Empezar entregando la información.
2. **Código solo dentro de bloques de código Markdown.**
3. **Nunca inventar datos ni asumir hechos.** Si falta información, o si una premisa del usuario parece falsa, verificar con búsqueda web; si tras buscar no aparece, decirlo abiertamente.
4. **Hacer consultas** (preguntas de aclaración) cuando haga falta contexto o una decisión. Ser sincero y crítico: "no me des la razón siempre".
5. **Repositorios en solo lectura.** Los repositorios de laboratorio (`INF384-lab3-20206234`, `darkJCdark/INF384-LAB4-20206234`) no se tocan: prohibido hacer commits o aplicar cambios sin aprobación explícita de Jorge. Toda solución a aplicar debe indicar: **ruta y nombre del archivo**, **la línea inmediatamente anterior** y **la inmediatamente posterior** al cambio. Jorge aplica los cambios a mano (navegador) para que el historial no muestre trabajo de Claude.
6. **Nunca manejar credenciales de AWS ni secretos.**
7. **Uso de IA declarado**: el sílabo exige declarar y citar la IA generativa; puede pedirse el reporte de prompts. Sin declaración, falta a la ética académica; el plagio se castiga con nota 0. Los laboratorios admiten IA con declaración; **los exámenes no** (el profesor pidió no pegar los casos en una IA).
8. Los archivos de contexto de cada clase se guardan en `2. Semanas\S#\S#-contexto-clase.md` dentro de la carpeta del curso (`D:\pucp-2026-2-inf384-advanced-topics-software-engineering-1`); las demás salidas de Claude, en `Claude outputs\`.

## 2. El curso en una página

- **Competencias** (sílabo): C1 diseño en ingeniería (pipelines CI/CD con automatización, seguridad e IaC); C2 resolución de problemas (métricas, procesos, riesgos, mejora continua); C3 gestión de ingeniería (prácticas DevSecOps alineadas a estándares).
- **Resultados de aprendizaje**: **RA1** analizar procesos con métricas, VSM y diagnóstico de flujos; **RA2** diseñar e implementar pipelines CI/CD con automatización e IaC; **RA3** integrar seguridad (estático, dinámico, dependencias, IaC); **RA4** observabilidad, mejora continua y gestión de incidentes (resiliencia y confiabilidad).
- **Evaluación** (sílabo): `Nota final = (3·Pb + 3·Ex1 + 4·Ex2) / 10`. Pb = promedio de 5 prácticas tipo B (laboratorios), sin eliminables. Ex1 (parcial) pesa 3; Ex2 (final) pesa 4. `[no sé]` los pesos individuales de cada laboratorio dentro de Pb ni cuáles de los laboratorios cuentan como práctica; las notas ya conocidas: **Lab 2 = 19,75/20**. `[no sé]` notas de Lab 0, Lab 1, Lab 3, Lab 4 ni del Examen 1.
- **Bibliografía obligatoria**: *Accelerate* (Forsgren, Humble, Kim, 2018); *The DevOps Handbook* 2.ª ed. (Kim et al., 2021); **NIST SP 800-218 SSDF v1.1**. Complementaria: *Investments Unlimited*, DORA AI Capabilities 2025, OWASP DevSecOps Guideline. Lectura asignada para la próxima sesión: *The DevOps Handbook*, **parte IV** (segunda vía: retroalimentación y telemetría).
- **Capítulos y laboratorios** (sílabo; en la práctica los laboratorios no coinciden del todo con el sílabo): Cap. 1 ciclo de vida y VSM/DORA · Cap. 2 introducción DevSecOps (CALMS) · Cap. 3 integración continua (versionado, librerías, build, calidad, seguridad) · Cap. 4 despliegue continuo (artefactos, IaC, observabilidad, aprobaciones) · Cap. 5 automatización de calidad · Cap. 6 automatización de seguridad · Cap. 7 prácticas DevSecOps (postmortem, blameless, víctima/protagonista, shift left/right, self healing, **Lego Chocolate Game**, plantillas de pipelines) · Cap. 8 DevSecOps en las organizaciones · Cap. 9 IA en DevSecOps (Lab 7: IA para revisión de PR y para incidentes de seguridad).

### Calendario conocido (según láminas y clases)

| Fecha | Evento | Fuente |
|---|---|---|
| 26/9 | S6 despliegue continuo II | S6 |
| 2/10 | S7 calidad I; anuncio del parcial | S7 |
| 9/10 | S8 calidad II + simulacro del Examen 1 | S8 |
| 16/10 `[inferido]` | Sin clase registrada (carpeta S9 no existe); semana del Examen 1 | no confirmado |
| 23/10 | Sesión 9 del curso (carpeta S10): seguridad I | S10 |
| 29/10 | **Lab 4** (individual, 120 min) | Lab 4 |
| 30/10 | Sesión 10 (carpeta S11): seguridad II | S11 |
| 6/11 | Sesión 11 (carpeta S12): postmortem y protagonista | S12 |
| **Jueves 12/11, 16:00** | **Laboratorio 5** | láminas S11 y S12 |
| 26/11 | Lab 6 | láminas |
| Sin fecha | Sesión 12 (juego por equipos, **llegar a las 19:00**), sesión 13 (Cap. 8), sesión 14 (Cap. 9, IA), Examen 2 | láminas |

Nota de numeración: el nombre de la carpeta va un número por delante del número de sesión del curso desde la sesión 9 (carpeta S10 = sesión 9; S11 = sesión 10; S12 = sesión 11).

## 3. La evaluación grupal que viene

**Qué se sabe** (todo del profesor, sin la guía oficial a la vista):

- **Lab 5, jueves 12/11, 16:00.** La lámina lo titula "Pipeline con validación de seguridad de componentes y de código de infraestructura": punto de partida los dos pipelines del Lab 3; se agrega **composición (SCA) como compuerta** con umbral escrito, inventario (SBOM) publicado con el digest y excepción con cuatro datos; **análisis de la imagen antes del push**; **acciones y base fijadas por digest**; **política propia sobre el plan de Terraform** con `deny` y `warn`, y excepción pegada al recurso. La guía debía publicarse el 5/11 en el aula virtual `[no tengo la guía]`.
- **Formato según lo dicho en clase el 6/11** (distinto al de las láminas): "un mix entre una parte técnica y una parte parecida a la actividad de hoy"; **grupal, escrito, un entregable por grupo, 4 personas (máximo 5)**, en salas de Zoom donde los profesores pasan; **no se actúa, se llena el documento**; **no se desarrolla ni se construye un pipeline, no hay que usar Claude**; "más de prácticas"; ejemplo de pregunta: "encontramos vulnerabilidades en contenedores, ¿qué herramienta usarían?". Los grupos los arma el profesor.
- Lo que el profesor repitió sobre cómo se califican los casos: se evalúa **el razonamiento**, "el porqué de lo que se hace"; lo técnico es anecdótico; la justificación con datos (alcance, exposición, explotación, parche) pesa más que la etiqueta.
- Después: Lab 6 (26/11), Examen 2 (4 de peso), juego por equipos en la próxima clase.

**Qué no sé**: si "la siguiente evaluación grupal" es el Lab 5, el Lab 6 u otra; la rúbrica; el caso que se usará (probablemente el caso transversal del grupo, ver sección 8). **Verificar con la guía publicada.**

## 4. Mapa de contenidos por sesión

| # | Tema | Qué hay que saber |
|---|---|---|
| S1 | Ciclo de vida, VSM, DORA | Tensión Dev (velocidad) vs. Ops (estabilidad). VSM = foto puntual del flujo (entrevistas); DORA = medición continua automática. Cuatro métricas DORA. "El tiempo de espera es el problema." |
| S2 | Introducción a DevSecOps | CALMS: Culture, Automation, Lean, Measure(ment), Sharing. DevSecOps es capacidad organizacional, no herramienta. Modelos de equipo: silos, capacidad embebida, equipo de plataforma, SRE; antipatrón: DevOps como tercer silo. CI vs. CD (Delivery con aprobación manual / Deployment sin intervención). |
| S3 | CI I | GitFlow, GitHub Flow, Trunk-Based (+ feature flags); mapeo de escenarios a métricas DORA; SemVer; lock file; "construir una vez, desplegar muchas"; build ~3 min; SonarQube/Cloud y quality gate (cobertura 80 % en código nuevo). "El código que no escribimos también es nuestro." |
| S4 | CI II (seguridad) | Un control vive en CI si es **rápido, determinista, acotable al cambio y sin ambiente desplegado**. Secretos, SCA, SAST, IaC (Checkov). **Severidad ≠ prioridad** (alcanzabilidad, exposición, explotación, corrección). **"Detiene lo nuevo, inventaría lo heredado."** Excepciones con vencimiento. NIST SSDF. Tres Vías. "Que el pipeline salga verde no significa que todo esté bien." |
| S5 | CD I: artefactos e IaC | Artefacto = imagen/plantilla; tag vs. **digest/SHA**; Dockerfile (capas, multi-stage, `.dockerignore`, usuario no root, healthcheck); IaC declarativo; **Terraform state** (único, centralizado, con lock), **drift**, `init/fmt/validate/plan/apply`. |
| S6 | CD II | Estrategias: reemplazo total, progresivo (rolling), **blue-green**, **canary**, A/B. El **esquema de datos** es el límite de toda reversión → **expand/contract**. Rollback vs. roll-forward (se decide antes del incidente; la reversión se ensaya). Aprobaciones: dan trazabilidad, no detección; *Accelerate*: el comité externo no mejora estabilidad. Ambientes protegidos, credenciales federadas. Observabilidad (métricas/trazas/logs, correlación, versión desplegada). SLA/SLO/SLI, error budget. |
| S7 | Calidad I | Pirámide de pruebas y triángulo invertido; tiempo de retroalimentación como presupuesto; **FIRST**; dobles (stub, mock, fake, spy); cobertura mide ejecución, no verificación (mutación); integración acotada vs. amplia con dependencias efímeras; datos de prueba sin estado compartido. |
| S8 | Calidad II + simulacro Examen 1 | E2E (criterios, diseño confiable, Playwright); humo, verificación sintética; carga/estrés/pico/resistencia; orquestación (rápido primero, paralelo, `needs`, `matrix`, `if: always()`); política de compuertas (detiene/informa/revierte); caso Bodega Online (VSM, DORA, defectos de pipeline, expand/contract, seguridad B-2). |
| S10 | Seguridad I | SAST (fuente → propagación → saneamiento → sumidero), SCA/SBOM, imagen de contenedor (superficie, digest, tres momentos de revisión, firma), cadena de suministro, política de severidad y plazos, ejercicio detener/informar/excepción. |
| S11 | Seguridad II | Policy as code sobre el **plan** (Rego `deny/warn`, Checkov `checkov:skip`), drift, DAST (pasivo/activo, ambientes efímeros, vs. ethical hacking), **mapa de 10 controles**, flujo único de hallazgos, NIST SSDF (PO.3, PW.4, PW.6–PW.9, RV.1–RV.3). |
| S12 | Prácticas I | Westrum, postmortem sin culpa (8 partes), cuatro intervalos (detectar/responder/mitigar/restaurar), causas contribuyentes, acciones débiles/intermedias/fuertes, ritual, víctima/protagonista, afirmaciones vs. juicios, triángulos drama/empoderamiento, DISC. |

Detalle completo y citas por lámina: en los `S#-contexto-clase.md` y `contexto-semana*.md`.

## 5. Marcos y tablas de consulta rápida

### 5.1 VSM y DORA

- **VSM**: por actividad, PT (tiempo de proceso), LT (lead time), espera = LT − PT, %C&A (completo y exacto). **Eficiencia de flujo = ΣPT / ΣLT** (rango típico 1–15 %). Ejemplo de clase: 8 h sobre 9 d 13 h = 3,5 %; Bodega Online: 30/312 ≈ 9,6 %. Intervenir donde se junten **más espera y menor %C&A** (en Bodega, QA manual con 88 h de espera y 60 %).
- **Cuatro métricas DORA**: frecuencia de despliegue; **lead time para cambios** (del commit a producción); **tasa de fallas en cambios**; **tiempo de restauración** (*time to restore*). La lámina S12 lo mide **desde el inicio del impacto hasta el servicio recuperado** (en el ejemplo, 43 min = 29 detectar + 6 responder + 8 mitigar).
- **Clasificación usada en el Examen 1**: tasa de fallas Élite 0–5 %, Alto 5–15 %, Medio 15–45 %, Bajo > 45 % (material de consulta v2). Con los datos de Bodega: 6 despliegues/4 semanas = Medio; lead time mediana 11 días = Medio; fallas 1/6 ≈ 16,7 % = Medio; restauración 6 h = Alto. `[no tengo los umbrales completos de las otras tres métricas]`.
- VSM responde **por qué** es lento y dónde intervenir (foto puntual, con entrevistas); DORA responde **si mejora** (automático, continuo). El lead time DORA coincide con el LT del VSM desde desarrollo hasta despliegue.
- Mapeo escenario → estrategia de ramas → métrica foco (S3): startup de 5 devs → Trunk-Based → lead time; banco con ventanas fijas → GitFlow → tasa de fallas; una rama desplegada con despliegues espaciados → GitHub Flow → tasa de fallas.

### 5.2 Despliegue, reversión y datos

| Estrategia | Indisponibilidad | Versiones conviven | Costo | Reversión | Expone a un fallo |
|---|---|---|---|---|---|
| Reemplazo total | Sí | No | Bajo | Lenta | Todos |
| Progresivo (rolling) | No | Sí | Bajo | Media | Parte creciente |
| Blue-green | No | En la transición | Alto (doble infra) | Inmediata | Todos tras conmutar |
| Canary | No | Sí, prolongada | Medio | Inmediata | Porcentaje acotado (exige observabilidad y reversión automática) |

- Desplegar ≠ publicar; el **feature flag** los separa (apagar la bandera: segundos).
- **Expand/contract** (4 pasos): expandir sin tocar lo existente → escribir en ambas estructuras y migrar → mover la lectura (con flag) → eliminar lo antiguo en un despliegue propio. Nunca eliminar/renombrar una columna en el mismo despliegue; no desplegar el cambio el primer día de un evento de alto tráfico.
- **Rollback** (por defecto en un incidente; requiere artefacto previo y esquema compatible) vs. **roll-forward** (única opción si el esquema ya cambió). La decisión se escribe antes. Un mecanismo de reversión que nunca se ensayó es una suposición.
- Secuencia de verificación: desplegar → esperar estabilidad → humo → ventana de observación → confirmar o revertir. Reversión automática sin aprobación humana.
- Credenciales: federadas de corta duración > llaves estáticas; permisos acotados.
- **SLA** (promesa al cliente) más holgado que el **SLO** (objetivo interno, más agresivo) y el **SLI** (medición real). **Error budget** = margen aceptado; gastarlo desplegando; agotado → prioridad a estabilidad.

### 5.3 Pruebas y pipeline

- Pirámide: unitarias (ms, localizan la línea), integración (s, frontera/consulta/migración), E2E (min, recorrido completo). Se justifica cada nivel por los defectos que los inferiores no ven. Triángulo invertido = antipatrón.
- Dónde corre qué: antes de publicar (unitarias, integración acotada, contratos; decide si existe el artefacto) vs. contra el ambiente desplegado (humo, E2E críticas, carga); producción: humo, ventana de observación, verificación sintética.
- E2E solo para recorridos que generan ingreso, cruzan servicios o tuvieron incidentes. Carga: **p95**, nunca en cada commit, ambiente similar a producción, no desde una laptop. Cobertura ≠ verificación.
- Orquestación: lo rápido y lo que más falla primero; lo independiente en paralelo; E2E repartidas con `matrix`; `if: always()` para conservar la evidencia; `needs` solo con dependencia real de datos.
- **Política de compuertas**: falla unit/integración → detiene la publicación; cobertura del código nuevo bajo umbral → detiene; E2E crítica o p95 sobre umbral → detiene la promoción; prueba en cuarentena → informa (con responsable y vencimiento); humo en producción → revierte. Se escribe **antes de la primera falla**.
- Defectos clásicos de pipeline/Dockerfile (Examen 1): dependencias sin versión fija, `|| true`, `FROM x:latest`, `COPY . .`, secretos en `ENV`, sin `USER`, sin healthcheck, herramientas en la imagen final, reconstruir por ambiente (debe promoverse el mismo **digest**), tag ligado al número de ejecución en vez del SHA, `on: push` en todas las ramas, seguridad solo como reporte, `deploy` sin `needs` de las compuertas.

### 5.4 Seguridad (S4, S10, S11)

- **Dónde cae cada control**: CI (rápido, determinista, acotable, sin ambiente): secretos, SAST, SCA, cadena de suministro, imagen (antes del push), política sobre IaC y sobre el plan. Con ambiente desplegado: DAST (pasivo en cada despliegue en certificación; **activo** programado/nocturno, **nunca contra producción** sin acuerdo). Lo vivo: postura/drift programado (informa).
- **Mapa de 10 controles (S11, diap. 15)**: 8 detienen (secretos; SAST; SCA; cadena de suministro; imagen; política IaC; política sobre el plan; DAST pasivo) y 2 informan (DAST activo; postura sobre lo vivo). Cada fila: etapa, efecto, umbral, evidencia.
- **Priorización**: severidad es solo el punto de partida; se ajusta con ¿alcanzable?, ¿expuesto?, ¿se explota?, ¿hay parche? Ejemplo S10: medio + alcanzable + API pública + explotación activa → **detiene**; crítico + no alcanzable + job interno + sin explotación → se registra y planifica (pero con parche disponible y umbral "alto con parche", detiene).
- **Decisión sobre un hallazgo**: **detiene** / **informa** (con plazo) / **excepción** (identificador, razón, responsable, vencimiento; junto al recurso) / **aceptar el riesgo** (firma de quien responde; "carta de riesgo"). Una **credencial nunca se exceptúa: se rota**. Crítico/alto sin parche → excepción acordada con seguridad o mitigar y registrar; no puede pasar a producción sin esa firma.
- **Detiene lo nuevo, inventaría lo heredado** (línea base). Regla nueva entra en `warn` y sube a `deny` por acuerdo.
- **Plazos de la política típica citada por el profesor**: críticas y altas no pasan; medias hasta 15 días; bajas hasta 1 mes (ver contradicción en 9).
- **Policy as code**: se evalúa sobre el **plan** (después de `plan`, no antes de `init` ni después de `apply`); `deny` detiene, `warn` informa; familias de reglas: exposición, identidad, datos (`deny`) y trazabilidad (`warn → deny`). Drift: corregir por pipeline, nunca por consola; ruta escrita para el cambio de emergencia.
- **Imágenes**: base pequeña (completa 300–500 paquetes / slim 80–120 / distroless 10–20), fijada por **digest**, reconstruida de forma programada aunque el código no cambie; tres momentos de análisis (antes de publicar, en el registro, al desplegar); **firma** une los tres; SBOM guardado con el digest.
- **Cadena de suministro**: nombre parecido (typosquatting), confusión de dependencias, mantenedor comprometido, script de instalación, acción alterada → lock file con hash, registro privado, versiones fijadas, instalación sin scripts, **acciones por digest de commit**.
- **NIST SP 800-218 (SSDF)**, mapeo del curso: **PO.3** cadena de herramientas segura (acciones/base por digest); **PW.4** reutilizar software de forma segura (SCA, SBOM); **PW.6** proceso de compilación seguro (build en CI que solo produce imagen si pasan las compuertas, firma); **PW.7** revisar/analizar código (SAST, secretos, IaC); **PW.8** probar código ejecutable (DAST, pruebas del Cap. 5); **PW.9** valores seguros por defecto (política sobre el plan, imagen reducida); **RV.1·RV.2** identificar y remediar continuamente (registro, postura, flujo de hallazgos); **RV.3** causa raíz y mejora del proceso (postmortem). El Examen 1 pidió PW.4 (composición) y PW.7 (estático, secretos, IaC); RV.1 solo si el caso lo declara. El **digest/hash** es **integridad**, no control de vulnerabilidades.
- **Flujo de hallazgos**: formato común → deduplicar → triage (4 preguntas, lo hace el equipo que escribió el código) → decisión registrada → seguimiento; **3 métricas**: tiempo de corrección por severidad, excepciones vigentes, % de hallazgos con decisión. Un tablero aparte del área de seguridad "se convierte en una lista que nadie abre".
- **Responsabilidad organizacional**: seguridad define la política como código y revisa excepciones/alto riesgo, ya no valida cada salida; seguridad de la información (gobierno, riesgos) vs. seguridad informática (técnica, dentro de infraestructura y operaciones); el Product Owner ve el dashboard de su aplicación.

### 5.5 Prácticas: postmortem y cultura (S12)

- **Westrum**: patológica (poder: se castiga al mensajero), burocrática (reglas: se ignora), **generativa** (desempeño: se entrena y escucha; se investiga la falla). *Accelerate*: la cultura generativa predice mejor desempeño.
- **Preguntas que abren la investigación**: ¿qué hizo que el comando pareciera correcto?, ¿qué decía el procedimiento y cuándo se actualizó?, ¿qué información tenía/le faltaba a quien aprobó?, ¿qué señal existía y por qué no llegó a nadie? (las que cierran: ¿quién lo ejecutó?, ¿por qué no siguió el procedimiento?).
- **Postmortem (8 partes)**: resumen; impacto; línea de tiempo (solo **afirmaciones**); detección; causas contribuyentes (técnicas, de proceso, de detección, de organización; los cinco porqués siguen una sola cadena y se detienen en la primera respuesta cómoda); lo que funcionó; acciones (dueño, fecha, tipo); preguntas abiertas.
- **Cuatro intervalos**: detectar (inicio del impacto → detección), responder (detección → persona trabajando), mitigar (hasta que el usuario deja de sufrir), restaurar (inicio → servicio recuperado = métrica DORA).
- **Acciones**: débiles (capacitar, enviar correo, "tener más cuidado"), intermedias (checklist, revisión extra, documentar), **fuertes** (el sistema impide o detecta solo: compuerta, política, alerta con dueño). Una acción fuerte vive en el pipeline o la plataforma; es el control nuevo que sale del incidente.
- **Ritual**: criterio escrito (sev 1–2, consumo de error budget, casi incidente); borrador en **5 días hábiles**; reunión facilitada por alguien ajeno; publicación abierta (nombres como roles si se prefiere); seguimiento con **% de acciones cerradas a 30 días**. Primero se restaura el servicio; el postmortem va después.
- **Víctima / protagonista**: posiciones en una conversación, no identidades; el protagonista actúa en su dominio, hace pedidos/ofertas concretas con fecha, **pide ayuda y pone límites** (no es asumir toda la culpa ni ser Superman). **Afirmación** (verificable) vs. **juicio** (opinión; se fundamenta o se retira). Triángulo del drama (perseguidor, víctima, salvador) → de empoderamiento (challenger, creador, coach).
- **DISC** (D, I, S, C): tendencias observadas, no determinantes; "observamos conductas, sin etiquetar personas".

## 6. Cómo razonar las preguntas de caso `[síntesis mía a partir de lo que el profesor premia]`

1. **Identificar el dato del caso que decide** (umbral, severidad, etapa, métrica) y citarlo; el profesor valora el razonamiento por encima de la etiqueta.
2. **Ubicar el control**: ¿es rápido, determinista, acotable, sin ambiente? ¿corre antes de publicar o contra el ambiente? ¿detiene, informa o revierte?
3. **Hallazgo de seguridad**: aplicar las cuatro preguntas (alcanzable, expuesto, explotado, con parche) → decisión (detiene/informa/excepción/aceptar) → acción concreta → quién autoriza → dato que pesó.
4. **Métrica DORA**: relacionar el evento con la métrica (picos de tráfico → tasa de fallas y restauración); explicar si sube, baja o no cambia y por qué.
5. **Cambio de esquema o despliegue riesgoso**: expand/contract, feature flag, despliegue antes del evento, condición de reversión escrita de antemano.
6. **Incidente**: separar hecho / lectura / acción; línea de tiempo con afirmaciones; intervalos; causas en cuatro categorías; acciones fuertes con dueño y fecha; evitar juicios sin fundamento.
7. **Respuesta de equipo (formato escrito)**: asignar un responsable de redacción y uno de verificación; mezclar perfiles (ver DISC, S12); usar la tabla de decisión de la sección 5.4; escribir la razón junto a cada decisión.
8. **No pegar el caso a una IA** donde el profesor lo prohíba; en los laboratorios, declarar siempre la herramienta y cómo se usó.

## 7. Banco de repaso rápido (preguntas y respuesta corta)

1. ¿Cuándo un control pertenece a CI? → Rápido, determinista, acotable al cambio, sin ambiente desplegado (S4).
2. ¿Por qué la política de IaC corre después de `plan`? → Porque el plan trae los valores resueltos y las acciones; antes del `init` no hay definiciones y después del `apply` el recurso ya existe (S11).
3. Un crítico sin parche, ¿qué se hace? → No pasa; excepción acordada con seguridad / carta de riesgo del dueño de negocio, mitigar y registrar; si ya está en producción, el líder de la tribu decide si se baja (S10).
4. Un medio alcanzable, expuesto, con explotación activa y parche → Detiene (la lámina); aclaración: en clase el profesor se inclinó por informar (ver 9).
5. ¿Qué diferencia hay entre excepción e informar? → Excepción: se exceptúa del gate para que pase (con 4 datos); informar: el pipeline no se detiene pero el hallazgo se reporta y se sigue (S10).
6. ¿Qué mide DORA "time to restore" y cómo se descompone? → Inicio del impacto → servicio recuperado; detectar + responder + mitigar (S12).
7. ¿Qué acción de postmortem es "fuerte"? → La que el sistema ejecuta solo: compuerta, política, alerta con dueño (S12).
8. ¿Por qué el DAST activo no corre contra producción ni contra el QA compartido? → Crea/modifica datos y puede tumbar la aplicación; se usa un ambiente efímero o dedicado (S11).
9. ¿Qué es drift y cómo se evita? → Diferencia entre lo declarado y lo real; plan programado + postura continua; cambios solo por pipeline (S5, S11).
10. ¿Por qué un esquema de datos limita la reversión? → El código vuelve en minutos, la migración aplicada no; se usa expand/contract (S6).
11. Cobertura 100 % sin aserciones, ¿qué significa? → Se ejecutó el código, no se verificó nada (S7).
12. ¿Qué cambia entre una cultura patológica y una generativa ante una falla? → Se busca un culpable vs. se investiga (S12).
13. ¿Qué parte del NIST cubre el SBOM/composición? → PW.4 (S8, S10, S11).
14. ¿Por qué no basta el comité de aprobación externo? → Da trazabilidad, no detección; *Accelerate* no encuentra mejora en estabilidad y sí más espera (S6).

## 8. Caso transversal del grupo y trabajo de laboratorio

- **Caso 3 · Seguros Pacífico Sur**, arquetipo **"Retrabajo invisible"** (rendimiento acumulado de calidad ≈ 11 %, 76 de 94 historias devueltas). Es el caso de las sesiones 1 y 4 que se reutiliza en la actividad A2 y probablemente en evaluaciones de equipo. `[no tengo el PDF del caso a la vista en esta sesión; los datos son los de la memoria]`
- **Laboratorios hechos**: Lab 0 y 1 (pipeline base a bucket AWS), **Lab 2** (calidad + unit test a Lambda; **19,75/20**; único descuento: en 4.2 la versión declarada debió ser 1.3.0 con 1 `feat` y 2 `fix`), **Lab 3** "Las fijas" (imagen Docker + ECS, plan/apply de Terraform; dos pipelines `app.yml` e `infra.yml` con compuertas `calidad`, `secretos`, `dependencias`, `estatico`), **Lab 4** "Monedero Andino" (individual, 29/10).
- **Lab 4, resumen de la entrega** (Alumno 20206234; presupuesto 11 min; métrica DORA *time to restore*): cinco defectos corregidos con un commit cada uno (`load` 12 min en cada PR; `smoke` sin esperar a `deploy` y sin `EXPECTED_COMMIT`; falta de `concurrency`; pruebas de `integration` encadenadas; `e2e` casi sin navegador); defecto introducido en `src/wallet.js` (`<` → `<=`), detenido por `unit` en el PR #1; duración reducida de 15:03 a 9:00 (carga constante de 10 a 4 min); SLO p95 < 150 ms (tras medir 51,32 y 81,18 ms), SLA 99,5 % y p95 ≤ 500 ms; riesgo aceptado: quitar `LIMIT 50` de `leerMovimientos`; `sast` se queda. Pendientes: respuesta del docente sobre runs del 30/10 frente a la regla "run iniciado el 29/10", subir el archivo `INF384-LAB4-[código]` a Paideia con el link al fork, y completar la declaración de uso de IA con cualquier otra herramienta usada. `[no sé]` la calificación.
- Lecciones útiles para el siguiente laboratorio: medir con datos observados (la evidencia sin dato no puntúa), un defecto por commit, enlazar runs, justificar umbrales con números, pensar el SLO con holgura frente a la variación entre runs, declarar la IA.

## 9. Discrepancias e incertidumbres conocidas

- **Formato del Lab 5**: láminas (pipeline técnico con composición, imagen y política sobre el plan) vs. lo dicho el 6/11 (escrito, grupal, sin desarrollar). Confirmar con la guía.
- **Hallazgo 4 del ejercicio de S10** (medio, alcanzable, expuesto, explotación activa, con parche): la lámina 11 dice que **detiene**; en clase el profesor dijo "informar" con 15 o 30 días. Justificar con los datos.
- **Plazos de corrección**: 15 días para medias y 1 mes para bajas (política típica) frente a "30 días" dicho para un medio en el ejercicio. En una evaluación citar el plazo que dé el caso.
- **Fecha del Lab 4**: la hoja de ruta decía 29/10; los runs del 30/10 plantean la duda (pendiente de respuesta del docente).
- **Tabla DORA**: las cifras de tasa de fallas cambiaron entre versiones del material de consulta (v2: Élite 0–5 %, Alto 5–15 %, Medio 15–45 %, Bajo > 45 %).
- **Cifras del profesor al hablar** (por ejemplo "48 minutos" en el ejemplo del timeout) no siempre coinciden con las láminas; valen las láminas (43 min).
- Las anécdotas del profesor (Oracle, war rooms, borrado con Terraform, incidente de AWS, firewall abierto, compra de Checkov por Palo Alto) son relatos suyos, no verificados.
- Láminas de **pruebas de contrato** (S8) y láminas 4–6 de S12 no están en los PDF; lo que las menciona viene solo de títulos o puntos clave.
- Transcripciones con ruido: S11 sin texto entre ~48:02 y ~51:18 (demo de Checkov); S12 sin audio del video ni de la preparación de sketches.

## 10. Índice de archivos

| Ruta (carpeta del curso) | Contenido |
|---|---|
| `1. Documentos generales\INF384-2026-2-SILABO.PDF` | Sílabo, evaluación, bibliografía |
| `1. Documentos generales\devops Curso y Libros\` | *Accelerate*, *DevOps Handbook* (intro, partes 1–2), *Phoenix Project* (extracto), material de certificación |
| `2. Semanas\S1…S15\` | Diapositivas y transcripciones por semana (S13–S15 aún vacías) |
| `2. Semanas\S6, S7, S8, S10, S11, S12\S#-contexto-clase.md` | Resúmenes detallados de cada clase |
| `Claude outputs\contexto-semana-1.md`, `contexto-semanas-2-3.md`, `contexto-semanas-4-5.md`, `declaracion-uso-ia.md` | Resúmenes S1–S5 y plantilla de declaración de IA |
| `3. Laboratorios\L1…L4\` | Entregas y repositorios de laboratorios (L4 solo `.gitkeep`; el repo vive en GitHub) |
| `4. Examenes\1. Parcial`, `2. Final` | Vacías en este momento |
| GitHub `darkJCdark/INF384-LAB4-20206234` | Fork del Lab 4 (solo lectura para Claude) |

## 11. Qué necesito de Jorge para afinar la preparación

1. La **guía del Lab 5** (publicada el 5/11) o el enunciado de la evaluación grupal.
2. El **caso asignado al grupo** (PDF del Caso 3 si se va a usar) y la composición del equipo.
3. Qué laboratorios cuentan para Pb y la calificación del Examen 1, para priorizar el estudio.
4. Si la evaluación es el Lab 5 de hoy, cuántas horas hay y si se puede consultar material.
