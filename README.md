# Mimir bundle

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [Mimir bundle](#mimir-bundle)
    - [Usage](#usage)
        - [Recommended or minimal deployment](#recommended-or-minimal-deployment)
        - [Reader-Writer path deployment:](#reader-writer-path-deployment)
    - [Overlays](#overlays)

<!-- markdown-toc end -->



Mimir is an open source, horizontally scalable, highly available, multi-tenant TSDB for long-term storage for Prometheus.

This Juju bundle deploys Mimir and a small object storage server, consisting of the following interrelated charms:

- [Mimir Coordinator](https://charmhub.io/mimir-coordinator-k8s) ([source](https://github.com/canonical/mimir-coordinator-k8s-operator))
- [Mimir Worker](https://charmhub.io/mimir-worker-k8s) ([source](https://github.com/canonical/mimir-worker-k8s-operator))
- [s3integrator](https://charmhub.io/s3-integrator) ([source](https://github.com/canonical/s3-integrator))

This bundle is under development.
Join us on:

- [Discourse](https://charmhub.io/topics/canonical-observability-stack)
- [Matrix chat](https://matrix.to/#/#cos:ubuntu.com)

## Usage

Before deploying the bundle you may want to create a dedicated model for Mimir components:

```shell
juju add-model mimir
juju switch mimir
```

### Recommended or minimal deployment

Deploy the bundle from a local file by running:

```shell
tox -e render-bundle -- bundle.yaml --template=bundle.yaml.j2 --channel=edge --recommended_deployment=True
juju deploy ./bundle.yaml --trust
```

Note that the `--recommended_deployment=True` parameter will generate a bundle with:

- 3 `ingester` units
- 2 `querier` units
- 2 `query-scheduler` units
- 1 `alertmanager` unit
- 1 `compactor` unit
- 1 `distributor` unit
- 1 `query-frontend` unit
- 1 `ruler` unit
- 1 `store-gateway` unit

and

- 1 `s3-integrator` unit
- 1 `coordinator` unit

if `--recommended_deployment` parameter is not used, will generate a bundle with:

- 1 `ingester` units
- 1 `querie`r units
- 1 `query-scheduler` units
- 1 `alertmanager` unit
- 1 `compactor` unit
- 1 `distributor` unit
- 1 `query-frontend` unit
- 1 `ruler` unit
- 1 `store-gateway` unit

and

- 1 `s3-integrator` unit
- 1 `coordinator` unit


### Reader-Writer path deployment:

Deploy the [read and write paths](https://grafana.com/docs/mimir/latest/get-started/about-grafana-mimir-architecture/#grafana-mimir-components) from a local file by running:

```shell
tox -e render-bundle -- bundle.yaml --template=bundle_reader_writer.yaml.j2 --channel=edge
juju deploy ./bundle.yaml --trust
```

This bundle will deploy:

- 1 `writer` unit
- 1 `reader` unit

and

- 1 `s3-integrator` unit
- 1 `coordinator` unit


Currently the bundle is available only on the `edge` channel, using `edge` charms.
When the charms graduate to `beta`, `candidate` and `stable`, we will issue the bundle in the same channels.

The `--trust` option is needed by the charms in the `mimir-lite` bundle to be able to patch their K8s services to use the right ports (see this [Juju limitation](https://bugs.launchpad.net/juju/+bug/1936260)).


## Overlays

TODO: Fix this outdated section

We also make available some [**overlays**](https://juju.is/docs/sdk/bundle-reference) as convenience.
A Juju overlay is a set of model-specific modifications, which reduce the amount of commands needed to set up a bundle like COS Lite.
Specifically, we offer the following overlays:

* the [`cos-relations` overlay](https://raw.githubusercontent.com/canonical/cos-lite-bundle/main/overlays/cos-relations-overlay.yaml) establishes relationshps between the charms in this bundle and those in the [`cos-lite` bundle](https://github.com/canonical/cos-lite-bundle), as well as exposing Mimir as a Prometheus compatible remote-write targets as an Juju offer which may be consumed as part of a cross-model relatoin.
* the [`storage-small` overlays](https://raw.githubusercontent.com/canonical/cos-lite-bundle/main/overlays/storage-small-overlay.yaml) provides a setup of the various storages for the Mimir bundle charms for a small setup.
  Using an overlay for storage is fundamental for a productive setup, as you cannot change the amount of storage assigned to the various charms after the deployment of the Mimir Bundle.

In order to use the overlays above, you need to:

1. Download the overlays (or clone the repository)
2. Pass the `--overlay <path-to-overlay-file-1> --overlay <path-to-overlay-file-2> ...` arguments to the `juju deploy` command

For example, to deploy the COS Lite bundle, then the Mimir bundle with the cos-relations overlay, you would do the following:

```sh
curl -L https://raw.githubusercontent.com/canonical/mimir-bundle/main/overlays/cos-relations-overlay.yaml -O
juju deploy cos-lite-bundle --channel=edge --trust

juju deploy mimir-bundle --channel=edge --trust --overlay ./cos-relations-overlay.yaml
```
```shell
./render_bundle.py bundle.yaml --channel=edge
charmcraft pack
charmcraft upload mimir-bundle.zip
charmcraft release mimir-bundle --channel=edge --revision=1
```
