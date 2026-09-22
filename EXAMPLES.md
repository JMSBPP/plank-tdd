# EXAMPLES — plank-tdd

## LaTeX on a type note

```markdown
# [TYPE:: IO](MAIN_REF# MODEL)

\[
\begin{aligned}
\mathrm{IO} &:: \mathrm{type} \to \mathrm{type} \\
\mathrm{io} &:: T \to \mathrm{IO}(T) \\
\mathrm{run} &:: \mathrm{IO}(T) \to \mathrm{Option}(\cdot) \\
\mathrm{View} &= \mathrm{staticcall} \\
\mathrm{Xfer} &= \mathrm{call} \\
\mathrm{None} &= \mathrm{revert},\ \mathrm{Some} = \mathrm{success}
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

## Define phase — `.btt` then Bulloak

Every define: write the tree, scaffold, then fill the hole. Example for `intro` (success + one invalid branch):

```
FooTest
├── when L is zero
│   └── it should revert ZeroLiquidity
└── when L and sqrt_p are nonzero and σ fits u88
    └── it should return tick Tick(√p) and σ
```

```bash
bulloak scaffold .spec/REALIZED_VOLATILITY.spec/types/Foo/Foo.btt \
  > test/types/Foo.t.sol
```

Do not pass `-w` (that would write `Foo.t.sol` under `.spec/`). Wire `PlankTestBase` + harness ABI into `test/types/Foo.t.sol`. Keep Bulloak’s function names.

## Define phase — one behavior (Plank hole)

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
2. “Std/host types that already mean this? (search `std/` first)”
3. “Domain key for `types.toml`?”
4. “Carriers of this algebra?”
5. “Operations and laws?”
6. “Kind: plain / generic / dependent / indexed?”
7. “Eff row? If none, this is a pure type — no IO.”
8. “Which single behavior should the first `.btt` lock?”

Only after (2) and (4)–(7): create notes and holes.

## Test name

Leaves in the `.btt` name the behavior (`it should …`). Bulloak generates the Solidity identifiers. Do not invent `test__beh__*` names that the tree does not contain.
