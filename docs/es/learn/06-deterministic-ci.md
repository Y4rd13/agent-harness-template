# 06. La capa de CI determinista

Un agente puede leer una regla e igual romperla. En CI una regla deja de ser un consejo: un check que hace
fallar el pull request no depende de que alguien haya leído algo. La primera pregunta ante una convención
nueva es "¿puede una máquina verificar esto?". Si la respuesta es sí, escribe el check y deja que el archivo
de instrucciones lo nombre.

## Principios

- **Un solo job de gate, todas las fallas a la vez.** Ejecuta lint, formato, type check, tests y build como
  steps de un mismo job, cada uno con `if: ${{ !cancelled() }}`, de modo que una ejecución en rojo liste
  todas las fallas y el autor las corrija en un solo push. Un solo job también instala las dependencias una
  sola vez.
- **CI ejecuta los mismos comandos que las personas y los agentes.** Los comandos de `AGENTS.md` son los
  comandos de `ci.yml`. Si difieren, el check local del agente pasa y CI falla.
- **Un check que no puede fallar no es un check.** Un job con `continue-on-error: true`, o con un `if:` que
  lo omite en las rutas que importan, reporta verde pase lo que pase.
- **Nunca debilites un check que falla para obtener verde.** Corrige la causa, o plantéala a un maintainer.
  Déjalo escrito en `AGENTS.md` bajo Never, porque los agentes bajo presión lo intentarán.
- **El harness prueba al harness.** Cada script de verificación que escribas tiene su propio test, y los
  workflows se ejecutan cuando cambian `.github/` o `.claude/`.

## Los checks

| Check | Por qué | Se entrega como |
| --- | --- | --- |
| Lint, formato, type check, test, build | Lo básico, en un solo job | [ci.yml](../../../plugins/agent-harness/templates/.github/workflows/ci.yml) |
| Título en Conventional Commit | El título del squash decide el release | [pr-title.yml](../../../plugins/agent-harness/templates/.github/workflows/pr-title.yml) |
| Escaneo de secretos | Git conserva el historial; un secreto filtrado debe rotarse | [secret-scan.yml](../../../plugins/agent-harness/templates/.github/workflows/secret-scan.yml) |
| Límites de los archivos de instrucciones, `paths:` presente | Una rule sin `paths:` se carga en todas partes, en silencio | [instruction-files.yml](../../../plugins/agent-harness/templates/.github/workflows/instruction-files.yml) |
| Los globs de `paths:` coinciden con algún archivo | Un glob que no coincide con nada nunca se carga | [pseudocódigo](../../../plugins/agent-harness/reference/checks.md#paths-globs-match) |
| Las rutas entre backticks existen | Un puntero a un archivo borrado no lleva al agente a ninguna parte | [pseudocódigo](../../../plugins/agent-harness/reference/checks.md#backtick-paths-exist) |
| Los comandos de AGENTS.md existen | Un comando que no existe enseña al agente a fallar | [pseudocódigo](../../../plugins/agent-harness/reference/checks.md#commands-exist) |
| Claims | Une una línea de un archivo de instrucciones con un hecho, para que no quede desactualizada | [pseudocódigo](../../../plugins/agent-harness/reference/checks.md#claims), [claims.json](../../../plugins/agent-harness/templates/.claude/claims.json) |
| Actions fijadas por SHA de commit | Un tag se puede mover a otro código | [pseudocódigo](../../../plugins/agent-harness/reference/checks.md#pinned-actions) |
| Sin `${{ }}` con datos del autor en `run:` | Un nombre de branch o un título en `run:` es código que controla el autor | [pseudocódigo](../../../plugins/agent-harness/reference/checks.md#run-interpolation) |
| Convenciones de comentarios | El historial en los comentarios se pudre | [pseudocódigo](../../../plugins/agent-harness/reference/checks.md#comment-conventions) |

Los marcados como pseudocódigo quedan para que los portes a tu lenguaje, porque un verificador es código que
vas a mantener. El skill `audit` ejecuta todos leyendo el repositorio, así que puedes empezar por ahí.

## Claims

Un claim dice: "esta línea de `AGENTS.md` es verdadera porque este hecho se cumple en el repositorio".

```json
{ "id": "testing-rule-exists", "source": "AGENTS.md",
  "quote": "- `.claude/rules/testing.md` - test conventions; ...",
  "type": "file_exists", "args": { "path": ".claude/rules/testing.md" } }
```

El verificador falla cuando la línea citada desaparece (actualiza el claim) o cuando el hecho deja de ser
verdadero (actualiza la línea o el código). Lee el archivo de claims desde la base branch en CI, para que un
pull request no pueda borrar el claim que rompe.

## Higiene de workflows

- Empieza cada workflow con `permissions: {}` y otorga a cada job solo lo que necesita.
- Usa `actions/checkout` con `persist-credentials: false`, salvo que un step posterior haga push.
- Fija cada action de terceros por SHA de commit, con la versión en un comentario, y deja que Dependabot
  actualice ambos ([dependabot.yml](../../../plugins/agent-harness/templates/.github/dependabot.yml)).
- Pasa los valores no confiables mediante `env:`, nunca con `${{ }}` dentro de `run:`.
- `pull_request_target` se ejecuta con secretos y con el workflow de la base branch: nunca hagas checkout del
  código del pull request en él. El check del título lo usa de forma segura porque lee el título como dato.
- `concurrency` con `cancel-in-progress` ahorra minutos cuando los push se suceden uno tras otro.

## En local, antes de CI

Las mismas reglas se ejecutan antes cuando es posible: un hook de pre-commit para secretos y formato, un hook
`commit-msg` para el formato del commit, y los gates locales del propio agente (el skill `ship-pr` los
ejecuta antes de abrir un pull request). CI sigue siendo el gate, porque los hooks locales se pueden omitir.

## Fuentes

- Endurecimiento de seguridad de GitHub Actions:
  https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions
- Certification guide 3.6, integración con CI/CD:
  https://claudecertificationguide.com/learn/3-claude-code-config/3-6-cicd-integration
