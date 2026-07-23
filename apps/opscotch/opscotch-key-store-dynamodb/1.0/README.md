# Opscotch Key Store DynamoDB

Deployment-only DynamoDB storage provider for `opscotch-key-store`.

This initial package metadata contains no signing keys. Add release trust
material before publishing.

## Release Links

- Release: https://github.com/opscotch/opscotch-apps/releases/tag/opscotch-key-store-dynamodb-1.0
- Docker Image: `ghcr.io/opscotch/opscotch-key-store-dynamodb:1.0`
- Catalog: https://github.com/opscotch/opscotch-apps/blob/main/apps/opscotch/opscotch-key-store-dynamodb/1.0/README.md

To use this as a docker image, here is an example Dockerfile:

```Dockerfile
FROM ghcr.io/opscotch/opscotch-key-store-dynamodb:1.0 AS opscotch-key-store-dynamodb
FROM ghcr.io/opscotch/opscotch-agent:latest

COPY --from=opscotch-key-store-dynamodb /apps/opscotch-key-store-dynamodb.oapp /apps/opscotch-key-store-dynamodb

# your custom bootstrap
COPY bootstrap.json /config/bootstrap.json
```
