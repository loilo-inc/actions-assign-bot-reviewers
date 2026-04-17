# actions-assign-bot-reviewers

A GitHub Composite Action that automatically assigns reviewers when a bot creates a pull request.

Use this as a replacement for CODEOWNERS when you only want reviewers assigned to bot-created PRs.

## Usage

```yaml
# .github/workflows/assign-bot-reviewers.yml
name: Assign reviewers for Bot PRs

on:
  pull_request_target:
    types: [opened, reopened]

jobs:
  assign:
    if: github.event.pull_request.user.type == 'Bot'
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      # Generate a GitHub App token for team reviewer assignment.
      # GITHUB_TOKEN lacks read:org, so a GitHub App or PAT is required.
      - uses: actions/create-github-app-token@v3
        id: app-token
        with:
          client-id: ${{ secrets.APP_CLIENT_ID }}
          private-key: ${{ secrets.APP_PRIVATE_KEY }}

      - uses: loilo-inc/actions-assign-bot-reviewers@v1
        with:
          token: ${{ steps.app-token.outputs.token }}
          reviewers: "loilo-inc/server-reviewers"
```

If you only assign **individual users** (no teams), `secrets.GITHUB_TOKEN` is sufficient:

```yaml
      - uses: loilo-inc/actions-assign-bot-reviewers@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          reviewers: "user1,user2"
```

### Why `pull_request_target`?

Dependabot-triggered workflows on the `pull_request` event always receive a
read-only `GITHUB_TOKEN` and cannot access repository secrets, so reviewer
assignment (and fetching a GitHub App token from secrets) fails. Using
`pull_request_target` runs the workflow against the base branch with full
token permissions and secret access.

This action only reads PR metadata from `github.event.pull_request` and calls
the GitHub API — it never checks out or executes PR code — so it is safe to
run under `pull_request_target`. Do **not** add a `checkout` step that pulls
the PR's head ref into this workflow.

## Inputs

| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `token` | Yes | - | GitHub token (see [Token permissions](#token-permissions) below) |
| `reviewers` | Yes | - | Reviewers to assign (comma-separated, supports both usernames and `org/team`) |
| `exclude-bots` | No | `""` | Bots to exclude (comma-separated) |

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

### `exclude-bots` examples

Use this to skip bots that have their own review flow (e.g. Claude Code's AI review loop).

```yaml
exclude-bots: "claude[bot]"
```

## Job-level filter

Adding `if: github.event.pull_request.user.type == 'Bot'` at the job level prevents the runner from starting for human PRs, saving runner costs. The action also checks internally, but combining both is recommended.

## License

MIT
