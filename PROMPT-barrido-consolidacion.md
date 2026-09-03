# Encargo: barrido de consolidación para completar Quipu

> Copia todo lo que sigue —desde «CONTEXTO» hasta el final— como primer mensaje de una sesión nueva.

---

## CONTEXTO

Trabajas sobre tres repositorios en `/mnt/datos/Programacion/`:

| Ruta | Rol |
|---|---|
| `QUIPU_ENTERPRISE/` | **El producto.** Laravel 12 + PostgreSQL 16 + React 19. El plano de control |
| `QUIPU_ENTERPRISE/sistema-a/` | **La metodología.** Paquete de ejecución agéntica (FLOTA), git independiente |
| `developTool/` | **El diseño.** Plan por fases, investigación profunda y auditorías. No es repo git |

**Objetivo general del proyecto** (el que hay que cumplir, no re-discutir): convertir Quipu Enterprise
en la infraestructura que coordina humanos y agentes de IA en todo el ciclo de vida del software de
forma **confiable, verificable, auditable y sostenible a largo plazo**, con destino comercial SaaS.
El primer y único caso real de uso es **el propio Quipu**: el producto gobierna su propio desarrollo
(dogfooding). La tesis registrada: *Git responde qué cambió; Quipu responde qué estaba permitido
cambiar, con qué autoridad y con qué prueba — y sólo cree en pruebas que vio producir.*

**Estado al 2026-09-01:** F0 y F1 cerradas, ninguna fase activa. Existen ya, hechos y cerrados:
una investigación profunda del estado del arte 2026 (14 documentos), una auditoría interna medida
contra el código y la base viva, un análisis estratégico externo, cinco documentos de plan y una
metodología ejecutable (`sistema-a`) con verificador determinista propio.

---

## FUERA DE ALCANCE: Chasqui — léelo antes que nada

`chasqui_n8n` **no forma parte de este trabajo, en ningún sentido y bajo ninguna forma.** No es el
cliente, no es el caso de prueba, no es fuente metodológica y no es criterio de priorización.

Esto importa porque **los documentos que vas a leer están llenos de Chasqui**: fue el destino
declarado del plan original y buena parte de las prioridades, las evidencias y las fases se
escribieron mirándolo. Tienes que leer esos documentos y **filtrarlo activamente**. Reglas:

1. **Nada de lo que produzcas lo menciona** salvo, si hace falta, para dejar constancia de que un
   elemento del origen se descartó por depender de él.
2. **No abres el repositorio `chasqui_n8n` ni ninguno de sus ficheros.** Tampoco lo usas como
   referencia metodológica aunque las fuentes lo elogien.
3. **Todo elemento cuya única justificación sea Chasqui se descarta.** Ejemplos: la fase F2 / puente
   Chasqui y su reescritura como F-D2; la decisión `DEC-2` (aplazar F2); el bloque `§A` de
   `DEUDA-F2.md`; cualquier prioridad cuyo argumento sea «desbloquea Chasqui».
4. **Todo elemento que Chasqui sólo *ilustraba* sobrevive, pero hay que re-justificarlo sin él.**
   Un mecanismo general de producto no deja de ser necesario porque el ejemplo que lo motivaba salga
   de alcance — pero si al quitar el ejemplo se queda sin ninguna evidencia que lo sostenga, eso es
   un hallazgo y hay que declararlo así, no rellenarlo con plausibilidad.
5. **Reclasifica lo que quede huérfano.** Varias prioridades `CRÍTICA` lo eran por su efecto sobre
   Chasqui. Al retirarlo, la prioridad de esos elementos cambia: dilo, con el criterio nuevo.
6. **Los IDs no se renumeran.** `DEC-2` no desaparece de la tabla: se marca `fuera de alcance por
   decisión del propietario`. Renumerar rompería la trazabilidad con los documentos de origen.
7. **El caso real que sustituye a Chasqui es el propio Quipu.** Donde una fuente razone sobre
   «gobernar un proyecto externo», evalúa si el argumento se sostiene con Quipu gobernándose a sí
   mismo. Si se sostiene, consérvalo con esa lectura; si no, descártalo.

---

## QUÉ TE PIDO

Un **barrido de consolidación**: reunir todo lo que esos documentos proponen, dispersos y con nombres
distintos, en **un solo inventario sin duplicados** y en **una sola metodología global** que Quipu
pueda adoptar para cumplir el objetivo general — y decir, con evidencia, **qué falta todavía** y
**qué nadie ha mirado**.

**Esto NO es el plan.** El plan lo escribe una sesión posterior. Si tu entregable contiene fases,
plazos, estimaciones, asignaciones o tareas, has hecho el trabajo equivocado.

El valor que se te pide no está en resumir lo que ya está escrito. Está en cuatro operaciones que
todavía nadie ha hecho:

1. **Unificar** — el mismo mecanismo aparece como `C5` en la auditoría, `M1` en la investigación,
   `P4` en la tesis y como convención en los ficheros de tarea de `sistema-a`. Son uno. Hace falta
   una tabla de equivalencias para que nadie lo cuente cuatro veces ni lo implemente dos.
2. **Contrastar existencia** — cada elemento debe clasificarse: *ya está en el producto*, *existe
   fuera del producto* (bash de `sistema-a`, convención en ficheros de tarea), *no existe*. Dos de
   los seis mecanismos críticos ya existen fuera del producto; probablemente haya más.
3. **Depurar el legado de Chasqui** — separar, en cada fuente, el mecanismo general del argumento
   que lo ataba a un cliente que ya no está en alcance. Ver la sección anterior.
4. **Detectar huecos ciegos** — lo que ninguno de los documentos cubre porque todos comparten el
   mismo punto de vista. Es la parte más difícil y la más valiosa del encargo.

---

## REGLAS DURAS

1. **No modificas nada** de `QUIPU_ENTERPRISE` ni de `sistema-a`. Sólo escribes tu entregable dentro
   de `developTool/`.
2. **No re-auditas.** El §0 de `analisisExterno/auditoria-interna-2026-09-01.md` lista lo que ya está
   medido y no hay que repetir (inventariar tools, contar migraciones, leer las 62 migraciones, correr
   el CI, sondear el perímetro). Reutiliza esos hechos como suelo. Puedes hacer comprobaciones
   puntuales de código para verificar existencia de un elemento concreto que la auditoría no cubra —
   pero cada una debe traer su comando o su referencia `fichero:línea`.
3. **No resuelves las decisiones del humano.** Las decisiones clase C (§14 de la auditoría,
   `DEC-1`…`DEC-8`, menos `DEC-2` que queda fuera de alcance) siguen abiertas. Donde un elemento
   dependa de una, exprésalo como condicional («si `DEC-3` = a, entonces…»), no elijas por él.
4. **No escribes plan.** Ver arriba.
5. **Etiquetas de evidencia obligatorias**, las que ya usan los documentos:
   `HECHO` (medido, con comando o `fichero:línea`) · `INTERPRETACIÓN` · `INFERENCIA` ·
   `RECOMENDACIÓN` · `DESCONOCIDO`. Y para fuentes externas, el grado `E1`…`E6` de
   `investigacion/README.md`. Un `E6` nunca sostiene una recomendación.
6. **Cero relleno.** Los documentos existentes son densos y sin paja. Iguala ese registro: tablas
   antes que prosa, una afirmación por línea, nada de resúmenes ejecutivos que repitan el cuerpo.
7. **Todo en español.**
8. Si algo no se puede determinar, se escribe `DESCONOCIDO` y se lista. No se rellena con plausibilidad.

---

## QUÉ LEER, EN ESTE ORDEN

**Bloque 1 — el estado real (empieza aquí, es lo único medido contra el código):**
- `developTool/analisisExterno/auditoria-interna-2026-09-01.md` — completo. Presta atención especial
  a §4 (hechos `H-01`…`H-26`), §7 (diagnóstico del core), §9 (ocho cambios `C1`–`C8`), §11
  (dependencias), §12 (qué NO desarrollar), §14 (decisiones), §16 (arquitectura objetivo), §17
  (métricas). **Su §5 entero trata de Chasqui: queda fuera de alcance.** El §10 (roadmap `F-A`…`F-H`)
  se lee sólo como origen de elementos, no como orden — y su `F-D2` sale.

**Bloque 2 — el estado del arte y la tesis:**
- `developTool/investigacion/` — los 14 documentos. Si el presupuesto aprieta, el orden de valor es
  `11` (tesis y conjunto mínimo), `10` (comparación área por área, `M1`–`M6` y §10.4 «qué NO
  incorporar»), `05` (verificación y procedencia), `08` (los 27 modos de fallo), `01`, y luego el
  resto. Es la fuente más limpia de contaminación por cliente: úsala como eje.

**Bloque 3 — la metodología viva:**
- `QUIPU_ENTERPRISE/sistema-a/INDICE.md`, `AGENTS.md`, `NORMATIVA/` (CONSTITUCION, ESCALAMIENTO,
  VALIDACION, HANDOFF), `ESTADO/ESTADO.md`, `ESTADO/CIERRE-F1.md`.
- `ESTADO/DEUDA-F2.md` — pese al nombre, su contenido `§B`–`§E` es deuda general del producto y vale.
  **Su `§A` queda fuera** (bloqueantes de la importación del cliente).
- `sistema-a/bin/` — 1.236 líneas de bash (`verificar.sh` 348, `metricas.sh` 264, `checkpoint.sh` 215,
  `estado.sh` 193, `sombra.sh` 175, `flota.sh` 41). **Léelas de verdad**: la auditoría afirma que el
  activo más valioso del ecosistema está ahí y fuera del producto. Tu barrido tiene que decir qué
  parte de esas líneas es producto, qué parte es andamiaje y qué parte sobra.
- `sistema-a/TAREAS/F1/T02-migraciones-normativas.md` como ejemplar del formato de tarea vigente.
- `sistema-a/.session/`, `ARCHIVO/` y `evidencia/` — el historial de sesiones y la evidencia real
  producida. Sirve para medir qué se usó de verdad y qué se escribió y nunca se ejecutó.

**Bloque 4 — el encargo original y el marco:**
- `developTool/README.md` y los documentos `01`…`05`. Histórico válido, **con la prioridad de
  absorber un cliente externo ya retirada**: `03-metodologia-agentes.md` sigue vigente entero; de
  `04` y `05` valen los principios, no el orden; `02-plan-fases.md` está parcialmente obsoleto y
  además ordenado alrededor de Chasqui — extráele elementos, nunca secuencia.
- `developTool/analisisExterno/InformeEstratégicoQuipu.md` — marco estratégico válido, puntuaciones
  refutadas por el §8 de la auditoría. Úsalo para el eje de producto/mercado, no para evaluar estado.

---

## ENTREGABLE

Un directorio nuevo, `developTool/consolidacion/`, con seis documentos y su índice.

### `README.md`
Punto de entrada: qué es esto, qué no es (no es plan), cómo leerlo, y las convenciones de etiquetas.
Declara en cabecera que Chasqui está fuera de alcance y que el caso real es el propio Quipu.

### `00-metodo-y-alcance.md`
Cómo hiciste el barrido, qué leíste, qué comprobaste contra código y qué diste por bueno de la
auditoría. Sesgos y límites del propio barrido. Qué queda explícitamente fuera. Incluye la
**lista de elementos descartados por depender de Chasqui**, con la fuente de cada uno: es la prueba
de que el filtrado se hizo y no se perdió nada por accidente.

### `01-inventario-unificado.md` — el núcleo del encargo
El catálogo maestro de **todo** lo que se ha propuesto en cualquier documento, deduplicado. Una fila
por elemento, y ningún elemento repetido bajo dos nombres. Para cada uno:

| Campo | Contenido |
|---|---|
| **ID canónico** | El que tú fijes; será el que use el plan posterior |
| **Nombre** | Corto y sin ambigüedad |
| **Qué problema resuelve** | Una frase |
| **Procedencia** | Todas las fuentes que lo proponen, con documento y sección (`auditoría §9 C5`, `investigación §10.3 M1`, …) |
| **Estado de existencia** | `en el producto` / `fuera del producto (dónde)` / `no existe` — con evidencia |
| **Punto de imposición** | Dónde vive la regla: trigger Postgres / PHP / runner / CI / documento / convención humana |
| **Evidencia que produce** | Qué queda en el expediente cuando se ejecuta. Si no produce nada, dilo |
| **Qué desbloquea** | Otros elementos del inventario que dependen de éste |
| **De qué depende** | Elementos y decisiones `DEC-n` |
| **Clasificación** | `crítico` / `importante` / `opcional` — reutiliza los criterios de la auditoría §11.1, no inventes escala nueva. Si la clasificación de origen dependía de Chasqui, márcala como **reclasificada** y di por qué |
| **Complejidad** | baja / media / alta, y por qué |

Abre el documento con la **tabla de equivalencias** que reconcilia las nomenclaturas: `C1`…`C8`
(auditoría §9) ↔ `M1`…`M6` (investigación §10.3) ↔ `P1`…`P5` (tesis §11.2) ↔ `H-01`…`H-26` (hechos) ↔
`DEUDA-F2 §B–E` ↔ las capabilities OpenSpec ↔ las reglas de `CONSTITUCION.md`. Sin esa tabla el resto
del inventario no es auditable.

Incluye también, porque hasta ahora sólo se ha inventariado el mecanismo y no el resto:
- **herramientas** (del ecosistema y propias) que habría que adoptar, con el criterio de adopción o
  descarte ya fijado en `03-metodologia-agentes.md` §4;
- **prácticas metodológicas** que hoy son convención humana y deberían ser mecánicas, o al revés;
- **métricas** (auditoría §17) tratadas como elementos de primera clase: cada una con qué la hace
  medible y qué mecanismo la produce;
- **superficies de interfaz** que faltan (web, MCP, CLI) para que un mecanismo sea usable de verdad.

### `02-metodologia-global.md` — la unificación que pide el encargo
Una sola metodología, no tres solapadas. Hoy conviven: la cadena de demanda y la capa normativa del
producto, el paquete de ejecución `sistema-a`/FLOTA y las prácticas del estado del arte 2026.
Fusiónalas en **un ciclo canónico único**, describiendo para cada etapa:

- qué entra y qué sale;
- quién puede ejecutarla (humano / agente / máquina) y con qué autoridad;
- **qué la impide avanzar** si no se cumple, y dónde vive esa imposición;
- qué evidencia deja;
- qué documento o mecanismo actual la cubre hoy, total o parcialmente.

Cierra con:
- **la versión mínima operable** — el subconjunto más pequeño de esa metodología que un humano solo
  con una flota de agentes puede ejecutar a diario sin ahogarse. La sobre-especificación es un riesgo
  real y declarado; hay que decir cuál es el mínimo que sigue siendo íntegro;
- **el destino de `sistema-a`** — qué partes se absorben en el producto, cuáles siguen siendo paquete
  externo y cuáles se retiran. La auditoría §13 marca como riesgo ALTO que el paquete de metodología
  se convierta en el producto: tu barrido tiene que pronunciarse sobre eso con evidencia;
- **cómo se gobierna Quipu a sí mismo con esta metodología** — el dogfooding es ahora el único caso
  real, así que la metodología tiene que ser ejecutable sobre el propio repositorio del producto;
- **cómo se ve la metodología desde fuera** — qué tiene que adoptar un equipo que no la escribió.

### `03-cobertura-y-huecos.md`
Dónde está el sistema respecto a lo que debería ser:

- **matriz de cobertura contra los 27 modos de fallo** de `investigacion/08-taxonomia-de-fallos.md`:
  para cada uno, si Quipu lo previene, lo detecta, lo registra o no lo ve, y con qué mecanismo;
- **matriz de cobertura contra las cinco responsabilidades** de la arquitectura objetivo
  (auditoría §16): verdad operativa, contexto acotado, procedencia verificable, autoridad, autonomía
  con expediente;
- **huecos ciegos**: lo que ningún documento cubre. Búscalos deliberadamente, al menos en estos
  frentes, y añade los que encuentres:
  arranque y adopción de un repositorio existente de punta a punta (empezando por el de Quipu) ·
  experiencia del agente (los 46 tools MCP ya exceden el presupuesto de atención) ·
  experiencia del humano que aprueba (la cola de firmas es el cuello de botella cuando esto escale) ·
  soporte multi-proveedor de agente y qué pasa cuando cambia el modelo ·
  ciclo de vida del propio dato de gobierno (respaldo, restauración, migración, retención del rastro) ·
  cómo se evalúa el producto a sí mismo (suite de regresión del gobierno, no del código) ·
  coste e instrumentación · recuperación cuando el gobierno se equivoca y bloquea trabajo legítimo ·
  qué ocurre con el trabajo hecho fuera de Quipu, que siempre existirá;
- **contradicciones entre fuentes**: donde dos documentos afirman cosas incompatibles, cuál manda y
  por qué. Regla de arbitraje: el código medido gana a la documentación, y la documentación medida
  gana a la opinión externa.

### `04-no-hacer.md`
La lista consolidada de lo que **no** debe construirse, reconciliando `investigacion §10.4` y
`auditoría §12`, con la razón de cada punto y su grado de evidencia. Marca cuáles son innegociables
(automatizar actos de autoridad, jueces LLM vinculantes sin meta-evaluación, duplicar estado
transaccional) y cuáles son «no ahora» con la condición que los reabriría. Este documento es tan
importante como el inventario: su función es impedir que el plan posterior crezca.

### `05-preguntas-abiertas.md`
- Las decisiones del humano ya conocidas (`DEC-1`…`DEC-8`), actualizadas si tu barrido cambia su
  alcance, con `DEC-2` marcada `fuera de alcance`, y **las nuevas** que el barrido destape — cada una
  con opciones, qué depende de ella y qué recomienda la evidencia sin decidirla.
- Los `DESCONOCIDO`: lo que no pudiste determinar y qué haría falta para determinarlo.

---

## CRITERIOS DE ACEPTACIÓN

El entregable se rechaza si:

- menciona Chasqui como cliente, caso de prueba, fuente metodológica o criterio de prioridad;
- arrastra un elemento cuya única justificación era Chasqui sin marcarlo como descartado;
- contiene fases, fechas, estimaciones o asignaciones de tarea;
- algún elemento del inventario aparece dos veces bajo nombres distintos, o falta la tabla de
  equivalencias;
- alguna fila del inventario carece de procedencia citada (documento + sección);
- alguna fila afirma «no existe» sin haber comprobado `sistema-a/bin/` y el esquema desplegado;
- resuelve por su cuenta cualquier decisión clase C;
- modifica algún fichero fuera de `developTool/consolidacion/`;
- confunde grado de evidencia: una recomendación sostenida en `E6`, o un `HECHO` sin comando.

Se acepta cuando la sesión siguiente puede escribir el plan **leyendo sólo tu entregable y el §14 de
la auditoría**, sin volver a abrir los 20 documentos de origen.

---

## CÓMO EMPEZAR

1. Comprueba que el mapa de rutas de arriba sigue siendo cierto y di si algo se movió desde
   el 2026-09-01.
2. Lee el bloque 1 completo antes de tocar nada más.
3. Cuando tengas el inventario en borrador, **antes de redactar**, enséñame dos cosas: la tabla de
   equivalencias con los IDs canónicos, y la lista de elementos descartados por depender de Chasqui.
   Si la unificación o el filtrado están mal ahí, todo lo demás hereda el error.
4. Pregúntame lo que necesites. Prefiero una pregunta a una suposición documentada como hecho.
