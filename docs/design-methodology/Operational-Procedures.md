# Operational Procedures

The AlwaysOn design methodology leans heavily on the principles *automation wherever possible* and *configuration as code* to drive reliable and effective operations through DevOps processes, with automated deployment pipelines used to execute versioned application and infrastructure code artifacts within a source repository. While this level of DevOps adoption requires substantial engineering investment to instantiate and discipline to maintain, it yields significant operational dividends, enabling consistent and accurate operational outcomes within minimal manual operational procedures.

- [DevOps Processes](#devops-processes)
- [Application Operations](#application-operations)

## DevOps Processes

DevOps is a fundamental characteristic of the AlwaysOn design methodology, providing the engineering mindset, processes, and tooling to deliver application services in a fast, efficient, and reliable manner. More specifically, DevOps brings together development and operational processes as well as teams into a single engineering function that encompasses the entire application lifecycle, leveraging automation and DevOps tooling to conduct deployment operations swiftly and reliably. This section will therefore explore how the adoption of DevOps and related deployment methods is used to drive effective and consistent operational procedures.

### Design Considerations

- DevOps processes support and sustain the concepts of continuous integration and continuous deployment (CI/CD), while fostering a culture of continuous improvement.

- DevOps can be difficult to apply when there are hard dependencies on central IT functions since it prevents end-to-end operational action.

- Key responsibilities of the DevOps team for an AlwaysOn application include:
  - Create and manage application and infrastructure resources through CI/CD automation.
  - Application monitoring and observability.
  - Azure RBAC and identity for application components.
  - Security monitoring and audit of application resources.
  - Network management for application components.
  - Cost management for application resources.

- DevSecOps expands the DevOps model by integrating security and quality assurance teams with development and operations throughout the application lifecycle.

- A DevOps engineering team can consider a variety of granular Azure RBAC roles for different technical personas, such as AppDataOps for database management.
  - A zero trust model can and should be applied across different application DevOps personas.

### Design Recommendations

- Define configuration settings and updates for application components or underlying infrastructure as code.
  - Manage any changes to code through consistent release and update process, including tasks such as key or secret rotation and permission management.
  - Prioritize pipeline-managed update processes, such as with scheduled pipeline runs, over built-in auto-update mechanisms.

- Do not use central processes or provisioning pipelines for the instantiation or management of AlwaysOn application resources, since this introduces external application dependencies and additional risk vectors, such as those associated with 'noisy neighbor' scenarios.
  - If centralized provisioning processes are mandated, ensure the availability requirements of leveraged dependencies are fully in-line with application requirements, and ensure operational transparency is provided to allow for holistic operationalization of the end-to-end application.

- Embrace continuous improvement and allocate a proportion of engineering capacity within each sprint to optimize platform fundamentals.
  - Consider allocating 20-40% of capacity within each sprint to drive fundamental platform improvements and bolster reliability.

- To accelerate the development of new services, consider the creation of a common engineering criteria and reference architectures/libraries for service teams to leverage, ensuring consistent alignment with core AlwaysOn principles.
  - Enforce a consistent baseline configurations for reliability, security, and operations through a policy-driven approach using Azure Policy.

> This common engineering criteria and associated artifacts, such as Azure Policies and Terraform for common design patterns, can also be leveraged across other workloads within the broader application ecosystem for an organization.

- Consider a DevSecOps for security sensitive and highly-regulated scenarios, to ensure security is baked within the DNA of engineering team throughout the development lifecycle rather than a specific release stage/gate.

- Apply a zero trust model within critical application environments, leveraging capabilities such as Azure AD Privileged Identity Management (PIM) to ensure consistent operations only occur through CI/CD processes or automated operational procedures.
  - No physical users should have standing write-access to any environment, maybe with the exception of development environments for easier testing and debugging.

- Define emergency processes for Just-in-time access to production environments.
  - Ensure 'break glass' accounts exist in the event of serious issues with the authentication provider.

- Consider AIOps as a method to continually improve operational procedures and triggers.

## Application Operations

The application design and platform recommendations provided within the AlwaysOn design methodology have a significant bearing on effective operations, and the extent to which these recommendations are adhered to will therefore greatly influence an optimal operational procedures.

Furthermore, there is a varied set of operational capabilities provided by different Azure services, particularly when it comes to high availability and recovery. It is therefore also important to understand and leverage the operational capabilities of leveraged services.

This section will therefore highlight key operational aspects associated with AlwaysOn application design and recommended platform services.

### Design Considerations

- Azure services provide a combination of built-in (enabled by default) and configurable platform capabilities, such as zonal redundancy or geo-replication. The service-level configuration of each service within an application must therefore be considered by operational procedures.
  - Many configurable capabilities incur an additional cost, such as the multi-write deployment configuration for Cosmos DB.

- AlwaysOn strongly endorses the principle of ephemeral stateless application resources, meaning that updates can typically be performed through a new deployment and standard delivery pipelines.

- The vast majority of the required operations are exposed and accessible via the Azure ARM management APIs or through the Azure Portal.

- Some more intensive operations, such as a restore from a periodic backup Cosmos DB or the recovery of a deleted resource, can only be performed by Azure Support Engineers via a Support Case.

- For stateless resources and resources which can be entirely configured from deployment, such as Azure Front Door and associated backends/origins, re-deployment will faster generally result in an operational resource than a Support process to attempt recovery of the deleted resource.

- Azure Policy provides a framework to enforce and audit security and reliability baselines, ensuring continued compliance with a common engineering criteria for an AlwaysOn application. More specifically, Azure Policy forms a key part of the Azure Resource Manager (ARM) control plane, supplementing RBAC by restricting what actions authorized users can perform, and can be leveraged to enforce vital security and reliability conventions across utilized platform services.
  - Azure Policy can be extended within Azure Kubernetes Service (AKS) via [Azure Policy for Kubernetes](https://docs.microsoft.com/azure/governance/policy/concepts/policy-for-kubernetes) which provides visibility of components deployed within clusters.

- Azure [resources can be locked](https://docs.microsoft.com/azure/azure-resource-manager/management/lock-resources) to prevent them from being modified or deleted.
  - Locks introduce management overhead within deployment pipelines which must remove locks, perform deployment steps, before subsequently re-enabling.
  - Generally, for most resource a robust RBAC process, with tight restrictions on who can perform write operations, should be preferred over locking resources.

**Update Management**

- Key, secret, and certificate expiry are common causes of application outage.

- The Kubernetes version within AKS needs to be updated on a regular basis, especially given support for older versions is not sustained.
  - Components running on K8s also need to be updated, such as cert-manager and Key Vault-csi, and aligned with the k8s version within AKS.

- Terraform providers need to be updated on a regular basis. Newer provider versions frequently contain breaking changes which must be properly tested before applied in production.

- Other application dependencies, such as the runtime environment (.NET, Java, Python), should be be monitored and kept up-to-date.

- New versions of packages, components, and dependencies need to be properly tested before consideration in a production context.

- Container registries will likely need regular housekeeping to delete old image versions that are not used anymore.

- Azure Policy provides native support for a wide variety of Azure resources.

### Design Recommendations

- Use an active-active deployment model, leveraging a health model and automated scale-operations to ensure no failover intervention is required.
  - If using an active-passive or active-standby model, ensure failover procedures are automated or at least codified within pipelines so that manual steps besides triggering is required during operational crises.

- Prioritize the use of Azure-native auto-scale functionality for all available services.
  - Establish automated operational processes to scale services which do not offer auto-scale.
  - Leverage scale-units composed of multiple services to provide requisite scalability under relevant circumstances.

- Identify operational procedures and tasks required by global (long-lived) application resources, such as Cosmos DB or ACR in the AlwaysOn reference implementation.
  - For example, in the event that a Cosmos DB resource or encompassed data is incorrectly modified or deleted, the possible ways of recovery should be well understood and a recovery process should exist.
  - Similarly, establish procedures to manage decommissioned container images in the registry.

- It is strongly recommended to practice recovery operations in advance, on non-production resources and data, as part of standard business continuity preparations.

- Use platform-native capabilities for backup and restore, ensuring they are aligned with RTO/RPO and data retention requirements.
  - Avoid building custom solutions unless absolutely necessary.
  - Define a strategy for long-term backup retention if required.

- Use built-in capabilities for SSL certificate management and renewal, such as those offered by Azure Front Door.

- Wherever possible, use Managed Identities in order to avoid dealing with Service Principal credentials or API keys.

- When storing secrets, keys or certificates in Azure Key Vault, make use of the expiry setting and have [alerting configured](https://docs.microsoft.com/azure/key-vault/general/event-grid-tutorial) for upcoming expirations.
  - All key, secret, and certificate updates should be performed using the standard release process.

- Identify critical operational alert and define target audiences and systems, with clear channels to reach them.
  - Avoid 'white-noise' by only sending actionable alerts, to prevent operational stakeholders for ignoring alerts and missing important information.
    - Leverage continuous improvement to optimize alerting and remove observed 'white-noise'.

- Apply the principle of policy-driven governance and Azure Policy to ensure the appropriate use of operational capabilities and a reliable configuration baselines across all application services.

- Apply a resource lock to prevent the deletion of long-lived global resources, such as Azure Cosmos DB.
  - Avoid the use of resource locks on ephemeral regional resources, and instead rely on the appropriate use of Role Based Access Control (RBAC) and CI/CD pipelines to control operational updates.

**Update Management**

- Update external libraries, SDKs, and runtimes frequently, treating it as any other change to the application. This will ensure the latest security vulnerabilities and performance optimizations are applied.
  - Ensure all updates are validated prior to production release.
  - Setup processes to monitor and automatically detect updates, such as [GitHub's Dependabot](https://github.com/dependabot).

- All operational tasks, such as key and secret rotation, should be handled using either Azure-native platform capabilities or via a standard release process applied for code and configuration changes.
  - Ensure key, secret, and certificate rotation is performed on a regular basis.

- Manual operational changes to update components should be avoided and only considered by emergency exception.
  - Ensure a process exists to reconciliate any manual changes back into the source repository, avoiding drift and issue recurrence.
  - Establish an automated housekeeping procedure to [remove old image versions from Azure Container Registry](https://docs.microsoft.com/azure/container-registry/container-registry-auto-purge).

---

|Previous Page|Next Page|
|:--|:--|
|[Security](./Security.md) |[AlwaysOn Reference Implementation](https://github.com/Azure/AlwaysOn/blob/main/docs/reference-implementation/README.md)

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

[AlwaysOn - Full List of Documentation](/docs/README.md)
