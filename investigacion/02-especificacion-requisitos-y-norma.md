# 02 — De la necesidad informal a la norma ejecutable

Cubre las áreas 1 y 2 del encargo: requirements engineering con agentes, y especificaciones como
fuente de verdad.

## 2.1 El estado real del requirements engineering con agentes

`HECHO` (`E1`, 2025-2026) — Hay trabajo experimental serio, pero acotado a un solo problema:
**detectar ambigüedad**.

- Estudio industrial (IEEE, 2025-2026, *Requirements Ambiguity Detection and Explanation with
  LLMs*): dataset híbrido con requisitos reales del James Webb Space Telescope más defectos
  inyectados sistemáticamente. Resultado central: incorporar **conocimiento de dominio
  auto-extraído** permite al modelo distinguir la ambigüedad genuina de la ambigüedad aceptable; sin
  él, marca como ambiguo lo que el dominio ya desambigua.
- arXiv:2604.21505 (2026) mide el efecto de la ambigüedad de requisitos sobre la generación de
  código a nivel de función: el enlace ambigüedad → defecto es directo y medible.
- arXiv:2604.18228 (2026) formaliza requisitos de especificaciones no estructuradas hacia
  propiedades temporales evaluables por *model checking*.

`HECHO` (`E2`, 2026) — En producto: Kiro (AWS) usa notación **EARS** (`WHEN <condición> THE SYSTEM
SHALL <comportamiento>`) para criterios de aceptación, y en 2026 añadió *Requirements Analysis* con
lógica formal y **solvers SMT para detectar contradicciones antes de generar código**.

`INFERENCIA` — La detección de contradicción entre requisitos ha dejado de ser un problema de
juicio para convertirse, al menos parcialmente, en un problema decidible: si los requisitos se
expresan en una forma restringida, un solver contesta. Esto importa mucho para Quipu, cuyo H-08
dice que la detección de contradicción hoy es juicio de un LLM y además voluntaria.

`INTERPRETACIÓN` — El resto del ciclo de requisitos —elicitación, priorización, negociación con el
usuario— **no tiene resultados experimentales publicados con agentes**. Hay herramientas y hay
entusiasmo; no hay medición. El encargo pedía distinguir esto y hay que decirlo con claridad: quien
afirme hoy que los agentes hacen buena elicitación está haciendo `E5` o `E6`.

## 2.2 Spec-driven development: adopción alta, evidencia baja

`HECHO` (`E2`, 2025-2026) — Existe y está ampliamente adoptado: GitHub **Spec Kit** (MIT, neutral
respecto al agente: Claude Code, Copilot, Gemini, Cursor), AWS **Kiro**, BMAD-METHOD, y en el
ecosistema OpenSpec —el que ya usa Quipu.

`HECHO` (`E6`) — Las cifras de eficacia que circulan son todas de proveedor y sin protocolo:
«orden de magnitud menos ciclos de regeneración desde cero» (GitHub), «funcionalidades de 40 h
entregadas en menos de 8 h de tiempo humano» (AWS), «3-10× más tasa de acierto a la primera»
(agregadores). **Ninguna sostiene una decisión.**

`HECHO` (`E3`, 2026) — La crítica sí es convergente y viene de sitios independientes: **los specs
derivan del código en horas**, y la sincronización spec ↔ código no está automatizada en ninguna de
las herramientas. La pregunta que la crítica formula —«¿cuál es el ciclo de vida del spec?»— es
previa a elegir herramienta.

> `INFERENCIA` — El valor demostrado del spec-driven no está en el documento sino en el **acto de
> forzar acuerdo antes de generar**. Y su fallo demostrado es que el documento envejece. Cualquier
> arquitectura que ponga la especificación como fuente de verdad sin un mecanismo de invalidación
> está construyendo el problema que la crítica ya identificó. Quipu tiene ese mecanismo —huella
> SHA-256 y propagación de sospecha, H-27— y aparentemente no sabe lo raro que es.

## 2.3 Protocol-Driven Development: la formalización más cercana a Quipu

`HECHO` (`E4`, may-2026) — He & Yu, *Protocol-Driven Development: Governing Generated Software
Through Invariants and Continuous Evidence* (arXiv:2605.12981v3). Define un protocolo como una
tripleta de invariantes:

| Clase | Qué fija | Mecanismo de imposición |
|---|---|---|
| **𝒮 estructurales** | tipos, esquemas, firmas de interfaz | typed handshakes: JSON Schema, OpenAPI, protobuf |
| **ℬ conductuales** | predicados sobre comportamiento observable | property-based assertions (p. ej. `f(f(x)) = f(x)`) |
| **𝒪 operacionales** | llamadas externas, uso de dependencias, disco, red, latencia, memoria | capability manifests: sandbox, motor de políticas, instrumentación |

La conjunción define el conjunto de implementaciones admisibles. Sobre eso:

- **Discovery Log**: registro *as-built* de lo que se produjo y observó.
- **Evidence Chain**: `E = H(𝒫, I, V, R, t)` — enlaza protocolo, implementación, validación,
  resultado y tiempo bajo un hash.
- **Dynamic Evidence Ledger**: apéndice inmutable `ℒt = ℒt-1 ‖ Et`.
- **Puerta de admisión**: `Validate(I, 𝒫) ∈ ℰ ∪ {⊥}` en tres capas secuenciales —estructural,
  conductual, operacional— con requisito explícito de que el validador sea **sound**.

Lema del paper: *«Code is transient; protocol is sovereign.»*

`HECHO` — **El paper no presenta ni un solo número.** Tiene casos de estudio ilustrativos (un
handler idempotente de creación de usuario, un pipeline ETL acotado, un microservicio de detección
de fraude) y una *agenda* de evaluación futura: reducción de ambigüedad, fiabilidad de
regeneración, sobrecoste de validación, sustituibilidad. Los autores lo dicen: «PDD requiere una
agenda empírica».

`HECHO` — Limitaciones declaradas por los propios autores, todas relevantes: si el validador es
*unsound*, si el protocolo omite una propiedad, o si el enlace de evidencia se compromete, la
garantía desaparece; la evidencia en runtime está acotada por la observabilidad (`Ω𝒫ʳ ⊆ Ω𝒫`); la
regeneración segura sólo vale para dependencias que se atengan al protocolo, no a comportamiento
accidental.

> `INFERENCIA` — **Esto es la tesis de Quipu escrita como paper académico, cuatro meses antes de
> esta investigación, por gente que no conoce Quipu.** Coinciden: invariantes de primera clase,
> evidencia continua como artefacto, cadena de evidencia con hash, ledger append-only, puerta que
> admite o rechaza. Divergen en un punto y es a favor de Quipu: PDD no tiene implementación ni
> medición, y Quipu tiene 118 tablas, 49 triggers y 562 tests corriendo.
>
> `INFERENCIA` — Y coinciden en el hueco: la garantía de PDD depende de que el validador **observe
> de verdad**. El paper lo asume; Quipu, según H-01 y H-02, tampoco lo hace todavía. Los dos tienen
> el mismo agujero en el mismo sitio, lo que es una confirmación bastante potente de que el agujero
> es el que importa.

`INTERPRETACIÓN` — La aportación conceptual de PDD que Quipu **no** tiene explícita es la tercera
clase de invariante: los **operacionales**. Quipu modela normas de dominio y criterios de
aceptación; no modela «este bloque no puede tocar la red», «no más de una escritura por request»,
«no puede instalar dependencias». Es una categoría entera de restricción que un agente viola sin
que ninguna puerta se entere.

## 2.4 Qué debería conservarse, qué estructurarse, qué permanecer humano

`INFERENCIA` a partir de §2.1-2.3 y del material de gobierno (doc 06):

**Debe conservarse literal e inmutable:** el enunciado original de la necesidad en las palabras del
usuario. Toda la evidencia sobre ambigüedad (§2.1) depende de poder volver al texto original; una
necesidad reescrita por un agente ya perdió la información que permite detectar que se
malinterpretó.

**Debe estructurarse:** lo que una máquina va a evaluar. Criterios de aceptación en forma
restringida (EARS o dado/cuando/entonces), invariantes con ancla verificable, alcance como conjunto
de rutas y símbolos, y el enlace entre ellos. La razón no es elegancia: es que sólo lo estructurado
puede alimentar un solver, una puerta o un diff.

**Debe permanecer humano:** la decisión de qué se construye, la aceptación de un cambio de norma, y
la resolución de una contradicción entre normas. No por principio moral sino por dos hechos
medidos: los jueces LLM derivan sistemáticamente de los humanos en tareas de valor (§5.4), y la
autoridad afirmada dentro del stream de tokens es falsificable (§4.2).

**Cómo detectar ambigüedad:** conocimiento de dominio explícito + comprobación de contradicción
mecánica sobre la forma restringida. La evidencia dice que sin dominio explícito el detector produce
falsos positivos que entrenan al humano a ignorarlo (§2.1) — y eso es el mismo mecanismo de la
fatiga de aprobación (§6.2).

**Cómo validar que el agente entendió:** no preguntándole. Haciendo que produzca un artefacto
verificable —el criterio ejecutable, el test que falla antes y pasa después, el alcance declarado—
y comparándolo mecánicamente con lo especificado. La confirmación verbal de comprensión no tiene
valor probatorio: es exactamente la clase de salida que un modelo produce igual de convincente
cuando entendió y cuando no.

## 2.5 Trazabilidad y análisis de impacto

`HECHO` (`E3`, 2026) — En la práctica industrial, la trazabilidad requisito→código se sostiene hoy
sobre tres cosas: identificadores estables, enlaces explícitos declarados en el momento del cambio,
y el diff. No hay evidencia de que la recuperación automática de enlaces *a posteriori* funcione a
escala.

`HECHO` (`E5`, 2026) — Sobre ADRs y agentes, la observación repetida por varios equipos: «un agente
que no puede ver **por qué** algo se construyó así, refactorizará alegremente la razón». Y el límite
conocido: un ADR generado por un agente escaneando el código captura **qué** se decidió pero
**fabrica el porqué**.

> `INFERENCIA` — Esto valida directamente el modelo de decisión de Quipu (cadena supersede,
> `confirmado|inferido` con evidencia obligatoria, promoción firmada contra la huella del contenido)
> y explica por qué H-06 —que el payload de `claim_block` no lleva ni una norma— es un fallo grave y
> no una omisión menor: es literalmente el escenario que la práctica describe como el que destruye
> arquitectura.
