# 02. Reglas con alcance por ruta

Algunas convenciones importan solo en una parte del árbol: cómo se escriben los handlers HTTP, cómo se
estructuran las migraciones, cómo se nombran los tests. Ponerlas en `AGENTS.md` cuesta contexto en cada
sesión, incluidas las que nunca tocan ese código. Una regla con alcance por ruta se carga solo cuando hace
falta.

## El formato

Un archivo Markdown en `.claude/rules/` con una lista `paths:` en su frontmatter:

```markdown
---
paths:
  - "src/http/**"
  - "tests/**/*.test.ts"
---

# HTTP handlers

- Handlers never query the database directly; they call a service.
```

Plantillas: [testing.md](../../../plugins/agent-harness/templates/.claude/rules/testing.md),
[layer.md](../../../plugins/agent-harness/templates/.claude/rules/layer.md).

## Cómo se carga

- Una regla se carga cuando Claude lee, escribe o edita un archivo que coincide con uno de sus globs
  (también cuando ve ese archivo con una lectura simple de shell como `cat`).
- `paths` es el único campo de frontmatter que Claude Code lee en una regla; los demás campos se ignoran
  sin avisar.
- **Una regla sin `paths:` se carga en todas las sesiones**, como un segundo `CLAUDE.md`. Lo mismo ocurre
  con una regla cuyo YAML no se puede interpretar. Nada te avisa. Por eso la plantilla
  `instruction-files.yml` hace fallar un pull request que agrega una regla sin `paths:`.
- Las reglas de nivel usuario en `~/.claude/rules/` funcionan igual y se aplican a todos los proyectos.

## Globs

| Patrón | Coincide con |
| --- | --- |
| `src/**/*` | todo lo que está bajo `src/` |
| `**/*.ts` | todos los archivos TypeScript |
| `src/**/*.{ts,tsx}` | expansión con llaves: dos patrones |
| `*.md` | archivos Markdown solo en la raíz |

Un glob que no coincide con ningún archivo es una regla que nunca se carga, normalmente porque se renombró
un directorio. El check `paths-globs-match` de
[checks.md](../../../plugins/agent-harness/reference/checks.md) lo detecta.

## ¿Regla, CLAUDE.md de directorio o AGENTS.md?

| La convención se aplica a | Ponla en |
| --- | --- |
| Todas las sesiones | `AGENTS.md` |
| Un directorio y todo lo que contiene | una regla con `paths: ["that/dir/**"]` (o un `CLAUDE.md` de directorio) |
| Un tipo de archivo repartido por el árbol (tests, migraciones, componentes) | una regla con un glob |
| Un invariante de seguridad o de datos | `AGENTS.md`, Critical Invariants, aunque afecte a una sola ruta |

La última fila importa: solo Claude Code lee `.claude/rules/`. Los demás agentes, y los revisores que leen
`AGENTS.md`, nunca ven una regla que esté ahí. Deja en `AGENTS.md` la línea que todo agente debe ver, y los
detalles en la regla. Lista cada regla en la sección "Detail on demand" de `AGENTS.md` para que los demás
agentes sepan que existe.

## Errores comunes

- Sin `paths:`, o con un error de escritura: la regla se carga en todas partes.
- Una regla para tests en `tests/CLAUDE.md` cuando los archivos de test están junto al código en muchos
  directorios: una regla con glob los cubre todos.
- Esperar que una regla se aplique antes de tocar el primer archivo que coincide: no está en el contexto
  hasta ese momento.
- Suponer que una regla sigue ahí después de `/compact`: vuelve en la siguiente lectura que coincida.

## Fuentes

- Documentación de Claude Code, reglas específicas por ruta:
  https://code.claude.com/docs/en/memory#path-specific-rules
- Guía de certificación 3.3, reglas específicas por ruta:
  https://claudecertificationguide.com/learn/3-claude-code-config/3-3-path-specific-rules
