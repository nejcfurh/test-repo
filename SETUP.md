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
                                          │ release-please
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
                                          │ sync workflow
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
- `release-please.yml` triggers on push to `develop`
- Release Please analyzes commits since last release
- Creates/updates a Release PR targeting `main`
- The Release PR includes:
  - Version bump in `package.json`
  - Updated `CHANGELOG.md`
  - All conventional commits summarized

### 3. Release to Production
When you're ready to release:
- Review and merge the Release PR to `main`
- Release Please creates a GitHub Release with tag
- `sync-main-to-develop.yml` automatically syncs `main` back to `develop`

### 4. Sync Back (Automatic)
After merging to `main`:
- The sync workflow merges `main` into `develop`
- This ensures `develop` has the version bump and changelog updates
- Prevents divergence between branches

## Files

```
.github/
└── workflows/
    ├── release-please.yml          # Creates release PRs
    └── sync-main-to-develop.yml    # Syncs after release
release-please-config.json          # Release Please configuration
.release-please-manifest.json       # Tracks current version
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
1. ✅ All files have been copied to your repository
2. Set `develop` as the default branch (if not already)
3. ✅ `.release-please-manifest.json` has been set to your current version (1.5.1)
4. Start making conventional commits to `develop`

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
- Ensure commits use conventional commit format
- Check that `release-please.yml` has correct permissions
- Verify `ORG_GITHUB_WRITE_TOKEN` secret is set

### Sync workflow fails
- Check if there are merge conflicts between `main` and `develop`
- Ensure the token has push permissions to `develop`

### Multiple release PRs
- This shouldn't happen with proper setup
- If it does, close duplicates and keep the most recent one

## Notes

- The Release PR accumulates all changes since the last release
- Merging multiple feature PRs to `develop` will update the same Release PR
- Only one Release PR exists at a time (targeting `main`)
- The sync workflow uses `chore:` prefix to avoid triggering another release

## Old Workflow

Your old workflow file (`github-pr-release.yml`) has been left in place. You can:
- Delete it if you want to use only the new release-please workflow
- Keep both if you want to migrate gradually
- The new workflow is simpler and more maintainable using Google's release-please action

