# homelab-ci

A reusable GitHub Actions workflow: build a multi-arch Docker image and deploy it through [homelab-broker](https://github.com/eayanwale/homelab-broker).

On every push, it builds your image for `linux/amd64,linux/arm64`, pushes it to `ghcr.io` tagged with the commit SHA and `main`, then calls the broker's `POST /api/instances/{id}/deploy` with that SHA and waits for the redeploy to actually finish (including the broker's own automatic rollback if the new image fails its health check).

## Use it

1. Your project needs a `Dockerfile` that builds and serves your app (see `examples/` for a static site and a Next.js starting point).
2. On the broker: launch the instance once by hand (any image is fine as a placeholder), then open its **Deploy** tab and create a deploy token. Copy it.
3. In your repo's GitHub settings, add a secret `BROKER_DEPLOY_TOKEN` with that value.
4. Add a caller workflow:

```yaml
# .github/workflows/deploy.yml
name: deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: eayanwale/homelab-ci/.github/workflows/build-and-deploy.yml@main
    with:
      image: ghcr.io/eayanwale/${{ github.event.repository.name }}
      instance_id: "123"  # the instance id from its detail page URL
    secrets:
      BROKER_DEPLOY_TOKEN: ${{ secrets.BROKER_DEPLOY_TOKEN }}
```

That's it -- push to `main` and it builds, pushes, and deploys.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `image` | yes | -- | Full image reference to build and push, e.g. `ghcr.io/eayanwale/my-app` |
| `instance_id` | yes | -- | The homelab-broker instance id to deploy to |
| `dockerfile` | no | `Dockerfile` | Path to the Dockerfile |
| `context` | no | `.` | Build context |
| `platforms` | no | `linux/amd64,linux/arm64` | Comma-separated platforms to build for |
| `broker_url` | no | `https://lab.knoch.dev` | Base URL of the broker instance |

## Secrets

| Secret | Required | Description |
|---|---|---|
| `BROKER_DEPLOY_TOKEN` | yes | A deploy token scoped to the one instance you're deploying to (create it on that instance's Deploy tab) |

`GITHUB_TOKEN` is used automatically for the `ghcr.io` push (your repo's own, standard permission -- no extra secret needed for that part).

## Why a new image repo won't deploy

A deploy token is scoped to one instance *and* its existing image repo -- the broker rejects a tag for a different image entirely. The instance has to already exist (launched once by hand, or via the broker's own API) before CI can redeploy it; this workflow updates an existing instance's image, it doesn't create new instances.

## Out of scope

Per-branch preview deployments, building on the broker's own hosts (this always builds on the GitHub-hosted runner), and stacks (multi-container apps) -- deploy tokens are scoped to a single instance today.
