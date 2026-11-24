# superheavy-dev/setup

Gives git-lfs access to a repository's LFS objects on Superheavy from a GitHub Actions job. The job's GitHub OIDC token is exchanged for a short-lived credential, so the workflow needs no secret.

```yaml
permissions:
  contents: read
  id-token: write

steps:
  - uses: actions/checkout@v6
  - uses: superheavy-dev/setup@v1
```

The action reads the LFS URL from `.lfsconfig`, or from `.lfsconfig` on origin's default branch when the checkout has none, configures git-lfs for the rest of the job, and runs `git lfs pull`. Later steps can run `git lfs push`, `git lfs fetch` or `git lfs lock` with no extra setup.

The action keeps the checkout's LFS objects in the Actions cache. A later run of the same commit downloads nothing, and a run of a later commit downloads only the objects that changed. Each object restored from the cache must match its oid, and git-lfs downloads any object that doesn't.

Jobs started by `push`, `workflow_dispatch`, `schedule`, `merge_group`, `release` and `repository_dispatch` can upload. Jobs started by any other event, pull requests included, can only download.

## Inputs

| Input       | Default                   | Description                                                                                                                                                                                                                                                                                                                                                    |
| ----------- | ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `path`      | `.`                       | Path of the repository's working tree, relative to the workspace.                                                                                                                                                                                                                                                                                              |
| `url`       | `lfs.url` in `.lfsconfig` | Superheavy LFS URL. It must use https, except on a loopback host. An `lfs.url` from `.lfsconfig` must also be on `superheavy.dev`, so a changed `.lfsconfig` can't send the job's OIDC token to another host. A checkout without `.lfsconfig`, such as a tag from before the repository moved to Superheavy, uses the `.lfsconfig` on origin's default branch. |
| `pull`      | `true`                    | Run `git lfs pull` after setup.                                                                                                                                                                                                                                                                                                                                |
| `audience`  | `superheavy.dev`          | OIDC audience that the Superheavy server accepts.                                                                                                                                                                                                                                                                                                              |
| `cache`     | `true`                    | Keep LFS objects in the Actions cache.                                                                                                                                                                                                                                                                                                                         |
| `telemetry` | `true`                    | Send Superheavy a report of the run. See [Telemetry](#telemetry).                                                                                                                                                                                                                                                                                              |

## Outputs

| Output         | Description                                                                  |
| -------------- | ---------------------------------------------------------------------------- |
| `scope`        | `read` or `write`, the access the job has to the repository.                 |
| `cache-hits`   | LFS objects of the checkout that were on the runner or in the Actions cache. |
| `cache-misses` | LFS objects of the checkout that git-lfs downloaded.                         |

## Telemetry

After `git lfs pull`, the action sends Superheavy one report of the run: the action version, the runner's OS, architecture and environment, whether the run succeeded, how long the OIDC exchange, cache restore, `git lfs pull` and cache save took, the number and total size of the checkout's LFS objects, cache hits and misses, and the size, download time and concurrency of up to 200 downloaded objects. The report carries no file names, paths, object ids, refs or commit SHAs. Superheavy uses it to measure CI download speed and the action's reliability.

Set the `telemetry` input to `false`, or set `SUPERHEAVY_TELEMETRY=0` or `DO_NOT_TRACK=1` in the job's environment, to turn the report off.
