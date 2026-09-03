# 03 — Arquitectura de agentes, planificación y coordinación

Cubre las áreas 3, 7, 13 y 14 del encargo.

## 3.1 Single vs multi-agente: la evidencia, no la moda

El encargo pedía explícitamente no asumir que multi-agente es mejor. La evidencia respalda esa
prudencia.

`HECHO` (`E2`, jun-2025) — Anthropic, sistema multi-agente de investigación: un líder Opus 4 con
subagentes Sonnet 4 superó en **+90,2 %** a Opus 4 en solitario sobre su eval interna. En la misma
publicación: los sistemas multi-agente consumen **~15× más tokens** que una interacción de chat, y
**el uso de tokens explica el 80 % de la varianza de rendimiento** en BrowseComp. Conclusión de los
propios autores: el patrón sólo es económicamente viable para tareas de alto valor con
paralelización pesada, información que excede una ventana, y muchas herramientas complejas.

`HECHO` (`E1`, mar-2025 / NeurIPS 2025) — MAST (Cemri et al., arXiv:2503.13657): 150 trazas
anotadas con κ = 0,88, después MAST-Data con más de 1.600 trazas sobre 7 frameworks. **14 modos de
fallo en 3 categorías**: diseño del sistema, desalineación entre agentes, y verificación de tarea.
El fallo más frecuente es **repetición de pasos, 17,14 %**; el segundo, desajuste
razonamiento-acción, 13,98 %. Y la frase que el campo prefiere ignorar: las ganancias de los
sistemas multi-agente en benchmarks populares son «a menudo mínimas».

`HECHO` (`E5`, jun-2025) — Cognition, *Don't Build Multi-Agents*: la decisión distribuida produce
fragilidad porque el contexto de las decisiones no se comparte. El ejemplo del Flappy Bird —dos
subagentes produciendo estilos artísticos incompatibles porque ninguno vio el encuadre original— es
anecdótico pero describe con precisión el modo de fallo que MAST cuantifica.

`HECHO` (`E5`, mediados 2026) — La misma empresa revisó su posición: los sistemas multi-agente sí
funcionan cuando **las escrituras permanecen single-threaded**; los enjambres de escritores
paralelos siguen siendo impracticables.

> `INFERENCIA` — El eje que decide no es «uno vs muchos». Es **si el trabajo puede partirse sin
> partir las decisiones**. Lectura ordenada de los cuatro hechos:
> - Multi-agente gana cuando la subtarea es *lectura*: buscar, explorar, resumir, verificar. Ahí la
>   partición es limpia y el subagente devuelve un resumen (Anthropic).
> - Multi-agente pierde cuando la subtarea es *escritura acoplada*: cada escritor toma decisiones
>   implícitas que los demás necesitan y no ven (Cognition, MAST).
> - Y el precio siempre se paga: ~15× tokens, más superficie de fallo, 14 modos nuevos.

`RECOMENDACIÓN` — Para una plataforma de gobierno, la consecuencia práctica es que la topología de
agentes **no debería ser una decisión de la plataforma**. La plataforma debe hacer posible el
paralelismo seguro (alcance declarado, leases, puertas) y dejar que quien despacha elija. Convertir
un patrón de orquestación concreto en parte del núcleo es apostar a que ese patrón sobrevive, y
Cognition cambió de opinión en doce meses.

## 3.2 Lo que converge en los agentes de código

`HECHO` (`E1`, abr-2026) — Rombaut, *Inside the Scaffold: A Source-Code Taxonomy of Coding Agent
Architectures* (arXiv:2604.03515v2). Analiza el **código fuente** de 13 agentes: OpenCode, Gemini
CLI, Codex CLI, Cline, Aider, OpenHands, SWE-agent, AutoCodeRover, Agentless, Prometheus, Moatless,
DARS-Agent y mini-swe-agent. Es la mejor evidencia disponible sobre qué es realmente común.

**Convergencia** (aparece en todos, inventada de forma independiente):
- Las mismas **cuatro capacidades**: `read`, `search`, `edit`, `execute`. El número de tools va de 0
  a 37, pero las categorías son cuatro en todos.
- Edición por sustitución de cadena (`str_replace`), reinventada por varios equipos sin coordinarse.
- Primitivas de bucle componibles: ReAct, generate-test-repair, plan-execute, reintento múltiple.
- Contenedores Docker en los agentes orientados a benchmark.

**Divergencia total** (no hay estándar):
- **Gestión de contexto**: siete estrategias distintas, desde reventar por overflow hasta compresión
  iniciada por el propio modelo.
- **Aislamiento de seguridad**: fragmentado. Aider delega en supervisión humana; Codex CLI usa
  evaluación por LLM; otros contenedores o políticas de permisos.
- **Representación del estado**: desde listas destructivas hasta *event sourcing* completo
  (OpenHands).
- **Descubrimiento de tools**: registro estático vs. carga dinámica por MCP.

Conclusión del autor: las arquitecturas de scaffold **resisten la clasificación discreta**; son
composiciones de primitivas sobre espectros continuos, no tipos.

`HECHO` (`E1`, 2024-2025) — Agentless demostró que un pipeline simple **localizar → reparar →
validar** supera a muchos agentes completos en SWE-bench con un orden de magnitud menos de coste.

> `INFERENCIA` — Dos consecuencias para una plataforma de gobierno.
> 1. Las cuatro capacidades y el bucle son estables: es seguro construir encima de la abstracción
>    «un agente lee, busca, edita y ejecuta». Es la abstracción más duradera que ofrece esta
>    literatura.
> 2. Lo que diverge —contexto, aislamiento, estado— es precisamente lo que la plataforma **no debe
>    asumir**. Cada proveedor lo resuelve distinto y lo cambia cada seis meses.

## 3.3 Planificación y descomposición

`HECHO` (`E1`, 2025-2026) — La convergencia hacia planificación **jerárquica** está medida:
ReAcTree (arXiv:2511.02424) obtiene 31,00 % de éxito frente al 13,00 % de ReAct plano con el mismo
modelo de 72B sin soporte de memoria. Task-Decoupled Planning (2026) descompone en un **DAG de
subobjetivos y confina la planificación y la replanificación al nodo activo**, lo que contiene la
propagación de errores y reduce la complejidad de tokens. Hay trabajo sobre HTN con heurísticas
generadas por LLM (arXiv:2605.07707, may-2026).

`INTERPRETACIÓN` — El valor de la jerarquía no es la elegancia del árbol; es el **confinamiento
del error**. Cuando la replanificación se limita al nodo activo, un fallo local no reescribe el
plan global. Eso conecta directamente con el hallazgo de HORIZON de que el 72,5 % de los fallos son
de proceso: la estructura del plan es parte del proceso.

`INFERENCIA` — Características que hacen que una tarea sea ejecutable y **verificable
autónomamente** por un agente, deducidas del conjunto de la evidencia:

1. **Criterio de éxito decidible por una máquina** — un comando que sale 0, un test que pasa, un
   esquema que valida. Si el criterio requiere juicio, la tarea no es autónoma; es asistida.
2. **Alcance declarado y acotado** — el conjunto de ficheros/símbolos que puede tocar. Sin esto no
   hay puerta de verificación posible ni propiedad frente a otros agentes.
3. **Contexto suficiente y cerrado** — todo lo que obliga a la tarea debe poder entregarse; si la
   tarea requiere descubrir decisiones ajenas mientras la ejecuta, es la situación del Flappy Bird.
4. **Reversibilidad** — un cambio atómico que se pueda descartar entero.
5. **Independencia de otras tareas activas** — o dependencia explícita y ordenada.
6. **Presupuesto acotado** — de tiempo, de tokens y de reintentos, con salida definida al agotarse.

`INTERPRETACIÓN` — Es notable que estas seis propiedades no dependen del modelo. Serían las mismas
para un becario. Eso es exactamente el tipo de invariante que el encargo pedía identificar.

## 3.4 Git como sustrato de coordinación: ¿sigue bastando?

`HECHO` (`E3`, 2026) — El worktree por agente pasó de técnica avanzada a línea base: cada agente con
su directorio de trabajo e índice propios sobre un object store compartido. Los fallos documentados
cuando no se hace: colisión de escrituras, checkouts que se interrumpen entre sí, lockfiles
corruptos, y colisión de puertos y servicios (el remedio recurrente es un `.env` por worktree).

`HECHO` (`E5`, mediados 2026) — El proyecto público más grande documentado de desarrollo paralelo
con agentes es Grit, la reescritura de Git en Rust de Scott Chacon: ~45.000 millones de tokens
repartidos entre varios agentes.

`HECHO` (`E3`, 2026) — El patrón de coordinación que la práctica ha convergido: **un fichero, un
dueño**; leases con TTL (5 minutos es el valor por defecto que aparece repetido), extensión por
heartbeat y reclamación automática de reclamos expirados; una fuente única de verdad de la que todos
leen y escriben bajo leases controlados.

> `INFERENCIA` — Git es necesario y **no es suficiente**, por una razón precisa: Git es un sistema
> de resolución de conflictos *a posteriori*. Detecta que dos cambios chocaron cuando ya se
> hicieron. Con agentes eso llega tarde: el trabajo ya se pagó en tokens y el conflicto semántico
> —dos agentes que asumieron cosas incompatibles sin tocar la misma línea— Git ni lo ve.
>
> Lo que falta encima de Git no es un reemplazo, son dos primitivas: **alcance declarado *antes* de
> empezar** (para detectar el conflicto en el momento de reclamar, no de fusionar) y **lease con
> caducidad** (para que un agente muerto no bloquee el trabajo). Ambas son filas en una base de
> datos, no una extensión de Git.

`INFERENCIA` — Sobre el conflicto **semántico**, que es el caro: la única defensa que aparece en la
evidencia es que las decisiones que gobiernan a los dos agentes sean las mismas y estén disponibles
para ambos —es la tesis de Cognition— o que exista un tercero que las imponga. Lo primero no escala
(el contexto compartido crece con el número de agentes); lo segundo es la capa de gobierno.

## 3.5 Cómo evitar bucles, conflictos y fallos parciales

`HECHO` (`E1`) — MAST cuantifica que la repetición de pasos es el fallo individual más frecuente
(17,14 %). HORIZON lo lista entre sus siete categorías como acumulación de errores del historial.

`INFERENCIA` — Los mecanismos que aparecen en la evidencia, ordenados por coste:

| Problema | Mecanismo mínimo con respaldo | Grado |
|---|---|---|
| Bucles / repetición | Presupuesto duro de intentos por tarea, con salida a escalamiento; estado de trabajo persistente que registre lo ya intentado | `E3` |
| Conflicto de ficheros | Worktree por agente + alcance declarado + lease con TTL | `E3` |
| Conflicto semántico | Norma común entregada a ambos agentes, o tercero que la impone | `E5`/`E4` |
| Fallos parciales | Ejecución durable con journal por paso (doc 07) | `E2` |
| Coste descontrolado | Fan-out acotado explícito por petición; profundidad de delegación ≤ 2 | `E5` |
| Trabajo duplicado | Fuente única de verdad sobre qué está reclamado | `E3` |
