# Security

Security is paramount for any mission-critical application, particularly given the myriad of threat actors that exist in present-day society. Security must therefore be treated as a first-class concern within the AlwaysOn architectural process, to ensure that security services and practices are interwoven within the solution.

Security is ultimately an extremely broad technical domain, encompassing a variety of threat vectors that collectively stretch across the entire application stack. However, given the primary aspiration of AlwaysOn is to maximize reliability for application scenarios which must remain performant and available, the security lens applied within this design area will focus on mitigating threats with the capacity to impact availability and hinder overall reliability. For example, how an application mitigates attack vectors such as DDoS and Slowloris will have a critical bearing on overall reliability, since successful DDoS attacks will have a catastrophic impact on availability and performance. Hence, an application must be fully protected against threats intended to directly or indirectly compromise application reliability to be truly 'always on'.

It is also important to note that there are often significant trade-offs associated with a hardened security posture, particularly with respect to performance, operational agility, and in some cases reliability. For example, the inclusion of inline Network Virtual Appliances (NVA) for Next-Generation Firewall (NGFW) capabilities, such as deep packet inspection, will introduce a significant performance penalty, additional operational complexity, and a reliability risk if scalability and recovery operations are not closely aligned with that of the application. It is therefore essential that additional security components and practices intended to mitigate key threat vectors are also designed to support the reliability target of an AlwaysOn application, which will form a key aspect of the recommendations and considerations presented within this section.

- [Zero Trust](#zero-trust)
- [Threat Modeling](#threat-modeling)
- [Network Intrusion Protection](#network-intrusion-protection)
- [Data Integrity Protection](#data-integrity-protection)
- [Policy Driven Governance](#policy-driven-governance)

## Zero Trust

The [Zero Trust](https://www.microsoft.com/security/business/zero-trust) security model provides a proactive and integrated approach to applying security across all layers of an application estate, to explicitly and continuously verify every transaction, assert least privilege, leverage intelligence and advanced detection to respond to threats in near real-time. It is ultimately centered on eliminating trust inside and outside of application perimeters, enforcing verification for anything attempting to connect to the system.

- **Verify explicitly**:  Always authenticate and authorize based on all available data points, including user identity, location, device health, service or workload, data classification, and anomalies.

- **Use least privileged access**: Limit user access with just-in-time and just-enough-access (JIT/JEA), risk-based adaptive polices, and data protection to help secure both data and productivity.

- **Assume breach**: Minimize blast radius and segment access. Verify end-to-end encryption and use analytics to get visibility, drive threat detection, and improve defenses.

> The AlwaysOn design methodology and foundational reference implementation adopt a Zero Trust model to structure and guide the security design and implementation approach.

### Design considerations

- Continuous security testing to validate mitigations for key security vulnerabilities.
  - *Is security testing performed as a part of automated CI/CD processes?*
  - *If not, how often is specific security testing performed?*
  - *Are test outcomes measured against a desired security posture and threat model?*

- Security level across all lower-environments.
  - *Do all environments within the development lifecycle have the same security posture as the production environment?*

- Authentication and Authorization continuity in the event of a failure.
  - *If authentication or authorization services are temporarily unavailable, will the application be able to continue to operate?*

- Azure provides [Azure AD](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-whatis) and [Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/overview) services for application user authorization.

- Automated security compliance and remediation.
  - *Can changes to key security settings be detected*?
  - *Are responses to remediate non-compliant changes automated?*

- Secret management and the risk associated with leakage.
  - Secret scanning to detect secrets before code is committed to prevent any secret leaks through source code repositories.
  - *Is authentication to services possible without having credentials as a part of code?*

- Securing the software supply chain
  - *Is it possible to track Common Vulnerabilities and Exposures (CVEs) within utilized package dependencies?*
  - *Is there an automated process for updating package dependencies?*

- Data protection key lifecycles
  - *Can service-managed keys be used for data integrity protection?*
  - If customer-managed keys are required, secure and reliable key lifecycle must be managed, opening up to a variety of additional risks.

- CI/CD tooling will require Azure AD service principals with sufficient subscription level access to facilitate control plane access for Azure resource deployments to all considered environment subscriptions.
  - When application resources are locked down within private networks, a private data-plane connectivity path is required so that CI/CD tooling can perform application level deployments and maintenance.
    - This introduces additional complexity and requires a sequence within the deployment process through requisite private build agents.

### Design recommendations

- Use Azure Policy to enforce security and reliability configurations for all service, ensuring that any deviation is either remediated or prohibited by the control plane at configuration-time, helping to mitigate threats associated with 'malicious admin' scenarios.

- Use Azure AD Privileged Identity Management (PIM) within production subscriptions to revoke sustained control plane access to AlwaysOn production environments, significantly reducing the risk posed from 'malicious admin' scenarios through additional 'checks and balances'.

- Use [Azure Managed Identities](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview) for all services that support the capability, since it facilitates the removal of credentials from application code and removes the operational burden of identity management for service to service communication.

- Use Azure AD Role Based Access Control (RBAC) for data plane authorization with all services that support the capability.

- Use first-party [Microsoft identity platform authentication libraries](https://docs.microsoft.com/azure/active-directory/develop/reference-v2-libraries) within application code to integrate with Azure AD.

- Consider secure token caching since this will allow for a degraded but available experience in the event that the Azure AD, or the chosen identity platform, is not available or is only partially available for application authorization.
  - Depending on what services provided by the identity service don't work.
  - The identity provider can be down completely, therefore no authentication and authorization can be done. That's quite rare though.
  - If the provider is unable to issue new access tokens, but still validates existing ones, the application and dependent services can operate without issues until their tokens expire.
  - Token caching is typically handled automatically by authentication libraries ([such as MSAL](https://docs.microsoft.com/azure/active-directory/fundamentals/resilience-client-app?tabs=csharp)).

- Use the principle of IaC and automated CI/CD pipelines to drive updates to all application components, including under failure circumstances.
  - Ensure CI/CD tooling service connections are safeguarded as critical sensitive information, and should not be directly available to any service team.
  - Apply granular RBAC to production CD pipelines to mitigate 'malicious admin' risks.
  - Consider the use of manual approval gates within production deployment pipelines to further mitigate 'malicious admin' risks and provide additional technical assurance for all production changes.
    - Additional security gates may come at a trade-off in terms of agility and should be carefully evaluated, with consideration given to how agility can be maintained even with manual gates.

- Define an appropriate security posture for all lower environments to ensure key vulnerabilities are mitigated.
  - Do not apply the same security posture as production, particularly with regards to data exfiltration, unless regulatory requirements stipulate the need to do so, since this will significantly compromise developer agility.

- Enable Microsoft Defender for Cloud (formerly known as Azure Security Center) for all AlwaysOn subscriptions.
  - Use Azure Policy to enable Azure Censure compliance.
  - Enable Azure Defender for all services that support the capability in the AlwaysOn subscriptions.

- Embrace [DevSecOps](https://docs.microsoft.com/azure/devops/devsecops/) and implement security testing within CI/CD pipelines.
  - Test results should be measured against a compliant security posture to inform release approvals, be they automated or manual.
  - Apply security testing as part of the CD production process for each release.
    - If security testing each release jeopardizes operational agility, ensure a suitable security testing cadence is applied.
  
- Limit public network access to the absolute minimum required for the application to fulfil its business purpose to reduce the external attack surface.
  - Use [Azure Private Link](https://docs.microsoft.com/azure/private-link/private-endpoint-overview#private-link-resource) to establish [private endpoints](https://docs.microsoft.com/azure/private-link/private-endpoint-overview) for Azure resources which require secure network integration.
  - Use a secure network path and [hosted private build agents](https://docs.microsoft.com/azure/devops/pipelines/agents/agents#install) for CI/CD tooling to deploy and configure Azure resources protected by Azure Private Link.
    - [Microsoft-hosted agents](https://docs.microsoft.com/azure/devops/pipelines/agents/agents#microsoft-hosted-agents) will not be able to directly connect to network integrated resources.

- Enable [secret scanning](https://github.blog/2020-08-27-secure-at-every-step-putting-devsecops-into-practice-with-code-scanning/) and dependency scanning within the source code repository.

## Threat modeling

Threat modeling provides a risk based approach to security design, using identified potential threats to develop appropriate security mitigations. There is ultimately a myriad of possible threats with varying probabilities of occurrence, and in many cases threats can chain in unexpected, unpredictable, and even chaotic ways. This complexity and uncertainty is precisely why traditional technology requirement based security approaches are largely unsuitable for mission-critical cloud applications, and unfortunately means that the process of threat modelling for an AlwaysOn application is complex and unyielding.

To help navigate these challenges, a layered defense-in-depth approach should be applied to define and implement compensating mitigations for modeled threats, considering the following defensive layers.

1. The Azure platform with foundational security capabilities and controls.
1. The AlwaysOn application architecture and security design.
1. Security features (built-in, enabled, and deployable) applied to secure Azure resources.
1. Application code and security logic.
1. Operational processes and DevSecOps.

> In an Enterprise-Scale context, the foundational platform provides an additional threat mitigation layer through the provision of centralized security capabilities within the Enterprise-Scale architecture.

### Design considerations

- [STRIDE](https://en.wikipedia.org/wiki/STRIDE_(security)) provides a lightweight risk framework for evaluating security threats across key threat vectors.
  - Spoofed Identity: Impersonation of individuals with authority. For example, an attacker impersonating another user by leveraging their -
    - Identity
    - Authentication
  - Tampering Input: Modification of input sent to the application, or the breach of trust boundaries to modify application code. For example, an attacker using SQL Injection to delete data in a database table.
    - Data integrity
    - Validation
    - Blacklisting/allowlisting
  - Repudiation of Action: Ability to refute actions already taken, and the ability of the application to gather evidence and drive accountability. For example, the deletion of critical data without the ability to trace to a malicious admin.
    - Audit/logging
    - Signing
  - Information Disclosure: Gaining access to restricted information. An example would be an attacker gaining access to a restricted file.
    - Encryption
    - Data exfiltration
    - Man-in-the-middle attacks
  - Denial of Service: Malicious application disruption to degrade user experience. For example, a DDoS botnet attack such as Slowloris.
    - DDoS
    - Botnets
    - CDN and WAF capabilities
  - Elevation of Privilege: Gaining privileged application access through authorization exploits. For example, an attacker manipulating a URL string to gain access to sensitive information.
    - Remote code execution
    - Authorization
    - Isolation

### Design recommendations

- Allocate engineering budget within each sprint to evaluate potential new threats and implement mitigations.

- Conscious effort should be applied to ensure security mitigations are captured within a common engineering criteria to drive consistency across all application service teams.
  
- Start with a service by service level threat modeling and unify the model by consolidating the thread model on application level.

## Network intrusion protection

Preventing unauthorized access to an AlwaysOn application and encompassed data is vital to maintain availability and safeguard data integrity. This section will therefore explore the platform capabilities required to secure network access to an AlwaysOn application.

### Design considerations

- The zero trust model assumes a breached state and verifies each request as though it originates from an uncontrolled network.
  - An advanced zero-trust network implementation employs micro-segmentation and distributed ingress/egress micro-perimeters.

- Azure PaaS services such as AKS or Cosmos DB are typically accessed over public endpoints. However, the Azure platform provides capabilities to secure public endpoints or even make them entirely private.
  - Azure Private Link/Private Endpoints provides dedicated access to an Azure PaaS resource using private IP addresses and private network connectivity.
  - Virtual Network Service Endpoints provide service-level access from selected subnets to selected PaaS services.
  - Virtual Network Injection provides dedicated private deployments for supported services, such as App Service through an App Service Environment.
    - Management plane traffic still flows through public IP addresses.
  
- For supported services, Azure Private Link using Azure Private Endpoints addresses [data exfiltration risks associated with Service Endpoints](https://docs.microsoft.com/azure/private-link/private-link-faq#what-is-the-difference-between-service-endpoints-and-private-endpoints-), such as a malicious admin writing data to an external resource.

- When restricting network access to Azure PaaS services using Private Endpoints or Service Endpoints, a secure network channel will be required for deployment pipelines to access both the Azure control plane and data plane of Azure resources in order to deploy and manage the application.
  - [Private self-hosted build agents](https://docs.microsoft.com/azure/devops/pipelines/agents/agents?view=azure-devops&tabs=browser#install) deployed onto the a private network as the Azure resource can be used as a proxy to execute CI/CD functions over a private connection. A separate virtual network should be used for build agents.
    - Connectivity to the private build agents from CI/CD tooling is required.
  - An alternative approach is to modify the firewall rules for the resource on-the-fly within the pipeline to allow a connection from an Azure DevOps agent public IP address, with the firewall subsequently removed after the task is completed. 
    - However, this approach is only applicable for a subset of Azure services. For example, this is not feasible for private AKS clusters.
  - To perform developer and administrative tasks on the application service jump servers can be used.
  
- The completion of administration and maintenance tasks is a further scenario requiring connectivity to the data plane of Azure resources.

- Service Connections with a corresponding Azure AD service principals can be leveraged within Azure DevOps to apply RBAC through Azure AD.

- Service Tags can be applied to Network Security Groups to facilitate connectivity with Azure PaaS services.

- Application Security Groups do not span across multiple virtual networks.

- Packet capture in Azure Network Watcher is limited to a maximum period of five hours.

### Design recommendations

- To maximize network security, limit network access to the absolute minimum required for the application to fulfil its purpose.
  - Use internal network paths, such as Azure Bastion, or automatically managed temporary firewall rules to enable direct data-plane infrastructure access.

- When dealing with private build agents, never open an RDP or SSH port directly to the internet.
  - Use [Azure Bastion](https://docs.microsoft.com/azure/bastion/bastion-overview) to provide secure access to Azure Virtual Machines and to perform administrative tasks on Azure PaaS over the Internet.

- Use a DDoS standard protection plan to secure all public IP addresses within the application.

- Use Azure Front Door with WAF policies to deliver and help protect global HTTP/S AlwaysOn applications that span multiple Azure regions.
  - Use Header Id validation to lock down public application endpoints so they only accept traffic originating from the Azure Front Door instance.

- If additional in-line network security requirements, such as deep packet inspection or TLS inspection, mandate the use of Azure Firewall Premium or Network Virtual Appliance (NVA), ensure it is configured for maximum high availability and redundancy.

- If packet capture requirements exist, use Network Watcher packets to capture despite the limited capture window.

- Use Network Security Groups and Application Security Groups to micro-segment application traffic.
  - Avoid using a security appliance to filter intra-application traffic flows.
  - Consider the use of Azure Policy to enforce specific NSG rules are always associated with application subnets.

- Enable NSG flow logs and feed them into Traffic Analytics to gain insights into internal and external traffic flows.

- Use Azure Private Link/Private Endpoints, where available, to secure access to Azure PaaS services within the AlwaysOn application design, such as AKS, Cosmos DB, Azure Key Vault, Azure Container Registry, and Azure Storage.

- If Private Endpoint is not available and data exfiltration risks are acceptable, use Virtual Network Service Endpoints to secure access to Azure PaaS services from within a virtual network.
  - Don't enable virtual network service endpoints by default on all subnets as this will introduce significant data exfiltration channels.

- For hybrid application scenarios, access Azure PaaS services from on-premises via ExpressRoute with private peering.

> In an Enterprise-Scale context, the foundational platform will provide network connectivity to on-premises data centers using Express Route configured with private peering.

## Data integrity protection

Encryption is a vital step toward ensuring data integrity and is ultimately one of the most important security capabilities which can be applied to mitigate a wide array of threats. This section will therefore provide key considerations and recommendations related to encryption and key management in order to safeguard data without compromising application reliability.

### Design considerations

- Azure Key Vault has transaction limits for keys and secrets, with throttling applied per vault within a certain period.

- Azure Key Vault provides a security boundary since access permissions for keys, secrets, and certificates are applied at a vault level.
  - Key Vault access policy assignments grant permissions separately to keys, secrets, or certificates.
    - Granular [object-level permissions](https://docs.microsoft.com/azure/key-vault/general/rbac-guide?tabs=azure-cli#best-practices-for-individual-keys-secrets-and-certificates) to a specific key, secret, or certificate are now possible.

- Role assignments innur a latency, taking up to 10 minutes (600 seconds) for a role to be applied after a role assignment is changed.
  - There is a AAD limit of 2,000 Azure role assignments per subscription.

- Azure Key Vault underlying hardware security modules (HSMs) are FIPS 140-2 Level 2 compliant.
  - A dedicated [Azure Key Vault managed HSM](https://docs.microsoft.com/azure/key-vault/managed-hsm/overview) is available for scenarios requiring FIPS 140-2 Level 3 compliance.

- Azure Key Vault provides high availability and redundancy to help maintain availability and prevent data loss.

- In the event of a region failover, it may take a few minutes for the Key Vault service to fail over.
  - During a failover Key Vault will be in a read-only mode, so it will not be possible to change key vault properties such as firewall configurations and settings.

- If private link is used to connect to Azure Key Vault, it may take up to 20 minutes for the connection to be re-established in the event of a regional failover.

- A backup creates a [point-in-time snapshot](https://docs.microsoft.com/azure/key-vault/general/backup?tabs=azure-cli#overview) of a secret, key, or certificate, as an encrypted blob which cannot be decrypted outside of Azure. To get usable data from the blob, it must be restored into a Key Vault within the same Azure subscription and Azure geography.
  - Secrets may renew during a backup, causing a mismatch.

- With service-managed keys, Azure will perform key management functions, such as rotation, thereby reducing the scope of application operations.

- Regulatory controls may stipulate the use of customer-managed keys for service encryption functionality.

- When traffic moves between Azure data centers, MACsec data-link layer encryption is used on the underlying network hardware to secure data in-transit outside of the physical boundaries not controlled by Microsoft or on behalf of Microsoft.

### Design recommendations

- Use Azure Key Vault to store all application secrets and certificates.
  - Deploy a separate collocated Azure Key Vault with every regional deployment stamp.

- Use service-managed keys for data protection where possible, removing the need to manage encryption keys and handle operational tasks such as key rotation.
  - Only use customer-managed keys when there is a clear regulatory requirement to do so.

- Use [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview) as a secure repository for all secrets, certificates, and keys if additional encryption mechanisms or customer-managed keys need considered.
  - Provision Azure Key Vault with the soft delete and purge policies enabled to allow retention protection for deleted objects.
  - Use HSM backed Azure Key Vault SKU for application production environments.

- Deploy a separate Azure Key Vault instance within each regional deployment stamp, providing fault isolation and performance benefits through localization, as well as navigating the scale limits imposed by a single key vault instance.
  - Use a dedicated Azure Key Vault instance for AlwaysOn global resources.

- Follow a least privilege model by limiting authorization to permanently delete secrets, keys, and certificates to specialized custom Azure AD roles.

- Ensure secrets, keys, and certificates stored within key vault are backed up, providing an offline copy in the unlikely event key vault becomes unavailable.

- Use Key Vault certificates to [manage certificate procurement and signing](https://docs.microsoft.com/azure/key-vault/certificates/certificate-scenarios#creating-a-certificate-with-a-ca-partnered-with-key-vault).

- Establish an automated process for key and certificate rotation.
  - Automate the certificate management and renewal process with public certificate authorities to ease administration.
    - Set alerting and notifications, to supplement automated certificate renewals.

- Monitor key, certificate, and secret usage.
  - Define [alerts](https://docs.microsoft.com/azure/key-vault/general/alert) for unexpected usage within Azure Monitor.

- Enable firewall and Private Endpoints on all application Azure Key Vault to control access.

## Policy driven governance

Security conventions are ultimately only effective if consistently and holistically enforced across all application services and teams. Azure Policy provides a framework to enforce security and reliability baselines, ensuring continued compliance with a common engineering criteria for an AlwaysOn application. More specifically, Azure Policy forms a key part of the Azure Resource Manager (ARM) control plane, supplementing RBAC by restricting what actions authorized users can perform, and can be leveraged to enforce vital security and reliability conventions across utilized platform services.

This section will therefore explore key considerations and recommendations surrounding the use of Azure Policy driven governance for an AlwaysOn application, ensuring security and reliability conventions are continuously enforced.

### Design considerations

- Azure Policy provides a mechanism to drive compliance by enforcing security and reliability conventions, such as the use of Private Endpoints or the use of Availability Zones.

> In the context of an Enterprise Scale environment, the enforcement of centralized baseline policy assignments will likely be applied for Landing Zone management groups and subscriptions.

- Azure Policy can be used to drive automated management activities, such as provisioning and configuration.
  - Resource Provider registration.
  - Validation and approval of individual Azure resource configurations.

- Azure Policy assignment scope dictates coverage and the location of Azure Policy definitions informs the reusability of custom policies.

- Azure Policy has [several limits](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#azure-policy-limits), such as the number of definitions at any particular scope.

- It can take several minutes for the execution of Deploy If Not Exist (DINE) policies to occur.

- Azure Policy provides a critical input for compliance reporting and security auditing.

### Design recommendations

- Map regulatory and compliance requirements to Azure Policy definitions.
  - For example, if there are data residency requirements, a policy should be applied to restrict available deployment regions.

- Define a common engineering criteria to capture secure and reliable configuration definitions for all utilized Azure services, ensuring this criteria is mapped to Azure Policy assignments to enforce compliance.
  - For example, apply a Azure Policy to enforce the use of Availability Zones for all relevant services, ensuring reliable intra-region deployment configurations.

> The AlwaysOn Foundational Reference Implementation contains a wide array of [security and reliability centric policies](https://github.com/Azure/AlwaysOn/blob/main/docs/reference-implementation/Policy-Driven-Governance.md) to define and enforce a sample common engineering criteria.

- Monitor service configuration drift, relative to the common engineering criteria, using Azure Policy.

> For AlwaysOn scenarios with multiple production subscriptions under a dedicated management group, prioritize assignments at the management group scope.

- Use built-in policies where possible to minimize operational overhead of maintaining custom policy definitions.

- Where custom policy definitions are required, ensure definitions are deployed at suitable management group scope to allow for reuse across encompassed AlwaysOn environment subscriptions to this allow for policy reuse across production and lower environments.
  - When aligning the application roadmap with Azure roadmaps, leverage available Microsoft resources to explore if critical custom definitions could be incorporated as built-in definitions.

> When deployed within an Enterprise Scale context, consider deploying custom Azure Policy Definitions within the intermediate company root management group scope to enable reuse across all applications within the broader Azure estate.

> When deployed in an Enterprise Scale environment, certain centralized security policies will be applied by default within higher management group scopes to enforce security compliance across the entire Azure estate. For example, Azure policies should be applied to automatically deploy software configurations through VM extensions and enforce a compliant baseline VM configuration as part of the Enterprise-Scale foundation.

- Use Azure Policy to enforce a consistent tagging schema across the application.
  - Identify required Azure tags and leverage the append policy mode to enforce usage.

> If the AlwaysOn application is subscribed to Microsoft Mission-Critical Support, ensure that the applied tagging schema provides meaningful context to enrichen the support experience with deep application understanding.

- Export Azure AD activity logs to the AlwaysOn global Log Analytics Workspace.
  - Ensure Azure activity logs are archived within the global Storage Account along with operational data for long-term retention.

> In an Enterprise-Scale context, Azure AD activity logs will also be ingested into the centralized platform Log Analytics workspace. It needs to be evaluated in this case if Azure AD are still required in the AlwaysOn global Log Analytics workspace.

- Integrate security information and event management with Microsoft Defender for Cloud (formerly known as Azure Security Center).

---

|Previous Page|Next Page|
|:--|:--|
|[Networking and Connectivity](./Networking.md) | [Operational Procedures](./Operational-Procedures.md) |

---

|Design Methodology|
|--|
|[How to use the AlwaysOn Design Methodology](./README.md)
|[AlwaysOn Design Principles](./Principles.md)
|[AlwaysOn Design Areas](./Design-Areas.md)
|[Application Design](./App-Design.md)
|[Application Platform](./App-Platform.md)
|[Data Platform](./Data-Platform.md)
|[Health Modeling and Observability](./Health-Modeling.md)
|[Deployment and Testing](./Deployment-Testing.md)
|[Networking and Connectivity](./Networking.md)
|[Security](./Security.md)
|[Operational Procedures](./Operational-Procedures.md)

---

[AlwaysOn | Documentation Inventory](/docs/README.md)
