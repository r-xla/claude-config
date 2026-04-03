# claude-config

A Claude Code skill marketplace for the r-xla ecosystem.

## Installation

Run `/plugin` in Claude Code and add this repository as a marketplace. Skills will then be available as slash commands in any project.

## Available Skills

| Skill | Command | Description |
|-------|---------|-------------|
| [r-xla-release-package](skills/r-xla/release-package/SKILL.md) | `/r-xla-release-package` | Release a single R package (from inside the package directory) |
| [r-xla-prepare-pr](skills/r-xla/prepare-pr/SKILL.md) | `/r-xla-prepare-pr` | Run all checks and prepare an R package PR (from inside the package directory) |

## Adding a New Skill

1. Create `skills/r-xla/<skill-name>/SKILL.md` with YAML frontmatter.
2. Register it in `.claude-plugin/marketplace.json` under the `skills` array.

See the [Anthropic guide on writing skills](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf) for best practices.
