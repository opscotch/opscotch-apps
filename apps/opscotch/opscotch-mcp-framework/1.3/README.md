# Opscotch MCP Framework

This package exposes the reusable Opscotch MCP Framework as a packaged public app.

## Bootstrap details

Run this deployment before sibling apps that register tools, prompts, or resources into it.

Each sibling deployment should:

- add an `allowDeploymentAccess` entry with `access: "call"` to deployment `mcp-framework`
- add a `runOnce` step that sends its registration payload to framework step `register`
- expose any callback steps referenced by registered tools, resources, or prompts

The framework deployment must also include `allowDeploymentAccess` entries with `access: "call"` for every sibling callback target it needs to invoke.

```
{
    "deploymentId": "mcp-framework",
    "packaging": {
        "packageId": "opscotch-mcp-framework"
    },
    "remoteConfiguration": "/apps/opscotch-mcp-framework.oapp",
    "startupPriority": 10,
    "allowHttpServerAccess": [
        {
            "id": "mcp-http",
            "port": 39590,
            "bindAddress": "127.0.0.1"
        }
    ],
    "allowDeploymentAccess": [
        {
            "id": "mcp-register",
            "access": "receive",
            "anyDeployment": true
        }
    ]
}
```

## Included MCP surface

- `POST /mcp`
- `GET /mcp`
- `GET /mcp` returns a lightweight JSON `200 OK` warm-up response
- `initialize`
- `tools/list`
- `tools/call`
- `resources/list`
- `resources/read`
- `prompts/list`
- `prompts/get`

## Registration model

Tools and prompts are normalized to `namespace.name`. Resource URIs must already be globally unique. Re-registering the same namespace requires `"replace": true`.

See the source app documentation at `community/opscotch-community/apps/mcp-framework/README.md` for a full registration payload example and sample sibling app wiring.

## Release Links

- Release: https://github.com/opscotch/opscotch-apps/releases/tag/opscotch-mcp-framework-1.3
- Docker Image: https://github.com/orgs/opscotch/packages/container/package/opscotch-mcp-framework

To use this as a docker image, here is an example Dockerfile:

```Dockerfile
FROM ghcr.io/opscotch/opscotch-mcp-framework:1.3 AS opscotch-mcp-framework
FROM ghcr.io/opscotch/opscotch-agent:latest

COPY --from=opscotch-mcp-framework /apps/opscotch-mcp-framework.oapp /apps/opscotch-mcp-framework

# your custom bootstrap
COPY bootstrap.json /config/bootstrap.json
```

## Signing Public Keys

| Key ID | Public Key |
| --- | --- |
| `opscotch-app` | `8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874` |
| `opscotch-mcp-framework-app` | `BBFC4D442A70486F4D4D5217AD0B57F24ED558AECC9AB9459D1F972F972F8B8C` |
| `opscotch-mcp-framework-1.x` | `770CF2E0135F98CBE66799B8C56AF6C7A55B5CC857DD63BB2FB96A50749CB00F` |

## Verify Artifact

Download `opscotch-mcp-framework.oapp`, `opscotch-mcp-framework.oapp.sig`, and `opscotch-mcp-framework.oapp.pem` from the release, then run:

```bash
cosign verify-blob \
  --signature opscotch-mcp-framework.oapp.sig \
  --certificate opscotch-mcp-framework.oapp.pem \
  --certificate-identity-regexp '^https://github.com/opscotch/builder/.github/workflows/app-release.yml@refs/(heads|tags)/.+$' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  opscotch-mcp-framework.oapp
```
