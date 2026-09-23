---
name: plank-tdd
description: Type-driven Plank development — Brady type/define/refine, types.toml, IO(T), BTT (.btt) + Bulloak-generated Foundry tests on every define, and CI-failure continuous refactor via /plank-ci-refactor + /request-refactor-plan. Use when writing or refining .plk types, implementing TokenFlow/IO/Xfer/View effects, fixing Plank-related CI failures, or when the user runs /plank-type, /plank-define, /plank-refine, or /plank-ci-refactor.
---

# Plank TDD (type → define → refine)

Hybrid of Brady **type, define, refine** and vertical behavioral TDD. Algebra leads. Side effects go through **IO(T)**, never void. **Heavy AskQuestions** — do not invent the algebra.

Slash commands select the phase: `/plank-type`, `/plank-define`, `/plank-refine`.
CI failure → `/plank-ci-refactor` (mission/policy: [AGENTS.md](AGENTS.md)).

## Quick start

1. Ask working directory (default **`.spec/`**) and Plank type root (default **`src/types/`**)
2. **Explore std / host types** that already carry the needed semantics — do this before proposing a new type
3. Ask algebra of the next type (one question at a time) before any body
4. Write LaTeX on the type note; register **`types.toml`**
5. Phase: **type** (signatures / holes) → **define** (one behavior: `.btt` → Bulloak suite → fill hole → **update type note**) → **refine** (laws + Eff)
6. If the type has effects: follow the [IO pattern](REFERENCE.md#io-side-effect-modules)
7. On **every define**, extend the type **`.md`** with the [math heading notation](REFERENCE.md#type-note-headings-math-notation) for that operation (signature subtitle + `aligned` laws). The note is the living algebra; do not leave define code-only.

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
- LaTeX / IO / `.btt` templates: [EXAMPLES.md](EXAMPLES.md)
- Mission / CI policy: [AGENTS.md](AGENTS.md)
- Sibling skills: `idris-tdd` (spec notes + `/idris-ci-refactor`), `request-refactor-plan`, `tdd` (vertical slices), type-driven-development (invariants before impl)
