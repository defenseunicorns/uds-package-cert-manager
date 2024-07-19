# [Cert Manager](https://cert-manager.io/)

"cert-manager is a powerful and extensible X.509 certificate controller for Kubernetes and OpenShift workloads. It will obtain certificates from a variety of Issuers, both popular public Issuers as well as private Issuers, and ensure the certificates are valid and up-to-date, and will attempt to renew certificates at a configured time before expiry." -- cert-manager.io

> [!WARNING]  
> Support for DUBBD has been deprecated as of release `0.2.0`. All functionality for overriding helm values or deploying custom manifests has been preserved.

### Installed Resources

The Cert-Manager package installs the controller, webhook, and cainjector resources, as a helm release, using Ironbank or upstream images based on desired package flavor.

## Components

- `deploy-chart` - installs helm chart
- `deploy-custom-manifests` (optional) -- applies your custom resource manifests (i.e. Issuers / Certificates)

## Flavors

This package can be built as either an `upstream` or `registry1` flavor. These flavors instruct Zarf as to which images and values files to use.

## Variables

| Variable         | Description                                         |
| ---------------- | --------------------------------------------------- |
| custom_manifests | Control for deploying custom Cert-Manager resources |

## Quick Start

Prerequisites:

1. k3d installed locally
1. [uds binary installed](https://uds.defenseunicorns.com/cli/quickstart-and-usage/#install)

### Do It All

- Setup Cluster, Create Package, and Deploy Package -- `uds run` or `uds run default`

### Do Each Step

- Create a package flavor -- `usd run create-pkg --set flavor=<flavor>`

- Deploy package

  - without custom manifests -- `uds run deploy-pkg`

  - with custom manifests -- `uds run deploy-pkg --set deploy_options="--components=deploy-custom-manifests --set custom_manifests=<manifests>"`

## Available UDS Tasks

From `uds run --list-all`
| TASK | Description |
|------ | ----------- |
| default | default task: setup cluster, create package (default flavor=upstream), deploy package with custom manifiests example |
| create-pkg | create:package entrypoint |
| deploy-pkg | deploy:package entrypoint |
| common-setup:k3d-test-cluster | |
| common-setup:k3d-full-cluster | |
| common-setup:print-keycloak-admin-password | |
| common-setup:create-doug-user | |
| create:package | Create pkg flavor of Cert-Manager |
| deploy:package | Deploy pkg flavor of Cert-Manager |
| remove:package | Remove pkg flavor of Cert-Manager |
| publish:package | publish cert-manager package |

## Controlling Values

Previously, as of release `0.1.3`, you could override cert-manager helm values by using the `deploy-cert-manager-values.yaml`, which Zarf would pick up and apply. As of `0.2.0`, since uds-package-cert-manager is now configured explicitly for use in the UDS ecosystem as part of [bundles](https://uds.defenseunicorns.com/bundles/), helm values can be overridden by declaring [bundle overrides](https://uds.defenseunicorns.com/cli/overrides/).

You can find a list of configurable values at [artifacthub.io](https://artifacthub.io/packages/helm/cert-manager/cert-manager).

## Deploy Custom Issuers and Certificates

The optional `deploy-custom-manifests` component is still part of this package and will apply your Issuer/ClusterIssuer/Certificates manifests. This component looks for a file passed to `###ZARF_VAR_CUSTOM_MANIFESTS###`. You can do this by setting `custom_manifests` via a `uds-config.yaml`, or by environment variable (`UDS_CUSTOM_MANIFESTS=<custom-manifest-file>`), or by cli at deploy time `--set CUSTOM_MANIFESTS=<path to your file>`

_You can of course deploy your resources manually after the fact if you want. The benefit of using this optional component is zarf will then manage the clean up for you if the cert-manager package is removed._

## Securing Istio Gateways

Because cert-manager is meant to be deployed after uds-core, if you want to secure Istio gateways with cert-manager certificate secrets you will need to patch the gateways.

**_If you want to use Pepr. [here's an example](./examples/pepr-capability-example.txt)_**
