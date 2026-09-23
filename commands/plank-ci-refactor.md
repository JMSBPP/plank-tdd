# Plank CI refactor

Orchestrate **continuous CI refactoring** for Plank under **plank-tdd**. Policy:
`~/.cursor/skills/plank-tdd/AGENTS.md` (mission: reduce build time, maximize
effectiveness). Do not invent a parallel CI policy.

**Trigger only:** an explicit CI failure, or a maintainer-pasted failed
`push-build` / Spec compile / forge log. Ordinary type work stays
`/plank-type` / `/plank-define` / `/plank-refine`.

1. Read `~/.cursor/skills/plank-tdd/SKILL.md` (CI loop section) and `AGENTS.md`.
2. If the skill is missing:
   `git clone https://github.com/JMSBPP/plank-tdd.git ~/.cursor/skills/plank-tdd`
3. Capture failing job/step + log (`gh run view --log-failed` or paste). If the
   failure is primarily Idris, hand off to `/idris-ci-refactor` (or keep ownership
   here and call `idris-tdd` for sibling slices — one plan issue, failing-step owns).
4. **Always** invoke `/request-refactor-plan` first (mission from `AGENTS.md`). Do
   not jump to code from a red log. Specialize **Testing Decisions**: named CI
   jobs/steps; success = green run URL; no local forge/plank as authority unless
   host `AGENTS.md` allows it.
5. Execute approved plan slices with `/plank-define` / `/plank-refine` (and
   `/plank-type` only when the plan requires new signatures). Speculative workflow
   YAML / cache / image optimizations are their own plan slices: try = branch/PR +
   measure wall-clock from Actions; cite GitHub Actions docs; never silently change
   `develop-gate` required checks.
   If the approved plan declares GSD tracking, the selected Plank command may
   synchronize its own `PLANK-STATE.md`; this command must not use that adapter
   for GSD planning, execution, review, verification, agents, or next-work routing.
6. Push; watch CI. Exit when the triggering failure is green **and** open plan
   slices for that failure are done or deferred on the issue.
7. Any args after `/plank-ci-refactor` are failure context (run URL, log excerpt) —
   incorporate them after confirming the failing step.
