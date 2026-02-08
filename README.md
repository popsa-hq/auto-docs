# auto-docs

A documentation auto-update pipeline powered by [Claude Code](https://claude.com/product/claude-code) and [GitHub Actions](https://github.com/features/actions). When code merges to your repositories, the pipeline detects which documentation files are affected, runs an AI agent to update them, and opens a pull request for human review.

## How it works

```text
Source Repo (merge to main)
  --> docs-notify.yml
       --> repository_dispatch to your-org/your-docs-repo
            --> auto-docs-update.yml
                 |-- checkout docs + source repo
                 |-- match changed paths to affected doc files
                 |-- run Claude Code to regenerate docs
                 --> open PR with "auto-docs" label
```

1. An engineer merges code to a source repository.
2. A lightweight **sender** workflow (`docs-notify.yml`) fires, collects the changed file paths and commit reference, and dispatches a [`repository_dispatch`](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#repository_dispatch) event to your central documentation repository.
3. The **receiver** workflow (`auto-docs-update.yml`) picks up the event and consults `docs-config.yml` to determine which documentation files could be affected by the change, using glob-based `watch_paths`.
4. For each affected doc, Claude Code reads the existing documentation, explores the source repository at the exact triggering commit, compares the two, and proposes targeted edits.
5. If changes are needed, a PR is opened for human review. If nothing meaningful changed, the pipeline closes cleanly (`NO_CHANGES_NEEDED`).

## Setup

### Prerequisites

- An [Anthropic API key](https://console.anthropic.com/) for Claude Code
- A [GitHub Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) (PAT) with `repo` scope, used to dispatch events and check out private source repos

### Docs repository

1. Copy `auto-docs-update.yml` into your docs repo at `.github/workflows/auto-docs-update.yml`.
2. Copy `docs-config.example.yml` to `.github/docs-config.yml` and customise it for your repositories (see [Configuration](#configuration) below).
3. Copy `auto-docs-prompt.example.md` to `.github/auto-docs-prompt.md` and adjust the rules to match your team's conventions.
4. Add the following secrets to your docs repo:
   - `CLAUDE_API_KEY` — your Anthropic API key
   - `DOCS_DISPATCH_TOKEN` — the GitHub PAT
5. Create an `auto-docs` label in your docs repo (used to tag auto-generated PRs).

### Source repositories

For each repo you want to participate:

1. Copy `docs-notify.yml` into the repo at `.github/workflows/docs-notify.yml`.
2. Update the `branches` trigger if your default branch isn't `main`.
3. Update the `repository` field in the dispatch step to point at your docs repo.
4. Add the `DOCS_DISPATCH_TOKEN` secret (the same GitHub PAT).
5. Add an entry in your docs repo's `docs-config.yml` for this source repo.

## Configuration

The `docs-config.yml` file maps source repositories to the documentation files they affect:

```yaml
repositories:
  org/backend:
    branch: main                    # Branch that triggers updates
    checkout_path: source/backend   # Where to clone the source repo
    docs:
      - file: standards/go.md      # Doc file in the docs repo
        scope: "Go coding standards, conventions, and tooling"
        watch_paths:                # Globs matched against changed files
          - "**/*.go"
          - "go.mod"
          - ".golangci.yml"
```

- **`branch`** — the source repo branch that triggers docs updates.
- **`checkout_path`** — a temporary directory for cloning the source repo during the workflow run.
- **`docs[].file`** — path to the documentation file in your docs repo.
- **`docs[].scope`** — a natural-language description of what this doc covers, passed to Claude as context.
- **`docs[].watch_paths`** — glob patterns matched against the changed file paths. Only docs whose watch_paths intersect with actual changes are regenerated.

Wildcard entries (e.g., `org/*`) act as a fallback for repositories without an explicit entry.

### Prompt template

The `auto-docs-prompt.md` template supports these variables, substituted at runtime:

| Variable | Description |
|---|---|
| `$DOC_FILE` | Path to the documentation file being updated |
| `$SOURCE_PATH` | Path to the checked-out source repository |
| `$REPOSITORY` | Full `org/repo` name of the source repository |
| `$SCOPE` | The scope string from `docs-config.yml` |
| `$CHANGED_PATHS` | Comma-separated list of files changed in the triggering commit |
| `$SHA` / `$SHA_SHORT` | Full and abbreviated commit SHA |
| `$BRANCH` | Source branch name |
| `$COMPONENT` | Derived from the doc filename (e.g., `standards/go.md` yields `go`) |

## Design decisions

- **Event-driven, not scheduled.** Source repos push events when changes happen. No polling, no cron. Any new repository can participate by adding the sender workflow and a config entry.
- **Concurrency queuing.** Only one documentation update per source repository runs at a time. If two PRs merge in quick succession, the second update queues rather than being discarded.
- **`NO_CHANGES_NEEDED` exit.** Not every code change affects documented conventions. Claude can determine that no docs update is required, and the pipeline closes cleanly without opening a PR.
- **Human review required.** Auto-merge is deliberately not included. Every change opens a PR for engineers to review, maintaining human ownership of the knowledge base.
- **Major version history (optional).** When a major semver bump is detected, an additional step can update an architecture history log. Remove this step if you don't need it.

## Origin

This pipeline was built at [Popsa](https://popsa.com) to keep engineering documentation in sync with a multi-repo codebase. You can read the full story in [I'd Rather Build the System](https://blog.popsa.com/id-rather-build-the-system/).

## Licence

[MIT](LICENSE)
