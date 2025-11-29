# Release Workflow Setup

This setup implements an automated release workflow with two branches:
- **develop** - staging/default branch where features are merged
- **main** - production branch for releases

## Workflow Overview

```
┌─────────────┐     PR merged      ┌─────────────┐
│   feature   │ ─────────────────► │   develop   │
│   branch    │                    │  (staging)  │
└─────────────┘                    └──────┬──────┘
                                          │
                                          │ push triggers
                                          │ auto-release-pr
                                          ▼
                                   ┌──────────────┐
                                   │ Release PR   │
                                   │ (auto-created│
                                   │  to main)    │
                                   └──────┬───────┘
                                          │
                                          │ merge release PR
                                          ▼
                                   ┌─────────────┐
                                   │    main     │
                                   │ (production)│
                                   └──────┬──────┘
                                          │
                                          │ push triggers
                                          │ create-release
                                          │ and sync workflow
                                          ▼
                                   ┌─────────────┐
                                   │   develop   │◄── synced with
                                   │             │    release commit
                                   └─────────────┘
```

## How It Works

### 1. Feature Development
- Create feature branch from `develop`
- Make changes using conventional commits (e.g., `feat:`, `fix:`, `chore:`)
- Open PR to merge into `develop`

### 2. Release PR Creation (Automatic)
When a PR is merged to `develop`:
- `auto-release-pr.yml` triggers on push to `develop`
- Workflow analyzes commits since last release
- **Automatically creates/updates a Release PR from `develop` to `main`**
- The Release PR includes:
  - Proposed version bump
  - All conventional commits summarized in the PR description

### 3. Release to Production (Automatic)
When you merge the Release PR to `main`:
- `create-release.yml` triggers automatically
- Updates `package.json` with new version
- Generates and updates `CHANGELOG.md`
- Creates a GitHub Release with tag
- `sync-main-to-develop.yml` automatically syncs `main` back to `develop`

### 4. Sync Back (Automatic)
After the release:
- The sync workflow merges `main` into `develop`
- This ensures `develop` has the version bump and changelog updates
- Prevents divergence between branches

## Files

```
.github/
└── workflows/
    ├── auto-release-pr.yml         # Creates PR from develop to main
    ├── create-release.yml          # Creates release when PR is merged
    └── sync-main-to-develop.yml    # Syncs after release
release-please-config.json          # Configuration (optional)
.release-please-manifest.json       # Tracks current version (optional)
```

## Setup Instructions

### 1. Required Secrets
Add the following secret to your repository:
- `ORG_GITHUB_WRITE_TOKEN` - A GitHub token with `contents: write` and `pull-requests: write` permissions

### 2. Branch Protection (Recommended)
Configure branch protection for both `develop` and `main`:
- Require pull request reviews
- Require status checks to pass
- Allow the GitHub Actions bot to bypass (for sync workflow)

### 3. Initial Setup
1. ✅ All files have been added to your repository
2. Set `develop` as the default branch (if not already)
3. Start making conventional commits to `develop`
4. Merge PRs to `develop` → Release PR will automatically be created to `main`

## Conventional Commits

Use these prefixes for proper changelog generation:

| Prefix | Description | Changelog Section |
|--------|-------------|-------------------|
| `feat:` | New feature | ✨ Features |
| `fix:` | Bug fix | 🐛 Bugfixes |
| `chore:` | Maintenance | 👽 Miscellaneous |
| `style:` | Code style | 👽 Miscellaneous |
| `deps:` | Dependencies | ⬆️ Dependency updates |
| `ops:` | Operations | ♻️ OPS |
| `ci:` | CI changes | ♻️ OPS |
| `docs:` | Documentation | 📜 Documentation |

### Examples
```
feat: add user authentication
fix: resolve login timeout issue
chore: update dependencies
docs: improve API documentation
```

## Troubleshooting

### Release PR not created
- Ensure commits use conventional commit format (`feat:`, `fix:`, etc.)
- Check that `auto-release-pr.yml` workflow ran successfully
- Verify `ORG_GITHUB_WRITE_TOKEN` secret is set

### Sync workflow fails
- Check if there are merge conflicts between `main` and `develop`
- Ensure the token has push permissions to `develop`

### Version not bumped correctly
- Check that commits follow conventional commit format
- Major bump: Include "BREAKING CHANGE" in commit message
- Minor bump: Use `feat:` prefix
- Patch bump: Use `fix:` prefix

## Notes

- The Release PR automatically updates when you merge more PRs to `develop`
- Only one Release PR exists at a time (from `develop` to `main`)
- The sync workflow uses `[skip ci]` to avoid triggering another workflow
- Version bumps and changelog updates happen automatically when merging to `main`

