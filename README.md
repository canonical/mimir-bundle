# Mimir bundle

Mimir is a shardable, horizontally scalable push-based metrics server, and alert rule evaluation engine which is a Prometheus/Cortex/Thanos-compatible by Grafana Labs.

This Juju bundle deploys Mimir and a small object storage server, consisting of the following interrelated charms:

- [Mimir](https://charmhub.io/mimir-k8s) ([source](https://github.com/canonical/mimir-k8s-operator))
- [s3proxy](https://charmhub.io/s3proxy-k8s) ([source](https://github.com/canonical/s3proxy-k8s-operator))

This bundle is under development.
Join us on [Discourse](https://discourse.charmhub.io/t/canonical-observability-stack/5132) and [MatterMost](https://chat.charmhub.io/charmhub/channels/observability)!

## Usage

Before deploying the bundle you may want to create a dedicated model for COS components, of which Mimir is one, if one does not already exist:

```shell
juju add-model cos
juju switch cos
```

You can deploy the bundle from charmhub with:

```shell
juju deploy mimir-bundle --channel=edge --trust
```

or, to deploy the bundle from a local file:

```shell
# generate and activate a virtual environment with dependencies
tox -e integration --notest
source .tox/integration/bin/activate

# render bundle with default values
./render_bundle.py bundle.yaml
juju deploy ./bundle.yaml --trust
```

Currently the bundle is available only on the `edge` channel, using `edge` charms.
When the charms graduate to `beta`, `candidate` and `stable`, we will issue the bundle in the same channels.

The `--trust` option is needed by the charms in the `mimir-lite` bundle to be able to patch their K8s services to use the right ports (see this [Juju limitation](https://bugs.launchpad.net/juju/+bug/1936260)).

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
