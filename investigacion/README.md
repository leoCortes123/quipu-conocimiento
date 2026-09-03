# Investigación profunda — Ingeniería de software con agentes de IA

> **Fecha:** 2026-09-01 · **Encargo:** `~/Investigacion_profunda.md`
> **Naturaleza:** investigación. **No es un plan.** No contiene fases, tareas, plazos ni asignaciones,
> y ningún fichero de Quipu Enterprise fue modificado. La toma de decisiones sobre estos hallazgos
> queda para una sesión posterior, tal como pidió el propietario.

---

## Qué es esto

Un barrido del estado del arte —académico e industrial— sobre cómo se construye software cuando
los agentes participan en todo el ciclo de vida, y no sólo en la generación de fragmentos de código.
La pregunta que lo gobierna es la del encargo:

> ¿Cómo debería construirse una infraestructura de ingeniería de software capaz de coordinar humanos
> y agentes de IA de manera confiable, verificable, auditable y sostenible a largo plazo?

El resultado no es un catálogo de herramientas. Es un intento de separar **lo que está demostrado**
de **lo que se afirma**, y de extraer de ahí los principios que probablemente sobrevivan a los
próximos cambios de modelo.

## Cómo leerlo

| # | Documento | Qué contiene |
|---|---|---|
| 0 | [`00-metodo.md`](00-metodo.md) | Cómo se hizo, qué cuenta como evidencia, qué sesgos tiene esta investigación y qué NO cubre |
| 1 | [`01-panorama-y-evidencia.md`](01-panorama-y-evidencia.md) | El estado real del campo en 2026: benchmarks, productividad, calidad, coste. Dónde la evidencia es dura y dónde es marketing |
| 2 | [`02-especificacion-requisitos-y-norma.md`](02-especificacion-requisitos-y-norma.md) | De la necesidad informal a la especificación ejecutable. Spec-driven, PDD, EARS, ambigüedad, contradicción, trazabilidad |
| 3 | [`03-arquitectura-planificacion-y-coordinacion.md`](03-arquitectura-planificacion-y-coordinacion.md) | Single vs multi-agente, planificación jerárquica, descomposición, colaboración, Git como sustrato |
| 4 | [`04-contexto-memoria-y-codigo.md`](04-contexto-memoria-y-codigo.md) | Context engineering, decaimiento de gobierno, memoria, obsolescencia, comprensión de bases de código grandes |
| 5 | [`05-verificacion-evidencia-y-procedencia.md`](05-verificacion-evidencia-y-procedencia.md) | **El núcleo.** Cómo demostrar que el trabajo de un agente es correcto sin confiar en otro LLM |
| 6 | [`06-gobierno-autoridad-y-seguridad.md`](06-gobierno-autoridad-y-seguridad.md) | Human-in-the-loop, autoridad, permisos, sandboxing, protocolos, identidad de agentes |
| 7 | [`07-ejecucion-observabilidad-y-evaluacion.md`](07-ejecucion-observabilidad-y-evaluacion.md) | Ejecución durable, máquinas de estado, trazas, métricas y evaluación continua |
| 8 | [`08-taxonomia-de-fallos.md`](08-taxonomia-de-fallos.md) | Los 27 modos de fallo con causa, síntoma, detección, prevención y recuperación |
| 9 | [`09-largo-plazo-y-abstracciones-duraderas.md`](09-largo-plazo-y-abstracciones-duraderas.md) | Deriva arquitectónica, proyectos de meses, qué partes de una arquitectura agéntica deberían ser estables |
| 10 | [`10-comparacion-con-quipu.md`](10-comparacion-con-quipu.md) | La tabla área por área: estado de Quipu, práctica encontrada, gap, clasificación |
| 11 | [`11-tesis-arquitectonica.md`](11-tesis-arquitectonica.md) | Las 13 preguntas del encargo respondidas, y el conjunto mínimo de cambios que se deduce |
| 12 | [`12-fuentes.md`](12-fuentes.md) | Bibliografía con año y tipo de evidencia de cada fuente |

**Si sólo se van a leer tres:** `01`, `05` y `11`.

## Convención de etiquetas

La misma que ya usa `analisisExterno/auditoria-interna-2026-09-01.md`, extendida con el eje que pedía
el encargo (distinguir investigación experimental de opinión):

| Etiqueta | Significa |
|---|---|
| `HECHO` | Medido y publicado. Trae fuente, año y —cuando existe— tamaño de muestra |
| `INTERPRETACIÓN` | Lectura de un hecho que sus autores no afirman explícitamente |
| `INFERENCIA` | Conclusión derivada de dos o más hechos citados |
| `RECOMENDACIÓN` | Criterio propuesto para Quipu. No verificado. Es opinión razonada, no resultado |

Y el grado de evidencia de cada fuente:

| Grado | Qué es |
|---|---|
| **E1 · experimental** | Estudio controlado, RCT, benchmark reproducible con protocolo publicado |
| **E2 · industrial medido** | Sistema en producción con números publicados y método descrito |
| **E3 · consenso técnico** | Convergencia independiente de varios equipos serios sin medición común |
| **E4 · propuesta teórica** | Formalización o arquitectura publicada, sin validación empírica |
| **E5 · opinión / anécdota** | Postmortem, blog de ingeniería sin datos, experiencia de un equipo |
| **E6 · marketing** | Número publicado por quien vende la cosa medida, sin protocolo |

Cada afirmación importante de estos documentos lleva su grado. Los `E6` aparecen **etiquetados como
tales y nunca sostienen una recomendación**; están porque el encargo pedía no confundir popularidad
con evidencia, y para eso hay que mostrar dónde está la popularidad.

## Las cinco conclusiones que más pesan

1. **La brecha no está en generar, está en verificar.** La capacidad de generación escala con
   cómputo; la de verificación no (`E1`, §5.1). Todo lo demás se deduce de ahí.
2. **La autoridad no puede vivir dentro del stream de tokens.** Está medido: una restricción de
   gobierno que sobrevive al resumen se obedece el 100 % de las veces; una que se pierde se viola el
   38 % (`E1`, §4.2). Y un adversario que sólo controla datos ingeridos puede provocar esa pérdida.
3. **Multi-agente no es una mejora por sí mismo.** Multiplica coste por ~15× y su ganancia depende
   por completo de si el trabajo es partible sin compartir decisiones (`E2`, §3.1).
4. **La velocidad ya no es el problema; la mantenibilidad sí.** Cuatro años de datos de repositorios
   muestran refactor −70 % y duplicación de bloques +81 % (`E2`, §1.3).
5. **Existe una capa que nadie ocupa:** la de gobierno por encima de MCP/A2A. Está descrita como
   hueco en la literatura de julio de 2026 (`E4`, §6.4) y formalizada como propuesta —sin
   implementación ni medición— en mayo de 2026 (`E4`, §2.3). Es el terreno exacto de Quipu.
