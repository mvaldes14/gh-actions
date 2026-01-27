# GitHub Actions

Centralized repository for reusable GitHub Actions.

## Actions

### [docker-build](./docker-build)

Build a container image and push it to a registry.

**Usage:**

```yaml
- uses: mvaldes14/gh-actions/docker-build@main
  with:
    registry: ghcr.io
    image: myorg/myapp
    tags: "latest,${{ github.sha }}"
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
```

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
