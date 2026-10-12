# 08. Flujo de pull request y release

El harness se encuentra con el equipo en el pull request. Esta lección recorre un cambio desde una branch
hasta un release, y muestra dónde actúa cada capa.

## El flujo

```text
1. branch        feat/short-name from main; the hook refuses commits on main
2. implement     agent follows AGENTS.md and the path-scoped rules it loads
3. local gates   lint, format, type check, tests (the same commands CI runs)
4. self-review   the author reads the full diff; a reviewer subagent can help
5. open          /ship-pr: Conventional Commit title, template body, draft
6. CI            deterministic checks; AI review in shadow or advise mode
7. human review  CODEOWNERS for harness files; findings verified, not obeyed
8. merge         squash, with an explicit maintainer OK
9. release       the squash title decides the version bump
```

## La plantilla de pull request

[pull_request_template.md](../../../plugins/agent-harness/templates/.github/pull_request_template.md) incluye:

- **una tabla de títulos** que asocia cada tipo de Conventional Commit con el release que provoca;
- **Summary** y **Test plan**;
- **Author checklist**, rotulada como autoatestación: "I have read the full diff and can explain every
  change". Ningún check la hace cumplir; está para que nadie pueda decir que no se le preguntó;
- **una línea sobre IA generativa**: una de dos oraciones simples, dejada como texto plano para que no sea
  una casilla que se marca sin leer;
- **Evidence** para cambios visibles al usuario o riesgosos: una captura de pantalla, una request y su
  response, la salida de los tests.

## Tamaño

Los pull requests grandes se leen por encima, tanto las personas como los modelos. Divide todo lo que supere
unas 1000 líneas que no sean de tests o que cruce tres o más capas, por ejemplo la capa de datos, luego el
servicio y luego la interfaz, en pull requests apilados. El template de revisión con IA hace cumplir el
límite de líneas negándose a revisar por encima de él.

## Propiedad del código

[CODEOWNERS](../../../plugins/agent-harness/templates/.github/CODEOWNERS) cubre solo el harness: `AGENTS.md`,
`CLAUDE.md`, `.claude/` y `.github/`. Un cambio en lo que lee cada agente, o en los checks que lo hacen
cumplir, necesita la revisión de un owner. El código de la aplicación queda fuera, para que los pull
requests normales no esperen a los owners. Mantén un cambio de instrucciones en su propio pull request
cuando la feature no lo necesite.

## Disciplina de revisión

- Un hallazgo de revisión, humano o de IA, es una afirmación. Verifícalo contra el código antes de actuar;
  rechaza los que no se sostienen, con el motivo en el hilo.
- Verifica después de cada corrección, no al final. La mayoría de los hallazgos nuevos en un pull request
  son regresiones introducidas por sus propias correcciones anteriores; antes de hacer push, vuelve a leer
  los puntos de llamada y los elementos vecinos de todo lo que tocaste.
- Agrupa las correcciones relacionadas en un solo commit, y apunta a dos rondas de revisión como máximo.
- Resuelve los hilos a medida que se cierran los hallazgos, para que lo que sigue abierto se vea de un
  vistazo.

## Protección de branches

Haz obligatorios los checks en `main` con un ruleset o una protección de branch: el gate de CI, el check del
título, el escaneo de secretos y, más adelante, `ai-review/gate`. Exige un pull request y, para las rutas
del harness, la revisión de un code owner. Dos pull requests que pasan por separado pueden romper `main`
juntos (dos heads de migración, un conflicto de lockfile): exige que las branches estén actualizadas antes
del merge, o usa una merge queue.

## Release a partir del título

Con los squash merges, el título del pull request se convierte en el commit de `main`. Una herramienta de
release lee el tipo del commit:

| Título | Release |
| --- | --- |
| `fix(auth): handle an expired token` | patch |
| `feat(search): add filters` | minor |
| `feat!: drop the v1 API` | major |
| `docs: ...`, `chore: ...`, `ci: ...` | ninguno |

semantic-release y release-please funcionan de esta manera. Por eso el título se verifica antes del merge, y
por eso un agente nunca debe elegir un tipo a la ligera.

Una trampa: GitHub omite los workflows de push cuando el mensaje del commit contiene el marcador para saltar
CI (las palabras "skip ci" entre corchetes), en el título o en el cuerpo. Un cuerpo de squash que lo cita,
por ejemplo en un pull request sobre el proceso de release, desactiva en silencio CI, release y deploy para
ese merge.

## Fuentes

- Conventional Commits: https://www.conventionalcommits.org
- GitHub, acerca de los code owners:
  https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners
- GitHub, omitir ejecuciones de workflows:
  https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/skipping-workflow-runs
