# Opscoth AWS Lambda integration

This package allows opscotch to run as a AWS lambda function.

## Bootstrap details

When using this as an AWS Lambda function it is critical that this app starts first.

The `deferLoading` and `startupPriority` settings are critical.

AWS will supply the `${AWS_LAMBDA_RUNTIME_API}`.

Use `allowDeploymentAccess` to declare which deploymentId the lamdba payloads should be sent to. Use the "data.eventRouting" to declare which step to send the payload to.

```
{
    "deferLoading" : "false",
    "startupPriority": 1,
    "deploymentId": "lambda",
    "packaging": {
        "packageId": "opscotch-aws-lambda",
    },
    "remoteConfiguration": "/apps/opscotch-aws-lambda.oapp",
    "allowExternalHostAccess": [
        {
            "id": "aws-lambda-runtime-api",
            "host": "http://${AWS_LAMBDA_RUNTIME_API}"
        }
    ],
    "allowDeploymentAccess": [
        {
            "id": "aws-lambda-processor",
            "access": "call",
            "deploymentId": "your-processor"
        }
    ]
    "data": {
        "eventRouting": {
            "apigw-v2": {
                "deploymentAccessId": "aws-lambda-processor",
                "stepId": "<your apigw-v2 step>"
            },
            "sqs": {
                "deploymentAccessId": "aws-lambda-processor",
                "stepId": "<your sqs step>"
            }
        }
    }
}
```

## Release Links

- Release: https://github.com/opscotch/opscotch-apps/releases/tag/opscotch-aws-lambda-1.0.0
- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-aws-lambda

To use this as a docker image, here is an example Dockerfile:

```Dockerfile
FROM ghcr.io/opscotch/opscotch-aws-lambda:1.0.0 AS opscotch-aws-lambda
FROM ghcr.io/opscotch/opscotch-agent:latest

COPY --from=opscotch-aws-lambda /apps/opscotch-aws-lambda.oapp /apps/opscotch-aws-lambda

# your custom bootstrap
COPY bootstrap.json /config/bootstrap.json
```

## Signing Public Keys

| Key ID | Public Key |
| --- | --- |
| `opscotch-app` | `8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874` |
| `opscotch-aws-lambda-app` | `F0F2BED9A53B5BA618868CDD2D248ED2ACC35959E14CC5A614D0F10945352351` |
| `opscotch-aws-lambda-1.x` | `E8A5AB7649D808D8150092BE5359861CE1C1894485B5BAC6155361E6B5D5B055` |

## Verify Artifact

Download `opscotch-aws-lambda.oapp`, `opscotch-aws-lambda.oapp.sig`, and `opscotch-aws-lambda.oapp.pem` from the release, then run:

```bash
cosign verify-blob \
  --signature opscotch-aws-lambda.oapp.sig \
  --certificate opscotch-aws-lambda.oapp.pem \
  --certificate-identity-regexp '^https://github.com/opscotch/builder/.github/workflows/app-release.yml@refs/(heads|tags)/.+$' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  opscotch-aws-lambda.oapp
```
