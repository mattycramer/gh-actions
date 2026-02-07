# gh-actions

## Structure

- `actions/` — shared composite actions (recommended paths)
- `.github/workflows/` — reusable workflows + self-tests

## Actions

### bws-fetch (gh-actions-shared)

Fetch secrets from Bitwarden Secrets Manager (BWS) into files and export `*_FILE`
environment variables for downstream steps. This repo consumes the shared action
from `shared-common/gh-actions-shared`.

Usage:

```yaml
- uses: shared-common/gh-actions-shared/.github/actions/bws-fetch@9ab134ec04b39da8a91b39df5911048f7ae00c38
  with:
    access-token: ${{ secrets.BWS_ACCESS_TOKEN }}
    project-id: ${{ secrets.BWS_PROJECT_ID }}
    secrets: GH_ORG_RELEASE,GH_ORG_RELEASE_APP_ID,GH_ORG_RELEASE_APP_INSTALL_ID,GH_ORG_RELEASE_APP_PEM
    bws-version: ${{ vars.BWS_VERSION }}
    bws-sha256: ${{ vars.BWS_SHA256 }}
```

### github-app-token

Creates a GitHub App installation token using secrets stored in BWS.

Usage:

```yaml
- uses: emcram/gh-actions/actions/github-app-token@<sha>
  with:
    bws-access-token: ${{ secrets.BWS_ACCESS_TOKEN }}
    bws-project-id: ${{ secrets.BWS_PROJECT_ID }}
    bws-version: ${{ vars.BWS_VERSION }}
    bws-sha256: ${{ vars.BWS_SHA256 }}
```

### require-release-tip

Ensures a tag points to the **tip** of a release branch; optionally requires it to **not** be a merge commit.

Usage (recommended: pin to SHA):

```yaml
- uses: emcram/gh-actions/actions/require-release-tip@dde7f9b094c1abd78c58f1975b27cc953b62dba7
  with:
    release_branch: mcr/release
```
```

Optional inputs:
- `release_branch` (default: `mcr/release`)
- `remote` (default: `origin`)
- `allow_merge_commits` (default: `false`)

Security notes:
- Pin to a full commit SHA for supply-chain safety.
- Keep this repo free of secrets or internal endpoints.

### require-gl-tag

Ensures a workflow runs only on `gl-*` tags (optionally validates a regex).

Usage:

```yaml
- uses: emcram/gh-actions/actions/require-gl-tag@<sha>
  with:
    tag_prefix: gl-
    tag_regex: '^gl-your-project-v[0-9]+\.[0-9]+\.[0-9]+(-r[0-9]+)?$'
```

### validate-tag-version

Validates a release tag and (optionally) matches it to an expected version.

Usage:

```yaml
- uses: emcram/gh-actions/actions/validate-tag-version@<sha>
  with:
    tag_prefix: gl-your-project-v
    expected_version: ${{ steps.version.outputs.package_version }}
    allow_revision_suffix: "true"
```

Notes:
- `tag_prefix` is required and must start with `gl-`.
- This enables a shared policy: **only `gl-*` tags can build releases**.

## Reusable workflows

### release-from-workflow-run

Creates a GitHub release from artifacts produced by a prior workflow run.

Usage:

```yaml
jobs:
  release:
    uses: emcram/gh-actions/.github/workflows/release-from-workflow-run.yml@<sha>
    with:
      tag: ${{ github.event.workflow_run.head_branch }}
      tag_prefix: gl-your-project-v
      run_id: ${{ github.event.workflow_run.id }}
      artifact_prefix: your-project-linux-x86_64-
      tarball_prefix: your-project-linux-x86_64-
      checksum_prefix: your-project-sha256-checksums-
      release_name: your-project v{version}
    secrets: inherit
```

Notes:
- Requires `BWS_ACCESS_TOKEN` and `BWS_PROJECT_ID` secrets (and the GitHub App secrets in BWS).
- `bws_version`/`bws_sha256` default to `vars.BWS_VERSION` and `vars.BWS_SHA256` when unset.
- Deletes workflow artifacts and caches for the referenced `run_id` and tag ref (logs are retained).
