---
name: r-xla-release
description: >
  Prepare a release of one or more r-xla packages (anvil, stablehlo, pjrt, tengen, xlamisc).
  Use when the user says "release", "make a release", "bump version", "cut a release", or
  wants to publish new versions of r-xla packages.
user_invocable: true
tools: Read, Edit, Glob, Grep, Bash, Write, AskUserQuestion, Skill
---

# r-xla Release

Your role is to prepare a release of one or more packages in the r-xla ecosystem.

All r-xla packages live as sibling directories under a common parent. From any
package directory, the others are reachable via `../<package>/`.

The R packages are: **anvil**, **stablehlo**, **pjrt**, **tengen**, **xlamisc**.

## Workflow

### Step 1: Discover Packages and Show Versions

Determine the parent directory. If the current working directory is one of the
package directories, use `..` as the parent. Otherwise, use `.`.

For each R package (anvil, stablehlo, pjrt, tengen, xlamisc), read its
`DESCRIPTION` file and extract the `Version:` and `Package:` fields. Also read
the first heading of each `NEWS.md` to show the current changelog status.

Present a table to the user:

```
Package     Version        NEWS.md status
anvil       0.1.0.9000     (development version)
stablehlo   0.1.0.9000     (development version)
pjrt        0.1.1.9000     (development version)
tengen      0.1.0.9000     (development version)
xlamisc     0.1.0.9000     (development version)
```

### Step 2: Ask Which Packages to Release

Use `AskUserQuestion` to ask the user which packages they want to release.

Then, for each selected package, suggest an appropriate release type based on
the NEWS.md content:

- If NEWS mentions "breaking changes" or similar → suggest **Major**
- If NEWS mentions only "bug fixes", "fixes", "patch" → suggest **Patch**
- Otherwise → suggest **Minor**

Use `AskUserQuestion` to confirm the release type for each package:

Question: "What type of release for `<package>` (currently `<version>`)?"
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

Display a summary: "Will release: anvil 0.1.0.9000 → 0.2.0, pjrt 0.1.1.9000 → 0.1.2"

### Step 3: Determine Processing Order

The packages have inter-dependencies. When releasing multiple packages together,
they **must** be processed in dependency order so that each package's CI can
install its dependencies from their release branches.

The dependency graph (fixed order):

```
Layer 0 (no r-xla deps):  tengen, xlamisc
Layer 1:                   pjrt     (depends on tengen)
Layer 2:                   stablehlo (depends on tengen, xlamisc; suggests pjrt)
Layer 3:                   anvil    (depends on pjrt, stablehlo, tengen, xlamisc)
```

Process packages in this order. Within a layer, order does not matter. Only
process packages the user selected for release — skip the others.

### Step 4: Create Release Branch

For each package (in dependency order), change into its directory and create a
release branch:

```bash
cd <parent>/<package>
git checkout main
git pull origin main
git checkout -b release/<version>
```

If a `release/*` branch already exists for this package, ask the user how to
proceed using `AskUserQuestion`.

### Step 5: Update DESCRIPTION — Version

Read `DESCRIPTION` and replace the `Version:` field with the new version
(e.g., `0.1.0.9000` → `0.2.0`). Use the Edit tool — do not rewrite the file.

### Step 6: Update DESCRIPTION — Remotes

**This is critical for multi-package releases.** When a package depends on other
r-xla packages that are also being released, its `Remotes:` field must point to
the release branches of those dependencies so CI installs the correct versions.

Read the existing `Remotes:` field in `DESCRIPTION`. For each remote that
references an r-xla package **that is also being released**, update it to point
to the release branch:

```
r-xla/<dep-package>@release/<dep-version>
```

For example, if releasing anvil 0.2.0 alongside stablehlo 0.2.0 and pjrt 0.1.2,
anvil's Remotes should become:

```
Remotes:
    r-xla/stablehlo@release/0.2.0,
    r-xla/tengen@release/0.2.0,
    r-xla/xlamisc@release/0.2.0,
    r-xla/pjrt@release/0.1.2
```

Only update remotes for packages that are part of this release. Leave remotes
for packages not being released unchanged.

Use the Edit tool for this change.

### Step 7: Update NEWS.md

Read `NEWS.md`. Replace the development version header with the release version.

The header format is: `# <package> (development version)`
Replace with: `# <package> <version>`

For example:
- `# anvil (development version)` → `# anvil 0.2.0`

Use the Edit tool for this change.

### Step 8: Commit and Push Release Branch

```bash
git add DESCRIPTION NEWS.md
git commit -m "$(cat <<'EOF'
release: <package> <version>
EOF
)"
git push -u origin release/<version>
```

**Important:** The branch must be pushed immediately so that downstream packages
(in later layers) can reference it in their Remotes during CI. This is why
packages are processed in dependency order.

After pushing, proceed to the next package (back to Step 4). Do NOT create the
PR yet — all release branches must be pushed first so that CI for downstream
packages can find their dependencies.

### Step 9: Create PRs and Monitor CI

Once **all** release branches are pushed, create PRs for each package (again in
dependency order). For each package:

Use the `/pr-create` skill to create a pull request for the release branch.

The PR title should be: `release: <package> <version>`

The PR body should include a summary of the NEWS.md entries for this release.

Wait for CI to pass before moving on to the next package's PR. The `/pr-create`
skill handles CI monitoring and fixing. If CI fails because a dependency's
release branch has issues, fix the dependency first, then re-run CI.

### Step 10: Merge, Tag, and Bump Dev Version

Once all PRs have passing CI, merge them **in dependency order** (same order as
above). For each package:

1. Merge the PR using squash merge:

```bash
cd <parent>/<package>
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

4. Bump to development version. Update `DESCRIPTION`:
   - Append `.9000` to the version (e.g., `0.2.0` → `0.2.0.9000`)
   - Revert Remotes back to development state (remove `@release/<version>`
     branch pins so they point to the default branch again)

   Update `NEWS.md` by adding a new section at the top:

```markdown
# <package> (development version)

```

   Commit and push:

```bash
git add DESCRIPTION NEWS.md
git commit -m "$(cat <<'EOF'
release: <package> <version>.9000
EOF
)"
git push origin main
```

### Step 11: Final Summary

When all packages are done, display a final summary:

```
## Release Summary

| Package   | Version | Tag    | PR     |
|-----------|---------|--------|--------|
| tengen    | 0.2.0   | v0.2.0 | #<num> |
| xlamisc   | 0.2.0   | v0.2.0 | #<num> |
| pjrt      | 0.1.2   | v0.1.2 | #<num> |
| stablehlo | 0.2.0   | v0.2.0 | #<num> |
| anvil     | 0.2.0   | v0.2.0 | #<num> |

All releases completed.
```

## Important Rules

1. **Always create release branches** — never commit directly to main
2. **Always use squash merge** for release PRs
3. **Always tag after merging** — the tag should point to the merge commit on main
4. **Always bump to dev version** after tagging
5. **Respect dependency order** — tengen/xlamisc first, then pjrt, stablehlo, anvil last
6. **Push all release branches before creating PRs** — so downstream CI can find dependencies
7. **Update Remotes** to point to release branches for co-released packages
8. **Revert Remotes** when bumping back to dev version after release
9. **Never skip CI** — wait for checks to pass before merging
10. **Use `/pr-create`** for PR creation and CI monitoring — don't reinvent it
