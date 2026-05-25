# GitHub Issue Watcher

Reusable Opscotch app that polls GitHub issues and routes matched issues to downstream deployment steps.

## What this app does

- Polls open issues for one repo/assignee pair at a time.
- Filters/routs by label using `githubIssueWatcherCriteria`.
- Calls `deploymentId` + `stepId` from the matched criterion.

## Bootstrap usage example

```json
[
  {
    "deploymentId": "github-issue-watcher",
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
          { "method": "GET", "uriPattern": "/repos/.+/issues.*" },
          { "method": "GET", "uriPattern": "/repos/.+/issues/.+/comments.*" }
        ]
      }
    ],
    "data": {
      "githubAuthHostId": "github-api-auth",
      "hostId": "github-api",
      "issueHandoffDelaySeconds": 120,
      "githubIssueWatcherCriteria": [
        {
          "label": "triage",
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
- `issueHandoffDelaySeconds` (number, optional): minimum issue age by `updated_at` before handoff.
- `issueWatcherDecisionLoggingEnabled` (boolean, optional): emits diagnostic matching logs.
- `githubIssueWatcherCriteria` (array, required): routing criteria.

Each `githubIssueWatcherCriteria[]` item requires:
- `label` (string)
- `assignee` (string)
- `repo` (string, `owner/repo`)
- `deploymentId` (string)
- `stepId` (string)

## Important nuance

- All criteria in a single deployment must share the same `repo` and `assignee` because polling is done with one upstream query and label selection happens in the results processor.
- This app routes only; it does not mutate GitHub issues.

## Release Links

- Release: https://github.com/opscotch/opscotch-apps/releases/tag/opscotch-github-issue-watcher-1.0.1
- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-github-issue-watcher

To use this as a docker image, here is an example Dockerfile:

```Dockerfile
FROM ghcr.io/opscotch/opscotch-github-issue-watcher:1.0.1 AS opscotch-github-issue-watcher
FROM ghcr.io/opscotch/opscotch-agent:latest

COPY --from=opscotch-github-issue-watcher /apps/opscotch-github-issue-watcher.oapp /apps/opscotch-github-issue-watcher

# your custom bootstrap
COPY bootstrap.json /config/bootstrap.json
```

## Signing Public Keys

| Key ID | Public Key |
| --- | --- |
| `opscotch-app` | `8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874` |
| `opscotch-github-issue-watcher-app` | `554C745A08B5991A169B7C8D6D7193B6E9B9A2B51C02DE1ED2072670E9B5530A` |
| `opscotch-github-issue-watcher-1.x` | `BBA0114A4648A246CD6312CC7757829240BADF991902F7348438373485939C49` |

## Verify Artifact

Download `opscotch-github-issue-watcher.oapp`, `opscotch-github-issue-watcher.oapp.sig`, and `opscotch-github-issue-watcher.oapp.pem` from the release, then run:

```bash
cosign verify-blob \
  --signature opscotch-github-issue-watcher.oapp.sig \
  --certificate opscotch-github-issue-watcher.oapp.pem \
  --certificate-identity-regexp '^https://github.com/opscotch/builder/.github/workflows/app-release.yml@refs/(heads|tags)/.+$' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  opscotch-github-issue-watcher.oapp
```
