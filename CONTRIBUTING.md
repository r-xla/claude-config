# Contributing to r-xla Claude Code Configuration

## Setup

Clone this repo as a sibling to your other r-xla repos:

```bash
cd /path/to/r-xla
git clone git@github.com:r-xla/claude-config.git
```

Your directory structure should look like:

```
r-xla/
├── claude-config/   # this repo
├── anvil/
├── stablehlo/
├── pjrt/
├── tengen/
├── xlamisc/
└── ...
```

## How It Works

Each r-xla repo's `CLAUDE.md` imports the shared configuration using the `@` directive, which inlines the referenced file's content:

```markdown
@../claude-config/AGENTS.md
```

The relative path resolves from the importing file's location. Since all repos are siblings, `../claude-config/` always points here.

## Structuring a Repo's `CLAUDE.md`

A repo's `CLAUDE.md` should follow this structure:

```markdown
@../claude-config/AGENTS.md

## Package Overview

Brief description of what this package does.

## Development Commands

Package-specific build, test, and check commands
(if they differ from the standard devtools workflow).

## Development Practices

Package-specific conventions, patterns, and gotchas.
```

**Guidelines:**

- **Start with the `@` import.** This gives Claude Code the ecosystem context (package relationships, shared conventions, common commands).
- **Only add what's specific to this package.** The shared config already covers the general ecosystem overview and standard R development commands. Don't repeat them.
- **Document non-obvious conventions.** For example, anvil documents its 1-based vs 0-based indexing boundary with stablehlo, and how to add new primitives.
- **Keep it concise.** `CLAUDE.md` is loaded into every conversation. Verbose instructions waste context.

## Repo Structure

```
claude-config/
├── AGENTS.md              # Shared instructions imported by all repos
├── .claude/
│   └── settings.local.json  # Shared Claude Code settings
└── skills/                # Shared skills available to all repos
    └── r-xla-release/
        └── SKILL.md
```

- **`AGENTS.md`** -- The main shared instructions file. Contains the ecosystem overview, package descriptions, directory layout conventions, and common development commands.
- **`.claude/settings.local.json`** -- Shared settings (permissions, etc.) that apply when Claude Code runs in any repo that imports this config.
- **`skills/`** -- Reusable Claude Code skills (slash commands) shared across repos.

## Making Changes

When editing `AGENTS.md`, keep in mind that the content is injected into every r-xla repo's context. Changes here affect Claude Code's behavior across the entire organization.
