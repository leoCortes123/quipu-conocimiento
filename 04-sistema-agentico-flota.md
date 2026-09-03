# 04 — Sistema de Desarrollo Agéntico Autónomo (FLOTA)

Definición operativa completa para ejecutar el plan de completación de Quipu Enterprise —
y después los proyectos que Quipu gobierne — mediante un equipo de agentes independientes,
con humanos en los puntos que la gobernanza exige y en ningún otro.

Nombre del sistema: **FLOTA** (Flota de agentes Locales Orquestados por Trazabilidad y
Autoridad). Su razón de ser coincide con la de Quipu: máxima autonomía mecánica, autoridad
humana intacta.

---

## 1. Principios rectores

| # | Principio | Consecuencia de diseño |
|---|---|---|
| P1 | **La BD es el jefe, no un LLM** | La orquestación es determinista (Protocol Engine); los LLM proponen, la BD dispone |
| P2 | **Autonomía graduada por evidencia** | Cada agente opera en un nivel L1–L4 por capacidad; subir de nivel lo decide la evidencia acumulada en BD, no la opinión de nadie |
| P3 | **Contexto acotado siempre** | Ningún agente carga el proyecto entero: recibe su microtarea + invariantes de dominio + resumen de checkpoint |
| P4 | **Evidencia tipada o no ocurrió** | Todo trabajo termina en filas (`evidence`), nunca en narrativa de chat |
| P5 | **Los handoffs son contratos, no conversaciones** | Checkpoint estructurado entre sesiones; la mayoría de fallos multi-agente son transferencias de contexto |
| P6 | **Verificación es el cuello de botella legítimo** | La flota nunca genera más rápido de lo que sus puertas verifican; si satura, baja barra *con deuda registrada* |
| P7 | **Un solo agente habla** | El humano ve al Orquestador; los trabajadores son invisibles salvo por su evidencia |

---

## 2. Los roles del equipo

Siete roles, todos agentes independientes (sesiones separadas, contextos separados,
posiblemente modelos distintos según costo). Ninguno sabe hacer el trabajo de otro.

### 2.1 ORQUESTADOR (única interfaz con el humano)
- Recibe intención del usuario, la traduce a operación Quipu (necesidad/cambio/decisión).
- Único rol con permiso de redactar mensajes al usuario; consolida reportes (single response principle).
- No escribe código ni ejecuta tareas técnicas: despacha y sintetiza.
- Modelo: potente (razonamiento). Volumen: bajo.

### 2.2 PLANIFICADOR (planner)
- Descompone un cambio autorizado en bloques y bloques en microtareas con:
  dueño-de-archivos explícito por worker, dependencias, tipo de evidencia exigida,
  presupuesto de tokens, fan-out máximo (default 3).
- Consulta `dominio_contexto` antes de planificar y declara las invariantes aplicables
  dentro del plan (quien ejecuta no busca: recibe).
- Escribe el checkpoint inicial (patrón progress-file).

### 2.3 EJECUTORES (workers generadores, N instancias)
- Un worker por microtarea, en **worktree git propia**, contexto = microtarea + invariantes
  declaradas + checkpoint previo si es continuación.
- Stack configurable por perfil: `db` (SQL/PL-pgSQL), `api` (Laravel), `web` (React),
  `docs`, `infra`. El perfil define herramientas permitidas y modelo.
- Devuelve resumen compacto + evidencia cruda referenciada (paths de test output, diffs).
- Nunca toca archivos fuera de su reparto (verificado por hook, no por honor).

### 2.4 VERIFICADOR (evaluator)
- Audita cada microtarea completada ANTES de permitir avance: corre la suite, aplica
  suite-diff contra baseline, valida evidencia contra el criterio GWT que dice probar.
- Modo Default-FAIL: ante duda, falla. Es el único rol cuyo "no" detiene la línea.
- Detecta deriva (drift): contrasta el estado actual contra el último checkpoint;
  divergencia ⇒ pausa y escala.
- Corre también el pase adversarial (five-pass blunder hunt) sobre diffs de bloque.

### 2.5 GOBERNANTE (governance agent)
- Gestiona el ciclo normativo: propone decisiones, detecta contradicciones
  (`verificar_contradicciones`), redacta supersede, mantiene invariantes con evidencia.
- **No aprueba nada**: prepara actos para firma humana (misma frontera que `block.approve`).

### 2.6 INTEGRADOR (integrator)
- Cuando los workers terminan: fusiona worktrees en orden de dependencias, resuelve
  conflictos triviales, ejecuta la suite completa integrada, prepara el expediente de
  cierre del bloque (puertas 1–6 de `is_block_closeable`) y lo deja en `verifying`.
- Si la fusión requiere decisiones de diseño, escala al Orquestador, no decide solo.

### 2.7 CENTINELA (sentinel, no-LLM donde sea posible)
- Proceso determinista (hooks + scripts): heartbeats, presupuesto BATS (HIGH/MEDIUM/LOW/
  CRITICAL), detección de fan-out excedido, timeouts, registro de drift en JSONL,
  promoción/democión de niveles según reglas.
- Existe porque los controles de confianza no deben depender del modelo que controlan.

---

## 3. Máquina de estados del trabajo (mapeada a Quipu)

Cada unidad de trabajo atraviesa el mismo ciclo, sin importar su tamaño:

```
necesidad ─► cambio(borrador) ─► [contradicciones verificadas] ─► cambio(autorizado·humano)
    ─► bloques(backlog) ─► claim ─► microtareas(worktree) ─► evidencia ─► verificación
    ─► bloque(verifying) ─► FIRMA HUMANA ─► done ─► cascada (contratos, dependientes)
```

Puntos de parada obligatoria del humano (y únicos):
1. Autorización del cambio (borrador→autorizado).
2. Promoción/supersede de decisiones.
3. Firma de cierre de bloque (verifying→done).
4. Cambio de nivel de autonomía de cualquier agente (promoción manual o revisión de democión).

Todo lo demás es autónomo dentro del presupuesto y los gates.

---

## 4. Niveles de autonomía (graduated autonomy aplicado)

Modelo L1–L4 por **capacidad** (no global), alineado con el dial de control de Quipu:

| Nivel | Qué hace el agente sin humano | Ejemplo de capacidad |
|---|---|---|
| L1 Sugerir | observa y recomienda; humano ejecuta | proponer refactor de deuda |
| L2 Redactar | produce el artefacto revisable; humano lo aplica | escribir migración propuesta |
| L3 Ejecutar-aprobado | ejecuta tras aprobación explícita, en sandbox/worktree | correr `/pedido` completo |
| L4 Autónomo-acotado | ejecuta dentro de guardarraíles; humano revisa resultados, no pasos | microtarea estándar con suite verde |

Reglas de promoción (trust tiers):
- **Evidencia cuantitativa, no intuición**: ≥N unidades de trabajo de esa capacidad con
  primera-pass de verificación, tasa de rechazo de evidencia < umbral, cero violaciones de
  reparto-de-archivos, periodo mínimo (30 días o 20 ciclos).
- **Promoción por capacidad**: un worker puede ser L4 en `web` y L2 en `db`.
- **Democión automática e inmediata**: cualquier violación de guardarrail (archivo fuera de
  reparto, evasión de gate, evidencia falsa detectada) ⇒ nivel −1 ese día + incidente en BD.
- La promoción es **una consulta SQL** contra historial de evidencia/firmas del agente
  (member) — diferencial directo: Quipu convierte la confianza en dato auditable.

---

## 5. Presupuestos y límites (BATS adaptado)

| Régimen | Presupuesto consumido | Comportamiento |
|---|---|---|
| HIGH | <60% | normal |
| MEDIUM | 60–80% | compactar contexto agresivo, prohibir exploración libre |
| LOW | 80–95% | sólo acciones dirigidas al cierre de la microtarea |
| CRITICAL | >95% | checkpoint forzado, devolver microtarea a backlog con nota |

Fan-out máx por bloque declarado por el Planificador (default 3). Profundidad de
jerarquía máx 2 (Orquestador→worker; nunca worker-de-worker).

---

## 6. Checkpoints y handoffs (formato contractual)

Entre sesiones/roles viaja exactamente este documento (en BD como `sesion.checkpoint`,
espejo en `.session/progress.json` del worktree):

```json
{
  "cambio": "CHG-0007", "bloque": "B-014", "microtarea": "MT-003",
  "estado": "hecho|parcial|bloqueado",
  "archivos_tocados": ["code/api/database/migrations/..."],
  "decisiones_tomadas": [{"que": "...", "por_que": "...", "cita": "DECISIONES-001"}],
  "evidencia": [{"tipo": "test_output", "ruta": "...", "resultado": "verde"}],
  "abierto": [{"pregunta": "...", "riesgo": "..."}],
  "siguiente_paso": "..."
}
```

Reglas: append-only, atómico, escrito ANTES de terminar la sesión (nunca después);
el Verificador reconstruye estado desde checkpoint+git, jamás desde transcripts.

---

## 7. Manejo de fallos

| Fallo | Respuesta |
|---|---|
| Crash de worker | Centinela detecta heartbeat perdido (>15 min) ⇒ relanza desde checkpoint (resume de sesión), misma worktree |
| Drift (divergencia checkpoint-realidad) | Verificador pausa sesión, escala al Orquestador con diff de supuestos |
| Evidencia rechazada | Microtarea vuelve a `backlog` marcada `rework`; contador alimenta la tasa de promoción/democión |
| Suite rota por refinamiento | Gate suite-diff ⇒ retorno automático a la última microtarea verde (baseline-aware) |
| Saturación de verificación | Centinela activa régimen CRITICAL y propone al humano: estrangular generación o vía rápida con deuda |
| Worker fuera de guardarrail | Democión + incidente + relanza con reparto reforzado |

---

## 8. Ejecución del plan por fases con la flota

| Fase | Autonomía | Nota |
|---|---|---|
| F0 estabilización | L3 (aprobado) | dedup tests y ESTADO.md son trabajo de Ejecutores con perfil docs/api |
| F1 decision-chain | mixto | migraciones/triggers: L2 (humano revisa SQL antes de aplicar); tools MCP: L3; specs OpenSpec: L3 |
| F2 puente Chasqui | L3 | importador corre contra copia de `chasqui_n8n`; dogfooding con humano firmando |
| F3 verificación | L4 tras pilotaje | los gates nuevos los implementa la flota pero los valida el humano en Chasqui real |
| F4/F5 | L3 | SaaS implica credenciales/producción: todo L2-L3, nada L4 |

Primera misión concreta de la flota (calibración): **ejecutar F0 completa** — es pequeña,
tiene verificación binaria (CI verde) y calibra presupuestos, heartbeats y handoffs sin
riesgo. Segunda misión: F1.1 (migraciones decision-chain) en modo L2 para ajustar el flujo
de revisión SQL.

---

## 9. Observabilidad mínima

- Tablero (pantalla Quipu nueva "Flota"): qué worker, qué microtarea, régimen de
  presupuesto, heartbeats, cola de verificación, incidentes, niveles por capacidad.
- Métricas por semana: tasa primera-pass de verificación, retrabajos, drifts detectados,
  presupuesto medio por microtarea, tiempo puerta-verificando→firma.
- Estas métricas alimentan promociones y el marketing posterior ("autonomía con
  expediente").
