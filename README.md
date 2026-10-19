# CleverQA Release

Declares a release to [CleverQA](https://cleverqa.com) from your CI, so crashes and incidents that follow are attributed to it.

```yaml
- name: Declare release to CleverQA
  uses: clementina-enterprises/cleverqa-release-action@v1
  with:
    api-token: ${{ secrets.CLEVERQA_TOKEN }}
    project: my-project
    version: ${{ github.ref_name }}
    platform: backend
```

## Full example

Declare the release when a version tag is pushed, after the build and deploy have succeeded:

```yaml
name: release

on:
  push:
    tags: ['v*.*.*']

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: make test
      - run: make deploy

      - name: Declare release to CleverQA
        id: cleverqa
        uses: clementina-enterprises/cleverqa-release-action@v1
        with:
          api-token: ${{ secrets.CLEVERQA_TOKEN }}
          project: my-project
          version: ${{ github.ref_name }}   # v1.4.2 is sent as 1.4.2
          build-number: ${{ github.run_number }}

      - run: echo "CleverQA release ${{ steps.cleverqa.outputs.release-id }}"
```

Declaring the same project, version and platform again updates the existing release instead of creating a duplicate, so re-running a job is safe.

## Inputs

| Name | Description | Required | Default |
|---|---|---|---|
| `api-token` | CleverQA API token (`tok_...`) with the `releases:write` scope | yes | |
| `project` | Project slug, as it appears in the project URL | yes | |
| `version` | Semver version, e.g. `1.4.2` or `2.0.0-beta.1`. One leading `v` is stripped. | yes | |
| `platform` | `ios`, `android` or `backend`. Must be a platform enabled on the project. | no | `backend` |
| `build-number` | Build number | no | |
| `status` | `draft`, `testing`, `ready`, `released` or `rolled_back` | no | `released` |
| `notes` | Release notes, up to 5000 characters | no | |
| `api-url` | CleverQA base URL. Only change this for self-hosted. | no | `https://cleverqa.com` |

## Outputs

| Name | Description |
|---|---|
| `release-id` | The CleverQA release ID |

## Failures

The step fails, and the job with it, whenever the release is not recorded. The log shows the HTTP status and CleverQA's reason; the token is never printed.

| Status | Usual cause |
|---|---|
| 400 | Version is not semver, or the platform is not enabled on the project |
| 401 | Token is wrong, revoked or expired |
| 402 | The workspace is on the Free plan; declaring releases needs Essentials or above |
| 403 | Token is missing the `releases:write` scope |
| 404 | No project with that slug in the token's workspace |

If declaring a release must never block your deploy, add `continue-on-error: true` to the step.

The action needs `bash`, `curl` and `jq`, which GitHub-hosted runners include. On a self-hosted runner without them the step fails with a message naming what is missing.

## Where do I get a token?

In CleverQA, a workspace admin opens **Settings → API tokens**, creates a token with the `releases:write` scope, and copies it. It is shown once. Store it as a repository or organisation secret, e.g. `CLEVERQA_TOKEN`.

## What is CleverQA?

CleverQA turns monitor incidents, crash spikes, server saturation, load-test breakpoints and release regressions into one tracked defect with a likely cause attached. Declaring releases is what lets crashes and incidents attribute back to the release that caused them. See [cleverqa.com](https://cleverqa.com).

## Releasing this action

Users pin `@v1`, so every release moves the major tag:

```bash
git tag -a v1.0.1 -m "Fix ..." && git push origin v1.0.1
git tag -f v1 v1.0.1 && git push -f origin v1
```

A change that breaks existing inputs or outputs goes in `v2` instead.

## Licence

MIT. See [LICENSE](LICENSE).
