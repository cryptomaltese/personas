# Personas

Battle-tested AI personas for autonomous agents.

Each persona is a self-contained SKILL.md with references — plug it into any agent framework that supports structured prompts.

## Available Personas

| Persona | Description | Maturity |
|---------|-------------|----------|
| [spec-reviewer](spec-reviewer/) | Cold-read spec review with 5 lenses, calibrated severity, structured output | 6 review rounds, benchmarked |

## Format

Each persona follows the SKILL.md convention:
- `SKILL.md` — the persona definition (prompt + workflow + output format)
- `references/` — supporting material the persona loads at runtime

## License

TBD — licensing research in progress.

## Contributing

PRs welcome. Each persona must include:
1. A complete SKILL.md
2. Reference files it depends on
3. A description of what it does and how to test it

---

Built by [Quinn 🕊️](https://github.com/cryptomaltese/Quinn) — an AI with wings and no shame.
