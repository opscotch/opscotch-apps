# Standalone Opscotch Package

This is a standalone docker container for producing opscotch packages.

Before starting this container, review and accept the Opscotch legal terms as described in the runtime startup documentation, you must follow the "prod mode" requirements:

- https://www.opscotch.co/docs/administrating/agent#legal-acceptance
- https://www.opscotch.co/docs/administrating/agent#running-the-runtime-in-docker

There are some important things to note with the packager as a docker container.
- You need to mount all files that you need to work with, this includes resources and workflow when doing a compare.
- Its likely that you will need the [community resources](https://github.com/opscotch/opscotch-community) - mount these to `/opscotch-community-resources`
- In this container there are two additional resource mounts for app specifc resources:
  - `/local-resources1`
  - `/local-resources2`
- When you are comparing workflows you will also need to mount these directories to 
  - `/workflow-compare1`
  - `/workflow-compare2`

# Versions

## 3.1.8.7
- embeds packager **1.2.17** (hopscotch#822 Fast Boot workflow version stamping)
- keeps licenser 1.1.0 on agent 3.1.8.1

## 3.1.8.6
- pins packager agent to `ghcr.io/opscotch/opscotch-agent:3.1.8.1` (GraalVM 25 / hopscotch `3.1.8-1` Full Release)
- keeps packager **1.2.16** + licenser 1.1.0

## 3.1.8.5
- embeds packager **1.2.16** (hopscotch#803 stream adoption) so Fast Boot package bytes survive HTTP write and `/fast-boot-inspect` succeeds
- keeps licenser 1.1.0 on agent 3.1.8

## 3.1.8.4
- keeps licenser 1.1.0 + packager 1.2.15 on agent 3.1.8
- creates `/tmp/licenser-persistence` and points licenser `persistenceRoot` there so the embedded licenser activates (3.1.8.3 failed load when the relative persistence dir was missing)

## 3.1.8.3
- embeds `opscotch-licenser` 1.1.0 alongside packager 1.2.15 on agent 3.1.8
- licenser listens on port **39576** (`POST /inspect` for Fast Boot license decode)
- packager remains on port **39575**

## 3.1.8.2
- keeps packager agent on `ghcr.io/opscotch/opscotch-agent:3.1.8`
- uses `opscotch-packager` 1.2.15 (Fast Boot package-time `noRoot`; license is request-only)
- unblocks `/fast-boot-package` when the Fast Boot license root is not registered in the packager agent

## 3.1.8.1
- pins packager agent to `ghcr.io/opscotch/opscotch-agent:3.1.8`
- uses `opscotch-packager` 1.2.14 (Fast Boot + `@opscotch-include`; hopscotch#800)

## 3.1.7.2
- interim pin to agent `3.1.7.2` + packager 1.2.13 while 1.2.14 was prepared

## 3.1.8
- first attempt with packager 1.2.13 failed on agent 3.1.8 include expansion; superseded by 3.1.8.1

## 1.1.7
- updates packager agent to 3.1.6

## 1.1.5
- uses ospcotch-packager-1.2.6 - fixes http headers

# Start up details

To use this container start it up, mounting the required directories, for example

This container is built on the standard Opscotch agent runtime image, so the packager will not start until legal acceptance has been supplied. For regular usage, set `OPSCOTCH_LEGAL_ACCEPTED` in your shell or system environment once, then pass it through to `docker run`.

```
export OPSCOTCH_LEGAL_ACCEPTED='<base64-acceptance-blob>'
docker run \
	-e OPSCOTCH_LEGAL_ACCEPTED \
	-v "/local/opscotch-community/resources:/opscotch-community-resources" \
	-v "/local-app-resources:/local-resources1" \
	-v "/additional-local-app-resources:/local-resources2" \
    -v "/some-workflows-to-compare:/workflow-compare1" \
    -v "/more-workflows-to-compare:/workflow-compare2" \
	-p "39575:39575"
    ghcr.io/opscotch/opscotch-packager-standalone:latest
```

This will start on port 39575.

# Bootstrap used in the docker container

If your reference this is the bootstrap that is used in the docker container

```
[
  {
    "deploymentId": "opscotch-packager",
    "remoteConfiguration": "/apps/opscotch-packager.oapp",
    "packaging" : {
      "packageId" : "opscotch-packager",
      "packagerIdentities" : [ "opscotch-packager-identiy" ],
      "requiredSigners": [ 
        { 
          "id" : "opscotch-app",
          "description" : "Official opscotch app"
        },
        {
          "id" : "opscotch-packager-app",
          "description" : "Official opscotch Packager"
        }, 
        {
          "id" : "opscotch-packager-1.x",
              "description" : "Opscotch Packager v2 version 1.x lock"
        }
      ]
    },
    "keys" : [
      {
        "id" : "opscotch-packager-identiy",
        "type" : "public",
        "purpose" : "authenticated",
        "keyHex" : "D1F8F5CD6424A638A938903EBEA7C140A678873417E5642A3993D2C1BE74A01E"
      },
      {
        "id" : "opscotch-app",
        "type" : "public",
        "purpose" : "sign",
        "keyHex" : "8C9AED01FF5E6695E4464E754697F22D653A9FB35E1233627D81D91F89CF2874"
      },
      {
        "id" : "opscotch-packager-app",
        "type" : "public",
        "purpose" : "sign",
        "keyHex" : "6D9841078C83239B1CF148BF92A533A4C8BA4901DA0AD1ADDE2FE2D23BA3F156"
      },
      {
        "id" : "opscotch-packager-1.x",
        "type" : "public",
        "purpose" : "sign",
        "keyHex" : "5912C087D6C958A84433F666784CE228B17EF34E39E5D9C7FD2BBA5927A760F9"
      }
    ],
    "persistenceRoot": "persistence",
    "errorHandling": {
      "enableLocalLogging": true
    },
    "workflow": {
      "errorHandling": {
        "enableLocalLogging": true
      },
      "metricOutput": {
        "enabled": false
      }
    },
    "allowHttpServerAccess": [
      {
        "id": "api",
        "port": 39575,
        "bindAddress": "0.0.0.0"
      }
    ],
    "allowFileAccess": [
      {
        "id": "localResources1",
        "directoryOrFile": "/local-resources1",
        "READ": true,
        "LIST": true
      },
      {
        "id": "localResources2",
        "directoryOrFile": "/local-resources2",
        "READ": true,
        "LIST": true
      },
      {
        "id": "communityResources",
        "directoryOrFile": "/opscotch-community-resources",
        "READ": true,
        "LIST": true
      },
      {
        "id": "workflowCompare1",
        "directoryOrFile": "/workflow-compare1",
        "READ": true,
        "LIST": true
      },
      {
        "id": "workflowCompare2",
        "directoryOrFile": "/workflow-compare2",
        "READ": true,
        "LIST": true
      }
    ],
    "data": {
      "resourceIds": [
        "localResources1",
        "localResources2",
	      "communityResources"
      ],
      "workflowSourceIds": [
        "workflowCompare1",
        "workflowCompare2"
      ]
    }
  }
]
```

## Release Links

- Docker Image: `ghcr.io/opscotch/opscotch-packager-standalone:3.1.8.7`
- Catalog: https://github.com/opscotch/opscotch-apps/blob/main/apps/opscotch/opscotch-packager-standalone/3.1.8.7/README.md