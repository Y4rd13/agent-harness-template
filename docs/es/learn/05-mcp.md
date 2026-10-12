# 05. Servidores MCP

Los servidores MCP (Model Context Protocol) le dan al agente herramientas más allá de las integradas: tu
gestor de incidencias, una base de datos, un navegador, un índice de documentación. Son potentes y cuestan
contexto, así que agrégalos con un propósito.

## Dónde se configuran

| Alcance | Se guarda en | Compartido | Uso |
| --- | --- | --- | --- |
| Local (el predeterminado) | `~/.claude.json`, bajo este proyecto | Solo tú, este proyecto | Probar un servidor, credenciales personales |
| Project | `.mcp.json` en la raíz del repositorio | El equipo, a través de git | Servidores que usa todo el equipo |
| User | `~/.claude.json` | Solo tú, todos los proyectos | Tus herramientas personales |

Agrega uno con `claude mcp add <name> --scope project ...`, o escribe `.mcp.json` a mano.

Plantilla: [.mcp.json.example](../../../plugins/agent-harness/templates/.mcp.json.example).

## El formato

```json
{
  "mcpServers": {
    "docs": {
      "type": "http",
      "url": "https://mcp.example.com/mcp",
      "headers": { "Authorization": "Bearer ${DOCS_MCP_TOKEN}" }
    },
    "local-db": {
      "command": "npx",
      "args": ["-y", "<package>@<exact version>"],
      "env": { "DATABASE_URL": "${DATABASE_URL:-postgresql://localhost:5432/dev}" }
    }
  }
}
```

- Un servidor remoto necesita `"type": "http"`. Una entrada con `url` y sin `type` se interpreta como un
  servidor local (stdio) y falla.
- `${VAR}` y `${VAR:-default}` se expanden desde el entorno. Una variable ausente y sin valor por defecto
  produce una advertencia y el servidor se carga con el texto sin expandir.
- **Nunca hagas commit de una credencial en `.mcp.json`.** Referencia una variable de entorno en su lugar.
- Fija la versión de un servidor que ejecutes con `npx` o `uvx`, como fijas cualquier otra dependencia.

## Buenas prácticas

- Prueba un servidor de la comunidad o de un proveedor antes de escribir el tuyo.
- Las herramientas y descripciones de cada servidor ocupan contexto. Mantén corta la lista del proyecto y
  deja los servidores que usas poco en tu alcance local.
- Las descripciones de las herramientas importan: una vaga pierde frente a las herramientas integradas y
  nunca se usa.
- Revisa el estado con `/mcp` dentro de una sesión, o con `claude mcp list` en el shell.
- En CI, inicia el agente con `--strict-mcp-config` y una lista explícita (o ninguna), para que un pull
  request no pueda agregar un servidor a la ejecución de la revisión (lección 07).

## Examen frente al comportamiento actual

La guía de certificación señala que el examen describe dos alcances, project (`.mcp.json`) y user
(`~/.claude.json`), mientras que el Claude Code actual tiene tres: local (el predeterminado), project y
user.

## Fuentes

- Documentación de Claude Code, MCP: https://code.claude.com/docs/en/mcp
- Guía de certificación 2.4, integración de servidores MCP:
  https://claudecertificationguide.com/learn/2-tool-design-mcp/2-4-mcp-server-integration
