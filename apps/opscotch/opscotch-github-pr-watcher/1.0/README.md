# GitHub PR Watcher

Reusable Opscotch app that polls GitHub pull requests (via the issues API) and routes matched PRs to downstream deployment steps.

## What this app does

- Polls open PRs for one repo/assignee pair at a time.
- Filters/routes by label using `githubPrWatcherCriteria`.
- Calls `deploymentId` + `stepId` from the matched criterion.

## Bootstrap usage example

```json
[
  {
    "deploymentId": "github-pr-watcher",
    "remoteConfiguration": "workflow.json",
    "allowExternalHostAccess": [
      {
        "id": "github-api-auth",
        "host": "https://api.github.com",
        "authenticationHost": true,
        "data": {
          "githubToken": "REPLACE_ME_WITH_GH_TOKEN"
        }
      },
      {
        "id": "github-api",
        "host": "https://api.github.com",
        "httpTimeout": 15000,
        "allowList": [
          { "method": "GET", "uriPattern": "/search/issues.*" },
          { "method": "GET", "uriPattern": "/repos/.+/issues.*" },
          { "method": "GET", "uriPattern": "/repos/.+/issues/.+/comments.*" },
          { "method": "GET", "uriPattern": "/repos/.+/pulls/.+" }
        ]
      }
    ],
    "data": {
      "githubAuthHostId": "github-api-auth",
      "hostId": "github-api",
      "watchEntity": "pr",
      "issueHandoffDelaySeconds": 120,
      "githubPrWatcherCriteria": [
        {
          "label": "ready for dev",
          "assignee": "YOUR_GITHUB_USERNAME",
          "repo": "YOUR_ORG/YOUR_REPO",
          "deploymentId": "target-deployment-access-id",
          "stepId": "target-step-id"
        }
      ]
    }
  }
]
```

## Data contract

- `hostId` (string, optional): GitHub API host id, defaults to `github-api`.
- `watchEntity` (string, required): must be `pr`.
- `issueHandoffDelaySeconds` (number, optional): minimum PR age by `updated_at` before handoff.
- `issueWatcherDecisionLoggingEnabled` (boolean, optional): emits diagnostic matching logs.
- `githubPrWatcherCriteria` (array, required): routing criteria.

Each `githubPrWatcherCriteria[]` item requires:
- `label` (string)
- `assignee` (string)
- `repo` (string, `owner/repo`)
- `deploymentId` (string)
- `stepId` (string)

## Important nuance

- All criteria in a single deployment must share the same `repo` and `assignee` because polling is done with one upstream query and label selection happens in the results processor.
- This app routes only; it does not mutate pull requests.

## Release Links

- Release: https://github.com/opscotch/opscotch-apps/releases/tag/opscotch-github-pr-watcher-1.0
- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-github-pr-watcher

To use this as a docker image, here is an example Dockerfile:

```Dockerfile
FROM ghcr.io/opscotch/opscotch-github-pr-watcher:1.0 AS opscotch-github-pr-watcher
FROM ghcr.io/opscotch/opscotch-agent:latest

COPY --from=opscotch-github-pr-watcher /apps/opscotch-github-pr-watcher.oapp /apps/opscotch-github-pr-watcher

# your custom bootstrap
COPY bootstrap.json /config/bootstrap.json
```

## Signing Public Keys

| Key ID | Public Key |
| --- | --- |
| `opscotch-app` | `8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874` |
| `opscotch-github-pr-watcher-app` | `8869EED16EDAE0F7589B52DF43A9A9CA0E44619EFCC21CC9AE8402786D195421` |
| `opscotch-github-pr-watcher-1.x` | `696668A4A7633A00EEF7578BEE0778D1B5C37554762964186D2BC8FA3F04F947` |

## Verify Artifact

Download `opscotch-github-pr-watcher.oapp`, `opscotch-github-pr-watcher.oapp.sig`, and `opscotch-github-pr-watcher.oapp.pem` from the release, then run:

```bash
cosign verify-blob \
  --signature opscotch-github-pr-watcher.oapp.sig \
  --certificate opscotch-github-pr-watcher.oapp.pem \
  --certificate-identity-regexp '^https://github.com/opscotch/builder/.github/workflows/app-release.yml@refs/(heads|tags)/.+$' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  opscotch-github-pr-watcher.oapp
```
