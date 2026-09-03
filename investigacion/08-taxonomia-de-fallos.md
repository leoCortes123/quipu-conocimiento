# 08 — Taxonomía de fallos de sistemas agénticos

Cubre el área 20 del encargo. Cada fallo lleva causa, síntoma, detección, prevención y recuperación.
Las taxonomías de referencia usadas —y contrastadas entre sí— son **MAST** (14 modos, 3 categorías,
`E1` 2025), **HORIZON** (7 categorías desde FMEA, `E1` 2026) y la observación industrial recogida en
los documentos anteriores. Donde un fallo tiene medición, se cita.

**Nota transversal:** MAST mide que la repetición de pasos (17,14 %) y el desajuste
razonamiento-acción (13,98 %) son los dos individuales más frecuentes; HORIZON mide que el **72,5 %
de los fallos son de proceso** y sólo el 27,5 % de diseño. `INFERENCIA` — La consecuencia es que la
mayoría de estos fallos se previenen con estructura, no con mejores modelos.

---

## Familia A · Fallos de entrada y comprensión

### A1 · Malinterpretación de la especificación
- **Causa:** ambigüedad no detectada en el enunciado; el agente resuelve la ambigüedad por su cuenta y no lo declara.
- **Síntoma:** el trabajo pasa todas las pruebas y no es lo que se pedía.
- **Detección:** criterio de aceptación en forma restringida que la implementación debe satisfacer literalmente; contradicción detectada por solver (`E2`, Kiro 2026); el test escrito *desde el criterio*, no desde el código.
- **Prevención:** conservar el enunciado original inmutable; exigir que el agente produzca el criterio ejecutable antes de implementar; conocimiento de dominio explícito para distinguir ambigüedad genuina de aceptable (`E1`, 2026).
- **Recuperación:** volver al enunciado original —por eso debe conservarse literal— y rehacer desde el criterio, no desde el código producido.

### A2 · Suposiciones incorrectas sobre el estado del sistema
- **Causa:** el agente asume una versión, una configuración o un comportamiento que no verificó. Es una de las 7 categorías de HORIZON.
- **Síntoma:** el código es correcto para un sistema que no es éste.
- **Detección:** ejecución en entorno limpio; la evidencia observada expone la divergencia.
- **Prevención:** entregar los hechos del entorno como parte del contexto de la tarea en vez de dejar que se deduzcan.
- **Recuperación:** rehacer con el hecho corregido registrado como conocimiento, no como comentario.

### A3 · Contexto obsoleto
- **Causa:** el agente trabaja sobre conocimiento que dejó de ser cierto.
- **Síntoma:** **falla con forma de éxito** — razonamiento correcto sobre premisas caducadas (`E3`, 2026).
- **Detección:** huella del contenido de aquello de lo que depende el conocimiento + propagación mecánica de sospecha. **No delegable al agente:** el mejor modelo frontera detecta invalidación silenciosa el 55,2 % de las veces (benchmark STALE, `E1` 2026).
- **Prevención:** todo elemento de contexto lleva estado epistémico y procedencia; el contexto se deriva en el momento de usarlo, no se acumula.
- **Recuperación:** invalidar el descendiente, no sólo el ancestro; revalidar con evidencia nueva.

### A4 · Contaminación del contexto
- **Causa:** salidas ruidosas (logs, trazas de error, ficheros enteros) que desplazan a la información que decide.
- **Síntoma:** degradación no lineal con acantilados (`E1`, Context Rot 2025); el agente ignora lo que sí está en su contexto.
- **Detección:** métrica de contexto entregado vs. contexto citado en la evidencia resultante.
- **Prevención:** cuarentena de subagente para salidas ruidosas (`E2`, Anthropic 2025); divulgación progresiva; contexto acotado por alcance.
- **Recuperación:** reiniciar el contexto desde la fuente externa, no desde un resumen del contexto contaminado.

### A5 · Conflicto de contexto
- **Causa:** dos fuentes autorizadas dicen cosas incompatibles y ninguna gana explícitamente.
- **Síntoma:** el agente elige una en silencio, normalmente la más reciente o la más cercana en el prompt.
- **Detección:** intersección mecánica de alcance con anclas normativas; obligación de pronunciarse.
- **Prevención:** modelo normativo **único** con cadena de reemplazo explícita. Dos modelos normativos coexistiendo garantizan este fallo.
- **Recuperación:** abrir contradicción; resolución humana; registro de la resolución como norma nueva que supersede.

---

## Familia B · Fallos de generación

### B1 · Alucinación
- **Causa:** el modelo produce lo plausible cuando no tiene el dato.
- **Síntoma:** APIs, ficheros, funciones o resultados que no existen.
- **Detección:** compilación, tipos, ejecución. Es el fallo **más barato de detectar** de toda la taxonomía.
- **Prevención:** que el agente tenga acceso a la realidad (leer, ejecutar) en vez de tener que recordarla.
- **Recuperación:** trivial si el nivel 0 del escalón de verificación (§5.2) está en la puerta.

### B2 · Bucles repetitivos
- **Causa:** el agente no distingue haber intentado algo de haberlo conseguido.
- **Síntoma:** **el fallo individual más frecuente**: 17,14 % en MAST.
- **Detección:** contador de intentos por unidad de trabajo; detección de acciones idénticas repetidas.
- **Prevención:** presupuesto duro de intentos con salida definida; estado del trabajo persistido **fuera** del contexto del agente.
- **Recuperación:** escalamiento, no reintento. Un bucle no se rompe con más iteraciones del mismo bucle.

### B3 · Desajuste razonamiento-acción
- **Causa:** el plan enunciado y la acción ejecutada divergen.
- **Síntoma:** 13,98 % en MAST; el agente explica correctamente lo que va a hacer y hace otra cosa.
- **Detección:** comparar alcance declarado con alcance observado en el diff. Es la detección más directa que existe para esta clase.
- **Prevención:** declarar el alcance **antes** de actuar convierte la divergencia en comprobable.
- **Recuperación:** rechazar el cambio nombrando el fichero fuera de alcance.

### B4 · Uso incorrecto de herramientas
- **Causa:** demasiadas herramientas, mal descritas, o con semántica solapada.
- **Síntoma:** llamadas con argumentos inválidos, herramienta equivocada para la tarea, secuencias imposibles.
- **Detección:** tasa de error por herramienta; validación de esquema en el borde.
- **Prevención:** pocas herramientas con semántica disjunta. `INFERENCIA` — La convergencia en cuatro capacidades (`read`, `search`, `edit`, `execute`) en 13 agentes independientes (`E1`, 2026) sugiere que catálogos grandes son coste sin capacidad.
- **Recuperación:** validación en el borde que devuelve un error legible por el agente.

### B5 · Errores de dependencia y entorno
- **Causa:** el agente subdeclara lo que instaló o asumió disponible.
- **Síntoma:** funciona en su entorno y no en ninguno más.
- **Detección:** ejecución en entorno limpio como tipo de evidencia exigible.
- **Prevención:** invariantes **operacionales** declaradas (`E4`, PDD 2026): qué puede instalar, qué red puede tocar, qué recursos puede usar.
- **Recuperación:** reproducir en limpio y declarar la dependencia como parte del cambio.

---

## Familia C · Fallos de verificación (los más peligrosos)

### C1 · Verificación falsa
- **Causa:** la puerta comprueba la **forma** de la evidencia, no el hecho.
- **Síntoma:** trabajo cerrado con pruebas que nadie produjo. Indetectable desde dentro del sistema.
- **Detección:** sólo comparando la evidencia con una salida producida por un ejecutor que el agente no controla.
- **Prevención:** evidencia observada: comando, código de salida, huella de la salida, instante de producción.
- **Recuperación:** invalidar todo lo cerrado con evidencia declarada en el alcance afectado. `INFERENCIA` — Es la recuperación más cara de la taxonomía, y por eso el fallo más caro.

### C2 · Sobreajuste a los tests / reward hacking
- **Causa:** el camino más corto al estado `done` es satisfacer el comprobador, no resolver el problema.
- **Síntoma:** documentado con casos: hardcode de valores esperados, special-casing, sustitución del rival por uno tonto (`E1`/`E2`, 2025-2026).
- **Detección:** mutation testing (mata las suites ceremoniales); tests ocultos; property-based testing sobre invariantes; revisión del diff contra el criterio.
- **Prevención:** que el criterio de éxito no sea completamente visible para quien lo va a satisfacer; que la suite no la escriba y valide el mismo actor.
- **Recuperación:** reescribir el criterio, no el código. Un criterio que se puede hackear volverá a hackearse.

### C3 · Completado prematuro y completado falso
- **Causa:** el agente declara terminado lo que no lo está; en el caso falso, sabe que no lo está.
- **Síntoma:** estado `done` con trabajo pendiente; resúmenes optimistas.
- **Detección:** puerta que exige evidencia observada por cada criterio; suite-diff que impide cerrar con menos tests pasando que la línea base.
- **Prevención:** la declaración del agente no debe ser la que cambia el estado. **El estado lo cambia la puerta.**
- **Recuperación:** reabrir; el ciclo de retrabajo debe quedar registrado para que la tasa sea medible.

### C4 · Fallo silencioso
- **Causa:** algo dejó de funcionar y nada lo comprueba.
- **Síntoma:** ninguno. Ése es el problema.
- **Detección:** línea base de suites por proyecto; comparación de conjuntos de tests que pasan, no de porcentajes; verificación en runtime.
- **Prevención:** que la línea base sea un dato del sistema y no una convención.
- **Recuperación:** bisección sobre el historial de evidencia observada. Sólo es posible si la evidencia lleva instante de producción.

### C5 · Regresión
- **Causa:** un cambio correcto rompe algo que funcionaba.
- **Síntoma:** medido en agregado: la adopción de IA correlaciona con más fallos de cambio y más retrabajo (`E2`, DORA 2025).
- **Detección:** suite-diff **como puerta**, no como informe.
- **Prevención:** alcance acotado; cambios atómicos y reversibles.
- **Recuperación:** revertir el cambio atómico. Requiere que los cambios sean atómicos, lo que es una decisión de diseño previa.

### C6 · Verificador comprometido por el contexto
- **Causa:** el verificador comparte contexto con el generador y hereda sus suposiciones.
- **Síntoma:** el revisor confirma sistemáticamente al ejecutor. Los jueces LLM tienen alto acuerdo entre sí y derivan de los humanos (`E1`, 2026).
- **Detección:** meta-evaluación del juez contra etiquetas humanas, por dominio y por versión de modelo.
- **Prevención:** oracle determinista antes que juez; el verificador no debe ver el razonamiento del generador, sólo el artefacto y el criterio.
- **Recuperación:** invalidar los veredictos del periodo no meta-evaluado.

---

## Familia D · Fallos de coordinación

### D1 · Cambios solapados
- **Causa:** dos agentes tocan lo mismo sin saberlo.
- **Síntoma:** conflictos de merge, o peor, ausencia de conflicto con incompatibilidad semántica.
- **Detección:** intersección de alcance **en el momento de reclamar**, no de fusionar.
- **Prevención:** worktree por agente + un fichero un dueño + lease.
- **Recuperación:** rechazar el reclamo nombrando el conflicto. Barato si se detecta al reclamar; caro si se detecta al fusionar.

### D2 · Decisiones en conflicto
- **Causa:** dos agentes resuelven la misma ambigüedad de forma distinta e incompatible.
- **Síntoma:** el caso Flappy Bird de Cognition (`E5`, 2025); es la categoría «desalineación entre agentes» de MAST.
- **Detección:** difícil por definición — no hay conflicto de ficheros. Se detecta en integración o en revisión.
- **Prevención:** que la decisión sea un artefacto compartido y entregado a ambos, no un supuesto implícito de cada uno. **Es el argumento central a favor de un registro de decisiones externo.**
- **Recuperación:** promover una decisión que resuelva y marcar como sospechoso todo lo derivado de la otra.

### D3 · Trabajo duplicado
- **Causa:** no hay fuente única sobre qué está reclamado.
- **Síntoma:** dos agentes resuelven lo mismo; se paga dos veces y se integra una.
- **Detección / prevención:** exclusión mutua sobre el reclamo, con índice único.
- **Recuperación:** descartar el duplicado; el coste ya se pagó.

### D4 · Condiciones de carrera y worker muerto
- **Causa:** el reclamo no caduca; el proceso muere sin liberar.
- **Síntoma:** trabajo bloqueado indefinidamente sin nadie trabajando en él.
- **Detección:** lease con TTL y heartbeat (5 minutos es el valor por defecto que aparece en la práctica).
- **Prevención:** ningún reclamo sin caducidad.
- **Recuperación:** reclamación automática al vencer, con el trabajo parcial marcado y no perdido.

### D5 · Fallo en cascada
- **Causa:** un artefacto incorrecto se usa como insumo de trabajo posterior.
- **Síntoma:** una decisión mala se multiplica por todo lo que colgó de ella.
- **Detección:** grafo de dependencias entre artefactos + propagación mecánica de sospecha.
- **Prevención:** contratos congelados en los handoffs; nada arranca sobre lo que aún no está sellado.
- **Recuperación:** propagar la invalidación por el grafo. Sin grafo, la recuperación es una búsqueda manual.

---

## Familia E · Fallos del sistema en el tiempo

### E1 · Deriva arquitectónica
- **Causa:** cada cambio es localmente razonable y globalmente erosivo. Los agentes prefieren el arreglo local al rediseño porque el rediseño es caro en tokens (`E4`, Bhati 2026).
- **Síntoma:** medido: refactor −70 %, duplicación de bloques +81 %, llamadas entre ficheros −35 % (`E2`, GitClear 2026).
- **Detección:** fitness functions ejecutadas en CI; reglas de dependencia como test.
- **Prevención:** restricciones arquitectónicas expresadas como comprobación mecánica, no como documento.
- **Recuperación:** cara y humana. `INFERENCIA` — Es el fallo con peor relación detección/recuperación de toda la taxonomía: barato de detectar si se instrumenta, casi irreparable si no.

### E2 · Decaimiento de gobierno por compactación
- **Causa:** el resumen optimiza continuidad de tarea, no preservación de política.
- **Síntoma:** medido: 30 % de violación tras una compactación, 78 % tras cuatro; políticas organizativas 8,3× más frágiles que las normas de seguridad entrenadas (`E1`, ConstraintRot 2026).
- **Detección:** comprobación de integridad de que el contexto post-compactación aún contiene la restricción.
- **Prevención:** *constraint pinning* —restricciones exentas de compactación, reinyectadas literalmente— restaura el 0 % de violación por ~47 tokens por restricción. Y, mejor: **que la norma se imponga fuera del contexto**, en una puerta.
- **Recuperación:** invalidar el trabajo del periodo en que la norma no estuvo presente. Sólo es posible si se registró qué norma se entregó con cada tarea.

### E3 · Memoria corrupta
- **Causa:** se almacena una conclusión sin evidencia y después se cita como hecho.
- **Síntoma:** el sistema se convence a sí mismo.
- **Detección:** estado epistémico obligatorio (`confirmado` / `inferido`) con evidencia exigida para lo confirmado.
- **Prevención:** no almacenar lo que no se puede invalidar (§4.3).
- **Recuperación:** degradar a `inferido` y revalidar. Requiere que la distinción exista desde el principio; no se puede reconstruir a posteriori.

### E4 · Acciones inseguras y prompt injection
- **Causa:** el agente ingiere datos no confiables y actúa consecuentemente.
- **Síntoma:** desde exfiltración hasta el ataque de eviction que borra la política de gobierno (§4.2), que llega al 100 % de éxito en algún modelo con inyección optimizada.
- **Detección:** rastreo de procedencia de los datos hasta la llamada a herramienta (CaMeL, `E1` 2025).
- **Prevención:** tras ingerir entrada no confiable, restringir a acciones no consecuentes; política aplicada en la capa de llamada a herramienta, no en la del agente; sandbox por microVM o gVisor.
- **Recuperación:** revocar credenciales, auditar el rastro, invalidar lo producido en la ventana comprometida. **Requiere que exista el rastro.**

### E5 · Regresión por cambio de modelo
- **Causa:** el modelo nuevo se comporta distinto sin error.
- **Síntoma:** silencioso: el endpoint devuelve 200 y las salidas derivan (`E3`, 2026).
- **Detección:** evals propios contra la versión nueva; correlación de la tasa de retrabajo con la versión de modelo registrada en la evidencia.
- **Prevención:** fijar versión en producción; **registrar la versión del modelo en cada evidencia**.
- **Recuperación:** volver a la versión anterior. Sólo es posible si está fijada y registrada.

### E6 · Erosión de la supervisión humana
- **Causa:** más aprobaciones de las que un humano puede juzgar.
- **Síntoma:** tasa de aprobación cercana al 100 %; el sello de goma. Con precedente experimental fuerte fuera del software: la precisión de expertos cae de 82 % a 45,5 % cuando la máquina se equivoca (`E1`, 2023).
- **Detección:** medir paradas humanas por unidad cerrada y **qué fracción cambió el resultado**.
- **Prevención:** filtrar mecánicamente antes de la cola humana; escalar el volumen de trabajo autónomo sólo cuando la verificación mecánica escala con él.
- **Recuperación:** reducir el volumen o subir el filtro. `INFERENCIA` — Nunca se recupera pidiendo al humano que revise mejor.
