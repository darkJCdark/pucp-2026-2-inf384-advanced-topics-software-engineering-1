# INF384 · Sesión 06 — Despliegue continuo: estrategias, aprobaciones y validación

Curso: INF384 Temas Avanzados en Ingeniería de Software 1 (PUCP, 2026-2) · Prof. José Felipe del Castillo Salazar
Capítulo 4 (2 de 2) · 3 horas · RA2, RA4 · Clase dictada el 26/9/2026, 6:50 p.m., duración 136 min.
Fuentes: transcripción de la clase (`Temas S6.txt`), diapositivas (`INF384_S6_Despliegue_continuo_II.pdf`), notas propias (`Clase 6.txt`).

## 0. Repaso del Laboratorio 3 (primeros ~35 min de clase)

El profesor mostró su propia resolución del Lab 3 y la usó para repasar conceptos, sin dar por sentado que la forma "correcta" es una sola.

**Terraform State — preguntas y respuestas trabajadas en vivo:**
- El `setup-infra` (creación del bucket S3 y el repositorio ECR) se corre **una sola vez, fuera** del proyecto Terraform del laboratorio, porque Terraform necesita un lugar ya existente donde guardar su estado antes de poder ejecutarse.
- El **Terraform State** es el registro de qué recursos ya fueron creados en la nube por ese proyecto. Sin él, Terraform no tendría forma de saber si algo ya existe o no. Terraform es agnóstico a la nube: por eso el estado se guarda aparte (en este caso en S3; en otros proveedores podría ser un blob storage, GCS, o el propio backend de HashiCorp).
- El backend de estado resuelve dos problemas: (1) dónde vive el estado de forma centralizada (no en la máquina local ni en el repo) y (2) qué pasa si dos personas ejecutan un cambio a la vez — para eso se usa un **lock** (semáforo/bandera; lo ideal es DynamoDB, aunque el laboratorio usó el lock nativo de S3 para no complicar el setup).
- Si se destruye infraestructura manualmente desde la consola de AWS, el Terraform State **no se entera**; recién al correr `plan`/`apply` Terraform intenta emparejar su registro con la realidad y puede intentar volver a crear lo que "cree" que falta.
- El bucket para el state también guarda una primera imagen Docker de referencia, porque la primera vez que se despliega el Lambda necesita que exista *alguna* imagen en el ECR; si no, el despliegue inicial falla.
- Comentario de arquitectura (anécdota de una excolega, empresa Caleidos): en escala empresarial real se arman **árboles de proyectos Terraform** — uno para networking, otro para observabilidad, otro para seguridad/roles, otro para el setup de herramientas DevOps, etc. — donde el output de un proyecto es input de otro. El laboratorio es la versión mínima de esa idea.
- Analogía usada: automatizar la creación de infraestructura es como construir una casa por partes (cimientos, tuberías, etc.), no todo de un golpe.

**Sobre los jobs del pipeline de aplicación (`app.yml`) y de infraestructura (`infra.yml`), repasados en vivo:**
- Las 4 compuertas de A2 (`calidad`, `secretos`, `dependencias`, `estatico`) corren en **paralelo**, no en secuencia, salvo cuando hay una dependencia real de datos: `calidad` sí necesita esperar a `pruebas` porque usa el archivo de cobertura que generan las pruebas unitarias como input para evaluar si se cumple el umbral.
- Regla general: el pipeline debe ser lo más rápido posible — paralelizar todo lo que no tenga una dependencia real. En el ejemplo del profesor, correr todo en secuencial hubiera tomado casi un minuto más que en paralelo (~1:38 promedio).
- En `infra.yml`: `validar` ejecuta primero `init` (equivalente a un `install` — descarga providers e inicializa el state) y luego `validate` (compila/chequea sintaxis, referencias a variables inexistentes, etc. — no es un `audit`, es más parecido a una verificación de compilación). `politicas` es el análisis de seguridad de la infraestructura declarada (no del código de la app, sino de si el recurso que se va a crear queda mal configurado, p. ej. un bucket o Lambda expuesto públicamente). `plan` muestra qué cambios se van a aplicar. `apply` es la ejecución final; técnicamente se podría invocar antes que `plan`, pero pierde sentido porque `plan` es lo que demuestra qué se va a hacer antes de hacerlo.
- Buenas prácticas mencionadas para hacer visibles los resultados de cada gate: que el job escriba un resumen legible en `$GITHUB_STEP_SUMMARY` o en el log de la ejecución (p. ej. "no se encontró ningún secreto en el código"), no solo que falle o pase en silencio.

## A. Estrategias de despliegue

Idea central de la sección: **desplegar** (llevar el artefacto al ambiente y ejecutarlo — operación técnica, riesgo de disponibilidad) y **publicar** (poner la funcionalidad a disposición del usuario — decisión de producto/negocio, riesgo de experiencia) son dos eventos separables. Un *feature flag* es lo que permite desacoplarlos: apagar una bandera toma segundos y no requiere pipeline; revertir un despliegue toma minutos y sí lo requiere.

Las estrategias de despliegue existen para controlar cuántas instancias y cuántos usuarios reciben la versión nueva mientras se verifica que funciona, es decir, resuelven el **riesgo técnico** del cambio (el feature flag resuelve el riesgo de negocio/funcional).

### Las cuatro estrategias vistas

1. **Reemplazo total / Rolling clásico / Multi-Service Deployment** (varios nombres para lo mismo)
   Se baja la versión anterior y se sube la nueva sobre el mismo servidor/instancia.
   - Hay una ventana **sin servicio**, del tamaño del tiempo de arranque.
   - No conviven dos versiones a la vez → no hay restricción de compatibilidad entre versiones ni de esquema de datos.
   - Revertir exige repetir toda la operación (con una segunda ventana de indisponibilidad).
   - Es la que "todos hemos hecho siempre" cuando no se piensa en estrategia.

2. **Reemplazo progresivo (rolling deployment)**
   Se reemplazan las instancias por tandas, no todas de golpe.
   - No hay ventana sin servicio: mientras unas instancias se actualizan, otras siguen atendiendo.
   - Las dos versiones **conviven temporalmente** y deben ser compatibles entre sí y con los datos.
   - Es el comportamiento por defecto de la mayoría de orquestadores (p. ej. lo que AWS hace internamente con Lambda: despliega el Lambda nuevo en paralelo y recién destruye el anterior cuando el nuevo ya está operativo — el usuario nunca lo nota si está monitoreando constantemente).

3. **Blue-Green Deployment ("dos ambientes completos y un conmutador")**
   Existe un ambiente activo (verde) sirviendo el 100% del tráfico y un ambiente nuevo (azul) donde se despliega y se prueba **sin tráfico real**. Cuando está validado, se conmuta todo el tráfico de una vez. El ambiente anterior no se destruye inmediatamente — se mantiene un tiempo por si hay que revertir (el "switch" de vuelta).
   - Necesita el **doble de servidores** (misma capacidad de cómputo que producción, en paralelo) → es la estrategia más cara.
   - Transición y reversión son **instantáneas** (solo cambia el conmutador).
   - Sesiones activas y datos **no se conmutan** automáticamente: sigue habiendo problemas de compatibilidad de datos.
   - Es la que da mayor certeza porque el ambiente nuevo se prueba completo antes de exponerlo a clientes reales.

4. **Canary Deployment**
   Con múltiples réplicas del mismo servicio (p. ej. 4 copias), se despliega la nueva versión primero en una sola, se monitorea cómo responde con tráfico real parcial, y si va bien se va escalando (2, 3... hasta reemplazar todas). El nombre viene de los canarios que los mineros llevaban para detectar gases tóxicos: si el canario "sobrevive" (la réplica nueva responde bien), es seguro seguir avanzando.
   - Expone el riesgo solo a un **porcentaje acotado** de usuarios/tráfico, no a todos.
   - Requiere poder direccionar tráfico, medir cómo responde cada réplica, y decidir si escalar.
   - Ejemplo trabajado en clase: exposición gradual con ventanas de observación — 5% (10 min), 25% (20 min), 50% (30 min), 100% (con señal estable en cada paso).
   - Sin volumen de tráfico suficiente, el 5% inicial no produce muestras suficientes para distinguir una falla real de variación normal — la estrategia deja de aportar información.
   - La contraparte obligatoria de esta estrategia es la **reversión automática**: si el umbral se rompe, el tráfico vuelve solo a la versión anterior sin esperar aprobación humana — eso es lo que reduce el tiempo de restauración.

5. **A/B Testing** (mencionada como variante/extensión del Canary, con otro objetivo)
   Varias versiones conviven en paralelo por un tiempo prolongado, cada una recibiendo un porcentaje del tráfico (ejemplo dado: 50% v1.1, 25% v1.2, 25% v1.0), y se va monitoreando el comportamiento de cada una antes de decidir a cuál migrar por completo.
   - Nace de marketing (probar qué variante convierte mejor), no solo de DevOps — puede evaluar aspectos funcionales, no solo técnicos.
   - Es la más compleja de las cuatro/cinco: hay que desplegar cada versión y sostener varias en paralelo durante un tiempo.

### Comparación (tabla trabajada en clase y en las diapositivas)

| Criterio | Reemplazo total | Progresivo | Blue-Green (dos ambientes) | Canary |
|---|---|---|---|---|
| Indisponibilidad | Sí | No | No | No |
| Convivencia de versiones | No | Sí | Durante la transición | Sí, prolongada |
| Costo de infraestructura | Bajo | Bajo | Alto (temporal) | Medio |
| Velocidad de reversión | Lenta | Media | Inmediata | Inmediata |
| Exige observabilidad | Deseable | Deseable | Necesaria | Indispensable |
| Usuarios expuestos a un fallo | Todos | Parte creciente | Todos tras conmutar | Porcentaje acotado |

Criterio para elegir: no se configuran las cuatro estrategias a la vez por aplicación (es muy costoso de mantener) — se decide **una estrategia por aplicación**, de antemano, en función de: (1) el costo/presupuesto disponible, (2) las capacidades ya existentes (¿hay observabilidad o no?), y (3) el riesgo del cambio/del sistema (ejemplo extremo dado en clase: sistemas de radares de aviones — ahí no importa el costo, se va directo a Blue-Green o Canary).

### El límite real de todas las estrategias: el esquema de datos

Ninguna estrategia de despliegue resuelve el problema de los datos: **el código se revierte en minutos; el esquema de datos, no.** Una migración que elimina o renombra una columna rompe la versión anterior y con ella cualquier posibilidad de reversión.

Patrón de migración segura discutido (expand/contract), en 4 pasos:
1. **Expandir**: se agrega la estructura nueva sin tocar la existente. Ambas versiones del código funcionan contra el mismo esquema.
2. **Migrar y escribir en ambas**: la versión nueva escribe en las dos estructuras mientras se copian los datos históricos; la versión anterior sigue operando sin enterarse.
3. **Trasladar la lectura**: cuando los datos están completos, la versión nueva lee de la estructura nueva. Sigue siendo posible volver atrás.
4. **Eliminar la versión anterior**: solo cuando ninguna versión desplegada usa ya la estructura antigua. Es un cambio propio, con su propio despliegue.

Anécdota del profesor: migración de un servidor de datos on-premise (4 TB) a la nube para una empresa de alimentos que operaba 24/7 — tuvo que hacerse en una ventana de domingo (única franja donde el negocio aceptaba parar), y debieron ejecutar **tres rollbacks** por problemas de throughput de disco no identificados al mover la infraestructura a máquinas virtuales en la nube.

### ¿Reversar o corregir?

(Contenido también resumido en la nota personal `Clase 6.txt`.)

- **Volver a la versión anterior (rollback)**: se despliega el artefacto anterior, que ya existe en el registro y ya fue validado. El tiempo es el de un despliegue. Requiere que el artefacto anterior siga disponible y que el esquema de datos lo admita. Es la opción **por defecto durante un incidente**.
- **Avanzar con una corrección (roll-forward)**: se escribe la corrección y se despliega una versión nueva. El tiempo incluye diagnosticar, corregir y recorrer el pipeline completo. Requiere un pipeline rápido y confiable porque corre bajo presión. Es la **única opción cuando el esquema de datos ya cambió**.
- La decisión de qué hacer (reversar o corregir, "go o no go") **se toma antes del incidente, no durante**: en la planificación del pase se define de antemano qué condición dispara la reversión y quién puede ejecutarla. "A las dos de la madrugada no se diseña política."
- Un mecanismo de reversión que nunca se ejecutó es una suposición: **la reversión se ensaya**.
- Regla práctica que dio el profesor con ejemplos bancarios (cálculo de intereses diarios, cierres de mes): hay procesos con una ventana de tiempo estricta que no pueden pasarse de cierto momento del día; eso determina si en un incidente se puede esperar/corregir o hay que reversar sí o sí.
- Ejemplo real (paso a producción de un e-commerce de venta de seguros por internet, migración de dominio): se usó un formato de planificación de pase con actividades detalladas — qué se hace, quién es responsable, hora de inicio y fin, actividades previas (todo lo que se puede adelantar antes del día D se adelanta), bajar la aplicación, pruebas de verificación (2 horas, manuales en ese caso), y el momento del **go/no-go** con ventana de rollback contemplada.

## B. Flujo de aprobaciones

### Qué aporta una aprobación y qué no

**Sí resuelve:**
- Segregación de funciones: quien escribe el cambio no es quien lo autoriza.
- Registro auditable de quién decidió, cuándo y sobre qué versión exacta.
- Coordinación con negocio cuando el cambio tiene fecha/ventana acordada (ejemplo dado: no hacer un pase un viernes de Black Friday por el riesgo de aumento de transacciones).

**No resuelve:**
- No detecta defectos: quien aprueba no ejecuta el código ni revisa las pruebas.
- No reduce el riesgo del cambio por sí sola; lo traslada de fecha y lo acumula con otros cambios.
- No sustituye una validación automática ausente, aunque se sienta como que sí.

Dato del sector citado en clase (bibliografía del curso, libro *Accelerate*): la aprobación de un comité externo al equipo **no mejora la estabilidad** y sí alarga el tiempo de entrega. La alternativa no es eliminar el control, sino cambiar su forma: revisión por pares dentro del equipo, validación automática y despliegue reversible producen la misma evidencia de control sin la espera. En contextos regulados, la aprobación se conserva pero se automatiza la evidencia (el pipeline registra qué versión se aprobó, quién la aprobó y qué validaciones pasó, sin trabajo manual adicional).

Frase clave: *"Una aprobación que nadie puede rechazar en la práctica es un paso de espera con apariencia de control."*

### El ambiente protegido como compuerta (mecánica en GitHub)

En GitHub esto se implementa con **rulesets** (protección de ramas) — funcionalidad no disponible en repos privados de la versión gratuita, sí en Teams/Enterprise:
- **Revisores requeridos**: el despliegue se detiene hasta que una persona o equipo designado lo autoriza; queda registrado quién fue.
- **Espera obligatoria**: tiempo mínimo entre la aprobación y la ejecución, para poder cancelar una aprobación apresurada.
- **Restricción de rama**: solo la rama principal puede desplegar a producción — cierra el camino de desplegar desde una rama sin validar. Esto es lo que impide el `push` directo a `main` y obliga a pasar por pull request (para que el pipeline corra en el PR y las validaciones ocurran ahí antes de poder aprobar).
- **Secretos por ambiente**: las credenciales de producción solo existen dentro de ese ambiente; un job que apunta a otro ambiente no puede alcanzarlas.
- Idea clave: la protección vive en la **plataforma** (configuración de GitHub), no en el archivo del pipeline — quien edita el YAML no puede levantarse a sí mismo el gate. Ahí está la segregación real.
- Quien aprueba necesita ver **qué** está aprobando: versión, diferencia respecto de lo desplegado y resultado de las validaciones. Una aprobación sin esa información es solo una firma.

### Con qué credenciales despliega el pipeline

- **Llave estática guardada como secreto** (lo que hace el laboratorio, por limitación de AWS Academy): vive hasta que alguien la rota, y casi nadie la rota. Quien la extrae una vez puede usarla desde cualquier lugar y en cualquier momento. Su alcance suele ser mucho mayor que lo que el pipeline realmente necesita.
- **Identidad federada de corta duración** (la práctica recomendada, no aplicable en el laboratorio): el pipeline presenta un token firmado que declara repositorio, rama y ejecución; la nube lo verifica y entrega credenciales que expiran en minutos. No hay secreto permanente que robar ni que rotar. La confianza se declara para un repositorio y una rama específicos. En AWS esto se implementa con un **conector/rol de GitHub** (identidad federada: se autoriza una vez el link entre la cuenta de GitHub y el rol de AWS; la contraseña nunca queda almacenada, solo la autorización; se puede filtrar a qué repositorios se les da acceso y con qué permisos).
- Principio general (retomado de la sesión 4): el pipeline es superficie de ataque, por lo que el permiso se acota a lo que ese trabajo necesita (p. ej. "publicar una imagen y actualizar un servicio", no administrar toda la cuenta) — en el laboratorio, en cambio, las credenciales usadas son de administrador del entorno académico, porque el entorno no permite configurar identidad federada; esa limitación es parte intencional del aprendizaje del Laboratorio 3, no la práctica recomendada.

### Anatomía del trabajo de despliegue (ejemplo de GitHub Actions mostrado en diapositivas)

```yaml
jobs:
  desplegar:
    needs: publicar
    if: github.ref == 'refs/heads/main'
    environment: produccion              # 1 compuerta de aprobación declarada en la plataforma
    steps:
      - name: Obtener credenciales
        uses: <accion-de-credenciales>    # 2 credenciales de corta duración
      - name: Registrar la version nueva
        run: <registrar-definicion>
          --imagen $REG/app@$DIGEST       # 3 se despliega por huella (digest), no por etiqueta
      - name: Actualizar el servicio
        run: <actualizar-servicio>
      - name: Esperar estabilidad
        run: <esperar-servicio-estable>   # 4 el pipeline espera el resultado real
      - name: Prueba de humo
        run: curl -fsS $URL/salud
      - name: Revertir si fallo
        if: failure()                     # 5 la reversión es parte del mismo trabajo
        run: <volver-a-la-revision-previa>
```

Puntos clave marcados sobre este job: (1) la compuerta de aprobación se declara en la plataforma (el `environment: produccion`), no en el archivo; (2) las credenciales se obtienen al inicio del trabajo y expiran con él; (3) se identifica el artefacto exacto que pasó las validaciones por su huella (digest), no por una etiqueta mutable; (4) el pipeline espera el resultado real de que el servicio arrancó, no termina antes de saberlo; (5) si la verificación falla, la reversión a la versión anterior es parte del mismo trabajo, no un paso manual aparte.

## C. Observabilidad

### Los tres pilares

| Señal | Pregunta que responde | Qué es |
|---|---|---|
| **Métricas** | ¿Hay un problema? | Series numéricas agregadas en el tiempo: tasa de error, latencia, saturación de recursos. Baratas de guardar, responden rápido, no dicen por qué. |
| **Trazas** | ¿Dónde está el problema? | El recorrido de una petición por todos los servicios que la atendieron, con el tiempo que pasó en cada uno. Localizan el componente responsable. |
| **Registros/Logs** | ¿Qué exactamente ocurrió? | El detalle de un evento puntual. Caros en volumen y de poco uso si no se pueden correlacionar con la petición que los originó. |

- El **identificador de correlación** es lo que vuelve útil al conjunto: sin un identificador común que atraviese los servicios, las tres señales son tres archivos que nadie puede cruzar durante un incidente.
- La **versión desplegada** se debe emitir como atributo de cada señal: es lo que permite afirmar que un error aparece en la versión nueva y no en la anterior — sin eso, la comparación es solo una impresión. Este punto conecta directamente con la trazabilidad de artefactos vista en el Capítulo 4 completo.
- Diferencia monitoreo vs. observabilidad (discutida en vivo): tener dashboards de CPU/RAM es monitoreo; saber cuántas transacciones están ocurriendo, tener alertas configuradas, y poder ejecutar un rollback automatizado cuando se detecta un problema, es observabilidad. Sin observabilidad, las estrategias Blue-Green y Canary son **imposibles de ejecutar de verdad**.
- Se puede construir métricas a partir de logs (ej. contar cada vez que aparece la palabra "error" en un log como una métrica); en la práctica se combina más logs con métricas.
- Reto de costo mencionado: herramientas de observabilidad completas (ejemplo dado: Dynatrace) pueden costar del orden de miles de dólares al año por unidades pequeñas de RAM monitoreada — el uso de IA sobre estas tres señales (ejemplo de un chatbot interno que correlaciona monitoreo de infraestructura y aplicación, y hasta el registro de pases a producción, para responder preguntas de diagnóstico) es una tendencia que se apoya sobre esta base, no la reemplaza.
- Retos propios de observabilidad: la **correlación** entre componentes muy pequeños/distribuidos (microservicios) y saber **qué versión** de la aplicación generó cada medida (no es la misma aplicación hoy que ayer).

### Cómo entra la observabilidad al pipeline

- La validación técnica post-despliegue (¿el servidor está arriba?, ¿las métricas de consumo están bien?, ¿la API sigue respondiendo 200?) es parte de la observabilidad, no es una validación funcional/de negocio.
- El propio pipeline también **alimenta** la observabilidad: debe registrar que hubo un pase (vía API o una acción específica), para poder correlacionar después un problema en producción con el pase que lo originó.
- Secuencia de verificación de un despliegue (misma disciplina del *quality gate* visto en el Capítulo 3, aplicada ahora al comportamiento en producción): **Desplegar → Esperar estabilidad → Prueba de humo → Ventana de observación → Confirmar o revertir.**
  - La verificación técnica (¿arrancó el proceso?) no es la de negocio (¿hace lo correcto?) — son dos comprobaciones distintas con dos umbrales distintos.
  - Duración, métrica y umbral de la ventana de observación se escriben **antes**, en el repositorio.
  - La reversión automática se ejecuta **sin consultar**: si esperar una aprobación es parte del camino de reversión, el tiempo de restauración incluye el tiempo de encontrar a esa persona.
  - La proporción de despliegues que requieren reversión o corrección urgente es la **tasa de fallo de cambios** vista en la sesión 1.

### SLA, SLO, SLI

- **SLA (Service Level Agreement)**: la promesa hacia el cliente, de cara externa (ej. 99.5% de uptime). La escribe/negocia normalmente legal + producto.
- **SLO (Service Level Objective)**: el objetivo interno del equipo, deliberadamente **más agresivo** que el SLA (ej. 99.9%), para dar margen de no incumplir la promesa al cliente. Analogía usada: si en un examen la meta es sacar 15 apuntando exactamente a 15, es probable sacar 14 — hay que apuntar más alto (16-17) para asegurar el 15.
- **SLI (Service Level Indicator)**: el dato real medido (ej. 99.7%), el que entregan las herramientas de monitoreo.
- La diferencia entre SLA y SLO es el **margen de error** del equipo. Cuanto más se reduce esa diferencia, más presión tiene el equipo para no fallar.
- El **presupuesto de error** (*error budget*) es la diferencia entre el objetivo y la perfección: la cantidad de fallo que la organización aceptó gastar, y se gasta desplegando. Mientras queda presupuesto, el equipo despliega a su ritmo; cuando se agota, la prioridad pasa a estabilidad hasta recuperarlo. Esto convierte la discusión entre velocidad y estabilidad en un número acordado de antemano, en vez de una negociación caso por caso ante cada incidente.
- Discusión adicional: casi ninguna herramienta pública cumple literalmente su SLA publicado; muchas empresas grandes (ejemplo: Cloudflare) ni siquiera publican un SLA formal al cliente. Herramientas de "status page" externas (no oficiales) permiten monitorear la disponibilidad real de un servicio de terceros de forma independiente.
- Demos mostradas en clase (sin datos en vivo por confidencialidad): AWS CloudWatch (ojo: cobra por ingesta de logs — hay que filtrar para no ingerir todo, especialmente con Kubernetes, que genera muchísimo log; normalmente no tiene sentido ingerir nivel `info`) y Azure Application Insights (ejemplo real de backend de una fintech: tasa de invocaciones, tiempo de respuesta por componente, tasa de fallos por API, "Smart Detection" con IA sobre el comportamiento, dashboards comparando dos versiones de runtime en paralelo durante una migración — .NET 6 a .NET 10 — para decidir cuándo retirar la versión antigua). Mención breve de **FinOps**: ajustar capacidad de cómputo según horario (ej. bajar casi a cero la capacidad nocturna de un servidor de oficina).

## Cierre de la sesión — Puntos clave (diapositiva final)

1. **Desplegar y publicar se pueden separar.** La bandera de funcionalidad y la estrategia de despliegue son dos mecanismos de reversión independientes.
2. **La estrategia se elige por el costo del fallo y la capacidad de detectarlo.** El nombre de la estrategia no indica madurez; la señal disponible sí indica viabilidad.
3. **El esquema de datos es el límite de toda reversión.** El código vuelve atrás en minutos; la migración aplicada, no.
4. **La aprobación aporta trazabilidad, no detección.** Y cuando es externa al equipo, la evidencia del sector muestra que alarga la entrega sin mejorar la estabilidad.
5. **Sin señal/alerta declarada, el despliegue automático entrega el error más rápido.** La ventana de observación y su umbral se escriben antes, igual que el quality gate.

## Contexto adicional / lo que viene

- **No hay laboratorio la próxima semana** (feriado, y después parciales) — pausa larga antes del Laboratorio 4.
- Próximos temas anunciados: automatización de calidad (pruebas unitarias, integración, performance, UI), automatización de seguridad (incluye evaluar el propio Docker), práctica de DevOps llevada a términos de negocio, DevSecOps en las organizaciones, IA en DevSecOps.
- El profesor va a coordinar 2-3 charlas cortas (~20-30 min, presenciales) con invitados: una enfocada en historia de vida/motivacional (transición a DevOps sin venir de TI), una sobre cómo se entrevista a perfiles DevOps en el mercado (selección de personal), y otra por definir con un tema de IA/guía.
- Temas mencionados como futuros, fuera del alcance evaluado de esta sesión: Kubernetes (a nivel de noción, no certificación — analogía usada: Kubernetes es como un tráiler gigante, solo tiene sentido si realmente se necesita llenar ese espacio; decisiones reales de cuántos clusters usar dependen de arquitectura, no solo de tecnología), **Golden Paths** (receta/estructura estándar para que todos los servicios nazcan con el mismo pipeline y tecnología, reduciendo el costo de mantener miles de servicios distintos), y **Platform Engineering** (evolución de DevOps: el desarrollador solo declara qué quiere lograr dentro de una plataforma ya gobernada —conectada a Jira u otra herramienta—, y la IA dispara el desarrollo, la infraestructura y el despliegue dentro de reglas y aprobaciones ya establecidas; el reto de fondo es dar velocidad con IA sin perder control ni exponer información sensible).
