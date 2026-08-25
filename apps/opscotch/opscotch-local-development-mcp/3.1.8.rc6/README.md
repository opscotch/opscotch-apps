# Opscotch Local Development MCP

`ghcr.io/opscotch/opscotch-local-development-mcp:latest` is a local-only streamable HTTP MCP server for running privileged verification flows against a mounted developer workspace.

This image is not a public execution surface.

## Endpoint

- `GET /mcp` exposes the streamable SSE endpoint
- `POST /mcp` accepts JSON-RPC MCP requests

## Required Runtime Environment

- `OPSCOTCH_LEGAL_ACCEPTED`

Optional runtime overrides:

- `OPSCOTCH_LOCAL_MCP_ARTIFACT_ROOT` default `/tmp/opscotch-local-mcp/artifacts`
- `OPSCOTCH_LOCAL_MCP_RUN_ROOT` default `/tmp/opscotch-local-mcp/runs`
- `OPSCOTCH_LOCAL_MCP_PORT` default `8080`

## Mounted Path Contract

The server exposes mounted paths through logical roots defined in `OPSCOTCH_LOCAL_MCP_ROOTS_JSON`.

Use that JSON object to declare every mounted root the server should understand, for example:

```json
{
  "workspace": "/workspace",
  "app": "/workspace-app",
  "artifacts": "/artifacts"
}
```

All tool inputs must stay inside these allowlisted roots. Relative paths are resolved directly under the target logical root. Absolute paths are accepted only when they already point inside the mounted root, or when the client also provides `hostMounts` metadata so the server can translate host-visible paths into container-visible mounted paths.

## Workspace Mounting

The most important rule is:

- the path you pass to a tool must be relative to the directory you mounted as the workspace root

This server does not interpret paths relative to your host repository root unless that exact repository root is what you mounted as `/workspace` or declared as another named workspace root.

### Host Mount Translation

When an MCP client only knows the host-visible absolute path, provide `hostMounts` in the tool input:

```json
{
  "root": "app",
  "hostPath": "/host/project/apps/example-app"
}
```

Then a tool path like:

```json
{
  "root": "app",
  "path": "/host/project/apps/example-app/unit-tests/example-resource.test.ts"
}
```

is translated to the configured mounted `app` root plus the relative suffix `unit-tests/example-resource.test.ts`.

### Good Mental Model

If you run:

```bash
-v /host/project:/workspace
```

then:

- `{"root":"workspace","path":"unit-tests/foo.test.ts"}` resolves to `/host/project/unit-tests/foo.test.ts`
- `{"root":"workspace","path":"tests/general/httpserver.test.json"}` resolves to `/host/project/tests/general/httpserver.test.json`

### Common Working Example

If you want to run tests from `community/opscotch-community`, mount that directory itself as `/workspace`:

```bash
-v /home/jeremy/dev/opscotch/community/opscotch-community:/workspace:ro
```

Then use tool paths like:

- resource test: `unit-tests/general/debug-print-body.test.ts`
- integration test: `tests/general/httpserver.test.json`

Do not include `community/opscotch-community/` in the MCP tool path when that directory is already mounted as `/workspace`.

### Common Mistake

If you mount:

```bash
-v /home/jeremy/dev/opscotch:/workspace:ro
```

then this is correct:

- `{"root":"workspace","path":"community/opscotch-community/unit-tests/general/debug-print-body.test.ts"}`

and this is wrong:

- `{"root":"workspace","path":"unit-tests/general/debug-print-body.test.ts"}`

because `/workspace/unit-tests/...` does not exist in that mount layout.

### Resource Test Constraint

`run_resource_unit_tests` expects the selected file to live under the mounted workspace's `unit-tests/` directory and to match `*.test.ts`.

### Workflow Test Constraint

`run_workflow_integration_test` expects the selected test file to be relative to the mounted workspace root. Some workflow tests also depend on sibling config/bootstrap/fixture files in the same test directory, so mount the project root that contains the full `tests/` tree, not just one file.

## Example

Minimal startup example (mounting the full repo as `/workspace`):

```bash
export OPSCOTCH_LEGAL_ACCEPTED='<base64-acceptance-blob>'

docker run --rm -p 18080:8080 \
  -e OPSCOTCH_LEGAL_ACCEPTED \
  -v /home/jeremy/dev/opscotch:/workspace:ro \
  ghcr.io/opscotch/opscotch-local-development-mcp:latest
```

With this mount layout, include the repo prefix in tool paths. For example:

- `{"root":"workspace","path":"community/opscotch-community/unit-tests/general/debug-print-body.test.ts"}`
- `{"root":"workspace","path":"community/opscotch-community/tests/general/httpserver.test.json"}`

Example with an additional named workspace:

```bash
docker run --rm -p 8080:8080 \
  -e OPSCOTCH_LEGAL_ACCEPTED \
  -e OPSCOTCH_LOCAL_MCP_ROOTS_JSON='{"workspace":"/workspace","app":"/workspace-app" }' \
  -v /home/jeremy/dev/opscotch/community/opscotch-community:/workspace:ro \
  -v /host/project/apps/example-app:/workspace-app:ro \
  ghcr.io/opscotch/opscotch-local-development-mcp:latest
```

## v1 Tools

- `check_local_test_capabilities`
- `scaffold_resource_unit_test_pair`
- `run_workflow_integration_test`
- `run_resource_unit_tests`

The execution tools return structured status first, with truncated logs and artifact paths attached second.

## Unreleased

- `run_resource_unit_tests` now accepts `unitTestPath` as either a single file or a directory.
- Directory targets recurse automatically, ignore non-`*.test.ts` files, and return one aggregate response with per-file execution detail.
- `diagnose_resource_unit_test_path`, guidance text, and tool descriptions now explain directory-mode behavior and empty-directory no-op results.

## Resource Scaffold Workflow

`scaffold_resource_unit_test_pair` reads `doc.inSchema(...)` and `doc.dataSchema(...)` from the resource source file itself, then writes:

- one fixture file under the target `fixtures/` directory
- one companion `*.test.ts` file beside the selected `unit-tests` output directory

The generated pair is schema-aware but app-agnostic: it derives deterministic sample `body` and `data` fixtures from the resource schemas, stubs `sendToStep(...)` with a generic success response, and produces a runnable starter test that callers can refine with app-specific assertions.

## Release Links

- Docker Image: `ghcr.io/opscotch/opscotch-local-development-mcp:3.1.8.rc6`
- Catalog: https://github.com/opscotch/opscotch-apps/blob/main/apps/opscotch/opscotch-local-development-mcp/3.1.8.rc6/README.md