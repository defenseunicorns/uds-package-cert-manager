# [Cert Manager](https://cert-manager.io/)

"cert-manager is a powerful and extensible X.509 certificate controller for Kubernetes and OpenShift workloads. It will obtain certificates from a variety of Issuers, both popular public Issuers as well as private Issuers, and ensure the certificates are valid and up-to-date, and will attempt to renew certificates at a configured time before expiry." -- cert-manager.io

### Installed Resources

The Cert-Manager package installs the controller, webhook, and cainjector resources, as a helm release, using Ironbank images.

## Components

- `namespace-istio-injection` -- creates cert-manager namespace with the `istio-injection: enabled` label.
- `set-cert-manager-values` -- loads user given values to merge with (or override) default values.
- `deploy-chart` - installs helm chart
- `deploy-custom-manifests` (optional) -- applies your custom resource manifests (i.e. Issuers / Certificates)

## Controlling Values

You can set cert-manager values via `deploy-cert-manager-values.yaml`. This file will get passed to the `CERT_MANAGER_VALUES` variable, which will populate a values file given to the `deploy-chart` component. This allows you to add to or override the default values found in [cert-manager-values.yaml](./values/cert-manager-values.yaml). It's important to keep `installCRDs: true`, unless you want to manually install them yourself via `kubectl`. _If you do want to install CRDs manually after package deploy, you will not want to deploy the optional `deploy-custom-manifests` component, because it will fail without the CRDs._

You can find a list of configurable values at [artifacthub.io](https://artifacthub.io/packages/helm/cert-manager/cert-manager).

**_Image values cannot be overridden. This is because a specific set of images ([see here](./zarf.yaml#L72)) are brought into the package at create time._**

## Order of Operations

We highly recommend you deploy cert-manager **AFTER** DUBBD. If you want to deploy cert-manager before DUBBD, you'll need to deploy the prometheus service monitor CRD.

## Deploy Custom Issuers and Certificates

The optional `deploy-custom-manifests` component will apply your Issuer/ClusterIssuer/Certificates manifests. This component looks for a file passed to `###ZARF_VAR_CERT_MANAGER_MANIFESTS###`. The default file it looks for in the **current directory** is `deploy-custom-cert-manager-manifests.yaml`. You can change this by passing a different name to `cert_manager_manifests` in your `zarf-config.yaml` or by using `--set CERT_MANAGER_MANIFESTS=<path to your file>` when running `zarf package deploy`. Also if you're using `uds-cli` to create a bundle with cert-manager (see [examples/uds-bundle.yaml](/examples/uds-bundle.yaml)) you can create a `uds-config.yaml` to set the `cert_manager_manifests` variable.

_You can of course deploy your resource manually after the fact if you want. The benefit of using this optional component is zarf will then manage the clean up for you if the cert-manager package is removed._

## Securing Istio Gateways

### Before DUBBD

If you want to initiate Istio gateways with cert-manager created certificate secrets, vs patching Istio afterward, either manually or via `deploy-custom-manifests` create certificates. **One certificate secret needs to be named `admin-cert` (i.e. `secretName: admin-cert`) and the other `tenant-cert` (i.e. `secretName: tenant-cert`). This is a hack right now for getting around the fact that Big Bang does not directly expose `gateways.tls.credentialName`.** (i.e. [custom resources](examples/deploy-custom-manifests.yaml#L42))

### After DUBBD (Recommended)

Though it is a little more work to patch Istio gateways with the cert-manager created certficate secrets, this approach is recommended and will avoid certain weird dependency issues between cert-manager and DUBBD.

**_If you want to use Pepr. [here's an example](https://github.com/TristanHoladay/cert-manager-pepr)_**
