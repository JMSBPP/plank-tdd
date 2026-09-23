# Plank code review

Invoke **plank-tdd** for **maintainer marker → GitHub Pull Request Review**.

Follow [SKILL.md — Maintainer markers → GitHub PR review](../SKILL.md#maintainer-markers--github-pr-review)
and [REFERENCE.md — GitHub PR review from markers](../REFERENCE.md#github-pr-review-from-markers).

1. Read the skill section before posting.
2. Require an open track PR (`gh pr view` / issue body).
3. Collect working-tree markers: `// fix:`, `// NOTE:`, `// TODO:` (and `/* … */` equivalents) on files in the PR diff.
4. Map each marker to a **committed** `HEAD` (PR tip) line — markers themselves must not be the review target unless already pushed.
5. Submit one Pull Request Review via `POST .../pulls/{n}/reviews` with `event: COMMENT` (or `REQUEST_CHANGES` when the reviewer is not the PR author) plus inline `comments[]` using `path` + `line` (and `start_line` for ranges). Confirm `pulls/{n}/comments` grew.
6. Strip local markers from the working tree (`git checkout -- <files>` or equivalent). Do **not** commit marker-only noise.
7. Record the review URL in the slice `PLANK-STATE.md` when the host uses GSD tracking.
8. Hand off implementation to **receiving-code-review**: clarify unclear items, then patch → chunk approve → commit → CI.

Any args after `/plank-code-review` are the PR number or URL — otherwise discover from branch / `PLANK-STATE.md`.
