# Plank refine

Invoke **plank-tdd** in the **refine** phase. Follow the skill exactly.

1. Read `~/.cursor/skills/plank-tdd/SKILL.md` (plus `REFERENCE.md` / `EXAMPLES.md` as needed).
2. If the skill is missing:
   `git clone https://github.com/JMSBPP/plank-tdd.git ~/.cursor/skills/plank-tdd`
3. Refuse to refine if define has no approved slice yet.
4. Tighten types and laws together. Update the LaTeX note and `types.toml` (`refined = true`, amend `laws`).
5. Side-effect modules: drop every Compose/EVM import not named by Eff (Transfer + `balanceOf` is enough when Eff is Xfer + ERC20View). Seed extra storage from tests.
6. Do not add new behaviors here — that is another `/plank-define` slice.
7. Any args after `/plank-refine` are the refinement target — apply them after confirming current laws.
