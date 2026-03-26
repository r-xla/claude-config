# claude-config

Shared Claude Code configuration for the r-xla organization.

## Setup

Clone this repo as a sibling to your other r-xla repos:

```bash
git clone git@github.com:r-xla/claude-config.git
```

Your directory structure should look like:

```
<parent>/
├── claude-config/   # this repo
├── repo1/
├── repo2/
└── ...
```

## Usage

In each r-xla repo's `CLAUDE.md`, add this line at the top to import the shared configuration:

```markdown
@../claude-config/CLAUDE.md

# Repo-specific instructions
...
```

The relative path `../claude-config/CLAUDE.md` resolves from the importing file's location, so it works from any sibling repo.
