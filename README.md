# GitHub Actions

Centralized repository for reusable GitHub Actions.

Two kinds of thing live here:

- **Composite actions** (`<name>/action.yml`) — steps you drop into a job you already own.
- **Reusable workflows** (`.github/workflows/<name>.yml`) — whole jobs you call with `uses:` at the job level. Reach for these when the work needs its own runner, a matrix, or several jobs.

All third-party actions are pinned to commit SHAs with the version in a trailing comment. `.github/dependabot.yml` keeps them moving.

Ready-to-copy caller workflows live in [`examples/`](./examples).

## Reusable workflows

### [docker-build-push](./.github/workflows/docker-build-push.yml)

Build one image and push it. The default case.

```yaml
jobs:
  image:
    permissions:
      contents: read
      packages: write
    uses: mvaldes14/gh-actions/.github/workflows/docker-build-push.yml@main
    with:
      image: mvaldes14/myapp
```

`packages: write` on the caller's job is required. A reusable workflow can only narrow the `GITHUB_TOKEN` it is handed, never widen it, so declaring the permission inside the workflow is not enough.

Pushes are suppressed automatically on `pull_request`.

### [docker-multiarch](./.github/workflows/docker-multiarch.yml)

Build a real multi-arch manifest: one runner per platform, pushed by digest, then a merge job stitches them into a single tagged manifest list.

```yaml
jobs:
  image:
    permissions:
      contents: read
      packages: write
    uses: mvaldes14/gh-actions/.github/workflows/docker-multiarch.yml@main
    with:
      image: mvaldes14/myapp
      platforms: '["linux/amd64","linux/arm64"]'
```

Prefer this over passing two platforms to `docker-build-push`. That path emulates the foreign architecture under QEMU inside one job and serialises the builds, which for anything that compiles is several times slower.

### [discover-dockerfiles](./.github/workflows/discover-dockerfiles.yml)

Emit every directory holding a `Dockerfile` as a JSON array for `strategy.matrix`, so a monorepo's build matrix stops going stale when someone adds a project. See [`examples/monorepo-images.yml`](./examples/monorepo-images.yml).

### [go-ci](./.github/workflows/go-ci.yml)

`test` and `lint` as two separate jobs, so a formatting nit does not mask a test failure.

```yaml
jobs:
  go:
    uses: mvaldes14/gh-actions/.github/workflows/go-ci.yml@main
```

## Actions

### [docker-build](./docker-build)

Build a container image with Buildx and push it to a registry. Buildx gives layer caching via the GitHub Actions cache; `docker/metadata-action` derives tags and OCI labels.

**Usage:**

```yaml
- uses: mvaldes14/gh-actions/docker-build@main
  with:
    image: mvaldes14/myapp
    password: ${{ secrets.GITHUB_TOKEN }}
```

`registry` defaults to `ghcr.io` and `username` to `github.actor`.

`tags` is a `docker/metadata-action` tag spec, one directive per line — not a comma-separated list of literal tags. The default covers branch, tag, PR, short SHA, and `latest` on the default branch. For a literal tag:

```yaml
    tags: |
      type=raw,value=v1.2.3
      type=raw,value=latest
```

There is no need to shell out to `date` for a timestamp tag; use `type=raw,value={{date 'YYYYMMDD-HHmmss'}}`.

Give each image its own `cache-scope` so concurrent builds do not evict each other's layers.

Outputs: `digest`, `image-id`, `tags`, `version`, `labels`.

### [go-ci](./go-ci)

Download modules, check `go.mod`/`go.sum` are tidy, `go vet`, `go build`, then `go test` with the race detector and a coverage summary in the run summary.

**Usage:**

```yaml
- uses: mvaldes14/gh-actions/go-ci@main
  with:
    working-directory: .
```

The Go version is read from `go.mod` by default, so CI cannot drift from the module's declared toolchain. Set `go-version` to override.

### [gotify-notification](./gotify-notification)

Send event notifications to a Gotify server.

**Usage:**

```yaml
- uses: mvaldes14/gh-actions/gotify-notification@main
  with:
    gotify-url: ${{ secrets.GOTIFY_URL }}
    gotify-token: ${{ secrets.GOTIFY_TOKEN }}
    title: "Deploy Complete"
    message: "Deployed ${{ github.repository }} @ ${{ github.sha }}"
```

### [vault-secrets](./vault-secrets)

Pull secrets from HashiCorp Vault and export them as environment variables or outputs. Supports token, AppRole, and GitHub auth methods.

**Usage:**

```yaml
- uses: mvaldes14/gh-actions/vault-secrets@main
  id: secrets
  with:
    vault-url: ${{ secrets.VAULT_URL }}
    auth-method: approle
    role-id: ${{ secrets.VAULT_ROLE_ID }}
    secret-id: ${{ secrets.VAULT_SECRET_ID }}
    secrets: |
      secret/data/myapp db_password | DB_PASSWORD
      secret/data/myapp api_key | API_KEY
```

### [golint](./golint)

Run golangci-lint on a Go project.

**Usage:**

```yaml
- uses: mvaldes14/gh-actions/golint@main
  with:
    go-version: "1.23"
    args: "--timeout 5m"
```
