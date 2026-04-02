# Opscotch Coder MCP

This image runs the Opscotch Coder MCP service for authoring and validating Opscotch workflows.

It exposes an MCP endpoint and includes Opscotch-specific tools and resources for:

- workflow validation
- bootstrap validation
- step, trigger, and processor validation
- Opscotch API and context lookup

## Running the container

Before starting this container, review and accept the Opscotch legal terms as described in the runtime startup documentation:

- https://www.opscotch.co/docs/administrating/agent#legal-acceptance
- https://www.opscotch.co/docs/administrating/agent#running-the-runtime-in-docker

This container is built on the standard Opscotch agent runtime image, so it will not start until legal acceptance has been supplied.

```bash
export OPSCOTCH_LEGAL_ACCEPTED='<base64-acceptance-blob>'
docker run \
  -e OPSCOTCH_LEGAL_ACCEPTED \
  -p 39590:39590 \
  ghcr.io/opscotch/opscotch-coder-mcp:latest
```

The MCP endpoint will then be available on `http://localhost:39590/mcp`.

## Registered MCP features

This app registers Opscotch-focused tools and resources intended to help LLMs author Opscotch workflows, including schema validation, bootstrap validation, and API/context lookup helpers.

## Release Links

- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-coder-mcp
## Signing Public Keys

| Key ID | Public Key |
| --- | --- |
| `opscotch-app` | `8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874` |
| `opscotch-coder-mcp-app` | `3FD68C5A128C79E808CB854F19E3F3C820774C91CF59808253E97707832AE3DB` |
| `opscotch-coder-mcp-1.x` | `CC39ED2B54096D0980EDD49348E03A8494E9FF0539B8E29A5B42A8DA5C811EE7` |

## Verify Artifact

Download `opscotch-coder-mcp.oapp`, `opscotch-coder-mcp.oapp.sig`, and `opscotch-coder-mcp.oapp.pem` from the release, then run:

```bash
cosign verify-blob \
  --signature opscotch-coder-mcp.oapp.sig \
  --certificate opscotch-coder-mcp.oapp.pem \
  --certificate-identity-regexp '^https://github.com/opscotch/builder/.github/workflows/app-release.yml@refs/(heads|tags)/.+$' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  opscotch-coder-mcp.oapp
```
