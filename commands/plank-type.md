# Plank type

Invoke **plank-tdd** in the **type** phase. Follow the skill exactly.

1. Read `~/.cursor/skills/plank-tdd/SKILL.md` (plus `REFERENCE.md` / `EXAMPLES.md` as needed).
2. If the skill is missing:
   `git clone https://github.com/JMSBPP/plank-tdd.git ~/.cursor/skills/plank-tdd`
3. Mandatory AskQuestions — one at a time — starting with working directory (default `.spec/`).
4. If this is a GSD-tracked child of an approved plan, require existing
   `.planning/ROADMAP.md` and `.planning/STATE.md`; synchronize only this
   child phase through the lifecycle in
   [REFERENCE.md](../REFERENCE.md#gsd-state-only-adapter). Stop if the
   prerequisites are absent. Do not invoke GSD planning, execution, review,
   verification, discussion, agents, or next-work routing.
5. **Before any new type:** search `lib/plank-monorepo/std/` and host `src/types/` for types that already have the semantics. Present candidates (reuse vs reject). Do not invent a parallel `Outcome`/`Result` if `std::option::Option` fits.
6. Then algebra. This phase may write: type notes, `types.toml`, Plank **signatures and holes**. No function bodies. No harness fill. No tests that require an implementation.
7. If Eff is non-empty, write the IO algebra (`io`, `run`, outcome type, View/Xfer) on the note — still no `xfer` body. The outcome type is std unless the std search rejected every candidate.
8. Any args after `/plank-type` are starting context — use them after the working-directory question.
