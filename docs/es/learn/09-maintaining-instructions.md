# 09. Mantener las instrucciones

Los archivos de instrucciones se pudren como el código, pero nada falla cuando eso ocurre: el agente
simplemente sigue una línea desactualizada. Trátalos como código: revisados, probados, podados.

## Cuándo cambiarlos

Un cambio en `AGENTS.md`, `CLAUDE.md` o una rule necesita una razón, nombrada en el cuerpo del pull request:

1. Un agente se equivocó dos veces con la misma convención, o una revisión detectó algo que un agente
   debería haber sabido.
2. Una invariante nueva, o una excepción a una existente, se entregó junto con el código que la necesita.
3. Una afirmación del archivo es falsa hoy.
4. La misma corrección se escribió en una sesión por segunda vez. Un procedimiento pegado por tercera vez
   se convierte en un skill.
5. Una actualización de modelo volvió innecesario un workaround.

No son razones: un workflow de CI nuevo (actualiza la tabla del README en su lugar), ni una reescritura que
dice lo mismo.

El skill `add-rule` te guía por la escalera de
[agent-instructions.md](../../../plugins/agent-harness/templates/docs/conventions/agent-instructions.md):
check, hook, skill, rule con alcance por ruta, Critical Invariants, `AGENTS.md`, o un documento de
convenciones.

## Cómo escribir una línea

- Verificable: una ruta, un comando, un número. "Ejecuta el formateador antes de hacer push", no "mantén
  todo ordenado".
- Un solo `IMPORTANT` por archivo como máximo.
- Sin contradicciones entre archivos: cuando dos líneas discrepan, el agente elige una y no dice cuál.
- Notas para los maintainers en comentarios HTML, que nunca llegan al modelo.

## Poda

Para cada línea: "¿Quitar esto haría que el agente cometa un error?". Si no, bórrala. Candidatas:

- workarounds para un modelo anterior;
- todo lo que el agente puede leer en el código;
- líneas que repiten un check de CI (nombra el check en su lugar);
- explicaciones largas (muévelas a `docs/conventions/` y deja una línea).

## La revisión mensual

Una vez al mes, y después de cada actualización de modelo:

1. Ejecuta `/agent-harness:audit`: límites, rules que se cargan en todas partes, rutas desactualizadas,
   comandos que faltan, contradicciones.
2. Abre `/memory` y `/context` en una sesión nueva y confirma qué se carga realmente.
3. Prueba una tarea pequeña que toque una rule con alcance por ruta y observa si el agente la sigue.
4. Borra lo que ya no cambia el comportamiento.

## Los claims mantienen las líneas honestas

Cuando una línea afirma un hecho que una máquina puede verificar ("el gate de CI se llama X", "esta rule
existe"), agrega un claim a `.claude/claims.json`. El check de claims falla cuando cambia el hecho o la
línea citada, de modo que el archivo se actualiza en el mismo pull request que lo volvió falso.

## Fuentes

- Mejores prácticas de Claude Code: https://code.claude.com/docs/en/best-practices
- Documentación de Claude Code, memoria: https://code.claude.com/docs/en/memory
- Certification guide 3.5, refinamiento iterativo:
  https://claudecertificationguide.com/learn/3-claude-code-config/3-5-iterative-refinement
