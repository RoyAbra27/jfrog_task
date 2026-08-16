# Build & Publish: the OIDC workflow

`.github/workflows/publish-build.yml` (the seeded workflow) now runs green:
[run 31956714927](https://github.com/RoyAbra27/jfrog_task/actions/runs/31956714927).
No long-lived secrets are stored anywhere - the workflow authenticates to the
JFrog Platform with a short-lived OIDC token minted per run.

## What the workflow does

1. `setup-jfrog-cli@v4` exchanges the GitHub Actions OIDC token for a JFrog
   access token via the `jfrog-github-oidc` provider (no stored credentials).
2. `jf npm-config` + `jf npm install` resolve dependencies through the `npm`
   virtual repository, so every package the build consumes is proxied and
   recorded by Artifactory.
3. Docker Buildx builds the image for linux/amd64 and linux/arm64 and pushes
   `user-management-service:latest` to `docker-local`, authenticating with the
   OIDC-derived user and token.
4. `jf rt build-docker-create` attaches the pushed image (by digest) to the
   build, then `build-collect-env` + `build-add-git` + `build-publish` publish
   Build Info `Workflow-Task-seed` to Artifactory - environment, git commit,
   and the image digest, all traceable from the UI.

## Platform-side setup (one-time)

- OIDC provider `jfrog-github-oidc`: type GitHub, issuer
  `https://token.actions.githubusercontent.com`, organization `RoyAbra27`.
- Identity mapping `github-jfrog-task`: claim `repository = RoyAbra27/jfrog_task`
  maps to a platform user with `applied-permissions/user` scope and a 2h token.
  Scoping the mapping to this exact repository means a workflow in any other
  repo - even mine - gets nothing from this provider.
- Repository variables on the fork: `JF_URL=royab.jfrog.io`,
  `NPM_VIRTUAL_REPO=npm`, `DOCKER_REPO=docker-local`.

## Changes to the seeded workflow

- Filled the two placeholders (`DOCKER_REPO`, `NPM_VIRTUAL_REPO`) from repo
  variables and set `IMAGE_NAME` to `user-management-service` to match the
  images already in `docker-local`.
- `jf npm ci` -> `jf npm install`: the first run failed because this repo
  ships no `package-lock.json`. Committing a modern lockfile would not help
  the image either - the Dockerfile's `node:14-alpine` carries npm 6, which
  cannot read lockfileVersion 2+ files, so the runner would be pinned while
  the shipped image ignored it. `npm install` is the honest choice for this
  repo as it stands.

## Verified results

- Build Info `Workflow-Task-seed` visible at
  `/artifactory/api/build/Workflow-Task-seed` (published 2026-08-16).
- `docker-local/user-management-service` tags: `baseline`, `fixed` (the
  Stage 1 scan-and-remediate images) and `latest` (this workflow's push).

## Known warning

GitHub annotates the run because the pinned actions (`checkout@v4`,
`setup-jfrog-cli@v4`, the docker actions) target Node.js 20, which the
runners are deprecating. They still run (forced onto Node 24). Bumping the
action versions is upstream's call for a seeded workflow; noted rather than
changed to keep this fork's diff purposeful.
