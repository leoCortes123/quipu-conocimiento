# 01 — Auditoría del repositorio QUIPU_ENTERPRISE

Fecha: 2026-08-23. Alcance: `/mnt/datos/Programacion/QUIPU_ENTERPRISE` (sólo lectura).

## 1. Estado real verificado

| Aspecto | Hallazgo |
|---|---|
| Historial git | 7 commits. `Base heredada de Quipu v1` + Fase 1 de cadena de demanda (F1-B10..B15) |
| Tools MCP | 41 implementadas en `code/api/app/Mcp/Tools/` (incluye demanda completa: `RegistrarNecesidad`, `CrearCambio`, `DeclararRequisitos`, `EnlazarCobertura`, `TriarNecesidad`, `ListarSospechas`, `ProponerRevalidacion`, `NextQuestion`, …) |
| Migraciones | Cadena completa al 2026-07-21: necesidad, cambio, requisito, enlace+propagación sospecha, firma+firmante_humano+segregación_funciones, línea base, vía rápida, retrofit |
| Tests | 48 suites en `tests/Feature/`, 0 en `Unit/`. Cobertura de cada capability (CambioGate, Sospecha, Firma, ViaRapida, Retrofit, Traza, Adopcion…) |
| Web | ~25 pantallas: Dashboard, Projects, BlockHub, FeatureHub, Cambios, Necesidades, Sospechas, Traza, Trace, Verification, Locks, BootstrapWizard, Team, Knowledge, Map, Schema… con tests vitest en las críticas |
| Specs OpenSpec | 4 capacidades (`block-workflow`, `traceability`, `demand-chain`, `mcp-interface`), formato Requirement/Scenario estricto |
| Deuda técnica explícita | Casi nula: **ni un TODO/FIXME/HACK** en `app/` ni `src/` |

Conclusión: el repo está **sano y avanzado**. La limpieza necesaria es menor y puntual.

## 2. Qué limpiar (hallazgos concretos)

### 2.1 `openspec/changes/archive/` vacío con 4 capabilities publicadas

El propio CLAUDE.md exige *"ningún change se archiva sin su delta sincronizado"*, pero el
archivo de cambios está vacío mientras las specs ya están sincronizadas. El flujo se usó
sin dejar historia (la base vino aplanada de Quipu v1). Consecuencia: no hay trazabilidad
de cómo cada requirement llegó a su texto.

**Acción (F0):** decidir una de dos y registrarlo:
- **A (recomendada):** aceptar la base aplanada como punto cero explícito — un commit que
  documente "las 4 capabilities existen; el historial delta arranca desde acá" — y aplicar
  el flujo estricto desde entonces.
- B: reconstruir deltas retroactivos (costoso, poco valor).

### 2.2 Suites de test posiblemente duplicadas

- `AdopcionTest.php` (319 líneas) **y** `AdoptionTest.php` (354 líneas): ambas prueban la
  adopción/importación. Verificar si una es MCP y otra REST o si hay solapamiento real;
  fusionar lo repetido.
- `AuthTokenTest.php` **y** `AgentTokenTest.php`: ídem para tokens.

**Acción (F0):** inventariar y deduplicar. No es urgente pero contamina la lectura del repo
y puede duplicar tiempo de CI.

### 2.3 Funcionalidad heredada de Quipu v1 sin destino definido

La base heredada incluye capacidades que el plan SaaS no usa hoy tal cual:
`project_template` (templates de bootstrap), `knowledge_entry` (conocimiento por proyecto),
`changelog_entry/block`, `screen_component` completo (catálogo de UI), `entity_asset`.

No son basura — varias son parte del producto — pero **no tienen spec propia en
openspec/specs/** (las 4 capabilities no las cubren todas).

**Acción (F0/F4):** inventario en `ESTADO.md`: qué heredado queda soportado, qué queda
congelado sin specs, qué se elimina. Regla: nada muerto dentro del repo.

### 2.4 Convención de evidencia semi-formal

Existe `code/web/evidencia/f1-b12/traza-CHG-0003.json`: evidencia real guardada en el repo,
citada por los tests. Es un buen hábito sin convención escrita.

**Acción (F0):** formalizar ruta y formato (`evidencia/<bloque>/<criterio>.json`) o moverla
a Quipu cuando la evidencia viva en BD (F3).

### 2.5 Sin documento de estado ni roadmap

README dice "cuatro capacidades hoy" y nada más sobre dirección. Para un proyecto que va a
absorber otro y luego comercializarse, falta el mapa.

**Acción (F0):** crear `ESTADO.md` (capabilities completas / a medias / congeladas) y
referenciar el plan de este repositorio.

## 3. Política entre los tres repos

| Repositorio | Rol desde ahora |
|---|---|
| `QUIPU_ENTERPRISE` | **Único activo.** Recibe todas las fases del plan |
| `QUIPU` (v1) | **Congelar** tras verificar que Enterprise cubre sus funciones vivas (el README de v1 ya deriva en Enterprise). No borrar: referencia histórica y fuente de docs aún válidas (`docs/spec/*`) |
| `CHASQUIxQUIPU` | Producto independiente (asistentes Telegram). Sin relación con este plan; no tocar |
| `chasqui_n8n` | Proyecto *cliente*: primero se gestiona con Quipu (F2), después se migra su metodología |

Riesgo detectado: mantener dos generadores de producto vivos (QUIPU y ENTERPRISE) garantiza
desalineación de docs. El congelamiento de v1 es la limpieza más importante de todas.

## 4. Lo que NO se toca (correcto y verificable)

- Motor PL/pgSQL y triggers: coherentes con las specs (verificado contra
  `demand-chain/spec.md`: `fn_cambio_transicion`, `precinto_requisito`,
  `coherencia_satisface`, `fn_firmante_humano`, `fn_propagar_sospecha`, `deuda_vencida`).
- Invariante "no existe tool de aprobación" con test que lo vigila.
- Errores 422/403 con mensaje del trigger intacto.
- Separación transporte HTTP/STDIO sobre un solo motor.

Estos son el activo diferencial del producto: el plan los extiende, nunca los relaja.
