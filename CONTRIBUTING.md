# Overview

This documents explains the processes and practices recommended for
contributing enhancements to the Mimir bundle.

- Generally, before developing enhancements to this charm, you should consider
  [opening an issue](https://github.com/canonical/mimir-bundle) explaining
  your use case.
- If you would like to chat with us about your use cases or proposed
  implementation, you can reach us on the
  [Charmhub Mattermost](https://chat.charmhub.io/charmhub/channels/charm-dev)
  or [Discourse](https://discourse.charmhub.io/).
- All enhancements require review before being merged.
  Apart from code quality and test coverage, the review will also take into
  account the Juju administrator user experience using the bundle.

## Development

There are two charms used for development:
- mimir
- s3proxy

### Deploy with local charms

To deploy the bundle using only/some local charms you need to render the
[`bundle.yaml.j2`](bundle.yaml.j2) template and then deploy the rendered bundle
as usual.

#### Render template using the rendering script
You can render and deploy a production bundle using:

```shell
# generate and activate a virtual environment with dependencies
tox -e integration --notest
source .tox/integration/bin/activate

./render_bundle.py bundle.yaml
juju deploy ./bundle.yaml
```

Now `juju switch` into the newly created model (or use `--model=MODEL` in
addition to `--keep-models` to use an existing model).

For more details, see the Testing section.

## Testing
Integration tests render the
[`bundle-testing.yaml.j2`](tests/integration/bundle-testing.yaml.j2) template
with the charm names/paths to use for the integration tests.
By default, all charms are deployed from charmhub. Alternatively, you can pass
local paths (or alternative charm names) as
[command line arguments](tests/integration/conftest.py) to pytest.

For a pure bundle test, no arguments should be provided. This way, the bundle
yaml is rendered with default values (all charms deployed from charmhub).

```shell
tox -e integration
```

You can also specify the channel:

```shell
tox -e integration -- --channel=edge
```

To keep the model and applications running after the tests completes:

```shell
tox -e integration -- --keep-models
```
