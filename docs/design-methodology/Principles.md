# AlwaysOn Design Principles

The AlwaysOn architectural framework presented within this repository is underpinned by 5 key design principles which serve as a compass for subsequent design decisions across technical domains and the critical design areas. Readers are strongly advised to familiarize themselves with these principles to better understand their impact and the trade-offs associated with non-adherence.

1. **Maximum Reliability** - Fundamental pursuit of the most reliable solution, ensuring trade-offs are properly understood.
1. **Sustainable Performance and Scalability** - Design for scalability across the end-to-end solution without performance bottlenecks.
1. **Operations by Design** - Engineered to last with robust and assertive operational management.
1. **Cloud-Native Design** - Focus on using native platforms services to minimize operational burdens, while mitigating known gaps.
1. **Always Secure** - Design for end-to-end security to maintain application stability and ensure availability.

[![AlwaysOn Design Principles](/docs/media/alwayson-design-principles.png "AlwaysOn Design Principles")](./Principles.md)

## Maximum Reliability

- **Design for failure** - Failure is impossible to avoid in a highly distributed multi-tenant cloud environment like Azure. By anticipating failures and cascading or correlated impact, from individual components to entire Azure regions, a solution can be designed and developed in a resilient manner.

- **Observe application health** - Before issues impacting application reliability can be mitigated, they must first be detected. By monitoring the operation of an application relative to a known healthy state it becomes possible to detect or even predict reliability issues, allowing for swift remedial action to be taken.

- **Drive automation** - One of the leading causes of application downtime is human error, whether that be due to the deployment of insufficiently tested software or misconfiguration. To minimize the possibility and impact of human errors, it is vital to strive for automation in all aspects of a cloud solution to improve reliability; automated testing, deployment, and management.

- **Design for self-healing** - Self healing describes a system's ability to deal with failures automatically through pre-defined remediation protocols connected to failure modes within the solution. It is an advanced concept that requires a high level of system maturity with monitoring and automation, but should be an aspiration from inception to maximize reliability.

## Sustainable Performance and Scalability

- **Design for scale-out** - Scale-out is a concept that focuses on a system's ability to respond to demand through horizontal growth. This means that as traffic grows, more resource units are added in parallel instead of increasing the size of the existing resources. A systems ability to handle expected and unexpected traffic increases through scale-units is essential to overall performance and reliability by further reducing the impact of a single resource failure.

- **Model capacity** - The system's expected performance under various load profiles should be modeled through load and performance tests. This capacity model enables planning of resource scale levels for a given load profile, and additionally exposes how system components perform in relation to each other, therefore enabling system-wide capacity allocation planning.

- **Test and experiment often** - Testing should be performed for each major change as well as on a regular basis. Such testing should be performed in testing and staging environments, but it can also be beneficial to run a subset of tests against the production environment. Ongoing testing validates existing thresholds, targets and assumptions and will help to quickly identify risks to resiliency and availability.

- **Baseline performance and identify bottlenecks** - Performance testing with detailed telemetry from every system component allows for the identification of bottlenecks within the system, including components which need to be scaled in relation to other components, and this information should be incorporated into the capacity model.

- **Use Containerized or serverless architecture** - Using managed compute services and containerized architectures significantly reduces the ongoing administrative and operational overhead of designing, operating, and scaling applications by shifting infrastructure deployment and maintenance to the managed service provider.

## Operations by Design

- **Loosely coupled components** - Loose coupling enables independent and on-demand testing, deployments, and updates to components of the application while minimizing inter-team dependencies for support, services, resources, or approvals.

- **Optimize build and release process** - Fully automated build and release processes reduce the friction and increase the velocity of deploying updates, bringing repeatability and consistency across environments. Automation shortens the feedback loop from developers pushing changes to getting automated near instantaneous insights on code quality, test coverage, security, and performance, which increases developer productivity and team velocity.

- **Understand operational health** - Full diagnostic instrumentation of all components and resources enables ongoing observability of logs, metrics and traces, and enables health modeling to quantify application health in the context to availability and performance requirements.

- **Rehearse recovery and practice failure** - Business Continuity (BC) and Disaster Recovery (DR) planning and practice drills are essential and should be conducted periodically, since learnings from drills can iteratively improve plans and procedures to maximize resiliency in the event of unplanned downtime.

- **Embrace continuous operational improvement** - Prioritize routine improvement of the system and user experience, leveraging a health model to understand and measure operational efficiency with feedback mechanisms to enable application teams to understand and address gaps in an iterative manner.

## Cloud Native Design

- **Azure-native managed services** - Azure-native managed services are prioritized due to their lower administrative and operational overhead as well as tight integration with consistent configuration and instrumentation across the application stack.

- **Roadmap alignment** - Incorporate upcoming new and improved Azure service capabilities as they become Generally Available (GA) to stay close to the leading edge of Azure.

- **Embrace preview capabilities and mitigate known gaps** - While Generally Available (GA) services are prioritized for supportability, Azure service previews are actively explored for rapid incorporation, providing technical and actionable feedback to Azure product groups to address gaps.

- **Landing Zone alignment** - Deployable within an [Azure Landing Zone](https://docs.microsoft.com/azure/cloud-adoption-framework/ready/landing-zone/) and aligned to the Azure Landing Zone design methodology, but also fully functional and  deployable in a bare environment outside of a Landing Zone.

## Always Secure

- **Monitor the security of the entire solution and plan incident responses** - Correlate security and audit events to model application health and identify active threats. Establish automated and manual procedures to respond to incidents leveraging Security Information and Event Management (SIEM) tooling for tracking.

- **Model and test against potential threats** - Ensure appropriate resource hardening and establish procedures to identify and mitigate known threats, using penetration testing to verify threat mitigation, as well as static code analysis and code scanning.

- **Identify and protect endpoints** - Monitor and protect the network integrity of internal and external endpoints through security capabilities and appliances, such as firewalls or web application firewalls. Use industry standard approaches to protect against common attack vectors like Distributed Denial-Of-Service (DDoS) attacks such as SlowLoris.

- **Protect against code level vulnerabilities** - Identify and mitigate code-level vulnerabilities, such as cross-site scripting or SQL injection, and incorporate security patching into operational lifecycles for all parts of the codebase, including dependencies.

- **Automate and use least privilege** - Drive automation to minimize the need for human interaction and implement least privilege across both the application and control plane to protect against data exfiltration and malicious actor scenarios.

- **Classify and encrypt data** - Classify data according to risk and apply industry standard encryption at rest and in transit, ensuring keys and certificates are stored securely and managed properly.

# Additional Project Principles

- **Production ready artifacts**: Every AlwaysOn technical artifact will be ready for use in production environments with all end-to-end operational aspects considered.

- **Rooted in 'customer truth'** - All technical decisions will be guided by the experience customers have on the platform and the feedback they share.

- **Azure roadmap alignment** - The AlwaysOn architecture will have its own roadmap that is aligned with Azure product roadmaps.

---

|Previous Page|Next Page|
|--|--|
|[How to use the AlwaysOn Design Guidelines](./README.md)|[AlwaysOn Design Areas](./Design-Areas.md)

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
