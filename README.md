# [Cert Manager](https://cert-manager.io/)

"cert-manager is a powerful and extensible X.509 certificate controller for Kubernetes and OpenShift workloads. It will obtain certificates from a variety of Issuers, both popular public Issuers as well as private Issuers, and ensure the certificates are valid and up-to-date, and will attempt to renew certificates at a configured time before expiry." -- cert-manager.io

> [!NOTE]  
> Version `0.1.3` introduces the `uds-config` chart for deploying cert-manager with `UDS Core`. The chart will look for the existence of the `uds.dev/v1alpha1` in your cluster, and if found it will then deploy the cert-manager UDS Package custom reesource.

> [!WARNING]  
> Support for DUBBD is deprecated and will be removed in release `0.2.0` (next planned release). All the functionality for overriding helm values or deploying custom manifests will be preserved.

### Installed Resources

The Cert-Manager package installs the controller, webhook, and cainjector resources, as a helm release, using Ironbank images.

## Components

- `istio-configuration` -- if `configure_for_istio` is `true`, then creates cert-manager namespace with the `istio-injection: enabled` label and creates an Istio PeerAuthentication custom resource to allow `PERMISSIVE` traffic to the `cert-manager-webhook` on port `10250`
- `set-values` -- loads user given values to merge with (or override) default values
- `deploy-chart` - installs helm chart
- `deploy-custom-manifests` (optional) -- applies your custom resource manifests (i.e. Issuers / Certificates)

## Flavors

This package can be built as either an `upstream` or `registry1` flavor. These flavors instruct Zarf as to which images and values files to use.

## Variables

| Variable            | Description                                                           |
| ------------------- | --------------------------------------------------------------------- |
| custom_values       | Control for adding or overriding values for either package flavor.    |
| custom_manifests    | Control for deploying custom Cert-Manager resources                   |
| configure_for_istio | Control for enabling Cert-Manager to deploy within Istio service mesh |

## Quick Start

From within the repo:

- Create a package flavor -- `zarf package create -f <flavor> --confirm`

- Deploy package

  - all defaults - `zarf package deploy zarf-package-*.zst`

  - with custom values - `zarf package deploy zarf-package-*.zst --set custom_values=<values-file>`

  - with custom manifests - `zarf package deploy zarf-package-*.zst --components=deploy-custom-manifests --set custom_manifests=<manifests>`

  - for use in cluster with istio - `zarf package deploy zarf-package-*.zst --set configure_for_istio=true`

## Controlling Values

You can set cert-manager values via `deploy-cert-manager-values.yaml`. This file will get passed to the `CUSTOM_VALUES` variable, which will populate a values file given to the `deploy-chart` component. This allows you to add to or override the default values found in [cert-manager-values.yaml](./values/cert-manager-values.yaml). It's important to keep `installCRDs: true`, unless you want to manually install them yourself via `kubectl`. _If you do want to install CRDs manually after package deploy, you will not want to deploy the optional `deploy-custom-manifests` component, because it will fail without the CRDs._

You can find a list of configurable values at [artifacthub.io](https://artifacthub.io/packages/helm/cert-manager/cert-manager).

> Warning:
> **_Image values cannot be overridden. This is because a specific set of images ([see here](./zarf.yaml#L78)) are brought into the package at create time._**

## Order of Operations

Due to dependency issues with deploying cert-manager before DUBBD, you'll need to deploy cert-manager **AFTER** DUBBD. [see here](./examples/uds-bundle.yaml)

If you're not using Cert-Manager with DUBBD, you will need to use the values override mechanism (see Controlling Values above) to disable service monitor.

```yaml
prometheus:
  servicemonitor:
    enabled: false
```

## Deploy Custom Issuers and Certificates

The optional `deploy-custom-manifests` component will apply your Issuer/ClusterIssuer/Certificates manifests. This component looks for a file passed to `###ZARF_VAR_CUSTOM_MANIFESTS###`. You can do this by setting `custom_manifests` via zarf-config.yaml, or `--set CUSTOM_MANIFESTS=<path to your file>` on package deploy, or uds-config.yaml if using uds-cli to bundle cert-manager.

_You can of course deploy your resources manually after the fact if you want. The benefit of using this optional component is zarf will then manage the clean up for you if the cert-manager package is removed._

## Securing Istio Gateways

Because cert-manager is deployed after DUBBD, if you want to secure Istio gateways with cert-manager certificate secrets you will need to patch the gateways.

**_If you want to use Pepr. [here's an example](https://github.com/TristanHoladay/cert-manager-pepr)_**
