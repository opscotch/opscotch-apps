# Opscotch Elasticbeat Elasticsearch Receiver

## 0.1.1

Add key-store-backed signing metadata required for packaged releases.

## 0.1.0

Initial release of the Elasticsearch-compatible receiver for Elastic Beats.

- Supports cluster info and common Beats probe endpoints.
- Accepts Elasticsearch bulk NDJSON and acknowledges parseable actions.
- Fans documents out asynchronously through Opscotch deployment access.
- Emits receiver, bulk-ingest, and fan-out metrics.

## Release Links

- Release: https://github.com/opscotch/opscotch-apps/releases/tag/opscotch-elasticbeat-receiver-0.1.1
- Docker Image: `ghcr.io/opscotch/opscotch-elasticbeat-receiver:0.1.1`
- Catalog: https://github.com/opscotch/opscotch-apps/blob/main/apps/opscotch/opscotch-elasticbeat-receiver/0.1.1/README.md

To use this as a docker image, here is an example Dockerfile:

```Dockerfile
FROM ghcr.io/opscotch/opscotch-elasticbeat-receiver:0.1.1 AS opscotch-elasticbeat-receiver
FROM ghcr.io/opscotch/opscotch-agent:latest

COPY --from=opscotch-elasticbeat-receiver /apps/opscotch-elasticbeat-receiver.oapp /apps/opscotch-elasticbeat-receiver

# your custom bootstrap
COPY bootstrap.json /config/bootstrap.json
```

## Signing Public Keys

| Key ID | Public Key |
| --- | --- |
| `opscotch-app` | `8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874` |
| `opscotch-elasticbeat-receiver-app` | `C4DAB04834F4C5BEB1B549794738752AF9ECC58668E64D70A74B91E7E96FAB33` |
| `opscotch-elasticbeat-receiver-1.x` | `43D140A34B33E9AFFC4E26586CCAAD91734E73200CD6DBA0FC14C6138185109A` |

## Verify Artifact

Download `opscotch-elasticbeat-receiver.oapp`, `opscotch-elasticbeat-receiver.oapp.sig`, and `opscotch-elasticbeat-receiver.oapp.pem` from the release, then run:

```bash
cosign verify-blob \
  --signature opscotch-elasticbeat-receiver.oapp.sig \
  --certificate opscotch-elasticbeat-receiver.oapp.pem \
  --certificate-identity-regexp '^https://github.com/opscotch/builder/.github/workflows/app-release.yml@refs/(heads|tags)/.+$' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  opscotch-elasticbeat-receiver.oapp
```
