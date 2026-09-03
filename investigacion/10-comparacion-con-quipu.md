# 10 — Comparación explícita con Quipu

> **Base de contraste:** el estado de Quipu Enterprise medido el 2026-09-01 en
> `analisisExterno/auditoria-interna-2026-09-01.md` (hechos H-01…H-28). Esta investigación **no
> re-auditó nada**; sólo dice qué opina la evidencia externa sobre cada punto.
>
> **Esto no es un plan.** No hay orden, fases ni esfuerzo. La clasificación indica **importancia
> según la evidencia**, no urgencia ni secuencia.

## 10.1 Escala de clasificación

| Clase | Significado |
|---|---|
| **CRÍTICO** | Sin esto, una promesa que Quipu ya hace es falsa, o un fallo con evidencia experimental queda sin defensa |
| **IMPORTANTE** | Cierra un hueco que la evidencia muestra que aparece con el uso; el sistema funciona sin ello pero degrada |
| **ÚTIL** | Mejora medible o probable, sin consecuencia estructural si falta |
| **EXPERIMENTAL** | La evidencia es prometedora pero insuficiente; adoptarlo hoy es apostar |
| **NO NECESARIO** | El problema es real en el campo pero **no lo es para Quipu**, o la solución mínima ya existe por otra vía |

---

## 10.2 La tabla

### A · Requisitos y especificación

| Área | Estado actual de Quipu | Prácticas encontradas | Gap | Clase |
|---|---|---|---|---|
| Captura de necesidad | `necesidad` con causa registrada; triaje; `NextQuestion` | Conservar el enunciado original literal; el resto no tiene evidencia experimental (§2.1) | Ninguno sustantivo. El catálogo de entrevista pregunta lo que el repositorio contesta (auditoría §7.3) | **ÚTIL** (recortar, no ampliar) |
| Especificación ejecutable | criterios dado/cuando/entonces **como filas**; cada escenario cita su test | Spec Kit, Kiro/EARS; crítica convergente: los specs derivan del código en horas (§2.2) | Quipu está por delante: la spec nace como dato, no como documento | — (ventaja) |
| Detección de ambigüedad | no existe | Conocimiento de dominio explícito mejora la discriminación (`E1`); SMT para contradicciones en Kiro (`E2`) | Hueco real, con solución conocida y acotada | **ÚTIL** |
| Detección de contradicción | `VerificarContradicciones` **no cruza alcance contra ancla**; abrir contradicción es voluntario (H-08) | Intersección mecánica; obligación de pronunciarse; solvers para el caso decidible (§2.1) | El modelo normativo existe y no se comprueba contra el trabajo | **CRÍTICO** |
| Invariantes operacionales | no existen (sólo normas de dominio) | PDD: la tercera clase de invariante —red, disco, dependencias, recursos— (§2.3) | Categoría entera de restricción que ninguna puerta ve | **IMPORTANTE** |
| Trazabilidad necesidad→código | cadena completa **dentro** de Quipu; **inerte para lo adoptado** (H-10, H-11) | Identificadores estables + enlaces declarados en el momento del cambio (§2.5) | El puente adopción↔demanda | **CRÍTICO** |

### B · Arquitectura, planificación y coordinación

| Área | Estado actual de Quipu | Prácticas encontradas | Gap | Clase |
|---|---|---|---|---|
| Topología de agentes | el producto **no llama a ningún LLM** (H-01); FLOTA es manual | Multi-agente cuesta ~15×; gana sólo si el trabajo se parte sin partir decisiones (§3.1) | Ninguno. La neutralidad es la posición correcta | — (ventaja) |
| Descomposición del trabajo | bloques → microtareas con criterios y evidencia exigida | Jerarquía con replanificación confinada al nodo activo (§3.3) | Las 6 propiedades de tarea autónoma se cumplen salvo *alcance declarado* | **IMPORTANTE** |
| Asignación / despacho | sin dispatcher, sin tabla de sesión (H-23) | Fuente única de verdad + leases (§3.4) | Hueco consciente. La evidencia dice que **despachar antes de verificar multiplica el problema** | **NO NECESARIO** hoy |
| Coordinación multiagente | claim libre de carrera por índice único; **sin lease ni heartbeat** (H-22) | Un fichero un dueño; lease TTL ~5 min + heartbeat + reclamación (§3.4) | Un worker muerto bloquea indefinidamente | **IMPORTANTE** |
| Conflicto semántico | no detectable | Sólo se previene con norma común impuesta por un tercero (§3.4, D2) | Es el caso de uso que justifica la existencia de Quipu, y hoy no lo cubre | **CRÍTICO** (vía alcance + norma) |
| Git como sustrato | ninguna integración; el alcance no existe como dato (H-13) | Worktree por agente = línea base; Git resuelve conflictos *a posteriori* | Falta la primitiva que permite detectar el conflicto **al reclamar** | **CRÍTICO** |

### C · Contexto, memoria y código

| Área | Estado actual de Quipu | Prácticas encontradas | Gap | Clase |
|---|---|---|---|---|
| Contexto por tarea | `claim_block` devuelve entregables, criterios, contratos, endpoints, pantallas — **ninguna norma** (H-06) | El contexto debe derivarse del alcance, acotado y con procedencia (§4.1) | La norma no viaja con el trabajo | **CRÍTICO** |
| Decaimiento de gobierno | no hay defensa, pero tampoco hay exposición: Quipu **impone fuera del contexto** | Compactación borra política: 30 % → 78 % de violación; pinning lo lleva a 0 % (§4.2) | Ninguno en el motor. El gap es que Quipu **no entrega** la norma que después impondría | **CRÍTICO** (mismo que arriba) |
| Memoria de agente | no la tiene, y es correcto | Números de benchmark no comparables (49 % vs 94 % para el mismo sistema) (§4.3) | Ninguno | **NO NECESARIO** |
| Invalidación de conocimiento | `fn_mantener_huella` (SHA-256) + `fn_propagar_sospecha` por sentencia, en 8 tablas (H-27) | Detección por el agente: **55,2 %** en el mejor modelo (STALE) | Quipu tiene construido lo que el mercado no tiene. El gap es que **no cubre el repositorio**, sólo entidades de Quipu | **IMPORTANTE** |
| Epistemología | dos: `invariante` con `confirmado\|inferido` + evidencia exigida; `knowledge_entry` sin nada (H-26) | Estado epistémico obligatorio; no almacenar lo no invalidable (§4.3) | Unificar | **IMPORTANTE** |
| Comprensión del código | `quipu:index` heurístico por sufijo de nombre; 0 filas (H-24) | Grep para un salto, grafo para varios; cifras de mejora casi todas `E6` (§4.4) | Para **gobernar** bastan 3 consultas de grafo pequeño sobre alcance↔artefacto | **ÚTIL** (grafo completo: **EXPERIMENTAL**) |
| Capa de diseño (UI como datos) | ~10 tablas, 0 filas, `ImpactAnalyzer` dependiente | Los agentes leen el código; lo que necesitan son las restricciones, no una réplica | Confirmado por evidencia externa | **NO NECESARIO** |

### D · Verificación y evidencia

| Área | Estado actual de Quipu | Prácticas encontradas | Gap | Clase |
|---|---|---|---|---|
| Evidencia observada | `add_evidence` guarda texto pegado; sin comando, exit_code, huella ni instante (H-02) | Comando + exit_code + huella + instante + versión de modelo; SLSA/in-toto; PDD Evidence Chain (§5.5) | **El gap central de todo el producto** | **CRÍTICO** |
| Puerta de evidencia | cuenta filas del tipo correcto; nunca inspecciona contenido (H-03) | Un gate que cuenta formas es un gate que se aprende a satisfacer (§5.3) | La puerta es superficie de reward hacking | **CRÍTICO** |
| Escalón de verificación | tipos declarativos (12 códigos), ninguno ejecutado (H-04) | Portafolio escalonado AWS; mutation testing en producción con 73 % de aceptación (§5.2) | El DoD tipado ya soporta añadir tipos: falta el productor | **IMPORTANTE** |
| Suite-diff / línea base | existe **fuera del producto**: `bin/verificar.sh`, 348 líneas (H-05) | Regresión medida en agregado (DORA); comparar conjuntos de tests, no porcentajes | El activo más valioso del ecosistema está fuera del producto | **CRÍTICO** |
| Jueces LLM | no existen, y hay test que vigila que no exista tool de aprobación | κ≈0,51; alto acuerdo entre jueces = sesgo compartido; meta-evaluación por dominio **y por versión** (§5.4) | Ninguno. La ausencia es correcta | — (ventaja) |
| Procedencia y linaje | firma inmutable contra huella; **firma polimórfica sin FK, 4 huérfanas** (H-09) | Hash encadenado, registro append-only, versión de modelo obligatoria (§5.5) | El rastro de autoridad puede apuntar a la nada | **CRÍTICO** |
| Verificación formal | no existe | AWS: portafolio escalonado; TLA+ útil en 2-3 semanas de aprendizaje | Fuera de alcance para una plataforma de gobierno: es del proyecto cliente | **NO NECESARIO** |

### E · Gobierno, autoridad y seguridad

| Área | Estado actual de Quipu | Prácticas encontradas | Gap | Clase |
|---|---|---|---|---|
| Perímetro | `POST /api/auth/session` emite credenciales **sin autenticar** (H-14); puertos en todas las interfaces (H-19) | Es la premisa de la que depende el valor de todo lo demás (§6.3) | Toda la cadena de autoridad es refutable con un `curl` | **CRÍTICO** |
| Autorización | 4 de 128 rutas con guardia de capacidad (H-16); gestión de miembros sin autorización (H-17) | Política en la capa de llamada, no en la del agente; default-deny (Cedar/OPA) (§6.3) | Un agente puede fabricar al humano que firma su trabajo | **CRÍTICO** |
| Separación de deberes | `fn_firmante_humano`, `fn_segregacion_funciones`, sin tool de aprobación, abilities de agente que excluyen firmar (H-18) | La autoridad afirmada en tokens es falsificable; debe vivir fuera (§4.2, §6.5) | Ninguno en el diseño; el gap es que sólo 4 rutas lo consultan | — (ventaja + **CRÍTICO** en el perímetro) |
| Fatiga de aprobación | no medida | 82 % → 45,5 % de precisión con IA incorrecta; filtrar mecánicamente antes de la cola humana (§6.2) | No hay métrica de si las paradas humanas cambian algo | **IMPORTANTE** |
| Sandboxing | el producto no ejecuta nada (H-01) | microVM / gVisor / contenedor endurecido (§6.3) | Aparece **sólo si** se construye un ejecutor de evidencia. Entonces es requisito, no opción | **CRÍTICO** condicional |
| Prompt injection | no aplica hoy (no hay LLM en el producto) | Rastreo de procedencia + restricción a acciones no consecuentes (CaMeL) (§6.3) | Aplicaría a los agentes clientes, no a Quipu | **NO NECESARIO** (para el núcleo) |
| Protocolos | MCP como transporte; 46 tools (excede el presupuesto de atención) | MCP/A2A **no pueden expresar** gobierno; se propone una capa por encima (§6.4) | Ninguno: es exactamente la posición de Quipu. El gap es de consolidación de tools | **ÚTIL** |
| Identidad de agente | miembros y tokens con abilities | Identidad no humana por agente; delegación verificable; intersección de permisos (§6.5) | Cubierto en lo esencial | **ÚTIL** |

### F · Ejecución, observabilidad y evaluación

| Área | Estado actual de Quipu | Prácticas encontradas | Gap | Clase |
|---|---|---|---|---|
| Motor de estados | bloques/estados/criterios/dependencias como filas con triggers | Ejecución durable: journal por paso, estado en el store, worker reemplazable (§7.1) | **Precedente sólido confirmado.** Quipu y los motores durables son duales | — (validado) |
| Reanudación y leases | sin lease, sin heartbeat, sin timeout (H-22) | TTL + heartbeat + reclamación automática | Un worker muerto retiene el bloque para siempre | **IMPORTANTE** |
| Observabilidad | errores 422/403 con mensaje del trigger; sin trazas de agente | OTel GenAI **pre-estable**; lo que hay que observar es el *acto*, no el modelo (§7.2) | Emitir sí; **depender del esquema ajeno, no** | **ÚTIL** |
| Versión de modelo en evidencia | no se registra | Regresión silenciosa al migrar; sin el campo no se puede correlacionar (§7.3) | Es un campo, no un proyecto | **IMPORTANTE** |
| Métricas de gobierno | ninguna instrumentada (D-01: métricas ciegas desde el port) | 10 métricas propuestas en §7.3 | Sin medición no hay evidencia de que el producto funcione | **IMPORTANTE** |
| Niveles de autonomía | L1-L4 definidos en FLOTA como documento | Deben ser **consulta** sobre historial, no configuración (§7.4) | Requiere historial de evidencia observada | **ÚTIL** (depende de la evidencia observada) |

### G · Evolución del sistema

| Área | Estado actual de Quipu | Prácticas encontradas | Gap | Clase |
|---|---|---|---|---|
| Deriva arquitectónica | no hay fitness functions; hay deriva **de esquema** ya ocurrida y no detectada (H-21) | Refactor −70 %, duplicación +81 % en 623M cambios; fitness functions como único sensor viable (§9.2) | Doble: el proyecto cliente y el propio Quipu | **IMPORTANTE** |
| Modelo normativo | dos coexistiendo; Chasqui perdió 20 de 63 invariantes al importar | Dos modelos normativos garantizan el fallo A5 (conflicto de contexto) | Elegir uno | **IMPORTANTE** |
| Rediseño barato | no existe como tipo de trabajo | Los agentes prefieren el parche local porque el rediseño es caro (§9.2) | Hueco que la literatura tampoco resuelve | **EXPERIMENTAL** |
| Vía rápida con deuda | `fn_cambio_deuda` + `deuda_vencida`, con expediente y plazo | Es la tercera respuesta legítima a la saturación de verificación | **Nadie más lo tiene.** Argumento comercial infrautilizado | — (ventaja) |

---

## 10.3 Principio de mínima complejidad aplicado

Las siete preguntas del encargo, sobre los seis mecanismos que la tabla clasifica como CRÍTICOS.
Se responde en corto porque el valor está en la disciplina, no en la extensión.

### M1 · Alcance como dato

- **Qué problema resuelve:** que no se puede comprobar nada sobre un cambio si no se sabe qué toca.
- **Evidencia de que es real:** MAST 13,98 % de desajuste razonamiento-acción; conflicto de ficheros como fallo #1 del paralelismo (`E3`); intersección con normas imposible sin él (H-08).
- **Por qué Quipu lo necesita:** es simultáneamente el insumo de **cinco** capacidades distintas —puerta de verificación, contexto por tarea, detección de contradicción, propiedad entre agentes, análisis de impacto—. Sin él las cinco son imposibles; con él, las cinco son consultas.
- **Solución mínima:** una tabla que ligue unidad de trabajo → rutas y símbolos, declarada al planificar y comparada contra el diff al evidenciar.
- **Complejidad que introduce:** una tabla, dos comprobaciones. Baja.
- **Alternativa más simple:** inferir el alcance del diff *a posteriori*. Se descarta: eliminaría la detección de conflicto al reclamar y la entrega de norma antes de trabajar, que son la mitad del valor.
- **¿Fase posterior?** No, porque es la dependencia de todo lo demás. Es la primitiva de mayor apalancamiento de esta investigación.

### M2 · Evidencia observada

- **Qué problema resuelve:** que el sistema no distingue una prueba de una afirmación.
- **Evidencia:** reward hacking documentado con casos concretos (`E1`/`E2`); PDD asume la observación y no la construye; el 59,4 % de fallos por tests defectuosos en un benchmark de referencia muestra lo lejos que puede llegar la falsa verificación.
- **Por qué Quipu lo necesita:** porque ya promete lo contrario. «Evidencia tipada o no ocurrió» es hoy «evidencia declarada o no ocurrió».
- **Solución mínima:** cinco columnas en `evidence` (`origen`, `comando`, `exit_code`, `huella_salida`, `producido_en`) y un ejecutor que las rellene; la puerta puede exigir `observada`.
- **Complejidad:** el ejecutor es un componente nuevo y trae consigo el requisito de sandboxing. Media-alta, y es la única de esta lista que lo es.
- **Alternativa más simple:** firmar la evidencia con la clave del agente. Se descarta: acredita quién la escribió, no que ocurriera.
- **¿Fase posterior?** No. Sin ella, todas las demás garantías son de forma.

### M3 · La norma viaja con el trabajo

- **Qué problema resuelve:** que la capa normativa no alcanza al trabajo.
- **Evidencia:** ConstraintRot es evidencia experimental directa —0 % de violación si la norma está presente, 38 % si se perdió—; y la observación de que un agente que no ve el porqué refactoriza la razón.
- **Por qué Quipu lo necesita:** H-06 y H-07. Existe un modelo normativo riguroso que nadie consulta, y dos tablas puente muertas.
- **Solución mínima:** el payload de reclamo incluye los invariantes que intersectan el alcance y las decisiones vigentes del dominio. Es un `JOIN` sobre M1.
- **Complejidad:** casi nula una vez existe M1.
- **Alternativa más simple:** documentarlo en el `AGENTS.md` del proyecto. Se descarta con evidencia: eso es exactamente lo que la compactación borra.
- **¿Fase posterior?** No, pero depende de M1.

### M4 · Puente adopción ↔ demanda

- **Qué problema resuelve:** que la cadena de gobierno no alcanza nada de lo adoptado.
- **Evidencia:** el cliente ya lo midió de forma independiente y escribió como norma vigente que las reglas de Quipu son un espejo de sólo lectura (auditoría §5).
- **Por qué Quipu lo necesita:** porque la adopción brownfield es la ventaja competitiva declarada, y hoy todo lo adoptado nace fuera del gobierno.
- **Solución mínima:** que un cambio alcance bloques directamente, no sólo vía `feature.cambio_id`.
- **Complejidad:** baja; una rama más en una función existente y una tool que la use.
- **Alternativa más simple:** crear features sintéticas para lo adoptado. Se descarta: falsifica el modelo para sortear una restricción de una consulta.
- **¿Fase posterior?** Es la única de las seis que podría esperar sin romper nada más — pero sin ella no hay ningún proyecto real que gobernar.

### M5 · Perímetro y autorización

- **Qué problema resuelve:** que cualquiera puede convertirse en el humano que firma.
- **Evidencia:** verificado en vivo (H-14); y el principio de que la autoridad debe ser inverificable desde dentro del contexto (§4.2) sólo tiene sentido si es verificable desde fuera.
- **Por qué Quipu lo necesita:** porque su producto **es** la cadena de autoridad. Un fallo de perímetro aquí no es una vulnerabilidad: es la refutación de la tesis.
- **Solución mínima:** depende de DEC-1, que sigue abierta. Las dos opciones —autenticación real, o binding local declarado como restricción dura— cierran el agujero con costes muy distintos.
- **Complejidad:** baja o media según la opción.
- **¿Fase posterior?** No. Es la única de las seis cuya ausencia invalida a las otras cinco.

### M6 · Integridad del rastro (firma y esquema)

- **Qué problema resuelve:** que el rastro de autoridad puede apuntar a la nada (4 firmas huérfanas), y que el esquema desplegado puede no ser el escrito (H-21, ya ocurrió en silencio).
- **Evidencia:** el consenso de audit trail exige hash encadenado e inmutabilidad; una firma sin integridad referencial no cumple ninguna de las dos.
- **Solución mínima:** trigger que impida borrar lo firmado, y comprobación de deriva de esquema en CI.
- **Complejidad:** baja.
- **¿Fase posterior?** No, por una razón de coste: cada día que pasa se acumula más rastro cuya integridad habrá que reconstruir.

---

## 10.4 Qué NO debería incorporarse, y por qué

`RECOMENDACIÓN` — Estos hallazgos son reales en el campo y **no aplican a Quipu**. Se listan porque
el encargo pedía explícitamente que un gap no implique construir.

| No incorporar | Razón basada en la evidencia |
|---|---|
| **Memoria de agente / base vectorial** | Los benchmarks no son comparables (§4.3). El estado persistente ya vive en un store transaccional, que es estrictamente mejor para lo que Quipu necesita: invalidable |
| **Grafo semántico completo del código** | Las cifras de mejora son `E6`. Las tres consultas que el gobierno necesita se resuelven con alcance↔artefacto y un `JOIN` (§4.4) |
| **Framework de orquestación (LangGraph/CrewAI/Temporal)** | Duplicaría el estado. Quipu ya es un motor de estados transaccional; los motores durables son duales, no superiores (§7.1) |
| **Mensajería agente-a-agente / A2A en el núcleo** | El handoff verificable es una fila, no una conversación. Y A2A no puede expresar gobierno (§6.4) |
| **Jueces LLM vinculantes** | κ≈0,51, alto acuerdo entre jueces por sesgo compartido, deriva respecto a humanos (§5.4) |
| **Verificación formal en el núcleo** | Es una capacidad del proyecto cliente, no del plano de control. Quipu debe poder **exigirla como tipo de evidencia**, no ejecutarla |
| **Dispatcher antes de la verificación** | La evidencia es explícita: paralelizar sin gate convierte un problema de calidad en uno de volumen (§3.1, §6.2) |
| **Más herramientas MCP** | 46 ya exceden el presupuesto de atención; 13 agentes independientes convergen en 4 capacidades (§3.2) |
| **Capa de diseño / catálogo de entrevista / plantillas** | Confirmado por evidencia externa: los agentes leen el código; lo que les falta son restricciones, no réplicas (§4.4) |
| **Esquema propio alineado con OTel GenAI** | Sigue pre-estable, los nombres pueden cambiar. Emitir sí, depender no (§7.2) |
| **Automatizar cualquier acto de autoridad** | Es el único mecanismo de la lista de abstracciones duraderas que nadie puede replicar sin una base transaccional, y el que la evidencia de suplantación (§4.2) hace indispensable |
