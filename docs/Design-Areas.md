# Critical design areas

The 8 design areas below represent the architecturally significant topics which must be discussed and designed for when defining a target Mission-Critical application architecture. In this regard, this section of the repository is intended to provide prescriptive and opinionated guidance to support readers in designing a Mission-Critical solution.

- [Application Design](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-application-design)
- [Application Platform](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-application-platform)
- [Data Platform](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-data-platform)
- [Health Modeling](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-health-modeling)
- [Deployment and Testing](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-deployment-testing)
- [Networking and Connectivity](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-networking-connectivity)
- [Security](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-security)
- [Operational Procedures](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-operational-procedures)

These eight critical design areas will be explored at length within ensuing pages, for which critical review considerations and design recommendations are provided along with their broader design impact across other areas. Ultimately, the design areas are interrelated and decisions made within one area can impact or influence decisions across the entire design, so readers are encouraged to use the provided design guidance to navigate the key design decisions.

[![Azure Mission-Critical Design Areas](/docs/media/mission-critical-design-areas.png "Azure Mission-Critical Design Areas")](./Design-Areas.md)

## Reference Architecture

An Azure Mission-Critical application architecture is defined by the various design decisions required to ensure both functional and non-functional business-requirements are fully satisfied. The target Mission-Critical architecture is therefore greatly influenced by the relevant business requirements, and as a result may vary between different application contexts.

The image below represents a target technical state recommended for mission-critical applications on Azure. It leverages a reference set of business requirements to achieve an optimized architecture for different target reliability tiers.

[![Azure Mission-Critical Online Foundational Reference Architecture](/docs/media/architecture-foundational-online.png "Azure Mission-Critical Online Foundational Reference Architecture")](./Design-Areas.md)

> The [foundational-online](https://github.com/Azure/Mission-Critical-Online) and [foundational-connected](https://github.com/Azure/Mission-Critical-Connected) reference implementations provide solution orientated showcases for the Azure Mission-Critical design methodology, demonstrating how this architecture pattern can be implemented alongside the operational wrappers required to maximize reliability and operational effectiveness.

## Cross Cutting Concerns

There are several critical cross-cutting themes which traverse the 8 design areas and are contextualized below for subsequent consideration within each design area.

### Scale limits

Various [limits and quotas within the Azure platform](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits) may have a significant bearing on large Mission-Critical application scenarios and must be appropriately considered by the target architecture.

> Limits and quotas may change as Azure seeks to further enhance the platform and user experience.

- Leverage subscriptions as scale units, scaling out resources and subscriptions as required
- Employ a scale unit approach for resource composition, deployment, and management
- Ensure scale limits are considered as part of capacity planning
- If available, use data gathered about existing application environments to explore which limits might be encountered

### Automation

Maximize reliability and operability through the holistic automation of all deployment and management activities.

- Automate CI/CD deployments for all application components
- Automate application management activities, such as patching and monitoring
- Use declarative management semantics over imperative
- Prioritize templating over scripting; only use scripting when it is not possible to use templates

### Azure roadmap alignment and regional service availability

Align the target architecture with the Azure platform roadmap to inform the application trajectory, and ensure that required services and features are available within the chosen deployment regions.

- Align with Azure engineering roadmaps and regional role out plans
- Unblock with preview services or by taking dependencies on the Azure platform roadmap
- Only take a dependency on committed services and features; validate roadmap dependencies with Microsoft engineering product groups

### Azure Landing Zone integration

[Azure Landing Zones](https://github.com/Azure/cloud-adoption-framework/ready/landing-zone/) provides prescriptive architectural guidance to define a reliable and scalable shared-service platform for enterprise Azure deployments with requisite centralised governance.

Azure Mission-Critical can integrate seamlessly within an Azure Landing Zone, and is deployable within both the *Online* or *Corp. Connected* Landing Zone formats as demonstrated within the image below.

[![Azure Mission-Critical and Landing Zone Integration](/docs/media/mission-critical-landing-zones.gif "Azure Mission-Critical Landing Zone Integration")](./Design-Areas.md)

It is crucial to understand and identify in which connectivity scenario a Mission-Critical application requires since Azure Landing Zones support different landing zones archetypes.

- In the context of an *Online* Landing Zone archetype, Mission-Critical operates as a completely independent solution, without any direct corporate network connectivity to the rest of the Enterprise-Scale architecture. The application will, however, be further safeguarded through the [*policy-driven management*]((https://github.com/Azure/Enterprise-Scale/wiki/How-Enterprise-Scale-Works#enterprise-scale-design-principles)) approach which is foundational to Enterprise-Scale, and will automatically integrate with centralized platform logging through policy.
  - A *Online* deployment can only really consider a public Mission-Critical application deployment since there is no private corporate connectivity provided.

- When deployed in a *Corp. Connected* Landing Zone context, the Mission-Critical application takes a dependency on the Enterprise-Scale platform to provide connectivity resources which allow for integration with other applications and shared services existing on the platform. This necessitates some transformation on-top of the *Online* integration approach, since some foundational resources are expected to exist up-front as part of the shared-service platform. More specifically, the Mission-Critical regional deployment stamp should not longer encompass an ephemeral Virtual Network or Azure Private DNS Zone since these will exist within the Enterprise-Scale *connectivity* subscription.
  - A *Corp. Connected* deployment can consider both a public or private Mission-Critical application deployment.

> The Azure Mission-Critical reference implementations are fully aligned with the Azure Landing Zones architectural approach and are immediately deployable within an *Online* or *Connected* Landing Zone subscription.

---

|Previous Page|Next Page|
|--|--|
|[Azure Mission-Critical Design Principles](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-design-principles)|[Application Design](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-application-design)

---

|Design Methodology|
|--|
|[How to use the Azure Mission-Critical Design Methodology](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-design-methodology)
|[Azure Mission-Critical Design Principles](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-design-principles)
|[Azure Mission-Critical Design Areas](./Design-Areas.md)
|[Application Design](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-application-design)
|[Application Platform](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-application-platform)
|[Data Platform](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-data-platform)
|[Health Modeling and Observability](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-health-modeling)
|[Deployment and Testing](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-deployment-testing)
|[Networking and Connectivity](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-networking-connectivity)
|[Security](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-security)
|[Operational Procedures](https://docs.microsoft.com/azure/architecture/framework/mission-critical/mission-critical-operational-procedures)

---

[Azure Mission-Critical | Documentation Inventory](/docs/README.md)
