# Auditoría interna de Quipu — dosier de traspaso

> **Fecha:** 2026-09-01 · **Alcance:** `QUIPU_ENTERPRISE` (código + base viva), `QUIPU_ENTERPRISE/sistema-a` (metodología), `developTool` (plan), `chasqui_n8n` (cliente).
> **Contraste:** `analisisExterno/InformeEstratégicoQuipu.md` (análisis externo, hecho sólo sobre las dos guías, sin leer el código).
> **Informe publicado:** https://claude.ai/code/artifact/b6f6a235-5276-4af2-86ff-e9f953de87c7
>
> **Nada del producto fue modificado durante la auditoría.** El único efecto lateral fue un token
> Sanctum emitido para probar el perímetro, revocado inmediatamente (`POST /api/auth/logout`, 200).

---

## 0 · Contrato de entrada — léeme primero

Este documento existe para que **una sesión fresca pueda escribir el plan nuevo sin repetir la
auditoría**. Todo lo que sigue está medido, no inferido de documentación, salvo lo que se marca
explícitamente como inferencia.

**Convención de etiquetas** (la misma que pidió el encargo original):

| Etiqueta | Significa |
|---|---|
| `HECHO` | Comprobado contra código fuente, esquema desplegado o respuesta HTTP. Trae comando o referencia `fichero:línea`. |
| `INFERENCIA` | Conclusión derivada de hechos citados. Nunca de documentación sola. |
| `RECOMENDACIÓN` | Decisión propuesta. No está verificada: es criterio. |
| `DESCONOCIDO` | No se pudo determinar con la evidencia disponible. Está listado en §6 para que no se dé por sabido. |

**Qué NO hay que volver a hacer:** inventariar tools, contar migraciones, leer las 62 migraciones,
correr el CI, sondear el perímetro, inventariar Chasqui. Todo está en §3 y §4 con su comando de
reproducción por si hace falta reverificar un punto concreto.

**Qué SÍ queda por hacer:** §14 (decisiones que necesitan al humano) y §15 (el trabajo concreto de
la sesión siguiente). El objetivo declarado por el propietario es **reestructurar el plan de
`developTool/02-plan-fases.md` o escribir uno nuevo**.

---

## 1 · Mapa de repositorios

| Ruta | Rol | Git propio | Estado |
|---|---|---|---|
| `/mnt/datos/Programacion/QUIPU_ENTERPRISE` | **El producto.** Laravel 12 + PostgreSQL 16 + React 19 | sí (50 commits, rama `main`) | activo |
| `…/QUIPU_ENTERPRISE/sistema-a` | **La metodología.** Paquete de ejecución agéntica (FLOTA) | sí, **independiente** (3 commits) | activo; el producto lo ignora en `.gitignore` |
| `/mnt/datos/Programacion/developTool` | **El plan.** 5 documentos de diseño | no es repo git | desactualizado tras esta auditoría |
| `/mnt/datos/Programacion/chasqui_n8n` | **Cliente.** Primer proyecto real a gobernar | sí | contiene la única auditoría previa de Quipu hecha desde fuera |
| `/mnt/datos/Programacion/QUIPU` | v1, **archivada** | sí | fuera de alcance, no tocar |

**Dato importante:** los dos historiales (producto y `sistema-a`) **no se cruzan nunca**. El commit
`aa23105` («Saca de versión el andamiaje de desarrollo») sacó `.claude/`, `.opencode/`, `CLAUDE.md`,
`ESTADO.md` y `PLAN_IMPLEMENTACION.md` del repo del producto y los movió a `sistema-a/`.

### Documentos vivos que hay que leer (y sólo estos)

| Documento | Qué contiene | ¿Sigue vigente? |
|---|---|---|
| `sistema-a/INDICE.md` | punto de entrada del paquete, las cuatro capas | sí |
| `sistema-a/NORMATIVA/CONSTITUCION.md` | 15 reglas EARS no negociables | sí |
| `sistema-a/NORMATIVA/VALIDACION.md` | tipos de evidencia y Definición de Hecho | sí |
| `sistema-a/NORMATIVA/HANDOFF.md` | contrato de checkpoint (validado por `bin/checkpoint.sh`) | sí |
| `sistema-a/NORMATIVA/ESCALAMIENTO.md` | clases A/B/C | sí |
| `sistema-a/ESTADO/ESTADO.md` | inventario de capabilities y deuda | sí, y **es veraz** (verificado) |
| `sistema-a/ESTADO/DEUDA-F2.md` | deuda consolidada A–E | sí, con reordenación (§11) |
| `sistema-a/ESTADO/CIERRE-F1.md` | comprobación de cierre de F1 | sí |
| `developTool/02-plan-fases.md` | plan por fases F0–F5 | **parcialmente obsoleto** — ver §10 |
| `developTool/03-metodologia-agentes.md` | estado del arte 2026 en verificación | sí, y es lo mejor del plan |
| `developTool/04` y `05` | FLOTA y su arquitectura | principios sí, orden no |
| `analisisExterno/InformeEstratégicoQuipu.md` | análisis externo | marco sí, puntuaciones no (§8) |
| `chasqui_n8n/pedidos/010-*.md` + `decisiones/PROCESO-002.md` | **auditoría previa del 2026-08-28** | crítico, ver §5 |

---

## 2 · Cómo se verificó (comandos reproducibles)

La pila estaba **corriendo** durante la auditoría (`quipu-ent-api`, `-postgres`, `-redis`, `-web`).

```bash
cd /mnt/datos/Programacion/QUIPU_ENTERPRISE

# CI completo — reejecutado, verde
docker compose exec -T -e COMPOSER_PROCESS_TIMEOUT=1800 api composer ci   # exit 0
docker compose exec -T web pnpm run ci                                    # exit 0

# Objetos de imposición en la base viva
docker compose exec -T postgres psql -U quipu -d quipu -t -c "
SELECT 'tablas', count(*) FROM information_schema.tables WHERE table_schema='public'
UNION ALL SELECT 'funciones plpgsql', count(*) FROM pg_proc p JOIN pg_namespace n ON n.oid=p.pronamespace
  WHERE n.nspname='public' AND p.prolang=(SELECT oid FROM pg_language WHERE lanname='plpgsql')
UNION ALL SELECT 'triggers', count(*) FROM pg_trigger WHERE NOT tgisinternal
UNION ALL SELECT 'checks', count(*) FROM pg_constraint WHERE contype='c'
UNION ALL SELECT 'fks', count(*) FROM pg_constraint WHERE contype='f';"

# Trabajo real cargado
docker compose exec -T postgres psql -U quipu -d quipu -c "
SELECT 'necesidad',count(*) FROM necesidad UNION ALL SELECT 'cambio',count(*) FROM cambio
UNION ALL SELECT 'requisito',count(*) FROM requisito UNION ALL SELECT 'decision',count(*) FROM decision
UNION ALL SELECT 'invariante',count(*) FROM invariante UNION ALL SELECT 'task',count(*) FROM task
UNION ALL SELECT 'evidence',count(*) FROM evidence UNION ALL SELECT 'firma',count(*) FROM firma;"

# Middleware real por ruta (revela guardias de capacidad)
docker compose exec -T api php artisan route:list --json
```

**Ojo con `route:list`:** el middleware aparece con su nombre de clase completo
(`App\Http\Middleware\RequiereAbility:bloque:aprobar`), **no** como el alias `ability:`. Filtrar por
`startswith('ability')` da cero falsos negativos… y una conclusión equivocada. Filtrar por
`'RequiereAbility' in m`.

---

## 3 · Inventario medido

### 3.1 El producto

| Métrica | Valor | Fuente |
|---|---|---|
| Migraciones | 62 (9.077 líneas) | `code/api/database/migrations/` |
| Tablas en la base viva | 118 | `information_schema` |
| Funciones PL/pgSQL | 53 | `pg_proc` |
| Triggers (no internos) | 49 | `pg_trigger` |
| CHECK constraints | 48 | `pg_constraint` |
| Claves foráneas | 202 | `pg_constraint` |
| Tools MCP | 46 | `app/Mcp/Tools/` |
| Rutas REST | 128 | `route:list` |
| Rutas **sin** autenticación | 3 (`auth/humans`, `auth/session`, `health`) | `route:list` |
| Rutas con guardia de capacidad | **4 de 128** | `route:list` |
| Servicios de `app/Protocol` | 29 (~6.700 líneas) | `app/Protocol/` |
| Controladores | 41 | `app/Http/Controllers/` |
| Modelos Eloquent | 38 | `app/Models/` |
| Comandos artisan | 4 (`adoptar`, `export`, `index`, `token`) | `app/Console/Commands/` |
| Suites de test API | 50 ficheros, 12.215 líneas | `tests/Feature/` |
| Tests API | **562 passed, 3408 aserciones, 216,08 s, exit 0** | reejecutado |
| Tests unitarios | **0** (`tests/Unit/` sólo tiene `.gitkeep`) | — |
| Web | 33 pantallas, ~14.813 líneas TS/TSX | `code/web/src/` |
| Tests web | **73 passed, 20 ficheros, 9,95 s, exit 0** | reejecutado |
| phpstan | nivel 8, `[OK] No errors`, 275 ficheros | CI |
| pint | PASS, 342 ficheros | CI |
| TODO / FIXME / HACK | **0** | grep |
| Capabilities OpenSpec | 5 · 33 requirements · 39 escenarios | `sistema-a/openspec/specs/` |

### 3.2 Contenido real de la base viva

```
project 1 (chasqui)   feature 8    block 8    member 6    knowledge_entry 8
necesidad 0   cambio 0   requisito 0   enlace 0   task 0   evidence 0
decision 0    invariante 0   contradiccion 0   linea_base 0
code_class 0  code_component 0
firma 4  ← todas huérfanas (ver H-09)
```

Los 8 bloques son `chasqui-*-P-002 … P-009`, cargados el 2026-08-28. Los 8 features tienen
`cambio_id = NULL` y `es_heredado = true`.

### 3.3 La metodología (`sistema-a`)

| Métrica | Valor |
|---|---|
| Ledger `.session/progress.json` | 71 entradas (23 ejecutor · 18 verificador · 30 orquestador) |
| Tareas ejecutadas | 13 (F0: T00–T06 · F1: T00–T06) |
| Tareas con retrabajo | 4 · `F1/T02` llegó al **ciclo 4**, `F1/T04` al 2, `F1/T05` al 2, `F0/T05` al 2 |
| Veredictos estructurados | 83, **todos `CUMPLE`** — los fallos de ciclos anteriores quedaron en prosa ⇒ no hay tasa de primera pasada calculable |
| Escalamientos clase C | 7 |
| Scripts `bin/` | 6 (`estado`, `checkpoint`, `verificar`, `sombra`, `metricas`, `flota`) — 1.236 líneas |
| Gasto LLM total de la flota | **$0,042914** en toda su historia, 13 turnos medidos |
| `QUIPU_VERIF_DETERMINISTA` | `0` (apagado) por omisión; racha de sombra 0/5 |

### 3.4 El origen de F2 (`chasqui_n8n`) — inventariado, cierra `DEUDA-F2 § A3`

```
decisiones/*.md        24      (frontmatter: id, dominio, estado, supersede, superseded_by,
                                motivo_reemplazo, relacionada_con, implementada_en, afecta, procedencia)
pedidos/*.md           10 abiertos
pedidos/archivo/*.md    1
```

---

## 4 · HECHOS

Cada uno con su referencia. Son la base de todo lo que sigue.

### Bloque A — La evidencia no se verifica

**H-01 · Quipu no ejecuta nada.**
`grep -rn "Process::\|exec(\|shell_exec\|proc_open" app/` → **cero coincidencias**. No hay
`symfony/process` en `composer.json` como dependencia usada por el dominio. Confirmado
independientemente por `sistema-a/PLAN/FLOTA/PLAN_IMPLEMENTACION.md` §«Cumplimiento de las
restricciones duras»: *«El producto no llama a LLM — Verificado en Fase 0»*.

**H-02 · `add_evidence` guarda texto pegado por el agente.**
`code/api/app/Mcp/Tools/AddEvidence.php:62` → `'content' => (string) $request->get('content')`.
El esquema de la tabla (`database/migrations/2026_07_12_130000_create_task_and_evidence_tables.php`)
declara `content TEXT NOT NULL` y `metadata JSONB`. **No hay** `exit_code`, ni huella de salida, ni
comando, ni marca temporal de producción. `grep -i "hash\|checksum\|sha256" app/Models/Evidence.php
app/Protocol/*.php` → sólo aparece en `TokenIssuer` (hash de credencial), nunca sobre evidencia.

**H-03 · El gate cuenta filas del tipo correcto, no hechos.**
`has_required_evidence()` (migración `2026_07_13_104000_create_strict_done_gate.php`) sólo comprueba
que exista **alguna** fila de `evidence` con el `evidence_type` exigido, ligada a una task del bloque.
El contenido nunca se inspecciona.

**H-04 · Nada de F3 existe en código.**
`grep -rn "mutation_score\|suite_diff\|env_limpio\|eval_report" app/ database/` → cero.
`lookup_evidence_type` tiene 12 códigos, todos declarativos: `ci_output`, `curl_request`,
`curl_response`, `screenshot`, `migration_rollback`, `test_output`, `audit_report`, `file_manifest`,
`response_shape`, `seed_output`, `visual_check`, `finding`.

**H-05 · La verificación determinista que falta ya existe, fuera del producto.**
`sistema-a/bin/verificar.sh`, 348 líneas. Hace: CI completo dentro de los contenedores; regla
suite-diff contra `.session/baseline-suites.json` (falla si `passed` baja, o si hay `failed`/
`skipped`/`incomplete`/`risky`); alcance tocado vs. declarado en el fichero de tarea (ensanchable
**sólo** por resolución de escalamiento que nombre el fichero, no por autodeclaración del ejecutor);
mensaje de commit literal (ignorando trailers). Emite JSON. Tres estados
(`0` / `sombra` / `1`), `FALLA` vinculante, `PASA` **no aprueba nada** — sólo habilita al Verificador
a juzgar semántica, preservando la regla 14 de la CONSTITUCIÓN.

### Bloque B — La norma no alcanza al trabajo

**H-06 · El payload de `claim_block` no lleva norma.**
`code/api/app/Protocol/TaskPayloadBuilder.php:33` (`forBlock`) devuelve: `block`, `feature`,
`dependencies`, `contracts` (:82), `endpoints` (:83), `screens` (:84), `gate` (:87), y dentro de
`block`: `deliverables`, `acceptance_criteria`, `required_evidence`.
`grep -ni "rule\|invariante\|decision" app/Protocol/TaskPayloadBuilder.php` → **cero coincidencias**.

**H-07 · `block_rule` y `rule_implementation` son tablas muertas.**
Creadas en `database/migrations/2026_07_12_110100_create_feature_and_block_tables.php:138` y `:68`.
`grep -rn "block_rule" app/ routes/ tests/ --include=*.php` → **sólo la migración**. Idem
`rule_implementation`. Son el único puente previsto entre norma y trabajo, y nadie lo usa.

**H-08 · La detección de contradicción no es mecánica.**
`app/Mcp/Tools/VerificarContradicciones.php` devuelve `pendientes[]` (leídas de
`contradiccion.cambio_id`) e `invariantes_aplicables[]` (los vigentes del dominio del cambio). **No
cruza** el alcance del cambio contra el ancla de los invariantes. El juicio es del LLM y **abrir la
contradicción es voluntario**: ninguna puerta lo exige.
Relacionado: `app/Mcp/Tools/InvariantesDe.php` empareja con `evidencia LIKE 'ruta%'` — **no** hace
JOIN contra `code_index` como afirma `developTool/02-plan-fases.md` §1.2. Divergencia plan↔código.

**H-09 · Las firmas pueden quedar huérfanas, y hay 4 ahora mismo.**
`firma` es polimórfica (`entidad_tipo`, `entidad_id`) y **no tiene FK hacia lo firmado**; sólo
`fk_fi_entidad_tipo → lookup_artefacto_tipo`. La base viva contiene 4 firmas `promuevo` sobre
`decision` 1, 2, 3 y 5, con `decision` vacía y `decision_id_seq.last_value = 6`.
`fn_firma_inmutable` impide borrar la firma; nada impidió borrar la decisión.

### Bloque C — La cadena de gobierno no alcanza lo adoptado

**H-10 · `cambio_ambito()` sólo llega a bloques por `feature.cambio_id`.**
`database/migrations/2026_07_20_160000_create_cambio_gate.php:56` define la función; la rama de
`criterio_bloque` está en `:87-91`:
```sql
SELECT 'criterio_bloque'::VARCHAR(30), bac.id
FROM block_acceptance_criterion bac
JOIN block   b ON b.id = bac.block_id
JOIN feature f ON f.id = b.feature_id
WHERE f.cambio_id = p_cambio_id;
```
Verificado también contra la función **desplegada** (`SELECT prosrc FROM pg_proc WHERE
proname='cambio_ambito'`).

**H-11 · Todo lo adoptado nace sin cambio.**
Los 8 features de `chasqui` en la base viva: `cambio_id = NULL`, `es_heredado = true`. Es el
comportamiento correcto del camino de adopción (`AdopcionEngine` marca lo heredado), no un fallo de
carga.

**H-12 · No hay tool MCP que cree un bloque colgando de un cambio.**
`import_project_structure` toma el payload del proyecto entero, sin vínculo a un cambio. Verificado
en `app/Mcp/Tools/ImportProjectStructure.php` y `app/Protocol/AdoptionEngine.php`.

**H-13 · No existe el alcance como dato.**
Ni `cambio` ni `block` declaran ficheros. `\d cambio` y `\d block` sobre la base viva lo confirman.
`block_deliverable` tiene una columna `path` (nullable, texto libre) que nada impone ni consume como
alcance. Es la primitiva ausente bajo cuatro capacidades distintas.

### Bloque D — El perímetro

**H-14 · `POST /api/auth/session` emite credenciales sin autenticación.** *(verificado en vivo)*
`code/api/routes/api.php:53` — fuera del grupo `auth:sanctum`.
`code/api/app/Http/Controllers/AuthController.php:42` — sólo valida `member_id` y llama a
`createToken('web')` (`:55`). El docblock lo declara deliberado (`:37`).
Prueba ejecutada: `curl -X POST localhost:8001/api/auth/session -d '{"member_id":1}'` → **HTTP 200**
con token; `GET /api/auth/me` con ese token → `roles: ["human_admin"]` y **22 permisos**, entre
ellos `block.approve`, `cambio.authorize`, `contradiccion.resolver`, `feature.approve`.
`POST /api/auth/logout` → `{"revoked":true}`; reintento → **401**. Sin residuo.

**H-15 · `GET /api/auth/humans` enumera personas sin autenticación.**
`routes/api.php:52`. Devuelve `id`, `username`, `display_name`, `member_type`, `is_active` de todos
los humanos activos. Es el insumo exacto que H-14 necesita.

**H-16 · Sólo 4 de 128 rutas llevan guardia de capacidad.**
`POST blocks/{block}/approve`, `POST blocks/{block}/reject` (`ability:bloque:aprobar`),
`POST decisiones/{decision}/promover` (`ability:decision:aprobar`),
`POST contradicciones/{contradiccion}/resolver` (`ability:contradiccion:resolver`).
Definidas en `routes/api.php:168-181`. Todas las demás llevan sólo `auth:sanctum`.

**H-17 · La gestión de miembros no tiene autorización.**
`routes/api.php:61-65` — `GET/POST members`, `POST members/{}/agents`,
`POST members/{}/regenerate-key`, `DELETE members/{}` — todas sólo bajo `auth:sanctum`.
`MemberController::store` (`:41`) crea `member_type = TYPE_HUMAN` (`:57`) y asigna
`['human_admin']` por omisión (`:64`). `grep -n "authorize\|Gate::\|can(" MemberController.php` →
**cero**. Sólo existe una Policy en todo el proyecto (`app/Policies/FirmaPolicy.php`) y **ningún
middleware global de autorización** (`bootstrap/app.php` sólo registra el alias `ability` y el
redirect de invitados).

**H-18 · Las abilities de agente excluyen firmar y aprobar — correctamente.**
`config/quipu.php:65-82`: `human => ['*']`, `agent => [bloque:leer, bloque:tomar, bloque:mover,
tarea:escribir, evidencia:escribir, diseno:escribir, conocimiento:escribir]`. El diseño es correcto;
el problema es que sólo 4 rutas lo consultan (H-16).

**H-19 · Los puertos se publican en todas las interfaces.**
`docker-compose.yml`: `"8001:8000"`, `"5436:5432"`, `"6382:6379"`, `"5174:5173"` — sin prefijo
`127.0.0.1`.

**H-20 · Ningún test cubre el camino de escalada.**
`grep -rn "members" tests/Feature/` → `AgentTokenTest` y `TeamTest` prueban secreto de la key, no
autorización de alta. No hay test que compruebe que un token de agente recibe 403 en `POST /members`.

### Bloque E — Integridad del propio motor

**H-21 · El esquema desplegado no coincide con las migraciones.**
`2026_08_26_120000_gobierno_contradiccion` figura en `migrations` (batch 7), pero
`SELECT proname FROM pg_proc WHERE proname LIKE '%mismo_proyecto%'` → **vacío**. La migración se
editó después de aplicarse; su propio docblock lo documenta y lo justifica («todavía no está
consolidada en `main` y vive sólo en esta rama»). Contradice la regla 11 de la CONSTITUCIÓN. **No
existe detección de deriva.**

**H-22 · No hay lease ni heartbeat sobre el claim.**
`WorkflowService::claim` usa el índice único parcial `uq_task_implement_abierta` — exclusión mutua
correcta y libre de carrera. Pero `grep -rni "lease\|heartbeat\|timeout\|expira" app/Protocol/
app/Mcp/Tools/` → **cero**. Un worker que muere retiene el bloque indefinidamente.

**H-23 · No hay dispatcher ni tabla de sesión.**
No existe `sesion_agente`, ni `quipu:dispatch`, ni tools `sesion_*`. FLOTA es 100 % manual: sesiones
que un humano lanza. Los 4 comandos artisan son `adoptar`, `export`, `index`, `token`.

**H-24 · `quipu:index` es heurística de nombre y ruta.**
`app/Console/Commands/IndexCodebase.php` (355 líneas), autodescrito «deliberadamente tonto». Deduce
el tipo de clase del sufijo del nombre (`*Controller` → controller, etc.) y busca `*.php` y `*.tsx`.
Sin grafo de llamadas, sin importaciones, sin dependencias. `code_class` y `code_component` = 0 filas.

**H-25 · No hay UI para decisiones, invariantes ni contradicciones.**
`code/web/src/routes.tsx` no tiene ninguna ruta para ellas; `grep -rn "decisiones\|invariantes\|
contradicciones" src/pages/` → sólo coincidencias falsas («las tres decisiones del triage» en
`Necesidades.tsx`). **Promover una decisión y resolver una contradicción son actos exclusivamente
humanos y el humano no tiene dónde hacerlos** salvo curl.

**H-26 · `knowledge_entry` no tiene estado epistémico.**
Columnas: `title, category, tags, content, root_cause, solution, prevention, related_block,
created_by, project_id`. Sin `confirmado|inferido`, sin evidencia obligatoria, sin supersede.
**Contraste:** `invariante` **sí** lo tiene (`etiqueta` ∈ `{confirmado, inferido}` + CHECK
`ck_inv_confirmado_evidencia` que exige `ruta:símbolo`). Hay dos epistemologías en el mismo sistema.

**H-27 · El motor de huella y sospecha existe y funciona.**
`fn_mantener_huella` (SHA-256 sobre columnas sustantivas declaradas en
`lookup_artefacto_tipo.columnas_huella`) + `fn_propagar_sospecha` (trigger **por sentencia**, con
benchmark documentado: 1.000 filas 162 ms vs 240 ms por fila). Enganchado a 8 tablas. Es un motor de
*knowledge freshness* ya construido — cubre entidades de Quipu, **no** el repositorio.

**H-28 · La documentación de estado es veraz.**
`ESTADO/ESTADO.md` declara «pest 562 passed (3408 assertions)», «vitest 20 archivos / 73 tests»,
«46 tools», «50 suites». Todo reejecutado y coincidente. Es un dato importante: **se puede confiar
en lo que el equipo ha escrito sobre sí mismo**.

---

## 5 · La auditoría previa que ya existía (crítica)

`chasqui_n8n/pedidos/010-el-desarrollo-se-traza-en-quipu.md` y
`chasqui_n8n/decisiones/PROCESO-002.md`, ambos del **2026-08-28**.

**Qué pasó:** el 27-08 se cargó `chasqui` en Quipu (8 features, 43 rules, 8 bloques, 32 endpoints,
2 pantallas, 8 lecciones). El 28-08 Chasqui auditó el código fuente de Quipu y la base viva. Sus
conclusiones coinciden **exactamente** con H-06, H-07, H-10 y H-11, medidas de forma independiente
un día después de cerrar F1.

Conclusión textual de `P-010`:

> «La cadena de gobierno de Quipu **no puede atar ningún bloque** de Chasqui, y no hay tool MCP que
> cree un bloque colgando de un cambio. […] `block_rule` existe como tabla pero **ninguna línea del
> código la lee ni la escribe**, y el payload de `claim_block` no incluye reglas. El agente que
> reclama un bloque **no ve ni un invariante**.»

`PROCESO-002` cerró el asunto repartiendo autoridad en tres registros y **degradando a Quipu**:

| Registro | Responde | Autoridad |
|---|---|---|
| `decisiones/` | qué debe ser Chasqui | **norma** |
| `pedidos/` | qué se cambia y quién lo autorizó | **expediente** |
| **Quipu** | qué se construyó y con qué prueba | **evidencia** |

Y escribió como invariante vigente:

> «las `business_rule` de Quipu son un espejo de sólo lectura; la norma vigente es `decisiones/` y
> **ninguna puerta de Quipu la comprueba**»

También midió que `business_rule` no tiene `estado`, `supersede`, `motivo_reemplazo` ni
`procedencia`, y que **las 43 rules importadas perdieron 20 de los 63 invariantes vigentes** al
comprimirse; `PRODUCTO-001` no llegó a cargarse.

**Consecuencia para el plan:** el criterio de salida de F2 en `02-plan-fases.md` §2 —«el siguiente
cambio real de Chasqui se propone, autoriza, ejecuta, verifica y cierra desde Quipu»— **ya se
intentó y ya falló**, con veredicto escrito por el cliente. Reejecutar F2 sin resolver H-10/H-12
repite el resultado.

---

## 6 · DESCONOCIDOS

Cosas que **no** pude determinar. No las des por sabidas.

| # | Qué no se sabe | Por qué importa | Cómo se resolvería |
|---|---|---|---|
| D-01 | Rendimiento real de la flota (tasa de primera pasada, coste por microtarea, deriva) | Es el KPI que decidiría si FLOTA merece dispatcher | `bin/metricas.sh` lee `opencode.db` y quedó ciego tras el port a Claude Code del 26-08 (`DEUDA-F2 § D5`). Reinstrumentar antes de invertir en D2/D3 |
| D-02 | Si los 83 veredictos `CUMPLE` ocultan una tasa de rechazo alta | Sin ella no hay línea base de calidad agéntica | Los ciclos 1–3 de `F1/T02` y `F1/T04` guardaron veredictos en prosa. Habría que releer el ledger entrada a entrada |
| D-03 | Cuánto de las 43 `business_rule` de Chasqui es recuperable | Condiciona el coste de la unificación normativa | Comparar `chasqui_n8n/decisiones/*.md` (24 ficheros, 63 invariantes) contra `business_rule` en la base |
| D-04 | Si `openspec validate --all --strict` pasaría | `DEUDA-F2 § A1`: nunca se ha ejecutado; `node`/`npx` están denegados | Abrir el permiso o declarar por escrito que la validación es manual |
| D-05 | Si borrar proyectos es caso de uso real | `fk_ds_sucesora` en `RESTRICT` lo impide cuando hay decisiones superadas (`DEUDA-F2 § B3`) | Decisión de producto, no técnica |
| D-06 | Cuál de las dos configuraciones de agente manda hoy | `.claude/` y `.opencode/` conviven; ningún documento lo declara | `sistema-a/ARCHIVO/TRABAJO-SIN-TAREA.md` §2 lo registra sin resolverlo |
| D-07 | Volumen y coste de las sesiones de Claude Code del humano | Podrían superar por completo a la flota | No están instrumentadas (`PLAN_IMPLEMENTACION.md` §5 R7, declarado como hueco) |

---

## 7 · Diagnóstico del core

### 7.1 Lo que es correcto y hay que preservar intacto

1. **Las reglas viven en Postgres.** Un `UPDATE` directo desde psql recibe el mismo rechazo que la API. Ninguna herramienta construida sobre markdown puede ofrecer esto.
2. **`is_block_closeable()` + `block_gate_report()`.** La puerta y su explicación son el mismo objeto: alimentan el 422 y el panel con una consulta. **Repetir ese patrón en todo lo nuevo.**
3. **Criterios dado/cuando/entonces como filas.** La especificación nace ejecutable.
4. **Evidencia enlazada a criterio** (`evidence_criterion`, `trg_ec_same_block`, `trg_bac_before_met`, `trg_ec_no_orphan_met`). La estructura es correcta; falta la sustancia.
5. **Motor de huella y sospecha** (H-27). Vale más de lo que el equipo parece creer: es *knowledge freshness* ya construido y medido.
6. **La cadena de decisión de F1.** DAG sin ciclos, `confirmado|inferido` con evidencia obligatoria, promoción firmada **contra la huella exacta del contenido** (promover un texto distinto del firmado es imposible). Es el mejor trabajo del repositorio.
7. **Firmas inmutables + segregación de funciones** (`fn_firma_inmutable`, `fn_segregacion_funciones`, `fn_firmante_humano`).
8. **La vía rápida con deuda de análisis y plazo** (`fn_cambio_deuda`, `deuda_vencida`). Es la tercera palanca legítima ante saturación de verificación —bajar la barra deliberadamente— con expediente obligatorio. **Genuinamente novedosa; debería ser argumento comercial.**
9. **Disciplina de errores:** 422/403 con el mensaje del trigger intacto (`bootstrap/app.php` traduce `SQLSTATE P0001`), nunca un 500 con SQL dentro.
10. **El flujo OpenSpec** con cada escenario citando su test.
11. **La invariante «no existe tool de aprobación»** con test que la vigila (`McpToolsTest.php:41`).

### 7.2 Los tres puntos donde el mecanismo se interrumpe

> `INFERENCIA` — La tesis «el protocolo es mecánico, no una convención» es **cierta para la forma
> del flujo de trabajo** y **falsa para las tres cosas que deciden si el desarrollo agentic mantiene
> la coherencia**: la verdad de la evidencia (H-01…H-04), el alcance de la norma (H-06…H-08) y la
> identidad del humano (H-14…H-17).
>
> Todo lo que Quipu puede demostrar hoy es que **el expediente está completo**, no que **el trabajo
> está bien**. El principio P4 de FLOTA —«evidencia tipada o no ocurrió»— es en realidad «evidencia
> *declarada* o no ocurrió». Es la distancia entre un registro contable y una auditoría.

### 7.3 Lo sobrediseñado o mal orientado

| Pieza | Qué asumía | Qué pasó | Recomendación |
|---|---|---|---|
| **Capa de diseño** (`screen_component`, `component_state`, `component_interaction`, `component_data_source`, ~10 tablas) | que modelar la UI como datos daría al agente el contexto que le falta | los agentes leen el código; 0 filas; exige modelar a mano toda la interfaz. `ImpactAnalyzer` depende de ella y por eso tampoco funciona | congelar, no ampliar |
| **Catálogo de entrevista** (`AnalysisCatalog` 621 líneas, `NextQuestion`, `lookup_analysis_field`) | que el diagnóstico brownfield se obtiene preguntando al humano | pregunta lo que un agente deduce leyendo el repo | sustituir por análisis derivado; conservar sólo lo que el código no dice |
| **Plantillas de bootstrap** | greenfield como caso principal | el valor está en brownfield | congelar |
| **Dos modelos normativos** (`business_rule` vs `invariante`) | que podían convivir | `business_rule` sin estado ni supersede; Chasqui perdió 20 de 63 invariantes al importar | elegir uno: **`invariante`** |
| **46 tools MCP** | más cobertura = más capacidad | excede el presupuesto de atención del agente | consolidar hacia `contexto_para` + pocos verbos de escritura |
| **`AdopcionEngine` vs `AdoptionEngine`** | — | dos clases distintas (sellar línea base / importar estructura) con nombres casi idénticos en dos idiomas | deuda menor; renombrar al tocar |

---

## 8 · Contraste con el análisis externo

El informe externo (`InformeEstratégicoQuipu.md`) leyó **sólo las dos guías**, y él mismo lo declara
(«las dos guías no contienen ese roadmap»). Su marco es correcto; varias puntuaciones no.

### 8.1 Donde acierta — hacerle caso

- El core merece conservarse; no rehacer la arquitectura. **Confirmado por el código.**
- MCP es transporte, no ventaja competitiva.
- Ser agnóstico al agente es decisión estratégica crítica. **Ya es cierto** (H-01: el producto no llama a ningún LLM).
- Contexto por tarea, inteligencia de proyecto, coordinación multiagente y memoria continua son la frontera correcta.
- No convertirlo en Jira/IDE/base vectorial; no añadir cientos de tools; **no automatizar nunca la aprobación humana**.
- La seguridad no está lista para comercializar. Cierto, y peor de lo que estimó.
- La adopción de legado es ventaja competitiva mayor. Correcto — y por eso H-10 es tan grave.

### 8.2 Donde hay que corregirlo

| Área | Externo | Lo que dice el código | Real |
|---|---:|---|---:|
| **Evidencia** | 8,5/10 | La estructura vale 8,5; la sustancia no llega a 2 (H-01…H-04). **La corrección más consecuente**: toda la tesis se apoya ahí | **2/10** |
| **Trazabilidad** | 9/10 | Impecable dentro de Quipu; **inexistente para trabajo adoptado** (H-10, H-11). El mismo informe llama a la adopción «principal ventaja competitiva»: las dos afirmaciones no son compatibles | **5/10** |
| **Decisiones/invariantes** | 8,5/10 | El modelo merece 9; su conexión con el trabajo es 0 (H-06, H-07, H-08, H-25) | **4/10** |
| **Human-in-the-loop** | 9/10 | 9 en el motor, 2 en el perímetro (H-14…H-17) | **4/10** |
| **Memoria epistémica** | «no existe» | Parcialmente incorrecto: `invariante.etiqueta` + `ck_inv_confirmado_evidencia` **es** memoria epistémica, y `huella`+`sospecha` es un motor de frescura funcionando (H-26, H-27). La tarea es **unificar**, no inventar | **5/10** |
| **Comprensión del código** | 6/10 | Generoso (H-24) | **3/10** |
| **Coordinación multiagente** | 3/10 | Confirmado, con matiz: el claim es libre de carrera, pero sin lease (H-22) | 3/10 |

### 8.3 El desacuerdo de fondo

El informe externo pone **Multi-Agent Orchestration** (fase D) **antes** de **Autonomous
Verification** (fase E).

> `INFERENCIA` — Ese orden es peligroso. Sin evidencia observada, sin alcance declarado y sin
> leases, paralelizar sólo consigue que el trabajo no verificable llegue antes y en mayor volumen.
> Los datos del propio proyecto lo anticipan: con **fan-out 1**, de 13 tareas **4 necesitaron
> retrabajo** y una llegó al ciclo 4. Además, el alcance declarado es simultáneamente el insumo del
> gate de verificación **y** el mecanismo que evita que dos agentes se pisen: construirlo una vez
> resuelve las dos cosas, y hacerlo en el orden inverso obliga a construirlo dos veces.

---

## 9 · Los ocho cambios recomendados

`RECOMENDACIÓN` — Ninguno rehace el core. Todos lo completan donde se interrumpe.

| # | Cambio | Sobre qué | Por qué |
|---|---|---|---|
| **C1** | Autenticación real para humanos, **o** binding a `127.0.0.1` declarado como restricción dura mientras tanto | `AuthController::session`, `docker-compose.yml` | La integridad de todo lo demás depende de esto (H-14, H-19). No es deuda de F5 |
| **C2** | Autorización sobre la gestión de miembros: alta, baja, alta de agente y rotación de key detrás de una ability sólo humana, **con test de que un token de agente recibe 403** | `routes/api.php:61-65`, `MemberController` | Es el camino por el que un agente puede fabricar al humano que firma su trabajo (H-17, H-20) |
| **C3** | Integridad referencial de la firma: trigger que impide borrar una entidad firmada + migración que concilia las 4 huérfanas | `firma`, `decision` | El rastro de autoridad es el activo del producto y hoy puede apuntar a la nada (H-09) |
| **C4** | Detección de deriva de esquema en CI + reponer `fn_contradiccion_mismo_proyecto` con **migración nueva** (regla 11) | CI, migraciones | «La base manda» sólo es cierto si la base desplegada es la escrita (H-21) |
| **C5** | **`artefacto_alcance`: el alcance como dato.** Tabla que liga unidades de trabajo (cambio / bloque / microtarea) a rutas y símbolos. Declarado al planificar, comprobado al evidenciar | nuevo | **La primitiva que falta.** Desbloquea a la vez: detección de contradicción, propiedad de ficheros para agentes paralelos, contexto por tarea e impacto desde código real (H-13) |
| **C6** | **Evidencia observada.** `evidence` gana `origen (declarada\|observada)`, `comando`, `exit_code`, `huella_salida`, `producido_en`. Un runner ejecuta y publica; `has_required_evidence()` puede exigir `observada` | `evidence`, nuevo runner | Convierte el expediente en auditoría. **Es lo que ninguna herramienta del mercado hace** (H-01…H-05) |
| **C7** | **La norma viaja con el trabajo.** El payload de `claim_block` incluye los invariantes que intersectan el alcance del bloque y las decisiones vigentes de su dominio. `block_rule` se resucita como enlace derivado o se elimina | `TaskPayloadBuilder` | Sin esto F1 es un modelo de datos que nadie consulta (H-06, H-07) |
| **C8** | **Puente adopción ↔ demanda.** Que un cambio alcance bloques directamente, no sólo por `feature.cambio_id` | `cambio_ambito()`, esquema | Cambio pequeño, consecuencia grande: desbloquea brownfield y con él F2 (H-10, H-11, H-12) |

---

## 10 · Roadmap revisado

`RECOMENDACIÓN` — Principio de ordenación único: **cerrar el ciclo sobre un cambio real, de punta a
punta, con evidencia que Quipu haya observado.** Lo que no acerca a ese momento, se pospone.

### Correspondencia con el plan actual (`02-plan-fases.md`)

| Plan actual | Destino |
|---|---|
| F0 estabilización | **cerrada**, sin cambios |
| F1 decision-chain | **cerrada**; su efecto llega en F-C |
| F2 puente Chasqui | **se aplaza** y se reescribe: pasa a ser F-D2, y sólo después de C8 |
| F3 verificación | **se adelanta**: es el núcleo de F-B, no una fase posterior |
| F4 dial de control + skills | se parte: el dial pasa a F-G (como modelo de riesgo); las skills se distribuyen |
| F5 SaaS | pasa a F-H, sin cambios, con el perímetro ya resuelto en F-A |
| — | **nuevas**: F-A (perímetro), F-D (dogfooding), F-E (contexto) |

### Las fases

**F-A · Cierre del perímetro y del expediente** — `CRÍTICA` — 1–2 semanas
Hacer ciertas las garantías que Quipu ya anuncia. No añade capacidades: repara la credibilidad de
las existentes. Contiene C1, C2, C3, C4 + una **suite de perímetro** (los sondeos de esta auditoría
convertidos en tests que deben devolver 401/403).

**F-B · Alcance y evidencia observada** — `CRÍTICA` — 3–4 semanas
La fase que cambia **lo que Quipu es**. Aquí se productiza `bin/verificar.sh`. Contiene C5, C6, más:
- suite-diff como tipo de evidencia **y como puerta** (línea base por proyecto; `passed` no baja);
- puerta de alcance (ficheros tocados del diff que captura el runner vs. alcance declarado);
- tipos declarables nuevos `mutation_score` y `env_limpio` (el DoD tipado ya los soporta: son filas de lookup + runner).

**F-C · La norma alcanza al trabajo** — `ALTA` — 2–3 semanas
Contiene C7, C8, más:
- **detección mecánica de contradicción**: cuando el alcance de un cambio intersecta el ancla de un invariante, el agente debe declarar explícitamente «no contradice» o abrir contradicción; el gate no deja avanzar hasta que todo invariante intersectado tenga declaración. *La base no juzga semántica — exige que alguien se pronuncie.*
- **pantallas web** de decisiones, invariantes y contradicciones (H-25);
- unificación normativa: `invariante` gana; `business_rule` se retira o queda como espejo declarado.

**F-D · Dogfooding real** — `CRÍTICA` — continua, arranca al cerrar F-B
- **D1: Quipu se convierte en proyecto dentro de Quipu.** F-C en adelante se planifica, ejecuta y verifica a través de él.
- **D2: Chasqui.** Con C8 aplicado el bloqueo de `P-010` desaparece y `PROCESO-002` puede revisarse.
- Reinstrumentar el coste (resuelve D-01).

**F-E · Contexto por tarea** — `ALTA` — 2–3 semanas
`contexto_para(bloque|microtarea)`: a partir del alcance, devuelve decisiones vigentes, invariantes
aplicables, contratos congelados, cambios cerrados relacionados, sospechas pendientes, deuda y
evidencia exigida — **cada elemento etiquetado** `confirmado` / `inferido` / `desconocido` /
`superado`. Unificar la epistemología (H-26). Cerrar el ciclo de aprendizaje: descubrimiento →
conocimiento propuesto → evidencia → validación → confirmado.

**F-F · Coordinación multiagente** — `MEDIA` — después de B, C y E
Leases con TTL y heartbeat (H-22); **detección de conflicto por alcance** (sale directamente de C5);
registro de capacidades y niveles L1–L4 como consulta SQL sobre historial de evidencia, con democión
automática. **Sólo entonces**, el dispatcher. Los agentes reportan a la base; no se hablan.

**F-G · Revisión agéntica y modelo de riesgo** — `MEDIA`
Clasificación de riesgo por operación gobernando cuánta atención humana exige cada acto (es la
implementación correcta del «dial» de la F4 original). Revisor / tester / seguridad como productores
de evidencia tipada; todo juez LLM exige meta-evaluación registrada antes de ser vinculante.

**F-H · SaaS** — `BAJA` hasta haber tracción
Sin cambios respecto al plan (tenancy con RLS, OAuth 2.1 + PKCE, billing, sólo metadatos en la nube).
Mucho más barato si el perímetro se arregló en F-A. Condicionada a 5–10 usuarios externos self-hosted.

---

## 11 · Prioridades, dependencias, criterios de cierre

### 11.1 Prioridad

| Prioridad | Cambios |
|---|---|
| **CRÍTICA** | C1 perímetro · C2 autorización de miembros · C5 alcance como dato · C6 evidencia observada · C8 puente adopción↔demanda · dogfooding propio |
| **ALTA** | C3 integridad de firma · C4 deriva de esquema · puerta suite-diff · puerta de alcance · C7 norma en el payload · detección mecánica de contradicción · UI de decisiones · `contexto_para` |
| **MEDIA** | unificación normativa · leases y conflicto por alcance · niveles de autonomía · modelo de riesgo · unificación epistémica · retirada de la capa de diseño |
| **BAJA** | dispatcher · SaaS · marketplace de skills · plantillas · limpieza de nomenclatura |

### 11.2 Reordenación de `DEUDA-F2.md`

`DEUDA-F2.md` es un documento excelente y su contenido sigue vigente. Cambian cuatro prioridades:

- **`B5`** (resurrección `superada → vigente`: `decision` no tiene grafo de transiciones como dato) **sube a alta** — F-C convierte a `decision` en gobierno efectivo.
- **`C3` y `C4`** (worktrees que replican `.env` a mano; pila Docker exclusiva que **serializa** el CI) **suben a alta** — F-F depende de poder verificar en paralelo. Hoy dos tareas paralelas rinden como una.
- **`D5`** (métricas ciegas) **sube a alta** — F-D no se puede medir sin ella.
- **`A3`** (inventario del origen de F2) **queda resuelto**: 24 decisiones, 10 pedidos abiertos, 1 archivado (§3.4).
- **`B1`** (`TRUNCATE` sortea los triggers) se absorbe en C4: es el mismo problema de integridad del motor.

### 11.3 Dependencias — las cinco que fijan el orden

```
F-A ──► F-B ──┬──► F-C ──┐
              │          ├──► F-D ──┬──► F-F
              └──► F-E ──┘          └──► F-G
                                            F-H (sólo con tracción)
```

1. **F-A antes que todo.** Mientras cualquiera pueda emitirse un token de administrador, cada garantía construida encima es decorativa.
2. **C5 (alcance) antes de cinco cosas a la vez:** puerta de alcance, norma en el payload, detección de contradicción, `contexto_para`, detección de conflicto entre agentes. **Es la primitiva de mayor apalancamiento del sistema.**
3. **C6 (evidencia observada) antes de** suite-diff, puerta de alcance y cualquier revisión agéntica. Los tres necesitan una salida en la que se pueda confiar.
4. **C8 (puente) antes de Chasqui.** Sin él, F2 choca con el mismo muro del 2026-08-28.
5. **F-E (contexto) antes de F-F (coordinación).** Paralelizar agentes con contexto indiscriminado multiplica el problema.

**Paralelizable:** F-C y F-E son independientes entre sí una vez cerrada F-B. Las pantallas web de
F-C no dependen del backend nuevo. La retirada de la capa de diseño y la limpieza de nomenclatura no
bloquean ni son bloqueadas. **Antes de intentar paralelismo real, resolver `DEUDA-F2 § C3` y `§ C4`.**

### 11.4 Criterios objetivos de cierre

Binarios y reejecutables. Ninguno admite «revisión humana» como única prueba.

**F-A**
- La suite de perímetro pasa: ningún endpoint emite credencial sin autenticar; un token de agente recibe 403 en toda ruta de gestión de miembros.
- Borrar una entidad firmada es imposible; **cero firmas huérfanas** en la base.
- El chequeo de deriva corre en CI y sale verde sobre base recién migrada **y** sobre la base viva.
- CI completo verde, sin suites desactivadas.

**F-B**
- Un bloque no llega a `done` sin al menos una evidencia con `origen = observada` cuyo `exit_code` y huella de salida produjo el runner.
- Cerrar con `passed` por debajo de la línea base es **rechazado por la puerta**, no por convención.
- Un fichero tocado fuera del alcance declarado hace fallar la evidencia, **con el nombre del fichero en el mensaje**.
- Demostrado sobre un cambio real, no sobre un fixture.

**F-C**
- `claim_block` devuelve los invariantes que intersectan el alcance del bloque; hay test que lo verifica.
- Un cambio cuyo alcance toca el ancla de un invariante no avanza hasta que ese invariante tenga declaración explícita.
- Un bloque de un proyecto **adoptado** puede colgar de un cambio y `cambio_ambito()` lo alcanza.
- Un humano promueve una decisión y resuelve una contradicción **desde la web**, sin curl.
- Existe **un único** modelo normativo.

**F-D**
- Quipu existe como proyecto en su propia base, con su línea base sellada.
- Al menos un cambio real **por proyecto** (Quipu y Chasqui) recorre necesidad → cambio autorizado → bloques → evidencia observada → firma → cerrado, íntegramente dentro de Quipu.
- El export markdown de ese cambio coincide con lo que la metodología file-based habría escrito.
- La instrumentación de coste vuelve a registrar actividad real.

**F-E**
- `contexto_para` devuelve todo elemento etiquetado con su estado epistémico; ninguno sin etiqueta.
- Medido sobre los dos proyectos con dogfooding: reducción de tokens de entrada y tasa de primera pasada frente a un agente sin la tool. **La mejora se declara con número, no con impresión.**

**F-F**
- Un worker que muere libera su bloque al vencer el lease, sin intervención.
- Un claim cuyo alcance intersecta otro activo se rechaza nombrando el conflicto.
- El nivel de autonomía sale de una consulta SQL sobre historial; una violación de guardarraíl lo baja **el mismo día**.
- Dos agentes cierran dos bloques del mismo feature en paralelo, sin conflicto y con evidencia observada.

---

## 12 · Qué NO debería desarrollarse

| No construir | Razón |
|---|---|
| Ampliación de la **capa de diseño** | modela como datos lo que los agentes leen del código; 0 filas tras un año |
| Ampliación del **catálogo de entrevista** | pregunta al humano lo que el repositorio ya responde |
| Más **plantillas de bootstrap** | el valor está en brownfield |
| **El dispatcher antes de F-B y F-C** | automatiza el reparto de trabajo que nadie puede verificar: convierte un problema de calidad en uno de volumen |
| **Mensajería agente-a-agente / A2A** | contradice P5 de la propia FLOTA: los handoffs son contratos en la base, no conversaciones |
| **Base vectorial / memoria semántica como capacidad principal** | el estado persistente ya vive en una base transaccional |
| **Más tools MCP** | 46 ya excede el presupuesto de atención de un agente |
| **Jueces LLM vinculantes sin meta-evaluación** | oracle determinista primero; juez estadístico después; juez sin meta-evaluación, nunca |
| **Automatizar cualquier aprobación humana** | destruye la única propiedad que nadie replica. Innegociable |
| **Un segundo modelo normativo** | `business_rule` e `invariante` ya causaron pérdida de norma al importar Chasqui |
| **Competir en generación de código** | Claude Code, Codex y Cursor poseen la interfaz del agente |

---

## 13 · Riesgos de seguir el roadmap actual sin modificarlo

| Riesgo | Gravedad | Evidencia de que es real |
|---|---|---|
| **F2 repite un fracaso ya medido** | CRÍTICA | §5: el cliente ya concluyó que Quipu no puede gobernarlo y lo registró como norma vigente |
| **La promesa central resulta falsa en la primera demo** | CRÍTICA | H-02: basta pegar un CI verde falso y cerrar un bloque |
| **El hueco de perímetro llega a producción** | CRÍTICA | H-14 verificado en vivo, con H-19 (puertos abiertos) |
| **Construir FLOTA antes que la verificación** | ALTA | §3.3: con fan-out 1, 4 de 13 tareas necesitaron retrabajo, una llegó al ciclo 4 |
| **La deriva de esquema pasa desapercibida** | ALTA | H-21: ya ocurrió, y en silencio |
| **Cero dogfooding convierte el roadmap en literatura** | ALTA | los tres huecos estructurales se descubren en la primera hora de uso real; ninguno estaba en `DEUDA-F2` |
| **El paquete de metodología se convierte en el producto** | ALTA | ya está pasando: `sistema-a` tiene la verificación determinista y el ledger; `PROCESO-002` ya formalizó la reducción de Quipu a «la prueba» |
| **Mantenimiento sobre apuestas caducadas** | MEDIA | capa de diseño + entrevista + plantillas: miles de líneas, cero filas en uso, y siguen apareciendo en F4 del plan |
| **Absorción por vendors de IDE** | MEDIA | se mitiga siendo el plano de control agnóstico y auditable; **no** añadiendo funcionalidades que ellos también tendrán |

---

## 14 · Decisiones que necesitan al humano (clase C)

La sesión fresca **no debe resolverlas sola**. Cada una cambia el alcance del plan.

| # | Decisión | Opciones | Impacto |
|---|---|---|---|
| **DEC-1** | **Modelo de autenticación.** ¿Autenticación real ahora, o `127.0.0.1` + declaración escrita de que Quipu es una herramienta local hasta F-H? | (a) auth real ya · (b) binding local + restricción documentada · (c) ambas por etapas | Fija si F-A dura 3 días o 2 semanas. **Bloquea todo lo demás** |
| **DEC-2** | **Aplazar F2 / Chasqui.** ¿Se acepta el diagnóstico de §5 y se pospone hasta después de C8? | (a) aplazar (recomendado) · (b) intentar F2 con workaround manual · (c) aceptar el reparto de `PROCESO-002` como definitivo | Define si Quipu aspira a gobernar o se conforma con ser «la prueba» |
| **DEC-3** | **Alcance del runner de evidencia.** ¿Binario propio (`quipu run <comando>`), hook del CLI agéntico, o servicio? | (a) CLI propio (recomendado: agnóstico al agente) · (b) hooks por proveedor · (c) daemon | Es la pieza más importante de F-B; condiciona la promesa comercial |
| **DEC-4** | **Destino de la capa de diseño.** ¿Congelar, retirar o mantener? | (a) congelar sin ampliar (recomendado) · (b) retirar con migración · (c) mantener | ~10 tablas + pantallas + `ImpactAnalyzer` |
| **DEC-5** | **Modelo normativo único.** ¿`invariante` absorbe a `business_rule`, o coexisten declarando espejo? | (a) absorber (recomendado) · (b) espejo declarado · (c) statu quo | Chasqui ya perdió 20 de 63 invariantes por esto |
| **DEC-6** | **`openspec validate --strict`.** ¿Se abre el permiso de `node`/`npx` o se declara validación manual? | (a) abrir · (b) declarar manual | `DEUDA-F2 § A1`; con más changes no escala |
| **DEC-7** | **Configuración de agente canónica.** ¿`.claude/` o `.opencode/`? | — | D-06; dos configuraciones divergiendo |
| **DEC-8** | **`docker-compose.yml` con `restart: "no"`** — pendiente de confirmación del propietario **desde F0** | — | `DEUDA-F2 § C2` |

---

## 15 · Qué hacer en la sesión siguiente

`RECOMENDACIÓN` — En este orden.

### Paso 0 — Antes de escribir nada
1. Leer §0, §4 y §14 de este documento. **No re-auditar.**
2. Plantear al humano las decisiones DEC-1 a DEC-5 (las otras pueden esperar). Sin DEC-1 y DEC-2 el plan no se puede escribir.

### Paso 1 — Acción inmediata, independiente del plan
Publicar los puertos del compose en `127.0.0.1` en vez de en todas las interfaces. Cambio de una
línea que cierra el hueco más grave mientras se diseña la solución real (H-19 + H-14).

### Paso 2 — Escribir el plan nuevo
Reemplazar o reescribir `developTool/02-plan-fases.md` con el roadmap de §10, y crear en el paquete:
- `sistema-a/PLAN/F-A-execplan.md`
- `sistema-a/TAREAS/F-A/T01…Tnn.md`

**Esto es obligatorio por el propio método:** `AGENTS.md` § Nivel de autonomía dice que sin
`PLAN/F<n>-execplan.md` con sus `TAREAS/`, **ningún agente tiene alcance pre-aprobado** y todo
trabajo de desarrollo es escalamiento clase C. Hoy no hay fase activa.

Formato de tarea: mirar `sistema-a/TAREAS/F1/T02-migraciones-normativas.md` como modelo — es la más
completa, y `bin/verificar.sh` extrae de ahí el alcance declarado (rutas entre comillas invertidas) y
el mensaje de commit literal (bloque cercado tras una línea que contenga «Commit»).

### Paso 3 — Registrar el cambio de rumbo
Escribir la decisión que aplaza F2 citando la medición del 2026-08-28 como evidencia. **El registro
donde hacerlo es hoy `chasqui_n8n/decisiones/`, porque Quipu todavía no puede sostenerlo** — esa
incomodidad es exactamente el argumento a favor del paso 5.

### Paso 4 — Restaurar la integridad del motor
Reponer `fn_contradiccion_mismo_proyecto` con una **migración nueva** (regla 11: una migración
aplicada nunca se edita) y añadir el chequeo de deriva al CI.

### Paso 5 — Prototipo mínimo
Prototipar el runner de evidencia contra **un solo comando real**. Es la rebanada más pequeña de C6:
decide si el enfoque es correcto antes de comprometer un mes, y produce la primera fila de evidencia
observada que la base ha visto nunca.

---

## 16 · Arquitectura objetivo (resumen para el plan)

La arquitectura elegida en `developTool/05-arquitectura-sistema-agentico.md` **es correcta y no debe
cambiarse**: Quipu es el plano de control y **ya es el orquestador**. No hace falta LangGraph, ni
CrewAI, ni un SDK de agentes en el núcleo — las transiciones de grafo ya son filas, y duplicar el
estado en dos sitios produce dos verdades. Lo que cambia no es la topología, sino **qué es capaz de
imponer** el plano de control.

### Las cinco responsabilidades del Quipu que debería existir

| Responsabilidad | Pregunta que contesta | Mecanismo |
|---|---|---|
| **Verdad operativa** | ¿qué es cierto hoy, y con qué certeza? | grafos de decisión / requisito / trabajo / evidencia, con etiqueta epistémica y huella; motor de sospecha propagando obsolescencia |
| **Contexto acotado** | para esta tarea, ¿cuál es el contexto correcto? | `contexto_para` derivado del alcance declarado. No «aquí tienes todo», sino «esto es lo que te obliga» |
| **Procedencia verificable** | ¿esta prueba la produjo alguien o la escribió alguien? | runner que ejecuta, sella y fecha; puertas que pueden exigir `observada`. **Éste es el foso** |
| **Autoridad** | ¿quién autorizó esto, y sigue siendo cierto lo que firmó? | firmas inmutables contra huella de contenido, segregación de funciones, integridad referencial, perímetro cerrado |
| **Autonomía con expediente** | ¿cuánta libertad se ha ganado este agente, y en qué capacidad? | L1–L4 por capacidad como consulta sobre historial de evidencia y violaciones; democión automática |

### Restricciones que hay que seguir respetando

- **El producto no llama a ningún LLM.** Verificado hoy; es lo que lo hace agnóstico al agente y auditable.
- **Las reglas viven en Postgres primero.** Sólo lo que no cabe en un trigger va en PHP.
- **Humanos y agentes por la misma puerta.** Sin puerta de servicio — y, tras F-A, sin puerta trasera.
- **Los agentes reportan a la base, no se hablan entre ellos.** El handoff es una fila verificable.
- **Ningún acto de gobierno es automatizable.**

### Diferenciación defendible

«Tenemos MCP» y «tenemos gobernanza» dejarán de distinguir en doce meses. Lo que no puede copiarse
encima de ficheros markdown ni dentro de una sesión de agente son dos cosas:

1. **El único sistema que rechaza trabajo cuya prueba no observó.** Los hooks de un CLI son por sesión y son consejos. Una puerta que exige evidencia sellada —entre sesiones, entre agentes y entre proveedores— no la tiene nadie.
2. **Autonomía con expediente.** Nadie más puede demostrar con filas de una base por qué un agente concreto merece más libertad en una capacidad concreta, ni retirársela el mismo día en que la pierde.

**Formulación corta:** Git responde *qué cambió*; Quipu responde *qué estaba permitido cambiar, con
qué autoridad y con qué prueba* — y sólo cree en pruebas que vio producir.

---

## 17 · Métricas que deberían gobernar el producto

Del análisis externo §37, filtradas por «¿es medible con lo que habrá tras F-B?»:

| Métrica | Medible tras | Cómo |
|---|---|---|
| **Evidence Coverage observada** | F-B | % de criterios cerrados con evidencia `origen = observada` |
| **Agent Rework Rate** | F-B + F-D | ciclos por tarea en el ledger; hoy no calculable (D-02) |
| **Decision Compliance** | F-C | % de cambios cuyo alcance intersectó un invariante y trae declaración explícita |
| **Requirement Coverage** | ya | % de cambios con origen (`necesidad`) y criterios verificables |
| **Context Accuracy** | F-E | tokens entregados vs. tokens citados en la evidencia resultante |
| **Knowledge Freshness** | ya (parcial) | sospechas pendientes / enlaces totales — `fn_propagar_sospecha` ya lo produce |
| **Agent Conflict Rate** | F-F | claims rechazados por intersección de alcance |
| **Human Intervention Rate** | F-G | paradas humanas por bloque cerrado |
| **Long-Horizon Success** | F-D + F-F | % de features completas sin pérdida de coherencia |

**KPI técnico principal propuesto:** *cuánto trabajo autónomo puede ejecutar un conjunto de agentes
sin perder coherencia del proyecto*. No número de funcionalidades.

---

## 18 · Frases que resumen la auditoría

Para reusar en el plan sin volver a razonarlas:

- **El motor es real; lo que impone no es lo que el producto promete.**
- Quipu demuestra que **el expediente está completo**, no que **el trabajo está bien**.
- «Evidencia tipada o no ocurrió» es hoy «evidencia **declarada** o no ocurrió».
- La capa normativa de F1 **está construida con rigor y no gobierna nada**.
- Para un proyecto brownfield, la cadena de gobierno de Quipu es **inerte**.
- Los triggers exigen correctamente un humano; **el perímetro deja que cualquiera lo sea**.
- El activo más valioso del ecosistema son **348 líneas de bash que están fuera del producto**.
- La verificación debe preceder a la coordinación: **el gate es lo que hace seguro paralelizar**.
- **Nada de esto es un rediseño.** El core es correcto y hay que conservarlo entero: hay que terminar el mecanismo en los tres puntos donde se interrumpe, y usar el producto para construirse a sí mismo.
