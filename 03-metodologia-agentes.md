# 03 — Metodología completa de desarrollo con agentes (2026) aplicada a Quipu

Investigación sobre el estado del arte del ciclo completo —análisis → especificación →
orquestación → implementación → **verificación**→ integración— y cómo se materializa en
Quipu Enterprise. La observación del usuario es correcta: el análisis anterior estaba
centrado en gobernanza y generación; la verificación de resultados y la orquestación
estaban subdesarrolladas. Este documento las cubre.

---

## 1. El mapa completo del ciclo con agentes en 2026

```
┌────────────────────────────────────────────────────────────────────────┐
│ 1 ANÁLISIS      elicitar → necesidad (causa registrada)                │
│ 2 ESPECIFICACIÓN requisitos GWT · decisiones · invariantes             │
│ 3 ORQUESTACIÓN  orchestrator-workers · worktrees · fan-out acotado     │
│ 4 IMPLEMENTACIÓN bloques → microtareas → workers aislados              │
│ 5 VERIFICACIÓN  oracle determinista primero; juez LLM sólo meta-evaluado│
│                 evidencia tipada: tests, mutantes, suite-diff, evals    │
│ 6 INTEGRACIÓN   gates humanos · firmas · trazabilidad auditable        │
└────────────────────────────────────────────────────────────────────────┘
```

Hallazgo central de la investigación: **la mayoría de fallos de sistemas multi-agente son
de orquestación y transferencia de contexto, no del modelo** ("Reliability lives and dies
en los handoffs"). Y en verificación: **la capacidad de generación escala con compute, la
capacidad de verificación no** — ese es el techo de calidad de cualquier equipo de agentes.
Quipu es exactamente infraestructura de handoffs y de verificación: su tesis de producto
coincide con el consenso técnico 2026.

---

## 2. Orquestación multi-agente: patrones validados

### 2.1 Patrones (Anthropic/Claude Code, Microsoft, producción 2026)

| Patrón | Qué es | Cuándo |
|---|---|---|
| **Subagents** (context isolation) | worker en ventana propia, devuelve *resumen*, nunca transcript | trabajo lateral que ensuciaría al principal |
| **Orchestrator-workers** | uno planifica, delega a especialistas, sintetiza; delegación dinámica en runtime | subtasks desconocidos hasta runtime; el patrón empresarial dominante 2026 |
| **Agent teams** | workers con lista de tareas compartida y mensajería directa entre ellos (experimental, Claude Code ≥2.1.32) | entregables interdependientes (API↔web↔tests) |
| **/batch** | cambio mecánico partido en 5–30 worktree-PRs independientes | migraciones mecánicas masivas |

Reglas duras extraídas:
1. **Worktree por worker siempre** que puedan tocar los mismos archivos (el error #1 es
   paralelismo sin worktrees). `/batch` y agent view lo hacen automático.
2. **Aislamiento de contexto es load-bearing**: si el orquestador relee transcripts,
   el patrón se rompe. Workers devuelven resúmenes compactos.
3. **Fan-out acotado**: costo/latencia escala lineal; cap explícito por petición.
4. **Profundidad ≤2**: cada nivel añade latencia y otra frontera de contexto.
5. **Single response principle**: un solo agente habla con el usuario/orquestador.
6. Empezar con un agente; dividir sólo ante necesidad real de modularidad.

### 2.2 Aplicación a Quipu

El modelo de datos ya tiene las unidades correctas — falta amarrarlas a la práctica:

| Concepto Quipu | Rol en orquestación |
|---|---|
| `block` | unidad de despacho paralelo: sus entregables sin dependencias ⇒ N workers en worktrees |
| `task` (microtarea) | unidad de worker: una microtarea = un worktree = un agente especialista |
| `evidence` | el resumen estructurado que el worker devuelve (no transcript: filas tipadas) |
| `contract_lock` | el handoff API↔web: ningún worker de frontend arranca sin contrato congelado — **es la implementación en BD del problema #1 de agent teams** |
| `sospecha` + propagación | el canal entre revisor y descendientes: un hallazgo se propaga mecánicamente a lo que depende |

Skills nuevas que operacionalizan esto (F3 del plan):
- **`/ejecutar-bloque`**: claim → planifica microtareas (declara dueño-de-archivos por
  worker y fan-out máx, default 3) → despacha orchestrator-workers → recolecta evidencia
  → reporta gate. Instrucción fija: workers devuelven resumen; prohibido re-leer transcripts.
- Entregables interdependientes → agent team con partición de archivos escrita en el plan.
- Cambio mecánico masivo → patrón batch: un `cambio` Quipu, N PRs, evidencia consolidada.

---

## 3. Verificación de código generado por IA

### 3.1 Qué dice la evidencia 2026

- Encuesta a 300 profesionales QA: **52% vio aumentar bugs desde que usan IA** (2% disminución).
  Defectos concentrados en errores lógicos y casos borde no manejados — código que pasa
  vista rápida y falla prueba real.
- Los agentes escriben pruebas ceremoniales: cobertura alta, mutantes vivos. MutGen: prompt
  LLM plano = 53% mutation score, estancado tras 4 iteraciones; con feedback de mutantes =
  89.5%. Google (Mutagenesis: ~17M mutantes, 760k cambios), Meta (73% aceptación) y
  Atlassian usan mutation testing como gate de suites generadas.
- Multi-turno refinamiento rompe silenciosamente lo que ya pasaba: correlación
  adherencia-instrucción ↔ corrección-funcional ≈ Φ 0.089. Antídoto: fijar la suite
  original, re-ejecutarla en cada turno, gatear por diff del conjunto de pases.
- Eval-driven development: para salidas no deterministas, definir umbrales y dataset golden
  (~100 casos) **antes** de construir; el umbral explícito es la parte que casi nadie formaliza.
- LLM-as-judge sin meta-evaluación es poco confiable (RuVerBench: balanced accuracy 94.7 →
  51.6 según dominio/modelo/prompt): medir su tasa de error contra labels humanos antes de
  confiar en veredictos.
- Especificación como oracle: derivar inputs de los criterios dado/cuando/entonces,
  ejecutarlos, comparar I/O — juzgar contra ejecución real, no contra razonamiento sobre el
  código (+38pp en defectos de completitud de spec cuando el test-writer recibe la spec
  como reglas enumeradas).
- Los agentes subdeclaran dependencias: validar instalación en entorno limpio.
- Saturación de verificación: sólo tres respuestas legítimas (escalar verificación /
  estrangular generación / bajar barra deliberadamente con registro).

### 3.2 Aplicación a Quipu (Fase 3 del plan)

Lo que Quipu ya tiene bien — y el mercado no tiene:
- Criterios **dado/cuando/entonces como filas** + hábito *"cada escenario cita su test"* ⇒
  la spec ya nace executable-first. Extensión natural: skill de test-writing que recibe el
  criterio como regla enumerada (patrón specification-grounded, +38pp medido).
- `is_block_closeable()` (4 puertas mecánicas) ⇒ pre-completion checklist impuesta por BD.
  Nuevas puertas: **suite_diff** (suite original verde tras cada turno) y criterio-sin-test
  bloqueante salvo evidencia ejecutada.
- `block_required_evidence` (DoD tipada) ⇒ añadir tipos: `mutation_score` (umbral por
  criticidad declarado en plantilla), `eval_report` (sólo para criterios no deterministas,
  con umbral pre-declarado), `env_limpio`.
- Vía rápida con deuda ⇒ la tercera palanca legítima de saturación, con expediente
  obligatorio (`deuda_vencida`). Correcta por diseño; documentarla así.
- Firmas humanas + segregación de funciones ⇒ el humano verifica veredictos de jueces, no
  sólo diffs: lugar natural para la meta-evaluación del juez (registrar su tasa de error).

Regla de oro resultante para el producto:

> **Oracle determinista primero; juez estadístico después; juez sin meta-evaluación, nunca.**
> Un criterio GWT ejecutable vale más que diez párrafos juzgados por otro modelo.

---

## 4. Herramientas del ecosistema: qué adoptar y qué descartar

| Herramienta/categoría | Papel frente a Quipu |
|---|---|
| Spec Kit / OpenSpec | Compatibles, no competidores: Quipu Enterprise usa OpenSpec internamente; importador de specs file-based es vía de adopción (como el de Chasqui) |
| Graphify / codebase-memory-mcp | Capa descriptiva de código externa (grafos, −70–90% tokens por consulta); Quipu gobierna, el grafo describe. Integración por adapter (F3/F4) |
| Graphiti/Zep (memoria temporal) | No prioritario: el estado persistente ya vive en BD transaccional; memoria conversacional es problema distinto |
| Task Master / Taskfolk | Precedentes del hueco "tracker vs markdown"; quedan cubiertos por bloques+tareas+gates de Quipu |
| DeepEval u otro framework EDD | Proveedor de `eval_report`; intercambiable — Quipu exige el reporte, no la librería |
| Stryker/PIT/mutmut | Proveedores de `mutation_score`, según stack del proyecto cliente |
| Kiro | Validación de mercado del espacio spec-driven; diferencial Quipu: agnóstico de agente + enforcement estructural + dial de control |

---

## 5. Conclusión

Con F1+F2+F3, Quipu Enterprise queda como un ciclo completo verificable: análisis con causa
registrada, especificación executable-first, orquestación con aislamiento probado, y una
verificación cuyo rigor es dato (evidencia tipada), no costumbre. Eso — y no la generación
— es donde el consenso 2026 sitúa el cuello de botella, y es el terreno donde Quipu ya
tiene ventaja estructural sobre todo lo analizado.
