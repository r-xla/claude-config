---
name: r-xla-release
description: >
  Prepare a release of a single R package. Use when the user says "release this
  package", "bump version", "cut a release", or wants to publish a new version
  of the current R package. Must be run from inside the package directory
  (contains a DESCRIPTION file).
user_invocable: true
tools: Read, Edit, Glob, Grep, Bash, Write, AskUserQuestion, Skill
---

# Single R Package Release

Your role is to prepare a release of a single R package.

## Workflow

### Step 1: Validate

1. Check that the working directory contains a `DESCRIPTION` file. If not,
   inform the user this must be run from an R package root directory and stop.
2. Check that `gh` is available and authenticated:
   ```bash
   gh repo view --json url
   ```
   If not, suggest `gh auth login` and stop.

### Step 2: Show Current State

Read `DESCRIPTION` and extract the `Package:` and `Version:` fields.
Read the first section of `NEWS.md` (everything under the first `#` heading
up to the next `#` heading) to understand what changes are included.

Display to the user:

```
Package:  <name>
Version:  <current version>
NEWS.md:  <first heading>
```

Then show the NEWS.md content for the upcoming release so the user can review it.

### Step 3: Check for Remotes

Check whether `DESCRIPTION` contains a `Remotes:` field. If it does, the package
cannot be released to CRAN with remote dependencies.

For each remote listed, look up the latest release version on CRAN (or the
package's repository if not on CRAN) and suggest replacing the remote dependency
with a minimum version requirement.

Display the remotes to the user:

```
Remotes found in DESCRIPTION:
- <owner>/<repo>
- <owner>/<repo2>
```

Use `AskUserQuestion` to ask which minimum version to require for each package.
Suggest the latest released version of each package as the default.

Once the user confirms, remove the `Remotes:` field from `DESCRIPTION` and update
the corresponding entries in `Imports:` or `Depends:` to include the agreed
version requirements (e.g., `package (>= x.y.z)`).

If there are no `Remotes:` entries, skip this step silently.

### Step 4: Ask Release Type

Suggest an appropriate release type based on the NEWS.md content:

- If NEWS mentions "breaking changes", "BREAKING", or similar → suggest **Major**
- If NEWS mentions only "bug fixes", "fixes", "patch" with no new features → suggest **Patch**
- Otherwise (new features, improvements) → suggest **Minor**

Use `AskUserQuestion` to confirm:

Question: "What type of release for `<package>` (currently `<version>`)?"
Header: "Release type"
Options (with recommended option first, marked "(Recommended)"):
- Major (X.0.0)
- Minor (x.X.0)
- Patch (x.x.X)

Calculate the new version:
- Strip `.9xxx` development suffix first
- Then bump according to release type
- Examples:
  - `0.1.0.9000` + Patch → `0.1.1`
  - `0.1.0.9000` + Minor → `0.2.0`
  - `0.1.1.9000` + Patch → `0.1.2`
  - `1.2.3.9000` + Major → `2.0.0`

Display: "Preparing release: `<package>` `<current>` → `<new version>`"

### Step 5: Check Git History for Missing NEWS.md Entries

Find the last release tag and list commits since then:

```bash
LAST_TAG=$(git tag --sort=-version:refname | head -1)
if [ -n "$LAST_TAG" ]; then
  git log "${LAST_TAG}..HEAD" --oneline --no-merges
else
  git log --oneline --no-merges
fi
```

Compare commits against NEWS.md entries. Look for commits describing user-facing
changes (new features, bug fixes, behaviour changes) without a corresponding
NEWS entry.

Ignore purely internal commits: CI changes, linting, dependency bumps without
user-visible effect, merge commits.

If there are likely missing entries, summarise them for the user and ask whether
to add them to NEWS.md before proceeding.

### Step 6: Create Release Branch

```bash
git checkout main
git pull origin main
git checkout -b release/<version>
```

If a `release/*` branch already exists, ask the user how to proceed using
`AskUserQuestion`:
- Delete it and create a fresh one
- Abort

### Step 7: Update DESCRIPTION — Version

Replace the `Version:` field with the new version (e.g., `0.1.0.9000` → `0.2.0`).
Use the Edit tool.

### Step 8: Update NEWS.md

Replace the development version header with the release version.

The header format is: `# <package> (development version)`
Replace with: `# <package> <version>`

For example:
- `# anvil (development version)` → `# anvil 0.2.0`

Use the Edit tool.

### Step 9: Commit and Push

```bash
git add DESCRIPTION NEWS.md
git commit -m "$(cat <<'EOF'
release: <package> <version>
EOF
)"
git push -u origin release/<version>
```

### Step 10: Create PR

Use the `/pr-create` skill to create a pull request for the release branch.

The PR title should be: `release: <package> <version>`

The PR body should include the NEWS.md entries for this release.

Wait for CI to pass. The `/pr-create` skill handles CI monitoring and fixing.

### Step 11: Merge and Tag

Once CI passes:

1. Merge the PR using squash merge:

```bash
gh pr merge --squash --delete-branch
```

2. Check out main and pull the merged changes:

```bash
git checkout main
git pull origin main
```

3. Tag the release commit and push the tag:

```bash
git tag v<version>
git push origin v<version>
```

### Step 12: Bump to Development Version

1. Update `DESCRIPTION`: append `.9000` to the version (e.g., `0.2.0` → `0.2.0.9000`).

2. Update `NEWS.md`: add a new section at the top:

```markdown
# <package> (development version)

```

3. Commit and push:

```bash
git add DESCRIPTION NEWS.md
git commit -m "$(cat <<'EOF'
release: <package> <version>.9000
EOF
)"
git push origin main
```

### Step 13: Create GitHub Release

Create a GitHub release for the tag pushed in Step 11.

Extract the NEWS.md section for this release — everything under the
`# <package> <version>` heading up to the next `#` heading — and use it as the
release notes body.

```bash
gh release create v<version> \
  --title "<package> <version>" \
  --notes "$(cat <<'EOF'
<NEWS.md section for this release>
EOF
)"
```

### Step 14: Done

Display a summary:

```
## Release Complete

Package:  <name>
Version:  <version>
Tag:      v<version>
PR:       #<number>
Release:  <url from `gh release view v<version> --json url -q .url`>
```

## Important Rules

1. **Always create a release branch** — never commit directly to main
2. **Always use squash merge** for release PRs
3. **Always tag after merging** — the tag should point to the merge commit on main
4. **Always bump to dev version** after tagging
5. **Never skip CI** — wait for checks to pass before merging
6. **Use `/pr-create`** for PR creation and CI monitoring — don't reinvent it
