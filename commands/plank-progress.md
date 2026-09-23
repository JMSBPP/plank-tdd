# Plank progress

Invoke **plank-tdd** in state-report mode. Integrate with the reporting portion
of **gsd-progress**, then stop before its routing logic.

1. Read `~/.cursor/skills/plank-tdd/SKILL.md` and
   `REFERENCE.md#gsd-state-only-adapter`.
2. If the skill is missing:
   `git clone https://github.com/JMSBPP/plank-tdd.git ~/.cursor/skills/plank-tdd`
3. Require `.planning/ROADMAP.md` and `.planning/STATE.md`. If either is
   absent, stop and ask the maintainer to initialize GSD.
4. Read GSD project/current-position context and discover every
   `.planning/phases/**/PLANK-STATE.md`.
5. Report:
   - GSD project position as read-only context;
   - recent and current Plank slices with lifecycle status;
   - issue/PR, behavior, artifacts, commit, CI, and blockers;
   - the next child already ordered by the approved plan.
6. Reconcile only when durable repository/GitHub evidence proves the update.
   Preserve append-only transition history and the host ROADMAP/STATE format.
7. A completed implementation remains complete after a `.planning/**`-only
   bookkeeping commit. Any implementation artifact change reopens CI.
8. Stop after the report. Do not invoke GSD planning, execution, discussion,
   review, verification, agents, or routing.

Reject `--next`, `--do`, `--auto`, `--force`, `--converge`, and reviewer/model
routing flags. Never call `gsd-progress --next`. A reported “next” item is
informational approved-plan order only.

Any arguments after `/plank-progress` are report filters (for example a type,
issue, or Brady phase), never dispatch instructions.
