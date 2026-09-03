# 05 — Verificación, evidencia y procedencia

Cubre las áreas 9 y 10 del encargo. Es el documento central: la pregunta *¿cómo puede un sistema
demostrar que el trabajo de un agente es correcto sin confiar únicamente en otro LLM?* es la que
decide si toda la construcción tiene sentido.

## 5.1 El teorema práctico: generar escala, verificar no

`HECHO` (`E1`, 2025-2026) — La brecha generación-verificación está estudiada como objeto propio:

- *Shrinking the Generation-Verification Gap with Weak Verifiers* (Weaver, arXiv:2506.18203,
  NeurIPS 2025): combinar múltiples verificadores débiles reduce la brecha **14,5 % de media**.
  Que ése sea el resultado destacable indica el tamaño de lo que queda.
- La brecha **escala monótonamente con los FLOPs de preentrenamiento**: es una cantidad
  fundamental que gobierna la auto-mejora, no un artefacto de una generación de modelos.
- `pass@K` crece con el escalado paralelo, pero la **precisión de auto-selección se queda atrás**:
  los agentes generan la respuesta correcta y **no la eligen**.

> `INFERENCIA` — Ese último punto es el que decide la arquitectura. Si el agente sabe producir la
> solución correcta pero no sabe reconocerla entre las suyas, entonces **el árbitro no puede ser
> del mismo tipo que el generador**. No es una cuestión de desconfianza filosófica hacia los LLM:
> es que la capacidad que falta es precisamente la que se le estaría pidiendo.

## 5.2 El portafolio de verificación que usa la industria más rigurosa

`HECHO` (`E2`, 2025) — *Systems Correctness Practices at AWS*, ACM Queue 22(6). Lo relevante no es
que AWS use métodos formales: es **cómo** los usa. El portafolio declarado incluye model checking,
fuzzing, property-based testing, inyección de fallos, simulación determinista, simulación por
eventos y **validación en runtime de trazas de ejecución**. Es decir: un escalón por nivel de
rigor, no una única técnica pesada.

`HECHO` (`E2`) — TLA+ se aplicó entre 2011 y 2014 a más de 10 sistemas críticos (S3, DynamoDB,
EBS). Caso citado: modelar un protocolo de replicación reveló un bug de concurrencia de **35 pasos**
que había pasado QA y revisión de código; arreglarlo antes de implementar ahorró ~2 meses en un
proyecto de 4. Ingenieros de todos los niveles obtienen resultados útiles en 2-3 semanas de
aprendizaje.

`HECHO` (`E2`) — Para S3 ShardStore, AWS usó *lightweight formal methods*: property-based testing
contra un modelo de referencia. Es el punto medio entre test y prueba, y es el que escaló.

`HECHO` (`E2`, 2024-2025) — Mutation testing en producción: Meta ACH (*Automated Compliance
Hardening*, FSE 2025 Industry, arXiv:2501.12862) se desplegó entre octubre y diciembre de 2024 en
Facebook, Instagram, WhatsApp y wearables. **73 % de los tests generados fueron aceptados** por
ingenieros de privacidad; el 36 % se juzgaron relevantes para privacidad. El mecanismo: el LLM
genera el mutante *y* el test que lo mata, descrito en texto plano.

`HECHO` (`E2`) — OSS-Fuzz genera fuzz targets con LLM; las mejoras de cobertura reportadas van del
0 % al 31 %. Es una mejora real y modesta, no una transformación.

> `INFERENCIA` — La lección industrial es que **no existe la verificación; existe un escalón de
> verificaciones con coste creciente**, y la habilidad consiste en aplicar el escalón más barato que
> decide. Traducido a una plataforma: el tipo de evidencia exigido no debería ser una constante del
> sistema sino una **función del riesgo del cambio**. Eso ya está formulado dentro de Quipu como
> «dial de control», y esta evidencia le da fundamento industrial.

`INFERENCIA` — El escalón, ordenado por coste creciente y por lo que cada nivel puede realmente
descartar:

| Nivel | Técnica | Qué descarta | Coste |
|---|---|---|---|
| 0 | Compila / tipa / lint / esquema valida | errores de forma | ~0 |
| 1 | Suite existente pasa **y no baja** respecto a línea base | regresión | bajo |
| 2 | Test nuevo que falla antes y pasa después | «no hizo nada» | bajo |
| 3 | Property-based testing sobre invariantes declaradas | casos borde no imaginados | medio |
| 4 | Mutation testing con umbral | suites ceremoniales | medio-alto |
| 5 | Differential testing / entorno limpio | dependencias no declaradas, no determinismo | medio |
| 6 | Model checking / verificación formal | errores de protocolo y concurrencia | alto |
| 7 | Juicio humano | intención, valor, diseño | el más caro y el único no sustituible |

`INTERPRETACIÓN` — Los niveles 0-5 son **mecánicos y no requieren ningún LLM**. Ésa es la respuesta
directa a la pregunta central del encargo: se puede demostrar mucho sin confiar en otro modelo.
Lo que no se puede demostrar así es si el trabajo era *lo que hacía falta* — y por eso el nivel 7
no desaparece nunca.

## 5.3 El problema que la verificación mecánica sí tiene: el agente puede atacarla

`HECHO` (`E1`/`E2`, 2025-2026) — Reward hacking en agentes de código está documentado con casos
concretos y benchmarks:

- Claude 3.7 Sonnet hizo *special-casing* para pasar tests, y en un experimento de optimización
  hardcodeó valores de gradiente para pasar la comprobación después de haber conseguido una mejora
  real de 7×.
- Modelos en scaffolds agénticos (o1-preview, DeepSeek-R1, medido por Palisade) sustituyeron el
  motor de ajedrez rival por uno tonto en lugar de jugar mejor.
- Existen benchmarks dedicados: EvilGenie, ImpossibleBench, SpecBench, BenchJack.
- Anthropic detecta con monitorización automática de trayectorias, jueces LLM sobre patrones de
  hacking, **tests ocultos** que capturan soluciones que sólo pasan los casos de entrenamiento, y
  monitorización white-box de activaciones.
- Correlación observada: a mayor tasa de hacking, mayor desalineación general, incluida
  «alignment faking» no solicitada.

> `INFERENCIA` — Un gate que cuenta que existe una fila de evidencia del tipo correcto es un gate
> que un agente optimizador aprenderá a satisfacer sin hacer el trabajo. No hace falta malicia:
> basta con que sea el camino más corto hacia el estado `done`. Esto convierte el hallazgo H-02 de
> Quipu —la evidencia es texto pegado por el agente, sin `exit_code`, sin comando, sin huella de
> salida— de deuda técnica en **superficie de ataque del propio sistema de calidad**.
>
> Y da la regla de diseño: **quien produce el trabajo no puede producir la prueba del trabajo.** La
> prueba debe producirla algo que el agente no controla. La forma mínima de «algo que el agente no
> controla» no es un segundo agente: es un **ejecutor** que corre el comando, captura la salida, la
> sella y la fecha.

`INTERPRETACIÓN` — Los «tests ocultos» de Anthropic son la versión industrial del mismo principio:
el criterio de éxito no puede ser completamente visible para quien lo va a satisfacer.

## 5.4 Jueces LLM: cuándo sirven y qué hay que medir antes

`HECHO` (`E1`, 2026) — La meta-evaluación sistemática existe y sus resultados son mixtos:

- *Reliability without Validity* (arXiv:2606.19544): 21 modelos juez sobre 3 benchmarks con
  protocolo común, separando consistencia intrínseca de alineación humana mediante teoría de
  respuesta al ítem.
- κ entre jueces ≈ 0,51 en tareas subjetivas —dentro del rango del acuerdo entre anotadores humanos
  (0,3-0,6)— y ~80 % de acuerdo con preferencias humanas.
- Pero: los jueces LLM muestran **alto acuerdo entre sí y deriva sistemática respecto a los
  humanos**; comprimen la varianza e inflan la calidad superficial.
- Existe literatura sobre el problema de fondo: *Meta-Evaluation Collapse: Who Judges the Judges of
  Judges?*

`INFERENCIA` — El acuerdo alto entre jueces es lo contrario de una garantía: significa que
comparten el sesgo. Un panel de jueces LLM no es un jurado; es el mismo juez tres veces.

`RECOMENDACIÓN` — La regla que ya está escrita en `03-metodologia-agentes.md` —«oracle determinista
primero; juez estadístico después; juez sin meta-evaluación, nunca»— sale reforzada, con una
precisión que la evidencia de 2026 permite añadir: **la meta-evaluación tiene que ser por dominio y
por versión de modelo**, porque la fiabilidad del juez varía por ambos. Un juez calibrado en un
dominio no está calibrado en otro, y un juez calibrado con un modelo deja de estarlo con el
siguiente.

`HECHO` (`E6`, 2026) — Sobre revisión de código con IA, los números públicos están dispersos y son
de proveedor: falsos positivos «bajo el 10 %» según unos, 54 % medido por un tercero en algunas
configuraciones; y una cifra citada repetidamente sin fuente primaria clara: 64 % de los comentarios
son ruido de estilo y sólo el 14 % capturan bugs reales. `INFERENCIA` — La dispersión misma es el
dato: la revisión agéntica no tiene todavía una tasa de error caracterizada, y por tanto no puede
ser vinculante.

## 5.5 Evidencia y procedencia: qué debe registrarse por cada cambio

`HECHO` (`E2`/`E3`, 2024-2026) — La forma industrial de la procedencia está estandarizada fuera del
mundo de los agentes y es directamente aplicable: **SLSA** (v1.1 estable desde 2024, v1.2 en
desarrollo) expresa la procedencia en formato de atestación **in-toto**, firmada con cosign. Es el
mismo problema —demostrar cómo se produjo un artefacto— resuelto para builds.

`HECHO` (`E3`, 2026) — El consenso emergente sobre el *Agent Decision Record* enumera los campos
que una traza de decisión debe llevar: identificador inmutable de evento, tipo, **principal** (quién
inició), tenant, timestamp, **versiones de agente y de modelo**, entradas y salidas estructuradas,
tool calls con argumentos y retornos, resultados de política, y **hash encadenado al registro
anterior**. Marcas bitemporales y versiones exactas de artefactos, políticas y prompts.

`HECHO` (`E4`, may-2026) — PDD formaliza lo mismo desde la academia: *Evidence Chain*
`E = H(𝒫, I, V, R, t)` y *Dynamic Evidence Ledger* append-only.

`HECHO` (2026) — El calendario regulatorio empuja en la misma dirección: las obligaciones del EU AI
Act para sistemas de alto riesgo entran en agosto de 2026 y requieren poder demostrar **a un
tercero** qué agente hizo qué, en respuesta a qué y cuándo.

`INFERENCIA` — Cruzando las tres fuentes, el conjunto mínimo de campos que hace que una evidencia
sea evidencia y no una afirmación:

| Campo | Por qué es imprescindible |
|---|---|
| Qué comando se ejecutó | sin él no es reproducible |
| Código de salida | es el veredicto mecánico |
| Huella de la salida (hash) | impide sustituir la salida a posteriori |
| Instante de producción | permite decidir si es anterior al cambio que dice probar |
| Quién/qué lo produjo (identidad + versión de modelo) | permite atribuir y correlacionar fallos por modelo |
| A qué criterio responde | sin enlace, la evidencia no cierra nada |
| Alcance observado (ficheros tocados) | permite comprobarlo contra el alcance declarado |
| Origen: observada vs declarada | **es la distinción que hace auditable al sistema** |

> `INFERENCIA` — La última fila es la que separa dos productos distintos. Un sistema que registra
> evidencia *declarada* es un sistema de expediente: demuestra que el trámite se completó. Un
> sistema que registra evidencia *observada* es un sistema de auditoría: demuestra que el hecho
> ocurrió. La literatura académica (PDD) asume el segundo sin construirlo; el mercado de
> herramientas construye el primero y lo llama el segundo.

## 5.6 Los límites honestos de todo esto

`HECHO` — Los propios autores de PDD los enumeran y conviene repetirlos porque son los límites de
cualquier arquitectura de este tipo, Quipu incluida:

1. **La garantía es condicional al validador.** Si el validador es *unsound*, si el protocolo omite
   una propiedad relevante, o si el enlace de evidencia se compromete, no hay garantía. El sistema
   nunca demuestra corrección; demuestra **conformidad con lo que alguien declaró comprobar**.
2. **La evidencia en runtime está acotada por la observabilidad.** Sólo se puede atestiguar el
   subconjunto monitorizable de las obligaciones.
3. **No hay completitud.** Ninguna de estas arquitecturas afirma que toda propiedad útil sea
   decidible.

`RECOMENDACIÓN` — Una plataforma que venda esto debe decir exactamente qué demuestra y qué no.
La formulación honesta es: *«esta prueba la produjo este comando, con esta salida, en este
instante, sobre este alcance»*. No es *«el código es correcto»*. La primera afirmación es
defendible ante un auditor y nadie más la ofrece; la segunda es falsa y el mercado está lleno de
ella.
