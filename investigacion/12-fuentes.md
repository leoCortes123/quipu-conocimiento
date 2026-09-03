# 12 — Fuentes

Ordenadas por área, con año y grado de evidencia (`E1` experimental · `E2` industrial medido ·
`E3` consenso técnico · `E4` propuesta teórica · `E5` opinión/anécdota · `E6` marketing).
Las fuentes `E6` se listan porque se citan en el texto **como ejemplo de lo que no es evidencia**.

## Arquitectura de agentes y fallos multi-agente

| Fuente | Año | Grado |
|---|---|---|
| Cemri et al., *Why Do Multi-Agent LLM Systems Fail?* — arXiv:2503.13657 (v1 mar-2025, v3 oct-2025), NeurIPS 2025. Taxonomía MAST | 2025 | `E1` |
| Rombaut, *Inside the Scaffold: A Source-Code Taxonomy of Coding Agent Architectures* — arXiv:2604.03515v2 | 2026 | `E1` |
| Anthropic, *How we built our multi-agent research system* (orchestrator-worker; +90,2 %; ~15× tokens; 80 % de la varianza explicada por tokens) | 2025 | `E2` |
| Cognition (Walden Yan), *Don't Build Multi-Agents* — cognition.com/blog | 2025 | `E5` |
| Cognition, revisión de postura: escrituras single-threaded | 2026 | `E5` |
| Xia et al., *Agentless* (pipeline localizar-reparar-validar) | 2024 | `E1` |
| Wang et al., *ReAcTree* — arXiv:2511.02424 (31,00 % vs 13,00 %) | 2025 | `E1` |
| *Hierarchical Task Network Planning with LLM-Generated Heuristics* — arXiv:2605.07707 | 2026 | `E1` |

## Contexto y memoria

| Fuente | Año | Grado |
|---|---|---|
| Hong, Troynikov, Huber (Chroma), *Context Rot: How Increasing Input Tokens Impacts LLM Performance* — 18 modelos | 2025 | `E1` |
| **Chen, *Governance Decay: How Context Compaction Silently Erases Safety Constraints in Long-Horizon LLM Agents* — arXiv:2606.22528v2. Benchmark ConstraintRot: 1.323 episodios, 7 familias de modelos** | 2026 | `E1` |
| Anthropic, *Effective context engineering for AI agents* (compaction, note-taking, sub-agent quarantine) | 2025 | `E2` |
| Anthropic, *Agent Skills* / SKILL.md como estándar abierto (divulgación progresiva en tres capas) | 2025-26 | `E2` |
| Benchmark STALE — detección de invalidación silenciosa, 1.200 consultas, 55,2 % en el mejor modelo | 2026 | `E1` |
| LoCoMo / LongMemEval / BEAM y los resultados contradictorios de Mem0 (49 %, 93,4 %, 94,4 %) | 2026 | `E6` |
| *Beyond Compaction: Structured Context Eviction for Long-Horizon Agents* — arXiv:2606.11213 | 2026 | `E1` |
| *Slipstream: Trajectory-Grounded Compaction Validation* — arXiv:2605.08580 | 2026 | `E1` |

## Verificación, evaluación y evidencia

| Fuente | Año | Grado |
|---|---|---|
| Bornholt et al., *Systems Correctness Practices at AWS* — ACM Queue 22(6). Portafolio escalonado; TLA+ en >10 sistemas críticos | 2025 | `E2` |
| Amazon Science, *Using Lightweight Formal Methods to Validate a Key-Value Storage Node in Amazon S3* (ShardStore) | 2021 | `E2` |
| Foster et al., *Mutation-Guided LLM-based Test Generation at Meta* — arXiv:2501.12862, FSE 2025 Industry. ACH: 73 % de aceptación | 2025 | `E2` |
| Meta Engineering, *LLMs are the key to mutation testing and better compliance* | 2025 | `E2` |
| *Shrinking the Generation-Verification Gap with Weak Verifiers* (Weaver) — arXiv:2506.18203, NeurIPS 2025. −14,5 % de brecha | 2025 | `E1` |
| *Reliability without Validity: … LLM-as-a-Judge across Agreement, Consistency, and Bias* — arXiv:2606.19544. 21 jueces, 3 benchmarks | 2026 | `E1` |
| *The Coin Flip Judge? Reliability and Bias in LLM-as-a-Judge* — arXiv:2606.13685 | 2026 | `E1` |
| *LongJudgeBench* — arXiv:2606.01629 | 2026 | `E1` |
| OpenAI, *Why we no longer evaluate SWE-bench Verified* (contaminación; 32,67 % leakage; 59,4 % de fallos por tests defectuosos) | 2026 | `E2` |
| *SWE-bench Pro* — arXiv:2509.16941 (276 tareas de 18 repos comerciales privados) | 2025 | `E1` |
| *UTBoost: Rigorous Evaluation of Coding Agents on SWE-Bench* — arXiv:2506.09289 | 2025 | `E1` |
| OSS-Fuzz, generación de fuzz targets con LLM (0-31 % de cobertura adicional) | 2024-25 | `E2` |
| Benchmarks de reward hacking: EvilGenie (arXiv:2511.21654), ImpossibleBench, SpecBench (arXiv:2605.21384), BenchJack (arXiv:2605.12673) | 2025-26 | `E1` |
| Casos documentados de reward hacking en Claude 3.7 Sonnet, o1-preview y DeepSeek-R1 (Palisade) | 2025 | `E2` |

## Especificación, requisitos y protocolo

| Fuente | Año | Grado |
|---|---|---|
| **He & Yu, *Protocol-Driven Development: Governing Generated Software Through Invariants and Continuous Evidence* — arXiv:2605.12981v3. Sin evaluación empírica** | 2026 | `E4` |
| *Requirements Ambiguity Detection and Explanation with LLMs: An Industrial Study* — IEEE (dataset JWST + defectos inyectados) | 2025-26 | `E1` |
| *Assessing the Impact of Requirement Ambiguity on LLM-based Function-Level Code Generation* — arXiv:2604.21505 | 2026 | `E1` |
| *Towards an Agentic LLM-based Approach to Requirement Formalization from Unstructured Specifications* — arXiv:2604.18228 | 2026 | `E1` |
| GitHub Spec Kit (MIT, neutral respecto al agente) | 2025-26 | `E2` |
| AWS Kiro: EARS + Requirements Analysis con solvers SMT | 2026 | `E2` |
| Cifras de eficacia de spec-driven publicadas por proveedores («orden de magnitud», «3-10×», «40 h → 8 h») | 2026 | `E6` |
| ADR (adr.github.io) y práctica 2026 sobre ADRs generados por agentes | 2011-2026 | `E3` |

## Gobierno, seguridad y protocolos

| Fuente | Año | Grado |
|---|---|---|
| **Kang & Diponegoro, *Governance Gaps in Agent Interoperability Protocols: What MCP, A2A, and ACP Cannot Express* — arXiv:2606.31498** | 2026 | `E4` |
| Beurer-Kellner et al., *Design Patterns for Securing LLM Agents against Prompt Injections* — arXiv:2506.08837 (seis patrones) | 2025 | `E1` |
| Debenedetti et al. (Google DeepMind), *Defeating Prompt Injections by Design* (CaMeL) — arXiv:2503.18813 | 2025 | `E1` |
| *CaMeLs Can Use Computers Too* — arXiv:2601.09923 | 2026 | `E1` |
| *AIP: Agent Identity Protocol for Verifiable Delegation Across MCP and A2A* — arXiv:2603.24775 | 2026 | `E4` |
| *Security Threat Modeling for Emerging AI-Agent Protocols (MCP, A2A, Agora, ANP)* — arXiv:2602.11327 | 2026 | `E4` |
| *Open Challenges in Multi-Agent Security* — arXiv:2505.02077 | 2025 | `E4` |
| MCP: especificaciones 2025-11-25 y 2026-07-28; roadmap (OAuth 2.1, CIMD, audit trails) | 2025-26 | `E2` |
| A2A v1.0 con Agent Cards firmados (>150 organizaciones) | 2026 | `E2` |
| AWS, *Enforce least-privilege authorization in multi-agent AI chains using Cedar*; Cedar en Bedrock AgentCore Policy | 2026 | `E2` |
| Open Policy Agent / Rego (CNCF graduated) | — | `E2` |
| Red Hat Emerging Technologies, *Zero trust for AI agents: why delegation beats impersonation* | 2026 | `E3` |
| SLSA v1.1 / in-toto attestation framework / cosign | 2023-26 | `E2` |
| Sandboxing de agentes: Firecracker, Kata, gVisor; `@anthropic-ai/sandbox-runtime` (Seatbelt/bubblewrap + proxy) | 2026 | `E2` |
| Estudio en *Radiology*: 27 radiólogos, caída de 82 % a 45,5 % con IA incorrecta | 2023 | `E1` |
| Revisión sistemática de 35 estudios sobre acuerdo con recomendaciones IA incorrectas | 2025 | `E1` |

## Productividad, calidad y economía

| Fuente | Año | Grado |
|---|---|---|
| METR, *Measuring the Impact of Early-2025 AI on Experienced Open-Source Developer Productivity* — arXiv:2507.09089. RCT, 16 devs, 246 tareas, **+19 % de tiempo** | 2025 | `E1` |
| METR, *Time Horizon 1.1* (228 tareas; duplicación 89-131 días; caveats declarados) | 2026 | `E1` |
| METR, *Measuring AI Ability to Complete Long Tasks* (metodología original) | 2025 | `E1` |
| DORA, *State of AI-assisted Software Development* (~5.000 profesionales; throughput ↑, inestabilidad ↑) | 2025 | `E2` |
| GitClear, *The Maintainability Gap* — 623M de cambios 2023-2026; refactor −70 %, duplicación +81 % | 2026 | `E2` |
| GitClear, *AI Copilot Code Quality* (2024: primer año con copy/paste > código movido) | 2025 | `E2` |
| Bhati, *Agentic AI in the Software Development Lifecycle* — arXiv:2604.26275v1 (arquitectura de 6 capas; 5 problemas abiertos) | 2026 | `E4` |
| Wang, Bai, Sun et al., *The Long-Horizon Task Mirage?* — arXiv:2604.11978 (HORIZON; 3.100+ trayectorias; 72,5 % fallos de proceso) | 2026 | `E1` |
| Anthropic, *2026 Agentic Coding Trends Report* (brecha de delegación: 60 % de uso, 0-20 % de delegación completa) | 2026 | `E5` |
| Estudios de campo Microsoft (+12,92-21,83 % PR/semana) y Accenture (+7,51-8,69 %) citados en el survey | 2024-25 | `E2` |
| Datos de coste por tarea y consumo de tokens agénticos; caso Uber | 2026 | `E5` |

## Ejecución durable y observabilidad

| Fuente | Año | Grado |
|---|---|---|
| Temporal, Restate, DBOS, Inngest, Hatchet, AWS Step Functions, Azure Durable Task — journal por paso, estado en el store | 2024-26 | `E2` |
| OpenTelemetry GenAI semantic conventions, v1.42.0 (jun-2026): `gen_ai.*` en repo dedicado, **pre-estable** | 2026 | `E2` |
| Práctica convergente de *Agent Decision Record* (hash encadenado, versión de modelo, política) | 2026 | `E3` |
| EU AI Act, obligaciones para sistemas de alto riesgo desde agosto de 2026 | 2024-26 | `E2` |
| Práctica de worktree por agente y leases con TTL ~5 min + heartbeat | 2026 | `E3` |
| Grit (Scott Chacon): reescritura de Git en Rust con agentes, ~45 mil millones de tokens | 2026 | `E5` |

## Arquitectura y deriva

| Fuente | Año | Grado |
|---|---|---|
| ArchUnit y la práctica de fitness functions como tests | 2016-2026 | `E3` |
| Ford, Parsons, Kua — *fitness functions* como concepto de arquitectura evolutiva | 2017 | `E3` |
| Práctica 2026 sobre detección de deriva arquitectónica con agentes | 2026 | `E3` |
| Cifras de reducción de tokens por grafo de código (97 %, 58-70 %, 10×, 121×) | 2026 | `E6` |
| RepoGraph, LocAgent — grafos de repositorio para agentes | 2025-26 | `E1` |

---

## Nota sobre trazabilidad de estas fuentes

`HECHO` — Todas las fuentes de esta lista se consultaron entre el 2026-08-31 y el 2026-09-01 por
búsqueda web y recuperación directa. De las marcadas `E1` y `E4` con identificador arXiv se
recuperó el texto completo en los casos señalados en negrita; del resto se trabajó sobre resúmenes
y sobre la descripción de resultados en fuentes secundarias.

`INFERENCIA` — Esa asimetría es una limitación real: las conclusiones que dependen de un número
concreto son más sólidas para las fuentes en negrita (ConstraintRot, PDD, MAST, HORIZON, *Inside the
Scaffold*, *Governance Gaps*) que para las demás. Cuando una recomendación de esta investigación
descansa sobre un número, ese número procede de esas seis siempre que fue posible.
