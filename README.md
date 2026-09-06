<img src="https://raw.githubusercontent.com/wolfware-labs/setup-moonlit/main/assets/moonlit.png" alt="Moonlit" width="88">

# moonlit-workflows

Reusable GitHub Actions workflows for [Moonlit](https://github.com/wolfware-labs/moonlit) release
pipelines.

```yaml
name: Release

on:
  push:
    branches: [main]

jobs:
  release:
    uses: wolfware-labs/moonlit-workflows/.github/workflows/release.yml@v1
    secrets: inherit
```

That is the whole caller. The workflow checks out with full history, installs the CLI, and runs
your pipeline.

## Why use these instead of the action directly

[`setup-moonlit`](https://github.com/wolfware-labs/setup-moonlit) installs the CLI and stops there,
by design — it composes with any job you write. These workflows go further and own the parts of the
job that are easy to get wrong:

- **`fetch-depth: 0`.** Moonlit reads commit history to decide the next version. A shallow clone
  does not fail; it silently produces the wrong version.
- **`permissions:`.** `contents: write` for release, `contents: read` for validate.
- **The pipeline's environment.** Secrets reach the pipeline as environment variables without you
  writing the mapping.

Use the action directly when you need a job shape these do not cover. Use these when you want the
defaults to be right.

## `release.yml`

Runs a pipeline. Requires `permissions: contents: write`, which the workflow declares itself.

| Input | Default | Description |
|---|---|---|
| `moonlit-version` | `latest` | `latest`, or an exact version such as `1.2.0` |
| `runs-on` | `ubuntu-latest` | Runner label |
| `config` | *(empty)* | Pipeline file. Empty uses the CLI default (`release.yml` / `release.yaml`) |
| `stage` | *(empty)* | Comma-separated stages. Empty runs every stage |
| `working-directory` | *(empty)* | Directory to run from. Empty uses the repository root |

```yaml
jobs:
  release:
    uses: wolfware-labs/moonlit-workflows/.github/workflows/release.yml@v1
    secrets: inherit
    with:
      moonlit-version: '1.2.0'
      stage: build,publish
```

## `validate.yml`

Parses the pipeline and resolves plugins without executing anything. Useful on pull requests that
change the pipeline.

| Input | Default | Description |
|---|---|---|
| `moonlit-version` | `latest` | `latest`, or an exact version such as `1.2.0` |
| `runs-on` | `ubuntu-latest` | Runner label |
| `config` | *(empty)* | Pipeline file. Empty uses the CLI default |
| `working-directory` | *(empty)* | Directory to validate from |

```yaml
name: Validate pipeline

on:
  pull_request:
    paths: ['release.yml', 'release.yaml']

jobs:
  validate:
    uses: wolfware-labs/moonlit-workflows/.github/workflows/validate.yml@v1
```

## Secrets

A reusable workflow cannot forward a secret whose name it does not know. `secrets: inherit` makes
your secrets readable inside the workflow, but each one still has to be mapped into the environment
explicitly — so these workflows forward a fixed set:

| Variable | Used by |
|---|---|
| `GITHUB_TOKEN` | GitHub plugin: releases, PRs, issues |
| `DOCKER_USER`, `DOCKER_PASSWORD` | Docker plugin: registry login |
| `OPENAI_API_KEY` | semantic-release plugin: AI changelog refinement |
| `MOONLIT_TOKEN` | Private plugin registries |

A secret you have not defined resolves to an empty string, so the extra names cost nothing. If your
pipeline needs a variable that is not on this list, open an issue — or call
[`setup-moonlit`](https://github.com/wolfware-labs/setup-moonlit) directly and write the job
yourself.

## Versioning

`@v1` tracks the latest v1 release and picks up fixes automatically. Pin an exact release
(`@v1.2.3`) or a commit SHA for byte-exact reproducibility.

## Licence

MIT OR Apache-2.0.
