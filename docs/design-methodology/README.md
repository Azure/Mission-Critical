# How to Use the Azure Mission-Critical design methodology

The Azure Mission-Critical design methodology is intended to define easy to follow guidance surrounding critical design decisions which must be made to produce a target Mission-Critical architecture.

## Critical design path

At the heart of a Mission-Critical target architecture definition lies a critical design path, comprised of 5 foundational [design principles](./Principles.md) and 8 [fundamental design areas](./Design-Areas.md) with heavily interrelated and dependent design decisions.

Ultimately, the impact of decisions made within each area will reverberate across other design areas and design decisions. Readers are strongly advised to familiarize themselves with these 8 critical design areas, reviewing provided considerations and recommendations to better understand the consequences of encompassed decisions, which may later produce trade-offs or unforeseen consequences within related areas. For example, to define a target architecture it is critical to determine how best to monitor application health across key components. In this instance, the reader should review the Health Modelling design area, using the outlined recommendations to help drive decisions.

## Design for business requirements

Not all business-critical applications have the same requirements, and as a result the review considerations and design recommendations provided by the Azure Mission-Critical design methodology may yield different design decisions and trade-offs which is to be expected.

### Reliability tiers

Reliability is a subjective concept and for an application to be appropriately reliable it must reflect the business requirements surrounding it. For example, a mission-critical application with a 99.999% availability Service Level Objective (SLO) requires a much higher level of reliability and operational rigour than another application with an SLO of 99.9%. However, there are obvious financial and opportunity cost implications for introducing greater reliability, and such trade-offs should be carefully considered.

The Azure Mission-Critical design methodology leverages several reliability tiers orientated around an availability SLO to introduce further specificity to design recommendations. These tiers are therefore intended as a reference to help readers better navigate requisite design decisions to achieve required levels of reliability.

|Reliability Tier (Availability SLO)|Permitted Downtime (Week)|Permitted Downtime (Month)|Permitted Downtime (Year)|
|--|--|--|--|
|99.9%|10 minutes, 4 seconds|43 minutes, 49 seconds|8 hours, 45 minutes, 56 seconds|
|99.95%|5 minutes, 2 seconds|21 minutes, 54 seconds|4 hours, 22 minutes, 58 seconds|
|99.99%|1 minutes|4 minutes 22 seconds|52 minutes, 35 seconds|
|99.999%|6 seconds|26 seconds|5 minutes, 15 seconds|
|99.9999%|<1 second|2 seconds|31 seconds|

> It is important to note that Azure Mission-Critical considers availability to be more than simple uptime, but rather a consistent level of application service relative to a known healthy application state which is captured by a codified health model.

The pursuit of a particular reliability tier ultimately has a significant bearing on the critical design path and encompassed design decisions, resulting in a different target architecture.

The image below demonstrates how the different reliability tiers and underlying business requirements influence the target architecture for the foundational reference implementation, particularly concerning the number of regional deployments and utilised global technologies.

[![Azure Mission-Critical Reliability Tiers](/docs/media/reliability-tiers.png "Azure Mission-Critical Reliability Tiers")](./README.md)

[![Azure Mission-Critical Availability Targets](/docs/media/SLO-visualization.gif "Azure Mission-Critical Availability Targets")](./README.md)

### Opportunity cost

There is a opportunity cost associated with achieving a Mission-Critical application design since it requires significant engineering investment in fundamental reliability concepts, such as fully embracing Infrastructure as Code, deployment automation, and chaos engineering. This comes at a cost in terms of time/effort which could be invested elsewhere to deliver new application functionality and features.

Furthermore, maximizing reliability with a Mission-Critical application design can also have a significant bearing on financial costs, primarily through the duplication of resources and the distribution of resources across regions to achieve high availability. To avoid excess costs, it is highly recommended that **Azure Mission-Critical solutions not be over-engineered/over-optimized/over-provisioned beyond relevant business requirements**.

## Synthetic application construction

In parallel to design activities, it is highly recommended that a synthetic Mission-Critical application environment be established using the [foundational-online](https://github.com/Azure/Mission-Critical-Online) and [foundational-connected](https://github.com/Azure/AlwaysOn-Foundational-Connected) reference implementations.

This provides hands-on opportunities to validate design decisions by replicating the target architecture, allowing for design uncertainty to be swiftly assessed. If applied correctly with representative requirement coverage, most problematic issues likely to hinder progress can be uncovered and subsequently addressed.

## Target architecture evolution

Application architectures established using the Azure Mission-Critical design methodology must continue to evolve in alignment with Azure platform roadmaps to support optimized sustainability.

---

|Previous Page|Next Page|
|--|--|
|[Introduction to Azure Mission-Critical](../introduction/README.md)|[Azure Mission-Critical Design Principles](./Principles.md)

---

|Design Methodology|
|--|
|[How to use the Azure Mission-Critical Design Methodology](./README.md)
|[Azure Mission-Critical Design Principles](./Principles.md)
|[Azure Mission-Critical Design Areas](./Design-Areas.md)
|[Application Design](./App-Design.md)
|[Application Platform](./App-Platform.md)
|[Data Platform](./Data-Platform.md)
|[Health Modeling and Observability](./Health-Modeling.md)
|[Deployment and Testing](./Deployment-Testing.md)
|[Networking and Connectivity](./Networking.md)
|[Security](./Security.md)
|[Operational Procedures](./Operational-Procedures.md)

---

[Azure Mission-Critical | Documentation Inventory](/docs/README.md)
