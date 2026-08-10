# lex-memory

Structured per-agent memory store for Lex agents — a small, mechanism-only
package extracted out of [`lex-agent`](https://github.com/alpibrusl/lex-agent)
once a second unrelated consumer needed the same primitive (see
[alpibrusl/lex-memory#1](https://github.com/alpibrusl/lex-memory/issues/1)).

## What it provides

On any caller-supplied `lex-orm` `ConnDb` handle (SQLite or Postgres):

- **KV memory** — upsert-on-key named facts (`store_kv`), superseding
  rather than deleting the previous value so "what was true when"
  history survives.
- **Append log** — ordered observations appended without a key.
- **Scoped / ranked recall** — `recall_scoped` (one scope) and
  `recall_ranked` (across every scope for an agent), both ordered by
  importance then recency, filtering out superseded and expired rows.
- **State blob** — opaque per-agent JSON, backward-compatible with the
  original `state_store` shape.
- **Context injection** — `to_context`/`to_context_labeled` format
  recalled entries as a bullet list ready to concatenate into a system
  prompt.

See `src/memory.lex` for the full API and `AGENTS.md` for the
project's discipline rules (mechanism vs. policy, schema ownership,
supersession contract).

## Usage

```lex
import "lex-orm/src/connection" as conn
import "lex-memory/src/memory" as mem

let db := conn.connect_sqlite("/var/lib/agent.db")
let __i := mem.init_schema(db)

let __w := mem.store_kv(db, "agent-1", "constraint", "budget", "5000",
  "semantic", "high", "global", "")

let entries := mem.recall_ranked(db, "agent-1", 10)
let prompt_suffix := mem.to_context(entries)
```

## Consumers

- [`lex-loom`](https://github.com/alpibrusl/lex-loom) — agent runner
  state, cross-sprint lessons.
- [`lex-soft`](https://github.com/alpibrusl/lex-soft) — `trace.lex`'s
  `remember_fact`/`remember_kv`/`recall_facts_text`/`recall_memory_json`.

## Testing

```sh
lex pkg install
lex check --strict src/
lex fmt --check src/ tests/
lex test tests/
```

## License

EUPL-1.2 — matches the rest of the lex ecosystem.
