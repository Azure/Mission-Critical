# Mission-critical workloads

This section of the [Microsoft Azure Well-Architected Framework](http://docs.microsoft.com/azure/architecture/framework) strives to address the challenges of building mission-critical workloads on Azure. The guidance is based on lessons learned from reviewing numerous customer applications and first-party solutions. This section provides actionable and authoritative guidance that applies Well-Architected best practices as the technical foundation for building and operating a highly reliable solution on Azure at-scale.

## What is a mission-critical workload?

The term _workload_ refers to a collection of application resources that support a common business goal or the execution of a common business process, with multiple services, such as APIs and data stores, working together to deliver specific end-to-end functionality.

The term _mission-critical_ refers to a [criticality classification](http://docs.microsoft.com/azure/cloud-adoption-framework/manage/considerations/criticality) where a significant financial cost (business-critical) or human cost (safety-critical) is associated with unavailability or underperformance.

A _mission-critical workload_ therefore describes a collection of application resources which must be highly reliable on the platform, and strive to always be operational and available.

## What are the common challenges?

Microsoft Azure makes it easy to deploy and manage cloud solutions. However, building mission-critical workloads that are highly reliable on the platform remains a challenge for these main reasons:

- Designing a reliable application at scale is complex. It requires extensive platform knowledge to select the right technologies _and_ optimally configure them to deliver end-to-end functionality.

- Failure is inevitable in any complex distributed system, and the solution must therefore be architected to handle failures with correlated or cascading impact. This is a change in mindset for many developers and architects entering the cloud from an on-premises environment; reliability engineering is no longer an infrastructure subject, but should be a first-class concern within the application development process.

- Operationalizing mission-critical workloads requires a high degree of engineering rigor and maturity throughout the end-to-end engineering lifecycle as well as the ability to learn from failure.

## Is mission-critical only about reliability?

While the primary focus of mission-critical workloads is [Reliability](http://docs.microsoft.com/azure/architecture/framework/#reliability), other pillars of the Well-Architected Framework are equally important when building and operating a mission-critical workload on Azure.  

- [Security](http://docs.microsoft.com/azure/architecture/framework/security/): how a workload mitigates security threats, such as Dedicated Denial of Service (DDoS) attacks, will have a significant bearing on overall reliability.

- [Operational Excellence](http://docs.microsoft.com/azure/architecture/framework/devops/): how a workload is able to effectively respond to operational issues will have a direct impact on application availability. 

- [Performance Efficiency](http://docs.microsoft.com/azure/architecture/framework/scalability/): availability is more than simple uptime, but rather a consistent level of application service and performance relative to a known healthy state.

Achieving high reliability imposes significant cost tradeoffs, which may not be justifiable for every workload scenario. It is therefore recommended that design decisions be driven by business requirements.

## What are the key design areas?

Mission-critical guidance within this series is composed of architectural considerations and recommendations orientated around these key design areas.

![Mission-critical design areas](./images/mission-critical-design-areas.png "Mission-critical design areas")

The design areas are interrelated and decisions made within one area can impact or influence decisions across the entire design. The focus is ultimately on building a highly reliable application.

- **Application design**&mdash;Cloud application design patterns that allow for scaling, and error handling. 
- **Application platform**&mdash;Hosting environment choices, application dependencies, frameworks, and libraries.
- **Data platform**&mdash;Choices in data store technologies, informed by evaluating required volume, velocity, variety, and veracity characteristics.
- **Networking and Connectivity**&mdash;Network topology considerations at an application level, considering requisite connectivity and redundant traffic management.
- **Health Modeling**&mdash;Observability considerations through customer impact analysis correlated monitoring to determine overall application health.
- **Deployment and testing**&mdash;Strategies for CI/CD pipelines and automation considerations, with incorporated testing scenarios, such as synchronized load testing and failure injection (chaos) testing.
- **Security**&mdash;Mitigation of attack vectors through Microsoft Zero Trust model.
- **Operational procedures**&mdash; Processes related to deployment, key management, patching and updates.

We recommend that readers familiarize themselves with these design areas, reviewing provided considerations and recommendations to better understand the consequences of encompassed decisions. For example, to define a target architecture it's critical to determine how best to monitor application health across key components. In this instance, the reader should review the **health modeling** design area, using the outlined recommendations to help drive decisions.

## Illustrative examples

The guidance provided within this series is based on a solution-orientated approach to illustrate key design considerations and recommendations. There are several reference implementations available as part of an open source project on GitHub. These implementations can be used as a basis for further solution development.

- [Mission-Critical Online](https://github.com/Azure/Mission-Critical-Online)

  Provides a foundation for building a cloud-native, highly scalable, internet-facing application on Microsoft Azure.
  
  _The workload is accessed over a public endpoint and doesn't require private network connectivity to a surrounding organizational technical estate._

- [Mission-Critical Connected](https://github.com/Azure/Mission-Critical-Connected) 

  Provides a foundation for building a corporate-connected cloud-native application on Microsoft Azure using existing network infrastructure and private endpoints.
  
  _The workload requires private connectivity to other organizational resources and takes a dependency on pre-provided Virtual Networks for connectivity to other organizational resources. This use case is intended for scenarios that require integration with a broader organizational technical estate for either public-facing or internal-facing workloads._

## Next step

Start by reviewing the design methodology for mission-critical application scenarios.

> [!div class="nextstepaction"]
> [Design methodology](mission-critical-design-methodology.md)
