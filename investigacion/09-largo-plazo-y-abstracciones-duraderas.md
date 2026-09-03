# 09 — Proyectos largos, deriva arquitectónica y abstracciones duraderas

Cubre las áreas 15, 21 y 22 del encargo.

## 9.1 Qué cambia cuando el horizonte crece

`HECHO` (`E1`, 2026) — La degradación por horizonte está medida y no es gradual. HORIZON documenta
una «transición abrupta de robustez parcial a fallo casi sistemático» más allá de un umbral pequeño
de profundidad composicional. La cifra que circula del mismo cuerpo de trabajo: los modelos frontera
resuelven casi el 100 % de las tareas que a un humano experto le llevan menos de ~4 minutos y menos
del 10 % de las que le llevan más de ~4 horas.

`HECHO` (`E1`, ene-2026) — METR mide que el horizonte crece rápido (duplicación cada 89-131 días
según ventana). `HECHO` — Y declara que sólo 5 de sus 31 tareas largas tienen línea base humana
medida, y que la tendencia es sensible a la composición del conjunto de tareas.

> `INFERENCIA` — Las dos cosas son ciertas a la vez: el horizonte crece deprisa **y** el punto de
> ruptura sigue siendo abrupto. Lo que se mueve es dónde está el acantilado, no que exista. Una
> arquitectura que dependa de que el acantilado desaparezca está apostando contra la única
> propiedad estable que se ha medido.

`INFERENCIA` — Lo que cambia en cada escala, ordenado, a partir de la evidencia de los documentos
anteriores:

| Escala | Qué se vuelve el problema dominante |
|---|---|
| **Horas** | contexto: compactación, ruido, pérdida de la instrucción original (§4.1, §4.2) |
| **Días** | continuidad: reanudación tras crash, estado que sobrevive a la sesión, leases (§7.1) |
| **Semanas** | coherencia: decisiones tomadas en la sesión 3 que la sesión 40 no conoce (§2.5, D2) |
| **Meses** | obsolescencia: lo que se registró como cierto dejó de serlo y nadie lo notó (§4.3) |
| **Cientos de miles de líneas** | localización: qué parte del sistema importa para esta tarea (§4.4) |
| **Múltiples agentes** | propiedad y conflicto semántico (§3.4, D1-D2) |
| **Múltiples versiones de modelo** | regresión silenciosa y comparabilidad del historial (§7.3, E5) |

`INTERPRETACIÓN` — Sólo el primer problema mejora con modelos mejores. Los otros seis son de
infraestructura y **empeoran** con modelos mejores, porque un modelo mejor produce más volumen sobre
el mismo sustrato.

## 9.2 Deriva arquitectónica: la pregunta del encargo

> *¿Cómo evitar que un sistema desarrollado por agentes se degrade arquitectónicamente aunque cada
> cambio individual parezca correcto?*

`HECHO` (`E2`, 2026) — El fenómeno está medido a escala: 623 millones de cambios entre 2023 y 2026;
refactorización −70 %, duplicación de bloques +81 %, mantenimiento de legado −74 %, construcciones
que enmascaran errores +47 % (§1.3).

`HECHO` (`E4`, abr-2026) — La hipótesis causal que propone el survey de Bhati: los agentes producen
más código porque producir es barato, y prefieren **arreglos locales al rediseño global** porque el
rediseño es caro en tokens. Lo lista como problema abierto: faltan estudios longitudinales de salud
de repositorios.

`HECHO` (`E3`, 2026) — El mecanismo de defensa sobre el que hay convergencia es antiguo y no
depende de la IA: las **fitness functions** —comprobaciones automáticas y objetivas de que una
característica arquitectónica se mantiene— ejecutadas como tests. ArchUnit es el ejemplo canónico:
construye un modelo en memoria de clases, paquetes y dependencias, y las reglas corren como tests
normales. La formulación que circula en 2026 captura por qué importa ahora: es *el sensor que
detecta al agente derivando la arquitectura a medianoche*, y funciona igual si el código lo escribió
una persona, una pareja o un prompt.

`INFERENCIA` — El argumento de por qué esto pasa de buena práctica a requisito: **el volumen de
código generado supera la capacidad de revisión manual de encontrar problemas estructurales**. No es
que las fitness functions se hayan vuelto mejores; es que la alternativa —que alguien lo note
revisando— dejó de existir.

`INFERENCIA` — La respuesta completa a la pregunta del encargo tiene tres partes, y las tres son
mecánicas:

1. **Que la restricción arquitectónica sea ejecutable.** Un ADR que dice «la capa de dominio no
   depende de infraestructura» no evita nada; una regla que falla en CI, sí. Todo lo que sea
   documento se erosiona a la velocidad a la que los agentes escriben.
2. **Que la norma llegue al agente junto con el trabajo.** Prevención antes que detección: cuesta
   ~47 tokens entregar una restricción y cuesta un rediseño detectarla violada tres meses después
   (§4.2).
3. **Que el rediseño tenga un camino barato.** Si el único camino es un cambio grande que ninguna
   puerta deja pasar, el sistema está premiando estructuralmente el parche local. `INFERENCIA` —
   Éste es el punto que la literatura no resuelve y que ninguna herramienta del mercado aborda: hace
   falta una unidad de trabajo de tipo «rediseño» que sea barata de proponer y cara de aprobar, en
   lugar de cara de proponer y por tanto inexistente.

## 9.3 Qué debería permanecer estable aunque cambie todo lo demás

El encargo pide identificar las abstracciones duraderas. El criterio que se aplicó: **una abstracción
es duradera si su justificación no menciona ninguna propiedad del modelo**. Con ese filtro, de toda
la evidencia recogida sobreviven nueve.

| # | Abstracción | Por qué es independiente del modelo |
|---|---|---|
| 1 | **La necesidad original, literal e inmutable** | Existe antes de cualquier agente. Es el único referente contra el que se puede detectar que se malinterpretó algo, y no lo produce el sistema |
| 2 | **La decisión con su razón y su cadena de reemplazo** | Un sistema tiene una historia de decisiones tanto si las toman personas como agentes. La cadena supersede es un hecho del dominio, no del andamiaje |
| 3 | **El invariante con ancla verificable** | Es una propiedad del software, no del productor. PDD llega a la misma primitiva por una vía completamente distinta (§2.3) |
| 4 | **El alcance como dato** | «Qué partes del sistema toca este trabajo» es una pregunta de ingeniería de software de los años setenta. Hoy resuelve a la vez verificación, contexto, contradicción y conflicto |
| 5 | **La evidencia con procedencia** | «Este comando, esta salida, este instante» es verdad o mentira con independencia de quién ejecutó. Es el mismo objeto que una atestación SLSA |
| 6 | **La firma contra la huella del contenido** | La criptografía no cambia con los modelos. Es la única forma de autoridad que no es falsificable escribiendo texto (§4.2) |
| 7 | **La puerta que decide sobre condiciones comprobables** | Un predicado sobre el estado. Sobrevive a cualquier ejecutor |
| 8 | **El estado del trabajo en un store transaccional** | El agente es reemplazable; el trabajo persiste. Es el mismo principio que la ejecución durable (§7.1) |
| 9 | **La segregación de funciones** | Quien hace no aprueba. Es un principio de control interno anterior a la informática |

`INFERENCIA` — Y su reverso, lo que **no** debe estar en el núcleo porque cambia cada seis a doce
meses, con la prueba en la propia evidencia recogida:

| No estabilizar | Prueba de que cambia |
|---|---|
| La topología de agentes | Cognition invirtió su posición pública en doce meses (§3.1) |
| La estrategia de contexto | Siete estrategias distintas en 13 agentes; ningún estándar (§3.2) |
| El catálogo de herramientas | De 0 a 37 tools para las mismas cuatro capacidades (§3.2) |
| El protocolo de transporte | MCP tuvo dos revisiones de especificación en nueve meses (§6.4) |
| El esquema de telemetría | OTel GenAI sigue pre-estable y sus nombres pueden cambiar (§7.2) |
| El proveedor y la versión del modelo | Regresión silenciosa en cada migración (§7.3) |
| La técnica de memoria | El mismo sistema puntúa 49 % y 94 % en el mismo benchmark (§4.3) |

`RECOMENDACIÓN` — La regla de diseño que se deduce: **todo lo de la primera tabla debe ser esquema;
todo lo de la segunda debe ser adaptador.** Un sistema que meta cualquier fila de la segunda tabla
en su modelo de datos se compromete a migrarlo.

## 9.4 Los problemas que probablemente aparecerán

`INFERENCIA` — Proyección razonada, marcada como tal. No hay evidencia de estos porque todavía no
han ocurrido a escala; hay señales de que ocurrirán.

1. **Auditoría del trabajo agéntico como requisito contractual.** Las obligaciones del EU AI Act
   para alto riesgo entran en agosto de 2026 y piden demostrabilidad ante terceros. La señal
   temprana: los protocolos de agentes no pueden expresar audit trails y ya hay literatura
   nombrándolo como hueco (§6.4).
2. **Deuda arquitectónica acumulada volviéndose visible.** GitClear mide la tendencia desde 2022;
   las consecuencias de una caída del 70 % en refactorización tardan años en manifestarse como
   incidentes. `INFERENCIA` — Esto crea demanda futura para exactamente la clase de herramienta que
   puede demostrar qué se cambió, por qué y con qué prueba.
3. **Colapso de la evaluación por contaminación.** SWE-bench Verified murió en tres años. El mismo
   ciclo consumirá a sus sucesores. `INFERENCIA` — La evaluación útil migrará hacia el historial
   privado de cada organización, que es precisamente lo que una plataforma de gobierno acumula sin
   proponérselo.
4. **Ataques a través del canal de gobierno.** El ataque de eviction ya existe en el laboratorio y
   llega al 100 % en algún modelo (§4.2). `INFERENCIA` — Cuando las restricciones organizativas
   valgan dinero, se atacarán en producción.
5. **Divergencia entre agentes de distintos proveedores sobre el mismo repositorio.** Hoy casi todo
   el mundo usa un único proveedor por proyecto. Cuando se mezclen —y la neutralidad de Spec Kit y
   de Agent Skills indica que se mezclarán— los supuestos implícitos de cada andamiaje colisionarán.
   `INFERENCIA` — Sólo una fuente de verdad externa y común puede arbitrar eso.
6. **La atención humana como el recurso que fija el techo.** Es el quinto problema abierto de Bhati,
   ya nombrado, y no tiene solución tecnológica a la vista: escalar la producción sin escalar la
   verificación mecánica lleva al sello de goma, y el sello de goma anula el valor de todo el resto.
