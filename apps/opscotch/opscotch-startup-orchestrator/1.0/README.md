# Opscotch Startup Orchestrator

This package exposes a small composite coordinator app that runs an ordered startup plan across sibling deployments.

## Purpose

Use this app when a composite needs deterministic startup sequencing without hard-coding sibling knowledge into the participating apps.

Typical uses include:

- registering composite participants with a shared framework
- activating listeners or polling loops only after registration completes
- replaying composite startup order after a coordinated reload

## Bootstrap details

The orchestrator expects a `startupPlan` array in deployment `data`. Each entry names a deployment access id and step id to call.

```json
{
    "deploymentId": "startup-orchestrator",
    "packaging": {
        "packageId": "opscotch-startup-orchestrator"
    },
    "remoteConfiguration": "/apps/opscotch-startup-orchestrator.oapp",
    "startupPriority": 100,
    "allowDeploymentAccess": [
        {
            "id": "participant-a",
            "deploymentId": "participant-a",
            "access": "call"
        },
        {
            "id": "participant-b",
            "deploymentId": "participant-b",
            "access": "call"
        }
    ],
    "data": {
        "startupPlan": [
            {
                "deploymentAccessId": "participant-a",
                "stepId": "register-capabilities"
            },
            {
                "deploymentAccessId": "participant-b",
                "stepId": "activate-runtime"
            }
        ]
    }
}
```

## Participant expectations

Apps called by the orchestrator should expose explicit cross-deployment integration steps such as:

- `register-*`
- `activate-*`
- `reconcile-*`

If those apps also support standalone mode, any autonomous startup behavior should be disable-able through bootstrap `data`.

## Release Links

- Release: https://github.com/opscotch/opscotch-apps/releases/tag/opscotch-startup-orchestrator-1.0
- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-startup-orchestrator

To use this as a docker image, here is an example Dockerfile:

```Dockerfile
FROM ghcr.io/opscotch/opscotch-startup-orchestrator:1.0 AS opscotch-startup-orchestrator
FROM ghcr.io/opscotch/opscotch-agent:latest

COPY --from=opscotch-startup-orchestrator /apps/opscotch-startup-orchestrator.oapp /apps/opscotch-startup-orchestrator

# your custom bootstrap
COPY bootstrap.json /config/bootstrap.json
```

## Signing Public Keys

| Key ID | Public Key |
| --- | --- |
| `opscotch-app` | `8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874` |
| `opscotch-startup-orchestrator-app` | `0FFC4A0A53B822930DF4B7C0EE8695B514F804066E3E19337DA1D87116B86CD3` |
| `opscotch-startup-orchestrator-1.x` | `8695478F2844D2C973CE975ADDEB076C71F99EBA4DE244178AC060473CBE03C3` |

## Verify Artifact

Download `opscotch-startup-orchestrator.oapp`, `opscotch-startup-orchestrator.oapp.sig`, and `opscotch-startup-orchestrator.oapp.pem` from the release, then run:

```bash
cosign verify-blob \
  --signature opscotch-startup-orchestrator.oapp.sig \
  --certificate opscotch-startup-orchestrator.oapp.pem \
  --certificate-identity-regexp '^https://github.com/opscotch/builder/.github/workflows/app-release.yml@refs/(heads|tags)/.+$' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  opscotch-startup-orchestrator.oapp
```
