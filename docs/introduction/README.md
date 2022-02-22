# AlwaysOn Introduction

Getting started on Microsoft Azure is now easier than ever, however, building mission-critical solutions that are highly reliable on the platform remains a challenge for three main reasons:

- Designing a reliable application at scale is complex and requires extensive platform knowledge to select the right technologies and optimally configure them as an end-to-end solution.

- Failure is inevitable in any complex distributed system, and the solution must therefore be architected to handle failures and correlated or cascading impact, which is a change in mindset for many developers and architects entering the cloud from an on-premises environment; reliability engineering is no longer an infrastructure topic, but should be a first-class concern within the application development process.

- Operationalizing mission-critical applications requires a high degree of engineering rigor and maturity throughout the end-to-end engineering lifecycle as well as the ability to learn from failure.

AlwaysOn strives to address the challenge of building mission-critical applications on Azure, leveraging lessons from numerous customer applications and first-party solutions to provide actionable and authoritative guidance that applies [Well-Architected](https://docs.microsoft.com/azure/architecture/framework/) best practices as the technical foundation for building and operating a highly reliable solution on Azure at-scale.

More specifically, AlwaysOn provides a design methodology to guide readers through the design process of building a highly reliable cloud-native application on Azure, explaining key design considerations and requisite design decisions along with associated trade-offs. Additionally, AlwaysOn provides a gallery of fully functional production-ready reference implementations aligned to common industry scenarios, which can serve as a basis for further solution development.

## What is AlwaysOn?

AlwaysOn is an open source architectural approach to building highly-reliable cloud-native applications on Microsoft Azure for mission-critical applications.

The 'AlwaysOn' project name refers to the highly-reliable and mission-critical nature of the architectural pattern it represents, where for given set of business requirements, an application should always be operational and available.

Because of the focus on reliability, the AlwaysOn design methodology presented within this repository adopts a globally distributed and highly scalable approach to building applications on Azure. However, this globally distributed approach to achieve high reliability comes at a development cost which may not be justifiable for every workload scenario. It is therefore strongly advocated that design decisions are driven by business requirements but informed by the opinionated guidance provided within this section.

## What Problem Does AlwaysOn Solve?

Building mission-critical applications on any hyper-scale cloud platform requires significant technical expertise and engineering investment to appropriately select and piece together services and features. This complexity often leads to a sub-optimal solution, particularly given the typical prioritization of business needs over platform fundamentals and the struggle of aligning with evolving best practices.

The AlwaysOn project strives to address this complex consumption experience for Microsoft Azure, by applying [Well-Architected](https://docs.microsoft.com/azure/architecture/framework/) best practices to mission-critical application scenarios, providing prescriptive and opinionated technical guidance alongside streamlined consumption mechanisms for common industry patterns through reference implementations; turn-key solutions that are implicitly aligned with Well-Architected best practices.

## What Does AlwaysOn Provide?

1. **Architectural Guidelines**: cloud-native design methodology to guide readers through the architectural process of building a mature mission-critical application on Microsoft Azure, articulating key design considerations and requisite design decisions along with associated trade-offs.

2. **Fully Functional Reference Implementations**: end-to-end reference implementations intended to provide a solution orientated basis to showcase mission-critical application development on Microsoft Azure, leveraging Azure-native platform capabilities to maximize reliability and operational effectiveness.
    - Design and implementation guidance to help readers understand and use the AlwaysOn design methodology in the context of a particular scenario.
    - Production-ready technical artifacts including Infrastructure-as-Code (IaC) resources and Continuous-Integration/Continuous-Deployment (CI/CD) pipelines (GitHub and Azure DevOps) to deploy an AlwaysOn application with mature end-to-end operational wrappers.

*Important Note: AlwaysOn will continue to develop additional reference implementations for common industry scenarios, with several implementations currently under development.*

---

|Previous Page|Next Page|
|--|--|
|[Home](/README.md)|[How to use the AlwaysOn Design Methodology](../design-methodology/README.md)

---

[AlwaysOn | Documentation Inventory](/docs/README.md)
