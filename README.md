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
| `local>nicebots-xyz/renovate-config` | `config:recommended` (dependency dashboard, `fix(deps)`/`chore(deps)` semantic commits, monorepo grouping), pinned digests, lock file maintenance, a 7-day minimum release age for npm and labels |
| `local>nicebots-xyz/renovate-config:automerge-python-lockfile` | Opt-in. Automerges lock file maintenance of Python package managers once every status check is green |

### Labels

Every PR gets `deps`, plus:

- `deps:python`, `deps:javascript`, `deps:github-actions`, `deps:docker`, `deps:optional` for the ecosystem
- `update:major`, `update:minor`, `update:patch`, `update:digest`, `update:pin`, `update:lockfile` for the kind of update

### Minimum release age

npm releases must be 7 days old before Renovate proposes them (`minimumReleaseAge` with
`internalChecksFilter: "strict"`, so no branch exists until then). This mirrors Renovate's own
`security:minimumReleaseAgeNpm` preset with a longer delay. As with Python, lock file maintenance isn't covered,
so pair it with the package manager's own cooldown where it has one.

### Automerge

`automerge-python-lockfile` sets `platformAutomerge: false`, so Renovate merges the PR itself only once every
status check on the branch has passed. It doesn't rely on GitHub auto-merge, which only waits for the checks that
branch protection marks as required.

Commits pushed by [autofix.ci](https://autofix.ci) or by [nicebots-xyz/fixer](https://github.com/nicebots-xyz/fixer)
(the NiceBots Autofixer app), e.g. reformatting after a formatter bump, are listed in `gitIgnoredAuthors`. Renovate
still treats the branch as its own: it keeps updating it and can still automerge it. When Renovate rebases, it drops
the fix commit and the fixer applies it again.

Renovate can't apply `minimumReleaseAge` to lock file maintenance, so set a cooldown in the package manager too,
e.g. for PDM:

```toml
[tool.pdm.resolution]
exclude-newer = "7d"
```
