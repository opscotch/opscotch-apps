# Opscotch Licensing App

This app can be used as a licensing server embedded in the agent, or as a licensing server that other agents can use.

# Boostrap details

The `allowHttpServerAccess` allows remote or local agents to request a license.

Define licenses in the `data.licensePools`

```
{
    "deploymentId": "opscotch-licenser",
    "remoteConfiguration": "/apps/opscotch-licenser.oapp",
    "packaging" : {
        "packageId" : "opscotch-licenser"
    },
    "persistenceRoot" : "persistence",
    "allowHttpServerAccess": [
        {
            "id": "api",
            "port": 39576
        }
    ],
    "data" : {
        "licensePools" : [
            {
                "id" : "default",
                "license" : "..."
            }
        ]
    }
}    
```

## Release Links

- Release: https://github.com/opscotch/opscotch-apps/releases/tag/opscotch-licenser-1.0.0
- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-licenser

To use this as a docker image, here is an example Dockerfile:

```Dockerfile
FROM ghcr.io/opscotch/opscotch-licenser:1.0.0 AS opscotch-licenser
FROM ghcr.io/opscotch/opscotch-agent:latest

COPY --from=opscotch-licenser /apps/opscotch-licenser.oapp /apps/opscotch-licenser

# your custom bootstrap
COPY bootstrap.json /config/bootstrap.json
```

## Signing Public Keys

| Key ID | Public Key |
| --- | --- |
| `opscotch-app` | `8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874` |
| `opscotch-licenser-app` | `40D95D78BF06858A8739F6DC76B0FC8EB47765EC527DC3FB3A43CDE22B9805A5` |
| `opscotch-licenser-1.x` | `BF40C8D742F43024C45A4BEBC00A5CD9FB2C1E8F31F1BC3580132F4F8F6DAE22` |

## Verify Artifact

Download `opscotch-licenser.oapp`, `opscotch-licenser.oapp.sig`, and `opscotch-licenser.oapp.pem` from the release, then run:

```bash
cosign verify-blob \
  --signature opscotch-licenser.oapp.sig \
  --certificate opscotch-licenser.oapp.pem \
  --certificate-identity-regexp '^https://github.com/opscotch/builder/.github/workflows/app-release.yml@refs/(heads|tags)/.+$' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  opscotch-licenser.oapp
```
