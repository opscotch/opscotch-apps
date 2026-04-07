# Opscotch Resource Testkit

`ghcr.io/opscotch/opscotch-resource-testkit:latest` is a docker-first unit test environment for Opscotch JavaScript resources.

This image is intended for fast authoring feedback when humans or AI need to test resource files directly without running the full Opscotch agent and testrunner harness.

## What It Includes

- Vitest as the JavaScript unit test runner
- an Opscotch resource runtime shim for `doc` and `context`
- a real in-memory byte buffer store with opaque handles
- stubbed and mockable complex contexts such as `crypto()`, `files()`, and `queue()`

## Scope

This image is for unit testing resource files.

It does not replace the existing Opscotch integration test harness.

## Running All Unit Tests

```bash
docker run --rm \
  -v /path/to/opscotch-community:/workspace:ro \
  ghcr.io/opscotch/opscotch-resource-testkit:latest
```

By default the image runs tests from `/workspace/unit-tests`.

## Running A Single Unit Test

Set `UNIT_TEST_PATH` to a single test file:

```bash
docker run --rm \
  -e UNIT_TEST_PATH=/workspace/unit-tests/general/standard-clear-body.test.ts \
  -v /path/to/opscotch-community:/workspace:ro \
  ghcr.io/opscotch/opscotch-resource-testkit:latest
```

## Running A Set Of Unit Tests

Set `UNIT_TEST_PATH` to a directory:

```bash
docker run --rm \
  -e UNIT_TEST_PATH=/workspace/unit-tests/general \
  -v /path/to/opscotch-community:/workspace:ro \
  ghcr.io/opscotch/opscotch-resource-testkit:latest
```

## Passing Additional Vitest Filters

Extra arguments after the image name are forwarded to `vitest run`.

Example filtering by test name:

```bash
docker run --rm \
  -e UNIT_TEST_PATH=/workspace/unit-tests/general \
  -v /path/to/opscotch-community:/workspace:ro \
  ghcr.io/opscotch/opscotch-resource-testkit:latest \
  --testNamePattern="clears the current body"
```

## Release Links

- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-resource-testkit