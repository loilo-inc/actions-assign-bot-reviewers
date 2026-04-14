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
          token: ${{ secrets.GITHUB_TOKEN }}
          reviewers: "loilo-inc/server-reviewers"
```

## Inputs

| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `token` | Yes | - | GitHub token (requires `pull-requests: write` permission) |
| `reviewers` | Yes | - | Reviewers to assign (comma-separated, supports both usernames and `org/team`) |
| `exclude_bots` | No | `""` | Bots to exclude (comma-separated) |

### `reviewers` examples

```yaml
# Team only
reviewers: "loilo-inc/server-reviewers"

# Users only
reviewers: "user1,user2"

# Mixed
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
