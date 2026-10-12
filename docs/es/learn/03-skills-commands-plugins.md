# 03. Skills, comandos, subagents y plugins

Los archivos de instrucciones describen cómo funciona el repositorio. Las skills describen cómo hacer un
trabajo: abrir un pull request, revisar uno, configurar un harness. Una skill casi no cuesta nada hasta que
se usa, así que los procedimientos van ahí y no en `CLAUDE.md`.

## Skills

Una skill es un directorio con un `SKILL.md`:

```text
.claude/skills/ship-pr/SKILL.md      ->  /ship-pr          (project, shared through git)
~/.claude/skills/ship-pr/SKILL.md    ->  /ship-pr          (personal)
<plugin>/skills/init/SKILL.md        ->  /<plugin>:init    (from a plugin)
```

Solo la `description` está siempre en el contexto, para que Claude sepa que la skill existe. El cuerpo se
carga cuando la skill se ejecuta, ya sea porque escribiste `/ship-pr` o porque Claude decidió que la
descripción coincidía.

Campos de frontmatter que conviene conocer:

| Campo | Qué hace |
| --- | --- |
| `description` | Qué hace la skill y cuándo usarla. Pon primero el disparador; Claude se guía por él |
| `when_to_use` | Frases de disparo adicionales, que se agregan a la descripción |
| `disable-model-invocation: true` | Solo tú puedes ejecutarla, con `/name`. Úsalo para todo lo que escribe, hace push o gasta |
| `allowed-tools` | Herramientas que Claude puede usar **sin preguntar** mientras la skill se ejecuta. Aprueba por adelantado; no restringe |
| `argument-hint` | La pista que se muestra en el autocompletado, como `[dry-run]` |
| `context: fork` | Ejecuta la skill en un contexto de subagent separado, para que su salida no llene tu sesión |

Dentro de una skill, `${CLAUDE_SKILL_DIR}` es el directorio de la skill y, en un plugin,
`${CLAUDE_PLUGIN_ROOT}` es el del plugin. Úsalos para leer los archivos que vienen con la skill; las
skills de este plugin leen así sus plantillas.

Plantilla: [ship-pr/SKILL.md](../../../plugins/agent-harness/templates/.claude/skills/ship-pr/SKILL.md).

### Comandos

`.claude/commands/<name>.md` es la forma anterior de una skill y sigue funcionando: también crea
`/<name>`. Si una skill y un comando tienen el mismo nombre, gana la skill. Prefiere las skills para
trabajo nuevo: pueden incluir archivos. Un archivo `.md` suelto directamente dentro de `.claude/skills/`
no crea nada; debe ser `skills/<name>/SKILL.md`.

## Subagents

Un subagent es un archivo Markdown en `.claude/agents/` con sus propias instrucciones y su propia ventana
de contexto. La sesión principal le entrega una tarea y recibe un resumen, así que una búsqueda o una
revisión larga no satura el contexto principal. Su campo `tools` limita lo que puede usar: un revisor con
`Read, Grep, Glob` no puede editar el código que revisa.

Plantilla: [reviewer.md](../../../plugins/agent-harness/templates/.claude/agents/reviewer.md).

## Plugins

Un plugin empaqueta skills, subagents, hooks y servidores MCP para que un equipo los instale con un solo
comando y reciba las actualizaciones. Un marketplace es un repositorio con
`.claude-plugin/marketplace.json` que lista plugins. Este repositorio es ambos:

```text
.claude-plugin/marketplace.json            the catalog: one plugin, "agent-harness"
plugins/agent-harness/.claude-plugin/plugin.json
plugins/agent-harness/skills/{init,audit,add-rule}/SKILL.md
plugins/agent-harness/templates/           files the skills copy
plugins/agent-harness/reference/checks.md  pseudocode the audit skill runs
```

Instálalo:

```bash
claude plugin marketplace add <owner>/agent-harness-template
claude plugin install agent-harness@agent-harness-template
```

Para desarrollar uno sin instalarlo: `claude --plugin-dir ./plugins/agent-harness`, y valídalo con
`claude plugin validate <path>`. Mantén igual el nombre de la entrada en `marketplace.json` y el `name` en
`plugin.json`.

## Qué va dónde

| Quieres | Usa |
| --- | --- |
| Un hecho verdadero en todas las sesiones | `AGENTS.md` |
| Un procedimiento que se ejecuta a pedido | una skill |
| Lectura o revisión pesada que no debe llenar tu contexto | un subagent |
| Compartir skills, agents y hooks entre repositorios | un plugin |
| Algo que siempre debe ocurrir | un hook (lección 04), no una skill |

## Examen frente al comportamiento actual

La guía de certificación marca dos respuestas en las que la guía del examen y el Claude Code actual
difieren. La guía del examen describe `allowed-tools` como una restricción de herramientas; el Claude Code
actual lo trata como aprobación por adelantado, y la restricción se hace con `disallowed-tools` o con
reglas deny. La guía del examen también describe `argument-hint` como un aviso que pide los argumentos que
faltan; hoy es solo una pista de autocompletado. La guía además señala que el examen llama `Task` a la
herramienta de subagents, que Claude Code ahora llama `Agent`.

## Fuentes

- Documentación de Claude Code, skills: https://code.claude.com/docs/en/skills
- Documentación de Claude Code, subagents: https://code.claude.com/docs/en/sub-agents
- Documentación de Claude Code, plugins: https://code.claude.com/docs/en/plugins/create y
  https://code.claude.com/docs/en/plugin-marketplaces
- Guía de certificación 3.2, comandos slash y skills:
  https://claudecertificationguide.com/learn/3-claude-code-config/3-2-slash-commands-skills
- Guía de certificación 1.3, subagents:
  https://claudecertificationguide.com/learn/1-agentic-architecture/1-3-subagent-invocation-context
