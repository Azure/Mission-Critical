# Frequently Asked Questions (FAQ)

## Design Methodology

> I need to design a highly reliable and mission-critical application on Azure. Where can I learn more about Azure Mission-Critical design methodology so?

The Azure Mission-Critical design principles and design areas are published within this repository and you can learn more about how to use the design methodology [here](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-design-methodology).
The Azure Mission-Critical design methodology is also available within the [Microsoft Azure Well-Architected Framework](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-design-methodology) for general consumption.

## Reference Implementations

> Can the reference implementations be used in any Azure environment without any restrictions?

The reference implementations is published under the [MIT open source license](/LICENSE) and can be used *as is*.
The Azure Mission-Critical engineering team is constantly improving the code and actively encourage community [contributions](/CONTRIBUTE.md). All reference implementations have been rigorously tested on *Azure Public* cloud infrastructure.

### Deployment

> How is the infrastructure getting deployed?

The application infrastructure is deployed using Terraform. Other approaches such as ARM Templates and Bicep can also be used, but are not yet implemented within the repositories.

### Patching & Updates

> How is the infrastructure getting updated?

Most infrastructure components used for Azure Mission-Critical are PaaS services and are maintained by Microsoft.
Some services, such as Azure Kubernetes Service (AKS) require dedicated maintenance activities, and for AKS this is achieved via [automatic node image upgrades](https://docs.microsoft.com/azure/aks/upgrade-cluster#set-auto-upgrade-channel) in combination with [planned maintenance windows](https://docs.microsoft.com/azure/aks/planned-maintenance) to automatically update the nodes to the most recent AKS node OS image. Larger changes, such as an upgrade of the K8s version are performed as-code by changing the K8s version within the reference implementation file `.ado/pipeline/config/configuration.yaml` and re-running the infrastructure pipeline.

### Security

> What is used to store secrets?

Wherever possible, Azure Managed Identities are used to avoid exposing any sensitive values like Service Principal client secrets (password).
All secrets are stored in Azure Key Vault at deployment time via Terraform. These secrets are then loaded into Azure Kubernetes Service as Kubernetes secrets (and where required as environment variables in the pods) or handed over at deployment time as parameters for helm charts etc. Some temporary secrets, such as SSL/TLS certificates managed by *cert-manager*, are stored within the Kubernetes cluster only.

---
[Azure Mission-Critical | Documentation Inventory](/docs/README.md)
