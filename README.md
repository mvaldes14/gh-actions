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
