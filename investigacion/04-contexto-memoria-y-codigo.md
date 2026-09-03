# 04 — Contexto, memoria y comprensión del código

Cubre las áreas 4, 5 y 6 del encargo.

## 4.1 El contexto no es una ventana: es un presupuesto que se degrada

`HECHO` (`E1`, jul-2025) — Chroma, *Context Rot* (Hong, Troynikov, Huber): 18 modelos frontera.
El rendimiento **no decae de forma lineal ni uniforme** con la longitud del input: hay
acantilados. Un modelo puede ir bien a 32K y colapsar a 64K. Los factores que modulan la caída son
la similitud entre la pregunta y el fragmento buscado, la presencia de distractores, y la
estructura del *haystack*. Conclusión de los autores: los benchmarks de contexto largo no
representan las cargas reales.

`HECHO` (`E2`, sep-2025) — Anthropic, *Effective context engineering for AI agents*: las técnicas
para horizontes extendidos son **compactación**, **toma de notas estructurada** y **arquitecturas de
subagentes**, incluida la *cuarentena de subagente* para salidas ruidosas (logs de tests, errores de
compilador). En 2026: la ventana de 1M redujo los eventos de compactación un 15 % en el uso real de
Claude Code.

`HECHO` (`E1`, 2026) — La compactación tiene modos de fallo propios y medidos: la compactación
reactiva llega tarde (el contexto ya lleva muchos pasos contaminado); la periódica corta a ciegas y
resume en mitad de un subobjetivo activo; y resumir bajo presión de longitud es un modo conocido de
alucinación, que introduce errores nuevos justo cuando queda menos presupuesto para detectarlos.

`HECHO` (`E1`, 2026) — Escala real: una sesión de agente sobre tareas tipo SWE-bench recorre
rutinariamente millones de tokens y más de cien turnos (Qwen3-Coder-Next promedia 8M tokens y 154
turnos por problema).

`HECHO` (`E2`, dic-2025) — El patrón de **divulgación progresiva** se estandarizó: Agent Skills
(SKILL.md, estándar abierto desde el 18-dic-2025, adoptado en semanas por OpenAI, Google, GitHub y
Cursor) carga al arranque sólo nombre y descripción (~80 tokens cada uno), las instrucciones
completas al activarse, y los recursos sólo durante la ejecución. El código ejecutable produce
operaciones deterministas **sin cargar su código en el contexto**.

> `INFERENCIA` — La arquitectura correcta para trabajar meses sobre un proyecto no es «una ventana
> más grande» ni «un resumen mejor». Es que **el contexto se derive de una fuente externa
> consultable, en el momento de usarlo, acotado por el trabajo concreto**. Tres propiedades se
> deducen de la evidencia:
> - **Derivado, no acumulado.** Lo que se acumula se contamina; lo que se deriva se recalcula.
> - **Acotado por alcance.** No «aquí tienes todo lo del proyecto» sino «esto es lo que obliga a
>   esta tarea». Es lo único que hace el volumen independiente de la edad del proyecto.
> - **Con procedencia y estado epistémico.** Cada elemento debe decir de dónde salió y si sigue
>   siendo cierto. Sin eso no hay invalidación posible.

## 4.2 Decaimiento de gobierno: la evidencia más importante de este documento

`HECHO` (`E1`, jun-2026) — Chen, *Governance Decay: How Context Compaction Silently Erases Safety
Constraints in Long-Horizon LLM Agents* (arXiv:2606.22528v2). Benchmark **ConstraintRot**: 9 tareas
(5 políticas organizativas «blandas» y 4 normas de seguridad «duras»), 7 familias de modelos, 1.323
episodios. Los resultados:

| Condición | Violación de la política |
|---|---|
| Control: política visible, sin compactación | **0 %** en todos los modelos |
| Tras **una** compactación | **30 %** agregado (DeepSeek-V4 59 %, Kimi 59 %, GPT-5.4-mini 41 %, Qwen 30 %, Claude-Sonnet 19 %, Gemini 4 %, GLM 0 %) |
| Tras **cuatro** compactaciones | **78 %** |
| Presupuesto de resumen de 300 → 15 palabras | supervivencia de la política 88 % → 23 % |

**El mecanismo causal está aislado:** si la restricción sobrevive al resumen, la violación es 0 %
(n = 90); si se elimina, es 38 % (n = 315). No es que el modelo desobedezca: es que la norma
desaparece.

**El hallazgo que más importa a una plataforma de gobierno:** las políticas *organizativas* decaen
+50 puntos porcentuales; las normas *de seguridad* entrenadas en el modelo sólo +6. Un factor de
**8,3×**. El alineamiento entrenado protege lo universal y deja indefenso **exactamente lo
específico del despliegue** —que es todo lo que una organización declara sobre su propio proyecto.

**Y hay un ataque.** Un adversario que sólo controla datos ingeridos (salidas de herramientas,
documentos recuperados) puede forzar el borrado de la norma: por volumen, provocando la
compactación; o inyectando instrucciones dirigidas al propio resumidor. Con inyección optimizada
por modelo: Claude-Sonnet pasa de 0 % a 65 %, GLM-5.1 a 85 %, DeepSeek-V4 a **100 %**.

**La mitigación funciona:** *Constraint Pinning* —las restricciones se aíslan en un buffer exento de
compactación, se reinyectan literalmente tras cada compactación y se comprueba su integridad—
restaura el 0 % de violación en los siete modelos, incluidos los ataques, por ~47 tokens por
restricción (menos del 0,5 % de sobrecoste). La utilidad no sufre: 99 % de acciones permitidas
completadas con 1 % de sobre-rechazo, mejor que la línea base sin defensa (90 %/10 %).

**Y la mitigación tiene un techo declarado por el autor:** una suplantación de operador colocada en
el contexto reciente eleva la violación al 17 % *incluso con pinning*; marcar la procedencia sólo lo
baja al 10 %. La frase del paper: mientras la autoridad se afirme dentro de los tokens, el modelo no
puede distinguir una actualización genuina del operador de una falsificación en contexto.

> `INFERENCIA` — Este resultado, por sí solo, justifica la existencia de una capa de gobierno
> externa al agente. Tres corolarios:
> 1. **La norma debe viajar con el trabajo, en cada entrega, literalmente.** No basta con
>    declararla al principio de la sesión. Es el argumento experimental de que el hallazgo H-06 de
>    Quipu —el payload de `claim_block` no lleva ninguna norma— no es una carencia de comodidad sino
>    de corrección.
> 2. **La verificación no puede vivir en el mismo stream que el trabajo.** Si el guardián comparte
>    contexto con el guardado, la compactación borra a los dos.
> 3. **La autoridad debe ser inverificable desde dentro del contexto.** Una firma en una base de
>    datos, con segregación de funciones, es de otra naturaleza que una instrucción en un prompt: no
>    es falsificable escribiendo texto. Ésta es, en términos de la evidencia disponible, la
>    propiedad más difícil de replicar que puede tener una plataforma.

## 4.3 Memoria: mucho producto, poca evidencia

`HECHO` (`E6`, 2026) — Los tres benchmarks de referencia son LoCoMo (1.540 preguntas), LongMemEval
(500) y BEAM (1M y 10M tokens). Los números **no son comparables**: el mismo sistema (Mem0) aparece
con 49,0 %, 93,4 % y 94,4 % en LongMemEval según la fuente. La explicación que dan los propios
implicados —sensibilidad al LLM subyacente, al reranking y a la versión— es la confesión de que la
métrica no mide el sistema de memoria.

`HECHO` (`E1`, 2026) — Hay un resultado que sí es útil y es negativo: el benchmark **STALE**
(1.200 consultas) mide si un agente detecta que sus creencias almacenadas fueron **invalidadas
silenciosamente** por contexto posterior. El mejor modelo frontera obtiene **55,2 %**.

`HECHO` (`E3`, 2026) — La observación convergente sobre deriva de contexto: rompe en la capa de
**recuperación**, no en la del modelo. El agente razona correctamente sobre premisas incorrectas, y
por eso **falla con forma de éxito**: la definición de la métrica es correcta, sólo que es la del
trimestre pasado.

> `INFERENCIA` — Éste es el argumento más fuerte contra construir «memoria de agente» como capacidad
> de producto, y a favor de construir **invalidación**. Un sistema que recuerda bien y no sabe
> cuándo dejó de ser cierto lo que recuerda es peor que uno que no recuerda: convierte un hueco
> visible en un error invisible. Con un 55,2 % de detección en el mejor modelo, delegar la
> invalidación al juicio del agente no es una opción defendible.
>
> `INFERENCIA` — Lo que sí se deduce que hace falta, y es mucho más barato que una memoria
> semántica: **huella del contenido + propagación de sospecha por dependencia**. Cuando cambia
> aquello de lo que un conocimiento depende, el conocimiento se marca como sospechoso
> mecánicamente, sin que nadie tenga que darse cuenta. Quipu ya tiene ese motor construido y medido
> (H-27) y el mercado de memoria de agentes no tiene nada equivalente.

`INFERENCIA` — Qué debe y qué **no** debe almacenarse, deducido de §4.1-4.3:

| Almacenar | No almacenar |
|---|---|
| Decisiones con su razón y su cadena de reemplazo | Transcripciones de sesión |
| Invariantes con ancla verificable y estado epistémico | Resúmenes generados sin procedencia |
| Evidencia con procedencia (qué comando, qué salida, cuándo) | Conclusiones de un agente sin evidencia que las sostenga |
| Alcance real de cada cambio | Preferencias inferidas del comportamiento |
| Contradicciones abiertas y su resolución | Cualquier cosa cuya obsolescencia no se pueda detectar |

La regla que resume la tabla: **almacena lo que puedas invalidar; no almacenes lo que no puedas.**

## 4.4 Comprensión de bases de código grandes

`HECHO` (`E1`, abr-2026) — De la taxonomía de scaffolds: las cuatro capacidades universales incluyen
`search`, y el debate real es qué alimenta esa búsqueda.

`HECHO` (`E6`, 2026) — El debate «grep agéntico vs índice semántico» se volvió empírico en 2026,
pero casi todos los números son de proveedor: «97 % menos tokens de entrada», «58-70 % menos tool
calls», «10× tokens y 2,1× tool calls en 31 repositorios». Se citan aquí como señal de dirección,
**no como evidencia**.

`HECHO` (`E3`) — Lo que sí es convergente y verificable por razonamiento: **grep gana en consultas
de un salto** (dónde está esto), **el grafo gana en consultas de varios saltos** (radio de impacto,
quién llama a quien llama, qué tests cubren esto, qué flujos se ven afectados). Proyectos de
investigación como RepoGraph y LocAgent construyen el grafo por la misma razón.

> `INFERENCIA` — Para una plataforma de gobierno la conclusión es de alcance limitado y por eso
> útil: **no hace falta un índice semántico del código para gobernar**. Hace falta poder contestar
> tres preguntas, todas de grafo pequeño y ninguna de embeddings:
> 1. ¿Qué rutas y símbolos toca este cambio? (alcance declarado, verificado contra el diff)
> 2. ¿Qué normas están ancladas en esas rutas y símbolos? (intersección)
> 3. ¿Qué otras unidades de trabajo activas intersectan ese alcance? (conflicto)
>
> Las tres se responden con una tabla de enlaces alcance↔artefacto y un `JOIN`. El grafo de llamadas
> completo mejora la calidad de la respuesta 1 pero no cambia la arquitectura. Es un candidato
> claro a *fase posterior* en términos del principio de mínima complejidad.

`INTERPRETACIÓN` — La lección negativa de Quipu es relevante aquí y coincide con la evidencia
externa: modelar la interfaz de usuario como datos (la capa de diseño con ~10 tablas y 0 filas)
fracasó porque los agentes leen el código. La representación que un agente necesita del código no es
una réplica del código en otra forma; es el **conjunto de restricciones que el código debe
respetar**. Eso sí es información que no está en el código.
