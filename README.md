# questkeep/workflows

Reusable GitHub Actions workflows shared by the Questkeep repositories.

## update-toolchain

Opens (or updates) a weekly pull request that bumps the exact versions pinned in the calling
repository's `mise.toml`. Dependabot cannot do this: it has no mise ecosystem.

For every tool you list, the workflow takes the newest stable release that is at least
`minimum-release-age` old (default `2d`, the same minimum age Questkeep uses for npm and Cargo
dependencies) and writes it into `mise.toml`, plain (`node = "26.13.1"`) or in table form
(`rust = { version = "1.100.0", ... }`). Versions only ever go up.

### Use it

`.github/workflows/update-toolchain.yml` in the calling repository:

```yaml
name: Update toolchain

on:
  schedule:
    - cron: "0 13 * * 5" # Fridays, 13:00 UTC
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  upgrade:
    uses: questkeep/workflows/.github/workflows/update-toolchain.yml@main
    with:
      tools: '{"node": "lts", "pnpm": "latest", "rust": "latest"}'
```

### Inputs

| Input | Default | Meaning |
|---|---|---|
| `tools` | required | JSON object: tool name from `mise.toml` to a version label (below) |
| `minimum-release-age` | `2d` | Ignore releases younger than this (a mise duration such as `2d`, `1w`) |
| `branch` | `chore/mise-upgrade` | Branch of the pull request |
| `commit-message` | `build(deps): update mise tools` | Commit message and pull request title |

Version labels:

| Label | Picks |
|---|---|
| `latest` | the newest stable release |
| `lts` | the newest release of the latest LTS major (`mise latest <tool>@lts`), for tools that have one such as `node`; never an odd-numbered Current release |
| a prefix such as `26` or `1.100` | the newest release below that prefix |

Tool names are the plain names used in `mise.toml` (`node`, `pnpm`, `rust`).

### Rules

- **Every tool pinned in `mise.toml` must be listed in `tools`**, or the run fails: a tool you add
  to `mise.toml` would otherwise never be updated. When you add or remove a tool, update `tools`
  in the calling workflow.
- A tool listed in `tools` but not pinned in `mise.toml` is skipped with a notice (so a template
  can list tools that only some generated repositories use).
- Pin tools exactly (`node = "26.13.1"`). Pins like `"26"` or `"latest"` are rewritten to the
  newest matching version as well, but then the file no longer says what is installed.

### The pull request

The branch is rebuilt from the default branch and force-pushed on every run, so an existing
branch and pull request are updated in place (commits, title and description); if everything is
current by then, an open pull request is closed and its branch deleted. Do not add commits to the
branch.

Pull requests opened with `GITHUB_TOKEN` do not start CI: close and reopen the pull request to run
the checks. The organization (or repository) setting **Allow GitHub Actions to create and approve
pull requests** must be on.

## Changing a workflow here

Callers use `@main`, so a change applies to every repository at once on its next run. Keep inputs
backwards compatible.
