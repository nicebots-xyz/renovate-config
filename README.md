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
| `local>nicebots-xyz/renovate-config` | `config:recommended` (dependency dashboard, `fix(deps)`/`chore(deps)` semantic commits, monorepo grouping), pinned digests, cooldowns, lock file maintenance, explicit titles and labels |
| `local>nicebots-xyz/renovate-config:automerge-python-lockfile` | Opt-in. Automerges lock file maintenance of Python package managers once every status check is green |
| `local>nicebots-xyz/renovate-config:automerge-low-risk` | Opt-in. Automerges patches, digest updates and minor GitHub Actions updates once every status check is green, Tegami excepted |

Repository configs only hold what is specific to the repository: its own files, packages and sources. A rule that
would make sense in every repository belongs here.

### Labels

Every PR gets `deps`, plus:

- `deps:python`, `deps:javascript`, `deps:github-actions`, `deps:docker`, `deps:optional` for the ecosystem
- `update:major`, `update:minor`, `update:patch`, `update:digest`, `update:pin`, `update:lockfile` for the kind of update

### Minimum release age

Renovate waits before proposing a release, so malware scanners can flag it and a pulled release never lands.
`internalChecksFilter: "strict"` means no branch exists until then.

| Source | Wait |
| --- | --- |
| npm | 7 days |
| PyPI | 7 days |
| Docker images, GitHub releases and tags (actions included) | 5 days |

Lock file maintenance isn't covered, so pair it with the package manager's own cooldown where it has one, with the
same 7 days for PyPI so Renovate never proposes a release the lock file refuses:

```toml
[tool.pdm.resolution]
exclude-newer = "7d"
```

A source without release timestamps never passes the wait. Turn it off for that source in the repository
(`"minimumReleaseAge": null`), or track the dependency through a source that has timestamps.

### Grouped updates

Grouped pull requests name their packages: `upgrade python dependencies (aiohttp, py-cord, pydantic)`, up to six
names and then `and N more`. A group that ends up with a single update keeps the usual title of that update.

### Lock file maintenance

Each package manager gets its own pull request (`renovate/lock-file-maintenance-<manager>`), so one lock file failing
to update never holds back another:

| Title | Managers |
| --- | --- |
| `update python lock file` | `pep621` (PDM, uv), `poetry`, `pipenv`, `pip-compile` |
| `update javascript lock file` | `npm` (npm, pnpm, Yarn), `bun` |
| `update <manager> lock file` | Every other manager |

### Python interpreter

Upgrading the interpreter (`requires-python`, the `python` Docker image) needs work around it: the Docker
`PYTHON_VERSION`, the type checker's target, a new lock file that Renovate cannot generate for PDM. These updates wait
on the dependency dashboard until someone ticks them. Patch releases and digest refreshes of the image are not held.

### Annotated versions

A version in a workflow or action file that no manager knows about is tracked with a comment on the line above it:

```yaml
- uses: nicebots-xyz/licensor/.github/actions/setup-licensor@<digest> # v0.1.2
  with:
    # renovate: datasource=github-releases depName=nicebots-xyz/licensor
    version: v0.1.2
```

`packageName=` and `versioning=` can follow `depName=`. The setup-licensor action and the licensor release it installs
are grouped, so they always move together.

### Automerge

`platformAutomerge` is off everywhere, so Renovate merges a pull request itself only once every status check on the
branch has passed. It doesn't rely on GitHub auto-merge, which only waits for the checks that branch protection marks
as required.

Automerging is opt-in per repository, through `automerge-python-lockfile` and `automerge-low-risk`: extend them where
the checks are trusted to catch a broken update.

Commits pushed by [autofix.ci](https://autofix.ci) or by [nicebots-xyz/fixer](https://github.com/nicebots-xyz/fixer)
(the NiceBots Autofixer app), e.g. reformatting after a formatter bump, are listed in `gitIgnoredAuthors`. Renovate
still treats the branch as its own: it keeps updating it and can still automerge it. When Renovate rebases, it drops
the fix commit and the fixer applies it again.
