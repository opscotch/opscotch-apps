# Opscotch Coder MCP

This image runs the Opscotch Coder MCP service for authoring and validating Opscotch workflows.

## 3.1.8rc4

- States that `validate_bootstrap_host` checks one `allowExternalHostAccess` host permitter, not the array and not a full bootstrap record.
- `transport` `http` requires `host`. `transport` `inProc` requires `inProcServerId`.
- Engineer guidance for Opscotch 3.1.8 tells callers to validate each host item before embedding it.

## 3.1.8rc3

- Refreshes the Opscotch 3.1.8 LLM guidance in the image: architect, engineer, operations, and root guidance.
- Updates both the current skill files and `docs/llm/versions/3.1.8/`, which is the tree `get_guidance` reads.
- Keeps the 3.1.8-rc1 bootstrap header-size schema, including architect guidance for `packageHttpMaxHeaderSize` and `workflow.httpMaxHeaderSize`.
- Updates the unversioned API reference to the copies on the docs `dev` branch. The versioned 3.1.8 API reference was already that copy.

## 3.1.8rc2

- Packages the public MCP server on `ghcr.io/opscotch/opscotch-agent:3.1.8-rc1-dev`.
- Updates the Opscotch 3.1.8 bootstrap schema to the `3.1.8-rc1` release schema.
- Adds versioned generator inputs for `packageHttpMaxHeaderSize` on bootstrap records and `httpMaxHeaderSize` on bootstrap workflow settings.
- Validator resources use file `$ref`s to the versioned schema files instead of embedded schema copies. The image sets `RESOURCE_DIR=/docs/static/versions` so the agent resolves `3.1.7/...` and `3.1.8/...` refs, and packaging mounts that same tree at `/local-resources2`.

## 3.1.8rc1

- Adds version-aware MCP tooling for both Opscotch 3.1.7 and 3.1.8 targets.
- Includes immutable versioned schema, LLM guidance, and community-example snapshots.
- Resolves numeric patch targets to their supported base release and reports compatible versions in every tool result.
- Uses `ghcr.io/opscotch/opscotch-agent-beta:3.1.8-36-dev-linux-amd64` for the packaged runtime.

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

# Packaging and keys

```json
{
  "deploymentId": "",
  "remoteConfiguration": "https://github.com/opscotch/opscotch-apps/releases/download/opscotch-coder-mcp-3.1.8rc4/opscotch-coder-mcp.oapp",
  "packaging": {
    "packageId": "opscotch-coder-mcp",
    "requiredSigners": [
      {
        "id": "opscotch-app"
      },
      {
        "id": "opscotch-coder-mcp-app"
      },
      {
        "id": "opscotch-coder-mcp-1.x"
      }
    ]
  },
  "keys": [
    {
      "id": "opscotch-app",
      "purpose": "sign",
      "type": "public",
      "keyHex": "8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874"
    },
    {
      "id": "opscotch-coder-mcp-app",
      "purpose": "sign",
      "type": "public",
      "keyHex": "3FD68C5A128C79E808CB854F19E3F3C820774C91CF59808253E97707832AE3DB"
    },
    {
      "id": "opscotch-coder-mcp-1.x",
      "purpose": "sign",
      "type": "public",
      "keyHex": "CC39ED2B54096D0980EDD49348E03A8494E9FF0539B8E29A5B42A8DA5C811EE7"
    }
  ]
}
```

## Release Links

- Release: https://github.com/opscotch/opscotch-apps/releases/tag/opscotch-coder-mcp-3.1.8rc4
- Docker Image: `ghcr.io/opscotch/opscotch-coder-mcp:3.1.8rc4`
- Catalog: https://github.com/opscotch/opscotch-apps/blob/main/apps/opscotch/opscotch-coder-mcp/3.1.8rc4/README.md

To use this as a docker image, here is an example Dockerfile:

```Dockerfile
FROM ghcr.io/opscotch/opscotch-coder-mcp:3.1.8rc4 AS opscotch-coder-mcp
FROM ghcr.io/opscotch/opscotch-agent:latest

COPY --from=opscotch-coder-mcp /apps/opscotch-coder-mcp.oapp /apps/opscotch-coder-mcp

# your custom bootstrap
COPY bootstrap.json /config/bootstrap.json
```

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
