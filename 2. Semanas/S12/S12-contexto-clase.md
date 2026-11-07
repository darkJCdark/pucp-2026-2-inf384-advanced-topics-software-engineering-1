# INF384 · Semana 12 — Prácticas DevSecOps I: aprender del incidente (postmortem sin culpa) y la posición de protagonista

Curso: INF384 Temas Avanzados en Ingeniería de Software 1 (PUCP, 2026-2) · Prof. José Felipe del Castillo Salazar
**Sesión 11 del curso: "Prácticas DevSecOps: aprender del incidente"** · Capítulo 7 (1 de 2) · 19:00–21:30 (según la lámina de portada) · RA4. Abre el bloque de prácticas (las sesiones 11 y 12 del curso; la 13 es DevSecOps en las organizaciones y la 14, IA). La carpeta del curso la rotula S12 (semana 12).
Clase del 6/11/2026 (grabación iniciada a las 11:09:19 p. m. según la herramienta de transcripción), duración 132 min.

Fuentes: (1) transcripción automática `Temas S12.txt` (Tactiq), con ruido fuerte: el tramo de preparación de los sketches (~01:34–01:46) quedó como texto basura ("Gracias por ver el video", "Suscríbete") y el video proyectado no tiene audio transcrito; (2) las diapositivas `INF384_S11_Practicas_DevSecOps_I.pdf` (31 páginas; la numeración impresa llega a 22 y **salta de la 03 a la 07**, así que las láminas 4 a 6 —sección A, "experiencia e incidente real / ruta profesional en primera persona"— no están en el PDF; la página 14 del PDF es solo la imagen fija de un video).

Cómo leer este documento: "(diap. N)" = número impreso en la lámina (varias no están numeradas: triángulo del drama, frases, triángulo de empoderamiento, reformulación, DISC y los cinco casos; las cito por título); el resto es lo dicho en clase. `[inferido]` = mi lectura de algo ambiguo o implícito. Donde la clase y las láminas difieren, lo señalo. Normalicé el ruido: "Cuba" = QA, "post-mortem/cosmortem" = postmortem, "disk/DIS" = DISC, "chelengers" = challenger.

## 0. Avisos de la clase

- **Laboratorio 5: jueves 12/11, 16:00** (diap. 22 y portada de ruta). Las láminas lo describen como un pipeline con validación de seguridad de componentes e infraestructura (ver `S11-contexto-clase.md`). **Formato dicho al cierre de esta clase** (sección 14): mezcla de una parte técnica y una parte parecida a la actividad de hoy (responder preguntas como grupo), en salas de Zoom, **escrito, un entregable por grupo, sin desarrollar ni construir un pipeline**, grupos de 4 (máximo 5). **No coincide con las láminas**; confirmar con la guía publicada.
- **Lectura previa a la sesión siguiente (diap. 22)**: *The DevOps Handbook*, **parte IV** (la segunda vía: retroalimentación y telemetría).
- **La siguiente sesión incluye un juego por equipos en el aula: llegar a las 19:00** (diap. 22).
- **Encuesta DISC**: la hicieron en clase por un enlace/sala que mostró la lámina (anónima); no la reproduzco.
- Al final un estudiante preguntó por los exámenes parciales de la semana anterior y el profesor respondió algo que la transcripción no deja claro ("próximo"); `[inferido]` aún no los había entregado o los entregaría después. Después se pasó lista.
- **Objetivos (diap. 2)**: (1) experiencia e incidente real: aprender con un postmortem; (2) víctima y protagonista: qué respuesta podemos elegir; (3) Actividad 1; (4) Actividad 2. RA4 (prácticas de mejora continua y gestión de incidentes y su impacto en la confiabilidad).

## 1. Apertura: de la observabilidad a mejorar (diap. 3 y diap. 7)

- Ya se recorrieron las etapas del pipeline; hoy toca **observar**: normalmente se piensa la observabilidad como logs y métricas (técnica), pero lo que sigue es "hemos encontrado tal problemática, ¿qué hacemos para mejorarla?", y esa mejora aplica a cualquier parte del proceso, no solo a producción. El postmortem trata lo que pasa **en producción** y cómo se convierte ese error en **un control nuevo del pipeline o del proceso**, "no solamente ya corregimos el problema, ya pasamos el incendio, sino cómo podemos anticiparnos".
- Los grandes proveedores de nube (AWS, Azure/Microsoft, Google, Oracle) hacen este análisis cuando ocurre algo; el profesor lo vivió desde dentro de Oracle (OCI).
- **Puente al capítulo 7 (diap. 7): un hecho, una lectura, una acción.**
  1. **El hecho**: algo ocurre y nadie lo eligió (una enfermedad, una caída de producción, un despido, un cambio de equipo).
  2. **La lectura**: la historia que la persona o el equipo se cuenta sobre ese hecho (quién falló, qué era posible, qué queda por hacer).
  3. **La acción**: lo que se hace después con lo que hay disponible (buscar culpable, pedir ayuda, aprender algo, cambiar un proceso).
  "El postmortem aplica la misma secuencia a un sistema: hechos verificables, una lectura compartida por el equipo y acciones con dueño y fecha."
- Ejemplo del profesor: "se cayó el servidor porque la aplicación se comió la RAM". El **hecho** es la caída; nadie la eligió. La **historia** es "le bajé la RAM porque el objetivo del mes era ahorrar el 50 % del presupuesto y no leí bien el consumo". La **acción**: en un lugar donde no se trabaja bien se busca al culpable "para tener algo que señalar en el correo"; en uno sano se pide ayuda, se aprende algo (aquí: la RAM no se observa minuto a minuto) y se cambia el proceso (alerta por WhatsApp/SMS/correo). Esas acciones van con **responsable y fecha**.
- Sobre "buscar el culpable": primero se **ubica el problema** (¿electrónico, base de datos, de dónde se origina?). La persona como culpable se ve después, y solo si hubo irresponsabilidad evidente **teniendo los controles puestos**; si no había ningún control y era parte de su rol decidir, el castigo no corresponde. Dijo que ve cada vez menos gente que piense en castigar y más que busque soluciones.
- Una pregunta de un estudiante sobre áreas de informática "donde no se recomienda trabajar" derivó en que depende de cada persona: al profesor no le gustaba la seguridad ni la operación, y terminó creando "una solución de las que se llaman DevOps" por su personalidad.

## 2. Tipología de Westrum (diap. 9)

| | Patológica · orientada al poder | Burocrática · orientada a reglas | Generativa · orientada al desempeño |
|---|---|---|---|
| Cooperación | Baja | Modesta | Alta |
| Quien trae malas noticias | Es castigado | Es ignorado | Es entrenado y escuchado |
| Responsabilidades | Se eluden | Se acotan al rol | Se comparten |
| Puentes entre equipos | Se desalientan | Se toleran | Se promueven |
| Ante una falla | Se busca un culpable | Se aplica la norma | Se investiga |
| Ante una idea nueva | Se aplasta | Genera problemas | Se implementa |

Lámina: "En los datos de *Accelerate*, la cultura generativa predice mejor desempeño de entrega y de la organización. Un postmortem sin culpa es su práctica más visible."

Lo que agregó el profesor:

- **Poder**: "el que tiene más poder gana", dos áreas no cooperan para no ceder poder. Su ejemplo: él advirtió una mala práctica del equipo de desarrollo y le dijeron que tenía prohibido tocar código; lo castigaron por la lucha de poder entre las gerencias de arquitectura y desarrollo.
- **Reglas**: "hasta acá es tu límite"; el que trae malas noticias es ignorado ("eso no te corresponde"); los componentes viejos que nadie quiere asumir ("no, es de él") son el caso típico; la cooperación existe solo en la medida en que las reglas la permiten.
- **Desempeño**: la meta de la organización es el fin; se coopera aunque el rol diga otra cosa ("tu rol dice A y el mío B, pero esto cae en medio, vamos juntos").
- **Innovación**: las grandes empresas crearon áreas de innovación porque cambiar un "músculo" tan grande es muy difícil. Dos modelos: crear una empresa aparte que el grupo financia (libertad total y mismo nivel del CEO, pero pocas sinergias con el resto) o una unidad digital dentro de la empresa (independencia parcial, pero pelea interna, críticas por deuda técnica y lineamientos de arquitectura). Es importante que quien dirige esa área tenga mentalidad generativa. El profesor remitió a una charla de un ejecutivo de innovación bancaria; no reproduzco el nombre porque la transcripción es poco confiable.
- Una startup funciona mejor generativa; una empresa del Estado está "entre reglas y poder"; la idea es ir "estirando el chicle" hacia la derecha. **No es por malas personas**, sino por cómo aprendieron a trabajar (por ejemplo, su mejor jefe lo limitaba por su época y su experiencia).

## 3. El error humano es donde empieza la investigación (diap. 10)

| Preguntas que **cierran** la investigación | Preguntas que **abren** la investigación |
|---|---|
| ¿Quién ejecutó el comando? | ¿Qué hizo que ese comando pareciera correcto en ese momento? |
| ¿Por qué no siguió el procedimiento? | ¿Qué decía el procedimiento y cuándo se actualizó por última vez? |
| ¿Quién aprobó ese cambio? | ¿Qué información tenía quien aprobó, y cuál le faltaba? |
| ¿Cómo no se dio cuenta antes? | ¿Qué señal existía y por qué no llegó a nadie? |

"Cada pregunta de la derecha termina en algo que el equipo puede cambiar: un procedimiento, un control del pipeline o una alerta."

En clase: "esto es clave, muchas veces tomamos decisiones con la información que teníamos y fallamos porque nos faltaba información adicional". La última pregunta remite a **observabilidad**: ¿qué alertas había y por qué no llegaron a nadie? Ejemplo del profesor: alertas configuradas hacia un correo genérico del tipo `admin@empresa` que ya no tenía dueño. El profesor aclaró que las preguntas de la izquierda también tienen sitio según lo que se quiera averiguar, pero no deben ser el punto de partida.

## 4. Cómo se ve un postmortem real: AWS y los "war rooms"

- **AWS Post-Event Summaries**: AWS publica los incidentes grandes. El profesor mostró el del año pasado en la región de Virginia (us-east-1), "la más usada del mundo", atribuido a **DynamoDB**; `[inferido]` coincide con el gran corte de AWS us-east-1 de octubre de 2025, pero no lo verifiqué en esta sesión. Tiene resumen ejecutivo, impacto por servicio, línea de tiempo y conclusiones. AWS **no publica todos**: solo los que escalaron más allá de lo controlado (si no sería "una sábana enorme"). Prefería el postmortem de otro incidente porque traía una tabla de acciones actualizada con fechas. Mostró también la página de estado de AWS.
- **War room ("sala de guerra")**: cuando hay un gran incidente se reúnen todos los equipos para solucionarlo; según la autoridad necesaria se escala. En Oracle participó en war rooms de **100–120 personas en Zoom**: un líder dirige, 2–3 personas por equipo, se va llamando equipo por equipo. En un incidente de centro de datos, mientras el equipo informaba "no prendía el hardware", él miraba el organigrama para ubicar a quien hablaba. Observó que en la empresa de ~12 niveles jerárquicos quienes lideraban técnicamente la solución eran gerentes con **mucha experiencia técnica**, "solución, solución, solución", sin espacio para excusas. Contrastó con empresas no tecnológicas (banca, seguros) donde el gerente de operaciones no suele tener ese perfil.
- **Contenido de un postmortem (como lo dijo)**: resumen, impacto, línea de tiempo, momento de detección, qué lo causó, qué funcionó, acciones para que no se repita y, al final, preguntas abiertas.

## 5. Anatomía de un postmortem (diap. 11)

| 1 Resumen | 2 Impacto | 3 Línea de tiempo | 4 Detección |
|---|---|---|---|
| Tres líneas: qué pasó, a quién afectó, cómo terminó | Usuarios, duración, transacciones perdidas, error budget consumido | Hechos con hora, desde el primer cambio hasta la resolución | Cómo se supo: una alerta, un cliente, una persona que miró por casualidad |

| 5 Causas contribuyentes | 6 Lo que funcionó | 7 Acciones | 8 Preguntas abiertas |
|---|---|---|---|
| Técnicas, de proceso, de detección y de organización | Lo que redujo el daño; "se protege igual que se corrige" | Cada una con dueño, fecha y tipo de cambio que produce | Lo que el equipo todavía no sabe y cómo lo va a averiguar |

"Se escribe en los cinco días hábiles siguientes, se publica para toda el área y se revisa en una reunión facilitada por alguien ajeno al incidente."

Orden de prioridades dicho en clase: **primero se restaura el servicio**; el postmortem va después y se alimenta con las notas que se van tomando durante el incidente.

## 6. El caso del profesor: borrado de una base de datos con Terraform

Lo contó "con licencia" porque no tiene el documento real; su mapeo a las partes del postmortem es el siguiente (las partes las puse yo, lo narrado es suyo; las horas son aproximadas, él mismo dijo "voy a decir cualquier cosa" en un detalle):

| Parte | Lo que contó |
|---|---|
| Resumen e impacto | Un viernes ~10:00, trabajando con Terraform, destruyó por error una base de datos de producción de una empresa de comunicación financiera (era consultor). Interrupción de **4 h** (10:00–14:00) en que las transacciones del POS no llegaron a esa base. El negocio no dejó de vender: el POS siguió con su backend; lo afectado fue el proceso asíncrono hacia analítica/reportes |
| Detección | Inmediata: él mismo lo notó "5 minutos después" y avisó a su jefa; podría haberse callado y que se descubriera luego por los dashboards, pero reportó |
| Respuesta | Escaló a su jefa y al gerente de arquitectura del cliente; ~10:30 se armó un **war room** con gente de AWS, su equipo y operaciones. Detuvieron el `destroy` |
| Mitigación y restauración | Recrearon la base desde el **respaldo** (arquitectura diseñada con respaldo en línea, desfase cero al momento del borrado). Las transacciones del intervalo se guardaron como archivos en otro lugar (proceso previsto cuando falla la base) y se **reprocesaron** después; carga del delta unos días más tarde (~miércoles de la semana siguiente). Base lista ~14:00 |
| Detalle técnico que dijo | AWS le comentó que revertir un borrado hecho **por consola** es más fácil que uno hecho **por Terraform**; "todavía no sé bien por qué" `[afirmación del profesor, no verificada]` |
| Qué funcionó | La arquitectura estaba preparada (no se perdió información), la comunicación y el armado del war room |
| Qué no funcionó | **Los controles para que el borrado no ocurra** |
| Acciones | Un checklist de mejoras, por ejemplo hacer que el pipeline valide antes de destruir; algunas se priorizaron y otras quedaron en backlog |

Lecciones que el profesor dejó implícitas: reportar rápido acorta el tiempo de detección; una buena arquitectura de respaldo limita el impacto; la falta no fue "de la persona" sino de **controles**.

## 7. Los cuatro intervalos de la línea de tiempo (diap. 12)

Ejemplo de la lámina (despliegue en el servicio de pagos que cambia el timeout de 5 s a 2 s):

| Hora | Evento |
|---|---|
| 21:04 | Despliegue |
| 21:09 | Inicio del impacto (la tasa de error sube a 18 %) |
| 21:38 | Detección (soporte reporta reclamos de clientes) |
| 21:44 | Respuesta (la persona de guardia confirma y abre el canal) |
| 21:52 | Mitigación (rollback al digest anterior; errores a 0,3 %) |
| Lun 10:00 | Resolución (el cambio vuelve con prueba de carga y política) |

| Intervalo | Duración | Definición |
|---|---|---|
| Tiempo a detectar | **29 min** | Del inicio del impacto a la detección; aquí detectó un cliente: "la alerta no existía" |
| Tiempo a responder | **6 min** | De la detección a una persona trabajando en el incidente |
| Tiempo a mitigar | **8 min** | Hasta que el usuario deja de sufrir el daño |
| Tiempo a restaurar | **43 min** | **La métrica DORA**: del inicio del impacto al servicio recuperado |

Verificación aritmética: 29 + 6 + 8 = 43 min (21:09 → 21:52). Mensaje de la lámina: "29 de los 43 minutos se fueron en enterarse. La acción con más efecto es una alerta sobre la tasa de error del servicio."

En clase el profesor se confundió con las cifras ("casi 50 minutos", "48 minutos") antes de aclarar que el impacto real fue de las 21:09 a las 21:52; **valen los 43 minutos de la lámina**. Aclaró que "respuesta" no significa solucionado, solo que alguien ya está mirando; y que "ni siquiera existen alertas, nos deberíamos enterar antes que el cliente". También dijo que un incidente puede tener orígenes técnicos, de proceso (por ejemplo, número de personas de guardia), de detección (mediciones) u organizacionales, con el ejemplo de los turnos nocturnos con **escalamiento de llamadas** (A, B, siguiente…, hasta el gerente si nadie contesta): si no existe ese esquema, es un problema de organización.

**Relación con el Laboratorio 4** `[mío]`: la métrica DORA "time to restore" que se te asignó aparece aquí con la definición de la lámina (inicio del impacto → servicio recuperado). Conviene comprobar que la definición usada en tu RESPUESTAS.md del Lab 4 sea coherente con esta (o justificar la diferencia con el caso).

## 8. Varias causas contribuyentes y los cinco porqués (diap. 13)

Mismo incidente de la lámina anterior (timeout de 5 s a 2 s en pagos, 43 min de errores, detectado por soporte):

| Técnica | Proceso | Detección | Organización |
|---|---|---|---|
| El proveedor de pagos responde en 3 s en hora punta; nadie lo había medido | Los cambios de configuración no pasan por la prueba de carga; solo los de código | Las alertas miran CPU y memoria; la tasa de error del servicio no tenía alerta | Despliegue a las 21:00 de un jueves con una sola persona de guardia y sin respaldo |

"Los cinco porqués siguen una sola cadena y se detienen en la primera respuesta cómoda. Cuatro categorías obligan a mirar donde la cadena no llega."

En clase, los cinco porqués (ejemplo: "¿por qué pasó? porque José hizo `terraform destroy` por error; ¿por qué lo hizo? porque trataba de…") llegan a la causa raíz hacia atrás. Un estudiante preguntó si **siempre se sabe el porqué**: respondió que hay que averiguarlo y que **a veces no se sabe** cuando faltan métricas (por ejemplo CPU y RAM no están al máximo pero las transacciones están lentas, y hay que mirar hilos concurrentes; si no se mide, no se puede confirmar). Eso mismo es una causa contribuyente de detección.

## 9. Acciones débiles, intermedias y fuertes (diap. 14)

| Débiles (dependen de que una persona recuerde) | Intermedias (agregan un paso que alguien ejecuta) | Fuertes (el sistema impide o detecta solo) |
|---|---|---|
| Capacitar al equipo; enviar un correo recordatorio; "tener más cuidado" con la configuración | Checklist antes de desplegar; revisión adicional en el PR; documentar el umbral del proveedor | Prueba de carga obligatoria para cambios de configuración; alerta sobre la tasa de error, con dueño; política que exige timeout ≥ latencia p99 del proveedor |

"Una acción fuerte vive en el pipeline o en la plataforma: una compuerta, una política, una alerta con dueño. Es el control nuevo que sale del incidente."

Comentarios del profesor: capacitar o enviar correo no sirve "si se le olvida o no lo lee"; un checklist **manual** o una tercera revisión de PR todavía depende de las personas; un documento de evidencias con 20 capturas de pantalla no dice qué se probó, frente a algo **codificado y automatizado**. Las acciones deben ser **concretas y realistas**.

## 10. El ritual del postmortem (diap. 15)

1. **Criterio escrito**: incidente de severidad 1 o 2, consumo de error budget sobre un umbral, o un casi incidente que el equipo considera relevante.
2. **Plazo corto**: borrador en cinco días hábiles, mientras la memoria y los registros siguen disponibles.
3. **Reunión facilitada**: conduce alguien que no participó; se revisa la línea de tiempo antes de hablar de causas.
4. **Publicación abierta**: todo el área lo puede leer; los nombres pueden aparecer como roles si el equipo lo prefiere.
5. **Seguimiento**: las acciones entran al backlog con fecha; se mide el **porcentaje cerrado a 30 días**.

"Un postmortem cuyas acciones nadie cierra enseña al equipo que escribirlo es un trámite." En clase: facilita alguien con mentalidad de "no buscamos culpables, buscamos soluciones" y no parcializado; se publica a los equipos cuidando la confidencialidad; "lo más importante es el seguimiento"; si nadie cierra las acciones, "un saludo a la bandera".

Pregunta de un estudiante: ¿qué pasa si la mala gestión viene de arriba? Respuesta: o se acepta hasta dónde se puede convivir con eso, o se decide irse; él mismo dejó una empresa de seguros por considerar que DevOps no mejoraría sin cambiar la forma de trabajar de infraestructura y operaciones, y reconoce que **pudo haber sido más valiente** y plantearlo a su gerente. Después proyectó un video (sin audio transcrito) cuyo contenido solo se infiere por lo dicho (ver sección 11).

## 11. Del incidente a la posición de protagonista (diap. 16 y 17)

**Diap. 16 · Del incidente al aprendizaje**: (1) ¿Qué ocurrió? Reconstruir hechos e impacto con evidencia; ¿qué condiciones lo hicieron posible?; (2) ¿Qué podemos cambiar? Distinguir lo que no controlamos de lo que podemos hacer, proponer o pedir a otros; (3) ¿Qué asumimos? Acordar una acción o conversación concreta: responsable, fecha y evidencia de mejora.

**Video** (la lámina 14 del PDF es solo la imagen fija): según lo que el profesor y el grupo comentaron, se ve a dos personas detenidas en una escalera sin funcionar; preguntó "¿qué rol están asumiendo?" y qué les permite ese rol (se respondió algo como "gestión/víctima" y "es muy cómodo", transcripción dudosa). `[inferido]` el rol es de **víctima**.

**Diap. 17 · Dos posiciones frente al mismo hecho.** Hecho: el despliegue falló porque alguien de infraestructura cambió una regla de red por consola, sin avisar.

| Posición de víctima | Posición de protagonista |
|---|---|
| "Infraestructura siempre hace cambios sin avisar. Mientras no cambien ellos, esto va a volver a pasar." | "No teníamos forma de detectar cambios hechos fuera del pipeline. Propongo una evaluación de drift diaria y el lunes lo converso con infraestructura." |
| La causa está afuera y el relato termina ahí; explica por qué no se pudo; espera que otro cambie | Busca lo que está en su dominio de acción; pregunta qué se puede hacer con lo que hay; hace pedidos y ofertas concretas, con fecha |

"Ser protagonista es elegir una respuesta posible: actuar, pedir ayuda o poner límites. No es asumir toda la culpa ni negar un daño real."

Lo que agregó el profesor:

- **Ser protagonista es una elección**; todos estamos a veces en rol de víctima y es **necesario pasar un rato** en él; lo importante es **cuánto se alarga**. Ser protagonista **no es** "ponerse la capa de Superman y hacerlo todo": es admitir lo que se puede, **aceptar límites y poner límites**, pedir ayuda.
- Aplicado a su propio incidente del `destroy`: pudo culpar al gerente que le pidió cambios en ese módulo, o el practicante pudo decir "no es mi problema"; "es totalmente diferente cómo vemos las problemáticas según qué rol nos colocamos".
- Explicó el **drift** como el equivalente a un `diff` entre lo declarado en Terraform y lo que existe en la nube, y cómo una evaluación diaria habría avisado.
- Ejemplo personal de él con un cuello de botella con operaciones: respirar, recordar que ellos cuidan estabilidad y seguridad, y plantear alternativas en lugar de quedarse en el "no". Otro, doméstico (llegar tarde cada día): pasar de culpar a organizarse para que ambos lleguen temprano.

## 12. Afirmaciones y juicios en el postmortem (diap. 18)

- **Afirmación**: se puede verificar con un registro, un commit o una métrica; es verdadera o falsa.
- **Juicio**: opinión sobre el hecho, bien o mal fundada; "siempre dice algo de quien la emite". Ejemplo del profesor: "el techo es blanco" (hecho) frente a "yo soy alto/chato" (compara contra un criterio).

| Frase del borrador | ¿Afirmación o juicio? |
|---|---|
| "El despliegue de las 21:04 cambió el timeout de 5 s a 2 s." | **Afirmación** |
| "El cambio se hizo sin el cuidado necesario." | **Juicio** (¿qué es "cuidado necesario"? para uno es revisar 20 veces, para otro una) |
| "La alerta de tasa de error no existía en el servicio de pagos." | **Afirmación** (con evidencia detrás) |
| "El equipo de guardia reaccionó tarde." | **Juicio** (se fundamenta: "se esperaba respuesta en menos de una hora y fue en dos") |

"La línea de tiempo se escribe solo con afirmaciones. Cada juicio se fundamenta con hechos o se retira del documento." En clase: "José siempre es descuidado" es un juicio; "José ejecutó el plan sin validar el plan antes del `apply`" es una afirmación. Los juicios, si aparecen, van en la parte de acciones y fundamentados.

## 13. Triángulo del drama y triángulo de empoderamiento

**Triángulo del drama** (lámina): Perseguidor ("Esto es tu culpa." Presiona, acusa o impone), Víctima ("No puedo hacer nada." Espera que otros cambien) y Salvador ("Déjalo, yo lo arreglo." Se hace cargo y desplaza al otro). "Son posiciones en una conversación, no identidades. Una misma persona puede pasar por los tres roles."

En clase usó la imagen de dos personas varadas en una escalera: un guardia de seguridad que dice "suban, siempre pasa lo mismo" (perseguidor); una persona que los carga o arregla la escalera por ellos (salvador, que "suena genial" pero hace que la víctima no se responsabilice y a veces busca sentirse importante); y los varados que dicen que la escalera no funciona (víctima). **Los roles rotan** (quien se queja puede pasar a salvador y luego a perseguidor). Anécdota personal en el bus escolar: esperaba que otro pasajero lo defendiera, o podía haber invertido los roles.

**Frases de equipo (lámina)**: Frontend: "No podemos avanzar porque backend todavía no termina la API." Desarrollo: "No podemos corregirlo porque QA no explica bien el error." QA: "Nos demoramos el doble porque desarrollo no nos responde." El profesor añadió una cuarta: el PM que dice "no sé qué informar porque no me responden".

**Triángulo de empoderamiento (lámina)**: Víctima → **Creador/Protagonista** ("¿Qué quiero y qué puedo hacer?", elige un paso y pide apoyo); Perseguidor → **Challenger/Retador** ("Esto necesita cambiar.", confronta con respeto y pone límites); Salvador → **Coach** ("¿Qué opciones tienes?", acompaña sin decidir por el otro). "Ser protagonista también es pedir ayuda y poner límites." En clase: el retador no ataca, "cree que yo puedo" (ejemplo del examen difícil como reto); el coaching —que dice practicar, **coaching ontológico**— acompaña con preguntas.

**Reformular las tres frases (lámina)**: "con una acción o un pedido concreto, una persona y un plazo". Observaciones del profesor: frontend puede sugerir otras formas de avanzar (por ejemplo contratos o mocks de la API); si el pedido "ya mandé correo y no responde" lleva una o dos semanas, la respuesta no puede ser esperar: **los correos son lo que menos se responde**, hay que lograr la comunicación (llamar, escalar con contexto): "tienes que lograr que suceda"; la respuesta a "que ellos lo desarrollen" de QA o dev lleva a que "dejen de ser Superman". "No te quedes parado con el primer no."

## 14. DISC: observar cómo respondemos (explicación en clase)

- Cuatro tendencias: **D** (dominante: orientada al resultado y a la acción, "no me importa cómo, hay que lograr resultados"), **I** (influyente: muchas ideas, persuade, le cuesta concretar), **S** (soporte/coordinación con calma: orientada a las personas, le preocupa "qué va a pensar el otro"), **C** (consciente/concienzudo: reglas, proceso, documentación; le cuesta romper el molde). Los pares son opuestos (D frente a S, I frente a C).
- La distribución del curso (puntos en un plano) se carga un poco hacia la izquierda; "es muy característico de ingeniería". Marketing creativo está a la derecha; marketing orientado a ventas, más a la izquierda `[inferido: izquierda = D y C]`.
- **Idea central**: de bebés "tenemos todos los colores" (imagen del pie de un bebé); el resultado de hoy **no determina** quién eres, cambia con las decisiones y la práctica. Sirve para ver fortalezas y costos: un D va orientado al logro y a veces se olvida de las personas; si lo sabe, puede ser más consciente. También sirve para escoger a quién apoyar para qué (por ejemplo elegir a la persona más dominante para liderar algo que debe salir rápido, sabiendo que a otros puede caerles mal). "Cada rol funciona para una cosa; no hay ninguno bueno o malo; la plasticidad de moverse entre cuadrantes según la situación es lo chévere." El profesor contó que su propio resultado se mueve entre I y S y que aún trabaja en concretar ideas; no reproduzco más.
- Lámina de cierre: "DISC describe tendencias; no determina nuestra respuesta. Cualquier tendencia puede actuar desde víctima o protagonista. **Observamos conductas** (sin etiquetar personas)."

## 15. Actividad: el mismo caso, dos conversaciones (diap. 19 y casos)

**Consigna (lámina)**: 50 minutos en total. (1) **Preparación 15 min**: elegir un caso y repartir los cinco roles, mezclando tendencias DISC y asegurando que todos intervengan. (2) **Víctima 2 min**: cada personaje explica por qué no puede avanzar y espera que otros cambien; "evitar caricaturas". (3) **Protagonista 2 min**: mismos hechos y límites; hacer pedidos, negociar y cerrar una decisión con responsable y plazo. (4) **Conversación 3 min**: el público propone tendencias DISC observadas y señala las frases que abrieron o cerraron posibilidades. Qué observar: ¿qué conducta sugiere D, I, S o C? ¿Qué cambió entre las dos escenas? Recordar: no hay una letra correcta que adivinar; observar conductas sin etiquetar personas; protagonista también pide ayuda y pone límites. Con tres o cuatro equipos: 15 min de preparación + hasta 28 min de escenas y conversación + 7 min de síntesis. No hace falta contar experiencias personales. En clase dijo que la preparación sería de ~10 min.

**Los cinco casos**:

| Caso | Situación | Roles |
|---|---|---|
| 1 · "Una campaña con fecha" | Marketing ya pagó la campaña del lunes; el formulario no está listo; ayer pidieron los últimos cambios y desarrollo dice que no llega | Líder de desarrollo, desarrollador, QA, responsable de marketing, coordinador de campaña |
| 2 · "Eso no estaba incluido" | En la demo el cliente pregunta por un reporte que necesita; la consultora dice que no estaba incluido, el cliente asegura que lo pidió; pocos días y presupuesto comprometido | Líder de proyecto de la consultora, desarrollador, ejecutivo de cuenta, responsable del cliente, usuario del sistema |
| 3 · "Seguridad frena la salida" | Mañana sale el sistema; seguridad encontró que algunos usuarios ven datos que no les corresponden; desarrollo dice que la revisión llegó tarde; negocio insiste en la fecha | Líder de desarrollo, desarrollador, analista de seguridad, responsable de seguridad, representante del negocio |
| 4 · "Nos dejaron solos" | Una actualización dejó el servicio fuera de línea; operaciones lo recuperó sin instrucciones ni a quién llamar; desarrollo dice que nadie le pidió acompañar la salida | Líder de desarrollo, desarrollador, operador de guardia, líder de operaciones, responsable del servicio |
| 5 · "Todos esperan a alguien" | La entrega volvió a retrasarse: frontend espera una API, backend dice que los requisitos cambian y QA recibe versiones tarde y reporta errores sin detalle | Desarrollador frontend, desarrollador backend, analista QA, product owner, líder del equipo |

**Cómo se desarrolló**: el profesor formó **tres grupos de cuatro** pidiendo que cada grupo tuviera una persona "roja" (dominante) y que se mezclaran colores (hubo dos rojos); con cuatro integrantes sobraba un rol de los cinco. Hizo un **ejemplo con el caso 4**, que ningún grupo tomó: en la escena de víctima, el desarrollador dice "nadie me avisó", el líder de desarrollo quiere escalar con la gerencia y el operador responde que no pueden ser los únicos responsables de que el sistema siga arriba; en la escena de protagonista cada uno se organiza distinto según su color (por ejemplo un líder verde que cuida a su desarrollador; "saquen su amarillo hoy día, es algo que los ingenieros necesitamos"). Indicó que pueden actuar o hacer mímica; lo valioso es **vivirlo**, para reconocerlo cuando "les toque en la cancha".

Escenas que alcanzó a registrar la transcripción (resumen mío, con las dudas de siempre):

- **Caso 1 (campaña)**: víctima: marketing presiona, desarrollo dice que ya delegó en QA y desarrollo y que "si no sale hoy habrá que esperar", cada uno traslada la responsabilidad ("ya hice lo que hice"). Protagonista: reunión para decidir **recortar a las funcionalidades básicas**, almacenar temporalmente la información y organizarla después de la campaña, monitorear cada avance y comunicar que "habrá algo funcional el día de la campaña".
- **Caso 3 (seguridad frena la salida)**: víctima: el analista dice que está sobrecargado; el desarrollador, que la revisión llegó tarde; negocio, que debe facturar mañana. Protagonista: revisión final **conjunta** del código con pruebas de seguridad, repartir actividades, priorizar lo crítico para llegar a tiempo, dejar lo demás para después y definir métricas.
- **Caso 2 (reporte no incluido)**: el ejecutivo de cuenta recibe el pedido, el líder de proyecto y el desarrollador revisan correos y no hay evidencia del reporte; se propone a la parte del cliente **una ampliación de un mes con costo adicional**, que el cliente rechaza ("mañana te entregan lo contratado"). La transcripción no separa con claridad las escenas de víctima y de protagonista.

**Retroalimentación del profesor**: "no entren al detalle de qué dijimos, sino cómo se sintió estar en una posición y en otra". Comentarios de estudiantes: "es fácil quejarse, más difícil solucionar", "más satisfactorio cuando superas un problema". A un participante con tendencia roja le señaló que se notaba muy enfocado al objetivo y que, al final, aterrizó en colaboración: "no pierdas la esencia del rojo, abraza un poco al verde, al amarillo". Sobre el caso 2 (cómo se sintió la conversación con el cliente): un estudiante dijo que "tal vez no mapeamos lo que el cliente esperaba, no documentamos en las entrevistas o no lo escuchamos". El profesor añadió que en Perú y gran parte de Sudamérica **la palabra "no" está mal vista** y se tiende a decirle sí a todo al cliente (y, desde el lado del cliente, "también me he aprovechado de proveedores"); **decir que no a tiempo es necesario**: "si el contrato dice esto, ya está". Contó un proyecto de data analytics en que los 6 reportes vendidos se volvieron 6 archivos de Power BI con 6 pestañas y 8 dashboards cada una, de los ~10 pedidos solo hicieron 4 y cerraron el proyecto para no aumentar la pérdida. Recomendación: **tener la conversación difícil cuanto antes**, hay más tiempo para negociar; dilatarla es peor.

## 16. Cierre, Laboratorio 5 y lo que se llevan

- **Formato del laboratorio (dicho por el profesor, cierre de clase)**: "un mix entre una parte técnica y una parte parecida a esta"; responder preguntas como grupo, grupal, en salas de Zoom donde los profesores irán pasando; "un casito cada uno"; **no se actúa, se llena el documento**; **no se desarrolla nada, no se usa Claude ("no va a haber ningún pipeline detrás"), es más de prácticas**; ejemplo de pregunta: "hemos encontrado vulnerabilidades en contenedores, ¿qué herramienta podríamos usar?" **Un entregable por grupo**, 4 personas, máximo 5; los grupos no se arman ahí mismo, el profesor revisará cuántos grupos salen. Mencionó la transmisión en línea por YouTube `[ambiguo en la transcripción]`. **Discrepancia con las láminas**: la lámina del Lab 5 describe un pipeline con composición, imagen y política sobre el plan; confirmar el entregable con la guía.
- El profesor pidió impresiones de cómo se sintieron y qué se llevan; dijo que los programas de liderazgo se dividen en dos: **trabajar en uno mismo** (reconocer cómo me comporto) y, luego, **estrategias con los demás** (feedback, entender a otros); que le gustaría hacer un taller de eso.
- **Puntos clave (diap. 21)**: (1) donde se castiga al mensajero, la información deja de llegar (la cultura generativa investiga la falla; *Accelerate* la vincula con mejor desempeño); (2) el error humano abre la investigación (las preguntas que abren terminan en un procedimiento, un control o una alerta); (3) DISC describe tendencias, no determina nuestra respuesta; (4) el aprendizaje se convierte en acuerdos (cambios de proceso, aprendizaje y conversaciones, con responsable, fecha y seguimiento); (5) el protagonista responde con lo que está en su dominio (afirmaciones para los hechos, juicios fundamentados, pedidos concretos).
- **Para cerrar juntos (diap. 22)**: ¿qué frases dejaron al equipo esperando que otros cambiaran?; ¿qué pedidos o límites abrieron una posibilidad?; ¿cómo se aplica esta distinción fuera del trabajo? (usar ejemplos ficticios; compartir algo personal es voluntario). Frase: "No elegimos todo lo que ocurre. Podemos explorar cómo responder, qué pedir y qué límites cuidar."

## 17. Conexión con sesiones anteriores (relaciones mías, no dictadas)

- **S11 (seguridad II)**: la fila RV.3 del NIST ("analizar la causa raíz y mejorar el proceso") dice que el postmortem es el cierre del ciclo; la pregunta "cuando un control detiene en producción a las 2 a. m., ¿quién decide?" se contesta con el criterio escrito; el drift y el cambio de emergencia por consola de S11 reaparecen aquí como el hecho de la lámina 17 (infraestructura cambió una regla por consola) y como la acción fuerte "evaluación de drift diaria".
- **S1/S2 (cultura)**: el "sin culpa" y "siempre es culpa del proceso" de la S4 y las anécdotas de la S10 (la del puerto SSH abierto se resolvió sin despido) son aplicaciones de la cultura generativa.
- **DORA / Lab 4**: el *time to restore* de la lámina 12 y los cuatro intervalos (detectar, responder, mitigar, restaurar) sirven para descomponer cualquier número de restauración; la acción "alerta con dueño" acorta el intervalo mayor.
- **S8/S9 (compuertas, umbrales)**: acciones fuertes = compuertas del pipeline (prueba de carga obligatoria, política de timeout ≥ p99 del proveedor).
- **Observabilidad**: "¿qué señal existía y por qué no llegó a nadie?" se conecta con la lectura de la parte IV de *The DevOps Handbook* (retroalimentación y telemetría) para la siguiente sesión.

## 18. Pendientes e incertidumbres

- **Láminas 4 a 6 ausentes** del PDF (sección A); la transcripción cubre solo una parte de la "ruta profesional" (historia del profesor, motivos de cambiar de empresa, que no le gustaban seguridad ni operaciones).
- **El video** proyectado (~46–48 min) no tiene audio transcrito; su contenido lo infiero por los comentarios; la transcripción de las respuestas del grupo es dudosa.
- **El tramo de preparación de sketches** (~01:34–01:46) quedó como ruido; las escenas de los casos 1, 2 y 3 están resumidas desde fragmentos y puede haber errores de asignación entre víctima y protagonista.
- **Formato del Lab 5**: discrepancia entre lámina y lo dicho; no sé cuál es el vigente. Conviene leer la guía del 5/11.
- La fecha de la siguiente sesión no consta en las fuentes. Tampoco consta cómo se entregará la nota o devolverán los parciales.
- Las anécdotas del profesor (Oracle, war rooms de 100–120 personas, el borrado con Terraform, el incidente de AWS del año pasado, los modelos de innovación) no están verificadas; la identificación con el corte de AWS de octubre de 2025 es mía y no la busqué en esta sesión.
- Las cifras del ejemplo de la lámina 12 (21:04…, 18 %, 0,3 %, 29/6/8/43 min) son ilustrativas del material; las que dijo el profesor al hablar ("48 minutos") no coinciden y no deben citarse.
