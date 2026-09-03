# 06 — Gobierno humano, autoridad, permisos y seguridad

Cubre las áreas 11 y 12 del encargo, más el estado de los protocolos de agentes.

## 6.1 La frontera entre agente autónomo y humano decisor

`INFERENCIA` — La literatura no ofrece una frontera universal, pero sí ofrece los criterios para
trazarla. De la evidencia de los documentos 4 y 5 se deducen tres pruebas, y una operación cruza al
lado humano si falla cualquiera:

1. **¿Es reversible barato?** Si deshacerlo cuesta más que hacerlo, no es autónoma.
2. **¿Tiene oráculo mecánico?** Si el éxito no se decide con un comando, alguien tiene que juzgar.
3. **¿Es constitutiva de autoridad?** Aprobar, firmar, promover una norma, resolver una
   contradicción, conceder permisos, ampliar el propio alcance. Estas operaciones **definen quién
   puede hacer qué**; automatizarlas es automatizar la propia frontera.

`INFERENCIA` — El punto 3 tiene un respaldo experimental que no es obvio: el resultado de
*Governance Decay* (§4.2) muestra que un modelo **no puede distinguir una actualización genuina del
operador de una falsificación en contexto** cuando ambas llegan como tokens. Por tanto, ningún acto
de autoridad puede validarse dentro del stream conversacional. Tiene que validarse fuera: en una
identidad, una firma, una fila de una base con segregación de funciones.

`INFERENCIA` — La lista de lo que nunca debería depender únicamente de un agente, entonces, no es
una preferencia de gobernanza; es una consecuencia del modo de fallo medido.

## 6.2 El fallo del human-in-the-loop: fatiga de aprobación

Es el error más caro que puede cometer una plataforma de gobierno, porque produce la apariencia de
control sin el control.

`HECHO` (`E1`, 2023) — Estudio en *Radiology* con 27 radiólogos y 50 mamografías: cuando la IA se
equivocaba, la precisión de los radiólogos inexpertos cayó de ~80 % a **menos del 20 %**, y la de
los experimentados (15+ años de práctica) de 82 % a **45,5 %**. La experiencia no protege del sesgo
de automatización; sólo lo amortigua.

`HECHO` (`E1`, 2025) — Revisión sistemática de 35 estudios revisados por pares en salud, finanzas,
seguridad nacional y administración pública sobre el acuerdo con recomendaciones de IA incorrectas.

`HECHO` (`E3`, 2026) — La formulación operativa que aparece repetida: la aprobación número 200 del
día no recibe la misma calidad cognitiva que la primera. Cuando las peticiones de aprobación llegan
más rápido de lo que un humano puede leerlas, la supervisión colapsa en sello de goma.

`HECHO` (`E4`, abr-2026) — El survey de Bhati llama a esto la **economía de la atención** y lo lista
como uno de los cinco problemas abiertos: si un agente produce diez parches plausibles por hora, la
revisión humana es el cuello de botella.

`HECHO` (`E3`, 2026) — La mitigación consensuada: evals automáticos, comprobación de tipos, linters
de política y simulaciones **deben filtrar lo mecánicamente incorrecto antes de que un humano lo
vea**, de modo que la cola humana contenga sólo decisiones que requieren juicio.

> `INFERENCIA` — Esto invierte la relación habitual entre verificación automática y aprobación
> humana. La verificación mecánica no está ahí para *sustituir* al humano: está ahí para
> **proteger su atención**, que es el recurso escaso y el que se degrada. Una plataforma que añada
> puertas humanas sin añadir filtros mecánicos delante está empeorando el gobierno mientras cree
> reforzarlo.
>
> `INFERENCIA` — Y de ahí sale una métrica que una plataforma de gobierno debería registrar y casi
> nadie registra: **cuántas paradas humanas por unidad de trabajo cerrada, y qué fracción de ellas
> cambió el resultado**. Un gate humano que aprueba el 100 % de lo que ve no está gobernando: está
> registrando.

## 6.3 Permisos, políticas y sandboxing

`HECHO` (`E2`, 2026) — El patrón de autorización que ha convergido en producción separa tres cosas:
un proveedor de identidad dedicado (autenticación), un **motor de políticas dedicado**
(autorización), y una capa de administración de políticas en tiempo real. Las dos implementaciones
de referencia son OPA/Rego (CNCF graduated) y **Cedar** (default-deny, `forbid` gana a `permit`,
evaluación independiente del orden, sin efectos secundarios).

`HECHO` (`E2`, mar-2026) — AWS embarcó Cedar como motor de políticas dentro de Amazon Bedrock
AgentCore Policy: intercepta **cada llamada agente→herramienta** en el límite del gateway y autoriza
contra un policy set.

`HECHO` (`E3`, 2026) — El principio arquitectónico que se repite: la política se aplica en la **capa
de llamada a herramientas, no en la capa del agente**. Decide el motor, no el agente. Y en
delegación, el patrón preferido es **intersección de permisos** —el agente nunca puede tener más
permisos que quien delega— frente a la suplantación de identidad.

`HECHO` (`E2`/`E3`, 2026) — Aislamiento de ejecución: tres patrones con perfiles distintos —microVM
(Firecracker, Kata) para aislamiento fuerte con kernel propio; gVisor para interceptación de
syscalls sin VM completa; contenedores endurecidos sólo para código confiable. En producto: Claude
Code usa un bash sandboxeado (Seatbelt/bubblewrap más proxy de red) y microVM en sus sesiones cloud;
Codex expone gates de aprobación, RBAC, políticas y sandbox a nivel de sistema operativo.

`HECHO` (`E1`, 2025-2026) — Defensa contra prompt injection con garantías estructurales, no
heurísticas:

- *Design Patterns for Securing LLM Agents against Prompt Injections* (arXiv:2506.08837, jun-2025):
  seis patrones —Action-Selector, Plan-Then-Execute, LLM Map-Reduce, Dual LLM, Code-Then-Execute,
  Context-Minimization— con el principio de fondo: **una vez que un agente ingiere entrada no
  confiable, debe quedar restringido a acciones no consecuentes**.
- **CaMeL** (arXiv:2503.18813, Google DeepMind, 2025): un LLM privilegiado genera el plan a partir
  de la consulta confiable; un LLM en cuarentena procesa los datos no confiables **sin acceso a
  herramientas**; y un intérprete propio **rastrea la procedencia de los datos** y aplica políticas
  antes de cada llamada a herramienta. Resuelve en la práctica la evaluación de seguridad de
  AgentDojo.

> `INFERENCIA` — CaMeL es la confirmación más limpia del principio que atraviesa toda esta
> investigación: **la seguridad y el gobierno se consiguen rastreando la procedencia y decidiendo
> fuera del modelo**, no pidiéndole al modelo que se porte bien. Es la misma idea que la evidencia
> observada (§5.5) aplicada al flujo de datos en vez de al flujo de trabajo.
>
> `INFERENCIA` — Para una plataforma de gobierno hay una consecuencia concreta: el perímetro no es
> una tarea de «endurecer para producción» que se pueda posponer. Es **la premisa de la que depende
> el valor de todo lo demás**. Un sistema de firmas inmutables con segregación de funciones cuyo
> endpoint de sesión emite credenciales sin autenticar no tiene un problema de seguridad: tiene un
> problema de veracidad. Toda la cadena de autoridad que registra es refutable con un `curl`.

## 6.4 Protocolos: qué existe y qué falta

`HECHO` (`E2`, 2025-2026) — MCP: especificaciones de 2025-11-25 y 2026-07-28; más de 110 millones de
descargas mensuales. El roadmap 2026 apunta a transporte HTTP sin estado, descubrimiento de
servidores, tareas asíncronas, **audit trails**, OAuth 2.1 y Client ID Metadata Documents.

`HECHO` (`E2`, abr-2026) — A2A alcanzó la v1.0 con más de 150 organizaciones y **Agent Cards
firmados** para identidad verificable. Hay propuestas académicas para delegación verificable entre
MCP y A2A (AIP, arXiv:2603.24775) y modelado de amenazas comparado (arXiv:2602.11327).

`HECHO` (`E4`, jul-2026) — Kang & Diponegoro, *Governance Gaps in Agent Interoperability Protocols:
What MCP, A2A, and ACP Cannot Express* (arXiv:2606.31498). Los protocolos **no pueden expresar**:
autorización más allá del patrón petición/respuesta, rendición de cuentas sobre acciones y
decisiones, aplicación de políticas, delegación de gobierno y consentimiento multi-parte, ni audit
trails para cumplimiento. Su taxonomía de huecos: alcance, expresión de intención, mecanismos de
aplicación, coordinación multi-parte, y dinámica temporal (no hay políticas con plazo ni
revocación). Proponen una **capa de gobierno por encima de los protocolos**, desacoplada del
transporte.

`HECHO` (`E5`, 2026) — Predicción de Gartner citada en el circuito de datos: para 2028, el 60 % de
los proyectos de analítica agéntica que dependan sólo de MCP fracasarán por ausencia de una capa de
contexto consistente. (Es una predicción de analista: `E5`, no evidencia.)

> `INFERENCIA` — Tres fuentes independientes —un survey académico que llama a L5 «la capa menos
> madura», un paper que enumera exactamente lo que los protocolos no pueden expresar, y un paper
> que propone la arquitectura de invariantes y evidencia sin implementarla— coinciden en describir
> el mismo hueco. Ese hueco es, literalmente, el objeto de Quipu.
>
> `INFERENCIA` — Y de ahí se deduce la posición defendible: **MCP es transporte**. Que Quipu hable
> MCP no es su ventaja; es su requisito de entrada. La ventaja es lo que MCP explícitamente no puede
> expresar y Quipu impone en Postgres.

## 6.5 Identidad de agentes y trazabilidad de la autoridad

`HECHO` (`E3`, 2026) — La práctica emergente asigna a cada agente una **identidad no humana**
propia, audita sus descargas de paquetes y sus consultas MCP, y exige que la delegación sea
verificable en lugar de suplantada.

`INFERENCIA` — Cruzando esto con §5.5 y con el modo de fallo de la suplantación de operador (§4.2),
la propiedad que una plataforma debe garantizar se puede enunciar con precisión: **para cada acto de
autoridad debe ser posible demostrar, sin recurrir al contenido de ningún prompt, quién lo ejecutó,
bajo qué delegación, contra qué contenido exacto y en qué momento.** Una firma contra la huella del
contenido firmado cumple esto; una aprobación registrada como texto no.

`INTERPRETACIÓN` — El diseño de Quipu en que las abilities de agente **excluyen firmar y aprobar**,
y en que existe un test que vigila que no exista ninguna herramienta de aprobación, es exactamente
esta propiedad implementada. La evidencia externa dice que es correcta y rara. El hallazgo H-16 —que
sólo 4 de 128 rutas consultan esa distinción— dice que está implementada en el motor y no en el
perímetro, que es donde se comprueba.
