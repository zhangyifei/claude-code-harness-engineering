---
description: "Prepare a release: changelog, version bump, tag. Trigger: user says 'release', 'cut a release', 'prepare release', 'changelog'."
---

# /release — Release Preparation

Prepare the project for a release: changelog, version bump, git tag.

## Step 1: Determine Version

Read recent commits since last tag:
```bash
git log $(git describe --tags --abbrev=0 2>/dev/null || echo "HEAD~20")..HEAD --oneline
```

Apply SemVer:
- **Breaking changes** (removed features, incompatible API) → **major** (X.0.0)
- **New features** (added functionality) → **minor** (x.Y.0)
- **Bug fixes, docs, refactors** → **patch** (x.y.Z)

If no tags exist, suggest `0.1.0` as first release.

## Step 2: Update Changelog

Create or update `CHANGELOG.md` using [Keep a Changelog](https://keepachangelog.com/) format:

```markdown
# Changelog

## [X.Y.Z] - YYYY-MM-DD

### Added
- [New features]

### Changed
- [Modified behavior]

### Fixed
- [Bug fixes]

### Removed
- [Removed features]
```

Group commits by category. Write human-readable descriptions, not raw commit messages.

## Step 3: Version Bump

Update version in the relevant file:
- `package.json` → Node.js / Next.js
- `pyproject.toml` → Python
- `go` → no version file (tag only)
- Other → ask user

## Step 4: Confirm with User

```
RELEASE PREPARED: vX.Y.Z

Changelog:
[show changelog entry]

Files modified:
- CHANGELOG.md
- [version file]

Next steps (manual):
1. Review the changelog
2. Commit: git commit -m "release: vX.Y.Z"
3. Tag: git tag vX.Y.Z
4. Push: git push && git push --tags
```

Do NOT commit, tag, or push automatically — let the user do it after reviewing.
