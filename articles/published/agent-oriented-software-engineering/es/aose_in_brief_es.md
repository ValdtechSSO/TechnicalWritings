# AOSE en pocas palabras: deja de confiar en tu agente. Empieza a verificarlo.

*Una breve introducción a la ingeniería de software orientada a agentes. El RFC completo estará disponible próximamente.*

---

Hace unos meses, un agente de IA me entregó un informe de implementación de una librería central que le había pedido extraer y empaquetar. El informe decía: **completo**. Tests pasando. Listo para merge.

No lo estaba. Cuando lo revisé personalmente, encontré diferencias de tests que faltaban, ninguna comparación de regresión frente al comportamiento anterior y código que jamás se había auditado. El informe era fluido, seguro de sí mismo y estaba bien estructurado, pero era incorrecto. Si hubiera confiado en él, habría incorporado una base defectuosa en dos productos a la vez.

Ese fallo no fue culpa del modelo. Fue mía. Había construido un workflow en el que «terminado» era algo que el agente *decía*, no algo que el sistema *demostraba*.

Este es el problema central de la ingeniería asistida por agentes en 2026 y la razón por la que escribí un RFC que propone una interpretación de la **ingeniería de software orientada a agentes (AOSE)** para este nuevo contexto. No trata de «cómo construir software usando agentes» —todo el mundo ya lo está haciendo—. Esta propuesta trata de cómo diseñar repositorios, workflows y controles para que el trabajo de los agentes pueda ser *confiable*, con niveles crecientes de autonomía, sin que tú te conviertas en el cuello de botella ni en la víctima.

## Nota sobre el término

No acuñé el término AOSE. Procede de una tradición de investigación consolidada, activa al menos desde el taller AOSE 2000, centrada en analizar, diseñar e implementar sistemas de software cuyos componentes en tiempo de ejecución son agentes autónomos o semiautónomos.

Este RFC toma ese término existente y adapta su acepción a un problema relacionado: cómo diseñar entornos de desarrollo de software en los que las personas colaboran con agentes de IA que programan. No sustituye a la AOSE clásica ni pretende apropiarse de ella; aplica su perspectiva orientada a agentes al contexto, los permisos, los contratos, los workflows y la evidencia que hacen confiable la ingeniería asistida por agentes.

Toda la metodología se resume en cuatro tesis.

---

## Tesis 1: el artefacto principal ya no es solo el código fuente

El código producido por un agente no puede evaluarse de forma aislada. Para juzgarlo, necesitas saber qué contexto recibió el agente, qué permisos tenía, qué contratos debía cumplir, qué comprobaciones ejecutó y qué evidencia produjo.

Eso significa que el *contexto de desarrollo* —archivos de instrucciones, workflows, skills, políticas, clasificaciones de riesgo y plantillas de evidencia— es ahora un artefacto de ingeniería de primer nivel. Vive en el repositorio. Está versionado. Se revisa. Cuando un agente falla, no solo corriges el código; corriges el contexto que permitió que ocurriera ese fallo.

Hoy la mayoría de los repositorios están optimizados para la memoria humana y el conocimiento tribal. Las instrucciones se reparten entre README, wikis, hilos de chat y la cabeza de ingenieros sénior. Un agente puede ser perfectamente capaz de escribir código correcto y, aun así, fallar porque nunca encontró ese comando, invariante o restricción que todos los mantenedores conocen de memoria.

La solución no es «más contexto». Volcarlo todo en un enorme `AGENTS.md` perjudica de forma medible la resolución de las tareas. La solución es el **contexto mínimo suficiente**: un pequeño archivo enrutador que dirige al agente exactamente a los documentos, skills y reglas acotados que necesita para *esta* tarea y nada más.

## Tesis 2: el modelo propone; los sistemas deterministas deciden

Los modelos de lenguaje son excelentes en el juicio semántico: interpretar solicitudes ambiguas, planificar, clasificar fallos y detectar patrones sospechosos. Son malas autoridades para hechos verificables por una máquina: si el build ha pasado, si una migración es reversible o si un despliegue está autorizado.

Por eso hay que dividir el sistema en dos:

- **Plano de razonamiento** (probabilístico): el modelo interpreta, planifica, genera y revisa.
- **Plano de control** (determinista): los esquemas validan, los tests se ejecutan, las políticas autorizan y las compuertas bloquean.

El patrón de ejecución siempre es: **Proponer → Validar → Autorizar → Ejecutar → Observar → Demostrar.**

La salida de un modelo nunca debe provocar directamente un efecto secundario sensible. Si tu prompt dice «nunca despliegues sin aprobación» pero el agente posee credenciales de despliegue, no tienes una política: tienes una sugerencia. La aplicación de la norma debe estar en el límite de la tool, en código, donde no se pueda persuadir para saltársela.

Aprendí este patrón construyendo un asistente conversacional para una plataforma de gestión deportiva. El LLM resuelve «muéstrame las tarjetas rojas de Adrián esta temporada» en una propuesta tipada y estructurada: intención, ID del jugador, ID de la temporada y tipos de evento. Después, código determinista valida el esquema, comprueba que los ID existen, verifica la autorización y ejecuta una consulta conocida. El modelo interpreta el lenguaje. Nunca inventa SQL, nunca salta la autorización y nunca decide que un ID inválido «probablemente está bien». La precisión en la selección de tools pasó del 65 % al 95 % con el mismo modelo, únicamente gracias a esta separación.

## Tesis 3: un buen agente supera a un comité de agentes mediocres

La estética dominante ahora mismo es el pipeline multiagente: Planner → Architect → Backend → Frontend → QA → Reviewer. Parece un organigrama, y ese es precisamente el problema. Cada traspaso pierde información. Cada especialista vuelve a leer el mismo contexto. El coste y la latencia se multiplican, y nadie es responsable de la síntesis final.

El punto de partida de AOSE es **un agente capaz, con buenas tools y contexto mínimo**. Las topologías multiagente son legítimas —planificador-ejecutor cuando un plan necesita aprobación antes de ejecutarse, revisión entre modelos para cambios de alto riesgo, trabajadores paralelos para tareas realmente independientes—, pero cada una debe justificarse por un fallo medido del diseño más sencillo, no adoptarse como una insignia de madurez.

La misma disciplina se aplica a la infraestructura de conocimiento. Empieza con Markdown estructurado y `grep`. Añade un índice generado cuando falle la búsqueda. Añade recuperación semántica cuando fallos de recuperación *medidos* lo justifiquen. No añadas un grafo de conocimiento salvo en casos muy excepcionales. Construir una base de datos vectorial antes de que tus documentos tengan propietarios y fuentes de verdad solo significa recuperar contradicciones obsoletas más rápido.

**El número de agentes es una variable arquitectónica, no una insignia de madurez.** También lo es tu pila de recuperación.

## Tesis 4: terminar es una afirmación que requiere evidencia

Esta es la tesis que me enseñó el incidente que bloqueó el merge, y es el corazón de la metodología.

Los agentes se expresan con fluidez incluso cuando se equivocan. «Todos los tests pasan», «este cambio es seguro», «la implementación está completa»: son afirmaciones de confianza, y la confianza es la forma más débil de evidencia. AOSE las sustituye por artefactos verificables: el comando que se ejecutó, el entorno en el que se ejecutó, el código de salida, el número de tests, la línea base y el resultado del benchmark, y las comprobaciones que se *omitieron* y por qué.

En concreto, toda tarea no trivial genera un **manifiesto de evidencia**: un registro legible por máquina en el que el estado de terminación se *deriva* de las políticas y las comprobaciones; nunca se establece porque el agente lo haya afirmado. Si no se implementó el test de concurrencia, el manifiesto dice `"status": "not-run"` y `"completionClaim": false`, por muy seguro que suene el resumen.

La regla se resume en cuatro palabras: **sin evidencia, no hay terminación.**

Esto se complementa con **límites de confianza** explícitos. La autonomía no es binaria; es una función del riesgo. Un agente que renombra una variable en una rama aislada necesita casi ninguna supervisión. Un agente que toca autenticación, pagos o migraciones destructivas necesita una validación determinista sólida y aprobación humana responsable. El nivel de autonomía permitido es una decisión de política escrita en el repositorio; nunca una suposición que el propio agente hace sobre sí mismo.

---

## Por dónde empezar (sin construir una plataforma)

No necesitas un framework de orquestación de agentes para adoptar esto. La versión mínima viable es:

1. Un `AGENTS.md` conciso que enrute en lugar de aleccionar.
2. Un workflow estándar para funcionalidades y correcciones.
3. Un único comando de validación autoritativo.
4. Una matriz sencilla de riesgo y aprobación: qué puede hacer el agente por sí solo, qué necesita revisión y qué está prohibido.
5. Una plantilla de hoja de trabajo de tarea, para que una tarea interrumpida pueda retomarla otro agente —o tú— sin trabajo de arqueología.
6. Una plantilla de evidencia y la regla de que terminar exige presentarla.
7. Un hábito: cada fallo recurrente se convierte en un test, una tool, una política o una corrección documental.

Es una configuración de un fin de semana y aporta la mayor parte del valor antes de plantearte la orquestación multiagente o la recuperación semántica.

## Lo que esto no es

AOSE no afirma que los agentes sustituyan a los ingenieros, no exige ejecutar múltiples agentes, no requiere una base de datos vectorial y desde luego no es una licencia para desplegar automáticamente en producción. Es lo contrario de los dos extremos del debate actual: ni «no se puede confiar en los agentes» ni «déjalos operar sin supervisión».

El futuro no es software construido por agentes sin supervisión. El futuro es una ingeniería de software en la que los agentes obtienen una autonomía cada vez mayor *porque* su trabajo está limitado por contratos, controlado por políticas, observado mediante trazas y demostrado con evidencia.

El RFC completo cubre las partes que este artículo omite: el modelo de control, el ensamblaje de contexto, las topologías de agentes y cuándo se justifican, la estrategia de pruebas —incluidas las auditorías de falsa confianza—, las superficies de seguridad e inyección de prompts, un modelo de madurez para la adopción progresiva y un catálogo de antipatrones («teatro multiagente», «terminado sin ejecución», «LLM como motor de políticas»).

**Lee la especificación aquí: [enlace]. Es un borrador de RFC —v0.2— y el debate está abierto. Dime en qué se equivoca.**

---

*Adrian Mustelier construye sistemas de producción donde el juicio de los LLM se encuentra con la aplicación determinista: una plataforma de fútbol base, un SaaS legal-tech y un motor de gestión del conocimiento; casi siempre en solitario y con agentes escribiendo gran parte del código.*
