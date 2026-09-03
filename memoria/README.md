# memoria — el porqué de las decisiones

Banco de conocimiento del desarrollo: por qué se decidió lo que se decidió, para que la
razón sobreviva a la sesión que la produjo y no haya que reconstruirla leyendo commits.

La metodología completa de este banco —tres registros, protocolo de captura, caducidad y
arranque por retro-extracción— está propuesta en `../06-banco-de-conocimiento.md`, con
etiqueta `RECOMENDACIÓN`. Este directorio es su primera materialización, deliberadamente
mínima: un archivo por decisión.

## `decisiones/`

Una decisión por archivo, `DEC-<n>-<slug>.md`. Contenido mínimo:

```markdown
# DEC-<n> · <título>

- **Estado:** abierta | resuelta | superada por DEC-<m>
- **Fecha:** AAAA-MM-DD
- **Decide:** humano
- **Origen:** dónde se planteó (documento y sección)

## Pregunta
## Opciones consideradas
## Decisión
## Porqué
## Consecuencias
## Qué la invalidaría
```

`DEC-1` a `DEC-8` están planteadas en
`../analisisExterno/auditoria-interna-2026-09-01.md` § 14 y **ninguna está resuelta
todavía**. DEC-1 y DEC-2 bloquean la redacción del plan (`../plan/README.md`).

Una decisión no se edita para cambiar su sentido: se marca superada y se escribe la
nueva. Es la misma cadena `supersede` que Quipu impone en su BD — aquí es convención,
porque el método no corre sobre Quipu.
