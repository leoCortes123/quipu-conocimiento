# conocimiento — investigación, memoria y el plan completo

Repositorio del **porqué**: qué debe llegar a ser Quipu Enterprise como herramienta de
desarrollo dirigido por agentes, con qué evidencia se sostiene, y el plan completo que se
deriva de ahí. Destino comercial SaaS; primer caso real previsto: absorber el desarrollo
de `chasqui_n8n`.

**Nada de esto modifica el repositorio de Quipu**: es diseño e investigación.

## Dos avisos de frontera

1. **Esto no es la metodología de desarrollo.** Las arquitecturas y metodologías que aquí
   se investigan son material de **diseño del producto**. Cómo trabajan los agentes que
   construyen Quipu se decide en `../metodologia/`, y Quipu no se usa para desarrollar
   Quipu. Ver `../README.md` § frontera 2.
2. **La ejecución no lee este repositorio.** Ni siquiera el plan completo de `plan/`: de
   él sólo sale la fase que el humano publica en `../ejecucion/fase-activa/`.

## Estructura

| Ruta | Qué es |
|---|---|
| `plan/` | **el plan completo**: vigente, borradores y planes superados |
| `memoria/` | el porqué de las decisiones — `decisiones/DEC-*.md` |
| `investigacion/` | estado del arte 2026, con `DESTINO.md`: qué hallazgo va al producto y cuál al método |
| `analisisExterno/` | auditoría interna contra el código y la base viva, y el análisis estratégico externo |
| `artifacts/` | informes navegables |
| `01`, `03`, `04`, `05`, `06`, `PROMPT-*` | documentos de encargo, registro histórico |

> ## ⚠️ Estado al 2026-09-01 — leer antes que nada
>
> Este plan está **parcialmente obsoleto**. Una auditoría contra el código, el esquema desplegado
> y la base de datos viva encontró tres huecos estructurales que el plan no contempla, y midió que
> **F2 (puente Chasqui) ya se intentó y ya falló** el 2026-08-28 por causas estructurales.
>
> **Punto de entrada actual:** [`analisisExterno/auditoria-interna-2026-09-01.md`](analisisExterno/auditoria-interna-2026-09-01.md)
> — dosier de traspaso con los hechos, sus comandos de reproducción, el roadmap revisado, las
> decisiones pendientes del humano y el trabajo de la sesión siguiente.
> Informe navegable: https://claude.ai/code/artifact/b6f6a235-5276-4af2-86ff-e9f953de87c7
>
> Los documentos 1–5 de abajo **siguen siendo válidos como registro histórico** del encargo tal como
> se dio; el `03-metodologia-agentes.md` sigue vigente en su totalidad. No se reescriben: hacerlo
> falsificaría contra qué se verificó el trabajo de F0 y F1.

## Documentos

| # | Documento | Qué contiene |
|---|---|---|
| 1 | [`01-auditoria-repositorio-quipu.md`](01-auditoria-repositorio-quipu.md) | Estado real del repo, qué limpiar, qué conservar, política entre QUIPU v1 y ENTERPRISE |
| 2 | [`plan/archivo/02-plan-fases.md`](plan/archivo/02-plan-fases.md) | El plan detallado por fases, priorizado para absorber Chasqui cuanto antes. **Superado** — el plan vive ahora en [`plan/`](plan/README.md) |
| 3 | [`03-metodologia-agentes.md`](03-metodologia-agentes.md) | Investigación 2026: orquestación multi-agente y verificación de código generado por IA, aplicada a Quipu |
| 4 | [`04-sistema-agentico-flota.md`](04-sistema-agentico-flota.md) | FLOTA: definición completa del sistema de desarrollo agéntico autónomo (roles, estados, autonomía L1–L4, fallos, ejecución del plan) |
| 5 | [`05-arquitectura-sistema-agentico.md`](05-arquitectura-sistema-agentico.md) | Guía de arquitectura: por qué Quipu es el orquestador, componentes, flujo canónico, adopción como producto y riesgos |
| — | [`archivo/sistema-a/`](../archivo/sistema-a/LEEME.md) | **Paquete ejecutable del Sistema A**, hoy repartido entre `../metodologia/` y `../ejecucion/` (completar Quipu): AGENTS.md, CONSTITUCION, ESCALAMIENTO, VALIDACION, HANDOFF, PLAN/F0 + 6 tareas atómicas. F0 y F1 cerradas |
| 6 | [`analisisExterno/auditoria-interna-2026-09-01.md`](analisisExterno/auditoria-interna-2026-09-01.md) | **Auditoría contra el código y la base viva.** Hechos reproducibles, diagnóstico del core, gap analysis, roadmap revisado F-A…F-H, decisiones pendientes. **Empieza aquí** |
| 7 | [`analisisExterno/InformeEstratégicoQuipu.md`](analisisExterno/InformeEstratégicoQuipu.md) | Análisis estratégico externo (hecho sólo sobre las guías, sin leer el código). Marco válido; puntuaciones corregidas en el §8 del documento 6 |
| 8 | [`investigacion/`](investigacion/README.md) | **Investigación profunda del estado del arte 2026** (2026-09-01): metodologías, arquitecturas y evidencia para ingeniería de software con agentes, con comparación explícita contra Quipu y tesis arquitectónica. Es investigación, **no un plan** |
| 9 | [`06-banco-de-conocimiento.md`](06-banco-de-conocimiento.md) | **Banco de conocimiento** (2026-09-01): metodología para registrar el porqué de las decisiones de diseño — tres registros (conocimiento / decisiones / código), protocolo de captura, caducidad y plan de arranque por retro-extracción. Es `RECOMENDACIÓN`, **no un plan**. Informe navegable: https://claude.ai/code/artifact/718c0a24-375e-4ccc-a2ba-f53d6b200a8b |

## Contexto en tres párrafos

Chasqui (`chasqui_n8n`) tiene la metodología más madura que existe sobre esto: decisiones
normativas con cadena supersede, invariantes con evidencia, pedidos como máquina de estados,
protocolo obligatorio antes de modificar, verificador mecánico. Su debilidad: todo vive en
archivos de texto sin integridad transaccional.

Quipu Enterprise ya resolvió la otra mitad con rigor mayor al propuesto inicialmente: BD
Postgres como regla de verdad (triggers/CHECKs, estados imposibles de violar), cadena de
demanda necesidad→cambio→requisito→evidencia→firma, agentes identificados y limitados
(ningún tool de aprobación), Web UI, adopción brownfield mecánica, y flujo OpenSpec propio.

Lo que falta es la mitad normativa de Chasqui (decisiones/invariantes/contradicción), la
capa guiada para agentes (skills), la verificación de resultados generados por IA según el
estado del arte 2026, y la envoltura SaaS. El plan cubre las cuatro, priorizando el puente
con Chasqui.

## Decisión clave registrada

No construir una herramienta nueva ni competir con Spec Kit/Kiro en su terreno: evolucionar
Quipu Enterprise, cuyo diferencial es que **el protocolo es mecánico, no convención** — un
estado inválido no está desaconsejado, es imposible. Ninguna herramienta analizada del
mercado (Spec Kit, OpenSpec, Kiro, Taskfolk, Task Master, Port) tiene eso.
