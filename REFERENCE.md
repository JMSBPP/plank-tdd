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

## Type note headings (math notation)

The type note (`{working_dir}/types/<Type>/<Type>.md`) is the **authoritative algebra**. Host projects may treat the checked-in note as the respected design state (e.g. `StateView.md` on the `type/StateView` track). Agents must **not** replace maintainer notation without approval.

### Phases

| Phase | What goes on the note |
|-------|----------------------|
| **Type** | Carriers, main `\[ \begin{aligned} … \]` blocks, `types.toml` kind/laws, std-reuse table, holes listed as signatures inside aligned math |
| **Define** | **Add or extend** a section per locked behavior: math **title/subtitle**, then laws block, BTT link, Plank/harness names |
| **Refine** | Tighten laws, Eff row, `refined = true`; optional `> KEEP THIS NOTATION` anchors |

### Document title

```markdown
# [TYPE:: STATE_VIEW](MAIN_REF# MODEL)
```

### Major sections (Eff, cross-cutting)

Use `##` + roman name when the block is not a single operation (e.g. Eff list):

```markdown
## SIDE_EFFECTS
\[
\begin{aligned}
\mathrm{Eff}^{\mathrm{StateView}} &= [\mathrm{ERC20View},\,\mathrm{Xfer},\,\mathrm{Swap}] \\
\mathrm{Xfer} &\subset \mathrm{Swap}
\end{aligned}
\]
```

### Operation subtitles (define phase) — **custom math headings**

Each **defined operation** gets a `###` whose **visible title is the type signature**, not plain English. Use **inline math split across lines** (renderer-tolerant pattern):

```markdown
### \(\mathrm{intro}_{\mathrm{anchor}}
::
\mathrm{Pool}(\mathrm{Algebra})
\to
\mathrm{StateViewAnchor}\)

\[
\begin{aligned}
\mathrm{intro}_{\mathrm{anchor}}(\mathrm{pool}) &= \bigl(\mathrm{pool},\; t_{\mathrm{init}} \leftarrow \mathrm{timestamp}\bigr) \\
\mathrm{pool\_word}(\mathrm{pool}) = 0 &\Longrightarrow \mathrm{revert}\ \mathtt{ZeroPool}
\end{aligned}
\]
```

Rules:

- **`###`** opens the operation; **`::`** and arrows **`→`** live in the same signature block as in the main algebra (repeat the **same** judgment, do not drift).
- Follow immediately with a display **`\[ aligned \]`** block: laws, reverts, and define-slice facts (BTT name, no Eff if pure anchor).
- Plank name mapping: optional short prose line or `### IO algebra (Plank names)` with `io` / `run` / `Outcome` when Eff is involved.
- For operations already declared in the main carrier block, you may add `> KEEP THIS NOTATION` and repeat the **`###`** signature header before the deeper step/cell laws (see `step_K` in StateView).

### Anti-patterns

- English-only `### Intro` with no signature line after a define slice lands
- Prose-only define docs with no aligned laws block
- Invalid TeX (`\forall_\K`, commands inside `\text{}`, unclosed `\[` )
- Rewriting maintainer blockquotes (`> DO NOT ERASE`, design notes) while filling define sections

Canonical host example: `cfmm-vol-markets` → `.spec/REALIZED_VOLATILITY.spec/types/StateView/StateView.md`.

## BTT / Bulloak (every define)

Define **creates behavioral semantics**. That is a [Branching Tree Technique](https://www.getfoundry.sh/guides/branching-tree-technique) file, then [Bulloak](https://github.com/alexfertel/bulloak) generates the Foundry suite. This is not optional and not refine-only.

```
{working_dir}/types/<Type>/<Type>.md     algebra
{working_dir}/types/<Type>/<Type>.btt    ← write this (one behavior)
        │
        │  bulloak scaffold <that.btt>   (stdout; do not -w)
        ▼
test/types/<Type>.t.sol                  ← generated suite
        │
        ▼
harness ABI + assertions → fill the Plank hole
```

`bulloak scaffold -w` writes a `.t.sol` **next to the tree**. That would drop tests into `.spec/`. Always redirect stdout into `test/**`.

Rules:

- One `.btt` tree per define slice (that behavior’s success + invalid branches from the type note)
- Path: `{working_dir}/types/<Type>/<Type>.btt` or `{Type}<Behavior>.btt` (same folder as the LaTeX note)
- **Update `{Type}.md`** in the same slice: add the operation’s **math heading** + laws (see [Type note headings](#type-note-headings-math-notation))
- The type note and `src/types/<Type>.plk` **point at that `.btt`** (`/// .btt: …` / markdown link). That is the behavior source.
- Root is the test contract (`FooTest` or `Foo::intro` if several trees share a file)
- Conditions: `when` / `given`. Leaves: `it …`
- Use `├` / `└` branches
- **Do not** hand-write a define-phase `*.t.sol`. Scaffold, then wire `PlankTestBase` and the harness ABI
- Do not rename Bulloak-generated test functions to dodge `bulloak check`
- If the host Bulloak pin only accepts `.tree`, keep the same basename and the same contents (`.btt` remains the skill name)
- `bulloak check` pairs `<name>.tree` with `<name>.t.sol` by basename. If the pin requires the same directory, do not move the `.btt`; keep the split and treat `check` as optional until the host CI copies or the pin grows an output path
- Follow the host validation rule (often push → CI). `bulloak scaffold` is generation, not a local proof

Missing Bulloak on the host: say so; do not silently skip to a hand-rolled suite. Adding Bulloak to CI is a host concern (see the host `TODO` / `AGENTS`).

## Anti-patterns

- New type without searching `std/` and host `src/types/`
- Hand-rolled `Outcome`/`Result` when `Option` already fits
- Void “do the effect” functions
- Importing unused Compose Mods (Approve/Mint when Eff is Transfer + balanceOf)
- Bodies before algebra agreement
- Skipping LaTeX on the type note
- Bulk tests before any type exists
- Hand-rolling a define-phase suite instead of `.btt` + Bulloak
- Omitting `types.toml` or `compile.toml`
- Local build as proof of correctness when the host forbids it

## GSD state-only adapter

### Trigger and ownership

Activate this adapter only when all of these are true:

1. a maintainer-approved plan exists;
2. the plan is decomposed into child slices;
3. a child slice is assigned to `/plank-type`, `/plank-define`, or
   `/plank-refine`; and
4. the plan declares GSD tracking.

The approved plan owns slice scope and order. Plank TDD owns algebra, Brady
phase gates, BTT/TDD, implementation, code approval, and CI acceptance. GSD
owns none of those decisions; its files are a durable progress ledger.

### Fail-closed prerequisites

Before starting the first tracked slice, require both:

- `.planning/ROADMAP.md`
- `.planning/STATE.md`

If either is absent, stop and ask the maintainer to initialize GSD. Do not
bootstrap GSD, create replacement top-level files, or silently run untracked.

### Phase registration and mapping

Register exactly one GSD phase for each approved child slice. Preserve the
host's phase numbering and ROADMAP/STATE formatting. The phase record must
identify:

- child issue or PR, when assigned;
- behavior name;
- approved-plan path or URL;
- Brady tag: `type`, `define`, or `refine`;
- phase-local directory and `PLANK-STATE.md`.

Do not combine child slices merely because they share a type, and do not split
an approved child slice through this adapter. A scope change returns to the
maintainer/approved-plan process.

### Lifecycle

The only Plank state progression is:

```text
pending → in_progress → code_approved → committed → ci_pending → complete
                                                               ↘ blocked
```

`blocked` may be entered from any active state. Recovery from `blocked`
returns to the last satisfied state, records the resolution, and then resumes
forward progression. Never infer approval, commit, or CI success from elapsed
time or file presence.

Transition meaning:

| Status | Required evidence |
|---|---|
| `pending` | Approved child slice registered |
| `in_progress` | Matching Plank command has started the slice |
| `code_approved` | Maintainer approved every changed code/documentation chunk required by the host |
| `committed` | Commit SHA recorded |
| `ci_pending` | Branch pushed and CI URL recorded |
| `complete` | Host-authoritative CI passed and slice acceptance criteria are met |
| `blocked` | Concrete blocker and prior status recorded |

Synchronize the phase-local file first, then update ROADMAP and STATE summaries
in the same editing operation. Use targeted direct writes; do not invoke GSD
workflow skills or commands.

### Phase-local `PLANK-STATE.md`

Store this file in the existing phase directory:
`.planning/phases/<phase-directory>/PLANK-STATE.md`.

```markdown
---
plank_state_version: 1
phase: "12"
slice: "refine/Example"
plank_phase: "refine"
status: "in_progress"
issue: "https://github.com/org/repo/issues/123"
pr: ""
approved_plan: "docs/plans/example.md"
behavior: "Describe the approved child behavior"
commit: ""
ci: ""
blocked_reason: ""
blocked_from: ""
updated_at: "2026-09-23T00:00:00Z"
---

# Plank TDD State

## Artifacts
- `path/to/artifact`

## Transition log
- `2026-09-23T00:00:00Z` — `pending → in_progress`
```

Use empty strings for unknown scalar values. Keep artifact paths
repo-relative. Append, rather than rewrite, transition history.

### Command synchronization

For `/plank-type`, `/plank-define`, and `/plank-refine`:

1. At command start, verify the phase matches the approved child slice and
   transition `pending → in_progress`.
2. As artifact paths become known, append them without changing scope.
3. After the maintainer's final chunk approval, record `code_approved`.
4. After commit, record the SHA and `committed`.
5. After push, record the CI URL and `ci_pending`.
6. After host-authoritative CI and acceptance criteria pass, record
   `complete`.
7. On an unresolved dependency, approval denial, failed CI awaiting a new
   approved action, or access failure, record `blocked` with the reason and
   previous status.

Do not mark `complete` merely because implementation files exist. Do not
advance a state transition belonging to a different child slice.

### Recovery and reconciliation

On resume, compare `PLANK-STATE.md` with repository and GitHub evidence:

- a recorded commit must resolve in the current repository;
- `ci_pending` must have a pushed commit and CI URL;
- `complete` must retain passing host-authoritative CI evidence;
- ROADMAP and STATE summaries must agree with the phase-local status.

If evidence is missing, move to `blocked`; do not fabricate it. If summaries
drift but phase-local evidence is valid, repair ROADMAP and STATE and append a
reconciliation entry. If multiple phase files claim the same child slice,
stop for maintainer resolution.

### Forbidden GSD surface

Never use this adapter to:

- invoke GSD planning, execution, discussion, review, verification, or
  `progress --next`;
- generate GSD `PLAN.md`, `SUMMARY.md`, research, context, review, or
  checkpoint artifacts;
- dispatch GSD agents;
- select, reorder, decompose, approve, execute, or validate Plank work;
- replace the approved plan, Plank command rules, AskQuestion approvals,
  BTT/Bulloak evidence, or host CI gate.

### `/plank-progress` report projection

`/plank-progress` integrates with `/gsd-progress` by using its context and
reporting model only:

1. require `.planning/ROADMAP.md` and `.planning/STATE.md`;
2. read GSD project/current-position context without mutating it;
3. discover `.planning/phases/**/PLANK-STATE.md`;
4. reconcile each ledger against repository and GitHub evidence;
5. report recent Plank work, current slice states, blockers, and the next child
   already fixed by the approved plan;
6. stop.

Never continue into `/gsd-progress` routing. Reject `--next`, `--do`,
`--auto`, `--force`, `--converge`, and model/reviewer routing flags. A bare
`next` request may be reported as plan order but must not dispatch anything.

The report must distinguish:

- **GSD project position** — read-only context from ROADMAP/STATE;
- **Plank slice position** — status from `PLANK-STATE.md`;
- **next approved child** — ordering already present in the approved plan;
- **recommended command** — informational text only, never an invocation.

#### Unrelated host roadmaps

When a tracked Plank child does not belong to the milestone represented by the
host ROADMAP:

- do not allocate or renumber a milestone phase;
- do not alter milestone counters, current phase, status, or next-phase fields;
- add/update a non-numbered `## Plank tracking (state-only)` appendix;
- store its ledger under
  `.planning/phases/PLANK/<issue>-<slice>/PLANK-STATE.md`;
- add a matching bounded summary in STATE without replacing GSD's current
  position.

This preserves GSD's milestone model while using its durable state tree.

#### Reconciliation after completion

A state-only commit that changes only `.planning/**` does not reopen a
completed implementation slice. Preserve the implementation commit and the CI
run that accepted it. Append a reconciliation entry naming the bookkeeping
commit when known. Any source, test, spec, manifest, workflow, or harness
change does reopen acceptance and requires a new `ci_pending` transition.
