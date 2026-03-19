# Opscoth AWS Lambda integration

This package allows opscotch to run as a AWS lambda function.

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
