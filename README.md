# actions-assign-bot-reviewers

A GitHub Composite Action that automatically assigns reviewers when a bot creates a pull request.

Use this as a replacement for CODEOWNERS when you only want reviewers assigned to bot-created PRs.

## Usage

```yaml
# .github/workflows/assign-bot-reviewers.yml
name: Assign reviewers for Bot PRs

on:
  pull_request:
    types: [opened, reopened]

jobs:
  assign:
    if: github.event.pull_request.user.type == 'Bot'
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - uses: loilo-inc/actions-assign-bot-reviewers@v1
        with:
          # Use a PAT or GitHub App token when assigning team reviewers.
          # secrets.GITHUB_TOKEN is sufficient for user-only reviewers.
          token: ${{ secrets.BOT_REVIEWER_TOKEN }}
          reviewers: "loilo-inc/server-reviewers"
```

## Inputs

| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `token` | Yes | - | GitHub token (see [Token permissions](#token-permissions) below) |
| `reviewers` | Yes | - | Reviewers to assign (comma-separated, supports both usernames and `org/team`) |
| `exclude_bots` | No | `""` | Bots to exclude (comma-separated) |

### Token permissions

| Reviewer type | Required token | Required permissions |
|---------------|---------------|---------------------|
| **Users only** | `secrets.GITHUB_TOKEN` | `pull-requests: write` |
| **Teams** (or mixed) | PAT or GitHub App token | `pull-requests: write` + `read:org` |

> **Note:** `GITHUB_TOKEN` does not have organization-level visibility, so requesting team reviewers with it may result in a 422 error.

### `reviewers` examples

```yaml
# Team only (requires PAT or GitHub App token)
reviewers: "loilo-inc/server-reviewers"

# Users only (GITHUB_TOKEN is sufficient)
reviewers: "user1,user2"

# Mixed (requires PAT or GitHub App token)
reviewers: "loilo-inc/server-reviewers,user1"
```

### `exclude_bots` examples

Use this to skip bots that have their own review flow (e.g. Claude Code's AI review loop).

```yaml
exclude_bots: "claude[bot]"
```

## Job-level filter

Adding `if: github.event.pull_request.user.type == 'Bot'` at the job level prevents the runner from starting for human PRs, saving runner costs. The action also checks internally, but combining both is recommended.

## License

MIT
