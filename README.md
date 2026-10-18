# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) presets for nicebots-xyz repositories.

## Usage

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["local>nicebots-xyz/renovate-config"],
  "forkProcessing": "enabled",
  "baseBranchPatterns": ["dev"]
}
```

`forkProcessing` has to stay in each repository: Renovate reads it from the repository's own config file.

## Presets

| Preset | What it does |
| --- | --- |
| `local>nicebots-xyz/renovate-config` | `config:recommended` (dependency dashboard, `fix(deps)`/`chore(deps)` semantic commits, monorepo grouping), pinned digests, lock file maintenance and labels |
| `local>nicebots-xyz/renovate-config:automerge-python-lockfile` | Opt-in. Automerges lock file maintenance of Python package managers once every status check is green |

### Labels

Every PR gets `deps`, plus:

- `deps:python`, `deps:javascript`, `deps:github-actions`, `deps:docker`, `deps:optional` for the ecosystem
- `update:major`, `update:minor`, `update:patch`, `update:digest`, `update:pin`, `update:lockfile` for the kind of update

### Automerge

`automerge-python-lockfile` sets `platformAutomerge: false`, so Renovate merges the PR itself only once every
status check on the branch has passed. It doesn't rely on GitHub auto-merge, which only waits for the checks that
branch protection marks as required.

Renovate can't apply `minimumReleaseAge` to lock file maintenance, so set a cooldown in the package manager too,
e.g. for PDM:

```toml
[tool.pdm.resolution]
exclude-newer = "7d"
```
