# CLAUDE.md — lex-memory

> Copy this file into the root of any Lex project repository as
> `CLAUDE.md` (read by Claude Code), `AGENTS.md` (read by Cursor /
> Aider / Codex CLI / Copilot CLI), or both. This repo ships both,
> kept in sync.

This repository is a **Lex** project — the shared agent-memory
primitive consumed by `lex-loom` and `lex-soft`. Read
`AGENTS.md` in full before writing code; it carries the
project-specific discipline (mechanism-only, schema ownership,
supersession-not-deletion) alongside the standard `lex
agent-guidelines` rules.

## The loop

```sh
lex pkg install
lex check --strict src/        # type-check with extra lints
lex fmt --check src/ tests/    # formatting (must be canonical)
lex test tests/                 # all tests/test_*.lex files
```

See `AGENTS.md` for the full discipline, including why this package
owns the `agent_memory`/`agent_state` schema exclusively and why
`store_kv` supersedes rather than deletes.
