# Opscotch Packaged Server Bridge

This app provides a small generic bridge for forwarding cross-deployment callback payloads into an internal HTTP server exposed through server access.

It is intended to keep transport-specific wrappers outside the packaged server workflow, while still allowing a caller deployment to invoke that server through an in-process host.

The target app must expose an HTTP server, and when the bridge is calling it through `transport: "inProc"` that server should be configured with `inProcOnly: true`.

Call flow:
```
transport-wrapper -> (via cross deployment) -> server-bridge -> (via inproc server) ->  packaged-server
```

## Required bootstrap wiring

This app is not useful on its own. It must be wired into a larger bootstrap with:

- a caller deployment that can `call` the bridge deployment
- a bridge deployment that loads `opscotch-packaged-server-bridge`
- a target deployment that exposes an HTTP server the bridge can reach through `allowExternalHostAccess`
- a target app HTTP server entry configured for in-process access, typically with `inProcOnly: true`

Example bootstrap:

```json
[
  {
    "deploymentId": "transport-wrapper",
    "allowDeploymentAccess": [
      {
        "id": "server-bridge",
        "access": "call",
        "deploymentId": "server-bridge"
      }
    ],
    "data": {
      "eventRouting": {
        "some-event-type": {
          "deploymentAccessId": "server-bridge",
          "stepId": "accept-forward-request"
        }
      }
    }
  },
  {
    "deploymentId": "packaged-server",
    "allowHttpServerAccess": [
      {
        "id": "server-http",
        "inProcOnly": true
      }
    ]
  },
  {
    "deploymentId": "server-bridge",
    "remoteConfiguration": "/apps/opscotch-packaged-server-bridge.oapp",
    "packaging": {
      "packageId": "opscotch-packaged-server-bridge"
    },
    "allowExternalHostAccess": [
      {
        "id": "target-server",
        "transport": "inProc",
        "inProcServerId": "server-http",
        "inProcDeploymentId": "packaged-server",
        "allowList": [
          {
            "method": "GET",
            "uriPattern": "/service.*"
          },
          {
            "method": "POST",
            "uriPattern": "/service.*"
          }
        ]
      }
    ],
    "allowDeploymentAccess": [
      {
        "id": "bridge-callback",
        "access": "receive",
        "deploymentId": "transport-wrapper"
      }
    ]
  }
]
```

In that example, `packaged-server` is the target app. Its `allowHttpServerAccess` entry exposes server id `server-http`, and `inProcOnly: true` keeps that server internal so the bridge can reach it through `transport: "inProc"`.

## Expected contract

The caller should send its event to step `accept-forward-request` on the bridge deployment.

The bridge expects either:

- a generic request envelope with fields such as `method`, `path`, `queryString`, `headers`, `body`, and `isBase64Encoded`
- or an API Gateway v2 style event with `rawPath`, `rawQueryString`, `headers`, `body`, `isBase64Encoded`, and `requestContext.http.method`

The bridge then:

- normalizes the inbound payload into a normal HTTP request
- forwards that request to host id `target-server`
- wraps the HTTP response into a `{ statusCode, headers, body, isBase64Encoded }` response object

## Release Links

- Release: https://github.com/opscotch/opscotch-apps/releases/tag/opscotch-packaged-server-bridge-1.2
- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-packaged-server-bridge

To use this as a docker image, here is an example Dockerfile:

```Dockerfile
FROM ghcr.io/opscotch/opscotch-packaged-server-bridge:1.2 AS opscotch-packaged-server-bridge
FROM ghcr.io/opscotch/opscotch-agent:latest

COPY --from=opscotch-packaged-server-bridge /apps/opscotch-packaged-server-bridge.oapp /apps/opscotch-packaged-server-bridge

# your custom bootstrap
COPY bootstrap.json /config/bootstrap.json
```

## Signing Public Keys

| Key ID | Public Key |
| --- | --- |
| `opscotch-app` | `8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874` |
| `opscotch-packaged-server-bridge-app` | `C4168125AF47158E02BFE5F601992639B4919F7F87968D38EF80887C7A53FFBC` |
| `opscotch-packaged-server-bridge-1.x` | `8DF25D5AE0655DBEF197CE5DFA1DF8FC26E41C8A75C1472956D329EAD8DD9E55` |

## Verify Artifact

Download `opscotch-packaged-server-bridge.oapp`, `opscotch-packaged-server-bridge.oapp.sig`, and `opscotch-packaged-server-bridge.oapp.pem` from the release, then run:

```bash
cosign verify-blob \
  --signature opscotch-packaged-server-bridge.oapp.sig \
  --certificate opscotch-packaged-server-bridge.oapp.pem \
  --certificate-identity-regexp '^https://github.com/opscotch/builder/.github/workflows/app-release.yml@refs/(heads|tags)/.+$' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  opscotch-packaged-server-bridge.oapp
```
