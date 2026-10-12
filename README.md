# Deploy with Teleport

GitHub Action that deploys a Docker Compose stack to a remote server through [Teleport](https://goteleport.com/). The runner authenticates to Teleport with short-lived certificates, copies `docker-compose.yml` (and an optional `.env`) to the server, logs in to Amazon ECR and runs `docker compose up`. The server needs no SSH port open to the internet.

## Usage

```yaml
name: Deploy

on:
  push:
    branches: [develop, main]

permissions:
  id-token: write # Teleport and AWS both authenticate with GitHub OIDC
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: anilrajrimal1/deploy-with-teleport@v1
        with:
          TELEPORT_HOST: teleport.example.com:443
          TELEPORT_TOKEN: github-deploy # name of the Teleport join token
          SSH_USER: deploy
          SSH_HOSTNAME: app-server-1
          BRANCH_NAME: ${{ github.ref_name }}
          PROJECT_DIR: /srv/app
          AWS_ROLE_ARN: arn:aws:iam::123456789012:role/github-deploy
          AWS_REGION: ap-south-1
          ECR_REGISTRY: 123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

The action checks out `BRANCH_NAME` and deploys `docker-compose.<branch>.yml` from `COMPOSE_FILE_PATH`, so `develop` deploys `./docker/server/docker-compose.develop.yml`.

### Secrets from Phase (optional)

Set `PHASE_SERVICE_TOKEN` to write a `.env` from [Phase](https://phase.dev) before deploying. It uses [phase-secrets-fetch-action](https://github.com/anilrajrimal1/phase-secrets-fetch-action).

```yaml
          PHASE_SERVICE_TOKEN: ${{ secrets.PHASE_SERVICE_TOKEN }}
          PHASE_APP_ID: ${{ secrets.PHASE_APP_ID }}
          PHASE_ENV: ${{ github.ref_name }}
```

Without Phase, any `.env` already in the workspace (for example, written by an earlier step) is copied instead. If there is none, the `.env` copy is skipped.

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `TELEPORT_HOST` | yes | | Teleport proxy, e.g. `teleport.example.com:443` |
| `TELEPORT_TOKEN` | yes | | Name of the Teleport join token (GitHub join method) |
| `TELEPORT_VERSION` | no | `14.3.19` | Teleport CLI version to install |
| `SSH_USER` | yes | | Login on the target node |
| `SSH_HOSTNAME` | yes | | Target node's Teleport hostname (not its IP) |
| `BRANCH_NAME` | yes | | Branch to check out; selects `docker-compose.<branch>.yml` |
| `PROJECT_DIR` | yes | | Directory on the server to deploy into |
| `COMPOSE_FILE_PATH` | no | `./docker/server/` | Directory holding the compose files |
| `ECR_REGISTRY` | yes | | ECR registry URL |
| `AWS_REGION` | no | `ap-south-1` | Region for ECR login |
| `AWS_ROLE_ARN` | no | | IAM role to assume with GitHub OIDC (recommended) |
| `AWS_ACCESS_KEY_ID` | no | | Access key, only if you can't use OIDC |
| `AWS_SECRET_ACCESS_KEY` | no | | Secret key, only if you can't use OIDC |
| `PHASE_SERVICE_TOKEN` | no | | Phase service token; leave empty to skip Phase |
| `PHASE_APP_ID` | no | | Phase app ID |
| `PHASE_ENV` | no | | Phase environment |
| `PHASE_HOST` | no | `https://console.phase.dev` | Only for self-hosted Phase |
| `ENV_FILE` | no | `.env` | `.env` file to copy to the server, if present |

## Outputs

| Output | Description |
| --- | --- |
| `status` | `success` when the stack is up |

## Setup

**Teleport.** Create a join token with the `github` join method that allows your repository, and a bot whose role can log in as `SSH_USER` on the target node:

```yaml
kind: token
version: v2
metadata:
  name: github-deploy
spec:
  roles: [Bot]
  bot_name: github-deploy
  join_method: github
  github:
    allow:
      - repository: your-org/your-repo
```

**AWS.** Create an IAM role that trusts GitHub's OIDC provider for your repository, with permission to pull from ECR (`ecr:GetAuthorizationToken`, `ecr:BatchGetImage`, `ecr:GetDownloadUrlForLayer`).

**Server.** Docker with the Compose plugin, and `PROJECT_DIR` writable by `SSH_USER`.

## Security notes

- No long-lived SSH keys: the runner gets a Teleport certificate that expires after 2 hours, and every session is recorded by Teleport.
- No long-lived AWS keys when you use `AWS_ROLE_ARN`.
- The ECR password is masked in logs and sent to the server over stdin, never as a command-line argument.
- Inputs reach shell steps through environment variables and are quoted before running on the server, so values such as branch names can't inject commands.

## License

[MIT](LICENSE)
