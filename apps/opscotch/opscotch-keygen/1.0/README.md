# Opscotch Key Generation

A reusable convienience app for exposing cryptographic key generation as per [the docs](https://docs.opscotch.co/docs/current/apireference#Key-purpose)

Supports key generation for `purpose` : "sign", "authenticated", "symmetric", "anonymous"

## Usage

Call the http endpoint like so:

```
curl -i "<host>/key-gen?purpose=<purpose>"
```

The response will be:
```
{
    "purpose":"",
    "publicKey":"",
    "secretKey":""
}
```
Key are in hex.

## Release Links

- Release: https://github.com/opscotch/opscotch-apps/releases/tag/opscotch-keygen-1.0
- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-keygen

To use this as a docker image, here is an example Dockerfile:

```Dockerfile
FROM ghcr.io/opscotch/opscotch-keygen:1.0 AS opscotch-keygen
FROM ghcr.io/opscotch/opscotch-agent:latest

COPY --from=opscotch-keygen /apps/opscotch-keygen.oapp /apps/opscotch-keygen

# your custom bootstrap
COPY bootstrap.json /config/bootstrap.json
```

## Signing Public Keys

| Key ID | Public Key |
| --- | --- |
| `opscotch-app` | `8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874` |
| `opscotch-keygen-app` | `90D8F1EDE9B8529A75FB6AF2D5FB7B6F4C60B2FFB4A9FE3BC3AD618CB80383E8` |
| `opscotch-keygen-1.x` | `7976FD85EE9CA4A70B8A1233A60DE9AF38561594B2E85C67907B883BAAC70EB2` |

## Verify Artifact

Download `opscotch-keygen.oapp`, `opscotch-keygen.oapp.sig`, and `opscotch-keygen.oapp.pem` from the release, then run:

```bash
cosign verify-blob \
  --signature opscotch-keygen.oapp.sig \
  --certificate opscotch-keygen.oapp.pem \
  --certificate-identity-regexp '^https://github.com/opscotch/builder/.github/workflows/app-release.yml@refs/(heads|tags)/.+$' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  opscotch-keygen.oapp
```
