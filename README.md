# gh-actions

## Actions

### require-release-tip

Ensures a tag points to the **tip** of a release branch and is **not** a merge commit.

Usage (recommended: pin to SHA):

```yaml
- uses: 0r4m3r/gh-actions/require-release-tip@dde7f9b094c1abd78c58f1975b27cc953b62dba7
  with:
    release_branch: mcr/release
```

Optional inputs:
- `release_branch` (default: `mcr/release`)
- `remote` (default: `origin`)

Security notes:
- Pin to a full commit SHA for supply-chain safety.
- Keep this repo free of secrets or internal endpoints.
