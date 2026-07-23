# opscotch-key-store-admin-http

Administrative HTTP adapter for importing existing immutable key pairs.

## Endpoint

`POST /admin/key-store/load` on the `api` HTTP server:

```json
{
  "load": {
    "keyId": "my-app-identity",
    "purpose": "sign",
    "keyPair": {
      "publicKeyHex": "...",
      "secretKeyHex": "..."
    }
  }
}
```

The endpoint returns `201` when the pair is loaded and `409` when the identity
already exists. Existing keys cannot be updated or rotated. The response never
contains the imported secret key.

This app must be separately authenticated and authorized. The core key-store
deployment receives requests through the `key-store-admin-call` deployment
access permitter.

## Release Links

- Release: https://github.com/opscotch/opscotch-apps/releases/tag/opscotch-key-store-admin-http-1.0
- Docker Image: `ghcr.io/opscotch/opscotch-key-store-admin-http:1.0`
- Catalog: https://github.com/opscotch/opscotch-apps/blob/main/apps/opscotch/opscotch-key-store-admin-http/1.0/README.md

To use this as a docker image, here is an example Dockerfile:

```Dockerfile
FROM ghcr.io/opscotch/opscotch-key-store-admin-http:1.0 AS opscotch-key-store-admin-http
FROM ghcr.io/opscotch/opscotch-agent:latest

COPY --from=opscotch-key-store-admin-http /apps/opscotch-key-store-admin-http.oapp /apps/opscotch-key-store-admin-http

# your custom bootstrap
COPY bootstrap.json /config/bootstrap.json
```
