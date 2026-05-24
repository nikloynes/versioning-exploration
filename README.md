# versioning-exploration

automated semantic versioning for a uv-managed python project, using conventional commits, pre-commit hooks, and python-semantic-release.

## tools

- [uv](https://docs.astral.sh/uv/) - python package manager
- [pre-commit](https://pre-commit.com/) - git hook management
- [conventional commits](https://www.conventionalcommits.org/) - commit message standard
- [python-semantic-release](https://python-semantic-release.readthedocs.io/) - automated versioning and changelog generation

## setup (once per contributor, after cloning)

1. install dependencies:
    ```bash
    uv sync
    ```
2. install git hooks to check msgs
    ```bash
    uv run pre-commit install --hook-type commit-msg
    ```

## commit message format

all commits must use [**conventional commits**](conventionalcommits.org). the pre-commit hook will reject any commit that doesn't.

| prefix | example | version effect |
|---|---|---|
| `fix:` | `fix: handle null input` | patch: `0.1.0 -> 0.1.1` |
| `feat:` | `feat: add login page` | minor: `0.1.0 -> 0.2.0` |
| `feat!:` or `BREAKING CHANGE:` footer | `feat!: remove legacy api` | major: `0.1.0 -> 1.0.0` |
| `chore:`, `docs:`, `ci:`, `test:` | `chore: update deps` | no bump |

## branch and versioning flows

### working on a feature

```bash
git checkout -b feat/my-feature    # branch from main or develop
# make changes
git commit -m "feat: add my feature"    # hook validates the message format
git push origin feat/my-feature
# open a pr to develop or main
```

### pr merged into develop

ci runs python-semantic-release on the merged commit history and:

- determines the version bump from commit prefixes (`fix:` -> patch, `feat:` -> minor, etc.)
- writes the pre-release version to `pyproject.toml` (e.g. `0.2.0.dev1`)
- commits that change as `chore(release): 0.2.0.dev1 [skip ci]` and pushes it
- creates git tag `v0.2.0.dev1`

subsequent merges to develop increment the dev counter: `0.2.0.dev2`, `0.2.0.dev3`, and so on.

### pr merged into main

ci runs python-semantic-release and:

- strips the pre-release suffix to produce the canonical version (e.g. `0.2.0`)
- writes it to `pyproject.toml`
- creates git tag `v0.2.0`
- creates a github release and updates `CHANGELOG.md`

canonical versions only exist on `main`.

## checking the version

from code:

```python
from importlib.metadata import version
print(version("versioning-exploration"))
```

or check pyproject.toml directly - it is always up to date after a ci release run.

## notes

- use `uv run semantic-release version --noop` to preview what a release would do locally. never run it without `--noop`.
- the `[skip ci]` tag in release commits prevents the workflow from triggering itself in a loop.
- if a merge contains only `chore:`, `docs:`, or `test:` commits, no version bump or release is created.
- do not create or move git tags manually. let ci own them entirely.
