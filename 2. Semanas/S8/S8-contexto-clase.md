# INF384 · Sesión 08 — Automatización de calidad II: pruebas end to end, pruebas sobre el ambiente desplegado, orquestación y simulacro del Examen 1

Curso: INF384 Temas Avanzados en Ingeniería de Software 1 (PUCP, 2026-2) · Prof. José Felipe del Castillo Salazar
Capítulo 5 (2 de 2) · 3 horas · RA2 · RA4.
Clase dictada el 9/10/2026 en la noche, hora de Lima (el archivo de grabación figura como 10/10/2026 por la zona horaria; duración 174 min, con un hueco de ~30 min mientras los grupos trabajaban sin grabación).
Fuentes: transcripción (`Temas S8.txt`, con ruido de transcripción automática), diapositivas (`INF384_S8_Automatizacion_calidad_II.pdf`), `INF384_Material_consulta_Examen1.pdf`, el caso modelo `INF384 · Caso modelo para el Examen 1 (Bodega Online).pdf` y capturas de pantalla de la guía de resolución que el profesor proyectó (carpeta S8).

Nota de fidelidad: la numeración de las diapositivas salta de la 3 a la 8; las láminas de **pruebas de contrato** que anuncian el título y el punto clave 1 no están en el PDF ni se dictaron en clase. Todo lo de contratos de este documento viene solo de esa mención en la diapositiva final.

## 0. Avisos de la clase

- Esta sesión **cierra la información que entra al parcial**. El parcial cubre hasta el capítulo anterior (Despliegue continuo) "con algunas cositas de calidad" (Capítulo 5).
- Estructura del examen que adelantó el profesor: **un caso obligatorio de unos 8 puntos** (como la Parte A del caso modelo) y **casos opcionales por 12 puntos en total** (se eligen tres de seis; el caso B-2 de muestra vale unos 4 puntos).
- El examen es **a mano y presencial**. El profesor entrega impreso el **material de consulta** (`INF384_Material_consulta_Examen1.pdf`, con teoría de todos los capítulos incluidos y el anexo del NIST SP 800-218). Objetivo de la sesión: que los alumnos se familiaricen con dónde está cada cosa en ese material.
- En el Dockerfile y el pipeline del examen **no se pide sintaxis**: se pide describir el problema, el impacto y cómo se corregiría. El profesor dijo que pondrá comentarios en el código como pistas.
- Dinámica de la clase: 5 grupos aleatorios, ~30 minutos para resolver el caso modelo (Parte A y B-2) y luego revisión conjunta del solucionario.
- Se pide al alumno **no copiar y pegar el caso a una IA**, porque en el examen no habrá esa opción.
- El profesor dijo que compartirá el repositorio de ejemplo `testing-example` (Playwright) y que puede compartir la grabación (con el hueco de ~30 min); le escribirá a Jorge para coordinar la mejor forma de hacerlo.

## 1. Dónde estamos en el pipeline

Commit → Build → Pruebas → Calidad → Seguridad → Publicar → Desplegar → Observar.

- Las pruebas de calidad se reparten en **tres momentos**: la fase de pruebas/certificación, justo después del despliegue (humo) y después en observabilidad (verificaciones sintéticas periódicas).
- Preguntas que la sesión responde: ¿cómo se comprueba que dos servicios siguen siendo compatibles si se despliegan por separado (contratos; solo en la diapositiva)? ¿Qué flujos justifican una prueba end to end? ¿Qué corre contra el ambiente desplegado? ¿Qué resultado detiene el despliegue a producción?
- Repaso oral: ¿qué se automatiza primero? **Las unitarias**: se miden en segundos, no necesitan la aplicación levantada, corren en la máquina o en el pipeline y son directas de ejecutar. La unitaria no valida que la base de datos funcione (para eso, integración) ni el flujo completo (para eso, end to end).

## 2. Pruebas end to end (E2E)

### Qué son y qué recorridos merecen una (diap. 9)

Prueban el recorrido de punta a punta (login, clic en menú, ingresar datos, etc.), invocando varios servicios en una sola prueba. Se pueden acotar (por ejemplo, solo login) y elegir qué conjunto correr cada vez.

Un recorrido entra a la suite si cumple alguno de estos criterios:

1. **Genera ingreso o cumple una obligación** (pagar, emitir comprobante, agendar, registrar despacho).
2. **Cruza varios servicios** (ninguna prueba inferior recorre el flujo completo).
3. **Tuvo incidentes previos en producción**. El profesor dio un ejemplo propio: un cliente con botón para llamar vía Twilio que fallaba ocasionalmente con el carrier; se monta como caso de prueba aislado. La bitácora de incidentes alimenta los casos E2E.
4. **Conjunto acotado y revisado periódicamente**: son los flujos que no se pueden caer. Cada uno cuesta minutos en cada ejecución y horas de mantenimiento al año, entra cuando su falla en producción costaría más que eso.

### Cuándo y dónde corren

- Cruzan componentes, y los pipelines son **por componente**; correrlas en cada commit de cada componente es ineficiente. Normalmente tienen **un pipeline aparte**, con reglas: programado (una vez al día o a la semana, para ver que no se haya roto nada), **a demanda** (el QA ejecuta el conjunto que corresponde, o `all` para regresión antes de pasar a producción).
- Corren contra un **ambiente real** (desarrollo, QA, staging, UAT) por URL; es raro usar un ambiente mock.
- Es preferible tenerlas **en un repo separado de pruebas**: su propio ritmo de actualización, permisos y desacople si la aplicación se distribuye en varios repos.
- **Beneficios de correrlas en pipeline frente a la máquina del QA** (el profesor estimó que solo ~40% automatiza y que 80–90% de eso corre en la máquina propia): la máquina no queda bloqueada, la configuración está estandarizada, el historial queda registrado (quién, cuándo, qué falló, hasta capturas de pantalla), y se puede paralelizar según la capacidad del runner.

### Diseño de una E2E confiable (diap. 10)

| Aspecto | Frágil | Confiable |
|---|---|---|
| Selección | Por posición o texto visible | Por atributos estables dedicados a la prueba (test id, rol) |
| Espera | Fija: `sleep` de 5 s | Por condición: elemento visible, respuesta 200 |
| Datos | Cargados a mano en el ambiente | Creados por la prueba vía API y eliminados al final |
| Independencia | Encadenadas (la 2 usa lo que creó la 1) | Cada prueba corre sola y en cualquier orden |
| Verificación | Pantalla completa, píxel a píxel | El resultado de negocio del recorrido |

Detalles que dijo el profesor: el reto principal es que el selector sea **único** (ejemplo: tres inputs llamados "nombre cliente"); los combos dependientes (país, provincia, distrito) necesitan espera por condición, "lo peor es hacer sleeps"; los datos vienen de un archivo o de APIs, y lo ideal es que la misma prueba **cree y borre su data** (la mayoría no la borra y el ambiente se llena de basura que entorpece otras pruebas); la validación puede ir más allá del 200, por ejemplo verificar que tras una venta el stock disminuyó.

Ejemplo serverless en la diapositiva 11: la URL llega del despliegue (`URL_API`), datos propios con referencia única (`e2e-${Date.now()}`), aserción del estado HTTP 201 y del cuerpo (tarifa 54), consulta del efecto persistido (estado `REGISTRADO`) y eliminación del pedido al final.

### Playwright

- **Qué es**: test runner que paraleliza, reintenta, reporta y guarda trazas; protocolo de comandos al navegador (Chromium, Firefox, WebKit, con o sin interfaz). Un **contexto aislado por prueba**, **espera automática** antes de cada acción, **localizadores** por rol/texto/test id (`getByRole`, `getByTestId`), **aserciones que reintentan** hasta timeout. El **trace viewer** guarda DOM, red, consola y captura de cada paso y en CI se sube como artefacto.
- **Chromium y no Chrome**: Chromium es la versión ligera que corre en contenedores (Chrome necesita librerías que el contenedor no tiene).
- **Demo en vivo**: proyecto de ejemplo (mini catálogo/compra). La prueba llena el formulario campo por campo y valida que la respuesta contenga el SKU/nombre del producto y que el listado tenga el último atributo esperado. Se ejecutó con GitHub Actions manualmente (`workflow_dispatch`, a demanda), con selección de rama y del feature (solo catálogo). El job instala dependencias, Chromium y levanta Postgres: **casi un minuto de preparación antes de correr**, mucho más que una unitaria. Al forzar un fallo, el reporte mostró el valor esperado vs. el obtenido, **captura de pantalla** antes del fallo y la traza (de 14 pruebas, 1 falló). El profesor no recomienda que corra automáticamente en cada cambio de archivo.
- **`codegen`**: comando de Playwright que abre el navegador con un inspector y **graba las acciones** del usuario para generar el código base (en la demo, en Python/pytest sobre Wikipedia); luego se puede copiar y pedir a una IA que lo refine. Es "un RPA muy ad hoc". Playwright sirve también para automatizar registro de información (anécdota del equipo DevOps que automatizó con Selenium la habilitación diaria de un permiso Just-In-Time Access de 4 horas).
- **Limitaciones**: pensado para aplicaciones web (HTML). Para apps móviles no está seguro de que sirva. Para cliente-servidor/escritorio legado se usan herramientas RPA (se mencionó UI Vision y Tosca, esta última cara). Selenium es la herramienta previa; Playwright "se la comió" por ser más rápida y fácil. En una aplicación React compleja con elementos que aparecen y desaparecen aparecen **falsos positivos** por condiciones de carrera.
- **E2E vs. unitaria vs. web scraping** (pregunta de un alumno): la unitaria prueba el componente en el código fuente, sin desplegar; usar un scraping/E2E para validar algo que podría validarse con unitaria es "matar una hormiga con una bazuca". Si el flujo interviene base de datos, APIs y despliegue, entonces es E2E.
- **Anexo RPA (contexto del profesor)**: el RPA nació para reemplazar tareas repetitivas manuales tercerizadas (cobro por volumen de transacciones), que simulan el flujo uno a uno incluso entre aplicaciones legadas de escritorio.

## 3. Pruebas sobre el ambiente desplegado

### El digest como candado de inmutabilidad (repaso S5/S6)

El artefacto (contenedor o librería) se identifica con un tag que normalmente es el **SHA del commit** (primeros 7 dígitos) y con el **digest**. Eso da **trazabilidad** (qué commit corresponde) e **inmutabilidad**: con un tag como `produccion` nada asegura qué commit contiene ni si fue retagueado sin pasar pruebas. Si el mismo digest se despliega en desarrollo y pasa pruebas automatizadas, hay evidencia de que **ese** contenedor cumple. La evidencia de certificación vale para producción solo si lo que se promueve es **exactamente** el digest probado.

### Secuencia de pruebas por etapa (diap. 15)

| Etapa | Pruebas |
|---|---|
| Integración continua | Unitarias, integración acotada y verificación de contratos. Sin ambiente. Segundos a minutos, en cada commit |
| Publicar | Imagen construida una vez e identificada por digest |
| Certificación | Digest desplegado, humo, E2E críticas y prueba de carga |
| Promover | El mismo digest al ambiente protegido, con aprobación |
| Producción | Prueba de humo, ventana de observación y verificación sintética |

Lo que corre antes de publicar decide si existe el artefacto: si falla, no hay imagen en el registro.

### Prueba de humo (smoke test)

- Pocas verificaciones, segundos, justo después de desplegar: ¿la aplicación levantó, la URL de salud responde 200, la versión es la esperada, alcanza sus dependencias, una lectura crítica sin efectos? Dura menos de un minuto.
- **No** verifica reglas de negocio ni escrituras con efecto real (eso es E2E). Son solo lectura.
- Si falla, dispara el **rollback** (volver al digest anterior).
- Script de la diapositiva 16: `curl -fsS --retry 5 --retry-delay 3 $URL/salud`, compara `/version` con la esperada, `/salud/dependencias`, y una lectura crítica como `/eventos?limite=1`.

### Verificación sintética

- Un recorrido crítico ejecutado cada pocos minutos contra producción, **haya o no despliegue**. Detecta certificado vencido o dependencia externa caída antes de que falle un cliente.
- Más larga y compleja que la prueba de humo. Usa **datos marcados como de prueba**: no cobra, no notifica y se excluye de las métricas de negocio.
- Su resultado es una señal de observabilidad (entra a la ventana de observación y al SLI/SLO). Herramientas: CloudWatch Synthetics (AWS), equivalentes en Azure y servicios gratuitos que mandan correo. Se configuran umbrales ("cada 5 min; si hay 3 errores seguidos, notifica") y se puede enganchar a acciones automáticas.

### Pruebas de rendimiento: cuatro preguntas distintas (diap. 18)

| Tipo | Pregunta | Perfil de carga | Cuándo corre |
|---|---|---|---|
| **Carga** | ¿Cumple el SLO con el tráfico esperado? | Sostenida al nivel esperado, 15 a 30 min | Antes de promover cambios relevantes (y en el primer lanzamiento) |
| **Estrés** | ¿Dónde se rompe y cómo falla? | Creciente hasta degradar | Periódica y antes de eventos |
| **Pico** | ¿Soporta un salto súbito? | De la base a 10 veces en segundos | Antes de un evento conocido (preventa, campaña) |
| **Resistencia** | ¿Se degrada con el tiempo? | Moderada durante horas | Programada: fugas de memoria y conexiones |

Ideas del profesor:
- **Carga vs. estrés** con un ejemplo: nuevo campus virtual con 1000 usuarios concurrentes esperados de matrícula → **carga** (hay un objetivo conocido; se prueba 500, 600, 800, 1000). Si no hay objetivo claro, o se quiere saber hasta dónde se rompe → **estrés** (100, 200… hasta 1000 o más). Pico: el ejemplo del link de e-commerce que apareció de repente en Yape.
- Se miden **percentiles** (p95 = 95% de las peticiones respondió bajo ese tiempo; el promedio oculta la cola lenta) y **varias señales a la vez**: latencia, throughput, tasa de error y saturación de CPU, memoria y **conexiones** (a menudo el cuello de botella, por ejemplo Apache/PHP).
- **El ambiente debe parecerse a producción en tamaño**; una prueba sobre un entorno 3 a 5 veces más chico no se extrapola. La nube ayuda: escalar el ambiente un rato y bajarlo.
- No correr desde la laptop: tiene un máximo de conexiones de salida (30–50 clientes concurrentes) y se vuelve el cuello de botella; se usan runners/contenedores distribuidos o servicios en la nube.
- **Limpiar la data** generada: identificar los datos de la prueba y lanzar un proceso que los borre (o borrar toda la base si es posible).
- Herramientas: **K6** (codificada, la más usada hoy; rampa, meseta, bajada; umbrales como `p(95)<400` y `http_req_failed rate<0.01`) y **JMeter** (más visual). El perfil declarado en el repositorio y los umbrales derivados del SLO hacen que el job falle si se incumplen. **Nunca en cada commit** (8 min y costo de infraestructura): antes de promover o programada. Se compara contra la línea base; una regresión de 20% es señal aunque cumpla el umbral.

## 4. Orquestación de pruebas en el pipeline

Principios (diap. 21):
1. **Lo rápido y lo que más falla corre primero** (un error de unitaria se conoce en segundos).
2. **Lo independiente corre en paralelo**: unitarias, integración y contratos no se esperan entre sí.
3. **Las E2E se reparten entre ejecutores** (matrix de GitHub Actions; 4 grupos bajan 30 min a ~8). El propio framework también puede paralelizar. El costo pasa a minutos de cómputo.

Pipeline de la diapositiva 22 (GitHub Actions):
- Jobs `unitarias`, `integracion`; `publicar` con `needs: [unitarias, integracion]` y `outputs: digest`; `desplegar-cert` con `needs: publicar` y `outputs: url`; `e2e` con `needs: desplegar-cert`, `strategy.matrix.grupo: [1,2,3,4]`, `env URL_API` tomada de la salida del despliegue y `actions/upload-artifact@v4` con **`if: always()`** para conservar la evidencia del rojo; `promover` con `needs: [publicar, e2e]` y `environment: produccion` (ambiente protegido con aprobación, mismo digest).

### Política de compuertas (diap. 23)

| Resultado | Efecto | Dónde actúa |
|---|---|---|
| Falla unitaria o de integración | **Detiene** | La publicación del artefacto |
| Cobertura del código nuevo bajo el umbral | **Detiene** | La publicación, vía quality gate (SonarQube/SonarCloud) |
| Falla un recorrido E2E crítico | **Detiene** | La promoción a producción |
| Latencia p95 o tasa de error sobre el umbral en la prueba de carga | **Detiene** | La promoción a producción |
| Falla una prueba en cuarentena | **Informa** | Registro con responsable y vencimiento (ejemplo: pruebas con ambiente dependiente que falla de forma eventual; falsos positivos, se revisan luego) |
| Falla la prueba de humo en producción | **Revierte** | Rollback: vuelve el digest anterior sin aprobación (ideal, aunque "muy pocos procesos" están en ese nivel) |

La política se escribe en el repositorio **antes de la primera falla**.

### Puntos clave de cierre (diap. 25)

1. El contrato protege a los consumidores sin ambiente compartido (solo mencionado).
2. La E2E se reserva para recorridos críticos que cruzan servicios.
3. Cada prueba tiene una etapa: antes de publicar o contra el ambiente desplegado; humo, E2E y carga reciben la dirección real del despliegue.
4. El mismo digest recorre todas las pruebas.
5. La política de compuertas se escribe antes de la primera falla.

## 5. Simulacro del Examen 1: caso Bodega Online

Material: caso modelo + guía de resolución del profesor (capturas). Se resolvió en grupos y luego en plenaria.

### Datos del caso

Bodega Online: app móvil de abarrotes en Lima. Servicio de pedidos (Python, contenedores) y servicio de inventario de otro equipo. 6 desarrolladores con ramas de ~2 semanas; 2 QA manuales en certificación compartido; revisión manual de seguridad con plazo de 3 días; despliegue manual por una persona de operaciones.

VSM (horas calendario):

| Paso | PT | LT | Espera | %C&A |
|---|---|---|---|---|
| Refinamiento | 3 | 48 | 45 | 80% |
| Desarrollo | 12 | 40 | 28 | 90% |
| Revisión de código | 2 | 30 | 28 | 85% |
| QA manual (compartido) | 8 | 96 | 88 | 60% |
| Revisión manual de seguridad | 2 | 72 | 70 | 90% |
| Despliegue manual | 3 | 26 | 23 | 75% |
| **Total** | **30** | **312** (13 días) | **282** | |

Datos del último mes (4 semanas): 6 despliegues a producción, 1 con rollback, 6 h para restaurarse, 10 commits con **mediana de 11 días** a producción. En cinco semanas llega el **Cyber Wow** (5 veces el tráfico normal); hay que reemplazar la columna `direccion` por `calle`, `numero`, `distrito` para asignar repartidores por zona. El equipo propone migrar eliminando `direccion` con **rolling** a las 9:00 del primer día.

Pipeline del caso (YAML): `on: push` en todas las ramas; build con `pip install -r requirements.txt` sin versiones fijas y tag con `run_number`; `pytest || true`; job de seguridad que solo publica un reporte; `deploy-prd` que **reconstruye** y sube `registro/cf-pedidos:prd`. Dockerfile: `FROM python:latest`, `COPY . .`, `apt-get install curl git`, `pip install pytest`, `ENV DB_PASSWORD=...`, sin `USER`, sin healthcheck, una sola etapa.

### A1 · VSM y eficiencia de flujo

- Se pide **diagramar** el VSM: un bloque por actividad (PT, LT, %C&A), un triángulo/campo de espera entre pasos y, ideal, una línea de tiempo. No importa lo bonito; importa simbolizar actividades y esperas.
- **Eficiencia de flujo = ΣPT / ΣLT = 30 / 312 × 100 ≈ 9,6%** (dentro del rango típico 1–15%, es baja: hay demasiadas esperas).
- **Dónde intervenir primero: QA manual.** Mayor espera (88 h, 28% del lead time) y menor %C&A (60%: 4 de cada 10 entregas vuelven a desarrollo). Intervención: pruebas automatizadas y **ambientes efímeros** en lugar de validación manual en el ambiente compartido; quality gate antes de QA para subir el %C&A. **Segunda prioridad válida**: la revisión manual de seguridad (70 h de espera), reemplazable por controles en el pipeline (caso B-2).

### A2 · Métricas DORA (clasificación del profesor)

| Métrica | Dato | Nivel (guía del profesor) |
|---|---|---|
| Frecuencia de despliegue | 6 en 4 semanas | Medio |
| Lead time para cambios | Mediana de 11 días | Medio (entre una semana y un mes) |
| Tasa de fallas en cambios | 1/6 | Media (el profesor dijo "como 15%") |
| Tiempo de restauración | 6 h | Alto (menos de un día) |

- **Métrica más crítica antes del Cyber Wow**: la **tasa de fallas en cambios** (con 5 veces el tráfico, cada despliegue fallido afecta a muchos más clientes) junto con el **tiempo de restauración** (6 h es el tiempo que tardaría en recuperarse; la restauración depende de una persona).
- **Relación con el VSM**: el lead time DORA coincide con el LT del proceso (en la guía: 40 + 30 + 96 + 72 + 26 = 264 h = 11 días, desde desarrollo hasta despliegue). VSM responde **por qué** el flujo es lento y dónde intervenir (se levanta con entrevistas, es una foto puntual y costosa); DORA responde **si está mejorando** (se extrae automático de las herramientas, idealmente mensual).
- Observación del profesor: la frecuencia del caso se cambió de trimestre a mes y el documento no se actualizó del todo, por eso había una versión antigua con tasa de fallas de 25% (1/4).

### A3 · Defectos del pipeline y del Dockerfile (guía del profesor)

| Defecto | Garantía que se pierde | Corrección |
|---|---|---|
| `requirements.txt` sin versiones fijas | Reproducibilidad | Lock file con versiones y huellas; instalar solo desde él |
| `pytest ... \|\| true` | Control: las pruebas nunca detienen | Quitarlo; publicar solo si el job de pruebas pasa (`needs`) |
| Análisis de dependencias solo como reporte | Control de seguridad | Compuerta con política por severidad y excepciones |
| `deploy-prd` no depende de pruebas ni seguridad | Las validaciones no protegen producción | `needs` sobre todas las compuertas; ambiente protegido |
| `on: push` en todas las ramas | Se despliega código no integrado | Desplegar solo desde `main` |
| `FROM python:latest`, herramientas en la imagen, sin `USER` | Reproducibilidad y superficie de ataque | Versión fija, dos etapas, usuario sin privilegios |

Otros defectos que aparecen en el caso y el profesor mencionó: el tag con el número de ejecución en vez del SHA del commit, la reconstrucción del artefacto en producción (debería promoverse el mismo digest), la contraseña hardcodeada en `ENV`, `COPY . .`, instalar `curl`/`git`/`pytest` en la imagen final y no tener healthcheck.

### A4 · Cambio de la columna `direccion`

- **Qué falla**: con rolling conviven versión vieja y nueva. Si la migración elimina `direccion`, las instancias antiguas aún activas **fallan** al tocar una columna que ya no existe, y **la reversión ya no es posible** porque el esquema cambió (solo podría corregirse hacia adelante).
- **Momento**: las 9:00 del primer día del Cyber Wow es el peor momento para un cambio de esquema. Se despliega **semanas antes** (la fecha de la campaña se mantiene; el cambio de esquema se adelanta).
- **Expand-contract**: (1) agregar `calle`, `numero`, `distrito` sin tocar `direccion`; (2) la versión nueva escribe en **ambas** estructuras y se migran los pedidos históricos; (3) la lectura y la asignación por zona pasan a las columnas nuevas, **con feature flag**; (4) `direccion` se elimina **después** del Cyber Wow en un despliegue propio.
- **Alternativas** que el profesor mencionó: feature flag (ambas funcionalidades en el código, se activa la nueva; si falla se apaga la bandera) y **canary release**, mezclado con feature flag.
- **Condición de reversión declarada antes**: por ejemplo, tasa de error al crear pedidos mayor a 1% durante 5 minutos, o pedidos sin distrito asignado (datos nuevos vacíos). El profesor también citó la **tasa de errores de la API** y revisar si los datos nuevos se registran vacíos.

### B-2 · Reemplazar la revisión manual de seguridad

| # | Control | Hallazgo | Decisión (guía) | Razón |
|---|---|---|---|---|
| 1 | Secretos | Contraseña de BD en un commit de hace un año, borrada después | **Detiene**; rotar la contraseña | Una credencial nunca se exceptúa; sigue en el historial |
| 2 | Composición (SCA) | CVE crítica en dependencia declarada; hay versión corregida | **Detiene**; subir a la versión corregida | Crítica, alcanzable por ser declarada y con corrección disponible |
| 3 | Análisis estático | 80 hallazgos preexistentes; ninguno nuevo en el PR | **Se registra e inventaría** | Heredado; ningún hallazgo nuevo |
| 4 | Análisis de IaC | Bucket con acceso público | **Detiene** | Exposición pública declarada en la infraestructura |

- **b) Papel del área de seguridad**: ya no es ejecutor/validador de cada salida, sino **definidor de la política como código** (umbrales, qué detiene y qué no, excepciones con los cuatro datos), que revisa los cambios de alto riesgo y las excepciones y audita; deja de ser una etapa de espera de 72 h. Su "biblia" es el NIST: la preparación previa (roles, herramientas, requisitos) ocurre antes del proceso de desarrollo.
- **c) Mapeo NIST SP 800-218 según la guía del profesor**: Composición → **PW.4** (reutilizar software existente y verificar componentes de terceros); análisis estático y secretos → **PW.7** (revisar/analizar el código legible para encontrar vulnerabilidades); análisis de IaC → **PW.7**. Para "identificación continua de vulnerabilidades" (RV.1), el profesor aclaró que **ese punto no se pide**: no se declaró como práctica en el caso.
- El SSDF agrupa 19 prácticas en cuatro familias (PO preparar la organización, PS proteger el software, PW producir software bien protegido, RV responder a vulnerabilidades). La numeración salta de PW.2 a PW.4 porque en v1.1 la antigua PW.3 se integró en PW.4. El profesor lo describió como la lista de buenas prácticas que usan los equipos de desarrollo y seguridad para evaluar procesos.

## 6. Qué llevarse para el parcial (síntesis)

1. **VSM**: calcular PT, LT, espera, eficiencia de flujo (ΣPT/ΣLT), identificar el cuello de botella con espera **y** %C&A, y proponer la intervención.
2. **DORA**: clasificar con la tabla del material de consulta; justificar la métrica crítica según el evento (tasa de fallas y restauración) y relacionar el lead time con el LT del VSM.
3. **Pipeline/Dockerfile**: reconocer defectos (versiones sin fijar, `|| true`, `latest`, `COPY . .`, secretos en `ENV`, sin `USER`, herramientas en la imagen, reconstruir por ambiente, tag no ligado al commit, seguridad solo como reporte) y explicar el impacto y la corrección sin escribir sintaxis.
4. **Cambio de esquema**: nunca eliminar una columna en el mismo despliegue; expand-contract, feature flag, despliegue semanas antes, métrica y umbral de reversión escritos de antemano.
5. **Seguridad en el pipeline**: decidir detiene / registra / excepción por tipo de control; la excepción lleva identificador, razón, responsable y vencimiento; una credencial nunca se exceptúa. Saber ubicar cada control en una práctica del NIST del anexo.
6. **Pruebas**: qué corre antes de publicar, qué contra el ambiente desplegado, qué detiene qué, y la diferencia entre carga, estrés, pico y resistencia.
