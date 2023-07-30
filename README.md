# Kellnr Helm-Chart

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/kellnr)](https://artifacthub.io/packages/search?repo=kellnr)

This repository provides a [Helm](https://helm.sh/) Chart for [Kellnr](https://www.bitfalter.com/kellnr), the Rust registry for self-hosting crates.

## Installation

Add the _Bitfalter_ helm repository which contains _Kellnr_:

```bash
# Add  repository
helm repo add bitfalter https://bitfalter.github.io/helm

# Update to latest version
helm repo update
```

For a list of possible configuration flags see below: [Configuration](#configuration)

To install a minimal _Kellnr_ installation with in-memory storage run the command below. The in-memory storage variant is useful for testing but not recommended for production as all data (crates, users, ...) will be lost if the container restarts.

```bash
# Replace the license with your license key and set the address where Kellnr will be reachable.
helm install kellnr bitfalter/kellnr --set kellnr.license="..." --set kellnr.apiAddress="kellnr.example.com"
```

For a persistent _Kellnr_ instance, a _PersistentVolumeClaim_ (PVC) is needed. The helm chart can create a _PersistentVolumeClaim_ and _PersistentVolume_ (PV), if you don't have one already.

```bash
# Use an existing PersistentStorage (storage_name) to get a PersistentStorageClaim.
# The storage class can be overwritten with "pvc.storageClassName" and defaults to "manual".
helm install kellnr bitfalter/kellnr \
    --set pvc.enabled=true --set pvc.name="storage_name" \
    --set kellnr.license="..." \
    --set kellnr.apiAddress="kellnr.example.com"
```

```bash
# Create a new PersistentVolume and a corresponding claim. The host path e.g. "/mnt/kellnr" has to exists before the PV is created.
helm install kellnr bitfalter/kellnr \
    --set pv.enabled=true --set pv.path="/mnt/kellnr" \
    --set pvc.enabled=true \
    --set kellnr.license="..." \
    --set kellnr.apiAddress="kellnr.example.com"
```

For information about the _Cargo_ setup and default values, see the official [Kellnr documentation](https://www.bitfalter.com/documentation).

## Upgrade

Upgrade _Kellnr_ by updating the Helm repository and applying the latest version.

```bash
# Update helm repositories
helm repo update

# Upgrade Kellnr
helm upgrade kellnr bitfalter/kellnr
```

## Configuration

All settings can be set with the `--set name=value` flag on `helm install`. Some settings are required and have to be provided other are recommended for security reasons.

### Basics

Basic settings to configure _Kellnr_.

| Setting           | Required    | Description                                                                                   | Default                          |
| ----------------- | ----------- | --------------------------------------------------------------------------------------------- | -------------------------------- |
| kellnr.license    | Yes         | _Kellnr_ license key.                                                                         |                                  |
| kellnr.apiAddress | Yes         | Address (IP or hostname) where _Kellnr_ is reachable                                          |                                  |
| kellnr.adminPwd   | Recommended | Password of the admin user used by the web-ui. The password can be changed anytime in the UI. | kellnr                           |
| kellnr.adminToken | Recommended | Token used by Cargo for the admin user. The token can be changed anytime in the UI.           | Zy9HhJ02RJmg0GCrgLfaCVfU6IwDfhXD |
| kellnr.apiProtocol | No         | Protocol where _Kellnr_ is reachable. Either _http_ or _https_.                              | http                            |
| kellnr.logLevel      | No          | Set the log level. Has to be one of _trace_, _debug_, _info_, _warn_, _error_.| info                            |
| kellnr.logLevelRocket      | No          | Set the log level for the Rocket web framework. Has to be one of _trace_, _debug_, _info_, _warn_, _error_.| warn                            |
| kellnr.logFormat     | No          | Set the log format. Has to be one of _compact_, _pretty_, _json_ | compact                            |
|kellnr.cratesIoProxy| No | Enable [crates.io](https://crates.io/) proxy mode to cache crates in _Kellnr_. The crates.io index takes ~10GB of storage + storage for all cached crates.| false |
|kellnr.cratesIoNumThreads | No | Number of threads used to update the crates.io index. | 10 |
|kellnr.rustdocAutoGen| No | Enable automatic _rustdoc_ generation for uploaded _crates_. | true |
|kellnr.cacheSize | No | Number of crates cached in-memory to decrease disk I/O. If set to `0` the cache is disabled. | 1000 |
|kellnr.maxCrateSize | No | Max. allowed upload size for crates in MB. | 100 |
|kellnr.maxDocsSize | No | Max. allowed upload size for crate docs in MB. | 100 |
|kellnr.gitIndex | No | Enable or disable the git index. Needed for cargo < 1.7.0. Recommended to disable if you use an up to date cargo version. | true |
|kellnr.authRequired | No | Enable or disable authentication on pull. | false |

### Service

Settings to configure the web-ui/API endpoint service and the crate index service.

| Setting                | Required | Description                                                   | Default   |
| ---------------------- | -------- | ------------------------------------------------------------- | --------- |
| service.api.type       | No       | Type of the service that exports the API and web-ui endpoint. | ClusterIP |
| service.api.port       | No       | Port of the service that exports the API and web-ui endpoint. | 80        |
| service.api.port_proxy | No       | If a proxy is used in front of Kellnr, specify the port of the proxy here. If no proxy is used, set to `service.api.port` value. | 80        |
| service.index.type     | No       | Type of the service that exports the crate index.             | NodePort  |
| service.index.port     | No       | Internal port of the service that exports the crate index.    | 9418      |
| service.index.nodePort | No       | External port of the service that exports the crate index.    | 30418     |

### Ingress

Setting to configure the ingress route for the web-ui and API.

| Setting             | Required | Description                                      | Default |
| ------------------- | -------- | ------------------------------------------------ | ------- |
| ingress.enabled     | No       | Enable an Kubernetes ingress route for _Kellnr_. | true    |
| ingress.className   | No       | Set an ingress className.                        | ""      |
| ingress.annotations | No       | Set ingress annotations.                         | {}      |
| ingress.tls.secretName | No   | Set the secret name for a TLS certificate         | kellnr-cert-secret |

### TLS Certificate

Settings to configure a TLS certificate with cert-manager. **Important** If you use _Kellnr_ with TLS,
make sure to set `kellnr.apiProtocol` to `https`.

| Setting | Required | Description | Default |
| ---------------------- | -------- | ------------------------------------------------------------- | --------- |
| certificate.enabled | No | Enable automatic TLS certificate generation with cert-mananger. | false |
| certificate.secretName | No | The name of the certificate secret which an ingress can refer to. | kellnr-cert-secret |
| certificate.issuerRef.kind | No | The kind of the certificate issuer, e.g. _ClusterIssuer_, or _Issuer_ | ClusterIssuer |
| certificate.issuerRef.name | Yes (if enabled) | The name of the certificate issuer e.g. cert-manager | "" |

### Trust a Certificate

Besides the option of using a certificate to secure the access to _Kellnr_, you can import a certificate into _Kellnr_ which it will trust as a
root certificate.

> Kellnr needs to trust it's own root certificate to be able to generate Rustdocs automatically!

If you use a self-signed certificate that is not trusted by default, you can import it into _Kellnr_ on application startup.

| Setting | Required | Description | Default |
| ---------------------- | -------- | ------------------------------------------------------------- | --------- |
| importCert.enabled | No | Enable the import of a root certificate to trust | false |
| importCert.useExisting | No | If you have an existing _ConfigMap_ that contains a certificate, set this flag to _true_, else a new _ConfigMap_ is created.  | false |
| importCert.configMapName | No | Set the name of the created or existing _ConfigMap_ to use. | "kellnr-cert" |
| importCert.volumeName | No | The volume name under which the _ConfigMap_ is mounted into the pod. | "kellnr-cert-storage" |
| importCert.certificate | No | The certificate to trust in the standard PEM format. See example below. | "" |

Example for `importCert.certificate`:

```yaml
importCert:
  enabled: true
  certificate: |
    —–BEGIN CERTIFICATE—–
    MIIDdTCCAl2gAwIBAgILBAAAAAABFUtaw5QwDQYJKoZIhvcNAQEFBQAwVzELMAkG
    YWxTaWduIG52LXNhMRAwDgYDVQQLEwdSb290IENBMRswGQYDVQQDExJHbG9iYWxT
    aWduIFJvb3QgQ0EwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDaDuaZ
    …
    jc6j40+Kfvvxi4Mla+pIH/EqsLmVEQS98GPR4mdmzxzdzxtIK+6NiY6arymAZavp
    38NflNUVyRRBnMRddWQVDf9VMOyGj/8N7yy5Y0b2qvzfvGn9LhJIZJrglfCm7ymP
    HMUfpIBvFSDJ3gyICh3WZlXi/EjJKSZp4A==
    —–END CERTIFICATE—–
```

### DNS

You can set DNS servers for _Kellnr_ which should be used instead of the default one.

> Kellnr needs to be able to resolve it's own domain name, to be able to generate Rustdocs automatically.

| Setting | Required | Description | Default |
| ---------------------- | -------- | ------------------------------------------------------------- | --------- |
| dns.enabled | No | Enable an additional _dnsPolicy_ | false |
| dns.dnsPolicy | No | Set the _dnsPolicy_ | "None" |
| dns.dnsConfig.nameservers | No | List of nameservers to use. | - "" |
| dns.dnsConfig.searches | No | List of searches to use. | - "" |

### Persistence

A _PersistentVolume_ can be created for _Kellnr_ to hold all stored data.

| Setting             | Required         | Description                                                        | Default |
| ------------------- | ---------------- | ------------------------------------------------------------------ | ------- |
| pv.enabled          | No               | Enable a _PersistentVolume_ to store the data from _Kellnr_        | false   |
| pv.name             | No               | Name of the _PersistentVolume_                                     | kellnr  |
| pv.storageClassName | No               | storageClassName of the _PersistentVolume_                         | manual  |
| pv.storage          | No               | Size of the storage.                                               | 5Gi   |
| pv.path             | Yes (if enabled) | Host path to the storage. Needs to exists before the PV is created | ""      |

A _PersistentVolumeClaim_ can be used by _Kellnr_ to hold all stored data.

| Setting              | Required | Description                                                      | Default |
| -------------------- | -------- | ---------------------------------------------------------------- | ------- |
| pvc.enabled          | No       | Enable a _PersistentVolumeClaim_ to store the data from _Kellnr_ | false   |
| pvc.name             | No       | Name of the _PersistentVolumeClaim_                              | kellnr  |
| pvc.storageClassName | No       | storageClassName of the _PersistentVolumeClaim_                  | manual  |
| pvc.storage          | No       | Size of the storage.                                             | 5Gi   |

For a full set of possible variables see: [values.yaml](./charts/kellnr/values.yaml)

## Feedback

If the Helm Chart does not work for you for any reason or you need a feature, feel free to create a Github issue or [contact us](mailto:contact@bitfalter.com) via e-mail.

## Release a new version

Run the following steps to release a new version of the chart.

1. Change the `appVersion` in [Chart.yaml](./charts/kellnr/Chart.yaml) to the current [Docker image version](https://hub.docker.com/repository/docker/bitfalter/kellnr).
2. Increase the `version` in [Chart.yaml](./charts/kellnr/Chart.yaml).
3. Push or create a Pull-Request to the `main`/`master` branch.
