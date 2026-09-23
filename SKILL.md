---
name: plank-tdd
description: Type-driven Plank development — Brady type/define/refine, types.toml, IO(T), BTT (.btt) + Bulloak-generated Foundry tests on every define, maintainer //fix|NOTE|TODO markers promoted to GitHub Pull Request Reviews via /plank-code-review, state-only GSD progress reporting via /plank-progress, and CI-failure continuous refactor via /plank-ci-refactor + /request-refactor-plan. Use when writing or refining .plk types, implementing TokenFlow/IO/Xfer/View effects, checking Plank slice progress, promoting inline review markers to GitHub, fixing Plank-related CI failures, or when the user runs /plank-type, /plank-define, /plank-refine, /plank-code-review, /plank-progress, or /plank-ci-refactor.
---

# Plank TDD (type → define → refine)

Hybrid of Brady **type, define, refine** and vertical behavioral TDD. Algebra leads. Side effects go through **IO(T)**, never void. **Heavy AskQuestions** — do not invent the algebra.

Slash commands select the phase: `/plank-type`, `/plank-define`, `/plank-refine`.
Maintainer review markers → `/plank-code-review` (GitHub PR Review).
CI failure → `/plank-ci-refactor` (mission/policy: [AGENTS.md](AGENTS.md)).

## Maintainer markers → GitHub PR review

Working-tree `// fix:`, `// NOTE:`, `// TODO:` (on harness/tests/impl) are **not** the review of
record. Promote them to a GitHub **Pull Request Review** so the track PR has a real
`reviews[]` entry with inline threads — that is what counts as code review on GitHub.

When the maintainer says the markers are a code review (or runs `/plank-code-review`):

1. Do **not** treat markers as already-approved patches; do **not** commit them as permanent source comments.
2. Follow [REFERENCE.md — GitHub PR review from markers](REFERENCE.md#github-pr-review-from-markers): map each marker to a line on the PR tip commit, submit one review via the Reviews API, verify inline comments exist, strip local markers.
3. Then run **receiving-code-review** on that GitHub review (clarify → implement → chunk approve → CI).

## GSD state adapter — tracking only

When an **approved plan** is decomposed into child slices for `type`, `define`, or
`refine`, mirror each child slice as one GSD phase. GSD is a durable state ledger
only; Brady + Plank TDD remain the workflow authority.

Before work on a tracked slice:

1. Require existing `.planning/ROADMAP.md` and `.planning/STATE.md`. If either is
   absent, stop and ask the maintainer to initialize GSD state.
2. Locate or register one phase for the approved child slice, tagged
   `plank_phase: type|define|refine`.
3. Create/update the phase-local `PLANK-STATE.md` described in
   [REFERENCE.md](REFERENCE.md#gsd-state-only-adapter).
4. Synchronize status at material transitions:
   `pending → in_progress → code_approved → committed → ci_pending → complete|blocked`.

Allowed GSD surface:

- `.planning/ROADMAP.md`
- `.planning/STATE.md`
- `.planning/phases/<phase>/PLANK-STATE.md`

Write targeted state updates directly while preserving the host files' existing
format. Do **not** invoke GSD workflows or agents.

Hard boundary — the adapter MUST NOT:

- invoke GSD plan, execute, review, verify, discuss, or progress-routing commands;
- create GSD `PLAN.md`, `SUMMARY.md`, research, context, checkpoint, or review artifacts;
- let GSD select, reorder, approve, execute, or verify a Plank slice;
- replace the host plan, AskQuestion gates, BTT/Bulloak flow, code-chunk approval,
  or host CI authority.

If no approved plan is decomposed into child slices, run Plank TDD normally without
creating GSD tracking state.

### `/plank-progress`

`/plank-progress` reuses the **reporting** portion of `/gsd-progress` over the
allowed state files, then stops. It reports GSD project context plus the Plank
slice ledger and may reconcile summaries from durable issue, commit, PR, and CI
evidence.

It never invokes `/gsd-progress --next`, `/gsd-progress --do`, or any GSD
route. It cannot choose or execute the next action. "Next" in its output means
the next child already ordered by the approved plan, not a dispatch decision.

A bookkeeping-only reconciliation commit does not reopen a completed
implementation slice when it changes only `.planning/**`; retain the accepted
implementation commit and host-authoritative CI URL as completion evidence.

## Quick start

1. If the approved plan has child slices, initialize/sync the state-only GSD phase.
2. Ask working directory (default **`.spec/`**) and Plank type root (default **`src/types/`**)
3. **Explore std / host types** that already carry the needed semantics — do this before proposing a new type
4. Ask algebra of the next type (one question at a time) before any body
5. Write LaTeX on the type note; register **`types.toml`**
6. Phase: **type** (signatures / holes) → **define** (one behavior: `.btt` → Bulloak suite → fill hole → **update type note**) → **refine** (laws + Eff)
7. If the type has effects: follow the [IO pattern](REFERENCE.md#io-side-effect-modules)
8. On **every define**, extend the type **`.md`** with the [math heading notation](REFERENCE.md#type-note-headings-math-notation) for that operation (signature subtitle + `aligned` laws). The note is the living algebra; do not leave define code-only.

## Iron laws

- No function bodies before the **algebra** is agreed
- **Std first:** search `lib/plank-monorepo/std/` and host `src/types/` for a type that already has the semantics. Record candidates. A new type is allowed only if none fit
- Side-effecting work is `io` then `run` → an **outcome** type (prefer std); never a void command
- Import **only** the EVM/Compose modules the `Eff` row needs
- Holes first; never horizontal “all tests then all code”
- **Every define** writes a `.btt` (BTT) for that one behavior; Bulloak generates the suite. Do not hand-roll define-phase `*.t.sol`
- Tests use the **public harness ABI** only (fill the generated file; keep Bulloak names)
- Add every new `.plk` to the domain `compile.toml`
- Follow the host repo’s validation rule (CI vs local); do not invent a toolchain
- GSD records state only; it never supplies Plank planning, execution, review, or verification semantics

## AskQuestions (mandatory)

Ask **one** question at a time. Cover at least:

- Working directory (default `.spec/`) and Plank root (default `src/types/`)
- Domain / `types.toml` section
- Algebra: carriers, operations, laws, invalid states
- **Std candidates:** which existing std/host types already mean this? Why reuse or reject each?
- Type kind: plain | generic | dependent | indexed | other
- Does it have **Eff**? If yes: View / Xfer / other, and which modules
- Next single behavior to lock

## Commands

| Command | Phase | Allowed |
|---------|--------|---------|
| `/plank-type` | Type | Std search, note, `types.toml`, signatures, holes. No bodies. |
| `/plank-define` | Define | Fill holes for **one** behavior; write `.btt`; Bulloak suite; **update type note** (math headings + laws for that op) |
| `/plank-refine` | Refine | Tighten types/laws; drop unused modules; set `refined = true` |
| `/plank-progress` | State report | Report/reconcile approved Plank child state using GSD files; never route or execute |
| `/plank-code-review` | Review promote | Working-tree `fix`/`NOTE`/`TODO` markers → GitHub Pull Request Review on the track PR |
| `/plank-ci-refactor` | CI loop | Explicit CI failure only → `/request-refactor-plan` → slices → push → watch |

Do not jump to define/refine until the current phase’s gate is approved.

## CI loop (`/plank-ci-refactor`)

Mission and full policy: [AGENTS.md](AGENTS.md). Summary:

1. **Trigger only** on an explicit CI failure / pasted failed log (not routine type work).
2. Capture failing job/step (`gh run view --log-failed` or paste). Failing-step owns;
   if cross-cutting with Idris, keep one plan issue and may call `idris-tdd` for sibling
   slices (or hand off to `/idris-ci-refactor` when the primary failure is Idris).
3. **Always** open/update a plan via `/request-refactor-plan` before code. Testing
   Decisions = named CI jobs/steps; success = green run URL; no local forge as
   authority unless host `AGENTS.md` allows it.
4. Execute slices with `/plank-define` / `/plank-refine` (and `/plank-type` when needed).
5. Workflow YAML / cache / image optimizations are in scope: cite GitHub Actions docs,
   try on a branch/PR, measure wall-clock; never silently change `develop-gate`
   required checks. Speculative opts are their own plan slices.
6. **Exit** when the triggering run is green and plan slices for that failure are done
   or deferred — no drive-by build-time hunting beyond the plan.

## Type file layout (Plank host)

1. `src/types/Foo.plk` — type module
2. `{working_dir}/types/Foo/Foo.btt` — BTT next to the type note
3. Type note + `Foo.plk` **point at that `.btt`** (behavior source)
4. `test/types/Foo.t.sol` — **Bulloak-generated** suite (`scaffold` stdout → this path)
5. `test/harness/types/FooHarness.plk` — deploy via `deployPlank`

## Details

- Std-first + IO pattern, kinds, `types.toml`, BTT/Bulloak: [REFERENCE.md](REFERENCE.md)
- GitHub PR review from markers: [REFERENCE.md](REFERENCE.md#github-pr-review-from-markers)
- LaTeX / IO / `.btt` templates: [EXAMPLES.md](EXAMPLES.md)
- Mission / CI policy: [AGENTS.md](AGENTS.md)
- Sibling skills: `idris-tdd` (spec notes + `/idris-ci-refactor`), `request-refactor-plan`, `tdd` (vertical slices), `receiving-code-review`, type-driven-development (invariants before impl)
