# ECS Deployment

This fork can deploy `9router` to the ECS host behind `https://9router.gnim.top` through a manually triggered GitHub Actions workflow.

## What the workflow does

- Builds the production standalone app on GitHub Actions.
- Packages the build output as a tarball.
- Uploads the tarball to the ECS host over SSH.
- Extracts it into `/home/mannix/apps/9router-releases/<release>`.
- Switches `/home/mannix/apps/9router-runtime` to the new release.
- Restarts the app and checks `http://127.0.0.1:20128/api/health`.
- Attempts to roll back to the previous runtime if the health check fails.

The workflow does not run `npm install` or `next build` on the ECS host.

## Data safety

Persistent application data is expected to remain outside the release directory:

- Data directory: `/home/mannix/.9router`
- Runtime env file: `/home/mannix/apps/9router-shared/.env`

The deploy workflow only writes under these app deployment paths:

- `/home/mannix/apps/9router-shared/incoming`
- `/home/mannix/apps/9router-releases`
- `/home/mannix/apps/9router-runtime`
- `/home/mannix/apps/9router-deploy`

## Required repository secret

Create this repository secret before running the workflow:

- `ECS_SSH_KEY`: private SSH key that can log in as `mannix` on `106.14.193.227`

The matching public key must be present in `/home/mannix/.ssh/authorized_keys` on the ECS host.

## Manual deployment

1. Open this repository on GitHub.
2. Go to `Actions`.
3. Select `Deploy 9router to ECS`.
4. Click `Run workflow`.
5. Use `master`, a tag, or a commit SHA as `ref`.
6. Keep `keep_releases` at `5` unless disk cleanup needs a different value.

## Rollback

The workflow keeps old releases under `/home/mannix/apps/9router-releases`. If automatic rollback is not enough, point `/home/mannix/apps/9router-runtime` back to an older release and run:

```bash
/home/mannix/apps/9router-deploy/run-9router.sh
```
