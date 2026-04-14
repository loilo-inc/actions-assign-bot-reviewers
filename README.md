# actions-assign-bot-reviewers

Bot が作成した PR にレビュアーを自動アサインする GitHub Composite Action。

CODEOWNERS の代替として、Bot PR のみにレビュアーを割り当てたい場合に使用します。

## 使い方

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
| `token` | Yes | - | GitHub token (`pull-requests: write` 権限が必要) |
| `reviewers` | Yes | - | アサインするレビュアー（カンマ区切り、ユーザー名・`org/team` 混在可） |
| `exclude_bots` | No | `""` | 除外する Bot（カンマ区切り） |

### `reviewers` の指定例

```yaml
# チームのみ
reviewers: "loilo-inc/server-reviewers"

# ユーザーのみ
reviewers: "user1,user2"

# 混在
reviewers: "loilo-inc/server-reviewers,user1"
```

### `exclude_bots` の指定例

他のレビューフロー（例: Claude Code の AI レビューループ）が存在する Bot を除外する場合に使用します。

```yaml
exclude_bots: "claude[bot]"
```

## job レベルのフィルタについて

`if: github.event.pull_request.user.type == 'Bot'` を job レベルに設定することで、人間の PR では job 自体が起動せず runner のコストを節約できます。Action 内部にも同じ判定がありますが、job レベルのフィルタとの併用を推奨します。

## ライセンス

MIT
