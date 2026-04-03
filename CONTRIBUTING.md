# Contributing to r-xla Claude Code Configuration

## Setup

Install the skill marketplace in Claude Code:

1. Run `/plugin` in any Claude Code session.
2. Add this repository as a marketplace.

Skills from this repo will then be available as slash commands.

## Repo Structure

```
claude-config/
├── .claude-plugin/
│   └── marketplace.json   # Plugin registry (lists all skills)
├── skills/
│   └── r-xla/
│       ├── release/
│       │   └── SKILL.md
│       └── release-package/
│           └── SKILL.md
├── CLAUDE.md              # Shared ecosystem instructions
└── CONTRIBUTING.md
```

- **`.claude-plugin/marketplace.json`** -- Registers this repo as a Claude Code plugin marketplace and lists all available skills.
- **`skills/r-xla/`** -- Skill definitions. Each subdirectory contains a `SKILL.md` with YAML frontmatter.
- **`CLAUDE.md`** -- Shared instructions loaded when this marketplace is installed. Contains the ecosystem overview, package descriptions, and common development commands.

## Adding a New Skill

1. Create a new directory under `skills/r-xla/<skill-name>/`.
2. Add a `SKILL.md` with YAML frontmatter (`name`, `description`, `user_invocable`, `tools`).
3. Register the skill path in `.claude-plugin/marketplace.json` under the `skills` array.

## Structuring a Repo's `CLAUDE.md`

Each r-xla package has its own `CLAUDE.md` with package-specific content. The shared ecosystem context (package relationships, common commands) is provided by this marketplace's `CLAUDE.md`.

**Guidelines:**

- **Only add what's specific to this package.** The shared config already covers the ecosystem overview and standard R development commands.
- **Document non-obvious conventions.** For example, anvil documents its 1-based vs 0-based indexing boundary with stablehlo.
- **Keep it concise.** `CLAUDE.md` is loaded into every conversation. Verbose instructions waste context.

## Making Changes

When editing `CLAUDE.md`, keep in mind that the content is loaded into every r-xla repo's context. Changes here affect Claude Code's behavior across the entire organization.
