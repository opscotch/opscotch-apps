# Opscotch Testrunner

This image publishes the standalone Opscotch testrunner binary as a container image.

It is intended to be reused by wrapper images such as `opscotch-testrunner-single`, or run directly when you want the testrunner binary in a container without bundling an agent runtime.

## What It Includes

- the published `linux-opscotch-testrunner-3.1.1` binary

## Example

```bash
docker run --rm \
  -v /path/to/tests:/tests:ro \
  -v /path/to/resources:/resources:ro \
  -v /path/to/license.txt:/license/license.txt:ro \
  ghcr.io/opscotch/opscotch-testrunner:latest \
  /tests/testrunner.config.json /tests/general/httpserver.test.json
```

## Release Links

- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-testrunner