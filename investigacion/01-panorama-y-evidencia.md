# 01 — Panorama 2026: qué está medido y qué no

## 1.1 El desplazamiento del objeto de estudio

`HECHO` (`E4`, abr-2026) — El survey más reciente sobre agentes en el ciclo de vida del software
(Bhati, *Agentic AI in the Software Development Lifecycle*, arXiv:2604.26275) formula el cambio así:
el objeto central de estudio pasó de **generación de código** a **ejecución delegada bajo
supervisión humana**. Propone una arquitectura de referencia de seis capas:

| Capa | Contenido | Madurez declarada |
|---|---|---|
| L0 | Modelo base | alta |
| L1 | Razonamiento, memoria, autorreflexión | media |
| L2 | Agent–Computer Interface | «empíricamente tan importante como el tamaño del modelo» |
| L3 | Tools y entorno (fs, shell, tests, compilador, VCS, CI/CD) | alta |
| L4 | Orquestación (bucle único o multi-agente) | media |
| L5 | **Gobierno y seguridad** (permisos, sandbox, audit logs, políticas) | **la menos madura** |

`INFERENCIA` — Que la capa L5 sea la menos madura según un survey académico, y que sea exactamente
donde Quipu opera, es el dato de posicionamiento más importante de toda esta investigación. No
significa que Quipu esté adelantado; significa que el hueco es reconocido y todavía no está ocupado.

## 1.2 Los benchmarks: qué miden y por qué ya no bastan

`HECHO` (`E1`) — Progresión de SWE-bench Verified: 1,96 % (oct-2023, RAG) → 12,5 % (nov-2024,
SWE-agent) → 33,2 %/49 % (principios 2025) → 62,3 %/72,7 % (mediados 2025) → 78,4 % (abr-2026).
Los sistemas **no agénticos** se estancaron cerca del 20 %.

`INTERPRETACIÓN` (Bhati 2026) — «La ganancia está dominada por el scaffolding, no por la capacidad
bruta del modelo». Es una afirmación fuerte y bien sostenida por los datos: el mismo modelo dentro
de un andamiaje distinto cambia decenas de puntos.

`HECHO` (`E2`, feb-2026) — OpenAI **dejó de usar SWE-bench Verified** por contaminación. Los
motivos publicados y verificables:

- 32,67 % de los parches exitosos presentan *solution leakage*: la solución aparece en el texto del
  issue o en sus comentarios.
- Los modelos recuerdan rutas de fichero del entrenamiento hasta un 76 % de las veces, frente a un
  53 % para ficheros externos.
- Auditoría manual de 138 fallos de o3: **59,4 % causados por tests defectuosos**, no por límites
  del modelo.

`HECHO` (`E1`, sep-2025) — SWE-bench Pro (arXiv:2509.16941) responde con 276 tareas de 18
repositorios comerciales privados que no están en internet. UTBoost (arXiv:2506.09289) documenta
por separado la insuficiencia de los tests del benchmark original.

`HECHO` (`E1`, ene-2026) — METR *Time Horizon 1.1*: la suite pasó de 170 a 228 tareas y las tareas
de más de 8 h de 14 a 31. El tiempo de duplicación del horizonte de tarea es de 196 días en el
periodo completo 2019-2025, pero **131 días desde 2023 y 89 días desde 2024**. Horizonte al 50 %:
Claude Opus 4.5 ≈ 320 min; GPT-5 ≈ 214 min; o3 ≈ 121 min.

`HECHO` — METR declara sus límites: intervalos de confianza muy anchos, sensibilidad de la
tendencia a la composición de tareas, y **sólo 5 de las 31 tareas largas tienen línea base humana
medida** (el resto son estimaciones). No discute la brecha con el trabajo real desordenado.

`HECHO` (`E1`, abr-2026) — El framework HORIZON (arXiv:2604.11978, UW-Madison/Berkeley/GaTech,
3.100+ trayectorias sobre GPT-5 y Claude-4) mide *dónde* se rompe el trabajo largo, no sólo cuándo:
**72,5 % de los fallos son de proceso** (PFMEA) y sólo 27,5 % de diseño. La caída no es gradual:
«transición abrupta de robustez parcial a fallo casi sistemático». Conclusión de los autores:
*escalar el modelo base por sí solo es poco probable que resuelva los mecanismos de fallo
dominantes*.

> `INFERENCIA` — Los tres hechos anteriores, juntos, dicen algo que ninguno dice solo: la capacidad
> crece rápido y de forma medible, la medición es frágil, y lo que rompe el trabajo largo es
> **proceso**, no inteligencia. Una plataforma que apueste a que «el modelo del año que viene lo
> arreglará» está apostando contra el 72,5 %.

## 1.3 Productividad y calidad: los dos resultados que no encajan

`HECHO` (`E1`, jul-2025) — RCT de METR (arXiv:2507.09089): 16 desarrolladores experimentados, 246
tareas reales en repositorios propios que llevaban ~5 años manteniendo. Con herramientas de IA
permitidas tardaron **19 % más**. Antes del estudio pronosticaban −24 %; después de vivirlo
estimaban −20 %. La brecha entre percepción y medición es de unos 40 puntos porcentuales.

`HECHO` (`E2`) — Estudios de campo del mismo periodo reportan lo contrario en agregado: +12,92 % a
+21,83 % de PRs por semana (Microsoft), +7,51 % a +8,69 % (Accenture), −55,8 % de tiempo en una
tarea controlada de servidor HTTP (Peng et al.).

`HECHO` (`E2`, 2025) — DORA, *State of AI-assisted Software Development*, ~5.000 profesionales: la
adopción de IA correlaciona **positivamente con el throughput y positivamente con la
inestabilidad** —más fallos de cambio, más retrabajo, ciclos más largos de resolución. Tesis
central: la IA es un **amplificador**; acelera a los equipos con base sólida y magnifica el caos en
los demás. El tiempo ahorrado en creación se reasigna a auditoría y verificación.

`HECHO` (`E2`, 2026) — GitClear, *The Maintainability Gap*: 623 millones de cambios analizados entre
2023 y 2026, ocho señales de calidad. Frente a 2022: llamadas a función entre ficheros **−35 %**,
líneas movidas por refactorización **−70 %**, mantenimiento de legado a largo plazo **−74 %**. En
sentido contrario: copy/paste dentro del commit **+41 %**, duplicación de bloques **+81 %**,
construcciones que enmascaran errores **+47 %**, churn a dos semanas **+15 %**. 2024 fue el primer
año registrado en que el copy/paste dentro del commit superó al código movido.

`HECHO` (`E2`, ene-2026) — GitClear también mide que los usuarios intensivos de IA producen 4-10×
más que los no usuarios, pero **la mayor parte de esa brecha precedía a la IA**; contra su propio
pasado la ganancia es ~25 %.

> `INFERENCIA` — La lectura conjunta es coherente y bastante deprimente: **el sistema produce más
> volumen, con menos estructura, y traslada el coste a la verificación y al mantenimiento.** No es
> un problema de modelos malos; es lo que ocurre cuando abaratas la producción sin abaratar la
> revisión. Es exactamente el «cuello de botella de verificación» que ya estaba nombrado en
> `03-metodologia-agentes.md`, ahora con cuatro años de datos de repositorios detrás.

`HECHO` (`E5`, abr-2026) — Anthropic, *2026 Agentic Coding Trends Report*: los desarrolladores usan
IA en ~60 % de su trabajo pero declaran poder **delegar completamente sólo el 0-20 %** de las
tareas. (Es dato de encuesta de un vendedor: `E5`, no `E1`.)

`INTERPRETACIÓN` — Esa «brecha de delegación» es el mismo fenómeno visto desde el usuario: la parte
que no se delega es la que no se puede verificar barato.

## 1.4 La economía

`HECHO` (`E2`/`E5`, 2026) — Órdenes de magnitud recogidos:

- Coste por tarea de codificación: entre ~$0,03 y ~$0,13 en el benchmark polyglot de Aider según
  modelo.
- Los flujos agénticos consumen entre 5× y 30× más tokens por tarea que un chatbot: 15.000-80.000
  tokens por tarea completada frente a 500-2.000 de un Q&A.
- Uber pasó del 32 % al 84 % de adopción de Claude Code entre dic-2025 y mar-2026 sobre ~5.000
  ingenieros; en abril el presupuesto anual de IA estaba consumido, con $500-$2.000 por ingeniero y
  mes. (`E5` — reportado, no auditado.)
- Sistemas multi-agente: ~15× más tokens que una interacción de chat (Anthropic, 2025, `E2`).

`INFERENCIA` — El coste no es lineal en el trabajo entregado sino en el trabajo **intentado**,
incluidos los reintentos. Cualquier plataforma que despache trabajo a agentes sin una puerta que
rechace barato está pagando por generación que después descartará. Esto convierte al *gate* en una
palanca económica, no sólo de calidad.

## 1.5 Qué está resuelto, qué no, y qué viene

`INFERENCIA` a partir de todo lo anterior:

**Resuelto o casi (consenso + evidencia):**
- Un agente puede editar código en un repositorio real con un bucle read/search/edit/execute y
  producir parches útiles. Está medido en 13 implementaciones independientes.
- Aislar la ejecución es un problema conocido con soluciones estándar (microVM, gVisor, contenedor).
- Paralelizar por worktrees evita la clase de conflicto más tonta.
- Los benchmarks públicos se contaminan; hay que renovarlos o privatizarlos.

**No resuelto (hay propuestas, no resultados):**
- Verificar que el trabajo es correcto sin un humano y sin otro LLM. Es *el* problema abierto.
- Mantener la coherencia arquitectónica a lo largo de miles de cambios.
- Detectar que el conocimiento almacenado quedó obsoleto (mejor modelo frontera: 55,2 %, §4.3).
- Que la norma organizativa sobreviva a la compactación de contexto sin mecanismos externos.
- Evaluar un agente por algo que no sea «pasó el benchmark».

**Probablemente vendrá (proyección, `INFERENCIA`):**
- Presión regulatoria sobre trazabilidad: las obligaciones del EU AI Act para sistemas de alto
  riesgo entran en agosto de 2026 y piden demostrar a un tercero qué agente hizo qué y cuándo.
- Estandarización de la capa de identidad y delegación de agentes (ya hay propuestas: A2A v1.0 con
  Agent Cards firmados, abr-2026; AIP para delegación verificable).
- Fragmentación y luego consolidación de la capa de gobierno, que hoy no existe en los protocolos.
