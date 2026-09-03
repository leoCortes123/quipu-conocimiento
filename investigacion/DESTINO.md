# DESTINO — a dónde va cada hallazgo

La investigación alimenta **dos cosas distintas** y confundirlas es el riesgo principal de
esta fase (`../../README.md` § frontera 2):

- **PRODUCTO** → entra en el plan (`../plan/`) y acaba en specs de `QUIPU_ENTERPRISE/openspec/`.
- **MÉTODO** → entra en `../../metodologia/`, reescrito como norma vinculante.

Un mismo documento puede alimentar los dos, pero **cada hallazgo concreto va a uno solo**.
Si un hallazgo parece ir a los dos, casi siempre es que dice «Quipu debería imponer X» y
además «nosotros deberíamos trabajar con X»: la primera mitad es producto, la segunda es
método, y la segunda **no puede depender de que Quipu funcione**.

## Primera pasada — a confirmar al redactar el plan

Clasificación por documento, según el tema que declara cada uno. No sustituye a leerlos:
marca por dónde empezar y evita releer los trece en cada sesión.

| # | Documento | PRODUCTO | MÉTODO | Nota |
|---|---|---|---|---|
| 00 | `00-metodo.md` | — | — | cómo se hizo la investigación y qué no cubre |
| 01 | `01-panorama-y-evidencia.md` | ● | ○ | evidencia de campo; el dato de mantenibilidad (refactor −70 %, duplicación +81 %) también obliga al método |
| 02 | `02-especificacion-requisitos-y-norma.md` | ● | ○ | spec-driven y EARS ya están en el método; ambigüedad y contradicción son producto |
| 03 | `03-arquitectura-planificacion-y-coordinacion.md` | ● | ● | multi-agente y coste ×15 es decisión de método; Git como sustrato ya lo es |
| 04 | `04-contexto-memoria-y-codigo.md` | ○ | ● | **el hueco mayor del método**: ingeniería de contexto, decaimiento de gobierno, qué sobrevive al resumen |
| 05 | `05-verificacion-evidencia-y-procedencia.md` | ● | ● | núcleo de las dos: para el producto es el runner de evidencia (DEC-3); para el método, `../../metodologia/normativa/VALIDACION.md` |
| 06 | `06-gobierno-autoridad-y-seguridad.md` | ● | ○ | autoridad y permisos son producto; sandboxing por harness es método |
| 07 | `07-ejecucion-observabilidad-y-evaluacion.md` | ● | ○ | máquinas de estado y trazas son producto; métricas de flota, método |
| 08 | `08-taxonomia-de-fallos.md` | ○ | ● | **hueco del método**: hoy hay escalamiento, no recuperación por modo de fallo |
| 09 | `09-largo-plazo-y-abstracciones-duraderas.md` | ● | ○ | qué partes de la arquitectura deben ser estables |
| 10 | `10-comparacion-con-quipu.md` | ● | — | gap analysis área por área: insumo directo del plan |
| 11 | `11-tesis-arquitectonica.md` | ● | — | las 13 preguntas y el cambio mínimo que se deduce: insumo directo del plan |
| 12 | `12-fuentes.md` | — | — | bibliografía |

● principal · ○ secundario · — no aplica

## Qué falta al método, según esto

Contra lo que hoy cubre `../../metodologia/` (contrato de entrada, constitución,
escalamiento, validación, handoff, tres roles, flujo OpenSpec y scripts):

| Hueco | Fuente |
|---|---|
| Playbook de recuperación por modo de fallo | `08` — 27 modos con detección y recuperación |
| Ingeniería de contexto: qué va en el prefijo, presupuesto por rol, qué hacer al compactar | `04` — una restricción que sobrevive al resumen se obedece el 100 %; una que se pierde, se viola el 38 % |
| Criterio para abrir o no un fan-out multi-agente | `03` — coste ×15, ganancia sólo si el trabajo es partible |
| Memoria de decisiones del propio desarrollo | `../06-banco-de-conocimiento.md`, aplicado al método y no al producto |
| Metodología de planeación: cómo se redacta un ExecPlan y una tarea | — (hay 13 ejemplos en `../../ejecucion/ARCHIVO/fases/`, ninguna plantilla normativa) |
| Apertura y cierre de fase como acto normado | — |
| Cómo se cambia la metodología misma | ya declarado en `../../metodologia/README.md`; falta el procedimiento |
| Evaluación y métricas de la flota | `07` — `metricas.sh` hoy sólo sirve con OpenCode |
