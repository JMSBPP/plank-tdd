# EXAMPLES — plank-tdd

## LaTeX on a type note

```markdown
# [TYPE:: IO](MAIN_REF# MODEL)

\[
\begin{aligned}
\mathrm{IO} &:: \mathrm{type} \to \mathrm{type} \\
\mathrm{io} &:: T \to \mathrm{IO}(T) \\
\mathrm{run} &:: \mathrm{IO}(T) \to \mathrm{Outcome} \\
\mathrm{View} &= \mathrm{staticcall} \\
\mathrm{Xfer} &= \mathrm{call} \\
\mathrm{Outcome} &= \mathrm{success} \mid \mathrm{revert}
\end{aligned}
\]
```

## Type phase — signatures only (Plank)

```plank
const IO = fn (comptime T: type) type {
    struct { inner: T }
};

const io = fn (tf: TokenFlow()) IO(TokenFlow()) {
    IO(TokenFlow()) { inner: tf }
};

/// hole: run_io not defined this phase
```

## Define phase — one behavior

Behavior: `run(io(amt, +))` transfers `from → to`.

Harness exports `run`; Plank executor is `run_io` to avoid the name clash.

```plank
const run_io = fn (m: IO(TokenFlow()), token: addr, from: addr, to: addr) bool {
    if m.inner.dir == 0 {
        xfer(token, from, to, m.inner.amount)
    } else {
        xfer(token, to, from, m.inner.amount)
    }
};
```

`xfer` is `@evm_call` of `transferFrom` (selector `0x23b872dd`) — only because Eff says Xfer.

## Refine phase — drop unused modules

Fixture imports **Transfer Mod** only; `balanceOf` reads the same ERC-8042 slot. Tests seed balance/allowance with `vm.store`. Do not import Approve or Mint.

## Session skeleton (AskQuestions)

1. “Working directory? `[.spec/]` — Plank root? `[src/types/]`”
2. “Domain key for `types.toml`?”
3. “Carriers of this algebra?”
4. “Operations and laws?”
5. “Kind: plain / generic / dependent / indexed?”
6. “Eff row? If none, this is a pure type — no IO.”
7. “Which single behavior should the first harness test lock?”

Only after (3)–(6): create notes and holes.

## Test name

Name tests after **behavior**, not after hole names:

`test__beh__run_positive_transfers_from_to`
