# Harness checks

Each check below keeps an agent harness honest. They are written as pseudocode on purpose: port the ones
you want to CI in your own language, or let the `audit` skill run them by reading the repository. Three of
them already ship as real code in the templates: `file-limits` and `rule-paths` in
`templates/.github/workflows/instruction-files.yml`, and the quote check of `ai-review.yml`.

Every check reports `file:line`, the check id, a severity (`error` blocks, `warn` informs) and the fix.

## file-limits

Instruction files past these sizes lose adherence and crowd every session's context.

```text
for (file, limit) in [(AGENTS.md, 200), (CLAUDE.md, 40)] + [(r, 200) for r in .claude/rules/*.md]:
    if file exists and line_count(file) > limit:
        error(file, "{n} lines, limit {limit}: move detail to docs/conventions/ or a path-scoped rule")
if AGENTS.md does not exist:
    error("AGENTS.md", "missing: it is the canonical conventions file")
```

## rule-paths

A rule without a parsable `paths:` loads in every session, and nothing reports it.

```text
for rule in .claude/rules/*.md:
    front = text between a first line "---" and the next "---"
    if front is missing or YAML does not parse or "paths" not in front:
        error(rule, "no paths: frontmatter, so it loads in every session")
```

## claude-imports-agents

When a repository has a CLAUDE.md, Claude Code reads it instead of AGENTS.md, so the import must exist.

```text
if CLAUDE.md exists:
    if no line of CLAUDE.md equals "@AGENTS.md":
        error(CLAUDE.md, "does not import AGENTS.md; add @AGENTS.md on its own line")
    for line in CLAUDE.md where line starts with "@":
        if file at path(line) relative to CLAUDE.md does not exist:
            error(CLAUDE.md:line, "import points to a missing file")
```

## paths-globs-match

A glob that matches nothing is a rule that never loads, usually after a directory was renamed.

```text
tracked = git ls-files
for rule in .claude/rules/*.md:
    for glob in rule.paths:
        if no file in tracked matches glob:      # "**" spans directories, "*" does not
            warn(rule, "paths glob {glob} matches no file")
```

## backtick-paths-exist

Instruction files point at files; a pointer to a deleted file sends the agent nowhere.

```text
for file in [AGENTS.md, CLAUDE.md] + .claude/rules/*.md + docs/conventions/*.md:
    for token inside `backticks` that looks like a repository path (has "/" or a file extension,
            no spaces, no "<", not a URL, not a glob):
        if path does not exist from the repository root:
            error(file:line, "points to {token}, which does not exist")
```

## commands-exist

A command in AGENTS.md that does not exist teaches the agent to fail.

```text
scripts = names under "scripts" in package.json, targets in Makefile or justfile, tasks in pyproject.toml
for each command line in AGENTS.md "## Commands":
    if it calls a script runner (npm run X, pnpm X, make X, just X, uv run task X) and X not in scripts:
        error(AGENTS.md:line, "command {X} is not defined")
```

## claims

A claim binds one line of an instruction file to a fact, so the line cannot go stale silently.

```text
for claim in .claude/claims.json "claims":
    if claim.quote is not exactly one whole line of claim.source:
        error(claim.source, "claim {id}: quoted line not found; the file changed, update the claim")
    case claim.type:
        file_exists:   ok = exists(claim.args.path)
        regex_present: ok = regex(claim.args.pattern) matches in file(claim.args.path)
        regex_absent:  ok = not regex(claim.args.pattern) matches in file(claim.args.path)
    if not ok:
        error(claim.source, "claim {id} is false: {quote}")
```

Read the claims file from the base branch in CI, not from the pull request: the author of a pull request
should not be able to delete the claim their change breaks.

## pinned-actions

A tag can be moved to other code after you reviewed it; a commit SHA cannot.

```text
for workflow in .github/workflows/*.yml, and every action.yml:
    for each "uses: X@REF" where X does not start with "./":
        if REF is not 40 hexadecimal characters:
            error(workflow:line, "{X}@{REF} is not pinned to a commit SHA")
    for each "image:" or "docker://" reference with a registry:
        if it has no "@sha256:" digest:
            error(workflow:line, "container image is not pinned by digest")
```

## run-interpolation

`${{ }}` inside `run:` is pasted into the shell before it runs. A pull request title or branch name there
is code the author controls.

```text
SAFE = [github.sha, github.run_id, github.run_number, github.repository, github.event.pull_request.number,
        github.event.pull_request.base.sha, github.event.pull_request.head.sha, steps.*.outputs.*,
        needs.*.outputs.*, vars.*, secrets.*, matrix.*, inputs.* of workflow_call]
for each run: block and each actions/github-script "script:" block:
    for each ${{ expr }} in it:
        if expr reads a context not in SAFE (title, body, head_ref, branch names, comments, labels, ...):
            error(workflow:line, "move {expr} to env: and use the variable in the script")
```

## comment-conventions

History in comments rots; it belongs in the commit and the ticket.

```text
for each source file changed by the pull request:
    for each comment (skip string literals and URLs):
        if it matches the ticket key pattern of your tracker (for example [A-Z]+-[0-9]+):
            error(file:line, "ticket key in a comment")
        if it contains an em-dash:
            warn(file:line, "em-dash in a comment; write the sentence plainly")
```

## single-important

When every line is important, none is.

```text
for file in [AGENTS.md, CLAUDE.md] + .claude/rules/*.md:
    n = count of lines containing "IMPORTANT" or "MUST" in capitals, outside code blocks
    if n > 1:
        warn(file, "{n} emphasised lines; keep one at most")
```

## contradictions

Two lines that disagree make the agent pick one at random, and it will not say which.

```text
statements = every rule-like line in AGENTS.md, CLAUDE.md, .claude/rules/*.md, docs/conventions/*.md
group statements by topic (a command, a tool, a directory, a limit, a branch, a format)
for each topic with more than one statement:
    if two statements require incompatible things (different commands for the same job, different
            limits for the same file, "always" against "never"):
        error(both locations, "contradiction: {a} vs {b}")
```

This one needs judgement; it is the reason the `audit` skill is an agent and not only a script.
