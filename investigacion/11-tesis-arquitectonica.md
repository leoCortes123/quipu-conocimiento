# 11 — Tesis arquitectónica

> Este documento responde las trece preguntas del encargo y enuncia la tesis que la evidencia
> sostiene. **No es un plan**: no fija orden, esfuerzo ni asignación, y varias de las decisiones que
> un plan necesitaría (DEC-1 a DEC-8 de la auditoría) siguen abiertas y son del propietario.

---

## 11.1 Las trece preguntas

### 1 · ¿Cuál es el estado actual de la ingeniería de software con agentes?

El objeto de estudio se desplazó de *generar código* a *ejecución delegada bajo supervisión humana*.
La capacidad crece deprisa y de forma medible (horizonte de tarea duplicándose cada 89-131 días), la
medición es frágil (SWE-bench Verified abandonado por contaminación tras tres años), y el trabajo
largo se rompe por proceso, no por inteligencia (72,5 % de los fallos). El volumen producido sube y
la mantenibilidad baja (refactor −70 %, duplicación +81 % en 623M de cambios). La capa menos madura
de la pila, según el survey académico más reciente, es la de gobierno y seguridad.

### 2 · ¿Qué problemas ya sabemos resolver?

Editar código en un repositorio real con un bucle read/search/edit/execute. Aislar la ejecución
(microVM, gVisor, contenedor). Paralelizar sin colisiones de fichero (worktree por agente). Recuperar
trabajo tras un crash (ejecución durable con journal). Expresar y aplicar políticas de autorización
fuera del modelo (Cedar, OPA). Impedir prompt injection por construcción rastreando procedencia de
datos (CaMeL). Detectar suites ceremoniales (mutation testing, en producción con 73 % de aceptación).
Preservar una restricción a través de la compactación (constraint pinning, 0 % de violación por
~47 tokens).

### 3 · ¿Qué problemas siguen abiertos?

Demostrar corrección sin humano y sin otro LLM —la brecha generación-verificación escala con el
cómputo y los agentes generan la respuesta correcta sin saber elegirla—. Mantener coherencia
arquitectónica a lo largo de miles de cambios. Detectar que el conocimiento almacenado quedó
obsoleto (55,2 % en el mejor modelo). Evaluar un agente por algo que no sea un benchmark
contaminable. Y el que fija el techo de todo: la atención humana como recurso escaso.

### 4 · ¿Qué problemas probablemente aparecerán?

Auditoría del trabajo agéntico como requisito contractual. Deuda arquitectónica acumulada
manifestándose como incidentes. Colapso por contaminación de los benchmarks sucesores. Ataques
dirigidos al canal de gobierno —el ataque de eviction ya alcanza el 100 % en laboratorio—.
Divergencia entre agentes de proveedores distintos sobre el mismo repositorio. Y el sello de goma
generalizado, si el volumen escala sin que escale el filtro mecánico.

### 5 · ¿Qué arquitecturas están demostrando ser robustas?

Las que **sacan la verdad del contexto del agente**: estado del trabajo en un store transaccional,
política aplicada en la capa de llamada a herramienta, procedencia de datos rastreada por un
intérprete externo, restricciones exentas de compactación, evidencia con hash encadenado. Y las que
**escalonan el rigor** en lugar de aplicar una técnica única: el portafolio de AWS es el mejor
ejemplo industrial.

Las que se han demostrado frágiles: enjambres de escritores paralelos, jueces LLM sin
meta-evaluación, gobierno declarado en el prompt, y specs como documentos que nadie invalida.

### 6 · ¿Qué principios parecen independientes del modelo utilizado?

Nueve abstracciones sobreviven al filtro «su justificación no menciona ninguna propiedad del
modelo»: necesidad original inmutable; decisión con razón y cadena de reemplazo; invariante con
ancla verificable; alcance como dato; evidencia con procedencia; firma contra huella de contenido;
puerta sobre condiciones comprobables; estado del trabajo en store transaccional; segregación de
funciones. Están desarrolladas en §9.3, con la lista inversa de lo que **no** debe estabilizarse.

### 7 · ¿Qué partes de Quipu ya están alineadas?

Más de las que el propio equipo parece creer:

- **Las reglas en Postgres.** Un `UPDATE` directo recibe el mismo rechazo que la API. Es el
  cumplimiento literal del principio «la verdad fuera del contexto del agente».
- **La spec como filas ejecutables** y el hábito de que cada escenario cite su test. La crítica
  central al spec-driven —los documentos derivan— no aplica a datos con huella.
- **Huella + propagación de sospecha.** Es un motor de frescura de conocimiento construido y
  medido, contra un 55,2 % de detección del mejor modelo frontera. Vale más de lo que el equipo cree.
- **Firmas inmutables contra huella del contenido, con segregación de funciones.** Es la única forma
  de autoridad que la evidencia de suplantación en contexto no puede falsificar.
- **La invariante «no existe tool de aprobación», con test que la vigila.**
- **El producto no llama a ningún LLM.** Neutralidad respecto al agente, que la evidencia de
  divergencia de scaffolds convierte en decisión estratégica, no estética.
- **La vía rápida con deuda y plazo.** La tercera respuesta legítima a la saturación de verificación,
  con expediente. No aparece en ninguna otra herramienta analizada.
- **El motor de estados transaccional.** Confirmado como dual de la ejecución durable, no inferior.

### 8 · ¿Qué partes están incompletas?

Las tres del diagnóstico de la auditoría, y la evidencia externa las agrava en vez de matizarlas:
la **verdad de la evidencia** (una puerta que cuenta formas es superficie de reward hacking), el
**alcance de la norma** (ConstraintRot demuestra experimentalmente que una norma ausente se viola el
38 % de las veces), y la **identidad del humano** (la autoridad afirmada en tokens es falsificable, y
el perímetro de Quipu permite afirmarla con un `curl`).

A eso se añade lo que la evidencia externa señala y la auditoría no priorizó: **no existe el alcance
como dato**, y sin él cinco capacidades distintas son imposibles a la vez.

### 9 · ¿Qué partes podrían estar conceptualmente equivocadas?

Tres, en orden decreciente de confianza:

1. **La capa de diseño.** Modelar la UI como datos asume que el agente necesita una réplica del
   sistema. La evidencia dice que necesita las **restricciones** del sistema. 0 filas tras un año no
   es falta de adopción: es la refutación.
2. **El catálogo de entrevista.** Asume que el diagnóstico brownfield se obtiene preguntando al
   humano. Se obtiene leyendo el repositorio; lo que hay que preguntar es sólo lo que el repositorio
   no contesta.
3. **Dos modelos normativos.** No es una decisión pendiente: es el fallo A5 de la taxonomía
   garantizado por construcción, y ya cobró su precio (20 de 63 invariantes perdidos al importar).

`INTERPRETACIÓN` — Y una cuarta, más incómoda porque afecta a la identidad del proyecto: **46
herramientas MCP**. Trece agentes independientes convergen en cuatro capacidades. Un catálogo grande
no es cobertura; es presupuesto de atención gastado en enseñar el sistema en vez de usarlo.

### 10 · ¿Qué capacidades nuevas debería incorporar?

Las seis del §10.3, sin orden: alcance como dato; evidencia observada; la norma viajando con el
trabajo; el puente adopción↔demanda; perímetro y autorización; integridad del rastro. Y por debajo,
como importantes y no críticas: leases con TTL, versión de modelo en la evidencia, fitness functions,
métricas de gobierno, unificación epistémica e invariantes operacionales.

### 11 · ¿Qué NO debería incorporar?

La tabla completa está en §10.4. En una frase: nada que duplique estado que ya es transaccional,
nada cuya evidencia sea de proveedor, y nada que automatice un acto de autoridad.

### 12 · ¿Cuál debería ser la arquitectura conceptual a largo plazo?

**Quipu como el plano de control que ninguna otra capa puede ocupar.** No porque sea mejor
orquestando —no lo es, ni debería intentarlo— sino porque las tres capas que existen no pueden
hacer esto:

- El **modelo** no puede: no distingue una actualización genuina del operador de una falsificación
  en contexto.
- El **andamiaje del agente** no puede: sus garantías son por sesión, y la compactación borra la
  norma dentro de la sesión.
- El **protocolo** (MCP/A2A) no puede: está documentado que no puede expresar autorización,
  rendición de cuentas, políticas con plazo ni audit trails.

Lo que queda es una capa que sostiene cinco responsabilidades —las mismas cinco que la auditoría ya
había derivado del código, y que esta investigación confirma desde fuera:

| Responsabilidad | Fundamento en la evidencia externa |
|---|---|
| **Verdad operativa** con estado epistémico y frescura | STALE 55,2 %: la invalidación no es delegable al agente |
| **Contexto acotado** derivado del alcance | Context Rot + ConstraintRot: lo que se acumula se contamina y lo que se resume se pierde |
| **Procedencia verificable** de la evidencia | Reward hacking + SLSA/in-toto + la agenda vacía de PDD |
| **Autoridad** no falsificable desde el contexto | ConstraintRot §límites: la autoridad en tokens es suplantable |
| **Autonomía con expediente** | La evaluación útil es longitudinal y propia, no transversal y pública |

`INFERENCIA` — La formulación corta de la auditoría sigue siendo la mejor y ahora tiene respaldo
externo: *Git responde qué cambió; Quipu responde qué estaba permitido cambiar, con qué autoridad y
con qué prueba — y sólo cree en pruebas que vio producir.*

### 13 · ¿Cuál es el conjunto mínimo de cambios?

**Seis**, y su mínimo carácter se argumenta en §10.3. Se enumeran como conjunto, no como secuencia:

1. **Alcance como dato** — la primitiva que desbloquea cinco capacidades a la vez.
2. **Evidencia observada** — comando, código de salida, huella, instante, y un ejecutor que los
   produzca.
3. **La norma en el payload** — un `JOIN` sobre el alcance, una vez existe (1).
4. **Puente adopción↔demanda** — que un cambio alcance bloques directamente.
5. **Perímetro y autorización** — la premisa de la que dependen las otras cinco.
6. **Integridad del rastro** — firma con integridad referencial y detección de deriva de esquema.

`INFERENCIA` — Cuatro observaciones sobre el conjunto que importan más que la lista:

- **Ninguno es un rediseño.** Los seis completan mecanismos existentes en los puntos donde se
  interrumpen. La arquitectura elegida no cambia.
- **Cinco son baratos y uno no.** El ejecutor de evidencia es el único componente genuinamente
  nuevo, y arrastra el requisito de aislamiento. Es también el único que nadie más tiene.
- **Uno bloquea a los demás en sentido lógico**, no cronológico: mientras el perímetro permita que
  cualquiera se emita credenciales de administrador, las garantías construidas encima son
  decorativas. Eso no dice cuándo hacerlo; dice qué significa no haberlo hecho.
- **Dos de los seis ya existen fuera del producto.** La verificación determinista vive en 348 líneas
  de bash en `sistema-a/bin/verificar.sh`, y el alcance declarado vive como convención en los
  ficheros de tarea. La distancia entre el prototipo y el producto es menor de lo que parece.

---

## 11.2 La tesis, en cinco proposiciones

`RECOMENDACIÓN` — Formuladas para poder ser discutidas y, si procede, refutadas.

**P1 · El cuello de botella es la verificación, y es estructural.**
La capacidad de generación escala con el cómputo; la de verificación no, y la brecha entre ambas
crece con los FLOPs de preentrenamiento. Un modelo mejor no cierra la brecha: la ensancha. Por tanto
una plataforma que se posicione en la verificación no está resolviendo un problema de esta
generación de modelos; está resolviendo el problema que cada generación agranda.

**P2 · Lo que se afirma dentro del contexto no puede gobernar.**
Está medido: la norma que sobrevive al resumen se obedece siempre y la que se pierde se viola el
38 % de las veces; las políticas organizativas son 8,3× más frágiles que las normas entrenadas; y un
adversario que sólo controla datos ingeridos puede provocar la pérdida. Toda garantía que viva en el
prompt es refutable. La única defensa completa es que la norma se imponga desde fuera del stream de
tokens.

**P3 · Una prueba que produce el interesado no es una prueba.**
El reward hacking está documentado con casos concretos en modelos frontera, y el camino más corto al
estado `done` es satisfacer al comprobador. Por tanto quien produce el trabajo no puede producir la
prueba del trabajo, y la forma mínima de un tercero no es otro agente: es un ejecutor.

**P4 · El alcance declarado es la primitiva de mayor apalancamiento del dominio.**
Es lo mismo que hace posible verificar (¿tocó lo que dijo?), acotar el contexto (¿qué le obliga
aquí?), detectar contradicción (¿qué norma cruza esto?), repartir trabajo (¿colisiona con otro?) y
analizar impacto (¿qué depende de esto?). Cinco capacidades, una tabla.

**P5 · La verificación mecánica existe para proteger la atención humana, no para sustituirla.**
La precisión de expertos cae de 82 % a 45,5 % cuando la máquina se equivoca; la aprobación número
200 del día no es la primera. El objetivo de la puerta no es eliminar al humano: es garantizar que
lo que llega a su cola merece su juicio. Una plataforma que mida cuántas paradas humanas cambiaron
el resultado sabrá si está gobernando o registrando; ninguna lo mide hoy.

---

## 11.3 Lo que esta investigación no puede decidir

`RECOMENDACIÓN` — Y que sigue siendo del propietario, sin que estos hallazgos lo cambien:

- **DEC-1 · el modelo de autenticación.** La evidencia dice que el perímetro es la premisa; no dice
  si la respuesta correcta es autenticación real o binding local con restricción declarada.
- **DEC-3 · el alcance del ejecutor de evidencia.** La evidencia dice que hace falta un tercero que
  ejecute; no dice si es un binario propio, un hook por proveedor o un servicio. `INTERPRETACIÓN` —
  la evidencia de divergencia de andamiajes (§3.2) sí inclina hacia lo agnóstico al proveedor,
  porque los hooks son por sesión y por CLI, y la propiedad que se busca es entre sesiones y entre
  proveedores.
- **DEC-2, DEC-4, DEC-5** y las demás son de producto, no de investigación.

Y una advertencia final, que es el resultado más incómodo de todo el trabajo:

`INFERENCIA` — La tesis de Quipu tiene ahora **formalización académica independiente** (PDD,
may-2026), **un hueco reconocido en la literatura de protocolos** (jul-2026), y **evidencia
experimental de que el problema que ataca es real** (ConstraintRot, jun-2026). Nada de eso existía
cuando se escribió el plan original. Pero las tres fuentes coinciden también en el punto donde
Quipu está incompleto: **la garantía depende de que el validador observe de verdad**. PDD lo asume
sin construirlo y Quipu lo promete sin hacerlo todavía. El primero que lo construya de verdad tendrá
algo que ni la academia ni el mercado tienen — y esa ventana no estará abierta indefinidamente.
