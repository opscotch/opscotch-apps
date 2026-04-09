# Opscotch Local Development MCP

`ghcr.io/opscotch/opscotch-local-development-mcp:latest` is a local-only streamable HTTP MCP server for running privileged verification flows against a mounted developer workspace.

This image complements the public Opscotch Coder MCP. It is not a public execution surface.

## Endpoint

- `GET /mcp` exposes the streamable SSE endpoint
- `POST /mcp` accepts JSON-RPC MCP requests

## Required Runtime Environment

- `OPSCOTCH_LEGAL_ACCEPTED`
- `OPSCOTCH_LOCAL_MCP_WORKSPACE_ROOT`
- `OPSCOTCH_LOCAL_MCP_LICENSE_ROOT`

Optional runtime overrides:

- `OPSCOTCH_LOCAL_MCP_ARTIFACT_ROOT` default `/tmp/opscotch-local-mcp/artifacts`
- `OPSCOTCH_LOCAL_MCP_RUN_ROOT` default `/tmp/opscotch-local-mcp/runs`
- `OPSCOTCH_LOCAL_MCP_PORT` default `8080`

## Mounted Path Contract

The server exposes mounted paths through logical roots. v1 supports:

- `workspace`
- `license`
- `artifacts` when `OPSCOTCH_LOCAL_MCP_ARTIFACT_ROOT` is configured

All tool inputs must stay inside these allowlisted roots. Absolute paths and traversal outside the root are rejected.

## Workspace Mounting

The most important rule is:

- the path you pass to a tool must be relative to the directory you mounted as the workspace root

This server does not interpret paths relative to your host repository root unless that exact repository root is what you mounted as `/workspace`.

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

Example for `community/opscotch-community` mounted as `/workspace`:

```bash
export OPSCOTCH_LEGAL_ACCEPTED='<base64-acceptance-blob>'

docker run --rm -p 8080:8080 \
  -e OPSCOTCH_LEGAL_ACCEPTED \
  -e OPSCOTCH_LOCAL_MCP_WORKSPACE_ROOT=/workspace \
  -e OPSCOTCH_LOCAL_MCP_LICENSE_ROOT=/license \
  -e OPSCOTCH_LOCAL_MCP_ARTIFACT_ROOT=/artifacts \
  -v /home/jeremy/dev/opscotch/community/opscotch-community:/workspace:ro \
  -v /path/to/license-dir:/license:ro \
  -v /tmp/opscotch-local-mcp-artifacts:/artifacts \
  ghcr.io/opscotch/opscotch-local-development-mcp:latest
```

## v1 Tools

- `check_local_test_capabilities`
- `run_workflow_integration_test`
- `run_resource_unit_tests`

The execution tools return structured status first, with truncated logs and artifact paths attached second.

## Release Links

- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-local-development-mcp