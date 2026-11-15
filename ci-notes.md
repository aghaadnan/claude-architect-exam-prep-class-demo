# CI Notes for Exam Prep

Notes on using Claude Code as a CI worker.

## 1. Get the `gh` CLI

The GitHub CLI (`gh`) lets you work with issues, pull requests, and Actions from the terminal.

- macOS: `brew install gh`
- Debian/Ubuntu: `sudo apt install gh`
- Windows: `winget install --id GitHub.cli`

Then authenticate and verify:

```bash
gh auth login
gh auth status
```

Useful commands: `gh issue list`, `gh pr create`, `gh pr view`, `gh run list`, `gh run watch`.

## 2. CI concept using a GitHub project

Continuous Integration (CI) automatically runs checks on every change pushed to a repository.

- Workflows are YAML files in `.github/workflows/`.
- A workflow is triggered by events (`push`, `pull_request`, `issue_comment`, ...).
- Each workflow has jobs that run on a runner (e.g. `ubuntu-latest`) and consist of steps.
- Results appear as status checks on the pull request, which can gate merging.

Workflows in this repository:

| File | Purpose |
| --- | --- |
| `.github/workflows/claude.yml` | Runs Claude when `@claude` is mentioned in an issue or PR |
| `.github/workflows/claude-code-review.yml` | Automated Claude review of pull requests |
| `.github/workflows/markdown-only.yml` | Checks that the repository contains only Markdown content |

## 3. Live demo: Claude as a CI worker

Demo flow:

1. Open an issue describing the task (e.g. "Add CI notes for exam preparation").
2. Comment `@claude resolve issue and create PR for main.`
3. The `claude.yml` workflow starts; Claude reads the issue, edits files on a new branch, and pushes commits.
4. Claude posts progress in a single comment and links to a prefilled PR.
5. Review and merge the PR like any other.

## 4. In-depth configuration of Claude Code as a CI worker

Uses the `anthropics/claude-code-action@v1` action.

- **Authentication:** store `CLAUDE_CODE_OAUTH_TOKEN` (or an API key) as a repository secret; never commit it.
- **Triggers:** gate the job with an `if:` on the event and the `@claude` phrase.
- **Permissions:** grant the minimum (`contents`, `pull-requests`, `issues`, `id-token: write`; `actions: read` to read CI results).
- **`prompt`:** optional fixed instruction; if omitted Claude follows the triggering comment.
- **`claude_args`:** pass CLI flags, e.g. `--allowed-tools`, `--max-turns`, `--model`.
- **`CLAUDE.md`:** repo-level guidance Claude reads on every run (conventions, build/test commands).
- **Limits:** the action cannot modify files in `.github/workflows/`, cannot approve PRs, and communicates through a single updated comment.
- **Safety:** keep tool allowlists narrow and review all Claude-authored PRs before merging.
