# INF384 · Semana 11 — Automatización de seguridad II: policy as code, análisis dinámico (DAST) y mapa de controles del pipeline

Curso: INF384 Temas Avanzados en Ingeniería de Software 1 (PUCP, 2026-2) · Prof. José Felipe del Castillo Salazar
**Sesión 10 del curso: "Automatización de seguridad: infraestructura, ejecución y política"** · Capítulo 6 (2 de 2) · 3 horas · RA3 · RA4. Cierra el bloque de seguridad que abrió la sesión 9 (carpeta S10). La carpeta del curso la rotula S11 (semana 11).
Clase del 30/10/2026 (grabación iniciada a las 11:30:57 p. m. según la herramienta de transcripción), duración 100 min.

Fuentes: (1) transcripción automática `Temas S11.txt` (Tactiq), con mucho ruido: empieza a mitad de frase, hay un tramo sin texto entre ~48:02 y ~51:18 (justo donde arranca la demo en vivo de Terraform y Checkov) y varios fragmentos ininteligibles; (2) las diapositivas `INF384_S10_Automatizacion_seguridad_II.pdf` (portada + 21 láminas numeradas).

Cómo leer este documento: lo marcado "(diap. N)" viene de las láminas (numeración impresa en cada lámina); lo demás es lo dicho en clase. `[inferido]` = mi lectura de algo ambiguo o implícito, no una afirmación textual. Donde la clase y las láminas difieren, lo señalo.

Ruido de la transcripción que normalicé: "Checo/Chaco/Chekhov" = **Checkov**; "Polizas/Policias/Policies Code/Policía Stop" = **policy as code**; "DAS/DASC/NAS/TAS/ZAST" = **DAST** (y SAST); "Cuba" = **QA**; "Terrafón/Terfons/teléfono" = **Terraform**; "Sengre/centre" = Semgrep `[inferido]`; "Gitlix" = Gitleaks `[inferido]`; "zona club" = SonarCloud `[inferido]`; "backlit S3" = bucket S3; "Prisma Cloud" = producto comercial de Palo Alto.

## 0. Avisos de la clase

- **Laboratorio 5: jueves 12/11, 16:00** (diap. 19 y 21). La guía se publica en el aula virtual una semana antes de la sesión de laboratorio, es decir el **5/11** (diap. 19 y 21). Punto de partida: los dos pipelines del Laboratorio 3. Qué se agrega: **composición como compuerta** con umbral escrito, inventario (SBOM) publicado con el digest y excepción con cuatro datos; **análisis de la imagen antes del push**; acciones y base **fijadas por digest**; **política propia sobre el plan de infraestructura**, con `deny` y `warn`, y la excepción pegada al recurso.
- **Formato del laboratorio según el profesor (30/10)**: "va a ser un laboratorio un poquito diferente", **grupal, en salitas de Zoom, virtual, sobre todo aprendizaje**. En la clase siguiente (6/11) dio más detalle del formato y es distinto de lo que dicen las láminas; ver el documento `S12-contexto-clase.md` (sección del Lab 5).
- **Apertura de la sesión (el audio empieza a mitad de frase)**: el profesor mostró el informe anual **DORA** ("ya se encuesta a todas las organizaciones del mundo", con muchas recomendaciones por hallazgo; este año muy centrado en el uso de IA, que "ha sido muy acelerado y cambiante" en los últimos seis meses) y un enlace a una carpeta de recursos que armó con sus equipos de DevOps para estudiar certificaciones (presentaciones, PDF, libros, preguntas de práctica; admite que parte del material está desfasado, de 2019). Mencionó una colega arquitecta con todas las certificaciones de AWS ("Golden Jacket"). No dio una lista de enlaces que pueda reconstruir.
- **Comentarios sobre el examen**: (a) en la pregunta de **NIST** "evalúo más el razonamiento que hay detrás que la respuesta específica", y NIST evalúa **todo** el proceso de tecnología (altas de usuarios, desarrollo, operación…), no solo el desarrollo; las respuestas que combinaban de todo se consideraron válidas si tenían relación con el proceso de desarrollo y se comentaron las que no; (b) en la de **detección de secretos** "casi todos o todos respondieron bien"; (c) confusión frecuente: el control de **integridad de la aplicación** (digest o hash del commit/imagen) **no tiene que ver con vulnerabilidades**: protege que el componente desplegado sea fiel a lo que se generó, como el hash que se verifica al descargar un archivo.
- **Lo que viene**: "con esta clase estamos llegando al último de los controles". Después viene el bloque de prácticas ("la parte que más me gusta de DevOps, la parte más humana") y al final la IA. La sesión 11 abre el Capítulo 7 con la pregunta de qué pasa después del incidente.
- **Para la sesión 11 (diap. 21)**: leer la sección de análisis dinámico de la **guía DevSecOps de OWASP** (bibliografía complementaria) y **traer un incidente que conozcan, propio o público, con su causa y cómo se comunicó**.
- **Objetivos (diap. 2)**: (1) policy as code sobre la infraestructura declarada; (2) análisis dinámico de la aplicación desplegada; (3) la política completa: mapa de controles y gestión de hallazgos; (A) actividad: políticas y análisis dinámico para su caso; (L) Laboratorio 5. RA3 (seguridad integrada con análisis de IaC y dinámico) y RA4 (impacto en la confiabilidad de lo desplegado).

## 1. Dónde queda la sesión 9 y qué sigue abierto (diap. 3)

La S9 dejó resuelto: el analizador estático sigue el dato de la fuente al sumidero y el de composición lee el lock file; la prioridad combina alcanzabilidad, exposición, explotación y parche; la imagen se analiza antes de publicar, en el registro y al desplegar (firma e inventario viajan con el digest); lo que entra al runner con credenciales se fija por digest.

Preguntas que la lámina deja abiertas: ¿cómo se revisa un bucket público o un rol con permisos totales **antes de que existan**?, ¿qué defecto solo aparece **con la aplicación corriendo** y en qué etapa se busca?, ¿cómo se ve la política completa de un pipeline con todos los controles y lo que decide cada uno?, ¿qué hace el equipo con 200 hallazgos de cinco herramientas distintas? Frase de cierre: "hoy la seguridad se extiende a la derecha del pipeline: a la infraestructura que se va a crear y a la aplicación que ya responde".

## 2. Parte A — Policy as code sobre la infraestructura

### 2.1 El problema: la configuración insegura es una vulnerabilidad sin código (diap. 5)

| Recurso declarado | Configuración frecuente | Consecuencia |
|---|---|---|
| Almacenamiento de objetos | Acceso público de lectura "para servir imágenes" | Cualquier archivo del bucket es descargable, incluidos respaldos |
| Grupo de seguridad | Puerto de administración abierto a `0.0.0.0/0` "para probar" | Acceso directo desde internet al servidor |
| Rol de ejecución | Política con acción `*` sobre recurso `*` | Una falla de la aplicación se vuelve control total de la cuenta |
| Base de datos | Sin cifrado en reposo ni respaldos automáticos | Exposición de datos y pérdida sin recuperación |
| Función serverless | Variables de entorno con credenciales en texto plano | Credenciales visibles para cualquiera con acceso de lectura |

Dos ideas de la lámina: (1) **ningún analizador de la S9 ve esto**: el código de la aplicación es correcto, la infraestructura que lo ejecuta es la que expone los datos; (2) la infraestructura como código **lo hace revisable antes de que exista**: un archivo declarado se analiza igual que el código fuente, en el commit, con una regla y una decisión.

En clase: el modelo "clásico" era crear la infraestructura a mano, pasarla a QA o producción y recién ahí llamaba el equipo de seguridad (dos o tres semanas de ethical hacking) a decir "tu repositorio está público, no aplicaste el último TLS" y se retrabajaba; con policy as code se corre una herramienta (la que usó fue **Checkov**, y mencionó que empresas que usan Terraform sobre AWS la emplean para levantar vulnerabilidades antes de desplegar) y se obtiene la lista **sin haber desplegado nada**. El profesor insistió en que la infraestructura es "incluso más crítica que la aplicación", porque la aplicación vive dentro de ella.

### 2.2 El flujo de Terraform y en qué paso corre la política (explicación en clase)

El profesor repasó los comandos preguntando al grupo:

| Comando | Qué hace según la clase | Relevancia para la política |
|---|---|---|
| `terraform init` | Descarga los proveedores y módulos ("las librerías") | Antes de él no hay nada que revisar: faltan las definiciones de recursos |
| `terraform fmt` | Solo formato (indentación); "más decorativo" | Ninguna |
| `terraform validate` | **Solo sintaxis**: detecta algo mal declarado, por ejemplo una variable local que no existe; es opcional y muy rápido | No busca vulnerabilidades |
| `terraform plan` | Compara el código con el **state** (el registro de lo ya ejecutado) y dice qué se agrega, cambia o destruye; con proyectos grandes puede tardar minutos | **Genera el archivo (el plan en JSON) que leen las herramientas de política**: "¿esto que quieres hacer está bien o mal?" |
| `terraform apply` | Ejecuta los cambios del plan | Evaluar aquí es tarde: "ya las inyectaste" |
| `terraform destroy` | Elimina todo lo declarado | Se usa poco; en el ambiente efímero se usa al final |

Respuesta a "¿en qué momento corre el análisis de política?": **después del plan**. No antes del `init` (no hay nada que revisar), y no después del `apply` (si había una vulnerabilidad ya se creó). Analogía del profesor: hacer el análisis de calidad con SonarQube después de pasar el control de calidad no tiene sentido. Aclaró que **también se puede correr directo sobre el código**, pero la buena práctica es hacerlo sobre el plan.

**Tres momentos para evaluar la infraestructura (diap. 6)**:

| | 1 · Sobre el código | 2 · Sobre el plan | 3 · Sobre lo que existe |
|---|---|---|---|
| Qué lee | Archivos declarados, con variables sin resolver | Plan resuelto: valores finales y acciones exactas | Configuración viva consultada al proveedor |
| Qué detecta | Patrones en la declaración: cifrado ausente, acceso público | Lo que las variables ocultaban y las **destrucciones** | **Drift**: lo que cambió por consola o fuera del pipeline |
| Qué se le escapa | Un valor que llega por variable de entorno o módulo | Nada de lo declarado; sí lo creado fuera del código | Lo que nunca se declaró y lo que va a cambiar mañana |
| Cuándo corre | En cada commit, segundos, sin credenciales | En cada PR, con el plan, sin aplicar nada | Programado: diario o continuo |
| Quién decide | El gate del pipeline | El gate, con el plan comentado | Alerta a operación; la corrección va por pipeline |

La lámina agrega: "el Laboratorio 3 evaluó el momento 1. El Laboratorio 5 agrega el momento 2: la misma regla sobre el plan ve los valores que la declaración no mostraba". El profesor dijo que lo normal es que corra **en cada pull request** y que funcione como un gate más, igual que el de calidad.

### 2.3 Anatomía de una política escrita como código (diap. 7)

Archivo `politicas/almacenamiento_publico.rego`, escrito sobre el plan (Rego). Seis puntos de la lámina:

1. **La política vive en el repositorio**: se versiona, se revisa en PR y tiene historial igual que el código.
2. **La entrada es el plan en JSON** (`input.resource_changes`): la regla ve valores resueltos y acciones `create`, `update`, `delete`.
3. **`deny` detiene; `warn` informa**: la misma distinción de la S4, escrita en la regla.
4. **La condición es explícita y legible**: en el ejemplo, un recurso `aws_s3_bucket_public_access_block` con `block_public_acls == false`; quien revisa el PR entiende qué se exige sin abrir la herramienta.
5. **El mensaje dice qué hacer**: nombra el recurso y la vía de excepción ("excepción junto al recurso").
6. **Lo informativo también queda escrito**: el ejemplo de `warn` es una función Lambda sin trazas activas; observabilidad y etiquetas entran como `warn` hasta que el equipo acuerde subirlas.

En clase el profesor no escribió Rego; mostró la lógica con Checkov (base de reglas ya hecha). `[inferido]` Rego es el lenguaje de la lámina (política propia, Lab 5) y Checkov el de la demo (reglas listas); son dos caminos al mismo control.

### 2.4 Demo en vivo con Checkov (el tramo 48:02–51:18 no tiene texto)

Lo que se puede reconstruir de la parte con audio:

- Un Terraform que declara un **bucket S3** con sus controles (acceso público bloqueado/ACL, versionamiento, cifrado con llave propia o la del servicio) y un archivo de texto de ejemplo (`Hola.txt`). El `plan` resumió que agregaba **5 recursos**.
- `validate` detectó una variable local inexistente ("estás usando una variable local que no existe") y `fmt` reformateó los tabs.
- **Checkov** puede escanear el código directamente o un plan específico. Resultado: **6 checks, 5 pasaron y 1 falló** (el profesor se confundió primero y se corrigió). Cada check tiene un **ID de su base de conocimiento** y un enlace a documentación pública que explica el hallazgo y cómo corregirlo. El que falló era el de restricción de acceso público del bucket.
- Con el bucket abierto "ya puedo ver el archivo" por URL; al cerrar la configuración sale **acceso denegado**. Mencionó que hay políticas más finas (compartir con internet, con otras cuentas de AWS o con un usuario concreto).
- **Excepción**: en el mismo código, junto al recurso, se agrega una instrucción `checkov:skip=<ID>` más un texto libre con el motivo. Ejemplo dado: un bucket público a propósito, de solo lectura, para memes que "queremos que todo el mundo use". La herramienta lo trata como *skip*, pasa, **pero queda registrado en el reporte** con el motivo. En la empresa el texto cumple dos fines: que todos sepan por qué, y **fijar una fecha desde la cual la excepción ya no debería existir**, si corresponde. La lámina 8 lo formaliza: "toda excepción vive junto al recurso, con vencimiento; al vencer, la política vuelve a detener sola".
- Tras saltar un check, el resumen volvió a mostrar fallidos y pasados (la transcripción de ese tramo es confusa). Lo importante: el reporte tiene **tres categorías**: pasó, falló y exceptuado (skip).
- Cómo clasifica: los hallazgos se catalogan como crítico/alto/medio/bajo y **se configura qué detiene el pipeline**; esa configuración queda a cargo del analista de seguridad (el equipo de desarrollo se "choca" con ella hasta alinearse).

**Escala y consolidación**: cada persona con su reporte de Checkov en su máquina no es manejable en una empresa. Opciones: una herramienta de gestión como **Prisma Cloud** (cara) que centraliza lo que Checkov reporta, o **código (Python) dentro del pipeline** que lea los resultados, los resuma en las tres categorías y los lleve a un lugar común sin pagar la herramienta ("los pros y contras").

### 2.5 Qué políticas escribir primero (diap. 8)

| Familia | Ejemplos | Efecto |
|---|---|---|
| Exposición | Acceso público, puertos abiertos a internet, endpoints sin autenticación | `deny` |
| Identidad | Comodines en acciones o recursos, roles compartidos entre funciones, claves de larga duración | `deny` |
| Datos | Cifrado en reposo y tránsito, respaldos, retención, borrado protegido | `deny` |
| Trazabilidad | Registros de acceso, trazas, etiquetas de dueño y ambiente | `warn → deny` |

Tres puntos: (1) primero lo que ya fue incidente y lo que el regulador pide por escrito; **diez a veinte reglas que todos entienden valen más que trescientas que nadie lee**; (2) **la regla nueva entra en `warn` y sube a `deny` por acuerdo**, como el umbral de la S3: se mide cuántos recursos la incumplen hoy antes de detener a alguien; (3) toda excepción con vencimiento; **el número de excepciones vigentes indica si la regla está bien calibrada**.

En clase: el orden que dio es exposición ("¿está público o no?"), identidad ("¿quién tiene acceso?"), datos ("¿están cifrados?") y trazabilidad, que "es lo que menos riesgo tiene, normalmente no se detiene, se informa como *warning*". Sobre sistemas que ya operan sin esto: existe una **línea base** con hallazgos antiguos, igual que cuando el SAST salió con "40 vulnerabilidades antiguas y dos nuevas": se gestiona por criticidad (detener lo nuevo, inventariar lo heredado, idea de la S4).

### 2.6 Drift y evaluación continua (diap. 9)

- **Cómo aparece**: un cambio por consola durante un incidente a las 2 a. m. (se abre un puerto, se amplía un rol, se desactiva una alarma). El incidente se resuelve, el cambio se queda y el archivo declarado sigue diciendo lo de antes.
- **Cómo se detecta**: un `plan` programado sin cambios de código; **si propone acciones, hay drift**. Y la misma política de la lámina 7 evaluada sobre la configuración viva, de forma continua, **con alerta en lugar de gate**.
- (1) **La corrección entra por el pipeline, nunca por consola**: si el cambio de emergencia era necesario se declara en el código y se aplica; si no, el `apply` lo revierte. (2) **El cambio de emergencia tiene una ruta escrita**: quién puede hacerlo, cómo se registra y en cuánto tiempo se vuelve a declarar; "es la aprobación de la sesión 6 para el camino de vuelta".
- Frase de la lámina: "el pipeline garantiza lo que pasa por él. Lo que entra por otra vía solo se detecta comparando lo declarado con lo que existe".
- En clase el profesor lo planteó como el límite del policy as code: alguien "por alguna razón abre el puerto en producción" aunque todo estaba validado; eso escapa y requiere algo que **escanee la infraestructura ya desplegada**. Mostró la herramienta Aikido con ese módulo de nube (ver 3.6).

### 2.7 Contexto organizacional: quién define las reglas (digresión larga del profesor)

- Estructura típica de un área de TI en una empresa grande: arquitectura, infraestructura y operaciones, desarrollo (con tech leads y QA), finanzas de TI y gobierno (donde suele estar la PMO). DevOps puede estar en infraestructura u otra caja según la empresa. TI es de los gastos más grandes de la empresa (entre el primero y el segundo, dijo), por eso tiene su propia área de finanzas.
- **La seguridad se partió en dos**: **seguridad de la información** (lineamientos de alto nivel, normas, auditoría, alineamientos; fue hacia riesgos y gobierno) y **seguridad informática** (políticas más técnicas del día a día; normalmente dentro de infraestructura y operaciones). En seguridad informática están los **analistas y arquitectos de seguridad** que dicen "tu bucket no puede ser público". Las dos áreas suelen estar tensionadas y "triangular entre ambas es difícil".
- Lo que define el analista de seguridad (qué lineamientos sí y cuáles no) es lo que el equipo de ingeniería implementa y con lo que "se va chocando" hasta alinearse. Ventaja de policy as code: **se revisa la infraestructura antes de desplegar siquiera en desarrollo**, en vez de terminar el proyecto, tocar la puerta de seguridad y retrabajar.
- Anécdota (cuenta una persona de su equipo de DevOps que abrió el firewall a cualquier puerto "para una prueba"): la alerta saltó al equipo de seguridad, se detectó al mediodía, hubo reunión de urgencia todo el día. El riesgo era que alguien cargara por URL un archivo de configuración (`.env` / `config.php` en una aplicación PHP) con usuario y contraseña de base de datos y de sistemas externos (correo, SMS); hubo que **rotar todas las credenciales** en una noche larga (con el cuidado de que otra aplicación podía usar la misma). Es la misma historia de la S10 (allí con el puerto SSH); `[inferido]` es el mismo caso contado con otro detalle.
- Otra anécdota: su jefa, para vender un servicio a un fabricante de autos que operaba en AWS, ejecutó scripts que encontraron **muchos buckets públicos que la empresa no sabía que estaban abiertos**.
- Historias de carrera que contó (promoción cuando se va el jefe, postulación frente a candidatos con más experiencia, fusión de empresas con reorganización de gerencias y roles de seguridad): su punto es que **los eventos no elegidos abren posibilidades y cada persona reacciona distinto; "no hay una forma correcta de lo que sientan"**. No las reproduzco; son contexto de vida del profesor, no contenido evaluable.

## 3. Parte B — Análisis dinámico (DAST)

### 3.1 Lo que el dinámico ve y el estático no (diap. 11)

| Aspecto | Análisis estático | Análisis dinámico |
|---|---|---|
| Qué necesita | El código fuente | La aplicación desplegada y una dirección |
| Qué observa | Caminos posibles en el código | Respuestas reales a peticiones reales |
| Qué detecta mejor | Flujos fuente-sumidero, secretos, patrones | Cabeceras y cookies, TLS, configuración del servidor, autenticación, componentes integrados |
| Qué se le escapa | Configuración en ejecución, servidor, red | Código que ninguna petición alcanza; la causa exacta en el código |
| Falsos positivos | Frecuentes sin calibrar | Pocos: el hallazgo trae la petición y la respuesta que lo prueban |
| Cuándo corre | En cada commit, minutos | Después de desplegar en certificación; minutos en modo pasivo, horas en activo |

Frase: "un hallazgo dinámico trae la petición que lo provocó y la respuesta que lo confirma. Es la evidencia más directa que produce cualquier control del capítulo".

En clase: el estático revisa el código (secretos, patrones); el dinámico "ve la vulnerabilidad de lo que ves en la punta, pero no de dónde viene". Tiene menos falsos positivos porque es "más blanco y negro": la cabecera con la vulnerabilidad está o no está. Ejemplo clásico: la respuesta que expone el nombre y la versión del servidor (un Apache/Tomcat con su versión) permite al atacante buscar vulnerabilidades de esa versión; la idea es controlar qué cabeceras salen al cliente final y ocultarlas (con capas intermedias), y eso lo valida el DAST.

### 3.2 Dos modos: observar y atacar (diap. 12)

| | Pasivo (línea base) | Activo |
|---|---|---|
| Qué hace | Recorre la aplicación con peticiones normales y observa las respuestas | Envía peticiones diseñadas para provocar fallas: inyección, recorrido de rutas, manipulación de parámetros |
| Detecta | Cabeceras ausentes, cookies inseguras, información filtrada en errores, versiones expuestas | Lo explotable desde fuera, con la prueba adjunta |
| Costo | Minutos; sin efecto sobre datos; en cada despliegue de certificación | Horas; **crea y modifica datos**; programado contra certificación, **nunca contra producción sin acuerdo** |

Tres notas de la lámina: (1) **sin autenticación el análisis ve solo la página de ingreso**: se le entrega una sesión de prueba o un token de un usuario de prueba, con datos de la S8 (propios y limpiados al final); (2) **una API sin interfaz se recorre con su especificación** (el documento de la API es el mapa: cada operación se prueba con sus parámetros y tipos); (3) **el umbral se declara como en cualquier gate**: un hallazgo alto en línea base detiene la promoción; lo demás se registra con responsable.

En clase: pasivo es "solo SELECT, no llego a insertar ni a malograr nada"; activo es "trato de hacer SQL injection… de romper tu aplicación". Con la especificación de la API (el contrato con métodos, entradas y salidas) se puede evaluar una API consumible desde internet; a veces hay que darle un usuario con privilegios y otro sin privilegios para recorrer caminos distintos; los resultados se clasifican media/alta/crítica y se decide si deja pasar.

### 3.3 Ambientes efímeros (explicación en clase)

- Riesgo del activo: si algo pasa, no debe afectar a clientes ni a un ambiente en uso (**no se corre contra el ambiente de QA donde el equipo de calidad está probando ni contra producción**; el activo "puede bajar la aplicación"). Dos salidas: un ambiente paralelo, o bajarle el alcance con reglas "para que no baje nada", lo que reduce el alcance del ataque. Contó que quiso hacerlo con un cliente y la herramienta mostró una advertencia ("¿está seguro?") y desistieron.
- Solución "súper avanzada": **ambiente efímero** creado por el propio pipeline. Flujo: el desarrollador trabaja en su rama feature y abre el PR → corre CI (todos los controles anteriores) → **IaC levanta la plataforma** → **CD despliega la aplicación** → *smoke* → **DAST** → **destroy** de todo. "Es un ambiente que nace como parte del propio pipeline y después se destruye." Requiere "una madurez de equipo y de proceso importante"; el profesor dice que él no lo ha usado todavía, y que un estudiante comentó que ya lo usan. Sirve también para pruebas de integración o end to end, según para qué se necesite.
- Ejemplos de hallazgos que puede dar: API sin autenticación, datos sin cifrar.

### 3.4 Dónde corre el dinámico en el pipeline y cómo se escribe (diap. 13)

Secuencia de la lámina: **Integración continua** (secretos, estático, composición, imagen, política sobre el código y el plan) → **Publicar** (imagen firmada con inventario, por digest) → **Certificación** (despliegue, humo, end to end, **DAST pasivo en cada despliegue**) → **Programado** (DAST activo nocturno sobre certificación; evaluación de postura sobre lo vivo) → **Producción** (humo, observación, verificación sintética; **sin análisis activo**).

Job de GitHub Actions de la lámina (`dast-base`): `needs: desplegar-cert` (1, recibe la dirección del despliegue), la acción del analizador **fijada por SHA**, `objetivo: ${{ needs.desplegar-cert.outputs.url }}`, `especificacion: openapi.yaml` (2, la especificación guía el recorrido), `umbral: alto` (3, el umbral decide la promoción) y `actions/upload-artifact@<sha>` con `if: always()` (4, **el reporte se conserva aunque falle**).

En clase: se corre en certificación o, si tarda horas, se programa (por ejemplo una vez por semana de noche). Necesita: la URL del despliegue, a veces la especificación funcional, a veces usuarios con y sin privilegios.

### 3.5 DAST frente al ethical hacking

- El ethical hacking "clásico" era pagar a alguien de seguridad al terminar el proyecto para que entregara una lista de vulnerabilidades. **El DAST cubre mucho de eso** (cabeceras, configuración del servidor, etc.).
- Lo que el DAST **no** valida y el ethical hacking sí: fallas de **lógica de negocio / definición de seguridad**. Ejemplo (el del teclado numérico de la banca por internet, de la S10): la definición es que viaje la posición/hash del teclado y el servidor traduzca a la clave real; si por error viaja el número real, el DAST ve "datos que viajan cifrados en el canal" y no lo marca como vulnerabilidad, porque no conoce la definición de seguridad.
- **Ejemplo de IDOR** (identificador manipulable): una URL con el código de alumno como parámetro; al cambiarlo por el de otra persona se ven sus notas. Si hay validación de sesión (la sesión trae metadatos y el servidor compara) puede estar bien, pero la mejor práctica es **no exponer identificadores secuenciales o reales** (usar un valor opaco, por ejemplo hexadecimal, que el servidor traduce), tanto para documentos como para IDs de base de datos: "ningún dato de eso se pueda tantear".

### 3.6 Herramienta mostrada: Aikido

Herramienta con mucha capacidad gratuita que combina varias soluciones: análisis del código fuente (el estático visto en la S9), análisis de **nube** (revisa lo ya desplegado: si hay un bucket público, lo dice) y un módulo de ataque tipo DAST ("Attack"). Es lo que el profesor usó para ilustrar el DAST; subraya que el DAST necesita una URL de la solución desplegada contra la cual atacar.

## 4. Parte C — La política completa

### 4.1 Mapa de controles del pipeline (diap. 15)

| Control | Etapa | Efecto | Umbral de ejemplo | Evidencia que produce |
|---|---|---|---|---|
| Detección de secretos | CI · cada commit | Detiene | Cualquier hallazgo | Hallazgo con archivo y commit |
| Análisis estático (SAST) | CI · cada commit | Detiene | Alto en reglas de alta confianza | Camino fuente-sumidero |
| Composición (SCA) | CI · cada commit | Detiene | Alto con parche disponible | Camino hasta la directa; inventario |
| Cadena de suministro | CI · revisión de PR | Detiene | Acción o base sin digest | Diff del workflow y del Dockerfile |
| Análisis de imagen | CI · antes del push | Detiene | Alto en SO o runtime con parche | Reporte por capa; firma |
| Política sobre el código IaC | CI · cada commit | Detiene | Exposición, identidad, datos | Regla y recurso |
| Política sobre el plan | CI · cada PR | Detiene | `deny` de la política; destrucciones | Plan comentado en el PR |
| Dinámico pasivo | Certificación · cada despliegue | Detiene | Alto | Petición y respuesta |
| Dinámico activo | Programado · nocturno | **Informa** | Alto → ticket con plazo | Petición y respuesta |
| Postura sobre lo vivo | Programado · continuo | **Informa** | Drift o `deny` sobre lo existente | Recurso y diferencia |

"Ocho controles detienen y dos informan. Cada fila tiene su umbral escrito y su evidencia; con eso la política completa cabe en una página del repositorio."

En clase el profesor recorrió la tabla: detección de secretos, SAST ("cada commit es un decir": puede ser en el pull request), composición, cadena de suministro (también basada en las versiones fijadas), imagen del contenedor, política (policy as code), DAST pasivo/activo y, por último, el análisis de "lo que está en producción ahora": correr un `plan` cada cierto tiempo para detectar **drift** o una política que antes estaba bien y ahora no. Todo esto es "postura sobre lo que está en vivo". Fue la síntesis de lo visto "clase a clase" en el frente de seguridad.

### 4.2 Un solo flujo de hallazgos (diap. 16)

Flujo: hallazgo → deduplicar → triage → decisión → seguimiento.

1. **Todos los reportes entran en un formato común, a un solo lugar.** El mismo componente vulnerable aparece en composición y en imagen: es un hallazgo con dos fuentes.
2. **El triage aplica las cuatro preguntas de la S9**: alcanzable, expuesto, explotado, con parche. Lo hace el equipo que escribió el código, con criterio escrito.
3. **La decisión es una de tres y queda registrada**: corregir con fecha, exceptuar con cuatro datos, o aceptar el riesgo con la firma de quien responde por él.
4. **Tres métricas**: tiempo de corrección por severidad, excepciones vigentes y porcentaje de hallazgos con decisión. "Un backlog creciente sin decisión es el síntoma de que el umbral está mal."

Idea de cierre: el flujo de hallazgos es un flujo de trabajo más del equipo, **en su misma herramienta y con su misma cadencia**; un tablero aparte del área de seguridad "se convierte en una lista que nadie abre".

En clase (reto de DevOps con tantas herramientas): lo normal es mezclar herramientas pagas y libres, y por eso **no hay un lugar centralizado**; hay que **codificar la automatización** para producir los reportes (el resumen del pipeline, una tabla al final) y, según la política, decidir qué deja pasar. Ejemplo que mostró (un pipeline real con herramientas libres): pruebas unitarias, SonarCloud, Semgrep (SAST), Checkov (infraestructura/imagen), Gitleaks (secretos) y Dependency-Track (librerías) `[inferido por los nombres ruidosos]`; resultado: **487 verificaciones en 35 segundos**, con errores que detienen y tres que solo alertan, y un resumen con la severidad, la regla incumplida, el archivo, la línea y la recomendación; el mismo resumen aparece en el **pull request**. Para un sistema antiguo que recién adoptó los controles "te inunda de cosas que tienes que hacer pero no puedes todavía", así que se deja pasar pero se sabe que hay un problema. El valor está en **traducir los hallazgos de manera efectiva al equipo de desarrollo**, no solo en conectar la herramienta para que falle.

### 4.3 Los controles en el marco NIST SP 800-218 (SSDF) (diap. 17)

| Práctica | Qué pide | Control del pipeline que la cubre |
|---|---|---|
| PO.3 | Implementar y mantener la cadena de herramientas de forma segura | Acciones y base fijadas por digest; runner con instalación sin scripts |
| PW.4 | Reutilizar software existente de forma segura | Composición sobre el lock file; inventario (SBOM) con cada digest |
| PW.6 | Configurar el proceso de compilación para mejorar la seguridad | Build en CI que solo produce imagen si las compuertas pasan; firma |
| PW.7 | Revisar y analizar el código legible por humanos | Análisis estático; detección de secretos; política sobre el código IaC |
| PW.8 | Probar el código ejecutable | Pruebas del Capítulo 5; análisis dinámico en certificación |
| PW.9 | Configurar el software con valores seguros por defecto | Política sobre el plan; imagen base reducida |
| RV.1 · RV.2 | Identificar vulnerabilidades de forma continua y remediarlas | Análisis en el registro; postura sobre lo vivo; flujo de hallazgos con métricas |
| RV.3 | Analizar la causa raíz y mejorar el proceso | Postmortem del Capítulo 7 |

"El marco no exige herramientas: exige que cada práctica tenga un control que la cubra y evidencia de que corrió. Esa es la columna derecha." Nota: la fila RV.3 conecta con la siguiente sesión (postmortem).

## 5. Actividad A2 — Políticas y análisis dinámico para su caso (diap. 18)

Diseñada para 25 minutos en grupo, sobre el caso de la sesión 1, con el pipeline de la sesión 4 y la cartera de pruebas de la sesión 7:

1. **Tres políticas**: recurso, condición y efecto (`deny` o `warn`) en una línea cada una; al menos una de exposición y una de identidad; en qué momento corre cada una (código, plan o lo vivo).
2. **El análisis dinámico**: dónde entra en su pipeline (etapa, modo, umbral); qué necesita (dirección, especificación, usuario de prueba); qué hace con el hallazgo alto (detiene qué).
3. **El dueño**: quién escribe las políticas y quién las aprueba; quién autoriza una excepción y por cuánto tiempo; qué cambia respecto de cómo el caso aprueba hoy.

Entregable verbal: cada grupo presenta en dos minutos su política más importante y la ubicación del análisis dinámico. Sin material impreso ni aula virtual. **En la transcripción no aparece la actividad desarrollada** (el profesor cerró con el resumen sin que se registren presentaciones de grupos); `[inferido]` no se hizo en la grabación o se hizo fuera de ella.

## 6. Qué llevarse

Puntos de cierre de la lámina (diap. 20):

1. **La configuración insegura es una vulnerabilidad sin código.** Se declara como código y se revisa como código: en el commit, en el plan y sobre lo vivo.
2. **La política se escribe, se versiona y se evalúa en el pipeline.** `deny` detiene, `warn` informa y la excepción vive junto al recurso con vencimiento.
3. **El análisis dinámico corre contra certificación con la dirección del despliegue.** Pasivo en cada despliegue, activo por reloj; su evidencia es la petición y la respuesta.
4. **Diez controles caben en una página** con etapa, efecto, umbral y evidencia. Esa página es la política de seguridad del pipeline.
5. **Un solo flujo de hallazgos, en la herramienta del equipo.** Triage con criterio escrito, decisión registrada y tres métricas.

Ideas que agregó el profesor:

6. **El plan es el insumo** de la política; ni antes del `init` ni después del `apply`.
7. **Las excepciones se escriben y versionan en el propio código** (o en el pipeline), con motivo y, si corresponde, fecha.
8. **El DAST corre contra un ambiente no productivo**; el modo activo exige un ambiente efímero o paralelo y una forma de volver atrás.
9. **El DAST no reemplaza al ethical hacking en lógica de negocio.**
10. **Hay que generar resúmenes legibles** (en el pipeline y el PR) para que el equipo los use, y automatizar la consolidación cuando no hay herramienta central.
11. **La integridad (digest/hash) no es un control de vulnerabilidades.**
12. En el examen se evalúa el razonamiento, sobre todo en preguntas de marcos como NIST.

## 7. Conexión con sesiones anteriores (relaciones mías, no dictadas)

- **S4 (integración continua II)**: `deny/warn` es el "detiene/informa"; la excepción con cuatro datos (identificador, razón, responsable, vencimiento); la regla nueva entra en `warn` como el umbral nuevo; "detiene lo nuevo, inventaría lo heredado".
- **S9 (seguridad I)**: las cuatro preguntas del triage (alcanzable, expuesto, explotado, parche) se reutilizan en el flujo de hallazgos; digest y firma se reutilizan en la fila de cadena de suministro y en PO.3.
- **S6 (despliegue continuo II)**: la ruta del cambio de emergencia es la aprobación "para el camino de vuelta".
- **S8 (calidad II)**: datos de prueba limpiados al final → usuarios de prueba del DAST; `needs` y orquestación → job `dast-base`.
- **Laboratorio 3** (momento 1, política sobre el código) → **Laboratorio 5** (momento 2, sobre el plan; composición como compuerta).
- **S11 (postmortem)**: RV.3 de la tabla NIST y la pregunta "cuando un control detiene en producción a las 2 a. m., ¿quién decide y con qué criterio escrito?" anticipan el capítulo 7.

## 8. Pendientes e incertidumbres

- La transcripción no permite confirmar: los nombres exactos de las herramientas del pipeline de ejemplo (normalicé por contexto), la demo completa de Checkov (el tramo 48:02–51:18 no tiene texto y la parte posterior es confusa) ni la cifra exacta de checks tras aplicar el `skip`.
- **Lab 5**: la lámina describe un laboratorio técnico (pipeline con composición como compuerta, imagen antes del push, política sobre el plan); lo dicho después en clase apunta a un formato distinto (ver `S12-contexto-clase.md`). Conviene confirmar con la guía del Lab 5 cuál es el entregable real.
- No consta si el profesor publicó el 5/11 la guía; las láminas dicen que sí.
- Las anécdotas (firewall abierto, buckets públicos del fabricante de autos, historias de carrera) son relatos del profesor, no verificados. La afirmación de que Checkov pertenece hoy a Palo Alto/Prisma Cloud la hizo el profesor; no la verifiqué.
- La actividad A2 no aparece desarrollada en la grabación; no hay respuestas "del profesor" que citar.
