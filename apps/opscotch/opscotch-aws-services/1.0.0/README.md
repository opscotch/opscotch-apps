# Opscotch AWS Services integration

This package allows Opscotch to integrate with AWS Services workflows.

## Bootstrap details

It is safe to set `"deferLoading" : "true"` so that this loads on first request.

It is important to add `allowExternalHostAccess` hosts for the services that you need to use. 

The `sts` host should always be added.

A fully exhaustive list of available hosts can be found [here](https://github.com/opscotch/opscotch-community/blob/3.1.1/apps/aws/aws-services/bootstrap.json)

```
{
    "deploymentId": "aws-services",
    "packaging": {
        "packageId": "opscotch-aws-services"
    },
    "remoteConfiguration": "/apps/opscotch-aws-services.oapp",
    "deferLoading" : "true",
    "allowExternalHostAccess": [
        {
            "id": "sts",
            "host": "https://sts.amazonaws.com",
            "data": {
                "region": "us-east-1",
                "host": "sts.amazonaws.com",
                "service": "sts",
                "accessKey": "${AWS_ACCESS_KEY_ID}",
                "secretKey": "${AWS_SECRET_ACCESS_KEY}",
                "amzSecurityToken": "${AWS_SESSION_TOKEN}"
            }
        },
        {
            "id": "sqs",
            "host": "https://sqs.${AWS_REGION}.amazonaws.com",
            "data": {
                "host": "sqs.${AWS_REGION}.amazonaws.com",
                "service": "sqs"
            }
        },
        {
            "id": "ses",
            "host": "https://email.${AWS_REGION}.amazonaws.com",
            "data": {
                "host": "email.${AWS_REGION}.amazonaws.com",
                "service": "ses"
            }
        }
    ],
    "allowDeploymentAccess": [
        {
            "id": "callers",
            "access": "receive",
            "anyDeployment": true
        }
    ],
    "data": {
        "awsRegion": "${AWS_REGION}",
        "awsAccount": "${AWS_ACCOUNT}"
    }
}
```

## Calling example

Review [the config](https://github.com/opscotch/opscotch-community/blob/3.1.1/apps/aws/aws-services/aws-services.config.json) to see the available steps, for example `sns-request`, `ec2-request` etc.

Here is how you call an aws service:

```
var response = context.sendToStep(`aws-services`, `sqs-request`, JSON.stringify( { aws_sqs_queue : 'test-queue', action : 'SendMessage', params : { MessageBody : `some body` } } ))
```

## Signing Public Keys

| Key ID | Public Key |
| --- | --- |
| `opscotch-app` | `8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874` |
| `opscotch-aws-services-app` | `03DBDC90F71E53AA84CD4FAA8BC8E5025743E55385C4163EE92B2FFA33EBA457` |
| `opscotch-aws-services-1.x` | `2C2698E5C868257C58A6B1D6D2D81388403FA0ED1A159068EEFB0F4DF87A3535` |

## Verify Artifact

Download `opscotch-aws-services.oapp`, `opscotch-aws-services.oapp.sig`, and `opscotch-aws-services.oapp.pem` from the release, then run:

```bash
cosign verify-blob \
  --signature opscotch-aws-services.oapp.sig \
  --certificate opscotch-aws-services.oapp.pem \
  --certificate-identity-regexp '^https://github.com/opscotch/builder/.github/workflows/app-release.yml@refs/(heads|tags)/.+$' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  opscotch-aws-services.oapp
```
