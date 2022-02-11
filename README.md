[![Azure AlwaysOn](./icon.png "Azure AlwaysOn")](./README.md)

## Welcome to Azure AlwaysOn

AlwaysOn is an open source project that provides a **prescriptive architectural approach to building highly-reliable cloud-native applications on Microsoft Azure for mission-critical workloads**. More specifically, this repository contains everything required to understand and implement an 'always on' application on Microsoft Azure, and is comprised of the following:

1. **Architectural Guidelines**: cloud-native design methodology to guide readers through the architectural process of building a mature mission-critical application on Microsoft Azure, articulating key design considerations and requisite design decisions along with associated trade-offs.

2. **Fully Functional Reference Implementations**: end-to-end reference implementations intended to provide a solution orientated basis to showcase mission-critical application development on Microsoft Azure, leveraging Azure-native platform capabilities to maximize reliability and operational effectiveness.
    - Design and implementation guidance to help readers understand and use the AlwaysOn design methodology in the context of a particular scenario.
    - Production-ready technical artifacts including Infrastructure-as-Code (IaC) resources and Continuous-Integration/Continuous-Deployment (CI/CD) pipelines (GitHub and Azure DevOps) to deploy an AlwaysOn application with mature end-to-end operational wrappers.

## AlwaysOn | Navigation

- [Introduction | What is AlwaysOn?](./docs/introduction/README.md) - Detailed introduction into AlwaysOn, the problem it is intended to solve and the value it can provide.

- [Design Methodology | AlwaysOn Architectural Approach](./docs/design-methodology/README.md) - Prescriptive guidance aligned to 8 critical design areas guides users to design and build an AlwaysOn application, outlining a recommended decision process.

- [Foundational Reference Implementation | Online](https://github.com/azure/alwayson-foundational-online) - Everything required to understand and build a copy of the foundational reference implementation intended for online scenarios that are public-facing and do not require private network connectivity to a surrounding organizational technical estate.

- [Foundational Reference Implementation | Connected](https://github.com/azure/alwayson-foundational-connected) - Everything required to understand and build a copy of the foundational reference implementation intended for private scenarios that require integration with an organizational technical estate for either public-facing or internal-facing workloads.

## Helpful Information

The foundational reference implementations are separated within dedicated repositories containing all relevant documentation and technical artifacts, along with a *getting started guide*:
  - [Foundational-Online Reference Implementation](https://github.com/Azure/AlwaysOn-foundational-online)
  - [Foundational-Connected Reference Implementation](https://github.com/Azure/AlwaysOn-foundational-connected)

[![AlwaysOn Repo Structure](/docs/media/alwayson-repo-structure.png "AlwaysOn Repo Structure")](./CONTRIBUTE.md)

> A list of [Frequently Asked Questions](./docs/FAQ.md) is provided to capture common issues and challenges associated with using the AlwaysOn project.

## Contributing

AlwaysOn is a community driven open source project that welcomes contributions as well as suggestions. Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us the rights to use your contribution. For details, visit the [CLA portal](https://cla.opensource.microsoft.com).

When you submit a pull request, a CLA bot will automatically determine whether you need to provide a CLA and decorate the PR appropriately (e.g. status check, comment). Simply follow the instructions provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

For more details, please read [how to contribute](./CONTRIBUTE.md).

## Microsoft Sponsorship

The AlwaysOn project was created by the **Microsoft Customer Architecture Team (CAT)** who continue to actively sponsor the sustained evolution of the AlwaysOn project through the creation of additional reference implementations for common industry scenarios.
