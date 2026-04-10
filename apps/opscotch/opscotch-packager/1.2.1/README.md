# Opscotch Package

This produces opscotch packages.

The API will be started on port 39575.

# Bootstrap detail

`allowHttpServerAccess` allows for api access to the packager.
`allowFileAccess` allows for access to resource directories, and when comparing workflows: workflow directories.

These data structures wire in the resource and workflows directories
```
"data": {
    "resourceIds": [
        "communityResources",
        "localResources",
        "anotherLotOfResources"
    ],
    "workflowSourceIds": [
        "compareWorkflow1",
        "compareWorkflow2"
    ]
}
```

Example bootstrap:
```
{
  "deploymentId": "opscotch-packager",
  "remoteConfiguration": "https://github.com/opscotch/opscotch-apps/releases/download/opscotch-packager-1.0.0/opscotch-packager.oapp",
  "persistenceRoot": "persistence",
  "allowHttpServerAccess": [
    {
      "id": "api",
      "port": 39575
    }
  ],
  "allowFileAccess": [
    {
      "id": "communityResources",
      "directoryOrFile": "/opscotch-community/resources",
      "READ": true,
      "LIST": true
    },
    {
      "id": "localResources",
      "directoryOrFile": "/localapp/resources",
      "READ": true,
      "LIST": true
    },
    {
      "id": "anotherLotOfResources",
      "directoryOrFile": "/localapp/moreResources",
      "READ": true,
      "LIST": true
    },
    {
      "id": "compareWorkflow1",
      "directoryOrFile": "/localapp/workflow1",
      "READ": true,
      "LIST": true
    },
    {
      "id": "compareWorkflow2",
      "directoryOrFile": "/localapp/workflow1",
      "READ": true,
      "LIST": true
    }
  ],
  "data": {
    "resourceIds": [
      "communityResources",
      "localResources",
      "anotherLotOfResources"
    ],
    "workflowSourceIds": [
      "compareWorkflow1",
      "compareWorkflow2"
    ]
  },
  "packaging": {
    "packageId": "opscotch-packager",
    "packagerIdentities": [
      "opscotch-packager-identiy"
    ],
    "requiredSigners": [
      {
        "id": "opscotch-app",
        "description": "Official opscotch app"
      },
      {
        "id": "opscotch-packager-app",
        "description": "Official opscotch Packager"
      },
      {
        "id": "opscotch-packager-1.x",
        "description": "Opscotch Packager v2 version 1.x lock"
      }
    ]
  },
  "keys": [
    {
      "id": "opscotch-packager-identiy",
      "type": "public",
      "purpose": "authenticated",
      "keyHex": "D1F8F5CD6424A638A938903EBEA7C140A678873417E5642A3993D2C1BE74A01E"
    },
    {
      "id": "opscotch-app",
      "type": "public",
      "purpose": "sign",
      "keyHex": "8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874"
    },
    {
      "id": "opscotch-packager-app",
      "type": "public",
      "purpose": "sign",
      "keyHex": "6D9841078C83239B1CF148BF92A533A4C8BA4901DA0AD1ADDE2FE2D23BA3F156"
    },
    {
      "id": "opscotch-packager-1.x",
      "type": "public",
      "purpose": "sign",
      "keyHex": "5912C087D6C958A84433F666784CE228B17EF34E39E5D9C7FD2BBA5927A760F9"
    }
  ]
}
```

## Release Links

- Release: https://github.com/opscotch/opscotch-apps/releases/tag/opscotch-packager-1.2.1
- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-packager

To use this as a docker image, here is an example Dockerfile:

```Dockerfile
FROM ghcr.io/opscotch/opscotch-packager:1.2.1 AS opscotch-packager
FROM ghcr.io/opscotch/opscotch-agent:latest

COPY --from=opscotch-packager /apps/opscotch-packager.oapp /apps/opscotch-packager

# your custom bootstrap
COPY bootstrap.json /config/bootstrap.json
```

## Signing Public Keys

| Key ID | Public Key |
| --- | --- |
| `opscotch-app` | `8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874` |
| `opscotch-packager-app` | `6D9841078C83239B1CF148BF92A533A4C8BA4901DA0AD1ADDE2FE2D23BA3F156` |
| `opscotch-packager-1.x` | `5912C087D6C958A84433F666784CE228B17EF34E39E5D9C7FD2BBA5927A760F9` |

## Verify Artifact

Download `opscotch-packager.oapp`, `opscotch-packager.oapp.sig`, and `opscotch-packager.oapp.pem` from the release, then run:

```bash
cosign verify-blob \
  --signature opscotch-packager.oapp.sig \
  --certificate opscotch-packager.oapp.pem \
  --certificate-identity-regexp '^https://github.com/opscotch/builder/.github/workflows/app-release.yml@refs/(heads|tags)/.+$' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  opscotch-packager.oapp
```
