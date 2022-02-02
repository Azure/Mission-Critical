# AlwaysOn - Frequently Asked Questions (FAQ)

## Design methodology

> I need to design a highly reliable and mission-critical application on Azure. Where can I learn more about AlwaysOn design methodology so?

The AlwaysOn design principles and design area are published in this repository and you can learn more [here](/docs/design-methodology/README.md). In near future this content will be part of the *Azure Architecture Center* for general consumption.

## Reference implementation

> Can the reference implementation be used in any Azure environment without any restrictions?

The reference implementations is published under the [MIT open source license](/LICENSE) and can be used *as it*. The AlwaysOn engineering team is constantly improving the code and [accept contribution](/CONTRIBUTE.md). All reference implementation have been tested in *Azure Public* cloud infrastructure.

### Deployment

> How is the infrastructure getting deployed?

The infrastructure is deployed using Terraform. Other toolkits like ARM Templates and Bicep were planned but not implemented, yet.

### Patch & Update

> How is the infrastructure getting updated?

Most infrastructure components used for AlwaysOn are PaaS services and are maintained by Microsoft. Some service like for example Azure Kubernetes Service (AKS) require dedicated maintenance. For AKS this is done via [automatic node image upgrades](https://docs.microsoft.com/azure/aks/upgrade-cluster#set-auto-upgrade-channel) in combination with [planned maintenance windows](https://docs.microsoft.com/azure/aks/planned-maintenance) to automatically update the nodes to the most recent AKS node OS image. Larger changes like an upgrade of the used K8s version are done as-code by changing the K8s version in in the reference implementations file `.ado/pipeline/config/configuration.yaml`  and re-running the infrastructure pipeline.

### Security

> What is used to store secrets in AlwaysOn?

Wherever possible, Azure Managed Identities are used to avoid exposing any sensitive values like Service Principal client secrets (password). All secrets are stored in Azure Key Vault at deployment time via Terraform. These secrets are then loaded into Azure Kubernetes Service as Kubernetes secrets (and where required as environment variables in the pods) or handed over at deployment time as parameters for helm charts etc. Some temporary secrets like for example SSL/TLS certificates managed by *cert-manager* are stored within the Kubernetes cluster only.

---
[AlwaysOn - Full List of Documentation](/README.md)
