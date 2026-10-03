# INF384 · Sesión 07 — Automatización de calidad I: pirámide de pruebas, pruebas unitarias y de integración en el pipeline

Curso: INF384 Temas Avanzados en Ingeniería de Software 1 (PUCP, 2026-2) · Prof. José Felipe del Castillo Salazar
Capítulo 5 (1 de 2) · 3 horas · RA2 (pipelines que aseguran calidad, trazabilidad y eficiencia del ciclo de vida).
Clase dictada el 2/10/2026 (grabación de 124 min).
Fuentes: transcripción (`Temas S7.txt`, con partes ruidosas por la transcripción automática) y diapositivas (`Sesión 07 - PPT Calidad 1.pdf`).

## 0. Avisos de la clase

**Examen parcial (próximo):**
- Es teórico-aplicado: "mucho de la teoría implementada en los casos"; no hay que programar un pipeline en papel, a lo sumo diseñarlo o dibujarlo.
- Se parece a lo trabajado en la sesión 1 con el **Value Stream Mapping**: trasladar ese mapa a cómo se usarían las estrategias automatizadas vistas en clase.
- Duración disponible: 3 horas, todo presencial, en sitio (no hay parte para resolver en casa).
- El profesor entregará **un anexo impreso con información de los capítulos** junto con el examen; no se permite material virtual (no hay equipos).
- En la siguiente sesión habrá un **simulacro previo**, más acotado, para dar una idea del formato.

**Ruta del curso:** sesión 7 es la primera del Capítulo 5 (Automatización de calidad). La sesión 8 completa el capítulo (pruebas end-to-end con Playwright, pruebas de rendimiento con K6). El Lab 4 se ubica después, en el bloque de automatización de seguridad según la ruta de la diapositiva.

**Tarea para la sesión 8 (diapositiva final):** revisar el job de pruebas del propio Laboratorio 3 y anotar **qué nivel de prueba cubre y cuánto tarda**.

## Dónde estamos en el pipeline

Commit → Build → **Pruebas** → Calidad → Seguridad → Publicar → Desplegar → Observar.

- Ya resuelto en sesiones anteriores: el quality gate bloquea la publicación si la cobertura del código nuevo no alcanza el umbral; existe un artefacto único identificado por su digest que recorre todos los ambientes; el despliegue se verifica con prueba de humo y ventana de observación; la tasa de fallo de cambios ya tiene una forma de medirse.
- Preguntas todavía sin decidir (las que esta sesión responde): ¿qué defecto detecta cada tipo de prueba que los otros no? ¿cuántos minutos de pipeline se justifican en pruebas? ¿qué pruebas corren antes de publicar el artefacto y cuáles después de desplegarlo? ¿qué se hace con una prueba que a veces pasa y a veces falla (flaky)?
- En el Laboratorio 3 el job `pruebas` producía el archivo de cobertura que leía el quality gate; esta sesión decide **qué debe probar esa etapa y cuánto puede tardar**.

## A. La forma de la cartera de pruebas

### Punto de partida: verificación y validación

Lo que ya traen de Ingeniería de Software: niveles de prueba (unitaria, integración, sistema, aceptación), diseño de casos (particiones de equivalencia, valores límite), V&V como actividades del ciclo de vida, planes de prueba y trazabilidad a requisitos.

Lo que agrega este capítulo:
- La prueba como **compuerta automática** que decide si el artefacto avanza.
- El **tiempo de ejecución como restricción de diseño** de la suite.
- **Determinismo**: el mismo commit produce el mismo resultado en cualquier runner.
- La evidencia de cada ejecución queda registrada junto al artefacto.

Frase clave: *una prueba protege el pipeline cuando corre en cada cambio, sin que nadie tenga que decidir ejecutarla.* El pipeline es lo que obliga a automatizar la prueba: si no pasa, el código no avanza.

### La pirámide de pruebas

| Nivel | Tiempo típico | Qué localiza cuando falla |
|---|---|---|
| End to end | minutos | Un recorrido de usuario roto, sin indicar en qué servicio |
| Integración | segundos | La frontera entre dos componentes: consulta, esquema, serialización |
| Unitarias | milisegundos | La función exacta y la línea que produce el resultado incorrecto |

Forma: base ancha (muchas unitarias), punta angosta (pocas end to end) — "menos pruebas · más lentas · más cerca del usuario". **La proporción se decide por el costo de detectar cada defecto.**

Argumentos del profesor para empezar por la base:
- En un banco por internet hay un front y cientos de APIs por debajo; encontrar dónde está el problema es mucho más rápido en una unitaria que en una end to end.
- Las pruebas end to end suelen correr a nivel de interfaz (HTML: IDs, etiquetas), por lo que sobre **software legado** que no se diseñó pensando en automatización generan falsos positivos y obligan a modificar el código de la aplicación. Ejemplo citado: un cliente con una plataforma de años donde intentar Playwright con IA exigió cambiar el código para que funcionara. Con IA o sin ella, esta teoría de la pirámide se mantiene vigente en aplicaciones heredadas.
- Aun si una aplicación nueva se genera con IA ("vibe coding"), a las dos semanas llega un cambio de reglas (ej. cambio en el cálculo del IGV) y, sin pruebas unitarias ni conocimiento del código, varios desarrolladores tocando a la vez lo rompen; las unitarias son lo que valida cada cambio.

### El triángulo invertido (antipatrón)

Muchas end to end y regresión manual en la punta, pocas unitarias y de integración en la base. Malas prácticas:
- La regresión se valida a mano antes de cada salida.
- La suite automatizada opera sobre la interfaz y tarda horas.
- Las unitarias existen solo para alcanzar el umbral de cobertura.
- El equipo vuelve a probar en certificación aunque el pipeline esté en verde (el "ejército de testers" manuales, porque el proceso aún no confía en la automatización).

Impacto: probar cuesta tanto que los cambios se acumulan en lotes grandes; la certificación pasa a ser la etapa más larga del flujo; el retrabajo aparece semanas después, cuando el responsable ya cambió de tarea. *Un equipo con el triángulo invertido reparte la calidad entre las personas que prueban a mano; el pipeline solo confirma que el código compila.* Cómo se arregla: más que técnica, es negociación, convencimiento, paciencia y conseguir aliados.

Otros contra-ejemplos contados en clase:
- La etapa de pruebas suele ser de las más largas, sobre todo con muchas dependencias (RENIEC, otros bancos, SBS) y es iterativa: se prueba, vuelve a desarrollo, se corrige.
- Anécdota de una universidad que exigía compatibilidad de su campus virtual con todos los navegadores y versiones (IE 6/7, Opera, Safari, Firefox) y se probaba a mano uno por uno, aunque Opera no llegaba al 2% de uso: un requisito de negocio sin sentido que volvió cautivo al equipo.
- Qué pruebas servirían para eso hoy: una mezcla; en front se pueden hacer unitarias de responsive, pero la compatibilidad real entre navegadores se acerca más a integración/end to end.

### El tiempo de retroalimentación es un presupuesto

Cartera de ejemplo de un servicio mediano (órdenes de magnitud, no mediciones):

| Nivel | Cantidad | Tiempo c/u | Total secuencial |
|---|---|---|---|
| Unitarias | 2 000 | 5 ms | 10 s |
| Integración | 150 | 2 s | 5 min |
| End to end | 40 | 45 s | 30 min |
| **Suite completa** | | | **≈ 35 min** |

- 35 minutos por commit cambian el comportamiento del developer: agrupa cambios para no esperar y el lote crece (menos commits, más grandes → más riesgo).
- El paralelismo compra tiempo con cómputo: cuatro ejecutores bajan las end to end a unos 8 minutos.
- No toda la suite corre en cada commit: las end to end completas pueden correr antes de promover a QA o producción.

### Qué nivel detecta cada defecto

| Defecto | Unitaria | Integración | End to end |
|---|---|---|---|
| Regla de negocio mal calculada (descuento, redondeo, tarifa) | Detecta | Detecta tarde | Detecta tarde |
| Consulta SQL incorrecta o migración incompatible | **No detecta: el mock lo oculta** | Detecta | Detecta tarde |
| Cambio de formato en la respuesta de una API | Parcial | Detecta | Detecta |
| Permisos, variables de entorno y configuración del ambiente | No detecta | Parcial | Detecta |
| Recorrido de usuario roto entre varios servicios/pantallas | No detecta | No detecta | Detecta |

Regla: cada nivel se justifica por los defectos que los inferiores no pueden ver. Una end to end que verifica un cálculo repite en minutos lo que una unitaria resuelve en milisegundos.

Ejemplos usados por el profesor para ilustrar la tabla:
- Cambio de contrato/formato de una API (agregar, quitar o renombrar un campo): la unitaria prueba la lógica interna, no la exposición hacia afuera; lo atrapa la integración.
- Recorrido roto entre dos plataformas: el flujo de pago del Estado cambia el formato/longitud del ID del comprobante en un lado, y la otra plataforma que lo consume se rompe — solo lo detecta una end to end.
- Ejemplo de lógica de negocio con end to end: una carga de alumnos asociada a un "aula"; la regla cambia a "código de institución + sección + año"; se cambia un componente y se olvida el otro: el profesor ya no encuentra la lista. Una prueba end to end valida ambas partes del proceso.

### Qué corre y cuándo (flujo con ramas)

- **Rama feature (local/por commit):** primero las unitarias, siempre (se pueden correr en la máquina del desarrollador; mínimo un commit diario). Cuando una feature está lista y se abre el pull request, corren unitarias e integración (acotada).
- Las end to end pueden correr al pasar a un ambiente, algunas en la transición y otras directamente en el ambiente (dependen mucho de la data y de que sí o sí esté desplegado).
- Luego, según las políticas de la empresa, el pase a producción puede exigir además pruebas de performance y de seguridad.
- **Pruebas de regresión** no es un nivel aparte: es el *conjunto* de pruebas (unitarias, de integración y/o end to end) que se elige correr para asegurar que un cambio no rompió nada. En las suites end to end el proyecto es "una sábana de casos" (cliente con DNI vencido, con DNI vigente, etc.) que se prenden o apagan según qué se quiera regresionar.
- **Cuándo la integración es crítica:** al modificar un contrato de API, crear una API nueva o cambiar su URL.

## B. Pruebas unitarias en el pipeline

### Propiedades de una prueba unitaria confiable — FIRST

1. **Rápida (Fast):** se mide en milisegundos; una suite de miles termina en segundos y se ejecuta en cada commit.
2. **Aislada (Isolated):** no depende de red, disco, reloj del sistema ni del orden en que corren las demás pruebas.
3. **Repetible (Repeatable):** el mismo commit produce el mismo resultado en la máquina del desarrollador y en cualquier runner (se puede ejecutar 20 veces y debe dar lo mismo).
4. **Auto-verificable (Self-validating):** termina en éxito o fallo por una aserción (`assert`/`expect`); nadie tiene que leer una salida para decidir.
5. **Oportuna (Timely):** se escribe con el cambio, en el mismo pull request que modifica el comportamiento; la validación ocurre **antes del merge**, para no ensuciar la rama con un problema.

Ejemplo de preguntas de clase: una prueba que usa la fecha del sistema y falla cada 31 de diciembre → viola **aislamiento/repetibilidad** (depende del reloj). Una prueba que llama a la función sin valor esperado y da 100% de cobertura → no verifica nada (no es auto-verificable / sin aserción real).

### Cómo se aíslan: dobles de prueba

El "doble" reemplaza **lo que el componente no controla**: red, reloj, base de datos, aleatoriedad, servicios externos. La lógica propia se prueba real.

| Doble | Qué hace | Ejemplo de clase |
|---|---|---|
| **Stub** | Devuelve respuestas fijas para forzar un caso | El servicio de tarifas responde 503 y se verifica el manejo del error |
| **Mock** | Verifica que una interacción ocurrió / simula la respuesta de una dependencia | Mock del ERP SAP que responde un JSON `{id, cantidad}` para validar la lógica de "hay stock / no hay stock" (>0 hay, ≤0 no hay) |
| **Fake** | Implementación funcional simplificada | Un repositorio en memoria o un CSV con datos predefinidos en lugar de la base de datos real (es un camino "fake": nunca existirá en producción, pero permite probar la lógica que manipula los datos) |
| **Spy** | Registra las llamadas sobre el objeto real | Registrar en un log "se envió el correo" cuando no se puede probar que el correo realmente llegue |

Reglas y criterios mencionados:
- **No moquear tu propia lógica**: ejemplo de la calculadora — la prueba de `suma(a, b)` debe calcular el resultado real a partir de entradas, no afirmar `2 == 2`. Si se mockea lo que se quiere probar, la prueba no prueba nada (el caso extremo es el `1 == 1`).
- Los dobles son útiles en integraciones con **terceros o recursos limitados/costosos**: ejemplos RENIEC (siempre responde igual; se hardcodea un JSON, a veces por tipo de DNI — "si termina en 01, cliente vigente; si termina en 02, ...") y los cajeros automáticos de otros bancos (recurso limitado, hay que coordinar tarjetas de prueba y laboratorio). Anécdota: una factura gigante de RENIEC a fin de mes porque alguien conectó pruebas al servicio de producción por error.
- Limitación importante: **un mock oculta defectos de consulta SQL y de migraciones** — por eso existe el nivel de integración.

### Anatomía de una prueba unitaria (ejemplo en JavaScript/Jest de la diapositiva)

```js
// src/tarifa.js
function calcularTarifa(pedido, reglas) {        // 1 las reglas entran como parámetro
  const base = pedido.pesoKg * reglas.precioPorKg;
  const recargo = pedido.urgente
    ? base * reglas.recargoUrgente : 0;
  return Math.round((base + recargo) * 100) / 100;
}

// test/tarifa.test.js
describe("calcularTarifa", () => {
  const reglas = { precioPorKg: 4.5, recargoUrgente: 0.2 };

  it("aplica recargo a pedidos urgentes", () => {
    const pedido = { pesoKg: 10, urgente: true };       // 2 datos mínimos creados en la prueba
    expect(calcularTarifa(pedido, reglas)).toBe(54);    // 3 una aserción sobre el resultado
  });

  it("redondea a dos decimales", () => {                // 4 valores límite como casos
    const pedido = { pesoKg: 3.333, urgente: false };
    expect(calcularTarifa(pedido, reglas)).toBe(15);
  });
});
```

Puntos: (1) las reglas entran como parámetro: la función no lee configuración ni base de datos, no necesita dobles; (2) datos mínimos, creados dentro de la propia prueba, a la vista; (3) una aserción sobre lo observable (el valor devuelto); (4) los valores límite del diseño de casos de V&V se convierten en pruebas ejecutables. Esta función tiene dos caminos (`urgente` verdadero/falso): hacen falta al menos dos pruebas.

### Cobertura: qué mide y qué no mide

- **Qué mide:** qué líneas y qué ramas del código ejecutó la suite durante la corrida. El archivo de cobertura da un porcentaje por archivo y un total. Si solo se prueba el camino `urgente: false`, la línea del recargo queda sin cubrir.
- **Qué no mide:** si alguna aserción verificó **realmente** el resultado de ese código. Un `it("calcula la tarifa", () => { calcularTarifa(...) })` sin `expect` da 100% de líneas cubiertas con 0 verificaciones. Y el desarrollador que pone `1 == 1` para "cubrir" sale como línea validada.
- El umbral del quality gate (SonarCloud, visto en el laboratorio) puede bloquear por **cobertura total o del código nuevo (incremento)**.
- Las **pruebas de mutación** miden verificación: cambian un `>` por un `>=` y comprueban que alguna prueba falle.
- La cobertura es confiable para una sola afirmación: **el código no cubierto no fue ejecutado por ninguna prueba.** Anécdota: un desarrollador de una aseguradora que, para cumplir la regla de "80% de cobertura o no pasa", escribía pruebas con `1 == 1`; el framework no sabe qué se valida. Mitigación mencionada: herramientas con IA que revisan pull requests (ej. **CodeRabbit**: analiza el PR, genera diagrama de secuencia, recomendaciones de seguridad y calidad).
- Cada lenguaje tiene su propia herramienta de cobertura (Java una, Python otra). En la demo, el archivo de cobertura XML mostró línea por línea qué líneas estaban cubiertas (ej. un `create_engine` sin cobertura).

## C. Pruebas de integración

### Dos alcances

- **Integración acotada:** un componente propio contra una dependencia **real y efímera** (el repositorio contra una base de datos creada para la corrida). Corre en el runner, en segundos. Detecta lo que los mocks ocultan: consultas SQL, migraciones, serialización, tiempos de espera, manejo de conexiones.
- **Integración amplia:** varios servicios desplegados juntos en un ambiente compartido (ej. contra la base de datos real de desarrollo). Requiere que todos estén en la versión correcta al mismo tiempo y hereda las esperas del ambiente compartido.
- La compatibilidad entre servicios se valida con **validación de contratos de APIs**.
- Regla: la mayor parte de la integración se prueba acotada, dentro del runner; la amplia queda para lo que solo existe con los servicios desplegados.

### Dependencias efímeras dentro del job (GitHub Actions)

```yaml
jobs:
  integracion:
    runs-on: ubuntu-latest
    services:
      db:                                   # 1 la base nace y muere con el job
        image: postgres:16
        env:
          POSTGRES_PASSWORD: prueba
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready           # 2 el job espera a que la base responda
          --health-interval 5s
    env:
      DB_URL: postgres://postgres:prueba@localhost:5432/postgres
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run migrar                 # 3 se aplican las migraciones del repositorio
      - run: npm run test:integracion       # 4 otro comando, otro job, otro tiempo
```

- Cada corrida parte de una base vacía; no hay datos de la corrida anterior.
- Sin el chequeo de salud, las primeras pruebas fallan por conexión.
- Una migración incompatible falla aquí, antes de llegar a un ambiente.
- Las unitarias no esperan a que la base arranque: van en otro job. La etiqueta `services:` de GitHub Actions levanta un contenedor antes de los steps y solo lo usan los steps; en Lab 3 no se había visto.
- Si dos corridas ocurren en paralelo, cada una levanta **su propia base**.

### Migraciones de base de datos (concepto)

Herramientas tipo **Flyway / Liquibase**: la base de datos se lleva como código versionado en scripts incrementales (nunca se edita el archivo anterior; se agrega uno nuevo con el `ALTER TABLE`). Permite montar la base cuando se quiera con tablas y datos de demo ya definidos, y preparar casos específicos (categoría sin productos, con uno, con muchos para probar paginación). Es un concepto "medio avanzado": pocas empresas lo usan bien (el profesor vio solo una que lo hacía más o menos correctamente); en la práctica casi todos usan una base de desarrollo compartida y no pueden probar bien por falta de tiempo para implementar esto. Es el camino ideal, pero no el más común.

### Datos de prueba sin estado compartido

1. Cada prueba crea los datos que necesita (no depende de registros que dejó otra prueba ni de una carga inicial manual).
2. Identificadores únicos por corrida (dos jobs en paralelo sobre la misma base no chocan por la misma clave).
3. Limpieza por transacción (la prueba revierte al terminar) o por esquema propio por corrida.
4. Datos sintéticos; si se usa una copia de producción, debe estar **anonimizada** y acotada al volumen necesario.

Advertencias:
- *Una prueba que depende de datos dejados por otra pasa o falla según el orden de ejecución; es la causa más frecuente de pruebas de integración inestables.*
- Anonimizar bien: conservar la coherencia donde se necesita (si todos los DNI son 8888, se pierde la unicidad; usar correlativos 001, 002...) pero poner datos por defecto a teléfonos/direcciones. Anécdota: en una aseguradora llegaron cartas de cobro dirigidas a clientes... con la dirección de la propia empresa, porque la dirección por defecto de los datos de prueba/migración quedó en producción.
- Traer la base de producción sin anonimizar es un riesgo de seguridad de datos.

## Demo en clase: proyecto en Python con pytest y GitHub Actions

Repositorio de ejemplo (servicio de productos/pedidos con acceso a una base de datos y una integración tipo SAP) con tres pipelines; el pipeline de **CI** se mostró en detalle:

1. Instalación de `requirements`.
2. Validación de sintaxis (lint; en Python, el equivalente a ESLint).
3. Pruebas unitarias con **pytest**, generando un reporte de resultados (XML, ej. 24 pruebas en < 1 s) y un reporte de cobertura por archivo (ej. un archivo al 100%, base de datos al 75%, mail al 95% → un total).
4. Pruebas de integración contra una base de datos Postgres como contenedor (`services:`), también con pytest (el mismo framework sirve para unitarias e integración; basta con meter la conexión a BD en la lógica de prueba).

Detalles técnicos señalados:
- **`strategy.matrix`** de GitHub Actions: ejecuta el mismo job para cada valor de un arreglo (el ejemplo corría las unitarias para Python 3.11, 3.12 y 3.13); equivale a un `for each` y se usa cuando hay que correr lo mismo con variables diferentes.
- Se puede acotar con un parámetro qué carpetas/conjunto de pruebas correr (ej. `app/` y `app/services/`).
- **El CI debe correr en el pull request y no en el merge**: la alerta llega antes de ensuciar la rama principal; si alguien más toma ese código como origen para una nueva rama, hereda el problema. Para que el PR **se bloquee** si el pipeline falla hay que configurar esa regla (rulesets de la rama; mismo tema que la sesión 6: protección en la plataforma, no en el archivo del pipeline).
- Cada lenguaje/herramienta genera la cobertura de forma distinta; en la demo se vio tanto el archivo XML como el reporte impreso en el log.

**Herramientas alternativas de integración:** pytest está pensado para unitarias, pero sirve para integración si el código lo conecta a la BD. Para integración orquestando APIs de forma más potente y con reportes más ricos, existe **Karate**: los escenarios se declaran en sintaxis tipo Gherkin/Cucumber ("dado un cliente regular y estos productos... espero que me devuelvas 200 puntos"), sin escribir código; se pueden etiquetar (`@tag`) escenarios para elegir cuáles correr en cada momento (suite completa vs. subconjunto). Mostró un reporte con funcionalidades y escenarios pasados (ej. 9 escenarios de "pedidos" pasaron). La automatización de pruebas es "todo un capítulo de metodología, formas y herramientas".

## Cierre — preguntas de repaso hechas en vivo (quiz)

Preguntas formuladas al final (formato de dinámica en línea) y sus respuestas razonadas:
- *Un cálculo de descuento devuelve un valor incorrecto. ¿Qué prueba lo detecta en 10 segundos?* → la unitaria (la más rápida y la que apunta a la línea).
- *Una suite tarda 35 minutos por commit. ¿Qué hará el equipo de desarrollo?* → acumular cambios en lotes más grandes (comportamiento perverso del tiempo de retroalimentación); no resuelve escribir más unitarias.
- *Una prueba usa la fecha del sistema y falla cada 31 de diciembre. ¿Qué propiedad viola?* → aislamiento/repetibilidad (depende del reloj del sistema).
- *Una prueba llama a la función sin valor esperado y la cobertura queda en ~100%. ¿Qué se verificó?* → nada: cobertura no es verificación (no hay aserción).
- *Un cambio de base de datos incompatible pasa todas las unitarias. ¿Qué lo detecta antes de existir un ambiente?* → integración acotada: se levanta la base como contenedor en el pipeline y se aplican las migraciones.

## Puntos clave (diapositiva final)

1. **La proporción de pruebas se decide por el costo de detectar cada defecto.** La base de la pirámide localiza en milisegundos; la punta confirma recorridos completos.
2. **El tiempo de retroalimentación es parte del presupuesto del pipeline.** Una suite que tarda media hora empuja al equipo a acumular cambios.
3. **Una prueba unitaria sirve como compuerta cuando es rápida, aislada y repetible.** La confianza en el verde se pierde con una sola prueba inestable tolerada.
4. **La cobertura señala el código que ninguna prueba ejecutó.** La verificación la aportan las aserciones, y las pruebas de mutación la miden.
5. **La integración acotada corre en el runner con dependencias efímeras.** Detecta consultas, migraciones y serialización antes de que exista un ambiente.

## Lo que viene (sesión 8)

- Pruebas **end to end con Playwright** y los retos de automatizar sobre HTML (selectores/IDs, falsos positivos, aplicaciones legadas).
- Pruebas de **rendimiento (performance)** con **K6** (herramienta de carga; el profesor traerá un ejemplo).
- Complemento de la estrategia de ejecución: qué corre en cada etapa (local, PR, ambiente).
- Simulacro del parcial.
- Recordatorio: llevar anotado el nivel de prueba y el tiempo del job `pruebas` de su Laboratorio 3.

> Nota de fidelidad: la transcripción automática de esta sesión tiene fragmentos ruidosos o repetidos (inicio, minutos ~03:00–04:00, ~01:02–01:03 y la parte del quiz). Lo anterior se reconstruyó cruzando la transcripción con las diapositivas; lo de formato/fechas del parcial se tomó de la conversación inicial de la grabación y conviene confirmarlo con el profesor o en Paideia.
