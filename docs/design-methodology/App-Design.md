# Application Design

Both functional application requirements and non-functional requirements, such as those surrounding high-availability and performance, are critical to inform key design decisions for an AlwaysOn application design. However, these requirements should be examined alongside key cloud application design patterns to ensure AlwaysOn aspirations are fully achieved. This design area will therefore explore requisite application design patterns for building a 'best of breed' reliable application on Azure.

There are ultimately a myriad of design patterns which can be applied to build reliable applications on Azure, from reliable virtual actors to circuit breaker for fault handling. This design area is not intended to cover all relevant patterns or substitute public [design pattern documentation](https://docs.microsoft.com/azure/architecture/patterns/), but rather strives to cover key requisite themes to create an AlwaysOn application design, establishing breadcrumbs for the reader to follow as they embark on a critical design path to define a target 'north star' architecture.

  - [Scale-Unit Architecture](#scale-unit-architecture)
  - [Global Distribution](#global-distribution)
  - [Loose Coupled Event-Driven Architecture](#loose-coupled-event-driven-architecture)
  - [Application-Level Resiliency Patterns and Error Handling](#application-level-resiliency-patterns-and-error-handling)

> The [foundational-online](https://github.com/Azure/AlwaysOn-Foundational-Online) and [foundational-connected](https://github.com/Azure/AlwaysOn-Foundational-Connected) reference implementations provide solution orientated showcases for how these foundational design concepts can be leveraged alongside Azure-native capabilities to maximize reliability.

## Scale-Unit Architecture

A scale-unit is a logical unit or function which can be scaled independently as required; it is a vital concept for achieving an AlwaysOn application design since all functional aspects of the solution must be capable of scaling to meet changes in demand. Architecturally, it is critical to optimize end-to-end scalability through the logical compartmentalization of operational functions into scale-units at all levels of the application stack, from code components to application hosting platforms and deployment stamps encompassing related components.

> Please refer to the [deployment stamps pattern](https://docs.microsoft.com/azure/architecture/patterns/deployment-stamp) for further details.

For example, the foundational reference implementation considers a user flow for processing game results which encompasses APIs for retrieving and posting game outcomes as well as supporting components such as an OAuth endpoint, datastore, and message queues. These stateless API endpoints for retrieving and posting results represent granular functional units that must be able to adapt to changes in demand, however, for these to be truly scalable the underlying application platform must also be able to scale in-kind. Similarly, to avoid performance bottlenecks in the end-to-end user flow and achieve sustainable scale, the downstream components and dependencies must also be able to scale to an appropriate degree, either independently as a separate scale-unit or together as part of a single logical unit.

[![AlwaysOn Scale Units](/docs/media/alwayson-scale-units.png "AlwaysOn Scale Units")](./App-Design.md)

The image above depicts the multiple scale-unit scopes considered by this reference implementation user flow, from microservice pods to cluster nodes and regional deployment stamps. 

Ultimately, a scale-unit architecture should be applied to optimize the end-to-end scalability of an AlwaysOn application design, so that all levels of the solution can appropriately scale. The relationship between related scale-units as well as components inside a single scale-unit should be defined according to a capacity model, taking into consideration non-functional requirements around performance.

### Design Considerations

- [Azure Subscription scale limits and quotas](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits) might have a bearing on application design, technology choices, and the definition of scale-units.

- The scale-unit architecture pattern goes great lengths to address scale limits of individual resources and the application as a whole.
  
- A scale-unit architecture helps with complex deployment and update scenarios since an entire regional stamp can be deployed as one unit, allowing for testing and validation of specific versions of components together prior to directing traffic to it.

- The scale-unit architectural pattern can also be applied to support multi-tenant requirements for customer segregation.

- The expected peak request rate (requests per second) and daily/weekly/seasonal traffic patterns are critical to inform core scale requirements. 
  - *How many requests is the solution required to support for each user flow? and are usage patterns predictable?*

- The expected growth patterns for both traffic and data volume inform the design with regards to sustainable scale. 
  - *Is traffic expected to grow? and at what rate?*

- The required performance of the solution under load is a critical decision factor when modelling required capacity. 
  - *Is a degraded service with high response times acceptable under load?*

### Design Recommendations

- Define a scale-unit when the scale-limits of a single deployment are likely to be exceeded.

- Ensure all application components are able to scale either as independent scale-units or as part of a logical scale-unit encompassing multiple related components.

- Define the relationship between scale-units according to a capacity model and non-functional requirements.

- **Define a regional deployment stamp** to unify the provisioning, management, and operation of regional application resources into a heterogenous but inter-dependent scale-unit. 
  - As load increases, additional stamps can be deployed within the same or different Azure regions to horizontally scale the solution. 

> When deploying an AlwaysOn solution within an Azure Landing Zone, ensure the Landing Zone subscription is dedicated to the application to provide a clear management boundary and avoid potential 'noisy neighbor' capacity risks.

- For high-scale application scenarios with significant volumes of traffic, design the solution to scale across multiple Azure Subscriptions to ensure the inherit scale-limits within a single subscription do not constrain scalability.
  - Define a subscription scoped deployment as a scale-unit to avoid a 'spill-and-fill' subscription model.
    - Deploy each regional deployment stamp within a dedicated subscription to ensure subscription limits only apply within the context of a single deployment stamp and not across the application as a whole.
      - Where appropriate, multiple deployment stamps can be considered within a single region, but should be deployed across independent subscriptions.
    - Separate 'global' shared resources within a dedicated subscription to allow for consistent regional subscription deployment; avoid having a specialized deployment for a 'primary' region.
> The use of multiple subscriptions necessitates additional CI/CD complexity which must be appropriately managed. It is therefore only recommended in extreme scale scenarios where the limits of a single subscription are likely to become a hindrance.

- Where multiple production subscriptions are needed to ensure requisite scale, consider using a dedicated application Management Group to simplify policy assignment through a policy aggregation boundary.

- Deploy any considered environments, such as production, development, or test environments, into separate subscriptions to ensure lower environments do not contribute towards scale limits, and to reduce the risk of lower environment updates polluting production by providing a clear management and identity boundary.

- Define and analyze non-functional requirements, such as the availability SLO, within the context of key end-to-end user-flows since technical and business scenarios will likely have distinct considerations for resilience, availability, latency, capacity, and observability. 
  - This will allow for relative flexibility in the AlwaysOn design approach, tailoring design decisions and technology choices at a user-flow level since one size may not fit all.

- Model required capacity around identified traffic patterns to ensure sufficient capacity provisioned at peak times to prevent service degradation.
  - Leverage traffic patterns to optimize capacity and resource utilization during periods of reduced traffic.

- Measure the time it takes to perform scale-out and scale-in operations to ensure natural variations in traffic do not create an unacceptable level of service degradation.
  - Track scale operation durations as an operational metric to drive continuous improvement.

### Reference Subscription Scale-Unit Approach

The image below demonstrates how the single subscription reference deployment model can be expanded across multiple subscriptions in extreme scale scenario to navigate subscription scale-limits.

[![AlwaysOn Subscription Scale Units](/docs/media/alwayson-subscription-scale.gif "AlwaysOn Subscription Scale Units")](./App-Design.md)

## Global Distribution

Unfortunately, failure is impossible to avoid in a highly distributed environment like Azure. Only by planning for failure can a solution be truly AlwaysOn.

[Availability Zones](/azure/availability-zones/az-overview#availability-zones) (AZ) allows highly-available regional deployments across different data centers within a region. Nearly all Azure services are available in either a zonal configuration (where service is pinned to a specific zone) or zone-redundant configuration (where the platform automatically ensures the service spans across zones and can withstand a zone outage). These configurations allow for fault-tolerance up to a datacenter level.

While Availability Zones can be used to mitigate many fault scenarios, to maximize reliability multiple Azure regions should be used to ensure regional fault tolerance, so that application availability remains even in the event of a disaster scenario, such as Godzilla stepping on an Azure region. When defining a multi-region AlwaysOn application design, consideration should be given to different deployment strategies, such as active-active and active-passive, alongside application requirements, since there are significant trade-offs between each approach.

An active-active deployment strategy represents the gold standard for an AlwaysOn solution, since it maximizes availability and allows for higher composite SLAs. While active-active is the recommended approach, it can introduce challenges around data synchronization and consistency for many application scenarios, and these challenges must be fully addressed at a data platform level, alongside additional trade-offs, from increased cost exposure and increased engineering effort.

Not every workload supports or requires multiple regions running simultaneously, and hence the precise application requirements should be weighed against these trade-offs to inform an optimal design decision. For certain application scenarios with lower reliability targets, different deployment models, such as active-passive or sharding, can be suitable alternatives.

It's important to note that some Azure services are deployable or configurable as global resources, which are not constrained to a particular Azure region. Consequently, when accommodating both 'Scale-Unit Architecture' and 'Global Distribution', careful consideration should be given to how resources are optimally distributed across Azure regions. For example, the foundational reference implementation for AlwaysOn consists of both global and regional resources, with regional resources deployed across multiple regions to provide geo-availability, in the case of regional outages and to bring services closer to end-users. These regional deployments also serve as scale-unit "stamps" to provide additional capacity and availability when required.

[![AlwaysOn Foundational-Online Architecture](/docs/media/alwayson-high-level-architecture.png "AlwaysOn Foundational-Online Architecture")](./App-Design.md)

The previous image depicts the high-level active-active design for the [foundational-online reference implementation](https://github.com/azure/alwayson-foundational-online), where a user accesses the application via a central global entry point that then redirects requests to a suitable regional deployment stamp.

### Design Considerations

- Not all services or capabilities are available in every Azure region, and consequently there can be service availability implications depending on the selected deployment regions.
  - For example, [Availability Zones](https://docs.microsoft.com/azure/availability-zones/az-region) are not available in every region.

- Azure regions are grouped into [regional pairs](https://docs.microsoft.com/azure/best-practices-availability-paired-regions) consisting of two regions within the same geography.
  - Some Azure services leverage paired regions to ensure business continuity and to protect against data loss. For example, Azure Geo-redundant Storage (GRS) replicates data to a secondary paired region automatically, ensuring that data is durable in the event that the primary region is not recoverable.

- If an outage affects multiple Azure regions, at least one region in each pair will be prioritized for recovery.

- The [Azure Safe Deploy Practice (SDP)](https://azure.microsoft.com/blog/advancing-safe-deployment-practices) ensures all code and configuration changes (planned maintenance) to the Azure platform undergo a phased roll-out, with health analyzed in case any degradation is detected during the release.
  - Once the 'Canary' and 'Pilot' phases have been successfully completed, platform updates are serialized across regional pairs, ensuring that only one region in each pair is updated at a time.

- Like any cloud provider, Azure ultimately has a finite amount of resources and as a result there are situations which can lead to the unavailability of capacity in individual regions. 
  - In the event of a regional outage there will be a significant increase in demand for resources within the paired region as impacted customer workloads seek to recover within the paired region. In certain scenarios this may create a capacity challenge where supply temporarily does not satisfy demand. 

- Compliance requirements around geographical data residency, data protection, and data retention can have a significant bearing on appropriate geographical distribution. 
  - *Are there specific regions where data must reside or where resources have to be deployed?*

- The geographic proximity and density of users or dependent systems should inform design decisions around the global distribution of an AlwaysOn application. 
  - *Where are the requests physically originating from?*

- The connectivity method by which users or systems access the application, whether over the public Internet or private networks leveraging either VPN or Express Route connectivity. 
  - *Are users going to connect from home and/or organizational networks?*
  - *Can all users be expected to have fast internet connections?*

- Different Azure regions have slightly different cost profiles for some services. There may be further cost implications depending on the precise deployment regions chosen.

- Availability Zones have a latency perimeter of less than two milliseconds between availability zones.
  - For workloads which are particularly 'chatty' across zones this latency can accumulate to form a non-trivial performance penalty, as well as incurring bandwidth charges for inter-zone data transfer.

  - An active-active deployment across Microsoft Azure and other cloud providers can be considered to further mitigate reliance on global dependencies within a single cloud provider.
    - A multi-cloud active-active deployment strategy introduces a significant amount of complexity where CI/CD is concerned, particularly given the significant difference in resource specifications and capabilities between cloud providers, which necessitates specialised deployment stamps for each cloud.  

### Design Recommendations

- Deploy the solution within a minimum of 2 Azure regions to protect against regional outages.
  - Prioritize the use of paired regions to benefit from SDP risk mitigations and platform recovery capabilities. 
  > For scenarios targeting a >= 99.99% SLO, a minimum of 3 deployment regions should be used to maximize the composite SLA and overall reliability.

- Use an active-active deployment strategy where possible to maximize reliability.
  - Where data/state consistency challenges exist explore the use of a) a globally distributed data store, b) stamped regional architecture, or c) a partially active-active deployment, where some components are active across all regions while others are located centrally within a primary region.

- Calculate the [composite SLA](https://docs.microsoft.com/azure/architecture/framework/resiliency/business-metrics#composite-slas) for all user flows.
  - Ensure the composite SLA is in-line with business targets.

- Deploy additional regional deployment stamps to achieve a greater composite SLA.
  - The use of global resources will constrain the increase in composite SLA from adding further regions.

- Define and validate that Recovery Point Objectives (RPO) and Recovery Time Objectives (RTO).

- Geographically co-locate Azure resources with users to minimize network latency and maximize end-to-end performance.
  - Technical solutions such as a Content Delivery Network (CDN) or edge caching can also be used to drive optimal network latency for distributed user bases.

- For high-scale application scenarios with significant volumes of traffic, design the solution to scale across multiple regions to navigate potential capacity constraints within a single region.

- Select deployment regions which offer requisite capabilities and characteristics to achieve performance and availability targets, while fulfilling data residency and retention requirements.
  - Within a single geography, prioritize the use of regional pairs to benefit from SDP serialized rollouts for planned maintenance, and regional prioritization in the event of unplanned maintenance.
  
- It is not uncommon that data compliance requirements will constrain the number of available regions and potentially force AlwaysOn design compromises. In such cases, additional investment in operational wrappers is highly recommended to predict, detect, and respond to failures.
  - If only a single Azure region is suitable, multiple deployment stamps ('regional scale-units') should be deployed within the selected region to mitigate some risk, leveraging Availability Zones to provide datacenter-level fault tolerance. However, such a significant compromise in geographical distribution will drastically constrain the attainable composite SLA and overall reliability. 
  - If suitable Azure regions do not all offer requisite capabilities, be prepared to compromise on the consistency of regional deployment stamps to prioritize geographical distribution and maximize reliability. 
    - For example, when constrained to a geography with two regions where only one region supports Availability Zones (3 + 1 datacenter model), create a secondary deployment pattern using fault domain isolation to allow for both regions to be deployed in an active configuration, ensuring the primary region houses multiple deployment stamps.

- Align current service availability with product roadmaps when selecting deployment regions; not all services may be available in every region on day 1.

- Leverage Availability Zones where possible to maximize availability within a single Azure region.

### Reference Global Distribution Approach

The image below demonstrates how the the AlwaysOn reference application can be designed to scale across multiple Azure regions, with consideration given to scenarios where constraints on available regions necessitate multiple deployment stamps within a single region.

[![AlwaysOn Global Distribution](/docs/media/alwayson-global-distribution.gif "AlwaysOn Global Distribution")](./App-Design.md)

## Loose Coupled Event-Driven Architecture

Loose coupling provides the cornerstone of a microservice architecture by allowing services to be designed in a way that each service has little or no knowledge of surrounding services. More specifically, it allows a service to operate independently ("loose") while still communicating with other services through well-defined interfaces ("coupling"), and in the context of AlwaysOn it further facilitates high-availability by preventing downstream failures from cascading to frontends or different deployment stamps. The following list captures the key characteristics of loose coupling, which should be evaluated when defining an AlwaysOn application design.

- Services are not constrained to use the same compute platform, programming language, runtime, or operating system.
- Services can scale independently, optimizing the use of infrastructure and platform resources.
- Failures can be handled separately and do not affect client transactions.
- Transactional integrity is harder to maintain because data creation and persistence happens within separate services.
- End-to-end tracing requires more complex orchestration.

When implementing loose coupling, **Event-driven architecture** and **asynchronous processing** are key design patterns which should be applied for interactions which do not require an immediate response. Events represent a change in state within entities and are generated by event *producers* (emitters). Producers do not know anything about how events should be processed or handled, since that is the responsibility of event *consumers*. When using asynchronous event-driven communication, a producer publishes an event when something happens within its domain which another component needs to be aware of, such as a price change in a product catalogue, which consumers will subscribe to receive so they can process the events asynchronously.

[![Asynchronous Event-Driven Communication](/docs/media/alwayson-asynchronous-communication.png)](./App-Design.md)
*Image source: [Asynchronous Message-Based Communication](https://docs.microsoft.com/dotnet/architecture/microservices/architect-microservice-container-applications/asynchronous-message-based-communication)*

In reality, applications can combine loose and tight-coupling, depending on business objectives. For example, the AlwaysOn reference implementation applies write operations asynchronously with a message bus and worker, while read operations are synchronous with the result directly returned to the caller.

### Design Considerations

- Not all content that a solution makes available over the Internet is generated dynamically. Applications serve both static assets (images, JavaScript, CSS, localization files, etc.) and dynamic content.
  - Workloads with frequently accessed static content benefit greatly from caching since it reduces the load on backend services and reduces content access latency for end users.

- Caching can be implemented natively within Azure using either Azure Front Door or Azure Content Delivery Network (CDN).
  - [Azure Front Door](https://docs.microsoft.com/azure/frontdoor/front-door-caching) provides Azure-native edge caching capabilities as well as routing features to divide static and dynamic content. 
    - By creating the appropriate routing rules in Azure Front Door, `/static/*` traffic can be transparently redirected to static content. 
  - More complex caching scenarios can be implemented using the [Azure CDN](https://azure.microsoft.com/services/cdn) service to establish a full-fledged content delivery network for significant static content volumes. 
    - The Azure CDN service will likely be more cost effective, but does not provide the same advanced routing and Web Application Firewall (WAF) capabilities which are recommended for other areas of an AlwaysOn application design. It does, however, offer further flexibility to integrate with similar services from third-party solutions, such as Akamai and Verizon.
  - When comparing the Azure Front Door and Azure CDN services, the following decision factors should be explored:
    - Can required caching rules be accomplished using the rules engine.
    - Size of the stored content and the associated cost.
    - Price per month for the execution of the rules engine (charged per request on Azure Front Door).
    - Outbound traffic requirements (price differs by destination).

### Design Recommendations

- Key functionality should be deployed and managed as independent loosely-coupled microservices with event-driven interaction through well-defined interfaces (synchronous and asynchronous).
  - The definition of microservice boundaries should consider and align with critical user-flows.

- Use event-driven asynchronous communication where possible to support sustainable scale and optimal performance.

- Separate the delivery of static and dynamic content to users and deliver relevant content from a cache to reduce load on backend services optimize performance for end-users.

- Given the strong recommendation (Network and Connectivity design area) to use Azure Front Door for global routing and WAF purposes, it is recommended to prioritize the use of Azure Front Door caching capabilities unless gaps exist. 

## Application-Level Resiliency Patterns and Error Handling

A reliable application architecture is fundamental to maximizing reliability, however, if application code is not also developed with resiliency in-mind, overall reliability will be severely constrained. It is therefore critical that application code be designed and developed to be resilient, ensuring that the application can respond to failure, which is ultimately an unavoidable characteristic of highly distributed multi-tenant cloud environments like Azure.

More specifically, all application components should be designed from the ground-up to apply key resiliency patterns for self-healing, such as [retries with back-off](https://docs.microsoft.com/dotnet/architecture/microservices/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly) and [circuit breaker](https://docs.microsoft.com/dotnet/architecture/microservices/implement-resilient-applications/implement-circuit-breaker-pattern). Such patterns go great lengths to transparently handle transient faults such as network packet loss, or the temporary loss of a downstream dependency. Ultimately, it is paramount that the application code cater for as many failure scenarios as possible in order to maximize service availability and reliability.

When issues are not transient in-nature and cannot be fully mitigated within application logic, it becomes the role of the health model and operational wrappers to take corrective action. However, for this to happen effectively, it is essential that application code incorporate proper instrumentation and logging to inform the health model and facilitate subsequent troubleshooting or root cause analysis when required. More specifically, application code should be implemented to facilitate [Distributed Tracing](https://docs.microsoft.com/dotnet/core/diagnostics/distributed-tracing-concepts), by providing the caller with a comprehensive error message that includes a correlation ID when a failure occurs.

> Tools like [Azure Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/distributed-tracing) can help significantly to query, correlate, and visualize application traces.

### Design Considerations

- Vendor provided SDKs, such as the Azure service SDKs, will typically provide built-in resiliency capabilities like retry mechanisms.

- It is not uncommon for application responses to transient issues to cause cascading failures.
  - For example, retry without appropriate back-off will exacerbate when a service is being throttled will likely exacerbate the issue.

- Retry delays can be linearly spaced, or increase exponentially to 'backoff' via growing delays.

- [Queue-Based Load Leveling](https://docs.microsoft.com/azure/architecture/patterns/queue-based-load-leveling) introduces a buffer between consumers and requested resources to ensure consistent load levels.
  - As consumer requests are enqueued, a worker process dequeues the requests and processes them against the requested resource at a pace set by the worker and the requested resource's ability to process the requests. 
  - If consumers expect replies to their requests, a separate response mechanism will also need to be implemented.

- The [Circuit Breaker](https://docs.microsoft.com/azure/architecture/patterns/circuit-breaker) design pattern provides stability by either waiting for recovery, or quickly rejecting requests rather than blocking while waiting for an unavailable remote service or resource.

- The [Bulkhead](https://docs.microsoft.com/azure/architecture/patterns/bulkhead) design pattern strives to partition service instances into groups based on load and availability requirements, isolating failures to sustain service functionality.

- The [Saga](https://docs.microsoft.com/azure/architecture/reference-architectures/saga/saga) pattern can be used to manage data consistency across microservices with independent datastores by ensuring services update each other through defined event or message channels.
    - Each service performs local transactions to update its own state and publishes an event to trigger the next local transaction in the saga.
    - If a service update fails, the saga executes compensating transactions to counteract preceding service update steps. 
    - Individual service update steps can themselves implement resiliency patterns, such as retry. 

### Design Recommendations

- Design and develop application code to anticipate and handle failures.

- Use vendor provided SDKs, such as the Azure SDKs, to connect to dependent services.
  - Leverage the resiliency capabilities provided by utilized SDKs instead of re-implementing resiliency functionality.
  - Ensure a suitable back-off strategy is applied when retrying failed dependency calls to avoid a self-inflicted DDoS scenario.

- Define **common engineering criteria** for all application microservice teams to drive consistency and acceleration regarding the use application-level resiliency patterns.
  - Developers should familiarize themselves with [common software engineering patterns](https://docs.microsoft.com/azure/architecture/patterns/) for resilient applications.

- Implement resiliency patterns using proven standardized packages, such as [Polly for C#](http://www.thepollyproject.org/) or [Sentinel for Java](https://github.com/alibaba/Sentinel).

- Implement [Health Endpoint Monitoring](https://docs.microsoft.com/azure/architecture/patterns/health-endpoint-monitoring) by exposing functional checks within application code through health endpoints which external monitoring solutions can poll to retrieve application component health statuses. 
  - Responses should be interpreted alongside key operational metrics to inform application health and trigger a operational responses, such as raising an alert or performing a compensating roll-back deployment.

- Implement [Queue-Based Load Leveling](https://docs.microsoft.com/azure/architecture/patterns/queue-based-load-leveling) to smooth out intermittent heavy loads which can cause requested resources to time out or fail. 
  - Apply a prioritized ordering so that the most important activities are performed first.

- Implement the [Retry](https://docs.microsoft.com/azure/architecture/patterns/retry) pattern to enable application code to handle transient failures elegantly and transparently.
  - Cancel if the fault is unlikely to be transient and is unlikely to succeed if the operation is re-attempted.
  - Retry if the fault is unusual or rare and the operation is likely to succeed if attempted again immediately.
  - Retry after a delay if the fault is caused by a condition that may need a short time to recover, such as network connectivity or high load failures.
    - Apply a suitable 'backoff' strategy with growing retry delays.

- Implement the [Circuit Breaker](https://docs.microsoft.com/azure/architecture/patterns/circuit-breaker) design pattern to handle faults that might take a variable amount of time to recover from when connecting to a remote service or resource. 

- Use [Throttling](https://docs.microsoft.com/azure/architecture/patterns/throttling) to control the consumption of resources used by application components, protecting them from becoming over encumbered. 
    - When a resource reaches a load threshold, it should safeguard its availability by deferring lower-importance operations and degrading non-essential functionality so that essential functionality can continue until sufficient resources are available to return to normal operation. 

- Consider the [Saga](https://docs.microsoft.com/azure/architecture/reference-architectures/saga/saga) pattern for scenarios where data consistency needs ensured across microservice boundaries.
    - Roll back or compensate if one of the operations in the sequence fails.

- Use correlation IDs for all trace events and log messages to tie them to a given request.
  - Return correlation IDs to the caller for all calls not just failed requests.

- Use [structured logging](https://stackify.com/what-is-structured-logging-and-why-developers-need-it/) for all log messages.

- Select a unified operational data sink for application traces, metrics, and logs to enable operators to seamlessly debug issues.
  - Ensure operational data is used in conjunction with business requirements to inform an [application health model](./Health-Modeling.md).

---

|Previous Page|Next Page|
|--|--|
|[AlwaysOn Design Areas](./Design-Areas.md)|[Application Platform](./App-Platform.md)

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
