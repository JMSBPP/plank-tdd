---
name: plank-tdd
description: Type-driven Plank development — Brady type/define/refine, types.toml, IO(T), and BTT (.btt) + Bulloak-generated Foundry tests on every define. Use when writing or refining .plk types, implementing TokenFlow/IO/Xfer/View effects, or when the user runs /plank-type, /plank-define, or /plank-refine.
---

# Plank TDD (type → define → refine)

Hybrid of Brady **type, define, refine** and vertical behavioral TDD. Algebra leads. Side effects go through **IO(T)**, never void. **Heavy AskQuestions** — do not invent the algebra.

Slash commands select the phase: `/plank-type`, `/plank-define`, `/plank-refine`.

## Quick start

1. Ask working directory (default **`.spec/`**) and Plank type root (default **`src/types/`**)
2. **Explore std / host types** that already carry the needed semantics — do this before proposing a new type
3. Ask algebra of the next type (one question at a time) before any body
4. Write LaTeX on the type note; register **`types.toml`**
5. Phase: **type** (signatures / holes) → **define** (one behavior: `.btt` → Bulloak suite → fill hole) → **refine** (laws + Eff)
6. If the type has effects: follow the [IO pattern](REFERENCE.md#io-side-effect-modules)

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
| `/plank-define` | Define | Fill holes for **one** behavior; write `.btt`; Bulloak generates the suite |
| `/plank-refine` | Refine | Tighten types/laws; drop unused modules; set `refined = true` |

Do not jump to define/refine until the current phase’s gate is approved.

## Type file layout (Plank host)

1. `src/types/Foo.plk` — type module
2. `{working_dir}/types/Foo/Foo.btt` — BTT next to the type note
3. `test/types/Foo.t.sol` — **Bulloak-generated** suite (`scaffold` stdout → this path)
4. `test/harness/types/FooHarness.plk` — deploy via `deployPlank`

## Details

- Std-first + IO pattern, kinds, `types.toml`, BTT/Bulloak: [REFERENCE.md](REFERENCE.md)
- LaTeX / IO / `.btt` templates: [EXAMPLES.md](EXAMPLES.md)
- Sibling skills: `idris-tdd` (spec notes), `tdd` (vertical slices), type-driven-development (invariants before impl)
