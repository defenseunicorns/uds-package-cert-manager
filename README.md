# [Cert Manager](https://cert-manager.io/)

"cert-manager is a powerful and extensible X.509 certificate controller for Kubernetes and OpenShift workloads. It will obtain certificates from a variety of Issuers, both popular public Issuers as well as private Issuers, and ensure the certificates are valid and up-to-date, and will attempt to renew certificates at a configured time before expiry." -- cert-manager.io

### Install

The Cert-Manager package installs the controller, webhook, and cainjector resources, as a helm release, using Ironbank images.

## Components

- namespace-istio-injection -- creates cert-manager namespace with the `istio-injection: enabled` label.
- deploy-smon-crd (optional) -- deploys the prometheus service monitor CRD if you want to deploy cert-manager before DUBBD.
- deploy-chart - installs helm chart
- deploy-custom-resources (optional) -- applies your custom resource manifests (i.e. Issuers / Certificates)

## Controlling Values

You can set cert-manager values via `values.yaml`. It's important to keep `installCRDs: true`, unless you want to manually install them yourself via `kubectl` after the fact.

You can find a list of configurable values at [artifcathub.io](https://artifacthub.io/packages/helm/cert-manager/cert-manager).

## Order of Operations

We recommend you deploy cert-manager after DUBBD. If you want to deploy cert-manager before DUBBD, you'll need to deploy the optional `deploy-smon-crd` component.

## Deploy Create Custom Issuers and Certificates

The optional `deploy-custom-resources` component will apply your Issuer/ClustIssuer/Certificates manifests. This component looks for a file passed to `###ZARF_VAR_CERT_RESOURCES###`. The default file it looks for in the **current directory** is `deploy-custom-resources.yaml`. You can change this by passing a different name to `cert_resources` in your `zarf-config.yaml` or by using `--set CERT_RESOURCES=<path to your file>` when running `zarf package deploy`.

_You can of course deploy your resource manually after the fact if you want. The benefit of using this optional component is zarf will then manage the clean up for you if the cert-manager package is removed._

## Securing Istio Gateways

### Before DUBBD

If you want to initiate Istio gateways with cert-manager created certificate secrets, vs patching Istio afterward, either manually or via `deploy-custom-resources` create certificates. **One certificate secret needs to be named `admin-cert` (i.e. `secretName: admin-cert`) and the other `tenant-cert` (i.e. `secretName: tenant-cert`). This is a hack right now for getting around the fact that Big Bang does not directly expose `gateways.tls.credentialName`.** (i.e. [custom resources](examples/deploy-custom-resources.yaml#L42))

### After DUBBD

If you are ok with DUBBD deploying first, you can create your cert-manager certificates afterward and patch the Istio gateways.

_If you want to use Pepr. [here's an example](https://github.com/TristanHoladay/cert-manager-pepr)_
