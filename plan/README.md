# plan — el plan completo de Quipu

Aquí vive el plan **entero**: todas las fases, sus dependencias y su porqué. Es el
producto de la planeación, no de la ejecución, y por eso vive en el repositorio de
conocimiento, junto a la investigación y la auditoría de las que se deriva.

**La ejecución no lee este directorio.** Cuando el humano abre una fase se publica sólo
esa fase en `../../ejecucion/fase-activa/`. Ver `../../README.md` § frontera 1.

## Estado al 2026-09-03

**No hay plan vigente redactado.** El anterior (`archivo/02-plan-fases.md`, F0–F5) quedó
parcialmente obsoleto: la auditoría del 2026-09-01 midió que F2 ya se intentó y falló por
causas estructurales, y propuso un roadmap distinto. F0 y F1 sí se ejecutaron y cerraron
bajo el plan viejo; su registro está en `../../ejecucion/ARCHIVO/fases/`.

El plan nuevo está por escribir. Sus insumos, en orden de peso:

| Insumo | Qué aporta | Dónde |
|---|---|---|
| Auditoría interna 2026-09-01 | hechos reproducibles, diagnóstico del core, roadmap F-A…F-H, decisiones DEC-1…DEC-8 | `../analisisExterno/auditoria-interna-2026-09-01.md` |
| Investigación profunda 2026 | estado del arte, gaps de Quipu área por área, tesis arquitectónica | `../investigacion/` (empezar por `10`, `11`) |
| Deuda consolidada de F1 | bloqueantes, deuda técnica y de entorno ya clasificadas | `../../ejecucion/ESTADO/DEUDA-F2.md` |
| Estado real del producto | capabilities, herencia congelada, tests | `../../ejecucion/ESTADO/ESTADO.md` |

## Bloqueo declarado

La auditoría (§15) lo dice sin rodeos: **sin DEC-1 y DEC-2 el plan no se puede escribir.**

| Decisión | De qué depende |
|---|---|
| DEC-1 · modelo de autenticación | fija si F-A dura 3 días o 2 semanas; bloquea todo lo demás |
| DEC-2 · aplazar F2 / Chasqui | define si Quipu aspira a gobernar o se conforma con ser la prueba |
| DEC-3 · alcance del runner de evidencia | la pieza más importante de F-B; condiciona la promesa comercial |
| DEC-4 · destino de la capa de diseño | ~10 tablas, pantallas y `ImpactAnalyzer` |
| DEC-5 · modelo normativo único | `invariante` vs `business_rule` |

Se registran resueltas en `../memoria/decisiones/`, una por archivo. DEC-6, DEC-7 y DEC-8
no bloquean el plan y además son decisiones de **método**, no de producto: viven en
`../../metodologia/decisiones/` como DEC-M2, DEC-M1 y DEC-M3.

## Convención

| Ruta | Qué es |
|---|---|
| `PLAN.md` | el plan vigente, cuando exista. Único, completo, con estado por fase |
| `borrador/` | plan en redacción, aún no aprobado por el humano |
| `archivo/` | planes superados. No se reescriben: son registro de qué se encargó |

Un plan pasa de `borrador/` a `PLAN.md` por acto humano explícito. Una fase pasa de
`PLAN.md` a `../../ejecucion/fase-activa/` por otro acto humano explícito.
