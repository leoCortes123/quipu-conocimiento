# Informe estratégico de Quipu frente a la evolución del desarrollo agentic

## 1. Resumen ejecutivo

### Conclusión principal

Quipu está construyendo una capa que probablemente será **cada vez más necesaria a medida que los agentes de IA pasen de asistentes de programación a ejecutores autónomos de trabajo de ingeniería**.

La hipótesis central de Quipu es correcta:

> El problema futuro del desarrollo con IA no será principalmente conseguir que un agente escriba código, sino conseguir que múltiples agentes puedan trabajar durante largos periodos sobre proyectos complejos sin perder contexto, violar decisiones, duplicar trabajo, introducir contradicciones o producir cambios imposibles de verificar.

La industria ya se está desplazando en esa dirección.

Anthropic describe en 2026 una transición desde agentes que resuelven tareas aisladas hacia agentes coordinados, trabajos de larga duración, supervisión humana selectiva y desarrollo fuera del ámbito de los programadores profesionales.

OpenAI está desarrollando Codex precisamente hacia un modelo de trabajo persistente, multiagente y de larga duración, donde el problema deja de ser únicamente "hacer que el agente programe" y pasa a ser "mantener continuidad, dividir trabajo, verificarlo y decidir cuándo intervenir".

Esto coincide de forma notable con varias decisiones arquitectónicas de Quipu.

Sin embargo:

**Quipu todavía no es la plataforma completa que esta tendencia va a necesitar.**

La versión actual tiene un core fuerte de:

* gobierno;
* restricciones;
* workflow;
* trazabilidad;
* evidencia;
* identidad;
* MCP;
* adopción de proyectos existentes;
* decisiones e invariantes;
* separación entre autoridad humana y ejecución del agente.

Pero todavía le falta convertir ese core en una **memoria operacional completa y continuamente actualizada del software**, y sobre todo en un sistema capaz de coordinar agentes de forma persistente a escala.

Mi conclusión es:

> **No recomiendo abandonar el diseño actual ni rehacer el core. Recomiendo conservarlo como núcleo de gobernanza y reorientar las fases siguientes alrededor de memoria, contexto, coordinación multiagente, verificación automática y evolución continua del proyecto.**

---

# 2. Estado real de Quipu

La versión actual ya constituye algo más que un gestor de tareas.

La estructura fundamental es:

```text
Proyecto
 └── Feature
      └── Bloque
           ├── Entregables
           ├── Criterios
           ├── Dependencias
           ├── Evidencia
           └── Microtareas
```

El trabajo tiene estados mecánicos:

```text
backlog
   ↓
ready
   ↓
in_progress
   ↓
verifying
   ↓
done
```

con `blocked` como estado transversal. Los gates son evaluados por PostgreSQL y el cierre final requiere aprobación humana.

El sistema tiene actualmente 46 tools para agentes, dos transportes MCP, siete roles y 22 permisos.

Esto significa que el core ya tiene una estructura suficientemente rica para que un agente no tenga que trabajar únicamente con texto libre.

---

# 3. Lo más importante que Quipu ya hace bien

## 3.1. Convierte conocimiento normativo en una estructura ejecutable

Esta es una de las decisiones más importantes.

Quipu no trata todo el conocimiento como documentación.

Distingue entre:

```text
texto humano
```

y:

```text
dato estructurado que puede imponer una regla
```

Los entregables, criterios, dependencias y estados existen como entidades que pueden ser consultadas y validadas.

La propia guía establece como principio que el texto libre está dirigido al humano, mientras que los entregables y criterios deben existir como datos estructurados.

Esto es exactamente lo que necesitan los agentes.

Un agente puede leer:

```text
"El endpoint debe mantener compatibilidad."
```

pero no puede hacer mucho con eso.

En cambio:

```text
contract_lock = active
endpoint = /api/users
breaking_change = forbidden
```

puede utilizarse como condición de ejecución.

---

# 4. La decisión arquitectónica más valiosa: el agente no es la autoridad

Quipu establece una separación clara:

```text
Agente
  ↓
propone / ejecuta / evidencia

Humano
  ↓
aprueba / firma / decide
```

El agente puede llevar un bloque hasta `verifying`, pero no puede aprobarlo.

Tampoco puede firmar actos de gobierno, promover decisiones o resolver contradicciones.

Esto coincide directamente con la evolución de la industria.

Anthropic observa que incluso con grandes aumentos de productividad, los desarrolladores siguen necesitando supervisión y juicio humano, especialmente para trabajo de mayor impacto.

OpenAI está adoptando una arquitectura similar: los agentes pueden operar autónomamente, pero los sistemas necesitan límites técnicos, puntos explícitos de aprobación y telemetría para explicar qué hizo el agente.

### Evaluación

Esta parte de Quipu está **adelantada respecto al problema que el mercado está empezando a enfrentar**.

No debería eliminarse.

Debería ampliarse.

---

# 5. Evidencia: probablemente el segundo gran acierto

El modelo:

```text
entregable
   ↓
microtarea
   ↓
evidencia
   ↓
criterio
```

es muy importante.

El agente no puede simplemente decir:

> "terminé".

Tiene que producir evidencia que demuestre los criterios.

Quipu incluso impide marcar un criterio como cumplido si no existe evidencia asociada.

Esto adquiere más importancia a medida que los agentes trabajan durante horas o días.

El problema futuro no será solamente:

> ¿Puede el agente hacerlo?

Será:

> ¿Cómo sé que realmente hizo lo que debía hacer?

La aparición de benchmarks como SWE-RPG confirma que medir solamente si el parche final pasa los tests es insuficiente. El benchmark separa recuperación de requisitos, planificación y generación del código, y encuentra que la recuperación de requisitos implícitos sigue siendo un cuello de botella importante.

Quipu ya tiene una estructura para atacar precisamente esa cadena.

---

# 6. Adopción de proyectos existentes: una capacidad estratégicamente importante

Quipu no solamente sirve para proyectos nuevos.

Tiene dos caminos:

```text
Bootstrap
Proyecto nuevo
```

y:

```text
Adopción
Proyecto existente
```

En adopción, el código existente se registra como línea base y únicamente lo pendiente entra como backlog.

La importación además es idempotente y puede ejecutarse mediante REST, MCP o consola.

Esto es especialmente relevante porque **la mayor parte del software empresarial futuro no será construido desde cero**.

Los agentes tendrán que trabajar sobre:

* sistemas antiguos;
* múltiples lenguajes;
* arquitecturas inconsistentes;
* documentación incompleta;
* deuda técnica;
* código generado anteriormente;
* cambios realizados por otros agentes.

La investigación actual muestra precisamente que los agentes todavía tienen problemas importantes con tareas de larga duración y migraciones completas de repositorios. En SWE Refactor Bench, solamente 5.4% de las ejecuciones evaluadas superaron simultáneamente auditoría de migración, comportamiento y verificación agentic.

Por tanto:

**la adopción de legado no es una feature secundaria de Quipu; puede convertirse en una de sus principales ventajas competitivas.**

---

# 7. El problema que Quipu todavía no resuelve suficientemente: comprender el proyecto

Aquí está uno de los principales huecos.

Actualmente existe:

```text
quipu:index
```

que escanea el repositorio y registra clases PHP y componentes React.

También existe la adopción mediante:

```text
features
blocks
baseline
endpoints
screens
knowledge
```

Pero eso todavía no equivale a una comprensión completa del sistema.

Un proyecto complejo necesita una representación de:

```text
arquitectura
componentes
dependencias
datos
APIs
interfaces
flujos
reglas de negocio
decisiones
convenciones
deuda técnica
riesgos
tests
comportamientos observados
relaciones entre módulos
```

Y además necesita saber:

```text
qué está confirmado
qué está inferido
qué está obsoleto
qué se desconoce
```

Quipu tiene elementos de esto, pero no parece haber llegado todavía a un **Project Knowledge Graph** completo.

Este debería ser uno de los objetivos principales de las siguientes fases.

---

# 8. El problema más importante descubierto por la investigación externa

Los agentes actuales están mejorando rápidamente en generación de código.

SWE-bench Verified ya muestra resultados cercanos al 80% en algunas configuraciones de agentes/modelos.

Pero eso no significa que estén cerca de resolver autónomamente el desarrollo de software completo.

Investigaciones recientes muestran problemas diferentes:

### Recuperación de requisitos

SWE-RPG encuentra que los agentes evaluados resolvieron solamente 31.5% de las tareas y que la recuperación de requisitos implícitos representaba entre 24.5% y 46% de las causas de fallo.

### Trabajo de larga duración

Las migraciones completas de repositorios siguen siendo extremadamente difíciles.

### Estado compartido

SWE-Touch encuentra que cuando un humano modifica el workspace durante el trabajo del agente, el rendimiento cae y los agentes tienen problemas para detectar, reconciliar y validar esos cambios.

### Coordinación

Anthropic prevé que los flujos pasen de un agente a equipos coordinados de agentes, lo que exige mecanismos de descomposición, coordinación, estado compartido y versionado.

### Conclusión

El cuello de botella está desplazándose:

```text
2024
¿Puede escribir código?

        ↓

2025
¿Puede resolver una tarea?

        ↓

2026
¿Puede completar una feature?

        ↓

2027+
¿Puede mantener coherentemente un sistema durante meses?
```

Quipu está mucho mejor posicionado para el último problema que para el primero.

---

# 9. El futuro no será un agente: será una organización de agentes

Este cambio es crítico para el roadmap.

OpenAI ya está posicionando Codex como un entorno para múltiples agentes que trabajan en paralelo y tareas que duran horas, días o semanas.

Anthropic plantea igualmente la evolución hacia equipos coordinados de agentes.

Esto cambia completamente el problema de diseño.

Hoy:

```text
Usuario
   ↓
Claude Code
   ↓
Quipu
```

Futuro:

```text
                     ┌── Agent arquitectura
                     │
                     ├── Agent backend
                     │
Usuario → Quipu → Orchestrator
                     ├── Agent frontend
                     │
                     ├── Agent testing
                     │
                     ├── Agent security
                     │
                     └── Agent reviewer
```

Todos trabajando sobre el mismo proyecto.

Aquí Quipu puede convertirse en algo mucho más importante que un MCP server.

Puede ser:

> **el estado compartido y la capa de coordinación de una organización de agentes.**

---

# 10. El MCP de Quipu está bien orientado, pero MCP no será la ventaja competitiva

MCP está evolucionando rápidamente.

La especificación 2026 introduce tareas, extensiones, mejoras de autorización y mecanismos orientados a interacciones más complejas.

GitHub ya ofrece MCP y agent skills como mecanismos para conectar contexto, herramientas y estándares organizacionales con agentes.

Por tanto:

```text
"Tenemos MCP"
```

no será una ventaja suficiente.

MCP se convertirá en infraestructura común.

La ventaja de Quipu debe ser:

```text
MCP
+
modelo formal del proyecto
+
memoria
+
gobernanza
+
estado
+
dependencias
+
evidencia
+
orquestación
```

MCP debería ser **el transporte**, no el producto.

---

# 11. Un punto que debe cambiar de prioridad: memoria

Quipu tiene conocimiento, decisiones e invariantes.

Pero para el futuro necesita una memoria mucho más amplia.

Yo separaría:

## Memoria normativa

```text
Decisiones
Invariantes
Reglas
Contratos
Permisos
```

## Memoria estructural

```text
Arquitectura
Componentes
APIs
DB
Dependencias
Pantallas
Servicios
```

## Memoria operacional

```text
Qué se hizo
Quién lo hizo
Qué agente lo hizo
Qué cambió
Qué pruebas pasaron
Qué falló
```

## Memoria histórica

```text
Qué existía
Qué se reemplazó
Por qué se reemplazó
Qué decisiones fueron superadas
```

## Memoria epistémica

```text
Qué sabemos
Qué inferimos
Qué creemos
Qué no sabemos
```

Esta última es especialmente importante.

El agente debería poder recibir:

```text
FACT
CONFIRMED
INFERRED
PROPOSED
UNKNOWN
SUPERSEDED
```

y no tratar todos los datos como igualmente fiables.

---

# 12. El verdadero producto futuro: contexto fiable

Esta puede ser la tesis comercial más importante de Quipu.

Los modelos están aumentando sus context windows.

Pero:

> **más contexto no significa mejor contexto.**

Un agente puede recibir 500.000 tokens y seguir confundido si contiene:

* decisiones antiguas;
* código muerto;
* documentación contradictoria;
* implementaciones duplicadas;
* requisitos obsoletos;
* conocimiento inferido presentado como hecho.

Quipu debería convertirse en un sistema que responda:

> "Para esta tarea, este es el contexto correcto."

No:

> "Aquí tienes todo el contexto."

La diferencia es fundamental.

---

# 13. Qué debería hacer Quipu antes de que un agente empiece una tarea

Idealmente:

```text
Agent
 ↓
get_project_context(task)
```

y Quipu debería devolver algo equivalente a:

```text
PROJECT
 ├── objective
 ├── architecture
 ├── affected components
 ├── relevant decisions
 ├── active invariants
 ├── applicable contracts
 ├── known constraints
 ├── previous related changes
 ├── existing implementation
 ├── tests
 ├── technical debt
 ├── unresolved contradictions
 ├── unknowns
 └── required evidence
```

El agente ya no tendría que hacer una exploración indiscriminada del repositorio.

Eso reduciría:

* tokens;
* tiempo;
* errores;
* decisiones inconsistentes;
* dependencia del contexto de una sesión anterior.

---

# 14. El siguiente problema: aprendizaje continuo

Quipu actualmente registra conocimiento.

El siguiente nivel debería ser:

```text
Agente trabaja
      ↓
descubre algo
      ↓
produce evidencia
      ↓
Quipu propone actualización
      ↓
humano/agente valida
      ↓
knowledge actualizado
```

Ejemplo:

El agente descubre que:

```text
PaymentService
```

siempre requiere:

```text
TransactionContext
```

Quipu debería poder detectar:

> Esto parece una convención no documentada.

Y proponer:

```text
Knowledge K-183
type = convention
status = proposed
evidence = [...]
```

Después de validarse:

```text
status = confirmed
```

Así el sistema **aprende del desarrollo**.

Esta característica sería mucho más diferenciadora que añadir más vistas de gestión.

---

# 15. Coordinación multiagente

La versión actual trabaja esencialmente con:

```text
un agente
→ un bloque
```

El futuro requiere:

```text
proyecto
→ múltiples agentes
→ múltiples bloques
→ múltiples contextos
```

Quipu ya tiene dependencias y `claim_block`, por lo que posee una base adecuada.

Pero debería evolucionar hacia:

```text
Agent A
claim B-12

Agent B
claim B-13

Agent C
review B-12

Agent D
run integration tests
```

con:

```text
resource locking
ownership
conflict detection
workspace state
dependency scheduling
agent health
agent capabilities
agent priority
agent budget
```

La coordinación será cada vez más importante porque los agentes pueden trabajar en paralelo. Anthropic identifica explícitamente la coordinación multiagente como una de las principales transformaciones de 2026.

---

# 16. Quipu debería incorporar "capacidad del agente"

No todos los agentes deberían poder hacer todo.

Por ejemplo:

```text
Agent Architecture
  capabilities:
    - inspect
    - design
    - propose_decision

Agent Backend
  capabilities:
    - php
    - sql
    - api

Agent QA
  capabilities:
    - test
    - evidence
    - review

Agent Security
  capabilities:
    - security_review
    - dependency_audit
```

Entonces Quipu puede asignar trabajo según capacidad.

Esto transforma:

```text
task manager
```

en:

```text
agent workforce manager
```

---

# 17. Otro cambio necesario: revisión automática

Hoy:

```text
Agente
 ↓
verifying
 ↓
Humano
 ↓
done
```

En el futuro:

```text
Agente constructor
       ↓
Agente reviewer
       ↓
Agente tester
       ↓
Agente security
       ↓
Quipu
       ↓
Humano
```

El humano debería recibir:

> "Hay un bloque listo para revisión."

y no:

> "Aquí tienes 400 líneas de código generadas por un agente."

Anthropic prevé precisamente que la revisión agentic se convierta en una forma de escalar el control humano sobre grandes volúmenes de código generado.

---

# 18. El concepto de evidencia debe evolucionar

Actualmente:

```text
evidence = output real
```

Eso está bien.

Pero en el futuro debería haber evidencia de diferentes niveles:

```text
STATIC
test
integration_test
runtime
security_scan
performance
human_review
agent_review
production_observation
```

Y Quipu podría calcular:

```text
confidence = 0.94
```

no necesariamente como una probabilidad matemática del modelo, sino como una **clasificación de fuerza de evidencia**.

Por ejemplo:

```text
criterio
 ├── unit test
 ├── integration test
 ├── static analysis
 └── human approval
```

es mucho más fuerte que:

```text
criterio
 └── agente afirma que funciona
```

---

# 19. El problema de seguridad será cada vez más importante

Aquí Quipu tiene una ventaja conceptual, pero la implementación actual todavía está lejos de un producto comercial.

La guía explícitamente indica que actualmente la autenticación no está endurecida y que la identidad está pensada para atribución, no para defensa.

Eso es aceptable para una herramienta local experimental.

No es aceptable para:

```text
SaaS
empresa
multiusuario
agentes autónomos
repositorios privados
producción
```

La evolución de agentes aumenta el problema.

Anthropic identifica el riesgo de prompt injection y acciones no deseadas como uno de los principales problemas de los sistemas agentic.

Además, incidentes recientes con agentes muestran que la autonomía creciente hace más importante controlar qué puede hacer un agente y qué sistemas puede tocar.

Por tanto Quipu necesitará eventualmente:

```text
agent identity
+
scoped permissions
+
resource permissions
+
secret isolation
+
audit logs
+
approval policies
+
sandboxing
+
network policy
+
action risk classification
```

---

# 20. Un cambio conceptual que recomiendo: "riesgo" como propiedad de la operación

No todas las acciones deberían requerir la misma intervención humana.

Por ejemplo:

```text
LOW
crear test
leer archivo
ejecutar lint

MEDIUM
modificar API
cambiar schema
actualizar dependencia

HIGH
migrar DB
cambiar auth
modificar infraestructura

CRITICAL
deploy
borrar datos
modificar permisos
rotar secretos
```

Quipu podría entonces aplicar:

```text
LOW
autónomo

MEDIUM
autónomo + review

HIGH
human approval

CRITICAL
human approval + segunda evidencia
```

Esto es mucho más escalable que:

```text
agente sí/no
```

y coincide con la dirección de los controles que los sistemas de agentes empresariales están desarrollando.

---

# 21. Qué NO recomiendo hacer

## No convertir Quipu en Jira

Eso desviaría el producto.

## No convertirlo en un IDE

Los agentes ya están ocupando ese espacio.

## No competir con Claude Code/Codex en generación de código

Es una batalla que no aporta ventaja.

## No construir otro vector database como feature principal

La memoria semántica es sólo una pieza.

## No añadir cientos de herramientas MCP

El número de tools no es la ventaja.

## No intentar automatizar la aprobación humana

Eso destruye una de las mejores propiedades actuales del sistema.

---

# 22. Qué debería ser el núcleo definitivo

La arquitectura futura debería verse así:

```text
                    ┌─────────────────────┐
                    │       HUMAN         │
                    │ strategy / approval │
                    └──────────┬──────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────┐
│                    QUIPU                        │
│                                                 │
│  Project Knowledge                              │
│  Decision Graph                                 │
│  Requirement Graph                              │
│  Architecture Graph                             │
│  Work Graph                                     │
│  Evidence Graph                                 │
│  Agent State                                    │
│  Risk / Permission Model                         │
│  Audit / History                                │
│                                                 │
│             Protocol / Policy Engine             │
└──────────────────────┬──────────────────────────┘
                       │
                MCP / APIs / Events
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
     Agent A        Agent B        Agent C
   architecture    backend          QA
        │              │              │
        └──────────────┼──────────────┘
                       ▼
                 Repository
                       │
                       ▼
                Tests / Runtime
                       │
                       ▼
                   Evidence
                       │
                       └────→ Quipu
```

Ese sería un producto mucho más difícil de sustituir.

---

# 23. Evaluación del estado actual

Mi evaluación después de contrastar Quipu con el estado actual del desarrollo agentic es:

| Área                               | Estado actual | Importancia futura |
| ---------------------------------- | ------------: | -----------------: |
| Protocol Engine                    |          9/10 |           Muy alta |
| Restricciones mecánicas            |          9/10 |           Muy alta |
| Human-in-the-loop                  |          9/10 |           Muy alta |
| Trazabilidad                       |          9/10 |           Muy alta |
| Evidencia                          |        8.5/10 |           Muy alta |
| Decisiones/invariantes             |        8.5/10 |           Muy alta |
| MCP                                |          8/10 |              Media |
| Adopción                           |          8/10 |           Muy alta |
| Memoria de proyecto                |          6/10 |            Crítica |
| Comprensión del código             |          6/10 |            Crítica |
| Contexto contextualizado por tarea |          5/10 |            Crítica |
| Coordinación multiagente           |          3/10 |            Crítica |
| Revisión automática                |          4/10 |           Muy alta |
| Aprendizaje continuo               |          3/10 |           Muy alta |
| Seguridad agentic                  |          3/10 |            Crítica |
| Experiencia no técnica             |          3/10 |               Alta |
| SaaS / colaboración remota         |          2/10 |               Alta |

Esto produce una conclusión interesante:

**el core que ya está construido tiene mayor valor que muchas de las funcionalidades que todavía faltan.**

No parece necesario cambiar la tesis.

Hay que completar la capa superior.

---

# 24. Cómo reorganizaría las próximas fases

No puedo afirmar que ésta sea la secuencia del roadmap original porque las dos guías no contienen ese roadmap. Pero, considerando el estado actual y el mercado, propondría esta reorganización.

## Fase A — Consolidar el Core

Objetivo:

> hacer que todo lo que ya existe sea coherente, estable y utilizable.

Prioridades:

* cerrar inconsistencias;
* estabilizar APIs MCP;
* completar operaciones faltantes;
* mejorar adopción;
* resolver limitaciones conocidas;
* completar web para las operaciones críticas;
* reforzar tests;
* eliminar estructuras heredadas incompletas.

No añadir grandes capacidades todavía.

---

# 25. Fase B — Project Intelligence

Ésta debería ser probablemente la siguiente gran fase.

Objetivo:

> Quipu debe comprender un proyecto existente.

Construir:

```text
repository index
architecture graph
component graph
dependency graph
API graph
database graph
screen graph
test graph
knowledge graph
```

Y especialmente:

```text
confirmed
inferred
unknown
```

El resultado debería ser:

> "Quipu conoce suficientemente este proyecto para que un agente pueda empezar a trabajar."

Esto convertiría la adopción actual en una verdadera **adopción inteligente**.

---

# 26. Fase C — Context Engineering

Objetivo:

> Quipu debe saber qué contexto entregar a cada agente para cada tarea.

Crear algo como:

```text
context_for(block)
context_for(feature)
context_for(agent)
context_for(decision)
context_for(component)
```

El agente no debería tener que recorrer arbitrariamente todo el proyecto.

Debería recibir:

```text
relevant knowledge
+
relevant code
+
relevant decisions
+
relevant contracts
+
relevant history
+
relevant constraints
```

Esto es probablemente una de las capacidades con mayor ROI.

---

# 27. Fase D — Multi-Agent Orchestration

Aquí empieza el producto realmente diferencial.

Construir:

```text
agent registry
agent capabilities
agent assignments
agent leases
parallel blocks
agent dependencies
agent communication
agent handoffs
agent conflicts
agent status
```

El objetivo:

```text
1 proyecto
10 agentes
100 bloques
```

sin perder control.

---

# 28. Fase E — Autonomous Verification

El flujo pasa de:

```text
Agent → Human
```

a:

```text
Agent
 ↓
Reviewer Agent
 ↓
Test Agent
 ↓
Security Agent
 ↓
Quipu
 ↓
Human
```

El humano solamente interviene cuando el sistema encuentra:

```text
uncertainty
conflict
high risk
missing evidence
policy violation
architectural ambiguity
```

Esto es exactamente la dirección hacia la que está evolucionando la supervisión agentic: no revisar todo, sino hacer que la atención humana se concentre donde realmente importa.

---

# 29. Fase F — Continuous Project Memory

Aquí Quipu empieza a comportarse como una memoria viva.

Cada cambio puede actualizar:

```text
architecture
knowledge
decisions
dependencies
technical debt
conventions
component catalog
```

Pero únicamente mediante evidencia.

Ejemplo:

```text
Agent discovers convention
        ↓
proposes knowledge
        ↓
evidence
        ↓
review
        ↓
confirmed
```

Esto crea un ciclo:

```text
Develop
 ↓
Discover
 ↓
Record
 ↓
Validate
 ↓
Knowledge improves
 ↓
Future agents become better
```

Éste puede ser uno de los mayores efectos de red de Quipu.

---

# 30. Fase G — Commercial / Enterprise Platform

Sólo después de las anteriores:

```text
multi-user
cloud
authentication
organizations
teams
billing
repository integrations
GitHub/GitLab
audit
security
secrets
SSO
RBAC
policy engine
usage
agent budgets
```

La versión actual es explícitamente local y su seguridad está orientada a atribución, no a defensa.

No conviene tratar eso como deuda urgente de la fase experimental, pero sí como requisito de comercialización.

---

# 31. Proyección 2026–2028

## Próximo horizonte: 6–12 meses

Los agentes seguirán mejorando en:

* contexto;
* ejecución;
* debugging;
* tests;
* tareas largas;
* múltiples agentes.

Esto ya está ocurriendo con Codex y Claude Code.

El valor se desplazará progresivamente de:

```text
"AI que programa"
```

hacia:

```text
"AI que ejecuta trabajo"
```

Por tanto, herramientas de control y coordinación ganarán relevancia.

---

## Horizonte 12–24 meses

Es razonable esperar:

```text
multi-agent
long-running
persistent memory
automatic review
background work
specialized agents
```

como comportamiento normal.

OpenAI ya describe agentes ejecutando trabajos de muchas horas y múltiples agentes en paralelo.

En ese entorno, un repositorio aislado ya no será suficiente como fuente de estado.

Será necesario algún tipo de:

```text
project control plane
```

Aquí Quipu puede encajar muy bien.

---

## Horizonte 24–36 meses

La principal transformación probablemente será:

```text
software engineering
=
specification
+
orchestration
+
verification
```

y no:

```text
software engineering
=
writing code
```

El ingeniero se convertirá cada vez más en:

```text
architect
+
product thinker
+
agent orchestrator
+
reviewer
```

Anthropic ya describe este cambio hacia arquitectura, estrategia, coordinación y evaluación como funciones centrales del ingeniero.

Esto favorece directamente la tesis de Quipu.

---

# 32. ¿Puede Quipu sobrevivir si los modelos mejoran muchísimo?

Sí, pero solamente si se posiciona correctamente.

Supongamos que en 2028 un modelo puede escribir correctamente el 95–99% de una tarea.

Quipu sigue teniendo utilidad si resuelve:

```text
¿qué tarea?
¿por qué?
¿qué restricciones?
¿qué decisiones?
¿qué arquitectura?
¿qué otros agentes trabajan?
¿qué cambió?
¿quién autorizó?
¿qué evidencia existe?
¿qué sigue?
```

Pero si Quipu solamente sirve para:

```text
crear bloques
crear tareas
guardar criterios
```

los propios agentes acabarán reproduciendo esas capacidades.

Por eso el objetivo debe ser:

> **ser el estado y la autoridad del proyecto, no simplemente su gestor de tareas.**

---

# 33. La mayor amenaza competitiva

No es Jira.

No es Linear.

No es Notion.

Es que:

```text
Claude Code
Codex
Cursor
GitHub Copilot
```

incorporen progresivamente:

```text
memory
skills
multi-agent
persistent context
planning
review
knowledge
project state
```

OpenAI ya está incorporando memoria, trabajo continuo y coordinación multiagente en Codex.

GitHub ya integra agent skills y MCP dentro de sus flujos de revisión.

Por eso Quipu no debería competir con ellos como interfaz del agente.

Debe ser **agnóstico al agente**.

```text
Claude
Codex
Gemini
OpenCode
Cursor
otros
   ↓
   Quipu
   ↓
project state
```

Ésta es una decisión estratégica crítica.

---

# 34. La oportunidad de Quipu

Si la arquitectura evoluciona correctamente, Quipu podría ocupar una posición parecida a:

```text
Git
```

pero para el **estado semántico y operativo del desarrollo agentic**.

Git responde:

> ¿Qué cambió?

Quipu podría responder:

> ¿Por qué cambió?

> ¿Qué debía cambiar?

> ¿Qué decisión lo autorizó?

> ¿Qué restricciones aplicaban?

> ¿Qué agente lo hizo?

> ¿Qué evidencia demuestra que está correcto?

> ¿Qué otros componentes pueden verse afectados?

> ¿Qué conocimiento nuevo produjo?

Eso es mucho más difícil de reemplazar que una interfaz de gestión.

---

# 35. Recomendación final sobre el desarrollo actual

No recomiendo detener el desarrollo de Quipu para rediseñar su arquitectura.

Tampoco recomiendo continuar simplemente implementando las features restantes del roadmap original sin reevaluarlas.

Recomiendo una tercera opción:

## Congelar conceptualmente el core

Mantener:

```text
Protocol Engine
Decision Chain
Requirement Chain
Block Workflow
Evidence
Human Authority
Baseline
MCP
Identity
Gates
```

como fundamento.

## Auditar el core

Antes de añadir capacidades nuevas:

```text
consistency
integrity
API completeness
MCP completeness
adoption correctness
knowledge correctness
security boundaries
test coverage
```

## Reorientar las siguientes fases

La prioridad debería ser:

```text
1. Core consolidation
2. Project Intelligence
3. Context Engine
4. Multi-Agent Orchestration
5. Agentic Verification
6. Continuous Memory
7. Enterprise/SaaS
```

---

# 36. Diagnóstico final

### Quipu actualmente

```text
             GOVERNANCE
                  ▲
                  │
       ┌──────────┴──────────┐
       │                     │
   WORKFLOW              EVIDENCE
       │                     │
       └──────────┬──────────┘
                  │
                MCP
                  │
                Agent
```

Es un **buen núcleo de control agentic**.

### Quipu que debería existir

```text
                 HUMAN
                   │
              governance
                   │
                   ▼
        ┌───────────────────────┐
        │         QUIPU         │
        │                       │
        │ Project Intelligence  │
        │ Project Memory        │
        │ Context Engine        │
        │ Decision Graph        │
        │ Requirement Graph     │
        │ Work Graph            │
        │ Evidence Graph        │
        │ Agent Orchestration   │
        │ Risk / Policy Engine  │
        │ Protocol Engine       │
        └───────────┬───────────┘
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       Agent A   Agent B   Agent C
          │         │         │
          └─────────┼─────────┘
                    ▼
                 CODEBASE
                    │
              tests/runtime
                    │
                    ▼
                 EVIDENCE
                    │
                    └──────────→ QUIPU
```

Ese segundo sistema tiene una propuesta mucho más fuerte.

---

# 37. Veredicto estratégico

**La dirección de Quipu es válida y, después de contrastarla con el estado actual del desarrollo agentic, considero que su tesis se vuelve más relevante, no menos.**

La industria está pasando de:

> agentes que escriben código

a:

> agentes que ejecutan procesos completos de ingeniería.

Y posteriormente:

> organizaciones de agentes que mantienen sistemas durante largos periodos.

El problema que aparece en cada transición es el mismo:

**estado, contexto, coordinación, memoria, restricciones y verificación.**

Quipu ya resuelve una parte sorprendentemente grande de:

```text
restricciones
gobernanza
estado de trabajo
dependencias
trazabilidad
evidencia
autoridad
```

Lo que falta desarrollar es precisamente lo que permitirá que ese core escale hacia el futuro:

```text
comprender
recordar
contextualizar
coordinar
verificar
aprender
```

Por tanto, **no veo la versión actual como una versión prematura que haya que descartar**.

La veo como:

> **un primer núcleo funcional de una infraestructura de control para desarrollo agentic.**

La decisión estratégica importante ahora no es "qué features faltan para terminar Quipu".

Es:

> **¿Cómo convertimos el core actual en el sistema que mantiene la verdad operativa de un proyecto mientras cada vez más agentes autónomos trabajan sobre él?**

Esa pregunta debería gobernar las siguientes fases.

Y hay un punto adicional que considero especialmente importante: **el siguiente salto de Quipu no debería medirse por cantidad de funcionalidades, sino por cuánto trabajo autónomo puede ejecutar un conjunto de agentes sin perder coherencia del proyecto.**

Ese debería convertirse en el principal KPI técnico del producto.

### Métricas futuras que propondría

```text
Context Accuracy
¿Cuánto del contexto entregado al agente era realmente relevante?

Requirement Coverage
¿Qué porcentaje de cambios tiene origen y criterios verificables?

Decision Compliance
¿Qué porcentaje de cambios respeta las decisiones vigentes?

Agent Rework Rate
¿Cuánto trabajo generado por agentes debe rehacerse?

Evidence Coverage
¿Qué porcentaje del trabajo tiene evidencia suficiente?

Human Intervention Rate
¿Cuántas intervenciones humanas requiere cada bloque?

Agent Conflict Rate
Cuántos conflictos producen agentes concurrentes?

Knowledge Freshness
Qué porcentaje del conocimiento sigue coincidiendo con el código real?

Long-Horizon Success
Qué porcentaje de features completas puede ejecutar el sistema sin pérdida de coherencia?

Project Recovery Time
Cuánto tarda un agente nuevo en comprender un proyecto existente?
```

**Si Quipu consigue demostrar mejoras significativas en esas métricas, entonces deja de ser simplemente una herramienta de desarrollo y empieza a convertirse en infraestructura para el desarrollo agentic.**
