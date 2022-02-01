# AlwaysOn - Full List of Documentation

## AlwaysOn Landing Page

- [Landing Page](/README.md)

## Introduction to AlwaysOn

- [Introduction](./introduction/README.md)

## AlwaysOn Design Methodology

- [How to use the AlwaysOn Design Methodology](./design-methodology/README.md)
- [Design Principles](./design-methodology/Principles.md)
- [Design Areas](./design-methodology/Design-Areas.md)
  - [Reference Architecture](./design-methodology/Design-Areas.md#reference-architecture)
  - [Cross Cutting Concerns](./design-methodology/Design-Areas.md#cross-cutting-concerns)
  - [Application Design](./design-methodology/App-Design.md)
  - [Application Platform](./design-methodology/App-Platform.md)
  - [Data Platform](./design-methodology/Data-Platform.md)
  - [Health Modeling and Observability](./design-design-methodology/Health-Modeling.md)
  - [Deployment and Testing](./design-design-methodology/Deployment-Testing.md)
  - [Networking and Connectivity](./design-design-methodology/Networking.md)
  - [Security](./design-design-methodology/Security.md)
  - [Operational Procedures](./design-methodology/Operational-Procedures.md)

---

## Documentation Conventions

- Overarching topics concerning the AlwaysOn architecture, design principles, design decisions, and cross-component integration are documented as separate markdown documents within the `/docs/` folder.

- Each source code component for the reference implementation has it's own `README.md` file which explains how that particular component works, how it is supposed to be used, and how it may interact with other aspects of the AlwaysOn solution.
  - Within the `main` branch, each `README.md` file must accurately represent the state of the associated component which will serve as a core aspect of PR reviews. Any modifications to source components must therefore be reflected in the documentation as well.
