# Mimir bundle

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [Mimir bundle](#mimir-bundle)
    - [Usage](#usage)
        - [Recommended distributed deployment](#recommended-distributed-deployment)
        - [Reader-Writer path deployment:](#reader-writer-mode-deployment)
        - [Monolithic deployment](#monolithic-deployment)
    - [Overlays](#overlays)

<!-- markdown-toc end -->



[Mimir](https://grafana.com/oss/mimir/) is an Open Source, horizontally scalable, highly available, multi-tenant TSDB for long-term storage for Prometheus.

Mimir is a single binary that can be configured to run in several modes to take up different roles in the ingestion/storage/read pipeline, such as `distributor`, `compactor` and so on, [read the official docs for a more detailed explanation.](https://grafana.com/docs/mimir/latest/get-started/about-grafana-mimir-architecture/#grafana-mimir-components).

Charmed Mimir consists of two charms, a [coordinator](https://github.com/canonical/mimir-coordinator-k8s-operator) and a [worker](https://github.com/canonical/mimir-worker-k8s-operator).

The coordinator is responsible for creating the configuration files for each worker and deciding whether the cluster is in a coherent state, based on the roles that each worker has adopted.

Several worker applications can be deployed and related to the coordinator, taking on different roles and enabling independent scalability of the various components of the stack.

The coordinator charm acts as single access point for bundle-level integrations such as TLS, ingress, self-monitoring, etc..., and as single source of truth for bundle-level configurations that otherwise would have to be repeated (and kept in sync) between the individual worker nodes.


This Juju bundle deploys Mimir and a small object storage server, consisting of the following interrelated charmed operators:

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
```

The bundle deploys Mimir in its [monolithic mode](https://grafana.com/docs/mimir/latest/references/architecture/deployment-modes/#monolithic-mode), which runs all required components in a single process and is the default mode of operation. Monolithic mode is the simplest way to deploy Grafana Mimir and is useful if you want to get started quickly or want to work with Grafana Mimir in a development environment.

```shell
tox -e render-bundle -- bundle.yaml
juju deploy ./bundle.yaml --trust
```

The bundle will deploy:

- 3 `worker` units
- 1 `coordinator` unit
- 1 `s3-integrator` unit

[note]
The default number of units is an odd number to prevent split-brain situations, and greater than 1 for the deployment to be HA.
[/note]

### Advanced deployment modes

Mimir can also be deployed distributing the roles across units freely. This includes for example the experimental [read/write deployment mode](https://grafana.com/docs/mimir/latest/references/architecture/deployment-modes/#read-write-mode).
You can achieve this (or any other allocation of roles) by deploying the components manually: 

```
juju deploy mimir-coordinator-k8s mimir
juju deploy mimir-worker-k8s read --config query-frontend=true --config querier=true
juju deploy mimir-worker-k8s write --config distributor=true --config ingester=true
juju deploy mimir-worker-k8s backend --config store-gateway=true --config compactor=true --config ruler=true --config alertmanager=true --config query-scheduler=true --config overrides-exporter=true
juju deploy s3-integrator

juju relate mimir read
juju relate mimir write
juju relate mimir backend
juju relate mimir s3-integrator
```

## Overlays

We also make available some [**overlays**](https://juju.is/docs/sdk/bundle-reference) as convenience.
A Juju overlay is a set of model-specific modifications, which reduce the amount of commands needed to set up a bundle like COS Lite.
Specifically, we offer the following overlays:

* the [`cos-relations` overlay](https://raw.githubusercontent.com/canonical/cos-lite-bundle/main/overlays/cos-relations-overlay.yaml) establishes relationships between the charms in this bundle and those in the [`cos-lite` bundle](https://github.com/canonical/cos-lite-bundle), as well as exposing Mimir as a Prometheus compatible remote-write targets as an Juju offer which may be consumed as part of a cross-model relation.
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
