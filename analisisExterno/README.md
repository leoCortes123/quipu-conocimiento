# analisisExterno — dónde empezar

Dos análisis sobre la misma pregunta, hechos con evidencia distinta. **Léelos en este orden.**

| # | Documento | Base de evidencia | Qué vale |
|---|---|---|---|
| 1 | **[`auditoria-interna-2026-09-01.md`](auditoria-interna-2026-09-01.md)** | código fuente, esquema desplegado, base de datos viva, CI reejecutado, sondeos HTTP | **Empieza aquí.** Es el dosier de traspaso: hechos con su comando de reproducción, roadmap revisado, decisiones pendientes y el trabajo concreto de la sesión siguiente |
| 2 | [`InformeEstratégicoQuipu.md`](InformeEstratégicoQuipu.md) | sólo `GUIA-USUARIO.md` y la guía técnica — **no leyó el código** | Marco estratégico y proyección 2026–2028: sólido. Puntuaciones del §23: **corregidas** en el §8 del dosier |

## Por qué el orden importa

El informe externo puntúa «Evidencia 8,5/10» y «Trazabilidad 9/10». La auditoría interna midió que
Quipu **no ejecuta nada** (la evidencia es un `TEXT` que el agente pega) y que la cadena de gobierno
**no alcanza a un proyecto adoptado**. Las dos afirmaciones no son compatibles, y el código decide.

Leer primero el informe externo lleva a un plan construido sobre puntuaciones que el código
desmiente.

## Informe publicado

Versión navegable de la auditoría interna:
https://claude.ai/code/artifact/b6f6a235-5276-4af2-86ff-e9f953de87c7

## Estado al 2026-09-01

- F0 y F1 **cerradas**; ninguna fase activa.
- `developTool/02-plan-fases.md` está **parcialmente obsoleto**: F2 se aplaza, F3 se adelanta. Ver §10 del dosier.
- Hay **8 decisiones pendientes del humano** (§14 del dosier). Sin DEC-1 y DEC-2 el plan nuevo no se puede escribir.
