# AGENTS.md

## Language

All **externally visible** content MUST be written in **English**:

- Git commit messages
- Pull request titles and descriptions
- GitHub Issues (title and body)
- Code comments
- README and documentation
- GitHub Release notes
- action.yml descriptions and labels

Internal conversations with the developer may be in any language.

## Project Overview

This is a GitHub Composite Action (`action.yml`) that auto-assigns reviewers to bot-created pull requests. It uses `actions/github-script` to call the GitHub API.

## Structure

```
action.yml   # The entire action logic (composite action with inline JS)
README.md    # Usage documentation
LICENSE      # MIT
```

## Commit Convention

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>: <description>
```

Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`

## Pull Requests

- Keep PRs small and focused on a single concern
- PR title follows the same conventional commit format
- PR body should explain **why**, not just **what**

## Code Guidelines

- The action is a single `action.yml` — keep it that way unless complexity demands extraction
- Inline JavaScript runs inside `actions/github-script`; follow standard JS conventions
- Prefer early returns over deeply nested conditionals
- Log meaningful messages with `core.info()` / `core.warning()` for debuggability
