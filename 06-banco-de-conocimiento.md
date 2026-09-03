# 06 — Banco de conocimiento: cómo se registra el porqué

> **Fecha:** 2026-09-01 · **Naturaleza:** `RECOMENDACIÓN`. No es un plan: no fija fases, plazos ni
> asignaciones, y no modifica ningún fichero de Quipu ni de Chasqui.
>
> **Base de contraste:** `analisisExterno/auditoria-interna-2026-09-01.md` (hechos H-01…H-28),
> `chasqui_n8n/decisiones/` (25 archivos, medidos el 2026-09-01) y la capability `decision-chain`
> de Quipu cerrada en F1. Las etiquetas son las de `investigacion/README.md`.

---

## 0 · El problema

El trabajo de diseño de esta herramienta produce documentos. Los documentos conservan **el
resultado** de cada decisión y pierden **su origen**: qué se sabía en ese momento, qué alternativas
estaban sobre la mesa, sobre qué evidencia se apoyó la elección, y qué hecho nuevo obligaría a
revisarla.

La consecuencia no es documental, es operativa: una decisión sin origen registrado **no se puede
invalidar**. Cuando la evidencia que la sostenía deja de ser cierta, nadie lo sabe, porque el enlace
entre una y otra nunca se escribió.

---

## 1 · Diagnóstico

### 1.1 No falta formato

`chasqui_n8n/decisiones/` ya resuelve el formato, y con más rigor que cualquier plantilla ADR
estándar: frontmatter con invariantes explícitos, cadena `supersede`/`superseded_by` con
`motivo_reemplazo` obligatorio, `afecta`, `implementada_en`, **`procedencia` obligatoria**, cuatro
secciones fijas (`Problema medido` / `Decisión` / `Alternativas descartadas` / `Consecuencias`),
verificador estructural (`bin/verificar_decisiones.py`) y servidor MCP de consulta
(`bin/mcp_decisiones.py`, deliberadamente sin índice ni embeddings).

`HECHO` — 25 archivos, 18 promovidos desde `candidatos/` tras triage humano el 2026-08-19, ciclo de
vida documentado (`candidato → vigente → superada`, sin borrado nunca).

### 1.2 No falta modelo normativo en Quipu

`HECHO` (auditoría §7.1.6) — La cadena de decisión de F1 es **"el mejor trabajo del repositorio"**:
DAG sin ciclos, etiqueta epistémica `confirmado|inferido` con evidencia obligatoria, y promoción
firmada **contra la huella exacta del contenido** — promover un texto distinto del firmado es
imposible.

`HECHO` (H-06, H-07, H-08) — Y no gobierna nada. `TaskPayloadBuilder::forBlock` devuelve `block`,
`feature`, `dependencies`, `contracts`, `endpoints`, `screens`, `gate`; `grep -ni
"rule\|invariante\|decision"` sobre él da **cero coincidencias**. `block_rule` y
`rule_implementation` existen sólo en la migración que las creó. La detección de contradicción no
cruza alcance contra ancla, y abrir una contradicción es voluntario.

> `INFERENCIA` — Los dos sistemas son mitades del mismo. Chasqui tiene **norma sin imposición**: el
> verificador se salta no ejecutándolo. Quipu tiene **imposición sin norma entregada**: un `UPDATE`
> desde psql recibe el mismo rechazo que la API, pero el agente que reclama un bloque no ve ni un
> invariante. Ninguno de los dos tiene la tercera pieza, que es la evidencia sobre la que se decide.

### 1.3 Los tres huecos reales

| # | Hueco | Manifestación medida |
|---|---|---|
| **1** | **No hay capa de evidencia** | `investigacion/` clasifica ~200 afirmaciones (`HECHO`/`INFERENCIA` × `E1..E6`) y vive como prosa: nadie puede citarla desde una decisión ni marcarla como refutada. En Quipu, `knowledge_entry` no tiene estado epistémico mientras `invariante` sí lo tiene — **dos epistemologías en el mismo sistema** (H-26) |
| **2** | **No hay captura en el nivel meta** | Chasqui registra las decisiones *de Chasqui*. Las decisiones sobre el diseño de la herramienta sólo dejan `01`–`05` e `investigacion/`. La decisión clave del README —«evolucionar Quipu, no construir una herramienta nueva»— no es una fila en ningún sitio |
| **3** | **No hay caducidad** | Ninguna decisión declara cuándo debe revisarse. Un banco sobre agentes de IA en 2026 miente en 2027 si nada lo obliga a revisarse |

---

## 2 · Los tres registros

Cada uno responde una pregunta distinta y ninguno puede responder la del otro. La separación
`decisiones` / `agent-context` de Chasqui ya la tiene a medias; esto la completa con el tercero.

| Registro | Pregunta | Naturaleza | Estado hoy |
|---|---|---|---|
| **`conocimiento/`** | ¿qué sabemos, y con qué grado de evidencia? | **caduca** | no existe |
| **`decisiones/`** | ¿cómo hemos decidido que debe ser? | **gobierna** | existe en Chasqui; en Quipu existe el modelo y no se entrega |
| **grafo de código** | ¿cómo está hoy? | **se deriva** | `quipu:index` es heurística de sufijos, 0 filas (H-24) |

`RECOMENDACIÓN` — No fusionarlos nunca. Un grafo de código no puede decir qué está bien; una
decisión no puede decir qué existe; y una unidad de conocimiento no obliga a nada. Es la misma razón
por la que `mcp_decisiones.py` documenta en su docstring que la pregunta hermana la contesta otro
servidor.

---

## 3 · Esquemas

### 3.1 La unidad de conocimiento

Atómica: **una afirmación por archivo**, igual que una decisión por archivo.

```yaml
---
id: EV-CTX-004
tipo: HECHO            # HECHO | INTERPRETACIÓN | INFERENCIA   (convención ya vigente)
grado: E1              # E1..E6                                (convención ya vigente)
afirmacion: >
  Si la restricción de gobierno sobrevive al resumen, la violación es 0 % (n=90);
  si se elimina, 38 % (n=315)
fuente: "Chen, Governance Decay, arXiv:2606.22528v2, jun-2026"
observado: 2026-09-01
revisar_en: 2027-03-01
estado: vigente        # vigente | caducado | refutado
refutado_por: null
citada_por: [...]      # GENERADO por el indexador, nunca a mano
---

## Qué mide exactamente
## Qué NO dice
```

El apartado **«Qué NO dice»** —los límites que el propio autor declara— es lo que impide que dentro
de seis meses un agente use un número fuera de su alcance. `investigacion/00-metodo.md` ya lo hace a
nivel de documento; esto lo baja al nivel de afirmación, que es donde se cita.

### 3.2 Tres campos nuevos en la decisión, y un estado nuevo

Sobre el esquema que Chasqui ya usa, sin tocar nada de lo existente:

```yaml
se_apoya_en: [EV-CTX-004, EV-VERIF-011]   # ← el origen que hoy se pierde
falsador: "qué hecho nuevo obligaría a reabrir esta decisión"
revisar_en: 2027-03-01                     # calculado: el mínimo de sus evidencias
estado: pendiente | vigente | superada | descartada   # ← `pendiente` es nuevo
```

**`falsador`** es el de mayor rendimiento y casi nadie lo escribe. Convierte la decisión de registro
estático en algo con disparador: cuando `EV-CTX-004` pase a `refutado`, el verificador lista **todas**
las decisiones que se apoyaban en él. Eso es propagación de contradicción mecánica sobre el corpus
normativo — el mismo patrón que `fn_propagar_sospecha` (H-27) ya ejecuta sobre las entidades de
Quipu, aplicado a la norma.

**`estado: pendiente`** existe porque las «decisiones que necesitan al humano» (§14 de la auditoría,
DEC-1…DEC-8) hoy viven en un `.md` y se evaporan entre sesiones. Como fila del banco, con dueño y
fecha límite, el verificador puede exigirlas y el banco registra también lo que **está abierto**, no
sólo lo cerrado.

### 3.3 La regla de enlace

> **Ninguna decisión sin al menos una unidad de conocimiento citada, y ninguna decisión con más
> certeza que su fuente.**

Si una decisión se apoya en un `E5` (opinión, postmortem sin datos) o un `E6` (marketing), lo declara
y caduca pronto. El verificador lo comprueba: es la misma clase de chequeo que
`verificar_decisiones.py` ya hace sobre `supersede` y `procedencia`.

---

## 4 · El protocolo de captura

Aquí está el problema real. Los tres huecos del §1.3 son de contenido; éste es de proceso.

### 4.1 El principio de asimetría

> **Capturar debe costar casi nada. Promover es donde vive el rigor.**

Si capturar cuesta escribir un documento, no se captura — y es exactamente lo que está pasando. El
triage de Chasqui ya funciona así (18 promovidos de un lote extraído automáticamente); lo que falta
es la mitad de captura continua.

### 4.2 Los cuatro mecanismos

| Mecanismo | Cuándo | Coste | Qué rescata |
|---|---|---|---|
| **Candidato de 30 segundos** | cualquier agente, en cualquier momento: 5 campos (pregunta, opción elegida, descartadas, procedencia, fecha) en `candidatos/por_promover/` | trivial | lo que el agente sabe que decidió |
| **Hook de cierre de sesión** | al terminar, relee el transcript propio y emite candidatos por cada momento en que se eligió entre alternativas | automático | **lo que nadie recuerda escribir** |
| **Puerta de cierre** | no se cierra un bloque que tocó un artefacto gobernado sin citar la decisión que implementa, o abrir una nueva | mecánico | lo que se decidió implícitamente al escribir código |
| **Triage humano en lote** | semanal; sólo el humano promueve candidato → decisión | caro, y debe serlo | la calidad del corpus |

La puerta de cierre es la única que puede ser mecánica hoy: `is_block_closeable()` ya es el patrón
correcto —«la puerta y su explicación son el mismo objeto»— y la auditoría recomienda explícitamente
repetirlo en todo lo nuevo (§7.1.2). Requiere C5 (`artefacto_alcance`) para saber qué artefactos
tocó el bloque.

### 4.3 La salvaguarda innegociable

Un LLM releyendo un transcript **inventa razones que nadie dio**. Por eso:

> `procedencia` debe apuntar a algo localizable —transcript + fecha + cita textual, migración,
> commit— y el verificador **rechaza el candidato sin ella**.

La regla ya está escrita en `chasqui_n8n/decisiones/README.md` («sin procedencia no se puede
distinguir un razonamiento verificado de uno inferido»). Lo que falta es hacerla ejecutable en el
momento de la captura automática, que es cuando más barato es inventar.

---

## 5 · Dónde vive

`RECOMENDACIÓN` — **Tres ubicaciones canónicas, una regla de autoridad por contexto.**

| Contexto | Fuente de verdad | Por qué |
|---|---|---|
| **Chasqui** | `decisiones/` en archivos | Ya está dictaminado por `PROCESO-002` (2026-08-28) y es norma vigente del cliente. No se reabre |
| **El diseño de la herramienta** (este repo) | archivos | No hay instancia de Quipu que gobierne `developTool`, y estas decisiones **gobiernan al producto**: no pueden vivir dentro de él |
| **Proyectos cliente dentro de Quipu** | la BD | Es el producto. `invariante` + `decision` con firma contra huella |

Para los dos primeros, archivo y no base de datos, a propósito y en contra de la tesis general del
producto:

- Se revisa por **diff en un PR**; `git blame` da procedencia gratis; un agente lo lee **sin levantar
  un servicio**; y sobrevive a que el producto cambie.
- Lo único que el archivo no da —integridad referencial— ya lo cubre `verificar_decisiones.py` en CI.
  Ese es el trato completo, y basta.

`INFERENCIA` — El camino BD tiene hoy su propio agujero de integridad que refuerza esta elección:
`firma` es polimórfica y sin FK hacia lo firmado, y la base viva contiene **4 firmas huérfanas** sobre
decisiones borradas (H-09). Hasta C3, la BD no es más fiable que el archivo para este uso.

**Definir el esquema una sola vez** de forma que se exprese como frontmatter y como filas, y que el
mismo verificador sirva para los dos. Cuando C7 exista («la norma viaja con el trabajo»), el payload
de `claim_block` entrega lo mismo venga de donde venga.

---

## 6 · Caducidad y contradicción

Dos mecanismos, ninguno nuevo — los dos son patrones que ya existen en el ecosistema:

**Contradicción como consulta, no como vigilancia.** Antes de modificar un artefacto, se pregunta
qué decisiones lo afectan (`afecta:` + MCP). Exige que `afecta` sea exacto, y por eso el verificador
ya comprueba que cada ruta declarada existe. Es barato porque es una consulta puntual, no un
escaneo.

**Caducidad explícita.** Cada unidad de conocimiento lleva `revisar_en`; cada decisión hereda el
mínimo de las suyas. Una revisión periódica lista lo vencido. Sin esto, el banco acumula afirmaciones
que fueron ciertas y las presenta como si lo siguieran siendo — que es peor que no tenerlas, porque
tienen la forma de una fuente verificada.

---

## 7 · Qué NO entra

Un banco de conocimiento muere por volumen antes que por ausencia. Tres pruebas; si falla una, no es
decisión:

1. Hubo **elección** entre alternativas reales. Si sólo se consultó un dato, es conocimiento.
2. **Prohíbe** algo. Si no hay invariante que se pueda violar, es una nota.
3. Revertirla **cuesta** algo.

`HECHO` — Esta lección ya está aprendida y escrita en `candidatos/README.md`: de 73 migraciones, 38
transcripts y la memoria del agente salieron 18 decisiones; las 22 de `FORMA.md` se descartaron
porque *«escribir hoy un invariante sobre un teclado obliga a superseder una decisión para cambiar
una palabra — el coste sin el beneficio»*. Súbase a criterio explícito del triage.

Y nunca entra lo que se deriva del código: cómo está implementado hoy lo responde el grafo, y
duplicarlo produce dos verdades que divergen.

---

## 8 · Plan de arranque

`RECOMENDACIÓN` — **No empezar por construir el registro vacío.** La razón de que `decisiones/` de
Chasqui funcione y la capa normativa de F1 no es que una nació como subproducto del trabajo y la otra
como especificación previa. La auditoría lo mide como riesgo ALTA: *«cero dogfooding convierte el
roadmap en literatura»*, y los tres huecos estructurales *«se descubren en la primera hora de uso
real; ninguno estaba en `DEUDA-F2`»*.

El orden que se deduce:

| Paso | Qué | Material |
|---|---|---|
| **1** | Extraer las unidades de conocimiento de `investigacion/` | 13 documentos con ~200 afirmaciones **ya etiquetadas** `HECHO`/`E1`. Es casi mecánico |
| **2** | Extraer las decisiones ya tomadas de `01`–`05` y de la auditoría | Al menos dos evidentes: «evolucionar Quipu en vez de construir herramienta nueva» (el README la titula literalmente «Decisión clave registrada» y no es fila en ningún sitio) y «oracle determinista primero; juez estadístico después; nunca sin meta-evaluación» |
| **3** | Cargar las decisiones abiertas como `pendiente` | DEC-1…DEC-8 de la auditoría §14, con dueño y fecha |
| **4** | Encender la captura | Hook de cierre + candidato de 30 s, ya con el esquema probado contra material conocido |
| **5** | Puerta de cierre | Depende de C5 (`artefacto_alcance`). No antes |

El paso 2 da además la medida de cuánto se ha perdido: **cada decisión que haya que reconstruir sin
poder citar procedencia localizable es una que este sistema habría salvado.** Ese número es el
argumento para mantener el protocolo cuando estorbe.

---

## 9 · Cómo se conecta con el roadmap de la auditoría

Este banco no es trabajo adicional al margen del plan revisado: es la forma concreta de tres de los
ocho cambios recomendados.

| Cambio de la auditoría | Qué aporta el banco |
|---|---|
| **C7 · La norma viaja con el trabajo** | Define **qué** viaja: decisiones vigentes del dominio + invariantes que intersectan el alcance, con su evidencia y su estado epistémico. Sin el registro, C7 entrega un modelo de datos vacío |
| **H-26 · Unificar la epistemología** | `conocimiento/` **es** la unificación: `knowledge_entry` gana lo que `invariante` ya tiene (estado epistémico, evidencia obligatoria, supersede) en vez de convivir con dos epistemologías |
| **C5 · `artefacto_alcance`** | El banco es su primer consumidor no hipotético: `afecta:` deja de ser texto libre verificado por existencia y pasa a ser intersección de alcance |
| **Métrica *Decision Compliance*** | Sólo es calculable si existe el corpus de decisiones vigentes contra el que medir |

---

## 10 · Cómo se sabe que funciona

Criterios objetivos, no impresiones:

| Criterio | Umbral |
|---|---|
| Toda decisión vigente cita ≥1 unidad de conocimiento con procedencia localizable | 100 %, comprobado en CI |
| Ninguna decisión declara más certeza que su fuente | 100 %, comprobado en CI |
| Decisiones creadas por captura automática vs. escritas a mano | la automática debe superar a la manual en 3 meses, o el hook no sirve |
| Candidatos promovidos / candidatos emitidos | entre 15 % y 40 %. Por encima, se está capturando poco; por debajo, el ruido está ahogando el triage |
| Decisiones con `revisar_en` vencido | 0 al cierre de cada revisión periódica |
| Reconstrucciones sin procedencia en el arranque | se mide una vez, en el paso 2, y es la línea base |

---

## 11 · Riesgos

| Riesgo | Gravedad | Mitigación |
|---|---|---|
| **Procedencia inventada por extracción automática** | CRÍTICA | Cita textual localizable obligatoria; el verificador rechaza sin ella. Es el riesgo que puede envenenar el corpus entero y volverlo peor que no tenerlo |
| **Volumen: el banco se vuelve ilegible** | ALTA | Las tres pruebas del §7 aplicadas en el triage, y `FORMA.md` como precedente de qué se descarta |
| **El protocolo estorba y se abandona** | ALTA | La asimetría del §4.1: si capturar cuesta más de 30 segundos, se abandonará. Medirlo con la métrica de captura automática vs. manual |
| **Tercer registro que nadie consulta** | MEDIA | `conocimiento/` sólo se justifica si las decisiones lo citan. Si a los 3 meses hay unidades con `citada_por` vacío, sobran |
| **Duplicar la norma en archivo y BD** | MEDIA | Una fuente canónica por contexto (§5), declarada por escrito, y sincronización en una sola dirección |
