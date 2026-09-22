# Plank define

Invoke **plank-tdd** in the **define** phase. Follow the skill exactly.

1. Read `~/.cursor/skills/plank-tdd/SKILL.md` (plus `REFERENCE.md` / `EXAMPLES.md` as needed).
2. If the skill is missing:
   `git clone https://github.com/JMSBPP/plank-tdd.git ~/.cursor/skills/plank-tdd`
3. Refuse to define if `/plank-type` (or an equivalent approved type note + `types.toml` + signatures) is not in place.
4. Ask which **one** behavior to lock. Fill only the holes that behavior needs.
5. Side effects: implement `io` wrap + `run`/`run_io` for that behavior; `xfer`/`view` only for selectors on the Eff row.
6. Harness + one Foundry test slice (RED then GREEN). Public ABI only.
7. Add new `.plk` paths to the domain `compile.toml`.
8. Any args after `/plank-define` are the behavior to define — confirm algebra still matches, then implement that slice only.
