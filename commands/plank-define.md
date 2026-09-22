# Plank define

Invoke **plank-tdd** in the **define** phase. Follow the skill exactly.

1. Read `~/.cursor/skills/plank-tdd/SKILL.md` (plus `REFERENCE.md` / `EXAMPLES.md` as needed).
2. If the skill is missing:
   `git clone https://github.com/JMSBPP/plank-tdd.git ~/.cursor/skills/plank-tdd`
3. Refuse to define if `/plank-type` (or an equivalent approved type note + `types.toml` + signatures) is not in place.
4. Ask which **one** behavior to lock. Fill only the holes that behavior needs.
5. **Every define writes behavioral semantics as a `.btt` file, then Bulloak generates the Foundry suite.** Do not hand-roll `*.t.sol` for a define slice. See [REFERENCE.md — BTT / Bulloak](../REFERENCE.md#btt--bulloak-every-define).
   1. Write `{working_dir}/types/<Type>/<Type>.btt` (next to the type note) — one tree for this behavior (success + invalid branches from the type note). Point the type note and `src/types/<Type>.plk` at that `.btt`.
   2. Generate into `test/**`, not into `.spec/`: `bulloak scaffold {working_dir}/types/<Type>/<Type>.btt > test/types/<Type>.t.sol` (do **not** use `-w`; that writes beside the `.btt`). If the pin only accepts `.tree`, keep the same basename and the same contents.
   3. Wire `PlankTestBase` + the public harness ABI into `test/types/<Type>.t.sol`. Keep Bulloak’s function names.
   4. RED (holes empty) → fill the hole → GREEN. Public ABI only.
6. Side effects: implement `io` wrap + `run`/`run_io` for that behavior; `xfer`/`view` only for selectors on the Eff row.
7. Add the harness `.plk` if new. Add every new `.plk` to the domain `compile.toml`.
8. Any args after `/plank-define` are the behavior to define — confirm algebra still matches, then implement that slice only.
