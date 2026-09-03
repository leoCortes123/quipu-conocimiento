# 02 — Plan por fases para completar Quipu Enterprise

Prioridad absoluta: **llegar a gestionar chasqui_n8n con Quipu lo antes posible**
(F0→F1→F2 es la ruta crítica; todo lo demás puede ir en paralelo o después).

Cada fase sigue el propio flujo del repo: `/opsx:propose` → delta specs → implementar →
CI verde → evidencia → `/opsx:sync` → `/opsx:archive`. Cada tarea cita su verificación.

---

## Mapa de fases

```
F0 Estabilizar ──► F1 decision-chain ──► F2 Puente Chasqui ──► DOGFOODING (Chasqui vive en Quipu)
       │                  │                                            │
       │                  ├──► F3 Verificación+Orquestación ◄───────────┤ (se endurece usando Chasqui real)
       │                  └──► F4 Dial de control + Skills metodología ─┘
       └────────────────────────────► F5 SaaS (multi-tenant, OAuth, billing)
```

F3 y F4 dependen sólo de F1; pueden solaparse con el dogfooding. F5 no bloquea nada anterior.

---

## Fase 0 — Estabilización e inventario (~3–5 días)

**Objetivo:** saber exactamente qué hay y dejar el repo sin ambigüedades antes de crecer.

| # | Tarea | Verificación |
|---|---|---|
| 0.1 | `docker compose up -d` + `composer ci` + `pnpm run ci` — dejar CI verde y medir tiempo total | suites en verde |
| 0.2 | Deduplicar tests: contrastar `AdopcionTest` vs `AdoptionTest`, `AuthTokenTest` vs `AgentTokenTest`; fusionar o documentar por qué coexisten | una sola suite por tema |
| 0.3 | Commit que declara punto cero del flujo OpenSpec (base aplanada; el historial delta arranca acá) — resuelve el archivo vacío | commit + nota en CLAUDE.md |
| 0.4 | Crear `ESTADO.md`: capabilities completas / a medias / heredado-congelado (`project_template`, `knowledge_entry`, `changelog_*`, `screen_component*`, `entity_asset`) / eliminado | revisión humana |
| 0.5 | Formalizar convención de evidencia en repo (`code/web/evidencia/`) o decidir migrarla a BD en F3 | nota en CLAUDE.md |
| 0.6 | Congelar QUIPU v1: verificar que Enterprise cubre sus funciones vivas y anunciar freeze en el README de v1 | README v1 actualizado |

**Criterio de salida:** CI verde, `ESTADO.md` completo, cero suites duplicadas, política de repos escrita (ver doc 01 §3).

---

## Fase 1 — Capability `decision-chain` (~2–3 semanas) ⭐ crítica

**Objetivo:** portar la mitad normativa de Chasqui al modelo de Quipu: decisiones con grafo
supersede, invariantes con evidencia, dominios, y regla de contradicción impuesta por la BD.
Es la capability que ninguna herramienta del mercado tiene y la que Chasqui necesita para
migrar su metodología.

### 1.1 Esquema de datos (migraciones)

```
dominio            (lookup: slug, nombre, descripcion)
decision           (id estable tipo DECISIONES-001 · dominio FK · estado vigente|superada|descartada
                    · titulo · cuerpo TEXT [sólo prosa para humanos] · fecha)
decision_supersede (predecesora FK · sucesora FK · motivo_reemplazo)   ← arista del grafo
invariante         (dominio FK · enunciado · evidencia ruta:simbolo NOT NULL
                    · etiqueta confirmado|inferido · decision FK)
contradiccion      (cambio FK · invariante FK · explicacion · resolucion pendiente|aceptada|revocada)
```

Reglas como constraints/triggers (estilo del repo):

| Regla | Enforcement |
|---|---|
| Una decisión `superada` exige arista `supersede` saliente + `motivo_reemplazo` | CHECK + trigger `fn_decision_superada` |
| Sin ciclos de supersede | trigger con CTE recursivo `fn_supersede_sin_ciclos` |
| Invariante `confirmado` sin evidencia | CHECK rechaza la fila |
| Invariante referida por un requisito/criterio debe estar `vigente` (su decisión no superada) | trigger `fn_invariante_vigente` |
| Cambio que toca un dominio con contradicción `pendiente` no avanza de estado | integración con `fn_cambio_transicion` |

### 1.2 Tools MCP nuevas (6)

| Tool | Qué hace | Escritura |
|---|---|---|
| `dominio_contexto(dominio)` | EL PRIMER MOVIMIENTO: decisiones vigentes+superadas relevantes, invariantes con evidencia, relacionadas de otros dominios | lectura |
| `decision_leer(id)` | decisión completa con historia supersede | lectura |
| `decision_proponer(...)` | agente propone decisión nueva o supersede; **queda `propuesta` hasta firma humana** (patrón `block.approve`) | gated |
| `invariantes_de(archivo\|funcion\|tabla)` | qué invariantes gobiernan este artefacto (JOIN contra code_index) | lectura |
| `registrar_contradiccion(cambio, invariante, explicacion)` | abre contradicción; congela el cambio | gated |
| `verificar_contradicciones(cambio)` | reporte previo obligatorio: cruza el alcance declarado del cambio contra invariantes vigentes | lectura |

Invariante innegociable: **ningún tool aprueba decisiones ni resuelve contradicciones**.
Hay un test que lo vigila (patrón `McpToolsTest`).

### 1.3 Integración con demanda existente

- `necesidad`/`cambio` ganan columna `dominio` (o tabla puente si un cambio toca varios).
- El gate `fn_cambio_autorizado` pasa a exigir: contradicciones verificadas (=0 pendientes)
  cuando el dominio tiene invariantes.
- Los criterios dado/cuando/entonces pueden citar `invariante.id` (FK opcional): conecta
  trazabilidad demanda↔normativa.

### 1.4 Delta OpenSpec

Nueva capability `openspec/specs/decision-chain/spec.md` con requirements estilo:
*"Una decisión superada tiene siempre sucesor y motivo"*, *"Sin ciclos de supersede"*,
*"Sólo humanos promueven decisiones"*, *"Un cambio con contradicción pendiente no se autoriza"*,
cada scenario citando su test.

### 1.5 Tests

`DecisionChainTest` (grafo, ciclos, estados), `ContradiccionGateTest` (el cambio se congela),
`DecisionMcpTest` (sin tool de aprobación), `DominioContextoTest`.

**Criterios de aceptación (GWT):**
- Dado un intento de marcar `superada` sin sucesor, cuando se inserta, entonces Postgres rechaza.
- Dado A→B→A en supersede, cuando se crea la última arista, entonces el trigger rechaza por ciclo.
- Dado un cambio sobre dominio con invariante vigente contradictorio, cuando se intenta autorizar, entonces 422 nombrando la contradicción.
- Dado el listado de tools MCP, cuando se busca una que promueva/apruebe decisiones, entonces no existe (test rojo si aparece).

---

## Fase 2 — Puente Chasqui (~1–2 semanas) ⭐ crítica

**Objetivo:** que chasqui_n8n quede gestionado por Quipu: su metodología importada, su
próximo cambio corrido end-to-end desde aquí.

### 2.1 Importador `quipu:importar-chasqui` (comando artisan)

Mapeo verificado contra los formatos reales de ambos proyectos:

| Fuente en chasqui_n8n | Destino en Quipu |
|---|---|
| `decisiones/*.md` (frontmatter id/dominio/estado/supersede/superseded_by/motivo/invariantes/afecta/implementada_en) | `decision` + `decision_supersede` + `invariante` (parser frontmatter ya existe en `bin/mcp_decisiones.py`, se reescribe en PHP) |
| R-I..R-IV de `AGENTS.md` | `business_rule` + `invariante` con evidencia a las secciones del archivo |
| `pedidos/NNN-slug.md` abiertos | `cambio` en estado equivalente (ver 2.2) |
| `pedidos/archivo/*` aplicados | `cambio` cerrados, históricos |
| `db/pruebas/`, `bin/verificar.sh` chequeos | referencia como `required_evidence` de los cambios futuros |
| `agent-context/` | **no importa** (capa descriptiva; sigue siendo del proyecto) |

Idempotente y transaccional (patrón `ImportProjectStructure`). Validador post-importación
al estilo `quipu:validate-vault`: decisiones sin dominio, aristas huérfanas, pedidos sin
estado válido.

### 2.2 Mapeo de máquina de estados pedido↔cambio

```
pedido propuesto   → cambio borrador        (agente propone)
pedido aprobado    → cambio autorizado      (humano: misma frontera de autoridad)
trabajando         → cambio en_construccion
verificar.sh verde → cambio en_validacion
aplicado           → cambio cerrado         (con evidencia adjunta)
rechazado          → cambio rechazado
```

Nota: Chasqui exige *"la decisión se escribe primero"* ante contradicción — eso ya queda
impuesto por F1 (`fn_cambio_transicion` + contradicciones), más fuerte que el texto del AGENTS.md.

### 2.3 Export determinista a markdown

`quipu:exportar --proyecto=chasqui` regenera `decisiones/*.md` y el índice byte-a-byte
(patrón `db/actual/` de Chasqui). La BD manda; git sigue mostrando diffs legibles. Permite
convivencia con herramientas file-based durante la transición.

### 2.4 Skills mínimas para agentes (instalables vía `.agents/skills/`)

- **`/pedido`**: protocolo obligatorio → `dominio_contexto` → `verificar_contradicciones`
  → proponer cambio → esperar autorización humana. Reproduce el orden intención→decisiones→
  realidad→impacto de Chasqui, ahora respaldado por gates.
- **`/diagnostico`**: brownfield con las tools existentes (`GetAnalysisFields`→`NextQuestion`→
  `RecommendEntryPath`→`RecordAnalysis`). Ya están implementadas; falta el skill que guía.

**Criterio de salida (dogfooding):** el siguiente cambio real de Chasqui (uno de los 9
pedidos abiertos, p.ej. `008-ningun-negocio-tiene-dueno`) se propone, autoriza, ejecuta,
verifica y cierra **desde Quipu**, con su decisión supersede si aplica, y el diff de
`quipu:exportar` coincide con lo que hoy habría escrito la skill file-based.

---

## Fase 3 — Verificación y orquestación según el estado del arte 2026

Fundamento completo en el doc 03. Resumen operativo:

### 3.1 Evidencia más fuerte (extiende `block_required_evidence`)

| Nuevo tipo de evidencia | Qué prueba | Herramienta |
|---|---|---|
| `mutation_score` | que los tests generados detectan fallas reales, no sólo cubren líneas (Meta/Google/Atlassian lo usan como gate; MutGen: 53%→89.5% con feedback de mutantes) | Stryker/PIT/mutmut según stack del proyecto cliente |
| `suite_diff` | que la suite original no se rompió tras cada refinamiento (baseline-aware: correr suite antes/después y gate por diff, no por pase absoluto) | patrón Phoenix, gate en `is_block_closeable` |
| `eval_report` | criterios no deterministas (tono, formato LLM) con umbral explícito pre-declarado (EDD: evals antes de construir; dataset golden ~100 casos) | DeepEval u otro; el *reporte* entra como evidencia |
| `env_limpio` | que el código declara todas sus dependencias (los agentes subdeclaran; validar instalación limpia) | build en contenedor fresco |

La Definición de Hecho por plantilla gana estos tipos; `has_required_evidence()` los exige.

### 3.2 Gates nuevos en el cierre

- **Re-ejecutar suite completa** en cada turno de refinamiento antes de permitir `verifying`
  (los agentes rompen lo que ya pasaba silenciosamente; Phi(adherencia, corrección)≈0.09).
- **Pre-completion checklist mecánica**: ya existe (`is_block_closeable`, 4 puertas); añadir
  puerta 5: suite_diff verde, puerta 6: criterios citan test o evidencia ejecutada.
- **Lenguaje-juez sólo meta-evaluado**: si un criterio se valida con LLM-as-judge, exigir
  registrar su tasa de error contra labels humanos antes de confiar (RuVerBench: 94.7→51.6
  según dominio). Sin meta-evaluación registrada, el criterio debe tener oracle determinista
  (spec-derived execution: derivar inputs del criterio GWT y comparar I/O).

### 3.3 Orquestación (skills + convenciones)

- **Bloque = unidad paralelizable**: entregables sin dependencias entre sí ⇒ workers en
  worktrees separadas (mecanismo estándar: subagents/`/batch`/agent teams todos usan worktrees).
- Skill **`/ejecutar-bloque`**: claim → plan de microtareas → despacho orchestrator-workers
  (planifica, delega, sintetiza; workers devuelven resumen, nunca transcripts) → evidencia
  por microtarea → gate. Fan-out cap declarado por bloque (columna nueva, default 3).
- Skill **`/revisar`**: pase adversarial (five-pass blunder hunt) sobre el diff del bloque
  antes de `verifying`; hallazgos entran como sospechas de Quipu (¡ya tienen propagación por
  triggers!).
- Entregables interdependientes (API+web+tests): agent team con reparto de archivos
  explícito en el plan de microtareas (evitar conflictos: cada worker dueño de su conjunto).
- Cambios mecánicos masivos (renombrar en 40 archivos): patrón `/batch` — un cambio Quipu,
  N worktree-PRs, evidencia consolidada.

### 3.4 Tesis de producto que esto confirma

*"Verification capacity is the quality ceiling"* (la generación escala con compute, la
verificación no). Quipu es literalmente infraestructura de capacidad de verificación: gates,
evidencia tipada, firmas humanas. La saturación del gate tiene tres palancas legítimas
(escalar verificación, estrangular generación, bajar barra deliberadamente) — la vía rápida
con deuda de análisis ya implementa la tercera **con expediente obligatorio**, que es justo
la versión correcta. Documentarlo como posicionamiento.

---

## Fase 4 — Dial de control + metodología empaquetada (~2 semanas)

**Objetivo:** el nivel de control lo elige el usuario por proyecto; la metodología viaja en
plantillas+skills, no en conversaciones.

- `nivel_control` por proyecto: `estricto` (cadena demanda completa + decisiones +
  contradicciones) / `equilibrado` (bloques+evidencia, decisiones sólo si cambia
  arquitectura) / `liviano` (sólo decisiones e invariantes; bloques opcionales).
  Implementación: filas en `entry_path_rule`/settings que activan-desactivan gates por
  trigger (la BD sigue mandando: el nivel es dato, no convención).
- Plantillas de bootstrap actualizadas: cada template trae su DoD con tipos de evidencia,
  sus skills recomendadas y su nivel sugerido.
- Wizard de onboarding self-service (web): alta proyecto → diagnóstico → nivel → skills
  emitidas → comando `claude mcp add` copiable.
- Docs de producto: guía "de cero a primer bloque cerrado con tu agente" (existe base en
  QUIPU v1 `docs/ONBOARDING.md`, portarla y actualizar).

---

## Fase 5 — SaaS (~6–10 semanas después de F2/F4 estables)

Arquitectura híbrida (detalle en conversación previa; resumen contractual):

| Pieza | Decisión |
|---|---|
| Tenancy | `tenant_id` + RLS Postgres (coherente: el aislamiento también es gate en BD) |
| MCP remoto | HTTP con OAuth 2.1 + PKCE; STDIO queda para self-hosted con token Sanctum |
| Datos | Sólo metadatos en nube (specs/estados/evidencia-referencia). Código nunca sale; índice de código opcional-local |
| Conector local | docker-compose (el actual) autenticado contra el control plane; export markdown → git del usuario |
| Billing | Stripe; free self-hosted / Solo ~$15 / Team ~$25-editor / Business auditoría+SSO ~$50 / Enterprise on-prem |
| Cumplimiento MCP | sin token passthrough, vault cifrado por tenant, tests anti-fuga cross-tenant en CI |

Riesgos principales: seguridad multi-tenant (referencia de mercado $60–120k hacerlo de
cero; aquí el motor existe, el costo es tenancy+OAuth+billing), absorción por vendors de IDE,
y foco del fundador. Mitigaciones: open-core para comunidad, diferencial estructural
(enforcement vs convención) difícil de copiar encima de archivos md, dogfooding como fuente
de roadmap.

---

## Orden de ejecución recomendado dado tu tiempo

1. **F0 completa** (días, desbloquea todo y te da el mapa que dices no tener).
2. **F1** delegada a agente con OpenSpec flow del repo (el repo se desarrolla solo:
   propose→tasks→tests→CI). Revisión humana en gates.
3. **F2.1–2.2** (importador+mapeo) — otro agente, rama aparte.
4. **Dogfooding tú mismo**: primer cambio real de Chasqui vía Quipu (valida todo y produce
   el caso de estudio).
5. F3/F4 en paralelo con agentes mientras usas el sistema; **F5 sólo con tracción**
   (primero 5–10 usuarios externos self-hosted).
