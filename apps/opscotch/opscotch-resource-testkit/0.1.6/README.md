# Opscotch Resource Testkit

`ghcr.io/opscotch/opscotch-resource-testkit:latest` is the published GHCR image for the Opscotch JavaScript resource unit test environment.

This image is intended for fast authoring feedback when humans or AI need to test resource files directly without running the full Opscotch agent and testrunner harness.

## What It Includes

- Vitest as the JavaScript unit test runner
- published `@opscotch/resource-testkit` runtime (currently `0.1.6`)
- a real in-memory byte buffer store with opaque handles
- stubbed and mockable complex contexts such as `crypto()`, `files()`, and `queue()`

## Scope

This image is for unit testing resource files.

It does not replace the existing Opscotch integration test harness.

## Running All Unit Tests

```bash
docker run --rm \
  -e WORKSPACE_DIR=/workspace \
  -e UNIT_TEST_PATH=/workspace/unit-tests \
  -v /path/to/project-root:/workspace:ro \
  ghcr.io/opscotch/opscotch-resource-testkit:latest
```

By default the image runs tests from `/workspace/unit-tests`.

For Opscotch community tests (repo mounted at `/workspace`):

```bash
docker run --rm \
  -e WORKSPACE_DIR=/workspace/community/opscotch-community \
  -e UNIT_TEST_PATH=/workspace/community/opscotch-community/unit-tests \
  -v /path/to/opscotch-repo-root:/workspace:ro \
  ghcr.io/opscotch/opscotch-resource-testkit:latest
```

## Running A Single Unit Test

Set `UNIT_TEST_PATH` to a single test file:

```bash
docker run --rm \
  -e WORKSPACE_DIR=/workspace \
  -e UNIT_TEST_PATH=/workspace/unit-tests/general/standard-clear-body.test.ts \
  -v /path/to/project-root:/workspace:ro \
  ghcr.io/opscotch/opscotch-resource-testkit:latest
```

## Running A Set Of Unit Tests

Set `UNIT_TEST_PATH` to a directory:

```bash
docker run --rm \
  -e WORKSPACE_DIR=/workspace \
  -e UNIT_TEST_PATH=/workspace/unit-tests/general \
  -v /path/to/project-root:/workspace:ro \
  ghcr.io/opscotch/opscotch-resource-testkit:latest
```

## Passing Additional Vitest Filters

Extra arguments after the image name are forwarded to `vitest run`.

Example filtering by test name:

```bash
docker run --rm \
  -e WORKSPACE_DIR=/workspace \
  -e UNIT_TEST_PATH=/workspace/unit-tests/general \
  -v /path/to/project-root:/workspace:ro \
  ghcr.io/opscotch/opscotch-resource-testkit:latest \
  --testNamePattern="clears the current body"
```

## Runtime Behavior

- The image is self-contained at runtime.
- Dependencies are installed at image build time (`npm ci` in Dockerfile).
- Running the container does not run `npm install`.
- Coverage output is written to `/tmp/opscotch-resource-testkit-coverage` inside the container.

## Test Authoring Samples (Suite Style)

The source-of-truth package supports the same suite-first test style as the npm package.

Use `createResourceSuite(...)` to register resources once, then execute by logical id.

### Single Resource Example

```ts
import path from 'node:path';
import { createJavascriptContext, createResourceSuite } from '@opscotch/resource-testkit';
import { describe, expect, it } from 'vitest';

const suite = createResourceSuite({
  baseDir: path.resolve(import.meta.dirname, '../resources'),
  resources: [{ id: 'sample-resource', resource: './sample-resource.js' }],
});

describe('sample-resource', () => {
  it('updates the payload body', async () => {
    const context = createJavascriptContext({
      body: JSON.stringify({
        operation: 'normalize',
        labels: ['A', 'b'],
      }),
    });

    await suite.run('sample-resource', { context });

    expect(JSON.parse(context.getBody() || '{}')).toEqual({
      status: 'ok',
      labels: ['a', 'b'],
    });
  });
});
```

### Multiple Resource Example

```ts
import path from 'node:path';
import { createJavascriptContext, createResourceSuite } from '@opscotch/resource-testkit';
import { describe, it } from 'vitest';

const suite = createResourceSuite({
  baseDir: path.resolve(import.meta.dirname, '../resources'),
  resources: [
    { id: 'log-fetch-url-generator', resource: './log-fetch-url-generator.js' },
    { id: 'log-fetch-payload-generator', resource: './log-fetch-payload-generator.js' },
  ],
});

describe('resource suite', () => {
  it('runs a resource by id', async () => {
    const context = createJavascriptContext({
      data: { openclawGatewayHostId: 'openclaw-local-gateway' },
    });

    await suite.run('log-fetch-url-generator', { context });
  });
});
```

## Release Links

- Docker Image: `ghcr.io/opscotch/opscotch-resource-testkit:0.1.6`
- Catalog: https://github.com/opscotch/opscotch-apps/blob/main/apps/opscotch/opscotch-resource-testkit/0.1.6/README.md