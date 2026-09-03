# 05 — Guía de diseño y arquitectura del sistema agéntico

Cómo está construida FLOTA (doc 04), por qué esta arquitectura y no otra, y cómo Quipu
Enterprise la adopta primero como método interno y después como capacidad de producto.

---

## 1. La decisión de arquitectura central

**Quipu ya es el orquestador.** La conclusión más importante del diseño: no se añade un
framework de orquestación encima, porque el motor determinista que los frameworks intentan
dar (estado persistente, transiciones válidas, gates humanos, auditoría) **ya existe en
Postgres con mayor rigor del que cualquier framework ofrece**.

Comparativa con las alternativas serias de 2026:

| Opción | Qué da | Por qué no como núcleo |
|---|---|---|
| LangGraph | grafo de estados + checkpoints + interrupts | duplicaría el Protocol Engine; estado en dos sitios = dos verdades |
| CrewAI / AutoGen | roles conversacionales, prototipado rápido | coordinación por conversación es justo lo que P5 prohíbe; handoffs frágiles |
| OpenAI/Claude Agent SDK como orquestador | primitivas de agente | son capas de agente, no de gobernanza; vendor-lock innecesario para el control plane |
| Devin-style full-autonomous | autonomía total | viola la razón de ser de Quipu (autoridad humana); sin evidencia estructurada |
| **Quipu control plane + harness de agentes CLI intercambiables** | lo mejor de ambos | **elegida**: la BD manda; los agentes son trabajadores desechables |

Advertencia incorporada (error #1 documentado del ecosistema): *over-orchestrating*. Un
agente + tres tools + revisión humana no necesita framework; aquí ni siquiera hace falta
para la flota entera, porque las "graph transitions" ya están en tablas.

### El patrón resultante

```
        HUMANO (Web UI / firma)
           ▲│ autoriza, firma, promueve
           │▼
┌─────────────────────────── CONTROL PLANE ───────────────────────────┐
│  Quipu Enterprise (Laravel + Postgres)                              │
│  · necesidad→cambio→bloque→microtarea   (grafo de estados en BD)    │
│  · triggers/CHECKs = guardarrails       · evidencia/firmas/auditoría │
│  · MCP HTTP+STDIO (41+N tools)          · dial de control por nivel  │
└──────────────┬──────────────────────────────────────────────────────┘
               │ cola de microtareas ready + despacho
┌──────────────▼──────────── DISPATCHER (nuevo, pequeño, PHP) ────────┐
│ polla microtareas → lanza sesiones headless → heartbeats → relanza  │
│ Symfony Process + tabla sesion_agente (checkpoint, régimen BATS)    │
└──────┬───────────────────────┬──────────────────────────────────────┘
       │                       │
┌──────▼─────┐   ┌─────────────▼─────────────┐   ┌──────────────────┐
│ WORKER A   │   │ WORKER B                  │   │ VERIFICADOR      │
│ claude -p  │   │ opencode exec / codex     │   │ sesión dedicada  │
│ worktree-1 │   │ worktree-2                │   │ Default-FAIL     │
│ perfil web │   │ perfil db                 │   │ suite-diff+drift │
└──────┬─────┘   └─────────────┬─────────────┘   └────────┬─────────┘
       │      MCP → Quipu      │                          │
       └───────────────────────┴──────────────────────────┘
              claim / evidencia / transición (mismos gates que REST)
```

Claves:
- **Los workers hablan con Quipu por MCP con su propio token**: pasan por los mismos gates
  que cualquier agente externo. Cero puertas de servicio nuevas.
- **El dispatcher NO usa LLM**: es código (Symfony Process + colas). Los LLM viven dentro
  de las sesiones de trabajo, no en la capa de control.
- **Intercambiabilidad**: cualquier CLI agéntico que ejecute instrucciones y hable MCP
  sirve de worker. La flota no queda casada con un proveedor.

---

## 2. Componentes y contratos

### 2.1 Control plane (existente, extensiones pequeñas)
- Nueva tabla `sesion_agente` (member FK, microtarea FK, worktree_path, heartbeat_at,
  presupuesto_régimen, drift_score, nivel_autonomía por capacidad).
- Nuevas tools MCP de ciclo de sesión: `sesion_iniciar`, `sesion_heartbeat`,
  `sesion_checkpoint`, `sesion_cerrar` — todas auditadas como el resto.
- Nada más: el control plane no sabe qué modelo ni qué CLI corre el worker.

### 2.2 Dispatcher (`php artisan quipu:dispatch`)
1. Polla microtareas `ready` con worker asignable.
2. Crea worktree (`git worktree add`) si no existe.
3. Lanza sesión headless: `claude -p "<instrucción+skill>" --allowedTools ... 
   --session-id <uuid>` (u `opencode run`, o `codex exec` — perfil define el binario),
   con env `QUIPU_AGENT_TOKEN` y MCP configurado a este proyecto.
4. Monitorea: heartbeats (tool `sesion_heartbeat` cada N pasos), timeout duro,
   régimen de presupuesto (tokens consumidos vía hook PostToolUse → Centinela).
5. Crash ⇒ resume desde checkpoint (misma worktree, `--resume`).
6. Al terminar ⇒ notifica Verificador (encola verificación).

Hooks obligatorios de la sesión worker (configurados por el dispatcher):
- `PreToolUse` bloquea escrituras fuera del reparto de archivos declarado.
- `PostToolUse` anexa métricas a `.session/drift.jsonl` (determinista, sobrevive al modelo).

### 2.3 Skills como contratos de comportamiento
Cada rol/capacidad tiene su skill instalada en la imagen del worker:
`/ejecutar-microtarea`, `/verificar`, `/revisar`, `/pedido`, `/diagnostico`.
La skill fija: protocolo de inicio (leer checkpoint, `dominio_contexto`), formato de
evidencia, prohibiciones, cuándo escalar. Las skills versionan en el repo de Quipu —
son producto, no utilidades.

### 2.4 Flujo end-to-end (secuencia canónica)

```
01 humano: intención → Orquestador registra necesidad            (MCP)
02 GOBERNANTE: verificar_contradicciones + dominio_contexto
03 humano: autoriza cambio                                        [PARADA]
04 PLANIFICADOR: bloques + microtareas + reparto + presupuestos
05 DISPATCHER: workers headless en worktrees (fan-out ≤ cap)      (autónomo)
06 WORKERS: implementan, evidencian, heartbeatean                 (autónomo)
07 VERIFICADOR: suite + suite-diff + criterio↔evidencia + drift   (autónomo, FAIL-closed)
08 INTEGRADOR: fusión ordenada + suite integrada + expediente     (autónomo)
09 humano: firma verifying→done                                   [PARADA]
10 cascada BD: contratos congelados, dependientes ready
11 GOBERNANTE: si cambió arquitectura → propuesta de decisión     [PARADA: firma]
```

---

## 3. Por qué esto escala donde otros sistemas fallan

| Problema documentado 2026 | Respuesta FLOTA |
|---|---|
| Cascadas de fallo en sesiones largas (65% de fallos empresariales: contexto/memoria) | unidades pequeñas (microtarea), checkpoint-resume, sin sesión larga jamás |
| Handoffs frágiles entre agentes | handoff = fila contractual en BD, verificable |
| Over-orchestration / frameworks duplicando estado | cero framework: el grafo vive en una sola verdad |
| Confianza binaria (todo-o-nada) | L1–L4 por capacidad con promoción por evidencia SQL |
| Jueces LLM poco confiables | Verificador con oracle determinista; LLM sólo propone, BD dispone |
| Vendor lock | workers intercambiables; control plane sin dependencia de modelo |
| Costo fuera de control | presupuestos BATS por microtarea, fan-out acotado, perfiles con modelo barato para tareas mecánicas |

---

## 4. Adopción por Quipu como producto (capability `agent-orchestration`)

Ruta en tres pasos, coherente con el plan de fases:

1. **Fase interna (F0–F2 del plan)**: FLOTA opera manualmente — el dispatcher es el
   humano lanzando comandos; las skills ya imponen el protocolo. Se aprende con Chasqui.
2. **Dispatcher MVP** (tras F3): `quipu:dispatch` en PHP puro + `sesion_agente`. Sigue
   local (docker compose existente). Pantalla Web "Flota" (doc 04 §9). Capability nueva
   en openspec con sus requirements/scenarios citando tests.
3. **Producto multi-flota** (con SaaS/F5): flotas por tenant, plantillas de equipo
   (roles/perfiles/niveles preconfigurados), marketplace de skills. El diferencial comercial
   se mantiene: *autonomía con expediente* — nadie más puede demostrar con filas de BD por
   qué su flota merece L4.

Compatibilidad estratégica: MCP como único contrato de herramientas (spec final 2026-07-28
sin acciones disruptivas para este diseño) y A2A observado pero no requerido — los agentes
de FLOTA no conversan entre sí, reportan a la BD.

---

## 5. Riesgos de arquitectura y mitigaciones

| Riesgo | Mitigación |
|---|---|
| Dispatcher mal escrito se convierte en segundo cerebro | mantenerlo tonto: polla+lanzar+medir; toda decisión pasa por gates BD |
| Hooks bypasseables por el modelo | Centinela audita diffs post-sesión contra reparto declarado; violación = democión automática |
| Worktree drift con el trunk largo | rebases automáticos del Integrador antes de fusionar; bloque pequeño ⇒ ventana corta |
| Costo de verificación > costo de generación | paralelizar Verificadores (lectura pura, modelos medios); régimen CRITICAL estrangula generación antes que la puerta |
| Acoplamiento a CLI específica | contrato de worker mínimo documentado (headless, hooks, MCP env); perfiles aíslan el binario |

## 6. Primera implementación concreta (orden sugerido)

1. Tabla `sesion_agente` + tools de sesión (detrás de feature flag off).
2. Skill `/ejecutar-microtarea` + `/verificar` versionadas en repo.
3. Dispatcher MVP lanzando UN worker real sobre F0 (dedup de tests) en modo L3.
4. Medir una semana: primera-pass rate, presupuesto medio, drifts. Ajustar umbrales.
5. Recién entonces paralelizar (fan-out 2–3) y activar Verificador autónomo completo.
