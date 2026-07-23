# Opscotch Key Store HTTP

HTTP adapter for the deployment-access-only `opscotch-key-store` app.

## Included workflow

The package contains the HTTP adapter workflow from
`opscotch-community/apps/opscotch-key-store-http/workflow.json` and its resource processor.

The adapter exposes `POST /key-store` and forwards the request to the
configured `opscotch-key-store` deployment through deployment access.

## Required bootstrap wiring

The consuming bootstrap must provide:

- an HTTP server permission with id `api` for the workflow's HTTP trigger;
- an outbound deployment permission with id `key-store-call` targeting the
  `opscotch-key-store` deployment.

The key-store deployment must separately authorize this deployment to receive
the `key-store-call` request and must be wired to its storage deployment.

## Packaging note

This initial metadata intentionally contains no signing keys. Signing and
release trust material should be added before publishing the package.

## Release Links

- Release: https://github.com/opscotch/opscotch-apps/releases/tag/opscotch-key-store-http-1.0
- Docker Image: `ghcr.io/opscotch/opscotch-key-store-http:1.0`
- Catalog: https://github.com/opscotch/opscotch-apps/blob/main/apps/opscotch/opscotch-key-store-http/1.0/README.md

To use this as a docker image, here is an example Dockerfile:

```Dockerfile
FROM ghcr.io/opscotch/opscotch-key-store-http:1.0 AS opscotch-key-store-http
FROM ghcr.io/opscotch/opscotch-agent:latest

COPY --from=opscotch-key-store-http /apps/opscotch-key-store-http.oapp /apps/opscotch-key-store-http

# your custom bootstrap
COPY bootstrap.json /config/bootstrap.json
```
