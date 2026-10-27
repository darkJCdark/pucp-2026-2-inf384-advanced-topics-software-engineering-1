# INF384 · Semana 10 — Automatización de seguridad I: SAST, SCA, escaneo de imágenes de contenedor y qué hacer con cada hallazgo

Curso: INF384 Temas Avanzados en Ingeniería de Software 1 (PUCP, 2026-2) · Prof. José Felipe del Castillo Salazar
Capítulo 6 (1 de 2): Automatización de seguridad. El profesor la llama "sesión 9" al inicio de la clase y dice que es la primera de dos sesiones de seguridad; la hoja de ruta del curso (diapositiva 1 de la S8) la ubica como "Automatización de seguridad I" (sesión 9), y la de la próxima semana como "Automatización de seguridad II" (sesión 10). La carpeta del curso la rotula S10.
Clase del 23/10/2026 (grabación iniciada a las 9:42 p. m.), duración 129 min.

Fuentes: transcripción automática (`TA S10.txt`, con mucho ruido: "Sengre/CENGRE" = Semgrep, "SAS" = SAST, "SSA" = SCA, "S-Bone/esboque" = SBOM, "Cuba" = QA, "Divelo/diólogo" = develop, etc.). **No hay diapositivas de esta sesión en la carpeta**: el PDF que está en la carpeta S10 (`INF384_S8_Automatizacion_calidad_II.pdf`) es el de la sesión 8 (calidad II), no el de seguridad. El profesor proyectó láminas y un repositorio demo que solo conozco por lo que dijo; donde aparece "lámina" es lo que se puede inferir del audio.

Marca de fidelidad: lo que está entre `[inferido]` es mi lectura de una transcripción ambigua o una decisión que el profesor dejó implícita, no una afirmación textual suya.

## 0. Avisos de la clase

- Es la **primera de dos sesiones de seguridad**: controles, revisiones y qué hacer cuando salen hallazgos. Esta sesión: SAST (ya usaron algo de Semgrep, ahora "más a detalle, no tan práctico con Semgrep"), análisis de librerías/componentes (SCA), análisis de contenedores y la política sobre los hallazgos.
- **Próximo laboratorio**: el profesor enviará **el caso antes** de la sesión de laboratorio (no la solución ni el producto) para que se lea con tiempo. Cambia el enfoque por el feedback del laboratorio anterior ("si no pailan, pero la hierba es con nosotros... no aprendimos mucho"): será parecido al examen, se da el caso y la solución planteada, y **lo técnico es anecdótico; lo importante es el porqué de lo que se hace**. `[inferido]` Corresponde al Lab 4 de la hoja de ruta.
- **Sesión 11**: ya hay una **invitada confirmada** que acompañará la clase.
- **Feriado por la visita del Papa**: se anunció un feriado parcial de dos días (desde la una de la tarde en adelante). El profesor aún espera las indicaciones de la universidad sobre cómo afecta a las sesiones.
- Mención de una fecha el 20 de noviembre que la transcripción no permite reconstruir.

## 1. Preámbulo: por qué importa la seguridad (anécdotas del profesor)

Son comentarios del profesor sobre noticias recientes; no los he verificado.

- Menciona una vulnerabilidad reciente en **PHP** (ransomware, CMS como WordPress, Drupal y Joomla, páginas de empresas peruanas) y un incidente en GitHub en que cuentas vulneradas accedieron a repositorios privados; también dice que "creo que hackearon a Interseguro". Su mensaje: no se puede evitar que aparezcan vulnerabilidades nuevas; **lo importante es reaccionar rápido** cuando algo pasa.
- **Supply chain (cadena de suministro) de software**: así como en la fabricación de autos se compran, almacenan y ensamblan piezas, nuestro software depende de **proveedores = las librerías que usamos**. Un ataque a la cadena de suministro vulnera a uno de esos proveedores (se publica una versión nueva con código malicioso) y quien la usa se ve afectado sin enterarse.
- **Log4j** como ejemplo: librería de logs en Java; sale una vulnerabilidad, la mitad de una empresa (ERP incluido) es Java, hay que subir a la versión parchada y **redesplegar 200 aplicaciones** (hoy con microservicios pueden ser miles). De ahí los **controles compensatorios o por capas**: mientras se parcha todo, se bloquea el tráfico malicioso en otra capa (por ejemplo el firewall de aplicación).
- **Seguridad por capas**: analogía de la tienda (sensores en la puerta, cámaras, vigilantes dentro y fuera): no hay un solo control, varios y complementarios. Por eso existen el WAF y el **SOC** (equipo que monitorea 24/7 todo lo de seguridad de la empresa).
- **Anécdota 1 (post mortem)**: un DevOps abrió el puerto SSH del firewall como público "por un rato" para una tarea y se olvidó de cerrarlo. Seguridad lo detectó al día siguiente. Alguien podría haber consultado una URL que devolvía el archivo de configuración de la aplicación (usuario y contraseña de la base de datos, claves de APIs). Se cambiaron **todas las credenciales en una noche y madrugada**, incluidas algunas de proveedores. **No lo despidieron**: la decisión fue sensibilizar y aprovechar su experiencia; el mensaje al resto era evitar que vuelva a ocurrir. Conecta con el **análisis post mortem** visto en las primeras clases de cultura (se volverá a tocar en prácticas).
- **Anécdota 2** (el profesor aclara que es "más rumor que hecho"): una billetera digital muy usada, con devs, DevOps e infra sentados juntos y flujo muy ágil; algunas personas malintencionadas metieron código malicioso para beneficiarse y nadie lo notó. Se separaron funciones y se montó un proceso mucho más riguroso (revisión antes de pasar de una etapa a otra; infra "con candado" que solo recibe tickets). Lección: el control revisado entre roles ("ya confío en ti") deja de ser suficiente cuando el riesgo y la reputación son altos.

## 2. Dónde está la seguridad en el pipeline

Commit → Build → Pruebas → Validación de calidad → Validación de seguridad → Publicar (artefacto) → Desplegar → Observabilidad/monitoreo. El curso recorre el camino por segunda vez haciendo "doble clic" en seguridad.

Mapa de controles que el profesor ordenó al final de la parte técnica:

| Qué se analiza | Control | Qué detecta |
|---|---|---|
| Código fuente propio | **SAST** (análisis estático) | Patrones de código vulnerables |
| Secretos en el repo | Detección de secretos | Credenciales subidas por error (visto en la S4) |
| Dependencias declaradas (npm, `package.json`/lock file…) | **SCA** (Software Composition Analysis) | Librerías con vulnerabilidades conocidas, directas y transitivas |
| Imagen del contenedor | **Container scanning** | Capas: sistema operativo de la imagen base, runtime (por ejemplo Node), dependencias de la aplicación |
| Aplicación desplegada | **DAST** | Se verá más adelante; "mucho más complejo y caro de implementar" |

Con SAST + SCA + escaneo de imagen "tenemos recontra asegurada la aplicación en que **no estoy inyectando nada nuevo de vulnerabilidad**".

## 3. SAST (análisis estático de seguridad)

### Qué hace y qué lo limita

- Analiza el código buscando **patrones** y los cataloga por impacto (alto, medio, bajo). La idea de ponerlo en el pipeline es tener **feedback lo antes posible**.
- Su base de conocimiento son **patrones ya conocidos del mercado**: no detecta vulnerabilidades que nadie ha identificado todavía (es lo que pasó con PHP: "nadie nunca detecta esa vulnerabilidad" hasta que se conoce).
- Algunas herramientas se pagan **por líneas analizadas**; las gratuitas (como Semgrep) sirven para una prueba de concepto, pero "a escala normalmente necesitas una herramienta pagada". Ejemplo de herramienta paga que mencionó: **Fortify**. Dijo que es raro que una empresa grande use solo la versión libre, y que muchas financieras y bancas todavía no lo tienen implementado.

### El dilema costo vs. etapa donde correrlo

Flujo de ramas y esfuerzo (promedios de un requerimiento grande de uno a tres meses):

| Etapa | Pases típicos por cada pase a producción | Tiempo para llegar `[inferido por la transcripción]` |
|---|---|---|
| Rama feature | 30 a 50 | ~1 día |
| Develop | 15 a 25 | ~1 semana |
| QA | 3 a 5 | ~2 semanas a 1 mes |
| Producción | 1 | ~1 a 3 meses |

- Ejemplo numérico del profesor: si el desarrollador cambia 20 líneas por pase, analizar en feature son 20 × 30 = **600 líneas** cobradas, en develop 20 × 15 = 300 y en producción 20.
- Como la herramienta cobra por línea analizada, **muchas empresas sacrifican los controles tempranos** y solo analizan en QA (y a veces antes de producción) porque "no nos da el presupuesto". En producción el análisis "empieza de cajón": **antes de ir a producción siempre hay que analizar**.
- Cada hallazgo que aparece más a la derecha **rebota** (te encontraron un bug, vuelves a corregir) y alarga el flujo, conectando con el VSM (S1) y con la S4: el mismo hallazgo cuesta distinto según dónde aparece.

### Cómo funciona el análisis: de la fuente al sumidero

Ejemplo de arquitectura (el profesor usó Yape como ejemplo): app móvil → HTTPS por el puerto 443 → **WAF** (firewall de aplicaciones, no solo de red) → API gateway → APIs propias de la aplicación (con su base de datos) → APIs del core del banco → base de datos del core (cuentas, saldos) y entre medias otros firewalls. Hay varios saltos hasta el dato, y cada herramienta de seguridad intenta ubicar **dónde ingresa el dato** (fuente) y **dónde ocurre la vulnerabilidad** (sumidero) para decidir dónde poner controles.

- **SQL injection** como caso: un `INSERT` arma el query concatenando el campo `nombre`; el atacante envía una cadena que cierra la comilla, termina el comando con `;` e incluye `DROP DATABASE`. Controles en capas: el **WAF** puede detectar patrones maliciosos en la entrada, y el **código** debe sanear (buena práctica: usar librerías o frameworks que escapan los caracteres, de modo que el nombre quede literalmente como texto "Drop the…" y no se interprete como comando). Casi todos los frameworks de acceso a base de datos ya vienen con esa protección.
- La capa de saneamiento puede estar **distribuida** (parte en el WAF, parte en el código) para tener controles diversos. Un saneamiento básico: si el dato es numérico, convertirlo a entero antes de usarlo.
- Se dan dos tipos de análisis: reglas de **patrones** (segundos) y **análisis de flujo de datos** a través de capas (minutos).
- **Seguridad por diseño**: no se arregla solo con herramientas. Ejemplos: el teclado numérico de una app bancaria que viaja por HTTP como **hashes de la posición** y no los dígitos reales, y la **MFA aplicada solo en los puntos críticos** (equilibrio entre seguridad y experiencia de usuario; el profesor criticó que algunos servicios piden MFA para todo).

### Falsos positivos, ruido y excepciones

- **Falso positivo**: el SAST marca como vulnerable un dato que en realidad ya fue saneado por una función propia que la herramienta no reconoce. Solución: **configurar la regla** para considerar esa función de saneamiento como válida.
- **Código de prueba junto al productivo**: los unit tests (carpeta paralela) contienen código con comportamiento a propósito que no va a producción; debe **exceptuarse** del análisis.
- Si hay muchas alertas sin riesgo real, la gente **deja de creerlas y de mirarlas** (como las notificaciones del celular). Hay que **tunear y depurar** para que el análisis sirva.
- Cuando se silencia un hallazgo, **siempre se deja un comentario con el motivo** (si no, la persona siguiente no sabrá por qué está silenciado ni si debe reactivarlo). Misma idea de la excepción con razón, responsable y vencimiento de la S4.
- **Falso negativo / zero-day**: lo que no está en la base de conocimiento no se detecta; "hay situaciones en que no puedo hacer nada y debo aceptarlo; lo importante es reaccionar rápido cuando se identifique".

## 4. SCA y SBOM (librerías)

- El SAST cubre el código propio; las **librerías** (declaradas en `package.json`, con su `package-lock`/lock file) traen sus propias vulnerabilidades. Ejemplo: 8000 líneas propias y 12 librerías declaradas, pero cada una invoca otras librerías de la comunidad: son las **dependencias transitivas** (indirectas), donde también pueden nacer vulnerabilidades como la de PHP.
- Existen **bases de conocimiento públicas** que dicen qué versión de qué librería tiene qué vulnerabilidad (CVE). El proceso: la herramienta revisa librería por librería contra esa base. Por eso importa declarar **la versión exacta** ("nombre y apellido"): el **lock file** fija el árbol completo y permite que el análisis sea reproducible.
- **SBOM** (inventario del árbol de componentes de cada aplicación): es el insumo del análisis y además permite la **búsqueda inversa**: "¿qué aplicaciones usan la versión X de esta librería?". En un incidente como el de PHP, en lugar de llamar a todos los developers a revisar su código, se consulta la herramienta que ya correlacionó los SBOM de todas las aplicaciones. El profesor lo llama "la potencia del SBOM".
- Ataques posibles a este eslabón: **componente malicioso** (el proveedor fue vulnerado y publicó una versión nueva con código inyectado; a veces el equipo se salva si la vulnerabilidad se publica antes de que la adopte) y **typosquatting** (librerías con nombre casi igual a una conocida). Mencionó que la **IA** puede inventar o equivocar el nombre de una librería y llevar a instalar una falsa, aunque dice que se ha visto poco.

## 5. Imágenes de contenedor

### Imagen base corporativa

- Las imágenes base vienen de la comunidad (internet). Una empresa suele crear su **propia versión corporativa** (por ejemplo un NGINX personalizado) y obliga a que todos los servicios (A, B, C…) partan de esa imagen. Ventajas: controla qué versión usan todos, centraliza cambios en el Dockerfile corporativo (con su propio versionado: 1, 2, 3…), y se sabe quién usa qué.
- Problema: los equipos de desarrollo **no son responsables de esa imagen**, pero el análisis de su aplicación **los bloquea** por una vulnerabilidad que viene de la base. Solución observada en la práctica: un **pipeline nocturno que analiza la imagen base más reciente** para alertar antes de que un equipo quiera hacer un cambio y se tope con el hallazgo.
- Si sale una vulnerabilidad en la versión 3.4.5 del motor y aún no existe la 3.4.6 (la comunidad tarda días o un par de semanas, "ya van más de 10 días"), se registra una **excepción por el identificador CVE** (por ejemplo "la 456") para que los equipos pasen mientras tanto.
- Origen posible del hallazgo: vulnerabilidad **transitiva** (viene de un tercero), o consecuencia de algo que la empresa agregó a la imagen base (por ejemplo una dependencia de una herramienta de monitoreo o de seguridad en ejecución), o líneas del Dockerfile de la imagen comunitaria.
- **Escenario poco visto**: una aplicación en producción desde hace un año, sin cambios; en ese tiempo aparecen 20 vulnerabilidades en la imagen base. Hay que modificar el Dockerfile, **reconstruir la imagen** y volver a pasar el pipeline aunque la aplicación no cambió. Es difícil en procesos porque suelen ser equipos distintos los que mantienen las imágenes y los que mantienen las aplicaciones. Es una buena práctica a instaurar.

### Superficie de ataque: elegir imagen base pequeña

Cuanto más grande es el sistema operativo de la imagen, más paquetes y más vulnerabilidades. Cifras dictadas por el profesor para NGINX: imagen completa **300–500 paquetes**, **slim 80–120**, **distroless 10–20** (solo el runtime y las librerías mínimas). Contrapartida de distroless: no tiene shell, así que no se puede entrar a depurar el contenedor. En **serverless** el proveedor administra la imagen base y el equipo solo sube código.

Resultados del demo (de la transcripción; las cifras pueden estar algo desordenadas):

| Imagen | Vulnerabilidades críticas que mostró |
|---|---|
| Completa | 49 en el sistema operativo + 1 en el runtime (Node) |
| Slim | 5 |
| Distroless | 1 |
| Base de Lambda | caso aparte (es la imagen que exige AWS Lambda) |

Orden de tamaño: completa > slim > distroless. El mensaje: pasar de 49 a 5 elimina 44 vulnerabilidades "que están de más porque no necesito esos paquetes".

### Digest y cuándo escanear

- Fijar la imagen base por **digest** (el "apellido" inmutable), no solo por etiqueta de versión, para que no cambie sin aviso (visto en S5/S6).
- Las vulnerabilidades dependen de lo que se descubre en la comunidad, así que **el escaneo no puede ser solo al inicio**: entre que el dev termina la rama feature y el cambio llega a QA pasan semanas, y en ese lapso pueden aparecer vulnerabilidades nuevas. Hay que correr estos análisis **antes de publicar, de forma continua y periódicamente aun en producción** ("hasta cierto tiempo una persona o un proceso va escaneando para ver si tiene alguna nueva vulnerabilidad"). Es la misma lógica de la S4 y del RV.1 del NIST.

## 6. Arquetipo de pipeline y orden de los controles

- El profesor mostró una plantilla: análisis estático → análisis de dependencias → análisis de imagen, **cada uno con su política de cuándo para y cuándo no**, con umbrales declarados (qué severidad detiene). El paso siguiente corre solo si el anterior no falla, y antes de desplegar la imagen final se vuelve a verificar que no tenga vulnerabilidades.
- Normalmente se coloca una **sección de seguridad** con las pruebas de seguridad que se definan, **en paralelo** cuando se puede. Orden sensato: primero el **build** ("si no hace build, ¿para qué hago el análisis?"); el **SAST va antes del despliegue**, nunca después.
- Principio de la S1/S8: si el paso demora mucho y se pone en todos los pull requests, **el equipo de desarrollo evita subir cambios**. El tiempo depende de la herramienta; se profundiza en la próxima clase.

## 7. Demo: repositorio con tres análisis

Repositorio con un pipeline por cada análisis (SAST, SCA, imagen) y una aplicación con vulnerabilidades simuladas.

- **SAST (Semgrep)**: reglas personalizadas en un YAML (qué se omite, reglas propias). El pipeline sale en rojo porque un dato **no se sanea** antes de usarse en el query; en el resumen del pipeline se ve la regla, el archivo, la línea y el mensaje (personalizado en el demo; en la práctica sale un código/CVE que se puede buscar). El arreglo: parsear a entero el dato antes de usarlo. En la herramienta (vista del proyecto) se ven por ejemplo **13 vulnerabilidades a nivel de código**, con la línea, la recomendación de cómo sanearlo y el enlace de referencia.
- **SCA**: toma el **lock file**, corre el escaneo y encuentra **15 vulnerabilidades: 6 bajas, 3 medias, 6 altas, 0 críticas**. La regla del pipeline es "alta o crítica detiene", así que se detiene (por las 6 altas; el profesor dijo "críticas" en un momento, pero antes había dicho cero críticas). La herramienta indica la versión actual y **la versión que ya trae el arreglo**. Normalmente es solo cambiar de versión (minor/patch, sin impacto), pero un **major upgrade** puede romper compatibilidad y obliga a modificar código (ejemplo: una librería de rutas en 0.1.7 que debería ir a 1.9.0; la transcripción no deja claro el nombre exacto). El equipo de seguridad mira esto para saber **cuánta presión** poner.
- **Imágenes**: construye varias variantes (completa, slim, distroless, base Lambda) y las analiza para comparar (resultados en la sección 5).
- **El pipeline también se analiza**: Semgrep revisa el YAML de los workflows y puede alertar de una *action* de terceros vulnerable, así que "el propio YAML del pipeline se detiene" por la misma vía.

## 8. Política: severidad, plazos y consolidación

- Con muchas fuentes (SAST + SCA + imágenes) por 200 aplicaciones, "es una sábana de cosas": hacen falta **herramientas de consolidación** que muestren qué aplicaciones tienen más vulnerabilidades y cuáles más críticas, porque seguridad no puede revisar el pipeline de cada aplicación.
- **Severidad no es prioridad** (idea de la S4: alcanzabilidad, exposición, existencia de corrección). Preguntas del profesor para evaluar un hallazgo: ¿qué tan fácil es llegar? (exposición a internet o red interna; la separación de redes la vuelve "no expuesta"), ¿qué podría hacer el atacante si la explota? (alcance: baja la app, roba información), ¿ya se está explotando?, ¿ya existe corrección?
- Ejemplo de una herramienta cara de gestión de riesgo en nube (Prisma Cloud) usada por un cliente: de **3000 vulnerabilidades**, 2000 eran críticas o altas, y solo **80 estaban expuestas a internet y eran alcanzables**; esas 80 son las que se atacan primero. Sin la herramienta hay que "meter cabeza y usar otras herramientas gratuitas".
- **Política típica de seguridad** que citó:
  - **Críticas y altas: no pasan.** Si la solución no existe (no hay parche), no pasa tampoco; para pasar se necesita una **carta de riesgo**, o sea que el dueño de negocio asuma el riesgo por escrito.
  - **Medias y bajas**: se puede pasar a producción con un **compromiso de corrección**: medias en máximo **15 días**, bajas en máximo **un mes** (con firma de compromiso). `[nota]` Más adelante, en el ejercicio, habló de "30 días" para un hallazgo medio: esa cifra no coincide con los 15 días dichos antes; en el examen conviene citar el plazo que dé el caso.
  - El dolor operativo es el **seguimiento** (perseguir, escalar con jefes); ahí entran las automatizaciones.
- **Excepción vs. informar** (aclaración del profesor): *excepción* = se exceptúa el hallazgo de la regla del pipeline para que el despliegue pase (por ejemplo un crítico sin parche, previo acuerdo con seguridad); *informar* = el pipeline no se detiene pero el hallazgo se reporta y no se pierde de vista. En la siguiente sesión se verá cómo se catalogan los hallazgos aceptados (con mensaje) frente a los no aceptados que se informan siempre.

## 9. Cómo se organiza la responsabilidad (parte organizacional)

El profesor dijo que lo técnico "termina siendo anecdótico" frente a esto:

- **Hace ~10 años**: negocio por un lado y tecnología por otro; el negocio pide, tecnología cotiza y entrega meses o años después.
- **Hoy en grandes empresas (modelo de tribus y squads, con ejemplo de Pacífico Seguros)**: las tribus agrupan squads; cada squad tiene un **Product Owner empoderado** (Scrum) responsable de la experiencia del cliente de una aplicación (ejemplo: la app "MiSeguro"), con un analista de negocio y los desarrolladores (frontends, backend…). El PO justifica un **presupuesto** (ejemplo del profesor: 10 mil por mes durante 3 meses para traer 50 mil al negocio) y cada trimestre **presenta en un comité de gerentes** con sustento demostrable.
- **Equipos transversales**: DevOps (un chapter de más de 30 personas; se asigna, por ejemplo, 30 % de un DevOps a una aplicación porque ya hay automatización), seguridad e infraestructura. **Seguridad da el visto bueno para pasar a producción** y marca las pautas que se vieron.
- Si el pipeline ya trae las validaciones y corta, seguridad no tiene que intervenir en cada despliegue; su trabajo más fuerte es el **diseño de seguridad**, que un desarrollador por sí solo difícilmente incorpora (requiere otros conocimientos y mentalidad).
- **Dashboard para el Product Owner**: para su aplicación muestra por ejemplo 30 críticas, 20 altas, 10 medias, 0 bajas. Al ponerle **nombre y apellido** y amarrar los hallazgos a los dueños, la vulnerabilidad deja de ser "del developer que la puso" y pasa a ser del dueño de la aplicación, lo que ayuda a priorizar y sensibilizar.
- Comentó también su experiencia en DP2: no se puede hacer *daily* real cuando el equipo no trabaja junto; hoy se mezclan enfoques ágiles y waterfall según el contexto.

## 10. Ejercicio: ¿detener, informar o excepción?

El profesor proyectó seis hallazgos y los fue discutiendo. Las decisiones marcadas `[inferido]` las dejó implícitas.

| # | Hallazgo | Dato clave | Decisión |
|---|---|---|---|
| 1 | SAST: parámetro HTTP concatenado en consulta SQL, sin saneamiento en el flujo completo | API pública, explotable | **Detener** (crítica) |
| 2 | SAST: uso de una función hash obsoleta para nombres de archivos temporales | No tiene consumo externo, no es explotable desde fuera; no se resuelve con un upgrade (hay que cambiar la lógica) | **Informar**, no detener `[inferido]` |
| 3 | SCA: hallazgo **crítico** heredado de una librería de compresión | No alcanzable desde afuera, la función vulnerable no se usa, **hay parche** | **Detener** y exigir el upgrade: es crítico y hay parche; como no se usa la función, el upgrade probablemente no afecte el código |
| 4 | SCA: hallazgo **medio** en un framework HTTP (manejo de cabeceras) | Alcanzable desde internet, exposición alta, hay parche | **Informar** con plazo de corrección (15–30 días) |
| 5 | Imagen: **14 vulnerabilidades altas** en paquetes del sistema operativo de la imagen base | La imagen base lleva 3 meses sin reconstruirse, los parches ya existen | **Detener** y obligar a corregir: alta + parche disponible |
| 6 | Cadena de suministro: 3 *actions* de terceros del workflow referenciadas por tag mayor; una está comprometida y accede a los secretos del pipeline | Roba credenciales; es difícil de detectar desde el propio pipeline | **Detener**; el profesor la considera la más crítica `[inferido]`. Semgrep también analiza el YAML del pipeline |

Reglas que se desprenden: (a) crítico/alto + corrección disponible → detener; (b) severidad media o baja con corrección → informar con plazo; (c) si el hallazgo es alto o crítico pero **no existe parche**, no queda más que una **excepción** acordada con seguridad (o carta de riesgo); (d) los hallazgos no alcanzables y sin uso no deben bloquear, pero tampoco se ocultan.

### Pregunta de un alumno: "¿qué pasa si es crítica, pero no existe parche y se detiene?"

Respuesta del profesor: seguridad y el arquitecto se reúnen para entender **qué pasaría si se explota**. Lo normal es que seguridad sea firme: **no puede pasar a producción**. Si la aplicación **ya está en producción** y se detecta el problema, a veces seguridad pide **bajar la aplicación**; ahí el gobierno deja de ser solo técnico: se convoca al líder de la tribu, que evalúa el riesgo y decide (por ejemplo, bajarla y avisar a los clientes que no hay aplicación por unos días). Siempre hay que "ponerse creativo" mientras la comunidad no publica el parche (mitigaciones, controles por capas como el WAF, reducir la operación), con el equipo de riesgos.

## 11. Qué llevarse

1. **El análisis de seguridad no es solo uno**: SAST (código), SCA (librerías, con lock file y SBOM) y escaneo de imagen (SO + runtime + dependencias) cubren capas distintas; la secuencia importa (build primero, SAST antes de desplegar).
2. **La base de conocimiento es el límite**: las herramientas solo detectan lo ya conocido, por eso el escaneo debe **repetirse en el tiempo** (antes de publicar, periódicamente y en producción) y hay que reaccionar rápido ante lo nuevo.
3. **El costo por línea analizada empuja a escanear tarde**; hay que balancear costo, etapa y velocidad del pipeline.
4. **Menor superficie de ataque**: imagen base pequeña (slim/distroless), fijada por digest, centralizada y escaneada de forma programada.
5. **Cada hallazgo se decide con contexto, no solo con la severidad**: exposición, alcance, explotabilidad, existencia de parche; resultado posible: detener, informar con plazo o excepción con motivo.
6. **Silenciar o exceptuar siempre con comentario y responsable**; ruido sin tunear = alertas ignoradas.
7. **Seguridad por capas** (WAF + saneamiento en código + controles compensatorios mientras se parcha).
8. **Responsabilidad con dueño**: el Product Owner ve el dashboard de vulnerabilidades de su aplicación; seguridad da el visto bueno a producción; el seguimiento se automatiza.
9. **Después de un incidente, post mortem sin culpables** (el caso del puerto SSH): se corrige el proceso, no solo se castiga a la persona (coincide con lo dicho en la S4: "siempre es culpa del proceso").

## 12. Conexión con sesiones anteriores (relaciones mías, no dictadas)

- **S4 (integración continua II)**: criterios para que un control viva en CI (rápido, determinista, acotable al cambio, sin ambiente desplegado), "detiene lo nuevo, inventaría lo heredado", severidad ≠ prioridad, excepción con identificador, razón, responsable y vencimiento, y la ubicación en NIST SP 800-218 (SCA → PW.4, SAST/secretos → PW.7, identificación continua de vulnerabilidades → RV.1). Esta sesión profundiza esos mismos controles con herramientas y ejemplos.
- **S5/S6 (despliegue continuo)**: digest inmutable y rollback; aquí se usa para fijar imágenes y para el pipeline que reconstruye la base.
- **S8 (calidad II)**: política de compuertas ("qué detiene, qué informa, qué revierte") y orquestación (rápido primero, paralelo); aquí aparece la versión de seguridad de esa política.
- Hallazgo 6 (acciones de terceros con acceso a secretos) relaciona con la detección de secretos y con el principio de que el pipeline también es código a analizar.

## 13. Pendientes e incertidumbres

- No tengo las láminas de la sesión (el PDF de la carpeta es el de S8); si quieres, súbelas y verifico cifras (tiempos por etapa, paquetes por imagen, resultados del demo) y las decisiones del ejercicio.
- La transcripción no permite confirmar: nombres exactos de algunas herramientas y librerías, la fecha del 20 de noviembre, y cuál etapa corresponde a cada tiempo de la tabla de la sección 3.
- Falta la **sesión 10** (seguridad II): ahí se verá la gestión de hallazgos aceptados/informados y, según el profesor, un adelanto del DAST.
