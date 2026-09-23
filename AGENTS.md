# plank-tdd — agent guide

## Mission statement

Continuously refactor Plank work so CI stays green with shorter wall-clock and higher
signal — every refactor slice is planned via `/request-refactor-plan`, executed under
`/plank-type` / `/plank-define` / `/plank-refine` (and `/plank-ci-refactor` when the
trigger is a CI failure), and verified only by the host project's existing push/gate
patterns (`compile.toml`, domain manifests, no local forge as the authority).

**Goals (ordered):**

1. **Reduce build time** — shrink the critical path in-repo *and* in workflows: lean
   `compile.toml` / domain scope, reuse pinned GHCR image (pull not rebuild), avoid
   duplicate Plank work, plus proactive workflow optimizations (caches, job shape,
   parallelization where the runner model allows). Prefer applying or trying changes
   toward that goal rather than only documenting them. When touching Actions YAML,
   cite current GitHub docs (workflow syntax, cache, reusable workflows, job
   summaries) so proposals stay grounded.
2. **Maximize effectiveness** — fail early on `push-build` / Spec compile; keep
   `develop-gate` as merge authority; never silence or skip gates to go green.
3. **Small batches** — tiny commits that leave the tree working; CI failure → agent
   loop → plan → fix → push → re-watch.

**GitHub Actions pointers** (refresh from docs when optimizing):

- Workflows: https://docs.github.com/en/actions/using-workflows
- Caching: https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows
- Reusable workflows: https://docs.github.com/en/actions/using-workflows/reusing-workflows
- Workflow commands / job summaries: https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions

## CI loop (with `/request-refactor-plan`)

When CI fails on a Plank-related step (or the maintainer asks for continuous CI
refactoring):

1. Capture the failing job/step and log excerpt (e.g. `gh run view --log-failed`).
2. Invoke `/request-refactor-plan` for a scoped plan issue (mission above).
3. Execute slices with `/plank-define` / `/plank-refine` (or `/plank-ci-refactor` as the
   orchestrator).
4. Push; treat GitHub Actions as the only verification — do not establish correctness
   with a local forge/plank run unless the host repo AGENTS.md says otherwise.

**Speculative CI optimizations:** always their own plan slice. “Try” = branch/PR + measure
wall-clock from Actions. Do not merge or silently change `develop-gate` required checks
without maintainer approve on that slice.

**Loop exit:** stop when the triggering run is green **and** open plan slices for that
failure are done or explicitly deferred in the issue. Do not hunt unrelated build-time
wins in the same invocation unless already listed on the plan.

## Artifacts (this skill)

Policy lives in this file. Implementation also ships:

- `commands/plank-ci-refactor.md` — orchestrator slash command
- CI loop section in `SKILL.md` (trigger → plan → slices → push → watch → exit)
- README install / symlink lines for the new command

## Testing Decisions (for `/request-refactor-plan` issues)

When the plan is opened from a CI-refactor trigger:

- Verification = named CI jobs/steps (`push-build` Spec compile / forge / `develop-gate`).
- Success = green run URL or log excerpt for those steps.
- Do **not** list “verify locally with forge/plank” unless the host repo `AGENTS.md`
  explicitly allows it.
- Still list harness / Foundry coverage when the fix is type *behavior*; CI remains the
  authority gate.
