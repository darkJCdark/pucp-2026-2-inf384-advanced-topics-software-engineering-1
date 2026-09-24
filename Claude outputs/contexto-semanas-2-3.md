# Semanas 2 y 3 — Introducción a DevSecOps e Integración Continua I

## Sesión 2: Introducción a DevSecOps

### Qué sí es DevSecOps
- Coordinar y compartir responsabilidades entre equipos.
- Es una capacidad organizacional que se va construyendo progresivamente, no una herramienta que se instala.
- Ejemplo BCP: para el App conviene implementar DevOps; el Core sigue en COBOL, con otra realidad tecnológica.

### El problema original: el muro entre Dev y Ops
- Dev: por ejemplo, subir una nueva versión de un sistema.
- Sec: ¿son seguras esas actualizaciones?
- Ops: mantener el sistema en producción.
- "Sec" se ubica en medio de Dev y Ops porque la auditoría de seguridad demora mucho tiempo y genera observaciones que retrasan el flujo.

### Origen del enfoque
- Libro *The Phoenix Project: A Novel About IT, DevOps, and Helping Your Business Win*.
- NIST: marcos de referencia de seguridad para el desarrollo de software.

### Las cinco dimensiones de DevSecOps
1. **Culture**: es la base; si falla, todo lo demás se cae. Ejemplo: AWS implementó análisis post-mortem.
2. **Automation**: velocidad en la retroalimentación; si el pipeline ya funcionó antes, da confianza de que no fallará después; reduce el riesgo del error humano.
3. **Lean**: mejora continua; es mejor tener poco trabajo en curso (Work In Progress) y mayor velocidad.
4. **Measurement**: métricas DORA; si la medición no es automática ni continua, es difícil tomar decisiones basadas en datos.
5. **Sharing**: compartir información y mejoras; la práctica del post-mortem es "nos equivocamos, lo compartimos, implementamos mejoras".

### Modelos de estructura de equipos
- **Silos separados**: Dev, Sec y Ops trabajan de forma aislada.
- **Capacidad embebida**: personas con el mismo conocimiento distribuidas en distintos equipos, cada una trabajando de forma distinta.
- **Equipo de plataforma**: equipo centralizado que construye los pipelines/plantillas que todos deben seguir.
- **SRE (equipo de confiabilidad)**: enfocado en la operación ("cuando esto esté en operación, va a funcionar de esta manera"); requiere alta madurez organizacional.
- Antipatrón: el equipo DevOps como un tercer silo aislado genera más problemas en vez de resolverlos. Lo mejor es mantener una sola plantilla de pipeline (no múltiples), y modificarla solo cuando sea necesario (p. ej. si la plataforma de despliegue actualiza algo).

### Anatomía del pipeline

**Integración Continua** (etapa CI):
- Commit: "falla rápido y falla barato" para no arrastrar errores hasta producción.
- Build: se compila antes de correr las pruebas unitarias.
- Prueba unitaria: en Java se requiere el binario para ejecutar; en Python las dependencias se instalan durante el build.
- Calidad de código: herramientas como SonarQube/SonarCloud, que detectan por ejemplo código duplicado.
- Seguridad: análisis de seguridad del código; nunca subir claves/tokens al repositorio (deben ir en el pipeline como secretos); revisar vulnerabilidades de las librerías usadas.
- Salida de esta etapa: un artefacto validado.

**Entrega y despliegue continuo** (etapa CD):
- Publicación del artefacto → provisión de infraestructura → despliegue a preproducción → aprobación → despliegue a producción.
- **Entrega continua (Continuous Delivery)**: el flujo llega hasta una aprobación manual (intervención humana que confirma el despliegue).
- **Despliegue continuo (Continuous Deployment)**: no hay intervención humana; la aprobación está automatizada.

### Cuatro ideas clave de la sesión
- DevSecOps es un proceso, no una herramienta.
- Se trabaja sin muros entre equipos; hay que alinear objetivos.
- El pipeline se construye una vez (y se reutiliza).

---

## Sesión 3: Integración Continua I

### Contexto: Laboratorio 2
- Objetivo de la sesión: entender el problema para saber cómo atacarlo, antes de construir el propio pipeline.

### A. Versionamiento de código

**Qué es realmente la integración continua (CI)**
- En la práctica real, integrar código constantemente es difícil de lograr al 100%.
- Meta: que cada persona integre su código al menos una vez al día.
- Mala práctica: líneas de código duplicadas.
- El costo de integrar tarde: se generan colas de espera y cuellos de botella.

**GitFlow**
- `main`: rama de producción.
- `tag`: nombre de las versiones, asociado a las release notes.
- `hotfix`: cambios urgentes de negocio; se abre una rama desde `main` y se corrige ahí.
- `develop`: rama de larga duración.
- `feature`: cada funcionalidad tiene su propia rama, que se integra a `develop`.
- `release`: rama de preparación final antes de pasar a producción (equivalente a un QA).
- Cherry-pick: copiar un commit específico de una rama a otra sin fusionar todo el historial de ambas ramas.
- GitFlow conviene cuando hay releases programados; mientras más lejos se está de producción, hay más controles y más demoras.

**Trunk-Based Development**
- Requiere alta madurez del equipo; no es fácil de alcanzar.
- Hay una rama por tarea o por bug de una tarea (una feature es más grande que una tarea).
- Feature flag: patrón if/else que permite incorporar código nuevo (aunque le falten pruebas) detrás de una bandera que se activa o desactiva. Ejemplo mencionado: integración con RENIEC.
- El flag habilita o deshabilita funcionalidades nuevas sin necesidad de nuevo despliegue; suele implementarse con variables de entorno.

**Comparación de estrategias**
- GitFlow: orientado a productividad y control.
- GitHub Flow: orientado a un producto que se quiere liberar.
- Trunk-Based: commits continuos.
- Ejemplos: Spotify y Netflix activan/desactivan funcionalidades con feature flags; DataDog es una plataforma de observabilidad.

**Los tres escenarios (estrategia de ramas ↔ métrica DORA)**
- A — Startup con 5 desarrolladores: Trunk-Based Development (requiere pruebas automatizadas); métrica foco: Lead Time para cambios.
- B — Banco peruano: GitFlow; métrica foco: Tasa de fallos en cambios (por tener ventanas de despliegue fijas).
- C — (equipo con una sola rama desplegada, despliegues cada varios días): GitHub Flow; métrica foco: Tasa de fallos en cambios, porque importa más la estabilidad que la velocidad.

**Versionado semántico (SemVer)**
- Formato X.Y.Z (ejemplo de referencia: Minecraft).
- Útil especialmente cuando se consumen librerías; está asociado a las release notes.
- Puede autoasignarse mediante convenciones de commits (p. ej. un commit tipo "feature" incrementa Y).
- Se pueden usar hooks locales para forzar que los commits sigan un patrón determinado.

### B. Todos usamos librerías de la comunidad
- Librerías muy usadas de npm fueron hackeadas; al instalarlas se inyectaba malware (caso real: un estudiante perdió su cuenta de GitHub, afectando sus conexiones).
- GitHub tiene límites de ejecuciones de pipeline al mes.
- Riesgo de seguridad: suplantación de nombre de librerías; NIST cubre el tema de riesgo de terceros.
- Archivo de bloqueo (lock file): fija la versión exacta de cada dependencia para que el pipeline instale siempre lo mismo, por decisión propia y no del proveedor.
- Repositorios de artefactos y proxy interno: se recomienda hacer whitelist de librerías de la comunidad y alojarlas en un repositorio privado local, para no depender de la disponibilidad del servidor público de la librería. Se mencionó la plataforma Sigrid para medir el propio proyecto.

### C. Build

**Construir una vez, desplegar muchas veces**
- El build se "bautiza" con una versión x.y.z; el compilado vive en un único lugar (usar un Secret Manager para credenciales).
- Antipatrón: reconstruir el artefacto en QA y en producción con librerías distintas — por eso es importante el archivo de bloqueo (lock).
- Se busca que lo que se despliega ya haya sido aprobado por todo el proceso.

**Anatomía de un trabajo de construcción (build job)**
- `runs-on`: define el entorno/versión fija donde corre el pipeline.
- Instalación de dependencias desde el archivo de bloqueo.
- Cache: guarda las librerías para no descargarlas de internet en cada ejecución, acelerando la compilación.
- Artefacto identificado con hash (`.sha`) para trazabilidad.

**El tiempo de construcción como métrica de proceso**
- Si el build tarda mucho, los desarrolladores evitan usar el pipeline (lo tienen que correr a diario).
- Meta orientativa: apuntar a ~3 minutos de build.

**Qué mide el análisis automático de calidad (SonarQube)**
- Mide calidad interna del código, no necesariamente lo que espera el usuario final.
- Cobertura: porcentaje del código fuente cubierto por pruebas unitarias.
- Duplicación: se espera un máximo de ~20% de código no cubierto por reutilización; ideal extraer funciones reutilizables en vez de duplicar.
- Complejidad: ciclos computacionales, qué tan complejo es de ejecutar el código.
- Issues detectados.
- Deuda técnica: sugiere qué corregir y estima el tiempo que tomaría.
- La calidad busca detener lo malo; las pruebas de aplicación (que verifican que todo funcione bien end-to-end) no forman parte de este análisis de calidad.

**Quality Gate: del reporte a la condición**
- Ejemplo: si un pull request no cubre el 80% de pruebas, el propio pipeline detiene al desarrollador.
- Automatización de pruebas: en Java, el framework identifica qué partes del código están o no están probadas.

**El umbral que nadie puede pasar termina desactivándose**
- Umbral ejemplo: 80%. Se pueden configurar excepciones.
- La idea del Quality Gate es que esté activo para que el equipo conozca el problema y lo repare, y que a futuro se ajusten los umbrales acercándose a la mejor práctica posible.

**Anatomía de la validación en el pipeline (YAML)**
- `jobs`: se declaran los jobs a ejecutar, por ejemplo `validar` y `desplegar`.
- Dentro de `validar`, los `steps` incluyen:
  - `uses`: para usar una acción/librería externa (por ejemplo, hacer checkout del código trayendo la última versión); pueden ser de la comunidad o propias.
  - Ejecutar pruebas unitarias (`pytest`) y generar reporte.
  - Ejecutar el análisis de calidad (SonarQube).

**SonarQube Cloud — detalles**
- Cobra según líneas de código.
- Coverage: porcentaje de cobertura de pruebas.
- New Code: se evalúa solo el pull request.
- Measures > New Code: lo marcado en rojo es lo que no está cubierto por pruebas.
- También detecta código duplicado por archivo.

### Conclusión de la sesión
**"El código que no escribimos también es nuestro."** (Aplica a librerías de terceros y dependencias: su calidad y seguridad también son responsabilidad del equipo.)
