# gh-actions

## Structure

- `actions/` — shared composite actions (recommended paths)
- `require-release-tip/` — compatibility shim (kept for backward compatibility)
- `.github/workflows/` — self-tests for shared actions

## Actions

### require-release-tip

Ensures a tag points to the **tip** of a release branch; optionally requires it to **not** be a merge commit.

Usage (recommended: pin to SHA):

```yaml
- uses: emcram/gh-actions/actions/require-release-tip@dde7f9b094c1abd78c58f1975b27cc953b62dba7
  with:
    release_branch: mcr/release

Compatibility path (legacy):

```yaml
- uses: emcram/gh-actions/require-release-tip@dde7f9b094c1abd78c58f1975b27cc953b62dba7
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
    tag_regex: '^gl-codex-rs-v[0-9]+\.[0-9]+\.[0-9]+(-r[0-9]+)?$'
```

### validate-tag-version

Validates a release tag and (optionally) matches it to an expected version.

Usage:

```yaml
- uses: emcram/gh-actions/actions/validate-tag-version@<sha>
  with:
    tag_prefix: gl-codex-rs-v
    expected_version: ${{ steps.version.outputs.package_version }}
    allow_revision_suffix: "true"
```

Notes:
- All tag validation enforces `gl-*` prefixes by default.
- This enables a shared policy: **only `gl-*` tags can build releases**.
