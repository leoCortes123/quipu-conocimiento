# 00 — Método, límites y sesgos de esta investigación

## 1. Qué se hizo

Barrido de literatura y práctica sobre las 22 áreas del encargo, priorizando en el orden que el
encargo fijó: papers y resultados experimentales primero, sistemas en producción después,
documentación técnica primaria, arquitecturas publicadas, benchmarks reproducibles, experiencia
open source, postmortems, y sólo al final discusión técnica.

Se contrastó cada afirmación contra el estado real de Quipu Enterprise medido el 2026-09-01 en
`analisisExterno/auditoria-interna-2026-09-01.md`. **No se volvió a auditar Quipu**: los hechos H-01
a H-28 de ese documento se toman como dados, y esta investigación se limita a decir qué dice la
evidencia externa sobre cada uno.

## 2. Qué cuenta como evidencia, y por qué importa la distinción

El campo tiene un problema de calidad de evidencia que hay que nombrar antes de citar nada.

`HECHO` (`E1`) — Hay muy pocos estudios controlados sobre desarrollo con agentes. Los que existen
—el RCT de METR (2025), el benchmark ConstraintRot (2026), la meta-evaluación de jueces LLM (2026)—
son pequeños, recientes, y a menudo **contradicen** las cifras de la industria.

`HECHO` (`E6`) — La mayor parte de los números que circulan sobre agentes son publicados por quien
vende la herramienta medida, sin protocolo, sin línea base y sin grupo de control. Ejemplos
recogidos en esta investigación: «97 % menos tokens», «3-10× más tasa de acierto a la primera»,
«orden de magnitud menos ciclos de regeneración». **Ninguno de esos números sostiene una
recomendación en estos documentos.** Aparecen sólo señalados como lo que son.

`INFERENCIA` — Cuando un número aparece repetido en muchos sitios, casi siempre es la misma fuente
primaria reescrita. Se rastreó el origen en todos los casos en que fue posible; cuando no lo fue, la
afirmación se degradó a `E5`.

### Contradicciones que se dejan a la vista

El encargo pedía explícitamente presentar los resultados contradictorios. Los tres más relevantes:

| Tema | Resultado A | Resultado B |
|---|---|---|
| Productividad | METR RCT 2025 (`E1`): devs experimentados **19 % más lentos** con IA en sus propios repos | Microsoft/Accenture campo 2024-25 (`E2`): **+7,5 % a +21,8 %** PRs/semana |
| Beneficio de multi-agente | Anthropic 2025 (`E2`): **+90,2 %** en su eval de investigación | MAST 2025 (`E1`): ganancias de MAS en benchmarks «a menudo mínimas»; 14 modos de fallo propios |
| Memoria de agentes | Mem0 en LongMemEval reportado como **49,0 %** por un tercero | y como **93,4 %** y **94,4 %** por otras fuentes, mismo benchmark |

`INTERPRETACIÓN` — La contradicción de productividad probablemente no es contradicción: METR midió
personas expertas en repos que ya dominaban; los estudios de campo miden poblaciones heterogéneas
en trabajo mixto. Ambos pueden ser ciertos, y juntos dicen algo más útil que cualquiera por
separado: **el beneficio depende del contexto y de quién trabaja, no de la herramienta.**

`INFERENCIA` — La contradicción de memoria sí es contradicción, y descalifica a los tres números.
Cuando el mismo sistema en el mismo benchmark puntúa 49 % y 94 % según quién publica, la métrica no
mide el sistema; mide la configuración de quien reporta. **Toda cifra de benchmark de memoria de
agentes debe tratarse como `E6` mientras no traiga protocolo.**

## 3. Límites de esta investigación

Hay que declararlos porque afectan a lo que se puede concluir.

1. **Ventana temporal asimétrica.** Los trabajos de 2023-2025 tienen réplicas, críticas y contexto.
   Los de 2026 —que son los más relevantes— son en muchos casos preprints sin revisión por pares y
   sin réplica independiente. Se marcan como tales.
2. **Sesgo de publicación en agentes.** Casi nadie publica el sistema multi-agente que no funcionó.
   La excepción —MAST, que estudia exclusivamente fallos— es por eso desproporcionadamente valiosa.
3. **No se ejecutó nada.** Esta investigación no reprodujo ningún benchmark ni midió ninguna
   herramienta. Todo número es reportado.
4. **Cobertura desigual por área.** Verificación, contexto y seguridad tienen literatura abundante.
   *Requirements engineering con agentes*, *evolución arquitectónica bajo agentes* y *evaluación de
   agentes a escala de proyecto* están casi vacías: hay propuestas, no resultados. Donde esto pasa,
   se dice.
5. **No se cubrió** el estado legal-regulatorio más allá de mencionar el calendario del EU AI Act,
   ni el mercado/competencia (ya cubierto en `analisisExterno/InformeEstratégicoQuipu.md`), ni la
   ingeniería de prompts a nivel táctico, que el encargo excluye implícitamente al pedir principios
   independientes del modelo.

## 4. Lo que esta investigación NO hace

- **No propone un plan.** No hay fases, orden, estimaciones ni tareas. El encargo lo prohíbe
  explícitamente y la razón es buena: las decisiones DEC-1 a DEC-8 de la auditoría siguen abiertas y
  cualquier orden que se escribiera hoy las prejuzgaría.
- **No re-audita Quipu.** Los hechos vienen del dosier del 2026-09-01.
- **No recomienda herramientas.** Cuando nombra una, es como evidencia de que un patrón existe en
  producción, no como sugerencia de adopción.
- **No asume que un gap deba cerrarse.** El documento 10 clasifica varios hallazgos como
  `NO NECESARIO` precisamente porque el encargo pedía el principio de mínima complejidad.

## 5. Cómo se aplicó el principio de mínima complejidad

Cada mecanismo que aparece como candidato en el documento 10 pasó por las siete preguntas del
encargo. En la tabla de comparación se resumen en tres columnas —*qué problema resuelve*,
*evidencia de que el problema es real*, *solución mínima*— y el resto se argumenta en prosa cuando
la respuesta no es obvia. Un mecanismo que no supera la segunda pregunta (evidencia de que el
problema es real **para Quipu**, no en abstracto) se clasifica como `NO NECESARIO` aunque la
industria entera lo esté construyendo.
