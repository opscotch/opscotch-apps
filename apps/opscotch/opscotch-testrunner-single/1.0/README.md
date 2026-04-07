# Opscotch Testrunner Single

This image wraps the existing Opscotch testrunner and agent binaries into a single-container harness for running one test at a time.

It is intended for both humans and AI agents that need a simple, repeatable way to execute Opscotch tests without manually starting two separate processes.

## What It Includes

- the published `linux-opscotch-testrunner-3.1.1` binary
- the published `ghcr.io/opscotch/opscotch-agent:3.1.2-dev` runtime
- the published `ghcr.io/opscotch/opscotch-testrunner` image, which includes `/opscotch-community/resources`
- two optional extra resource mount points:
  - `/local-resources1`
  - `/local-resources2`

## Required Runtime Inputs

- `OPSCOTCH_LEGAL_ACCEPTED`
- a mounted license file at `/license/license.txt`
- a mounted test directory at `/tests`
- `TEST_PATH` pointing at the test file to run

## Example

```bash
export OPSCOTCH_LEGAL_ACCEPTED='<base64-acceptance-blob>'

docker run --rm \
  -e OPSCOTCH_LEGAL_ACCEPTED \
  -e TEST_PATH=/tests/general/httpserver.test.json \
  -v /path/to/tests:/tests:ro \
  -v /path/to/license.txt:/license/license.txt:ro \
  ghcr.io/opscotch/opscotch-testrunner-single:latest
```

Optional extra resource overlays:

```bash
-v /path/to/resources1:/local-resources1:ro \
-v /path/to/resources2:/local-resources2:ro
```

When a test uses `fromDirectory`, preserve that directory structure under `/tests`.

## Release Links

- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-testrunner-single