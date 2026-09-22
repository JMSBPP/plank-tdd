# REFERENCE — plank-tdd

## Brady — Type, define, refine

Search the machine for Edwin Brady, *Type-Driven Development with Idris* (Manning, 2017) before inventing process. Do not vendor copyrighted book text.

Core mantra (**Type, define, refine** — Brady §1.2.4):

1. **Type** — write input/output types (and data types) first
2. **Define** — define using the structure of the input types; incomplete defs OK via holes / stubs
3. **Refine** — tighten types and definitions together as the model clarifies

Types are a **plan**, not only a check. Prefer interactive hole-driven editing over writing full bodies up front.

## Algebra-Driven Design (Maguire)

Treat the **algebra** (carriers, operations, laws) as the abstraction clients cannot escape. Ask until carriers, operations, and laws are explicit. Invalid states should be unrepresentable when the kind allows.

Reference: Sandy Maguire, *Algebra-Driven Design*. Use a local copy if you have one; do not paste book text into this repo.

## Std first (mandatory before a new type)

Search **`lib/plank-monorepo/std/`** (and host **`src/types/`**) for types that already have the semantics. List candidates on the type note / `types.toml` `notes`. Implement a new type only after rejecting each candidate.

There is **no** `Result` / `Either` / `Outcome` in std today.

| Need | Look first |
|------|------------|
| Presence / absence, optional payload | `std::option::{Option, Some, None, unwrap}` |
| Abort when a predicate fails | `std::error::require` (void; `@evm_revert`) |
| Call success bit | `bool` from `@evm_call` / `@evm_staticcall` |
| Address | `std::core::addr` |
| Checked arithmetic | `std::core::ops` (`checked_mul`, `neg_u256`, …) |

### Example — `Outcome = success | revert`

| Candidate | Semantics already there? | Fit |
|-----------|--------------------------|-----|
| `Option(T)` | `Some` / `None`; `unwrap` reverts | **Yes** — `None` = revert, `Some` = success (payload `T` if returndata matters, else a unit) |
| `bool` | EVM call flag | Same information as `{ ok: bool }`, no constructors |
| `require` | Reverts; returns nothing | Not a returned outcome |
| Hand-rolled `Outcome { ok }` | — | **Reject** unless Option/bool are rejected in AskQuestions |

Default: reuse `Option`. A domain `Outcome` wrapper is a refine only if the algebra needs names `success`/`revert` that Option cannot carry.

## IO side-effect modules

A Plank type that touches the EVM is a **description**, then an **IO** of that description, then **run**.

```
IO      : type → type
io      : T → IO(T)
run     : IO(T) → Outcome   // prefer std::option::Option unless rejected
View    = staticcall
Xfer    = call
Outcome = success | revert  // Option(T) or bool — not a new std type
```

Rules:

- `T` is the command (e.g. `TokenFlow(σ_F, dt, token, from, to)`), not `void`
- `IO` is **generic** in `T` (command-indexed), not a single ambient IO
- `io` only wraps; it does not execute
- `run(io(…))` is the only executor; name it `run_io` in Plank if `run` clashes with a harness export
- List **Eff** on the description type: which View/Xfer selectors exist
- `run` maps description fields onto those selectors (e.g. `run(io(amt,+)) = Xfer(from,to,amt)`)
- Solidity fixtures **import only the Compose/EVM modules Eff needs** (Transfer + `balanceOf`, not Approve/Mint/full Data)
- Seed unused storage from the test, or do not need it
- No extra on-chain balance proof unless the algebra says so

Split files when kinds differ: plain carrier, dependent description, generic `IO`.

## Type kinds (categorize before defining)

Record one primary kind in `types.toml`:

| Kind | Meaning |
|------|---------|
| `plain` | Non-parameterised wrapper / simple alias |
| `generic` | Parameterised, non-dependent (`IO(T)`) |
| `dependent` | Type depends on values |
| `indexed` | Family indexed by another type/value |
| `other` | Explain in metadata `notes` |

## `types.toml` schema

Path: `{working_dir}/types.toml` (create if missing).

```toml
[TokenFlow]
module = "TokenFlow"
kind = "dependent"
file = "types/TokenFlow/TokenFlow.md"
carriers = ["TokenAmount", "Dir"]
operations = ["intro"]
laws = ["intro(σ_F, ΔW) = ((σ_F · |ΔW|)/RAY, sign(ΔW))", "Eff = [ERC20View, Xfer]"]
refined = false
notes = "Type depends on values σ_F, dt, token, from, to"

[IO]
module = "IO"
kind = "generic"
file = "types/IO/IO.md"
carriers = ["T"]
operations = ["io", "run"]
laws = ["run(io(amt,+)) = Xfer(from,to,amt)", "Outcome = success | revert"]
refined = false
notes = "IO(T); T means execute that family"
```

On the type note / `.plk`, add a pointer: `-- types.toml: TokenFlow`

When refining, set `refined = true` and amend `laws`.

## Host layout (cfmm-vol-markets and similar)

- Type: `src/types/Foo.plk`
- Harness: `test/harness/types/FooHarness.plk`
- Suite: `test/types/Foo.t.sol` inherits `PlankTestBase`, declares the harness ABI
- Domain compile list: `.spec/<DOMAIN>.spec/compile.toml` — every new `.plk`
- Validation: follow `AGENTS.md` / `CLAUDE.md` of the host (often **push → CI**, no local `forge` / `plank`)

## Compose / SCOP (Solidity fixtures only)

When the Eff row names an ERC-20 (or other Compose) surface:

- No inheritance in contracts (tests excepted)
- File-level Mods, not `library`
- No `modifier`, no ternary, no `public` (external/internal)
- Underscore params
- Import **only** the Mods the Eff row uses

## Hybrid with behavioral TDD

From sibling `tdd`:

- Vertical slices only (one behavior → implement → next)
- Tests describe WHAT through the public harness ABI
- No mocking Plank internals; no testing hole names

Plank-specific RED: harness selector missing / stub returns 0 / revert. GREEN: fill the hole for that behavior only.

## Anti-patterns

- New type without searching `std/` and host `src/types/`
- Hand-rolled `Outcome`/`Result` when `Option` already fits
- Void “do the effect” functions
- Importing unused Compose Mods (Approve/Mint when Eff is Transfer + balanceOf)
- Bodies before algebra agreement
- Skipping LaTeX on the type note
- Bulk tests before any type exists
- Omitting `types.toml` or `compile.toml`
- Local build as proof of correctness when the host forbids it
