# Semanas 4 y 5 — Integración Continua II (seguridad) y Despliegue Continuo I (artefactos e IaC)

## Sesión 4: Integración Continua II — Validación de seguridad

### Contexto: cierre del Laboratorio 2
- Se trabajaron los 4 defectos del pipeline.
- Al evaluar duraciones: un pipeline puede volverse más lento pero, a cambio, más seguro (trade-off aceptable).
- Relación con DORA: los temas de integración continua ayudan a mejorar la tasa de despliegue y la tasa de fallo; el tiempo mínimo de recuperación a veces no depende de la integración continua.
- Lo que no queda resuelto solo con CI: no hay garantía de detectar si alguien subió una credencial, ni certeza total de que el paquete final publicado sea exactamente el que se quiere.

### Entregación continua y el costo del hallazgo según su ubicación
- El mismo hallazgo (bug, vulnerabilidad) cuesta distinto según en qué punto del flujo aparece: en el commit, antes de fusionar, o antes de publicar.
- Ejemplo: un pull request se aprobó sin revisar vulnerabilidades, pero la etapa de validación posterior sí las detectó, lo que obliga a redirigir al autor o corregir después.
- Mientras más a la derecha (más tarde) se identifica el error, mayor es el tiempo de espera generado y más se afecta el Value Stream Mapping.

### Cuándo un control de seguridad pertenece a la etapa de CI
1. **Rápido**: un control que demora mucho hace que el desarrollador evite subir código, agrandando la cola de trabajo.
2. **Determinista**: el resultado debe ser blanco o negro, sin ambigüedad.
3. **Acotable al cambio**: debe poder identificar el delta del cambio.
   - *New code*: evalúa solo lo introducido en el pull request.
   - *Overall code*: evalúa todo el repositorio.
4. **No exige ambiente desplegado**: por ejemplo, el Ethical Hacking (pruebas de penetración) requiere que el proyecto ya esté desplegado, por lo que no aplica en esta etapa.

### Controles que califican y no califican para esta etapa
- **Detección de secretos**: controles que impiden subir secretos al repositorio; no existe una herramienta perfecta; revisa tanto el código nuevo como el historial.
- **Análisis de composición de librerías (SCA)**: identifica vulnerabilidades en las dependencias/componentes usando bases de conocimiento públicas. En Docker, la imagen almacena librerías; si el artefacto no es seguro, puede desactivarse. Herramienta mencionada: Dependency Tracker (en red privada), que muestra componentes/microservicios con su cantidad de vulnerabilidades y un score de qué tan riesgosa es cada una.
- **Análisis estático de seguridad (SAST)**: similar en concepto a SonarQube Cloud pero enfocado en hallazgos de seguridad del código fuente.
- **Análisis de configuración e IaC**: herramienta mencionada, Checkov, evalúa la infraestructura como código.
- El **análisis dinámico** (requiere aplicación desplegada) NO pertenece a esta etapa.

### Diseño del hallazgo: severidad no es lo mismo que prioridad
- Severidad declarada: ¿se está explotando la vulnerabilidad ahora mismo?
- Alcanzabilidad: ¿es posible llegar a ese punto vulnerable del código?
- Exposición: ¿qué tan fácil es llegar a él?
- Existencia de corrección: ¿ya existe un fix disponible?

### Diseño del control: por qué son dos "compuertas" (gates) y no una
- Análisis de calidad: un falso positivo mal gestionado se acumula y se percibe como ruido.
- Análisis de seguridad: un falso positivo mal gestionado también se acumula; es un reto característico de las herramientas de seguridad.
- Ejemplo de pipeline (representado con drawio): Build App (¿se genera el compilado en el PR?) → Escaneo SAST (vulnerabilidades del código fuente).

### Diseño del gate de seguridad
- Disyuntiva: **bloquear vs. registrar**.
  - No conviene bloquear todo desde el primer día.
  - Tampoco conviene "registrar siempre, detener nunca" (sin ningún mecanismo de corte).
- Principio aplicado: **"detiene lo nuevo, inventaría lo heredado"** — lo nuevo se bloquea si falla, lo ya existente se registra como inventario a resolver progresivamente.
- Se recomienda un umbral distinto por tipo de control, no un umbral único general.

### Política del gate
- Las excepciones deben tener **fecha de vencimiento** (se mencionó Checkov como ejemplo de herramienta que soporta este tipo de política).

### NIST SP 800-218 — Secure Software Development Framework (SSDF)
Marco de referencia para ubicar dónde cae cada control:
- Reutilizar software de manera segura.
- Revisar y analizar el código.
- Ejecutar pruebas automatizadas en un ambiente.
- Analizar el código constantemente, porque pueden surgir nuevas vulnerabilidades con el tiempo.

### Caso 1 (trabajado en clase)
- Contexto VSM: 5 equipos de desarrollo, un equipo de QA, y un equipo de Infraestructura que ocupa una ventana de tiempo.
- Flujo: Dev envía cambios a QA; Infra reporta vulnerabilidades a Dev; el cuello de botella está en QA.
- Validaciones de seguridad disponibles: dependencias (SCA) sí disponible, SAST sí disponible; configuración del servidor no se puede atacar porque requiere el servidor corriendo.
- El pipeline de escaneo de seguridad (SCA y SAST) se ubica en la etapa de Dev; flujo Dev → QA → Seguridad, con un Quality Gate.
- Concepto de "ventanas de freezing" (por ejemplo, 4 ventanas de prueba por semana).
- Respuestas discutidas: (1) solo se escanea la configuración del servidor cuando corresponde; (2) todo lo antiguo se registra, todo lo nuevo se retiene si falla; (3) depende de la severidad — condicionales solo al ID de la vulnerabilidad; (4) métricas DORA relevantes: Lead Time y Tiempo de restauración.

### Caso 2 (trabajado en clase)
- Controles aplicables: dependencias, y SAST enfocado en credenciales.
- Gate: se compara una "radiografía" de la foto actual (ej. 10 issues críticos identificados con nombre y apellido) contra el estado tras el cambio (10 vs. 10+1) — se detiene lo nuevo y se registra lo viejo.
- Caso concreto: se aprobó un pipeline que llegó a producción un día determinado; tres semanas después, un analista de seguridad hizo pruebas manuales (las automatizables y las no automatizables) y sí encontró vulnerabilidades adicionales.
- ¿Por qué el analista encontró algo que el pipeline no? Porque la base de datos de vulnerabilidades usada en producción no es la misma que la del analista, y en esas tres semanas nacieron nuevas vulnerabilidades.
- Conclusión: la responsabilidad siempre recae en el proceso, no en una persona puntual.
- Métrica DORA relevante: Frecuencia de despliegue.

### The Three Ways of DevOps (marco general)
1. Flow and System Thinking (pensamiento de flujo y sistema).
2. Amplify Feedback Loops: evitar que las conversaciones de mejora ocurran solo eventualmente cada vez que surge un problema, sin mejora continua real.
3. Culture of Continual Experimentation & Learning: equipo que busca constantemente mejoras incrementales.

### Conclusiones de la sesión
- Que el pipeline "salga bien" (en verde) no significa que todo esté bien.
- Calidad y seguridad son conceptos diferentes.
- Algunos controles de seguridad se pueden implementar en esta etapa y otros no.
- Es importante priorizar qué issues de seguridad atender primero; existen herramientas que indican cuáles issues son explotables desde internet (mayor riesgo real).

---

## Sesión 5: Despliegue Continuo I — Artefactos e Infraestructura como Código (IaC)

### Contexto: laboratorio de la sesión
- Se trabaja con 2 pipelines y se estudia cómo un step depende de otro.
- Se revisan comandos de Docker y acceso a AWS (localstack o similar).

### Dónde estamos en el pipeline
- El archivo de bloqueo (lock) sigue siendo la pieza que fija las versiones de las librerías.

### Entrega continua vs. despliegue continuo
- En Perú, "CD" suele entenderse como Continuous Delivery.
- Despliegue continuo (Continuous Deployment): requiere alta madurez del equipo/proceso; no necesita revisión manual antes de pasar a producción.

### Qué es exactamente un artefacto desplegable
- **Imagen de contenedor**: contiene la aplicación junto con sus dependencias.
- **Plantilla de infraestructura**: por ejemplo, una carpeta compartida donde se pueden almacenar archivos (file storage).
- Si el artefacto se "rompe", genera problemas en QA.

### Tag vs. huella criptográfica (hash)
- `latest`: apunta a la última versión estable.
- `sha` (hash del commit): mejor opción para trazabilidad exacta de qué código generó ese artefacto.

### VM / contenedores / serverless
- Serverless: no hay que preocuparse por lo que corre por debajo; el foco está solo en la aplicación.

### Docker: Dockerfile, imagen y contenedor
- Ya existen imágenes públicas oficiales (por ejemplo, de JDK de Java).
- Construcción y almacenamiento de una imagen:
  - Se construye por capas.
  - La imagen debe ser lo más liviana posible (evitar incluir cosas que no se usarán).
  - Al modificar una capa, se reconstruyen todas las capas siguientes a partir de ese punto (importancia del orden y el cacheo de capas).
  - Docker Hub aloja imágenes públicas.
  - `WORKDIR` define el directorio de trabajo dentro del contenedor.
  - `HEALTHCHECK`: mecanismo de chequeo de salud del contenedor.
  - En Kubernetes, se manejan réplicas del contenedor corriendo, y el healthcheck avisa si algo falla.
  - Ejemplo: `FROM python` hereda el sistema operativo base (Debian) de la imagen; Gunicorn se usa para levantar la aplicación Python en un puerto.
  - Orden típico de un build: primero se descargan las librerías, luego se copia la aplicación.

**Dockerfile con malas prácticas**
- No definir un usuario específico implica que el contenedor corre como usuario root (riesgo de seguridad).

**Dockerfile con buenas prácticas**
- Dividir el Dockerfile en etapas (multi-stage build), de forma que se pueda validar primero si compila.
- Uso de un archivo `.dockerignore` para excluir archivos innecesarios de la imagen.

### Del build al registro
- Docker Hub: repositorio público de imágenes.
- Cada proveedor de nube tiene su propio registro (ej. ECR en AWS); se descarga desde el registro correspondiente a la nube usada.
- El identificador del commit permite trazar de dónde proviene la imagen.
- Se generan tokens temporales para el acceso al registro.

---

### B. Infraestructura como Código (IaC)

**Qué problema resuelve IaC**
- Permite aplicar la misma "receta" de infraestructura en diferentes ambientes de forma consistente.
- Si la infraestructura se crea manualmente (por wizard/consola), no queda claro quién la valida; se necesita un módulo de auditoría.

**Enfoques de IaC**
- Enfoque imperativo (paso a paso).
- Enfoque declarativo (se declara el estado deseado) — ejemplo: Terraform.
- Se necesita mantener el estado de la infraestructura y su historial en todas las máquinas/entornos involucrados, para saber qué recursos ya fueron creados.

**El archivo de estado (state) en Terraform**
- Es un archivo que registra qué recursos ya existen (por ejemplo, qué máquinas virtuales con qué características).
- Mal manejo: si cada persona genera su propio archivo state de forma local, se duplican o pierden recursos al no compartir la misma fuente de verdad.
- Buen manejo:
  - Debe existir un único archivo state, actualizado de forma centralizada con cada cambio nuevo.
  - Se usa un "lock" del state (bandera que se activa) para asegurar que los cambios se apliquen en secuencia y evitar condiciones de carrera entre personas.
  - El state centralizado normalmente se guarda en un bucket S3 (o equivalente), y el lock puede apoyarse en una tabla tipo DynamoDB.
  - El archivo state puede contener datos sensibles, por lo que debe protegerse.

**Familias de herramientas y criterios de selección**
- Kubernetes: se declara en un archivo YAML la orquestación (por ejemplo, cuántas réplicas de un contenedor se necesitan).
- Terraform puede usar distintos backends según la nube (por ejemplo S3 para AWS; otras librerías para Azure).
- **Drift**: diferencia entre lo que existe realmente en la infraestructura y lo que está declarado en el código.
- Un bucket S3 se describe como análogo a una carpeta compartida (tipo Google Drive) donde se pueden desplegar archivos; se define en un archivo `.tf`.

**Cuando el ambiente real deja de parecerse a lo declarado**
- Terraform es una herramienta de Infraestructura como Código que permite definir y automatizar la creación de recursos en la nube mediante archivos de texto legibles; en DevSecOps se usa para integrar la seguridad desde el inicio, logrando una infraestructura configurada de forma segura, repetible y sin errores humanos.
- AWS expone una API; el proyecto "pasa por" Terraform, pero en la práctica las librerías se instancian localmente para luego comunicarse con AWS.
- `main.tf`: archivo principal que declara qué recursos se van a usar; existen archivos `.tf` separados para la conexión a la nube y para la configuración de Terraform en sí.

**Secuencia de comandos de Terraform**
1. `init`: descarga los módulos/dependencias necesarias en la máquina local.
2. `fmt` (format): corrige automáticamente el formato/indentación del código.
3. `validate`: revisa que los atributos y su estructura sean correctos según lo esperado por cada recurso.
4. `plan`: se conecta al state, aplica el lock, y compara el estado actual contra el cambio propuesto — siempre debe revisarse antes de aplicar.
5. `apply`: crea efectivamente los recursos (por ejemplo, un bucket en S3).

- El nombre de un bucket S3 debe ser único a nivel mundial; si se cambia el nombre, por consistencia de la API el recurso se destruye y se vuelve a crear (no se renombra in-place).

**Drift y su gestión**
- Drift: diferencia entre el ambiente real y lo declarado en código; buena práctica: mantener un pipeline que lo detecte periódicamente (ejecuciones de solo `plan`).
- La infraestructura recorre el mismo pipeline que el resto del código; un problema práctico es que el pipeline puede quedar bloqueado esperando confirmación manual ("yes"), por lo que se recomienda auto-aprobación configurada donde aplique.
- Herramienta de seguridad para IaC mencionada nuevamente: **Checkov**.

### Objetivo general de estas dos sesiones
Cerrar el ciclo completo del pipeline: de la validación de calidad y seguridad del código (Sesión 4) a la construcción de artefactos desplegables (imágenes Docker) y la gestión de infraestructura como código con Terraform (Sesión 5) — base directa del Laboratorio 3 ("Las fijas").
