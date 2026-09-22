# Plank type

Invoke **plank-tdd** in the **type** phase. Follow the skill exactly.

1. Read `~/.cursor/skills/plank-tdd/SKILL.md` (plus `REFERENCE.md` / `EXAMPLES.md` as needed).
2. If the skill is missing:
   `git clone https://github.com/JMSBPP/plank-tdd.git ~/.cursor/skills/plank-tdd`
3. Mandatory AskQuestions — one at a time — starting with working directory (default `.spec/`) and algebra.
4. This phase may write: type notes, `types.toml`, Plank **signatures and holes**. No function bodies. No harness fill. No tests that require an implementation.
5. If Eff is non-empty, write the IO algebra (`io`, `run`, `Outcome`, View/Xfer) on the note — still no `xfer` body.
6. Any args after `/plank-type` are starting context — use them after the working-directory question.
