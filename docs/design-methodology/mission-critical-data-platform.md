# Data platform considerations for mission-critical workloads on Azure

The selection of an effective application data platform is a further crucial decision area, which has far-reaching implications across other design areas. Azure ultimately offers a multitude of relational, non-relational, and analytical data platforms, which differ greatly in capability. It's therefore essential that key non-functional requirements be fully considered alongside other decision factors such as consistency, operability, cost, and complexity. For example, the ability to operate in a multi-write configuration will have a critical bearing on suitability for a globally available platform.

This design area expands on [application design](mission-critical-application-design.md), providing key considerations and recommendations to inform the selection of an optimal data platform.

## The Four Vs of Big Data

The 'Four Vs of Big Data' provide a framework to better understand requisite characteristics for a highly available data platform, and how data can be used to maximize business value. This section will therefore explore how the Volume, Velocity, Variety, and Veracity characteristics can be applied at a conceptual level to help design a data platform using appropriate data technologies.

- **V**olume: how much data is coming in to inform storage capacity and tiering requirements -that is the size of the dataset.
- **V**elocity: the speed at which data is processed, either as batches or continuous streams -that is the rate of flow.
- **V**ariety: the organization and format of data, capturing structured, semi-structured, and unstructured formats -that is data across multiple stores or types.
- **V**eracity: includes the provenance and curation of considered data sets for governance and data quality assurance -that is accuracy of the data.

### Design Considerations

**Volume**

- Existing (if any) and expected future data volumes based on forecasted data growth rates aligned with business objectives and plans.
  - Data volume should encompass the data itself and indexes, logs, telemetry, and other applicable datasets.
  - Large business-critical and mission-critical applications typically generate and store high volumes (GB and TB) on a daily basis.
  - There can be significant cost implications associated with data expansion.

- Data volume may fluctuate due to changing business circumstances or housekeeping procedures.

- Data volume can have a profound impact on data platform query performance.

- There can be a profound impact associated with reaching data platform volume limits.
  - *Will it result in downtime? and if so, for how long?*
  - *What are the mitigation procedures? and will mitigation require application changes?*
  - *Will there be a risk of data loss?*

- Features such as Time to Live (TTL) can be used to manage data growth by automatically deleting records after an elapsed time, using either record creation or modification.
  - For example, Azure Cosmos DB provides an in-built [TTL](http://docs.microsoft.com/azure/cosmos-db/sql/time-to-live) capability.

**Velocity**

- The speed with which data is emitted from various application components, and the throughput requirements for how fast data needs to be committed and retrieved are critical to determining an optimal data technology for key workload scenarios.
  - The nature of throughput requirements will differ by workload scenario, such as those that are read-heavy or write-heavy.
    - For example, analytical workloads will typically need to cater to a large read throughput.
  - *What is the required throughput? And how is throughput expected to grow?*
  - *What are the data latency requirements at P50/P99 under reference load levels?*

- Capabilities such as supporting a lock-free design, index tuning, and consistency policies are critical to achieving high-throughput.
  - Optimizing configuration for high throughput incurs trade-offs, which should be fully understood.
  - Load-levelling persistence and messaging patterns, such as CQRS and Event Sourcing, can be used to further optimize throughput.

- Load levels will naturally fluctuate for many application scenarios, with natural peaks requiring a sufficient degree of elasticity to handle variable demand while maintaining throughput and latency.
  - Agile scalability is key to effectively support variable throughput and load levels without overprovisioning capacity levels.
    - Both read and write throughput must scale according to application requirements and load.
    - Both vertical and horizontal scale operations can be applied to respond to changing load levels.

- The impact of throughput dips can vary based on workload scenario.
  - *Will there be connectivity disruption?*
  - *Will individual operations return failure codes while the control plane continues to operate?*
  - *Will the data platform activate throttling, and if so for how long?*

- The fundamental application design recommendation to use an active-active geographical distribution introduces challenges around data consistency.
  - There's a trade-off between consistency and performance with regard to full ACID transactional semantics and traditional locking behavior.
    - Minimizing write latency will come at the cost of data consistency.

- In a multi-region write configuration, changes will need to be synchronized and merged between all replicas, with conflict resolution where required, and this may impact performance levels and scalability.

- Read-only replicas (intra-region and inter-region) can be used to minimize roundtrip latency and distributing traffic to boost performance, throughput, availability, and scalability.

- A caching layer can be used to increase read throughput to improve user experience and end-to-end client response times.
  - Cache expiration times and policies need to be considered to optimize data recentness.

**Variety**

- The data model, data types, data relationships, and intended query model will strongly affect data platform decisions.
  - *Does the application require a relational data model or can it support a variable-schema or non-relational data model?*
  - *How will the application query data? And will queries depend on database-layer concepts such as relational joins? Or does the application provide such semantics?*

- The nature of datasets considered by the application can be varied, from unstructured content such as images and videos, or more structured files such as CSV and Parquet.
  - Composite application workloads will typically have distinct datasets and associated requirements.

- In addition to relational or non-relational data platforms, graph or key-value data platforms may also be suitable for certain data workloads.
  - Some technologies cater to variable-schema data models, where data items are semantically similar and/or stored and queried together but differ structurally.

- In a microservice architecture, individual application services can be built with distinct scenario-optimized datastores rather than depending on a single monolithic datastore.
  - Design patterns such as [SAGA](http://docs.microsoft.com/azure/architecture/reference-architectures/saga/saga) can be applied to manage consistency and dependencies between different datastores.
    - Inter-database direct queries can impose co-location constraints.
  - The use of multiple data technologies will add a degree of management overhead to maintain encompassed technologies.

- The feature-sets for each Azure service differ across languages, SDKs, and APIs, which can greatly impact the level of configuration tuning that can be applied.

- Capabilities for optimized alignment with the data model and encompassed data types will strongly influence data platform decisions.
  - Query layers such as stored procedures and object-relational mappers.
  - Language-neutral query capability, such as a secured REST API layer.
  - Business continuity capabilities, such as backup and restore.

- Analytical datastores typically support polyglot storage for various types of data structures.
  - Analytical runtime environments, such as Apache Spark, may have integration restrictions to analyze polyglot data structures.

- In an enterprise context, the use of existing processes and tooling, and the continuity of skills, can have a significant bearing on the data platform design and selection of data technologies.

**Veracity**

- Several factors must be considered to validate the accuracy of data within an application, and the management of these factors can have a significant bearing on the design of the data platform.
  - Data consistency.
  - Platform security features.
  - Data governance.
  - Change management and schema evolution.
  - Dependencies between datasets.

- In any distributed application with multiple data replicas there's a trade-off between consistency and latency, as expressed in the [CAP](https://en.wikipedia.org/wiki/CAP_theorem) and [PACELC](https://en.wikipedia.org/wiki/PACELC_theorem) theorems.
  - When readers and writers are distinctly distributed, an application must choose to return either the fastest-available version of a data item, even if it out of date compared to a just-completed write (update) of that data item in another replica, or the most up-to-date version of the data item, which may incur additional latency to determine and obtain the latest state.
  - Consistency and availability can be configured at platform level or at individual data request level.
  - *What is the user experience if data was to be served from a replica closest to the user which doesn't reflect the most recent state of a different replica? i.e. can the application support possibly serving out-of-date data?*

- In a multi-region write context, when the same data item is changed in two separate write-replicas before either change can be replicated, a conflict is created which must be resolved.
  - Standardized conflict resolution policies, such as "Last One Wins", or a custom strategy with custom logic can be applied.

- The implementation of security requirements may adversely impact throughput or performance.

- Encryption at-rest can be implemented in the application layer using client-side encryption and/or the data layer using server-side encryption if necessary.

- Azure supports various [encryption models](http://docs.microsoft.com/azure/security/fundamentals/encryption-overview), including server-side encryption that uses service-managed keys, customer-managed keys in Key Vault, or customer-managed keys on customer-controlled hardware.
  - With client-side encryption, keys can be managed in Key Vault or another secure location.

- MACsec (IEEE 802.1AE MAC) data-link encryption is used to secure all traffic moving between Azure datacenters on the Microsoft backbone network.
  - Packets are encrypted and decrypted on the devices before being sent, preventing physical 'man-in-the-middle' or snooping/wiretapping attacks.

- Authentication and authorization to the data plane and control plane.
  - *How will the data platform authenticate and authorize application access and operational access?*

- Observability through monitoring platform health and data access.
  - *How will alerting be applied for conditions outside acceptable operational boundaries?*

### Design Recommendations

**Volume**

- Ensure future data volumes associated with organic growth aren't expected to exceed data platform capabilities.
  - Forecast data growth rates aligned to business plans and use established rates to inform ongoing capacity requirements.
  - Compare aggregate and per-data record volumes against data platform limits.
  - If there's a risk of limits being reached in exceptional circumstances, ensure operational mitigations are in place to prevent downtime and data loss.

- Monitor data volume and validate it against a capacity model, considering scale limits and expected data growth rates.

- Define application data tiers to classify datasets based on usage and criticality to facilitate the removal or offloading of older data.
  - Consider classifying datasets into 'hot', 'warm', and 'cold' ('archive') tiers.
    - For example, the foundational reference implementations use Cosmos DB to store 'hot' data that is actively used by the application, while Azure Storage is used for 'cold' operations data for analytical purposes.

- Configure housekeeping procedures to optimize data growth and drive data efficiencies, such as query performance, and managing data expansion.
  - Configure Time-To-Live (TTL) expiration for data that is no-longer required and has no long-term analytical value.
    - Validate that old data can be safely tiered to secondary storage, or deleted outright, without an adverse impact to the application.
  - Offload non-critical data to secondary cold storage, but maintain it for analytical value and to satisfy audit requirements.
  - Collect data platform telemetry and usage statistics to enable DevOps teams to continually evaluate housekeeping requirements and 'right-size' datastores.

- In-line with a microservice application design, consider the use of multiple different data technologies in-parallel, with optimized data solutions for specific workload scenarios and volume requirements.
  - Avoid creating a single monolithic datastore where data volume from expansion can be hard to manage.

**Velocity**

- The data platform must inherently be designed and configured to support high-throughput, with workloads separated into different contexts to maximize performance using scenario optimized data solutions.
  - Ensure read and write throughput for each data scenario can scale according to expected load patterns, with sufficient tolerance for unexpected variance.
  - Separate different data workloads, such as transactional and analytical operations, into distinct performance contexts.

- Load-level through the use of asynchronous non-blocking messaging, for example using the [CQRS](http://docs.microsoft.com/azure/architecture/patterns/cqrs) or [Event Sourcing](http://docs.microsoft.com/azure/architecture/patterns/event-sourcing) patterns.
  - There might be latency between write requests and when new data becomes available to read, which may have an impact on the user experience.
    - This impact must be understood and acceptable in the context of key business requirements.

- Ensure agile scalability to support variable throughput and load levels.
  - If load levels are highly volatile, consider overprovisioning capacity levels to ensure throughput and performance is maintained.
  - Test and validate the impact to composite application workloads when throughput can't be maintained.

- Prioritize Azure-native data services with automated scale-operations to facilitate a swift response to load-level volatility.
  - Configure autoscaling based on service-internal and application-set thresholds.
  - Scaling should initiate and complete in timeframes consistent with business requirements.
  - For scenarios where manual interaction is necessary, establish automated operational 'playbooks' that can be triggered rather than conducting manual operational actions.
    - Consider whether automated triggers can be applied as part of subsequent engineering investments.

- Monitor application data read and write throughput against P50/P99 latency requirements and align to an application capacity model.

- Excess throughput should be gracefully handled by the data platform or application layer and captured by the health model for operational representation.

- Implement caching for 'hot' data scenarios to minimize response times.
  - Apply appropriate policies for cache expiration and house-keeping to avoid runaway data growth.
    - Expire cache items when the backing data changes.
    - If cache expiration is strictly Time-To-Live (TTL) based, the impact and customer experience of serving outdated data needs to be understood.

**Variety**

- In alignment with the principle of a cloud- and Azure-native design, it's highly recommended to prioritize managed Azure services to reduce operational and management complexity, and taking advantage of Microsoft's future platform investments.

- In alignment with the application design principle of loosely coupled microservice architectures, allow individual services to use distinct data stores and scenario-optimized data technologies.
  - Identify the types of data structure the application will handle for specific workload scenarios.
  - Avoid creating a dependency on a single monolithic datastore.
    - Consider the [SAGA](http://docs.microsoft.com/azure/architecture/reference-architectures/saga/saga) design pattern where dependencies between datastores exist.

- Validate that required capabilities are available for selected data technologies.
  - Ensure support for required languages and SDK capabilities. Not every capability is available for every language/SDK in the same fashion.

- Validate technology scale-limits and define scale-units to align with expected growth rates.
  - Ensure scale operations align with storage, performance, and consistency requirements.
  - When a new scale-unit's introduced, underlying data may need to be replicated which will take time and likely introduce a performance penalty while replication occurs. So ensure these operations are performed outside of critical business hours if possible.

**Veracity**

- Adopt a multi-region data platform design and distribute replicas across regions for maximum reliability, availability, and performance by moving data closer to application endpoints.
  - Distribute data replicas across Availability Zones (AZs) within a region (or use zone-redundant service tiers) to maximize intra-region availability.

- Where consistency requirements allow for it, use a multi-region write data platform design to maximize overall global availability and reliability.
  - Consider business requirements for conflict resolution when the same data item is changed in two separate write replicas before either change can be replicated and thus creating a conflict.
    - Use standardized conflict resolution policies such as "Last one wins" where possible
      - If a custom strategy with custom logic is required, ensure CI/CD DevOps practices are applied to manage custom logic.

- Test and validate backup and restore capabilities, and failover operations through chaos testing within continuous delivery processes.

- Run performance benchmarks to ensure throughput and performance requirements aren't impacted by the inclusion of required security capabilities, such as encryption.
  - Ensure continuous delivery processes consider load testing against known performance benchmarks.

- When applying encryption, it's strongly recommended to use service-managed encryption keys as a way of reducing management complexity.
  - If there's a specific security requirement for customer-managed keys, ensure appropriate key management procedures are applied to ensure availability, backup, and rotation of all considered keys.

> [!NOTE]
> When integrating with a broader organizational implementation, it's critical that an application centric approach be applied for the provisioning and operation of data platform components in an application design.
>
> More specifically, to maximize reliability it's critical that individual data platform components appropriately respond to application health through operational actions which may include other application components. For example, in a scenario where additional data platform resources are needed, scaling the data platform along with other application components according to a capacity model will likely be required, potentially through the provision of additional scale units. This approach will ultimately be constrained if there's a hard dependency of a centralized operations team to address issues related to the data platform in isolation.
>
> Ultimately, the use of centralized data services (that is Central IT DBaaS) introduces operational bottlenecks that significantly hinder agility through a largely uncontextualized management experience, and should be avoided in a mission-critical or business-critical context.

### Additional references

Additional data-platform guidance is available within the Azure Application Architecture Guide.

- [Azure Data Store Decision Tree](http://docs.microsoft.com/azure/architecture/guide/technology-choices/data-store-decision-tree)
- [Criteria for choosing a Data Store](http://docs.microsoft.com/azure/architecture/guide/technology-choices/data-store-considerations)
- [Non-Relational Data Stores](http://docs.microsoft.com/azure/architecture/data-guide/big-data/non-relational-data)
- [Relational OLTP Data Stores](http://docs.microsoft.com/azure/architecture/data-guide/relational-data/online-transaction-processing)

## Globally distributed multi-write datastore

To fully accommodate the globally distributed active-active aspirations of an application design, it's strongly recommended to consider a distributed multi-write data platform, where changes to separate writeable replicas are synchronized and merged between all replicas, with conflict resolution where required.

>[!IMPORTANT]
> The microservices may not all require a distributed multi-write datastore, so consideration should be given to the architectural context and business requirements of each workload scenario.

Azure Cosmos DB provides a globally distributed and highly available NoSQL datastore, offering multi-region writes and tunable consistency out-of-the-box. The design considerations and recommendations within this section will therefore focus on optimal Cosmos DB usage.

### Design considerations

**Azure Cosmos DB**

- Azure Cosmos DB stores data within Containers, which are indexed, row-based transactional stores designed to allow fast transactional reads and writes with response times on the order of milliseconds.

- Cosmos DB supports multiple different APIs with differing feature sets, such as SQL, Cassandra, and Mongo DB.
  - The first-party SQL API provides the richest feature set and is typically the API where new capabilities will become available first.

- Cosmos DB supports [Gateway and Direct connectivity modes](http://docs.microsoft.com/azure/cosmos-db/sql-sdk-connection-modes), where Direct facilitates connectivity over TCP to backend Cosmos DB replica nodes for improved performance with fewer network hops, while Gateway provides HTTPS connectivity to frontend gateway nodes.
  - Direct mode is only available when using the SQL API and is currently only supported on .NET and Java SDK platforms.

- Within Availability Zone enabled regions, Cosmos DB offers [Availability Zone (AZ) redundancy](http://docs.microsoft.com/azure/cosmos-db/high-availability#availability-zone-support) support for high availability and resiliency to zonal failures within a region.

- Cosmos DB maintains four replicas of data within a single region, and when Availability Zone (AZ) redundancy is enabled, Cosmos DB ensures data replicas are placed across multiple AZs to protect against zonal failures.
  - The Paxos consensus protocol is applied to achieve quorum across replicas within a region.

- A Cosmos DB account can easily be configured to replicate data across multiple regions to mitigate the risk of a single region becoming unavailable.
  - Replication can be configured with either single-region writes or multi-region writes.
    - With single region writes, a primary 'hub' region is used to serve all writes and if this 'hub' region becomes unavailable, a failover operation must occur to promote another region as writable.
    - With multi-region writes, applications can write to any configured deployment region, which will replicate changes between all other regions. If a region is unavailable then the remaining regions will be used to serve write traffic.

- In a multi-region write configuration, [update (insert, replace, delete) conflicts](http://docs.microsoft.com/azure/cosmos-db/conflict-resolution-policies) can occur where writers concurrently update the same item in multiple regions.

- Cosmos DB provides two conflict resolution policies, which can be applied to automatically address conflicts.
  - Last Write Wins (LWW) applies a time-synchronization clock protocol using a system-defined timestamp `_ts` property as the conflict resolution path. If of a conflict the item with the highest value for the conflict resolution path becomes the winner, and if multiple items have the same numeric value then the system selects a winner so that all regions can converge to the same version of the committed item.
    - With delete conflicts, the deleted version always wins over either insert or replace conflicts regardless of conflict resolution path value.
    - Last Write Wins is the default conflict resolution policy.
    - When using the SQL API a custom numerical property, such as a custom timestamp definition, can be used for conflict resolution.
  - Custom resolution policies allow for application-defined semantics to reconcile conflicts using a registered merge stored procedure that is automatically invoked when conflicts are detected.
    - The system provides exactly once guarantee for the execution of a merge procedure as part of the commitment protocol.
    - A custom conflict resolution policy is only available with the SQL API and can only be set at container creation time.

- In a multi-region write configuration, there's a dependency on a single Cosmos DB 'hub' region to perform all conflict resolutions, with the Paxos consensus protocol applied to achieve quorum across replicas within the hub region.
  - The platform provides a message buffer for write conflicts within the hub region to load level and provide redundancy for transient faults.
    - The buffer is capable of storing a few minutes worth of write updates requiring consensus.

> The strategic direction of the Cosmos DB platform is to remove this single region dependency for conflict resolution in a multi-region write configuration, utilizing a 2-phase Paxos approach to attain quorum at a global level and within a region.

- The primary 'hub' region is determined by the first region Cosmos DB is configured within.
  - A priority ordering is configured for additional satellite deployment regions for failover purposes.

- The data model and partitioning across logical and physical partitions plays an important role in achieving optimal performance and availability.

- When deployed with a single write region, Cosmos DB can be configured for [automatic failover](http://docs.microsoft.com/azure/cosmos-db/autoscale-faq) based on a defined failover priority considering all read region replicas.

- The RTO provided by the Cosmos DB platform is ~10-15 minutes, capturing the elapsed time to perform a regional failover of the Cosmos DB service if a catastrophic disaster impacting the hub region.
  - This RTO is also relevant in a multi-region write context given the dependency on a single 'hub' region for conflict resolution.
    - If the 'hub' region becomes unavailable, writes made to other regions will fail after the message buffer fills since conflict resolution won't be able to occur until the service fails over and a new hub region is established.

> The strategic direction of the Cosmos DB platform is to reduce the RTO to ~5 minutes by allowing partition level failovers.

- Recovery Point Objectives (RPO) and Recovery Time Objectives (RTO) are configurable via consistency levels, with a trade-off between data durability and throughput.
  - Cosmos DB provides a minimum RTO of 0 for a relaxed consistency level with multi-region writes or an RPO of 0 for strong consistency with single-write region.

- Cosmos DB offers a [99.999% SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db/v1_3/) for both read and write availability for Database Accounts configured with multiple Azure regions as writable.
  - The SLA is represented by the Monthly Uptime Percentage, which is calculated as 100% - Average Error Rate.
  - The Average Error Rate is defined as the sum of Error Rates for each hour in the billing month divided by the total number of hours in the billing month, where the Error Rate is the total number of Failed Requests divided by Total Requests during a given one-hour interval.

- Cosmos DB offers a 99.99% SLA for throughput, consistency, availability, and latency for Database Accounts scoped to a single Azure region when configured with any of the five Consistency Levels.
  - A 99.99% SLA also applies to Database Accounts spanning multiple Azure regions configured with any of the four relaxed Consistency Levels.

- There are two types of throughput that can be provisioned in Cosmos DB, standard and [autoscale](http://docs.microsoft.com/azure/cosmos-db/provision-throughput-autoscale), which are measured using Request Units per second (RU/s).
  - Standard throughput allocates resources required to guarantee a specified RU/s value.
    - Standard is billed hourly for provisioned throughput.
  - Autoscale defines a maximum throughput value, and Cosmos DB will automatically scale up or down depending on application load, between the maximum throughput value and a minimum of 10% of the maximum throughput value.
    - Autoscale is billed hourly for the maximum throughput consumed.

- Static provisioned throughput with a variable workload may result in throttling errors, which will impact perceived application availability.
  - Autoscale protects against throttling errors by enabling Cosmos DB to scale up as needed, while maintaining cost protection by scaling back down when load decreases.

- When Cosmos DB is replicated across multiple regions, the provisioned Request Units (RUs) are billed per region.

- There's a significant cost delta between a multi-region-write and single-region-write configuration which in many cases may make a multi-master Cosmos DB data platform cost prohibitive.

| Single Region Read/Write | Single Region Write - Dual Region Read | Dual Region Read/Write |
|---|---|---|
| 1 RU | 2 RU | 4 RU |

> The delta between single-region-write and multi-region-write is actually less than the 1:2 ratio reflected in the table above. More specifically, there's a cross-region data transfer charge associated with write updates in a single-write configuration, which isn't captured within the RU costs as with the multi-write configuration.  

- Consumed storage is billed as a flat rate for the total amount of storage (GB) consumed to host data and indexes for a given hour.

- `Session` is the default and most widely used [consistency level](http://docs.microsoft.com/azure/cosmos-db/consistency-levels) since data is received in the same order as writes.

- Cosmos DB supports authentication via either an Azure Active Directory identity or Cosmos DB keys and resource tokens, which provide overlapping capabilities.

![Cosmos DB Access Capabilities](http://docs.microsoft.com/azure/cosmos-db/media/how-to-restrict-user-data/operations.png "Cosmos DB Access Capabilities")

- It's possible to disable resource management operations using keys or resource tokens to limit keys and resource tokens to data operations only, allowing for fine-grained resource access control using Azure Active Directory Role-Based Access Control (RBAC).
  - Restricting control plane access via keys or resource tokens will disable control plane operations for clients using Cosmos DB SDKs and should therefore be thoroughly [evaluated and tested](http://docs.microsoft.com/azure/cosmos-db/role-based-access-control#check-list-before-enabling).
  - The `disableKeyBasedMetadataWriteAccess` setting can be configured via [ARM Template](http://docs.microsoft.com/azure/cosmos-db/role-based-access-control#set-via-arm-template) IaC definitions, or via a [Built-In Azure Policy](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4750c32b-89c0-46af-bfcb-2e4541a818d5).

- Cosmos DB Azure Active Directory RBAC support applies to account and resource control plane management operations.
  - Application administrators can create role assignments for users, groups, service principals or managed identities to grant or deny access to resources and operations on Cosmos DB resources.
  - There are several [Built-in RBAC Roles](http://docs.microsoft.com/azure/cosmos-db/role-based-access-control#built-in-roles) available for role assignment, and [custom RBAC roles](http://docs.microsoft.com/azure/cosmos-db/role-based-access-control#custom-roles) can also be used to form specific [privilege combinations](http://docs.microsoft.com/azure/role-based-access-control/resource-provider-operations#microsoftdocumentdb).
    - [Cosmos DB Account Reader](http://docs.microsoft.com/azure/role-based-access-control/built-in-roles#cosmos-db-account-reader-role) enables read-only access to the Cosmos DB resource.
    - [DocumentDB Account Contributor](http://docs.microsoft.com/azure/role-based-access-control/built-in-roles#documentdb-account-contributor) enables management of Cosmos DB accounts including keys and role assignments, but doesn't enable data-plane access.
    - [Cosmos DB Operator](http://docs.microsoft.com/azure/role-based-access-control/built-in-roles#cosmos-db-operator), which is similar to DocumentDB Account Contributor, but doesn't provide the ability to manage keys or role assignments.

- Cosmos DB resources (accounts, databases, and containers) can be protected against incorrect modification or deletion using [Resource Locks](http://docs.microsoft.com/azure/cosmos-db/resource-locks).
  - Resource Locks can be set at the account, database, or container level.
  - A Resource Lock set at on a resource will be inherited by all child resources. For example, a Resource Lock set on the Cosmos DB account will be inherited by all databases and containers within the account.
  - Resource Locks **only** apply to control plane operations and do **not** prevent data plane operations, such as creating, changing, or deleting data.
  - If control plane access isn't restricted with `disableKeyBasedMetadataWriteAccess`, then clients will be able to perform control plane operations using account keys.

- The [Cosmos DB Change Feed](http://docs.microsoft.com/azure/cosmos-db/change-feed) provides a time-ordered feed of changes to data in a Cosmos DB Container.
  - The Change Feed only includes insert and update operations to the source Cosmos DB Container; it doesn't include deletes.

- The Change Feed can be used to maintain a separate data store from the primary Container used by the application, with ongoing updates to the target data store fed by the Change Feed from the source Container.
  - The Change Feed can be used to populate a secondary store for additional data platform redundancy or for subsequent analytical scenarios.

- If delete operations routinely affect the data within the source Container, then the store fed by the Change Feed will be inaccurate and unreflective of deleted data.
  - A [Soft Delete](http://docs.microsoft.com/azure/cosmos-db/sql/change-feed-design-patterns#deletes) pattern can be implemented so that data records are included in the Change Feed.
    - Instead of explicitly deleting data records, data records are _updated_ by setting a flag (e.g. `IsDeleted`) to indicate that the item is considered deleted.
    - Any target data store fed by the Change Feed will need to detect and process items with a deleted flag set to True; instead of storing soft-deleted data records, the _existing_ version of the data record in the target store will need to be deleted.
  - A short Time-To-Live (TTL) is typically used with the soft-delete pattern so that Cosmos DB automatically deletes expired data, but only after it's reflected within the Change Feed with the deleted flag set to True.
    - Accomplishes the original delete intent whilst also propagating the delete through the Change Feed.

- Cosmos DB can be configured as an [analytical store](http://docs.microsoft.com/azure/cosmos-db/analytical-store-introduction), which applies a column format for optimized analytical queries to address the complexity and latency challenges that occur with the traditional ETL pipelines.

- Azure Cosmos DB automatically backs up data at regular intervals without affecting the performance or availability, and without consuming RU/s.

- Cosmos DB can be configured according to two distinct backup modes.
  - [Periodic](http://docs.microsoft.com/azure/cosmos-db/configure-periodic-backup-restore) is the default backup mode for all accounts, where backups are taken at a periodic interval and the data is restored by creating a request with the support team.
    - The default periodic backup retention period is 8 hours and the default backup interval is fourhours, which means only the latest two backups are stored by default.
    - The backup interval and retention period are configurable within the account.
      - The maximum retention period extends to a month with a minimum backup interval of one hour.
      - A role assignment to the Azure "Cosmos DB Account Reader Role" is required to configure backup storage redundancy.
    - Two backup copies are included at no extra cost, but additional backups incur additional costs.
    - By default, periodic backups are stored within separate Geo-Redundant Storage (GRS) that isn't directly accessible.
      - Backup storage exists within the primary 'hub' region and is replicated to the paired region through underlying storage replication.
      - The redundancy configuration of the underlying backup storage account is configurable to [Zone-Redundant Storage or Locally-Redundant Storage](http://docs.microsoft.com/azure/cosmos-db/configure-periodic-backup-restore#backup-storage-redundancy).
    - Performing a **restore operation requires a [Support Request](http://docs.microsoft.com/azure/cosmos-db/configure-periodic-backup-restore#request-restore)** since customers can't directly perform a restore.
      - Before opening a support ticket, the backup retention period should be increased to at least seven days within eight hours of the data loss event.
    - A restore operation creates a new Cosmos DB account where data is recovered.
      - An existing Cosmos DB account can't be used for Restore
      - By default, a new Cosmos DB account named `<Azure_Cosmos_account_original_name>-restored<n>` will be used.
        - This name can be adjusted, such as by reusing the existing name if the original account was deleted.
    - If throughput is provisioned at the database level, backup and restore will happen at the database level
      - It's not possible to select a subset of containers to restore.
  - [Continuous](http://docs.microsoft.com/azure/cosmos-db/continuous-backup-restore-introduction) backup mode allows for a restore to any point of time within the last 30 days.
    - Restore operations can be performed to return to a specific point in time (PITR) with a one-second granularity.
    - The available window for restore operations is up to 30 days.
      - It's also possible to restore to the resource instantiation state.
    - Continuous backups are taken within every Azure region where the Cosmos DB account exists.
      - Continuous backups are stored within the same Azure region as each Cosmos DB replica, using Locally-Redundant Storage (LRS) or Zone Redundant Storage (ZRS) within regions that support Availability Zones.
    - A self-service restore can be performed using the [Azure portal](http://docs.microsoft.com/azure/cosmos-db/restore-account-continuous-backup#restore-account-portal) or IaC artifacts such as [ARM templates](http://docs.microsoft.com/azure/cosmos-db/restore-account-continuous-backup#restore-arm-template).
    - There are several [limitations](http://docs.microsoft.com/azure/cosmos-db/continuous-backup-restore-introduction#current-limitations) with Continuous Backup.
      - The continuous backup mode isn't currently available in a multi-region-write configuration.
      - Only SQL API and MongoDB API can be configured for Continuous backup at this time.
      - If a container has TTL configured, restored data that has exceeded its TTL may be _immediately deleted_
    - A restore operation creates a new Cosmos DB account for the point-in-time restore.
    - There's an [additional storage cost](http://docs.microsoft.com/azure/cosmos-db/continuous-backup-restore-introduction#continuous-backup-pricing) for Continuous backups and restore operations.

- Existing Cosmos DB accounts can be migrated from Periodic to Continuous, but not from Continuous to Periodic; migration is one-way and not reversible.

- Each Cosmos DB backup is composed of the data itself and configuration details for provisioned throughput, indexing policies, deployment region(s), and container TTL settings.
  - Backups don't contain [firewall settings](http://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts?tabs=json#ipaddressorrange-object), [virtual network access control lists](http://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts/privateendpointconnections), [private endpoint settings](http://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts/privateendpointconnections), [consistency settings](http://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts?tabs=json#consistencypolicy-object) (an account is restored with session consistency), [stored procedures](http://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts/sqldatabases/containers/storedprocedures), [triggers](http://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts/sqldatabases/containers/triggers), [UDFs](http://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts/sqldatabases/containers/userdefinedfunctions), or [multi-region settings](http://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts?tabs=json#Location).
    - Customers are responsible for redeploying capabilities and configuration settings. These aren't restored through Cosmos DB backup.
  - Azure Synapse Link analytical store data is also not included in Cosmos DB backups.

- It's possible to implement a custom backup and restore capability for scenarios where Periodic and Continuous approaches aren't a good fit.
  - A custom approach introduces significant costs and additional administrative overhead, which should be understood and carefully assessed.
    - Common restore scenarios should be modeled, such as the corruption or deletion of an account, database, container, on data item.
    - Housekeeping procedures should be implemented to prevent backup sprawl.
  - Azure Storage or an alternative data technology can be used, such an alternative Cosmos DB container.
    - Azure Storage and Cosmos DB provide native integrations with Azure services such as Azure Functions and Azure Data Factory.

- The Cosmos DB documentation denotes two potential options for implementing custom backups.
  - [Cosmos DB change feed](http://docs.microsoft.com/azure/cosmos-db/change-feed) to write data to a separate storage facility.
    - An [Azure Function](http://docs.microsoft.com/azure/cosmos-db/change-feed-functions) or equivalent application process uses the [Change Feed Processor](http://docs.microsoft.com/azure/cosmos-db/change-feed-processor) to bind to the change feed and process items into storage.
  - Both continuous or periodic (batched) custom backups can be implemented using the Change Feed.
  - The Cosmos DB change feed doesn't yet reflect deletes, so a soft-delete pattern must be applied using a boolean property and TTL.
    - This pattern won't be required when the change feed provides full-fidelity updates.
  - [Azure Data Factory Connector for Cosmos DB](http://docs.microsoft.com/azure/data-factory/connector-azure-cosmos-db) ([SQL API](http://docs.microsoft.com/azure/data-factory/connector-azure-cosmos-db) or [MongoDB API](http://docs.microsoft.com/azure/data-factory/connector-azure-cosmos-db-mongodb-api) connectors) to copy data.
    - Azure Data Factory (ADF) supports manual execution and [Schedule](http://docs.microsoft.com/azure/data-factory/concepts-pipeline-execution-triggers#schedule-trigger), [Tumbling window](http://docs.microsoft.com/azure/data-factory/concepts-pipeline-execution-triggers#tumbling-window-trigger), and [Event-based](http://docs.microsoft.com/azure/data-factory/concepts-pipeline-execution-triggers#event-based-trigger) triggers.
      - Provides support for both Storage and Event Grid.
    - ADF is primarily suitable for periodic custom backup implementations due to its batch-oriented orchestration.
      - It's less suitable for continuous backup implementations with frequent events due to the orchestration execution overhead.
    - ADF supports [Azure Private Link](http://docs.microsoft.com/azure/data-factory/data-factory-private-link) for high network security scenarios

> Azure Cosmos DB is used within the design of many Azure services, so a significant regional outage for Cosmos DB will have a cascading effect across various Azure services within that region. The precise impact to a particular service will heavily depend on how the underlying service design uses Cosmos DB.

### Design Recommendations

- In line with microservices application design approach, it's strongly recommended to have a separate datastore instance/type per microservice.
  - Separate analytical workloads from application workloads using different data technologies optimized for distinct performance, reliability, and scalability requirements.

**Azure Cosmos DB**

- Use Azure Cosmos DB as the primary data platform where requirements allow.

- For mission-critical workload scenarios, configure Cosmos DB with a write replica inside each deployment region to reduce latency and provide maximum redundancy.
  - Configure the application to [prioritize the use of the local Cosmos DB replica](http://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationregion) for writes and reads to optimize application load, performance, and regional RU/s consumption.
  - The multi-region-write configuration comes at a significant cost and should be prioritized only for workload scenarios requiring maximum reliability.

- For less-critical workload scenarios, prioritize the use of single-region-write configuration (when using Availability Zones) with globally distributed read replicas, since this offers a high level of data platform reliability (99.999% SLA for read-, 99.995% SLA for write-operations) at a more compelling price-point.
  - Configure the application to use the local Cosmos DB read replica to optimize read performance.

- Select an optimal 'hub' deployment region where conflict resolution will occur in a multi-region-write configuration, and all writes will be performed in a single-region-write configuration.
  - Consider distance relative to other deployment regions and associated latency in selecting a primary region, and requisite capabilities such as Availability Zones support.

- Configure Cosmos DB with [Availability Zone (AZ) redundancy](http://docs.microsoft.com/azure/cosmos-db/high-availability#availability-zone-support) in all deployment regions with AZ support, to ensure resiliency to zonal failures within a region.

- Use the Cosmos DB native SQL API since it offers the most comprehensive feature set, particularly where performance tuning is concerned.
  - Alternative APIs should primarily be considered for migration or compatibility scenarios.
    - When using alternative APIs, validate that required capabilities are available with the selected language and SDK to ensure optimal configuration and performance.

- Use the Direct connection mode to optimize network performance through direct TCP connectivity to backend Cosmos DB nodes, with a reduced number of network 'hops'.

> The Cosmos DB SLA is calculated by averaging failed requests, which may not directly align with a 99.999% reliability tier error budget. When designing for 99.999% SLO, it's therefore vital to plan for regional and multi-region Cosmos DB write unavailability, ensuring a fallback storage technology is positioned if a failure, such as a persisted message queue for subsequent replay.

- Define a partitioning strategy across both logical and physical partitions to optimize data distribution according to the data model.
  - Minimize cross-partition queries.
  - Iteratively [test and validate](http://docs.microsoft.com/azure/cosmos-db/how-to-model-partition-example) the partitioning strategy to ensure optimal performance.

- [Select an optimal partition key](http://docs.microsoft.com/azure/cosmos-db/partitioning-overview#choose-partitionkey).
  - The partition key can't be changed after it has been created within the collection.
  - The partition key should be a property value that doesn't change.
  - Select a partition key that has a high cardinality, with a wide range of possible values.
  - The partition key should spread RU consumption and data storage evenly across all logical partitions to ensure even RU consumption and storage distribution across  physical partitions.
  - Run read queries against the partitioned column to reduce RU consumption and latency.

- [Indexing](http://docs.microsoft.com/azure/cosmos-db/index-overview) is also crucial for performance, so ensure index exclusions are used to reduce RU/s and storage requirements.
  - Only index those fields that are needed for filtering within queries; design indexes for the most-used predicates.

- Leverage the built-in error handling, retry, and broader reliability capabilities of the [Cosmos DB SDK](http://docs.microsoft.com/azure/cosmos-db/sql/best-practice-dotnet#checklist).
  - Implement [retry logic](http://docs.microsoft.com/azure/architecture/best-practices/retry-service-specific#cosmos-db) within the SDK on clients.

- Use service-managed encryption keys to reduce management complexity.
  - If there's a specific security requirement for customer-managed keys, ensure appropriate key management procedures are applied, such as backup and rotation.

- Disable [Cosmos DB key based metadata write access](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F4750c32b-89c0-46af-bfcb-2e4541a818d5) by applying the built-in Azure Policy.

- Enable [Azure Monitor](http://docs.microsoft.com/azure/cosmos-db/monitor-cosmos-db) to gather key metrics and diagnostic logs, such as provisioned throughput (RU/s).
  - Route Azure Monitor operational data into a Log Analytics workspace dedicated to Cosmos DB and other global resources within the  application design.
  - Use Azure Monitor metrics to determine if application traffic patterns are suitable for autoscale.

- Evaluate application traffic patterns to select an optimal option for [provisioned throughput types](http://docs.microsoft.com/azure/cosmos-db/how-to-choose-offer).
  - Consider auto-scale provisioned throughput to automatically level-out workload demand.

- Evaluate Microsoft [performance tips for Cosmos DB](http://docs.microsoft.com/azure/cosmos-db/performance-tips) to optimize client-side and server-side configuration for improved latency and throughput.

- When using AKS as the compute platform: For query-intensive workloads, select an AKS node SKU that has accelerated networking enabled to reduce latency and CPU jitters.

- For single write region deployments, it's strongly recommended to configure Cosmos DB for [automatic failover](http://docs.microsoft.com/azure/cosmos-db/high-availability#multi-region-accounts-with-a-single-write-region-write-region-outage).

- Load-level through the use of asynchronous non-blocking messaging within system flows, which write updates to Cosmos DB.
  - Consider patterns such as [Command and Query Responsibility Segregation](http://docs.microsoft.com/azure/architecture/patterns/cqrs) and [Event Sourcing](http://docs.microsoft.com/azure/architecture/patterns/event-sourcing).

- Configure the Cosmos DB account for continuous backups to obtain a fine granularity of recovery points across the last 30 days.
  - Consider the use of Cosmos DB backups in scenarios where contained data or the Cosmos DB account is deleted or corrupted.
  - Avoid the use of a custom backup approach unless absolutely necessary.

- It's strongly recommended to practice recovery procedures on non-production resources and data, as part of standard business continuity operation preparation.

- Define IaC artifacts to re-establish configuration settings and capabilities of a Cosmos DB backup restore.

- Evaluate and apply the [Azure Security Baseline](http://docs.microsoft.com/dotnet/security/benchmark/azure/baselines/cosmos-db-security-baseline#backup-and-recovery) control guidance for Cosmos DB Backup and Recovery.
  - [BR-1: Ensure regular automated backups](http://docs.microsoft.com/dotnet/security/benchmark/azure/baselines/cosmos-db-security-baseline#br-1-ensure-regular-automated-backups)
  - [BR-3: Validate all backups including customer-managed keys](http://docs.microsoft.com/dotnet/security/benchmark/azure/baselines/cosmos-db-security-baseline#br-3-validate-all-backups-including-customer-managed-keys)
  - [BR-4, Mitigate risk of lost keys](http://docs.microsoft.com/dotnet/security/benchmark/azure/baselines/cosmos-db-security-baseline#br-4-mitigate-risk-of-lost-keys)

- For analytical workloads requiring multi-region availability, use the Cosmos DB Analytical Store, which applies a column format for optimized analytical queries.

## Relational data technologies

For scenarios with a highly relational data model or dependencies on existing relational technologies, the use of Azure Cosmos DB in a multi-write configuration might not be directly applicable. In such cases, it's vital that used relational technologies are designed and configured to uphold the multi-region active-active aspirations of an application design.

Azure provides many managed relational data platforms, including Azure SQL Database and Azure Database for common OSS relational solutions, including MySQL, PostgreSQL, and MariaDB. The design considerations and recommendations within this section will therefore focus on the optimal usage of Azure SQL Database and Azure Database OSS flavors to maximize reliability and global availability.

### Design considerations

- Whilst relational data technologies can be configured to easily scale read operations, writes are typically constrained to go through a single primary instance, which places a significant constraint on scalability and performance.

- [Sharding](http://docs.microsoft.com/azure/sql-database/sql-database-elastic-scale-introduction) can be applied to distribute data and processing across multiple identical structured databases, partitioning databases horizontally to navigate platform constraints.
  - For example, sharding is often applied in multi-tenant SaaS platforms to isolate groups of tenants into distinct data platform constructs.

**Azure SQL Database**

- Azure SQL Database provides a fully managed database engine that is always running on the latest stable version of the SQL Server database engine and underlying Operating System.
  - Provides intelligent features such as performance tuning, threat monitoring, and vulnerability assessments.

- Azure SQL Database provides built-in regional high availability and turnkey geo-replication to distribute read-replicas across Azure regions.
  - With geo-replication, secondary database replicas remain read-only until a failover is initiated.
  - Up to four secondaries are supported in the same or different regions.
  - Secondary replicas can also be used for read-only query access to optimize read performance.
  - Failover must be initiated manually but can be wrapped in automated operational procedures.

- Azure SQL Database provides [Auto Failover Groups](http://docs.microsoft.com/azure/azure-sql/database/auto-failover-group-overview?tabs=azure-powershell), which replicates databases to a secondary server and allows for transparent failover if a failure.
  - Auto-failover groups support geo-replication of all databases in the group to only one secondary server or instance in a different region.
  - Auto-failover groups aren't currently supported in the Hyperscale service tier.
  - Secondary databases can be used to offload read traffic.

- Premium or Business Critical service tier database replicas can be [distributed across Availability Zones](http://docs.microsoft.com/azure/azure-sql/database/high-availability-sla) at no extra cost.
  - The control ring is also duplicated across multiple zones as three gateway rings (GW).
    - The routing to a specific gateway ring is controlled by Azure Traffic Manager.
  - When using the Business Critical tier, zone redundant configuration is only available when the Gen5 compute hardware is selected.

- Azure SQL Database offers a baseline 99.99% availability SLA across all of its service tiers, but provides a higher 99.995% SLA for the Business Critical or Premium tiers in regions that support availability zones.
  - Azure SQL Database Business Critical or Premium tiers not configured for Zone Redundant Deployments have an availability SLA of 99.99%.

- When configured with geo-replication, the Azure SQL Database Business Critical tier provides a Recovery Time Objective (RTO) of 30 seconds for 100% of deployed hours.

- When configured with geo-replication, the Azure SQL Database Business Critical tier has a Recovery point Objective (RPO) of 5 seconds for 100% of deployed hours.

- Azure SQL Database Hyperscale tier, when configured with at least two replicas, has an availability SLA of 99.99%.

- Compute costs associated with Azure SQL Database can be reduced using a [Reservation Discount](http://docs.microsoft.com/azure/cost-management-billing/reservations/understand-reservation-charges).
  - It's not possible to apply reserved capacity for DTU-based databases.

- [Point-in-time restore](http://docs.microsoft.com/azure/azure-sql/database/recovery-using-backups#point-in-time-restore) can be used to return a database and contained data to an earlier point in time.

- [Geo-restore](http://docs.microsoft.com/azure/sql-database/sql-database-recovery-using-backups) can be used to recover a database from a geo-redundant backup.

**Azure Database For PostgreSQL**

- Azure Database For PostgreSQL is offered in three different deployment options:
  - Single Server, SLA 99.99%
  - Flexible Server, which offers Availability Zone redundancy, SLA 99.99%
  - Hyperscale (Citus), SLA 99.95% when High Availability mode is enabled.

- [Hyperscale (Citus)](http://docs.microsoft.com/azure/postgresql/tutorial-hyperscale-shard) provides dynamic scalability through sharding without application changes.
  - Distributing table rows across multiple PostgreSQL servers is key to ensure scalable queries in Hyperscale (Citus).
  - Multiple nodes can collectively hold more data than a traditional database, and in many cases can use worker CPUs in parallel to optimize costs.

- [Autoscale](https://techcommunity.microsoft.com/t5/azure-database-support-blog/how-to-auto-scale-an-azure-database-for-mysql-postgresql/ba-p/369177) can be configured through runbook automation to ensure elasticity in response to changing traffic patterns.

- Flexible server provides cost efficiencies for non-production workloads through the ability to stop/start the server, and a burstable compute tier that is suitable for workloads that don't require continuous compute capacity.

- There's no additional charge for backup storage for up to 100% of total provisioned server storage.
  - Additional consumption of backup storage is charged according to consumed GB/month.

- Compute costs associated with Azure Database for PostgreSQL can be reduced using a either a [Single Server Reservation Discount](http://docs.microsoft.com/azure/postgresql/concept-reserved-pricing) or [Hyperscale (Citus) Reservation Discount](http://docs.microsoft.com/azure/postgresql/concepts-hyperscale-reserved-pricing).

### Design Recommendations

- Flexible Server is recommended to use it for business critical workloads due to its Availability Zone support.

- When using Hyperscale (Citus) for business critical workloads, enable High Availability mode to receive the 99.95% SLA guarantee.

- Consider sharding to partition relational databases based on different application and data contexts, helping to navigate platform constraints, maximize scalability and availability, and fault isolation.
  - This recommendation is particularly prevalent when the application design considers three or more Azure regions since relational technology constraints can significantly hinder globally distributed data platforms.
  - Sharding isn't appropriate for all application scenarios, so a contextualized evaluation is required.

- Prioritize the use of Azure SQL Database where relational requirements exist due to its maturity on the Azure platform and wide array of reliability capabilities.

**Azure SQL Database**

- Use the Business-Critical service tier to maximize reliability and availability, including access to critical resiliency capabilities.

- Use the vCore based consumption model to facilitate the independent selection of compute and storage resources, tailored to workload volume and throughput requirements.
  - Ensure a defined capacity model is applied to inform compute and storage resource requirements.
    - Consider [Reserved Capacity](http://docs.microsoft.com/azure/azure-sql/database/reserved-capacity-overview) to provide potential cost optimizations.

- Configure the Zone-Redundant deployment model to spread Business Critical database replicas within the same region across Availability Zones.

- Use [Active Geo-Replication](http://docs.microsoft.com/azure/azure-sql/database/active-geo-replication-overview) to deploy readable replicas within all deployment regions.

- Use Auto Failover Groups to provide [transparent failover](http://docs.microsoft.com/azure/azure-sql/database/designing-cloud-solutions-for-disaster-recovery) to a secondary region, with geo-replication applied to provide replication to additional deployment regions for read optimization and database redundancy.
  - For application scenarios limited to only two deployment regions, the use of Auto Failover Groups should be prioritized.

- Consider automated operational triggers, based on alerting aligned to the application health model, to conduct failovers to geo-replicated instances if a failure impacting the primary and secondary within the Auto Failover Group.

>[!IMPORTANT]
> For applications considering more than fourdeployment regions, serious consideration should be given to application scoped sharding or refactoring the application to support multi-region write technologies, such as Azure Cosmos DB. However, if this isn't feasible within the application workload scenario, it's advised to elevate a region within a single geography to a primary status encompassing a geo-replicated instance to more evenly distribute read access.

- Configure the application to query replica instances for read queries to optimize read performance.

- Use Azure Monitor and [Azure SQL Analytics](http://docs.microsoft.com/azure/azure-monitor/insights/azure-sql#analyze-data-and-create-alerts) for near real-time operational insights in Azure SQL DB for the detection of reliability incidents.

- Use Azure Monitor to evaluate usage for all databases to determine if they have been sized appropriately.
  - Ensure CD pipelines consider load testing under representative load levels to validate appropriate data platform behavior.

- Calculate a health metric for database components to observe health relative to business requirements and resource utilization, using  [monitoring and alerts](http://docs.microsoft.com/azure/azure-sql/database/monitor-tune-overview) to drive automated operational action where appropriate.
  - Ensure key query performance metrics are incorporated so swift action can be taken when service degradation occurs.

- Optimize queries, tables, and databases using [Query Performance Insights](http://docs.microsoft.com/azure/azure-sql/database/query-performance-insight-use) and common [performance recommendations](http://docs.microsoft.com/azure/azure-sql/database/database-advisor-find-recommendations-portal) provided by Microsoft.

- [Implement retry logic](http://docs.microsoft.com/azure/azure-sql/database/troubleshoot-common-connectivity-issues) using the SDK to mitigate transient errors impacting Azure SQL Database connectivity.

- Prioritize the use of service-managed keys when applying server-side Transparent Data Encryption (TDE) for at-rest encryption.
  - If customer-managed keys or client-side (AlwaysEncrypted) encryption is required, ensure keys are appropriately resilient with backups and automated rotation facilities.

- Consider the use of [point-in-time restore](http://docs.microsoft.com/azure/azure-sql/database/recovery-using-backups#point-in-time-restore) as an operational playbook to recover from severe configuration errors.

**Azure Database For PostgreSQL**

- Use the [Hyperscale (Citus)](http://docs.microsoft.com/azure/postgresql/concepts-hyperscale-configuration-options) server configuration to maximize availability across multiple nodes.

- Define a capacity model for the application to inform compute and storage resource requirements within the data platform.
  - Consider the [Hyperscale (Citus) Reservation Discount](http://docs.microsoft.com/azure/postgresql/concepts-hyperscale-reserved-pricing) to provide potential cost optimizations.

## Caching for Hot Tier Data

An in-memory caching layer can be applied to enhance a data platform by significantly increasing read throughput and improving end-to-end client response times for hot tier data scenarios.

Azure provides several services with applicable capabilities for caching key data structures, with Azure Cache for Redis positioned to abstract and optimize data platform read access. This section will therefore focus on the optimal usage of Azure Cache for Redis in scenarios where additional read performance and data access durability is required.

### Design Considerations

- A caching layer provides additional data access durability since even if an outage impacting the underlying data technologies, an application data snapshot can still be accessed through the caching layer.

- In certain workload scenarios, in-memory caching can be implemented within the application platform itself.

**Azure Cache for Redis**

- Redis cache is an open source NoSQL key-value in-memory storage system.

- The Enterprise and Enterprise Flash tiers can be deployed in an active-active configuration across Availability Zones within a region and different Azure regions through geo-replication.
  - When deployed across at least three Azure regions and three or more Availability Zones in each region, with active geo-replication enabled for all Cache instances, Azure Cache for Redis provides an SLA of 99.999% for connectivity to one regional cache endpoint.
  - When deployed across three Availability Zones within a single Azure region a 99.99% connectivity SLA is provided.

- The Enterprise Flash tier runs on a combination of RAM and flash non-volatile memory storage, and while this introduces a small performance penalty it also enables very large cache sizes, up to 13TB with clustering.

- With geo-replication, charges for data transfer between regions will also be applicable in addition to the direct costs associated with cache instances.

- The Scheduled Updates feature doesn't include Azure updates or updates applied to the underlying VM operating system.

- There will be an increase in CPU utilization during a scale-out operation while data is migrated to new instances.

### Design Recommendations

- Consider an optimized caching layer for 'hot' data scenarios to increase read throughput and improve response times.

- Apply appropriate policies for cache expiration and housekeeping to avoid runaway data growth.
  - Consider expiring cache items when the backing data changes.

**Azure Cache for Redis**

- Use the Premium or Enterprise SKU to maximize reliability and performance.
  - For scenarios with extremely large data volumes, the Enterprise Flash tier should be considered.
  - For scenarios where only passive geo-replication is required, the Premium tier can also be considered.

- Deploy replica instances using geo-replication in an active configuration across all considered deployment regions.

- Ensure replica instances are deployed across Availability Zones within each considered Azure region.

- Use Azure Monitor to evaluate Azure Cache for Redis.
  - Calculate a health score for regional cache components to observe health relative to business requirements and resource utilization.
  - Observe and alert on key metrics such as high CPU, high memory usage, high server load, and evicted keys for insights when to scale the cache.

- Optimize [connection resilience](http://docs.microsoft.com/azure/azure-cache-for-redis/cache-best-practices-connection) by implementing retry logic, timeouts, and using a singleton implementation of the Redis connection multiplexer.

- Configure [scheduled updates](http://docs.microsoft.com/azure/azure-cache-for-redis/cache-administration#schedule-updates) to prescribe the days and times that Redis Server updates are applied to the cache.

## Analytical Scenarios

It's increasingly common for mission-critical applications to consider analytical scenarios as a means to drive additional value from encompassed data flows. Application and operational (AIOps) analytical scenarios therefore form a crucial aspect of highly reliable data platform.

Analytical and transactional workloads require different data platform capabilities and optimizations for acceptable performance within their respective contexts.

| Description | Analytical | Transactional |
| --- | --- | --- |
| Use Case | Analyze very large volumes of data ("big data") | Process very large volumes of individual transactions |
| Optimized for | Read queries and aggregations over many records | Near real-time Create/Read/Update/Delete (CRUD) queries over few records |
| Key Characteristics | - Consolidate from data sources of record<br />- Column-based storage<br />- Distributed storage<br />- Parallel processing<br />- Denormalized<br />- Low concurrency reads and writes<br />- Optimize for storage volume with compression | - Data source of record for application<br />- Row-based Storage<br />- Contiguous storage<br />- Symmetrical processing<br />- Normalized<br />- High concurrency reads and writes, index updates<br />- Optimize for fast data access with in-memory storage

Azure Synapse provides an enterprise analytical platform that brings together relational and non-relational data with Spark technologies, using  built-in integration with Azure services such as Azure Cosmos DB to facilitate big data analytics. The design considerations and recommendations within this section will therefore focus on optimal Azure Synapse and Azure Cosmos DB usage for analytical scenarios.

### Design Considerations

- Traditionally, large-scale analytical scenarios are facilitated by extracting data into a separate data platform optimized for subsequent analytical queries.
  - Extract, Transform, and Load (ETL) pipelines are used to extract data will consume throughput and impact transactional workload performance.
  - Running ETL pipelines infrequently to reduce throughput and performance impacts will result in analytical data that is less up-to-date.
  - ETL pipeline development and maintenance overhead increases as data transformations become more complex.
    - For example, if source data is frequently changed or deleted, ETL pipelines must account for those changes in the target data for analytical queries through an additive/versioned approach, dump and reload, or in-place changes on the analytical data. Each of these approaches will have derivative impact, such as index re-creation or update.

**Azure Cosmos DB**

- Analytical queries run on Cosmos DB transactional data will typically aggregate across partitions over large volumes of data, consuming significant Request Unit (RU) throughput, which can impact the performance of surrounding transactional workloads.

- The [Cosmos DB Analytical Store](http://docs.microsoft.com/azure/cosmos-db/analytical-store-introduction) provides a schematized, fully isolated column-oriented data store that enables large-scale analytics on Cosmos DB data from Azure Synapse without impact to Cosmos DB transactional workloads.
  - When a Cosmos DB Container is enabled as an Analytical Store, a new column store is internally created from the operational data in the Container. This column store is persisted separately from the row-oriented transaction store for the container.
  - Create, Update and Delete operations on the operational data are automatically synced to the analytical store, so no Change Feed or ETL processing is required.
  - Data sync from the operational to the analytical store doesn't consume throughput Request Units (RUs) provisioned on the Container or Database. There's no performance impact on transactional workloads. Analytical Store doesn't require allocation of additional RUs on a Cosmos DB Database or Container.
  - Auto-Sync is the process where operational data changes are automatically synced to the Analytical Store. Auto-Sync latency is usually less than two (2) minutes.
    - Auto-Sync latency can be up to five (5) minutes for a Database with shared throughput and a large number of Containers.
    - As soon as Auto-Sync completes, the latest data can be queried from Azure Synapse.
  - Analytical Store storage uses a consumption-based [pricing model](https://azure.microsoft.com/pricing/details/cosmos-db/) that charges for volume of data and number of read and write operations. Analytical store pricing is separate from transactional store pricing.

- Using Azure Synapse Link, the Cosmos DB Analytical Store can be queried directly from Azure Synapse. This enables no-ETL, Hybrid Transactional-Analytical Processing (HTAP) from Synapse, so that Cosmos DB data can be queried together with other analytical workloads from Synapse in near real-time.

- The Cosmos DB Analytical Store isn't partitioned by default.
  - For certain query scenarios, performance will improve by [partitioning Analytical Store](http://docs.microsoft.com/azure/cosmos-db/configure-custom-partitioning) data using keys that are frequently used in query predicates.
  - Partitioning is triggered by a job in Azure Synapse that runs a Spark notebook using Synapse Link, which loads the data from the Cosmos DB Analytical Store and writes it into the Synapse partitioned store in the primary storage account of the Synapse workspace.

- [Azure Synapse Analytics SQL Serverless pools can query the Analytical Store](http://docs.microsoft.com/azure/synapse-analytics/sql/query-cosmos-db-analytical-store) through automatically updated views or via `SELECT / OPENROWSET` commands.

- [Azure Synapse Analytics Spark pools can query the Analytical Store](http://docs.microsoft.com/azure/synapse-analytics/synapse-link/how-to-query-analytical-store-spark-3) through automatically updated Spark tables or the `spark.read` command.

- Data can also be [copied from the Cosmos DB Analytical Store into a dedicated Synapse SQL pool using Spark](http://docs.microsoft.com/azure/synapse-analytics/synapse-link/how-to-copy-to-sql-pool), so that provisioned Azure Synapse SQL pool resources can be used.

- Cosmos DB Analytical Store data can be queried with [Azure Synapse Spark](http://docs.microsoft.com/azure/synapse-analytics/synapse-link/how-to-query-analytical-store-spark-3).
  - Spark notebooks allow for [Spark dataframe](http://docs.microsoft.com/azure/synapse-analytics/synapse-link/how-to-query-analytical-store-spark-3#load-to-spark-dataframe) combinations to aggregate and transform Cosmos DB analytical data with other data sets, and use other advanced Synapse Spark functionality including writing transformed data to other stores or training AIOps Machine Learning models.

![Cosmos DB Analytical Column Store](http://docs.microsoft.com/azure/cosmos-db/media/analytical-store-introduction/transactional-analytical-data-stores.png "Cosmos DB Analytical Column Store")

- The [Cosmos DB Change Feed](http://docs.microsoft.com/azure/cosmos-db/change-feed) can also be used to maintain a separate secondary data store for analytical scenarios.

**Azure Synapse**

- [Azure Synapse](http://docs.microsoft.com/azure/synapse-analytics/overview-what-is) brings together analytics capabilities including SQL data warehousing, Spark big data, and Data Explorer for log and time series analytics.
  - Azure Synapse uses _linked services_ to define connections to other services, such as Azure Storage.
  - Data can be ingested into Synapse Analytics via Copy activity from [supported sources](http://docs.microsoft.com/azure/data-factory/copy-activity-overview?context=/azure/synapse-analytics/context/context&tabs=synapse-analytics#supported-data-stores-and-formats). This permits data analytics in Synapse without impacting the source data store, but adds time, cost and latency overhead due to data transfer.
  - Data can also be queried in-place in supported external stores, avoiding the overhead of data ingestion and movement. Azure Storage with Data Lake Gen2 is a supported store for Synapse and [Log Analytics exported data can be queried via Synapse Spark](https://techcommunity.microsoft.com/t5/azure-monitor/how-to-analyze-data-exported-from-log-analytics-data-using/ba-p/2547888).

- [Azure Synapse Studio](http://docs.microsoft.com/azure/synapse-analytics/overview-what-is#unified-experience) unites ingestion and querying tasks.
  - Source data, including Cosmos DB Analytical Store data and Log Analytics Export data, are queried and processed in order to support business intelligence and other aggregated analytical use cases.

![Azure Synapse Analytics](http://docs.microsoft.com/azure/synapse-analytics/media/overview-what-is/synapse-architecture.png "Azure Synapse Analytics")

### Design Recommendations

- Ensure analytical workloads don't impact transactional application workloads to maintain transactional performance.

**Application Analytics**

- Use Azure Synapse Link with Cosmos DB Analytical Store to perform analytics on Cosmos DB operational data by creating an optimized data store, which won't impact transactional performance.
  - Enable [Azure Synapse Link](http://docs.microsoft.com/azure/cosmos-db/configure-synapse-link#enable-synapse-link) on Azure Cosmos DB accounts.
  - [Create a container enabled for Analytical Store](http://docs.microsoft.com/azure/cosmos-db/configure-synapse-link#create-analytical-ttl), or [enable an existing Container for Analytical Store](http://docs.microsoft.com/azure/cosmos-db/configure-synapse-link#update-analytical-ttl).
  - [Connect the Azure Synapse workspace to the Cosmos DB Analytical Store](http://docs.microsoft.com/azure/synapse-analytics/synapse-link/how-to-connect-synapse-link-cosmos-db) to enable analytical workloads in Azure Synapse to query Cosmos DB data. Use a connection string with a [read-only Cosmos DB key](http://docs.microsoft.com/azure/cosmos-db/database-security?tabs=sql-api#primary-keys).

- Prioritize Cosmos DB Analytical Store with Azure Synapse Link instead of using the Cosmos DB Change Feed to maintain an analytical data store.
  - The Cosmos DB Change Feed may be suitable for very simple analytical scenarios.

**AIOps and Operational Analytics**

- Create a single Azure Synapse workspace with linked services and data sets for each source Azure Storage account where operational data from resources are sent to.

- Create a dedicated Azure Storage account and use it as the workspace primary storage account to store the Synapse workspace catalog data and metadata. Configure it with hierarchical namespace to enable Azure Data Lake Gen2.
  - Maintain separation between the source analytical data and Synapse workspace data and metadata.
    - Do not use one of the regional or global Azure Storage accounts where operational data is sent.

## Next step

Review the considerations for networking considerations.

> [!div class="nextstepaction"]
> [Network and connectivity](mission-critical-networking-connectivity.md)







