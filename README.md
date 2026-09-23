# plank-tdd

Cursor / agent **skill** for Plank type development: Brady **type, define, refine**, `types.toml`, and **IO(T)** side-effect modules (description → `io` → `run` → `Outcome`).

## Install (Cursor)

```bash
mkdir -p ~/.cursor/skills ~/.cursor/commands
git clone https://github.com/JMSBPP/plank-tdd.git ~/.cursor/skills/plank-tdd
ln -sf ~/.cursor/skills/plank-tdd/commands/plank-type.md ~/.cursor/commands/plank-type.md
ln -sf ~/.cursor/skills/plank-tdd/commands/plank-define.md ~/.cursor/commands/plank-define.md
ln -sf ~/.cursor/skills/plank-tdd/commands/plank-refine.md ~/.cursor/commands/plank-refine.md
ln -sf ~/.cursor/skills/plank-tdd/commands/plank-ci-refactor.md ~/.cursor/commands/plank-ci-refactor.md
```

- **Skill:** agent may auto-load `SKILL.md` from description match.
- **Slash commands:** `/plank-type`, `/plank-define`, `/plank-refine`, `/plank-ci-refactor`.
- **Mission / CI policy:** [AGENTS.md](AGENTS.md) — CI failure → plan via
  `/request-refactor-plan` → slices → push → watch.

## Layout

| File | Role |
|------|------|
| [SKILL.md](SKILL.md) | Main workflow (required) |
| [AGENTS.md](AGENTS.md) | Mission statement + CI refactor policy |
| [REFERENCE.md](REFERENCE.md) | Brady, IO pattern, kinds, `types.toml`, Compose, BTT/Bulloak |
| [EXAMPLES.md](EXAMPLES.md) | LaTeX / holes / harness templates |
| [commands/](commands/) | Slash-command stubs |

## License

MIT — see [LICENSE](LICENSE)
