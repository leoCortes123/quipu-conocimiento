# 07 — Ejecución durable, observabilidad y evaluación

Cubre las áreas 16, 17, 18 y 19 del encargo.

## 7.1 Máquinas de estado y motores de proceso: los precedentes de Quipu

`HECHO` (`E2`, 2026) — La **ejecución durable** es una clase de motor bien definida y con múltiples
implementaciones en producción: Temporal, Restate, DBOS, Inngest, Hatchet, Cloudflare Workflows,
AWS Step Functions / Lambda Durable, Azure Durable Task. La propiedad que define a la clase:
**cada paso se registra en un journal, cada paso es reintentable de forma independiente, y el
estado de «dónde vas» vive en el store durable, no en el worker en memoria**. Restate usa el journal
para impedir la ejecución duplicada al reanudar.

`HECHO` (`E3`, 2026) — El stack de producción que se ha vuelto común para agentes largos no es un
framework de agentes solo, sino **framework de agentes + motor durable** (LangGraph + Temporal, o
+ Restate, o + DBOS). El propio ecosistema lo formula así: para un agente que dura más que un café,
el framework de agentes no basta.

> `INFERENCIA` — El enfoque de Quipu —bloques, estados, criterios, dependencias y transiciones como
> filas en Postgres con triggers— **tiene precedente sólido y es la misma idea**: el estado del
> trabajo vive en un store transaccional y el ejecutor es reemplazable. La diferencia es de
> intención: Temporal garantiza que el flujo *se complete*; Quipu garantiza que el flujo *no avance
> sin cumplir condiciones*. Son duales. Un motor durable protege contra la pérdida de trabajo; un
> motor de puertas protege contra la aceptación de trabajo malo.
>
> `INFERENCIA` — Y por eso duplicar el estado en un motor externo sería un error: el estado del
> trabajo ya es transaccional. Lo que Quipu **no** tiene y los motores durables sí, y es lo único
> que merece copiarse, es el **journal por paso con reanudación e idempotencia**: hoy un worker que
> muere retiene el bloque indefinidamente (H-22), que es exactamente el fallo que un lease con TTL y
> heartbeat resuelve, y que la práctica de agentes ya adoptó con valores por defecto conocidos
> (§3.4).

`INFERENCIA` — Lo que la evidencia dice que hace falta para tareas largas, con su forma mínima:

| Problema | Forma mínima | Precedente |
|---|---|---|
| Crash del worker | lease con TTL + heartbeat + reclamación automática | `E3` práctica de agentes; `E2` motores durables |
| Pérdida de contexto | el contexto se deriva del estado externo, no se acumula | `E1` §4.1 |
| Reintentos | presupuesto explícito de intentos, con salida a escalamiento | `E3` |
| Reanudabilidad | la unidad de trabajo es atómica y su estado es una fila | `E2` |
| Completado parcial | estado intermedio explícito, nunca implícito en «lo que hay en el disco» | `E2` |
| Idempotencia | la operación de avance de estado debe ser idempotente | `E2` |
| Compensación | transición de reversión declarada, no inferida | `E2` |

## 7.2 Observabilidad de sistemas agénticos

`HECHO` (`E2`, jun-2026) — OpenTelemetry: en la versión v1.42.0 (12-jun-2026) los atributos `gen_ai.*`
se movieron a un repositorio dedicado de convenciones GenAI, pero **siguen siendo pre-estables y
experimentales, sin 1.0**. Cubren spans de cliente LLM, **spans de agente** para flujos multi-paso,
eventos para prompt/completion, métricas agregadas, y convenciones nuevas para MCP. Copilot, Codex y
Claude Code ya emiten con estas convenciones.

`INFERENCIA` — El estado «adoptado pero no estable» tiene una consecuencia práctica: es correcto
**emitir** hacia esas convenciones y es prematuro **depender** de ellas como esquema interno. La
distinción importa: una plataforma cuyo modelo de datos sea el estándar de otro queda atada a los
cambios de ese estándar, y este está declarado como sujeto a cambios de nombres.

`INFERENCIA` — Lo que una plataforma de gobierno necesita observar es distinto de lo que necesita
una plataforma de LLMOps. LLMOps observa el *modelo*: latencia, tokens, coste, calidad de la salida.
Una plataforma de gobierno observa el **acto**: qué se reclamó, con qué alcance, qué normas se
entregaron, qué comandos se ejecutaron con qué resultado, qué puerta rechazó y por qué, quién firmó.
La segunda lista es la que sostiene una auditoría; la primera es la que sostiene una factura.
Ambas hacen falta, pero confundirlas lleva a instrumentar la que ya instrumenta el proveedor del
modelo.

## 7.3 Evaluación: más allá del benchmark

`HECHO` (`E1`, 2026) — Los benchmarks de agentes tienen tres problemas estructurales medidos y
documentados en §1.2: contaminación (32,67 % de leakage en SWE-bench Verified), tests defectuosos
(59,4 % de los fallos auditados de un modelo), y sensibilidad a la infraestructura de evaluación
(METR midió diferencias estadísticamente significativas entre dos harnesses para los mismos
modelos).

`HECHO` (`E5`, abr-2026) — Y hay un fenómeno que descalifica el uso ingenuo de benchmarks para
decidir: Opus 4.7 subió en 12 de 14 benchmarks (SWE-bench Verified de 80,8 % a 87,6 %) y en 48 horas
había reportes masivos de regresión percibida en uso real. Sea cual sea la explicación, **el
benchmark y el uso real se desacoplaron**.

`HECHO` (`E3`, 2026) — La regresión al cambiar de modelo es silenciosa por construcción: el endpoint
sigue devolviendo 200, las salidas simplemente derivan —cambia el formato de las tool calls, se
afloja la adherencia al JSON, se mueven los límites de rechazo, cambia el presupuesto de tokens—.
La práctica recomendada es fijar la versión del modelo en producción y correr evals propios contra
la nueva antes de migrar.

> `INFERENCIA` — Para una plataforma que gobierna agentes, esto significa que **la versión del
> modelo es un dato de primera clase de la evidencia**, no un metadato opcional. Sin él no se puede
> contestar la pregunta que va a aparecer inevitablemente: «¿esta regresión empezó cuando cambiamos
> de modelo?». Es un campo, no un proyecto.

`INFERENCIA` — Qué debería medir una plataforma de gobierno, deducido del conjunto de la evidencia y
filtrado por «¿es medible con lo que la plataforma ya ve?»:

| Métrica | Qué problema real detecta | Evidencia de que el problema existe |
|---|---|---|
| % de criterios cerrados con evidencia **observada** (vs declarada) | expediente que no prueba nada | §5.3 reward hacking |
| Tasa de retrabajo por tarea (ciclos hasta cerrar) | calidad agéntica real | MAST: repetición 17,14 % |
| Tasa de regresión: cierres que rompieron algo cerrado antes | daño invisible del volumen | DORA 2025: inestabilidad |
| Cobertura de norma: % de cambios cuyo alcance intersectó una norma **y trae declaración** | norma que no gobierna | H-06/H-08; §2.5 |
| Frescura: sospechas pendientes / enlaces totales | conocimiento obsoleto | STALE 55,2 % |
| Precisión de contexto: entregado vs citado en la evidencia | contexto inflado que degrada | Context Rot |
| Tasa de conflicto de alcance entre agentes | coordinación real | §3.4 |
| **Paradas humanas por unidad cerrada, y % que cambió el resultado** | sello de goma | §6.2 |
| Coste por unidad **cerrada** (no por tarea intentada) | economía real | §1.4 |
| Éxito a horizonte largo: features completas sin pérdida de coherencia | lo que de verdad importa | HORIZON, §9 |

`INTERPRETACIÓN` — La octava métrica es la que ninguna plataforma publica y la que mejor distingue
gobierno real de teatro de gobierno. Una organización que descubra que el 100 % de sus aprobaciones
humanas confirman lo propuesto tiene un dato accionable: o el filtro mecánico previo es bueno y hay
que reducir las paradas, o la supervisión ya se degradó y hay que reducir el volumen.

## 7.4 Evaluación de los propios agentes

`INFERENCIA` — El encargo pregunta cómo evaluar agentes más allá de benchmarks. La respuesta que la
evidencia permite sostener es que **la evaluación útil es longitudinal y propia, no transversal y
pública**: el historial de evidencia observada del propio proyecto dice más sobre si este agente,
en esta capacidad, merece más autonomía, que cualquier puntuación de benchmark.

Esto convierte los niveles de autonomía en una **consulta**, no en una configuración: si el nivel de
un agente en una capacidad sale de contar sus cierres con evidencia observada, sus regresiones y
sus violaciones de guardarraíl, entonces subir o bajar de nivel deja de ser una decisión y pasa a ser
un hecho derivado. `RECOMENDACIÓN` — Es, junto con la evidencia observada, la propiedad más difícil
de replicar de las que aparecen en esta investigación, porque exige haber acumulado historial en un
store transaccional; no se puede improvisar en una sesión de agente ni en ficheros markdown.
