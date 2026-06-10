# Opscotch Testrunner Single

`ghcr.io/opscotch/opscotch-testrunner-single:latest` is the official all-in-one image for running a single Opscotch test in a one-container harness.

This image contains both the Opscotch testrunner and the Opscotch agent runtime in a single container for running one test at a time.

It is intended for both humans and AI agents that need a simple, repeatable way to execute Opscotch tests without manually starting two separate processes.

## What It Includes

- the published `linux-opscotch-testrunner-3.1.6` binary
- the published `ghcr.io/opscotch/opscotch-agent:3.1.6-dev` runtime
- the published `ghcr.io/opscotch/opscotch-testrunner` image, which includes `/opscotch-community/resources`
- two optional extra resource mount points:
  - `/local-resources1`
  - `/local-resources2`

## Required Runtime Inputs

- `OPSCOTCH_LEGAL_ACCEPTED`
- a mounted test directory at `/tests`
- `TEST_PATH` pointing at the test file to run

## Example

```bash
export OPSCOTCH_LEGAL_ACCEPTED='<base64-acceptance-blob>'

docker run --rm \
  -e OPSCOTCH_LEGAL_ACCEPTED \
  -e TEST_PATH=/tests/general/httpserver.test.json \
  -v /path/to/tests:/tests:ro \
  ghcr.io/opscotch/opscotch-testrunner-single:latest
```

For AI clients and other automation, capture stdout and stderr to a file and inspect only the summary line first:

```bash
docker run --rm \
  -e OPSCOTCH_LEGAL_ACCEPTED \
  -e TEST_PATH=/tests/general/httpserver.test.json \
  -v /path/to/tests:/tests:ro \
  ghcr.io/opscotch/opscotch-testrunner-single:latest \
  > testrunner-single.log 2>&1

grep -q "1 succeeded" testrunner-single.log && echo "success"
grep -q "1 failed" testrunner-single.log && echo "failure"
```

This keeps token use low. On failure, inspect `testrunner-single.log` for the detailed trace.

Optional extra resource overlays:

```bash
-v /path/to/resources1:/local-resources1:ro \
-v /path/to/resources2:/local-resources2:ro
```

When a test uses `fromDirectory`, preserve that directory structure under `/tests`.

## Release Links

- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-testrunner-single