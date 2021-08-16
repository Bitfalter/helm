# Kellnr Helm-Chart

This repository provides a [Helm](https://helm.sh/) Chart for [Kellnr](https://www.bitfalter.com/kellnr), the Rust registry for self-hosting Crates.

## Installation

Add the *Bitfalter* helm repository which contains *Kellnr*:
```bash
# Add  repository
helm repo add bitfalter https://bitfalter.github.io/helm

# List latest version
helm search repo bitfalter
```

For a list of possible configuration flags see below: [Configuration](#Configuration)

To install a minimal *Kellnr* installation with in-memory storage run the command below. The in-memory storage variant is useful for testing but not recommended for production as all data (crates, users, ...) will be lost if the container restarts.

```bash
# Replace the license with your license key and set the address where Kellnr will be reachable.
helm install bitfalter/kellnr --set kellnr.license="..." --set kellnr.apiAddress="kellnr.example.com"
```

For a persistent *Kellnr* instance, a *PersistentVolumeClaim* (PVC) is needed. The helm chart can create a *PersistentVolumeClaim* and *PersistentVolume* (PV), if you don't have one already.

```bash
# Use an existing PersistentStorage (storage_name) to get a PersistentStorageClaim.
# The storage class can be overwritten with "pvc.storageClassName" and defaults to "manual".
helm install bitfalter/kellnr  \ 
    --set pvc.enabled=true --set pvc.name="storage_name"
    --set kellnr.license="..." \
    --set kellnr.apiAddress="kellnr.example.com"
```

```bash
# Create a new PersistentVolume and a corresponding claim. The host path e.g. "/mnt/kellnr" has to exists before the PV is created.
helm install bitfalter/kellnr  \ 
    --set pv.enabled=true --set pv.path="/mnt/kellnr" \
    --set pvc.enabled=true \
    --set kellnr.license="..." \
    --set kellnr.apiAddress="kellnr.example.com"
```

For information about the *Cargo* setup and default values, see the official [Kellnr documentation](https://www.bitfalter.com/documentation).

## Configuration

All settings can be set with the `--set name=value` flag on `helm install`. Some settings are required and have to be provided other are recommended for security reasons.

### Basics

Basic settings to configure *Kellnr*.

|Setting|Required|Description|Default|
|---|---|---|---|
|kellnr.license| Yes | *Kellnr* license key. |  |
|kellnr.apiAddress | Yes | Address (IP or hostname) where *Kellnr* is reachable | |
|kellnr.adminPwd| Recommended | Password of the admin user used by the web-ui. The password can be changed anytime in the UI. | kellnr|
|kellnr.adminToken| Recommended | Token used by Cargo for the admin user. The token can be changed anytime in the UI. | Zy9HhJ02RJmg0GCrgLfaCVfU6IwDfhXD |
|kellnr.debug | No | Start *Kellnr* in debug mode for additional console output. | false |

### Service

Settings to configure the web-ui/API endpoint service and the Crate index service.

|Setting|Required|Description|Default|
|---|---|---|---|
|service.api.type| No | Type of the service that exports the API and web-ui endpoint. | ClusterIP |
|service.api.port| No | Port of the service that exports the API and web-ui endpoint. | 8000 |
|service.index.type | No | Type of the service that exports the Crate index. | NodePort |
|service.index.port | No | Internal port of the service that exports the Crate index. | 9418 |
|service.index.nodePort | No | External port of the service that exports the Crate index. | 30418 |

### Ingress

Setting to configure the ingress route for the web-ui and API.

|Setting|Required|Description|Default|
|---|---|---|---|
|ingress.enabled | No | Enable an Kubernetes ingress route for *Kellnr*. | true |
|ingress.className | No | Set an ingress className. | "" |
|ingress.annotations | No | Set ingress annotations. | {} |

### Persistence

A *PersistentVolume* can be created for *Kellnr* to hold all stored data.

|Setting|Required|Description|Default|
|---|---|---|---|
|pv.enabled | No | Enable a *PersistentVolume* to store the data from *Kellnr* | false |
|pv.name | No | Name of the *PersistentVolume* | kellnr |
|pv.storageClassName| No | storageClassName of the *PersistentVolume* | manual |
|pv.storage | No | Size of the storage. | 500Mi |
|pv.path | Yes (if enabled) | Host path to the storage. Needs to exists before the PV is created | "" |

A *PersistentVolumeClaim* can be used by *Kellnr* to hold all stored data.

|Setting|Required|Description|Default|
|---|---|---|---|
|pvc.enabled | No | Enable a *PersistentVolumeClaim* to store the data from *Kellnr* | false |
|pvc.name | No | Name of the *PersistentVolumeClaim* | kellnr |
|pvc.storageClassName| No | storageClassName of the *PersistentVolumeClaim* | manual |
|pvc.storage | No | Size of the storage. | 500Mi |


For a full set of possible variables see: [values.yaml](./charts/kellnr/values.yaml)

## Feedback

If the Helm Chart does not work for you for any reason or you need a feature, feel free to create a Github issue or [contact us](mailto:contact@bitfalter.com) via e-mail.

## Release a new version

Run the following steps to release a new version of the chart.

1. Change the `appVersion` in [Charts.yaml](./charts/kellnr/Charts.yaml) to the current [Docker image version](https://hub.docker.com/repository/docker/bitfalter/kellnr).
2. Increase the `version` in [Charts.yaml](./charts/kellnr/Charts.yaml).
3. Push or create a Pull-Request to the `main` branch.