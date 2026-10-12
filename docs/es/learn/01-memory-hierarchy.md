# 01. La jerarquía de memoria: CLAUDE.md, AGENTS.md y archivos relacionados

Un asistente de programación con IA empieza cada sesión sin saber nada de tu repositorio. Los archivos de
instrucciones son la forma en que aprende tus comandos, tu arquitectura y tus reglas antes de tocar algo.
Esta lección explica qué archivos lee Claude Code, en qué orden, y cómo encaja `AGENTS.md`.

## Los niveles

| Nivel | Ubicación | Se comparte con | Úsalo para |
| --- | --- | --- | --- |
| Managed policy | `/etc/claude-code/CLAUDE.md` (Linux), definido por TI | Todos los usuarios de la máquina | Reglas de la organización que nadie puede desactivar |
| User | `~/.claude/CLAUDE.md` y `~/.claude/rules/` | Solo tú, en todos los proyectos | Cómo te gusta trabajar |
| Project | `./CLAUDE.md` o `./.claude/CLAUDE.md`, más `.claude/rules/` | El equipo, a través de git | Las convenciones del repositorio |
| Local | `./CLAUDE.local.md` (en gitignore) | Solo tú, en este proyecto | Tus URLs de sandbox, tus datos de prueba |
| Directory | `src/payments/CLAUDE.md` | El equipo | Reglas para un subárbol, cargadas bajo demanda |

Plantillas: [AGENTS.md](../../../plugins/agent-harness/templates/AGENTS.md),
[CLAUDE.md](../../../plugins/agent-harness/templates/CLAUDE.md),
[CLAUDE.local.md.example](../../../plugins/agent-harness/templates/CLAUDE.local.md.example),
[user-level/](../../../plugins/agent-harness/templates/user-level/CLAUDE.md).

## Cómo se cargan

- **Se concatenan, no se sobrescriben.** Cada archivo encontrado se agrega al contexto. Ninguno "gana": si
  dos líneas se contradicen, el modelo puede seguir cualquiera de las dos. Entre directorios, el contenido
  se ordena desde la raíz del sistema de archivos hasta tu directorio de trabajo.
- **Los ancestros al iniciar, los subdirectorios bajo demanda.** Los archivos del directorio de trabajo y
  de los directorios superiores se cargan cuando empieza la sesión. Un `CLAUDE.md` en un subdirectorio se
  carga cuando Claude lee o edita un archivo de ese subdirectorio.
- **Es contexto, no configuración.** El modelo lee estos archivos y normalmente los sigue, pero nada lo
  obliga. Lo que siempre debe cumplirse va en settings, hooks o CI (lecciones 04 y 06).
- **Compactación.** Después de `/compact`, el `CLAUDE.md` de la raíz del proyecto se vuelve a leer desde
  disco. Los archivos anidados y las reglas con alcance por ruta vuelven solo cuando se lee de nuevo un
  archivo que coincide. Una instrucción que escribiste solo en el chat desaparece: si importa, ponla en un
  archivo.

## Imports

Una línea formada por `@` y una ruta importa ese archivo:

```markdown
@AGENTS.md
@docs/conventions/testing.md
```

Las rutas relativas se resuelven desde el archivo que importa. Los imports se anidan hasta cuatro niveles.
Un import no ahorra contexto: el archivo importado se carga al iniciar, igual que si lo hubieras pegado.

## AGENTS.md: un archivo para todos los agentes

`AGENTS.md` es una convención en Markdown plano que leen muchos agentes de programación. Mantén ahí tus
convenciones, una sola vez, y deja `CLAUDE.md` como un archivo mínimo que lo importa:

```markdown
@AGENTS.md

## Claude-specific notes
- Skills, plugins or MCP servers that only Claude Code uses.
```

Las versiones recientes de Claude Code leen `AGENTS.md` por su cuenta, pero solo cuando no hay un
`CLAUDE.md` ni un `CLAUDE.local.md` en el directorio de trabajo ni por encima de él, y algunas sesiones no
pueden leerlo directamente. El import `@AGENTS.md` funciona en todas las sesiones y nunca carga el archivo
dos veces, así que consérvalo.

## Tamaño y estilo

- Apunta a menos de 200 líneas por archivo. Los archivos más largos cuestan contexto en cada sesión y se
  siguen menos.
- Para cada línea, pregúntate: si la quito, ¿el agente cometería un error? Si no, elimínala.
- Los comentarios HTML de bloque (`<!-- note -->`) se eliminan antes de que el modelo vea el archivo.
  Úsalos para notas dirigidas a quienes mantienen el archivo; las plantillas lo hacen.
- No pegues lo que el agente puede leer en el código (listados de directorios, versiones de dependencias)
  ni lo que ya sabe ("escribe código limpio").

## Ver qué se cargó

- `/memory` lista los archivos de memoria de la sesión y abre uno para editarlo.
- `/context` muestra qué hay en la ventana de contexto, con una sección "Memory files".

Ninguno de los dos comandos carga nada.

## Errores comunes

- Convenciones del equipo en `~/.claude/CLAUDE.md`: funcionan solo en tu máquina, y el agente de un nuevo
  compañero se comporta distinto.
- Un `CLAUDE.local.md` agregado a un repositorio que depende de que Claude lea `AGENTS.md` directamente:
  cuenta como un archivo `CLAUDE.md`, así que `AGENTS.md` deja de cargarse. Usa el import.
- Un `CLAUDE.md` que dice "lee AGENTS.md" con palabras en lugar de importarlo: Claude ve el archivo solo si
  decide abrirlo.
- Un `CLAUDE.md` de directorio para una convención que abarca varios directorios (por ejemplo, los tests en
  todas partes): usa una regla con alcance por ruta (lección 02).

## Examen frente al comportamiento actual

La guía de certificación acepta tanto `./CLAUDE.md` como `./.claude/CLAUDE.md` como archivo del proyecto;
ambos son válidos. Señala que no existe una palabra clave `@import`: el import es una línea que empieza con
`@`.

## Fuentes

- Documentación de Claude Code, memoria: https://code.claude.com/docs/en/memory
- El formato de `AGENTS.md`: https://agents.md
- Guía de certificación 3.1, jerarquía de CLAUDE.md:
  https://claudecertificationguide.com/learn/3-claude-code-config/3-1-claude-md-hierarchy
