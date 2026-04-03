# Opscotch AWS API Gateway v2 HTTP Bridge

This app converts AWS API Gateway v2 Lambda events into standard Opscotch HTTP request envelopes and forwards them to a configured target step.

It is intended to sit between the `opscotch-aws-lambda` listener and another deployment that expects normal HTTP-shaped request payloads.

Call flow:
```
opscotch-aws-lambda -> opscotch-aws-apigw2-http-bridge -> target deployment
```

## Required bootstrap wiring

This app must be wired into a larger bootstrap with:

- a caller deployment that can `call` the bridge deployment
- a bridge deployment that loads `opscotch-aws-apigw2-http-bridge`
- a target deployment callback the bridge can invoke

Example bootstrap:

```json
[
  {
    "deploymentId": "lambda",
    "allowDeploymentAccess": [
      {
        "id": "transport-wrapper",
        "access": "call",
        "deploymentId": "transport-wrapper"
      }
    ],
    "data": {
      "eventRouting": {
        "apigw-v2": {
          "deploymentAccessId": "transport-wrapper",
          "stepId": "accept-apigw-v2"
        }
      }
    }
  },
  {
    "deploymentId": "transport-wrapper",
    "remoteConfiguration": "/apps/opscotch-aws-apigw2-http-bridge.oapp",
    "allowDeploymentAccess": [
      {
        "id": "bridge-callback",
        "access": "receive",
        "deploymentId": "lambda"
      },
      {
        "id": "target-server",
        "access": "call",
        "deploymentId": "server-bridge"
      }
    ],
    "data": {
      "deploymentAccessId": "target-server",
      "stepId": "accept-forward-request"
    }
  }
]
```

## Expected contract

The caller should send an API Gateway v2 event to step `accept-apigw-v2`.

The bridge:

- decodes the event body when `isBase64Encoded` is true
- extracts method, path, query string, headers, and body
- forwards the normalized request to the configured target callback
- wraps the downstream result into a `{ statusCode, headers, body, isBase64Encoded }` response object

## Release Links

- Release: https://github.com/opscotch/opscotch-apps/releases/tag/opscotch-aws-apigw2-http-bridge-1.0
- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-aws-apigw2-http-bridge

To use this as a docker image, here is an example Dockerfile:

```Dockerfile
FROM ghcr.io/opscotch/opscotch-aws-apigw2-http-bridge:1.0 AS opscotch-aws-apigw2-http-bridge
FROM ghcr.io/opscotch/opscotch-agent:latest

COPY --from=opscotch-aws-apigw2-http-bridge /apps/opscotch-aws-apigw2-http-bridge.oapp /apps/opscotch-aws-apigw2-http-bridge

# your custom bootstrap
COPY bootstrap.json /config/bootstrap.json
```

## Signing Public Keys

| Key ID | Public Key |
| --- | --- |
| `opscotch-app` | `8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874` |
| `opscotch-aws-apigw2-http-bridge-app` | `4BA9234399E09D05FEDDCF5246817E230B36070DC672C739D018BF8F280E4361` |
| `opscotch-aws-apigw2-http-bridge-1.x` | `BD55CC9C4F0839EB8BA6753A3923788D28F4DACCACFD334C2463EE38E412E431` |

## Verify Artifact

Download `opscotch-aws-apigw2-http-bridge.oapp`, `opscotch-aws-apigw2-http-bridge.oapp.sig`, and `opscotch-aws-apigw2-http-bridge.oapp.pem` from the release, then run:

```bash
cosign verify-blob \
  --signature opscotch-aws-apigw2-http-bridge.oapp.sig \
  --certificate opscotch-aws-apigw2-http-bridge.oapp.pem \
  --certificate-identity-regexp '^https://github.com/opscotch/builder/.github/workflows/app-release.yml@refs/(heads|tags)/.+$' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  opscotch-aws-apigw2-http-bridge.oapp
```
