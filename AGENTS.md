# lex-memory — Agent Guidelines

Structured per-agent memory store: KV upsert-with-supersession, an
append log, scoped/ranked recall, a state blob, and context-injection
formatting — on any caller-supplied `lex-orm` `ConnDb` handle (SQLite
or Postgres). Extracted out of `lex-agent/src/memory` once a second
unrelated consumer (`lex-loom`, alongside `lex-agent`'s own
`lex-soft`) needed the same primitive — see `alpibrusl/lex-memory#1`.

Read `lex agent-guidelines` in full before writing code. The four
highest-leverage discipline rules:

1. **Narrow effects, always.** `fn foo() -> [fs_write("/tmp/x")] T`,
   not `[fs_write]`. If the type checker rejects, narrow the body.
2. **Repair, don't regenerate.** `lex --output json check` →
   `lex repair --apply`. Only regenerate after two failed repairs.
3. **`examples {}` blocks on every pure fn.** They fold into the SigId
   and run at `lex check` time.
4. **Use the stdlib.** `std.crypto` for ids, `std.list` folds over
   hand-rolled recursion.

## The loop

```sh
lex pkg install
lex check --strict src/
lex fmt --check src/ tests/
lex test tests/
```

## Project-specific overrides — lex-memory

- **Mechanism only, never policy.** `kind`, `scope`, `mtype`,
  `importance` are caller-supplied free-form strings (or the fixed
  `mtype` enum: `semantic`/`episodic`/`procedural`). This package
  invents no domain vocabulary — no "lesson", "constraint", "brand",
  "sprint". Hosts (lex-loom, lex-soft) supply that.
- **`store_kv` marks superseded, never deletes.** The "what was true
  when" history is load-bearing for any future consolidation job
  built on top of this table — don't change the upsert path to a
  delete-then-insert; that's what the older `store` function does,
  and it stays for backward compatibility, not as the model for new
  work.
- **`recall_scoped`/`recall_ranked`'s live-view filter
  (`superseded=0`, unexpired) is the contract.** Any new recall
  function must honor the same filter unless it explicitly documents
  why it needs the raw (including-superseded) rows.
- **`ConnDb` dialect-portability is the only DB abstraction.** Every
  query goes through `ormq.for_dialect` so SQLite and Postgres share
  one code path — no dialect-specific branches in this package.
- **`ConnDb.handle` is a raw `Db` from `std.sql`.** Callers own the
  connection lifecycle (open/close); this package never opens its own
  connection outside tests.
- **`init_schema` is idempotent and additive-only.** New columns are
  added via `ADD COLUMN ... DEFAULT ...` behind an error-swallowing
  helper (`add_column_tolerant`), never a destructive
  `DROP`/`RENAME` — a consumer's existing production table must
  survive an upgrade untouched except for the new columns.
- **This package owns `agent_memory` and `agent_state` schema.** A
  consumer must not hand-declare its own DDL for these tables (see
  `alpibrusl/lex-loom#207` — a duplicate, drifted DDL in lex-loom
  caused every memory recall to silently return empty for a period
  because `recall_all`'s own `Err(_) => []` fallback swallowed the
  resulting SQL error). Call `init_schema` and nothing else.
