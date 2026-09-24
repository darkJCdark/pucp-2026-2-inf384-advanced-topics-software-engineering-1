# Semana 1 — Ciclo de vida del desarrollo y Value Stream Mapping

## Introducción del curso
- Los laboratorios se realizan con apoyo de IA; los exámenes no.
- El profesor y los JPs tienen experiencia práctica en DevOps.

## Ciclo de vida del desarrollo de software (SDLC)
- El diagrama teórico del SDLC no refleja lo que ocurre realmente en una organización.
- El comité de cambios (o "pase") lo lidera el equipo de Operaciones, no Desarrollo.
- Equipo de Desarrollo: busca sacar funcionalidades lo más rápido posible para satisfacer al negocio.
- Equipo de Operaciones: vela por mantener el software en producción, que el costo sea bajo y que no se caiga.
- Esta tensión entre Dev (velocidad) y Ops (estabilidad) es la raíz del problema que DevOps busca resolver.

## Value Stream Mapping (VSM)
- Metodología para levantar información: sentarse con la persona involucrada en el proceso; si el proceso cambia, hay que rehacer el levantamiento.
- Un VSM es una fotografía de un recorrido completo de un proceso (de principio a fin), no una medición continua.
- Ejemplo trabajado en clase: eficiencia de 3.5%, equivalente a 8 horas de trabajo efectivo sobre 9 días 13 horas de tiempo total transcurrido.
- En ese ejemplo, el cuello de botella se ubicaba en la etapa de QA.
- Los "lead time" del VSM se miden en tiempo calendario (no solo tiempo de trabajo efectivo).

## Métricas DORA
- Clasificación de Google: nivel Elite (despliegan varias veces al día) vs. nivel Low.
- Diferencia clave: el VSM es una foto puntual; DORA es monitoreo continuo (levanta información automáticamente y compara el hoy contra la semana pasada).
- El VSM puede ser un insumo para DORA.
- En el enfoque DORA, el cuello de botella observado suele estar en el Lead Time.

Las cuatro métricas DORA:
1. **Frecuencia de despliegue**: qué tan rápido se despliega una funcionalidad acotada que aporte valor al cliente.
2. **Lead time para cambios**: tiempo desde que se hace el commit de código hasta que llega a producción.
3. **Tasa de fallas en cambios**: no tiene sentido desplegar muy seguido si eso genera muchos problemas.
4. **Tiempo de restauración del servicio**: relacionado con SRE — que producción no se caiga y, si se cae, que se recupere automáticamente.

Herramientas de apoyo mencionadas: Trello, Jira (gestión de proyectos); OpenSearch de Oracle (observabilidad) — cada acción de usuario se registra en un panel, y ante errores del servidor debe generarse una alerta que se cierra automáticamente al resolverse.

## Ideas clave de la sesión
- Velocidad y estabilidad no se intercambian: en producción se avanza más lento por razones de seguridad, y eso es intencional.
- Problemáticas comunes en las organizaciones:
  - Mentalidad de crecimiento: tener el problema identificado y pensar continuamente qué más hacer para solucionarlo.
  - Revisar el riesgo asociado a cada "pase" a producción.
  - Tratar QA como etapa final del proceso, en vez de integrar pruebas automatizadas a lo largo de todo el flujo (pirámide de automatización de pruebas).
- Conclusión de la sesión: **"El tiempo de espera es el problema."**
