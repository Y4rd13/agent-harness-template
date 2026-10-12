# 04. Settings, permisos y hooks

Los archivos de instrucciones son consejos: el modelo los lee y normalmente los sigue. Los settings y los
hooks son cumplimiento obligatorio: Claude Code los aplica decida lo que decida el modelo. Todo lo que
deba cumplirse siempre, sin excepción, va aquí.

## Archivos de settings

| Archivo | Compartido | Uso |
| --- | --- | --- |
| Managed settings (definidos por TI) | Todos | Política que nadie puede anular |
| `.claude/settings.local.json` | Solo tú, en gitignore | Tus ajustes locales para este proyecto |
| `.claude/settings.json` | El equipo, a través de git | Los permisos y hooks del proyecto |
| `~/.claude/settings.json` | Solo tú, en todos los proyectos | Tus valores por defecto |

Precedencia, de mayor a menor: managed, luego flags de línea de comandos, luego local, luego project y
luego user. Las reglas deny son especiales: un deny en cualquier nivel bloquea la llamada, aunque otro
nivel la permita.

Plantilla: [settings.json](../../../plugins/agent-harness/templates/.claude/settings.json).

## Permisos

```json
{
  "permissions": {
    "allow": ["Bash(npm test)", "Bash(npm run lint)"],
    "ask": ["Bash(git push *)"],
    "deny": ["Read(./.env)", "Read(./.env.*)", "Read(./secrets/**)"]
  }
}
```

- `allow` se ejecuta sin preguntar, `ask` siempre pide confirmación, `deny` rechaza.
- Deny se evalúa primero: una regla allow no puede crear una excepción dentro de una regla deny.
- Las reglas de Bash comparan el texto del comando por prefijo, con comodines `*`. Eso hace que sea fácil
  esquivarlas: `Bash(git push *)` no coincide con `git -C . push origin main`. Úsalas por comodidad y como
  primera línea de defensa, no como tu única protección para algo peligroso.
- Las reglas deny de `Read` son la forma de mantener los secretos fuera del contexto del modelo.
- Las reglas `allow` de un proyecto se aplican solo después de que aceptas el diálogo de confianza del
  workspace para esa carpeta.

## Hooks

Un hook es un comando que Claude Code ejecuta en un punto de su ciclo. Es código, así que un prompt no
puede hacerlo cambiar su decisión.

| Evento | Se ejecuta | Puede bloquear |
| --- | --- | --- |
| `PreToolUse` | antes de una llamada a una herramienta | sí: código de salida 2, o una decisión `deny` |
| `PostToolUse` | después de una llamada a una herramienta | no: la acción ya ocurrió. Úsalo para revisar o dar formato |
| `UserPromptSubmit` | antes de que tu prompt llegue al modelo | sí |
| `SessionStart` | cuando empieza una sesión | no; puede agregar contexto |

El hook recibe el evento como JSON por stdin. El código de salida 2 bloquea la acción y envía stderr de
vuelta a Claude, para que pueda corregir el rumbo. Los demás códigos de salida distintos de cero se
reportan como errores del hook y la acción continúa.

Un hook que bloquea se impone sobre una regla allow. No evita las reglas deny ni ask: esas se siguen
aplicando.

### El hook de la plantilla

La plantilla bloquea los commits y los pushes a `main`. En pseudocódigo:

```text
command = stdin.tool_input.command
branch  = git branch --show-current
if command runs "git ... push" with a refspec that is main (main, HEAD:main), as a whole word:
    block "no commits or pushes to main"
if branch is main and command runs "git ... commit", or a bare "git push" / "git push <remote>":
    block "no commits or pushes to main"
otherwise allow
```

Viene como una línea de shell en `settings.json`, porque un hook necesita código para ejecutarse. También
coincide con `git -C <dir> push`, que la regla de permisos anterior no detecta.

## Cuándo usar cada uno

| Necesitas | Usa |
| --- | --- |
| Comodidad: dejar de recibir preguntas sobre comandos seguros | reglas `allow` |
| Una segunda revisión antes de algo riesgoso | reglas `ask` |
| Mantener archivos fuera del alcance del modelo | reglas deny de `Read` |
| Una regla que debe cumplirse incluso cuando el comando está escrito de forma rara | un hook `PreToolUse` |
| Una regla para todos, incluidas las personas y otros agentes | un check de CI (lección 06) |

Una regla práctica de la guía de certificación: si un solo fallo costaría dinero o datos reales, usa un
hook, no una instrucción.

## Errores comunes

- Bloquear en `PostToolUse`: la acción ya se ejecutó.
- Confiar en que una regla de permisos de Bash detendrá un comando hecho a propósito para esquivarla:
  compara el texto por prefijo.
- Un hook que falla en silencio (ruta incorrecta, `jq` ausente): la acción continúa. Prueba los hooks con
  JSON de ejemplo por stdin antes de depender de ellos.
- Hacer commit de `.claude/settings.local.json`: es personal; mantenlo en `.gitignore`.

## Fuentes

- Documentación de Claude Code, settings: https://code.claude.com/docs/en/settings
- Documentación de Claude Code, permisos: https://code.claude.com/docs/en/permissions
- Documentación de Claude Code, hooks: https://code.claude.com/docs/en/hooks
- Guía de certificación, referencia rápida del dominio 3:
  https://claudecertificationguide.com/learn/quick-reference/domain-3
- Guía de certificación 1.5, hooks:
  https://claudecertificationguide.com/learn/1-agentic-architecture/1-5-agent-sdk-hooks
