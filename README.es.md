# agent-harness-template

[English](README.md)

Templates, un plugin de Claude Code y lecciones breves para lograr que un asistente de programación con IA
siga las convenciones de tu repositorio, con checks que no dependen de que las recuerde.

El repositorio es principalmente Markdown. Casi no contiene código: donde un check necesita lógica, recibes
pseudocódigo para portar, y el código aparece solo donde nada funciona sin él (un hook, un paso de shell, un
gate de revisión).

## La idea: capas, desde la recomendación hasta el cumplimiento obligatorio

```text
  recomendación                                                                   cumplimiento obligatorio
  ------------------------------------------------------------------------------------------------------>
  AGENTS.md      .claude/rules/     skills         settings + hooks           checks de CI     gate de revisión   revisión
  CLAUDE.md      (por ruta)         subagents      (del lado del cliente)     (deterministas)  con IA             humana
                                                                                                (modelo + código)
  cada sesión    al tocar una       bajo demanda   cada llamada a             cada pull        cada pull          merge
                 ruta                              herramienta                request          request
```

- **Los archivos de instrucciones** le dicen al agente cómo funciona el repositorio. Normalmente los sigue;
  nada lo obliga.
- **Settings y hooks** los aplica Claude Code sin importar lo que decida el modelo.
- **CI** hace fallar el pull request para todos: personas, cualquier agente, cualquier herramienta.
- **La revisión con IA** detecta lo que ninguna regla establece, dentro de un sandbox, y solo bloquea con
  evidencia que un script verificó.
- **Una persona** es responsable del merge.

Regla práctica: si una máquina puede verificarlo, escribe el check; si siempre debe cumplirse, usa un hook o
un check obligatorio; solo lo que queda va en prosa.

## Inicio rápido A: deja que el plugin lo configure

```bash
claude plugin marketplace add <owner>/agent-harness-template
claude plugin install agent-harness@agent-harness-template
```

Luego, dentro de Claude Code en tu repositorio:

| Comando | Qué hace |
| --- | --- |
| `/agent-harness:init dry-run` | Lee tu repositorio e imprime el harness que crearía, con tus comandos reales. No escribe nada |
| `/agent-harness:init` | Pregunta lo que no puede leer, muestra cada archivo y escribe solo lo que apruebes |
| `/agent-harness:audit` | Revisa un harness existente: límites, rules que se cargan siempre, rutas obsoletas, actions sin fijar, contradicciones |
| `/agent-harness:add-rule <instruction>` | Decide dónde corresponde una instrucción nueva (check, hook, skill, rule, AGENTS.md o un documento) y propone el cambio |

## Inicio rápido B: copia a mano

1. Copia [AGENTS.md](plugins/agent-harness/templates/AGENTS.md) y
   [CLAUDE.md](plugins/agent-harness/templates/CLAUDE.md) a la raíz de tu repositorio y reemplaza cada
   `<...>`. Borra las líneas que no apliquen en lugar de dejar placeholders.
2. Agrega una rule por cada parte del árbol con convenciones propias, a partir de
   [rules/](plugins/agent-harness/templates/.claude/rules/testing.md), cada una con un `paths:` que coincida
   con tus archivos.
3. Copia [settings.json](plugins/agent-harness/templates/.claude/settings.json) a `.claude/` y completa la
   allow list con tus comandos.
4. Copia los workflows que quieras desde
   [.github/workflows/](plugins/agent-harness/templates/.github/workflows/ci.yml), la
   [plantilla de PR](plugins/agent-harness/templates/.github/pull_request_template.md) y
   [CODEOWNERS](plugins/agent-harness/templates/.github/CODEOWNERS).
5. Agrega `CLAUDE.local.md` y `.claude/settings.local.json` al `.gitignore`. Si tu gitignore global excluye
   `.claude/`, agrega `!.claude/` al `.gitignore` del repositorio.
6. Abre Claude Code y ejecuta `/memory` para confirmar que `CLAUDE.md` y `AGENTS.md` se cargaron.

## Qué hay en los templates

| Template | Qué hace | Cuándo se carga o se ejecuta |
| --- | --- | --- |
| [AGENTS.md](plugins/agent-harness/templates/AGENTS.md) | Comandos, stack, arquitectura, Critical Invariants, límites. Canónico para todos los agentes | Cada sesión |
| [CLAUDE.md](plugins/agent-harness/templates/CLAUDE.md) | Importa `AGENTS.md`, más notas solo para Claude | Cada sesión de Claude Code |
| [CLAUDE.local.md.example](plugins/agent-harness/templates/CLAUDE.local.md.example) | Notas personales, nunca se commitean | Cada sesión, solo para ti |
| [.claude/rules/](plugins/agent-harness/templates/.claude/rules/layer.md) | Convenciones para una parte del árbol | Cuando se lee o edita un archivo que coincide |
| [.claude/settings.json](plugins/agent-harness/templates/.claude/settings.json) | Permisos y un hook que bloquea commits y pushes a `main` | Cada llamada a una herramienta |
| [skill ship-pr](plugins/agent-harness/templates/.claude/skills/ship-pr/SKILL.md) | Gates locales, título Conventional Commit, pull request en draft, nunca hace merge | Cuando ejecutas `/ship-pr` |
| [subagent reviewer](plugins/agent-harness/templates/.claude/agents/reviewer.md) | Revisión de solo lectura de un diff contra las convenciones | Cuando se le delega |
| [.mcp.json.example](plugins/agent-harness/templates/.mcp.json.example) | Servidores MCP con credenciales tomadas del entorno | Cada sesión |
| [agent-instructions.md](plugins/agent-harness/templates/docs/conventions/agent-instructions.md) | Qué va en cada lugar, y cuándo se justifica un cambio en las instrucciones | Cuando se abre |
| [comments.md](plugins/agent-harness/templates/docs/conventions/comments.md) | El POR QUÉ y no el QUÉ, sin historia en el código | Cuando se abre |
| [user-level/](plugins/agent-harness/templates/user-level/CLAUDE.md) | Tu capa personal de `~/.claude/` | Cada sesión, cada proyecto |
| [ci.yml](plugins/agent-harness/templates/.github/workflows/ci.yml) | Un job de gate: lint, format, tipos, tests, build | Cada pull request |
| [pr-title.yml](plugins/agent-harness/templates/.github/workflows/pr-title.yml) | Título Conventional Commit | Cada pull request |
| [secret-scan.yml](plugins/agent-harness/templates/.github/workflows/secret-scan.yml) | gitleaks sobre los commits del pull request | Cada pull request |
| [instruction-files.yml](plugins/agent-harness/templates/.github/workflows/instruction-files.yml) | Límites de los archivos de instrucciones y `paths:` | Cada pull request |
| [ai-review.yml](plugins/agent-harness/templates/.github/workflows/ai-review.yml) | Revisión del modelo en sandbox más un gate que verifica las citas, modos off, shadow, advise | Cada pull request listo |
| [pull_request_template.md](plugins/agent-harness/templates/.github/pull_request_template.md) | Resumen, plan de pruebas, checklist del autor, declaración de uso de IA, evidencia | Cada pull request |
| [CODEOWNERS](plugins/agent-harness/templates/.github/CODEOWNERS) | Revisión del owner solo para los archivos del harness | Cada pull request |
| [dependabot.yml](plugins/agent-harness/templates/.github/dependabot.yml) | Actualizaciones semanales agrupadas, incluidas las actions | Semanal |
| [claims.json](plugins/agent-harness/templates/.claude/claims.json) | Líneas de las instrucciones ligadas a hechos que un check verifica | Cuando ejecutas el check de claims |
| [checks.md](plugins/agent-harness/reference/checks.md) | Pseudocódigo de cada check del harness | Cuando portas uno, o ejecutas `audit` |

## Lecciones

1. [La jerarquía de memoria: CLAUDE.md, AGENTS.md y compañía](docs/es/learn/01-memory-hierarchy.md)
2. [Rules por ruta](docs/es/learn/02-path-scoped-rules.md)
3. [Skills, comandos, subagents y plugins](docs/es/learn/03-skills-commands-plugins.md)
4. [Settings, permisos y hooks](docs/es/learn/04-settings-permissions-hooks.md)
5. [Servidores MCP](docs/es/learn/05-mcp.md)
6. [La capa determinista de CI](docs/es/learn/06-deterministic-ci.md)
7. [La capa de revisión con LLM](docs/es/learn/07-llm-review-layer.md)
8. [Flujo de pull requests y release](docs/es/learn/08-pr-flow-and-release.md)
9. [Mantener las instrucciones](docs/es/learn/09-maintaining-instructions.md)

Y [Lecciones aprendidas](docs/es/lessons-learned.md): qué salió mal en repositorios reales, y qué hacer.

## Estructura del repositorio

```text
.claude-plugin/marketplace.json        este repositorio es un marketplace de Claude Code
plugins/agent-harness/
  .claude-plugin/plugin.json
  skills/init|audit|add-rule/SKILL.md  las tres skills
  templates/                           cada archivo que copian las skills, también usable a mano
  reference/checks.md                  pseudocódigo de los checks del harness
docs/learn/                            las lecciones
docs/lessons-learned.md
docs/es/                               traducción al español
```

## Créditos y fuentes

- Documentación de Claude Code: https://code.claude.com/docs
- El formato `AGENTS.md`: https://agents.md
- Claude Certification Guide, un recurso de estudio independiente cuyas lecciones enlazan las guías de este
  repositorio:
  https://claudecertificationguide.com
- Hardening de seguridad de GitHub Actions:
  https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions

## Licencia

[MIT](LICENSE)
