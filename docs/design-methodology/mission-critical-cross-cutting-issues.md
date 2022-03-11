# Cross-cutting concerns of mission-critical workloads

There are several cross-cutting concerns that traverse the [key design areas](mission-critical-overview.md#what-are-the-key-design-areas). This article contextualizes these cross-cutting concerns for subsequent consideration within each design area.

## Scale limits

Azure applies various _limits_ or _quotas_ to ensure a consistent level of service for all customers. Examples of these limits include restrictions on the number of deployable resources within a single subscription, and restrictions to network and query throughput.

Service limits may have a significant bearing on a large mission-critical workload. Consider the limits of the services used in the target architecture carefully to ensure sustainable scale. Otherwise, you may hit one or more of these limits as the workload grows.

> [!IMPORTANT]
> Limits and quotas may change as the platform evolves. Be sure to check the current limits at [Azure subscription and service limits, quotas, and constraints](http://docs.microsoft.com/azure/azure-subscription-service-limits).

### Recommendations

- Use subscriptions as scale units, scaling out resources and subscriptions as required.
- Employ a scale unit approach for resource composition, deployment, and management.
- Ensure scale limits are considered as part of capacity planning.
- If available, use data about existing application environments to explore which limits might be encountered.

## Automation

A holistic approach to automation of deployment and management activities can maximize the reliability and operability of the workload.

### Recommendations

- Automate continuous integration and continuous delivery (CI/CD) pipelines for all application components.
- Automate application management activities, such as patching and monitoring.
- Use declarative management semantics, such as Infrastructure as code (IaC), instead of over imperative approaches.
- Prioritize templating over scripting. Defer to scripting only when using templates isn't possible.

## Azure roadmap alignment

Azure is constantly evolving through frequent updates to services, features, and regional availability. It's important to align the target architecture with Azure platform roadmaps to inform an optimal application trajectory. For example, making sure that the required services and features are available within the chosen deployment regions.

Refer to [Azure updates](https://azure.microsoft.com/updates/) for the latest information about new services and features. 

### Recommendations

- Align with Azure engineering roadmaps and regional rollout plans.
- Unblock with preview services or by taking dependencies on the Azure platform roadmap.
- Only take a dependency on committed services and features; validate roadmap dependencies with Microsoft engineering product groups.

## Next step

Explore the design areas that provide critical considerations and recommendations for building a mission-critical workload.

> [!div class="nextstepaction"]
> [Architecture pattern](mission-critical-architecture-pattern.md)
