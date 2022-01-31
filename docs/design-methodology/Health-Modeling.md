# Health Modeling and Observability

Health modeling and observability are essential concepts to maximize reliability, which focus on robust and contextualized instrumentation and monitoring to gain critical insight into application health, promoting the swift identification and resolution of issues.

Most business-critical applications are significant in terms of both scale and complexity and therefore generate high volumes of operational data which makes it extremely challenging to evaluate and determine optimal operational action. Health modeling ultimately strives to maximize observability by augmenting raw monitoring logs and metrics with key business requirements to quantify application health, driving automated evaluation of health states to achieve consistent and expedited operations.

This design area will therefore focus on the process to define a robust health model, mapping quantified application health states through observability and operational constructs to achieve operational maturity.

- [Layered Application Health](#layered-application-health)
- [Unified Data Sink for Correlated Analysis](#unified-data-sink-for-correlated-analysis)
- [Dashboarding](#dashboarding)
- [Automated Incident Response](#automated-incident-response)
- [Predictive Action and AIOps](#predictive-action-and-aiops)

> There are ultimately three overarching levels of operational maturity which should be used as a reference when striving to maximize reliability.
>   1) *Detect* and respond to issues as they happen.
>   1) *Diagnose* issues that are occurring or have already occurred.
>   1) *Predict* and prevent issues before they take place.

## Layered Application Health

In order to build a health model it is first necessary to define what application health means in the context of key business requirements, quantifying ‘healthy’ and ‘unhealthy’ states in a layered and measurable format. More specifically, health definitions for each distinct application component should be captured in the context of a steady running state and aggregated according to application user flows in conjunction with key non-functional business requirements for performance and availability. The health states for each individual user flow can then subsequently be aggregated to form a meaningful representation of overall application health. Once established, these layered health definitions should be used to inform critical monitoring metrics across all system components and validate operational sub-system composition.

> When defining what 'unhealthy' states represent for all levels of the application, it is important to distinguish between transient and non-transient failure states to qualify service degradation relative to unavailability.

### Design Considerations

- The process of modeling health is a top-down design activity that starts with an architectural exercise to define all user flows and map dependencies between functional/logical components, thereby implicitly mapping dependencies between Azure resources.

- A health model is entirely dependent on the context of the solution it represents, and therefore cannot be solved 'out-of-the-box' since 'one size does not fit all'.
  - Applications will differ in composition and dependencies
  - Metrics and metric thresholds for resources must also be finely tuned in terms of what values represent healthy and unhealthy states, which is heavily influenced by encompassed application functionality and target non-functional requirements.

- A layered health model enables application health to be traced back to lower level dependencies which helps to quickly root cause service degradation.

- To capture health states for an individual component, that component's distinct operational characteristics must be understood under a steady state that is reflective of production load. Performance testing is therefore a requisite capability to define and continually evaluate application health.

- Failures within a cloud solution may not happen in isolation. An outage in a single component may lead to several capabilities or additional components becoming unavailable.
  - Such errors may not be immediately observable.

### Design Recommendations

- Define a measurable health model as a priority to ensure a clear operational understanding of the entire application.
  - The health model should be layered and reflective of the application structure.
  - The foundational layer should consider individual application components (i.e. Azure resources).
  - Foundational components should be aggregated alongside key non-functional requirements to build a business-contextualized lens into the health of system flows.
  - System flows should be aggregated with appropriate weights based on business criticality to build a meaningful definition of overall application health.
    - Financially significant or customer-facing user flows should be prioritized.
  - Each layer of the health model should capture what ‘healthy’ and ‘unhealthy’ states represent.
  - Ensure the heath model can distinguish between transient and non-transient unhealthy states to isolate service degradation from unavailability.

- Represent health states using a granular health score for every distinct application component and every user flow by aggregating health scores for mapped dependent components, considering key non-functional requirements as coefficients.
  - The health score for a user flow should be represented by the lowest score across all mapped components, factoring in relative attainment against non-functional requirements for the user flow.
  - The model used to calculate health scores must consistently reflect operating health, and if this is not the case, the model should be adjusted and redeployed to reflect new learnings.
  - Define health score thresholds to reflect health status.

- The health score must be calculated automatically based on underlying metrics, which can be visualized through observability patterns and acted on through automated operational procedures.
  - The health score should become core to the monitoring solution, so that operating teams no longer have to interpret and map operational data to application health.

- Leverage the health model to calculate availability SLO attainment instead of raw availability, ensuring the demarcation between service degradation and unavailability is reflected as separate SLOs.

- Leverage the health model within CI/CD pipelines and test cycles to validate application health is maintained after code and configuration updates.
  - The health model should be used to observe and validate health during load testing and chaos testing as part of CI/CD processes.

- Building and maintaining a health model is an iterative process and engineering investment should be aligned to drive continuous improvements.
  - Define a process to continually evaluate and fine-tune the accuracy of the model, and consider investing in machine learning models to further train the model.

### Reference Layered Health Model

> Please note that this section provides a simplified representation of a layered application health model to assist readers with the underlying concept. For a more comprehensive and contextualized health model reference please refer to the [foundational-online](https://github.com/azure/alwayson-foundational-online) and [foundational-connected](https://github.com/azure/alwayson-foundational-connected) reference implementations.

When implementing a health model it is critical to first define the health of individual components through the aggregation and interpretation of key resource-level metrics. An example of how resource metrics can used is the image below:

[![AlwaysOn Example Health Definitions](/docs/media/alwayson-example-health-definitions.png "AlwaysOn Example Health Definitions")](./Health-Modeling.md)

This definition of health can subsequently be represented by a KQL query, as demonstrated by the example AKS query below that aggregates InsightsMetrics (AKS Container Insights) and AzureMetrics (Azure Diagnostics) and compares (inner join) against modelled health thresholds.

``` kql
// ClusterHealthStatus
let Thresholds=datatable(MetricName: string, YellowThreshold: double, RedThreshold: double) [
    // Disk Usage:
    "used_percent", 50, 80,
    // Network errors in:
    "err_in", 0, 0,
    // Network errors out:
    "err_out", 0, 0,
    // Average node cpu usage %:
    "node_cpu_usage_percentage", 60, 90,
    // Average node disk usage %:
    "node_disk_usage_percentage", 60, 80,
    // Average node memory usage %:
    "node_memory_rss_percentage", 60, 80
    ];
InsightsMetrics
| summarize arg_max(TimeGenerated, *) by Computer, Name
| project TimeGenerated,Computer, Namespace, MetricName = Name, Value=Val
| extend NodeName = extract("([a-z0-9-]*)(-)([a-z0-9]*)$", 3, Computer)
| union (
    AzureMetrics
    | extend ResourceType = extract("(PROVIDERS/MICROSOFT.)([A-Z]*/[A-Z]*)", 2, ResourceId)
    | where ResourceType == "CONTAINERSERVICE/MANAGEDCLUSTERS"
    | summarize arg_max(TimeGenerated, *) by MetricName
    | project TimeGenerated, MetricName, Namespace = "AzureMetrics", Value=Average
    )
| lookup kind=inner Thresholds on MetricName
| extend IsYellow = iff(Value > YellowThreshold and Value < RedThreshold, 1, 0)
| extend IsRed = iff(Value > RedThreshold, 1, 0)
| project NodeName, MetricName, Value, YellowThreshold, IsYellow, RedThreshold, IsRed
```

The resulting table output can subsequently be transformed into a health score for easier aggregation at higher levels of the health model.

```kql
// ClusterHealthScore
ClusterHealthStatus
| summarize YellowScore = max(IsYellow), RedScore = max(IsRed)
| extend HealthScore = 1-(YellowScore*0.25)-(RedScore*0.5)
```

These aggregated scores can subsequently be represented as a dependency chart using visualization tools like Grafana to illustrate the health model. The image below depicts an example layered health model from the [foundational-online](https://github.com/azure/alwayson-foundational-online) reference implementation, and demonstrates how a change in health state for a foundational component can have a cascading impact to user flows and overall application health (the example values correspond to the table in the previous image).

[![AlwaysOn Example Health Model Visualization](/docs/media/images/alwayson-example-fault-states.png "AlwaysOn Example Health Model Visualization")](./Health-Modeling.md)

## Unified Data Sink for Correlated Analysis

Numerous operational datasets must be gathered from all system components to accurately represent a defined heath model, considering logs and metrics from both application components and underlying Azure resources. This vast amount of data ultimately needs to be stored in a format that allows for near-real time interpretation to facilitate swift operational action. Moreover, correlation across all encompassed data sets is required to ensure effective analysis is unbounded, allowing for the layered representation of health.

A unified data sink is therefore required to ensure all operational data is swiftly stored and made available for correlated analysis to build a 'single pane' representation of application health. Azure provides several different operational technologies under the umbrella of [Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/overview#overview), and Azure Monitor Log Analytics serves as the core Azure-native data sink to store and analyze operational data.

[![AlwaysOn Health Data Collection](/docs/media/alwayson-health-data-collection.png "AlwaysOn Health Data Collection")](./Health-Modeling.md)

### Design Considerations

**Azure Monitor**

- Azure Monitor is enabled by default for all Azure subscriptions, but Azure Monitor for Logs (Log Analytics) and Azure Application Insights resources must be deployed and configured to incorporate data collection and querying capabilities.

- Azure Monitor supports three types of observability data: Logs, Metrics, and Distributed Traces.
  - Logs are stored in Azure Monitor Logs (previously called Log Analytics) workspaces based on [Azure Data Explorer](https://docs.microsoft.com/azure/data-explorer/).
    - Log queries are stored in query packs that can be shared across subscriptions, and are used to drive observability components such as dashboards, workbooks, or other reporting and visualization tools.
  - Metrics are stored in an internal time-series diagnostic service database, which for most Azure resources is [retained](https://docs.microsoft.com/azure/azure-monitor/essentials/data-platform-metrics#retention-of-metrics) for 93 days.
    - Metric collection is configured through resource Diagnostic settings.

- All Azure resources expose logs and metrics, but resources must be appropriately configured to route diagnostic data to your desired data sink.

> Azure provides a variety of [Built-In Policies](https://docs.microsoft.com/azure/azure-monitor/policy-reference) which can be applied to ensure deployed resources are configured to send logs and metrics to an Azure Monitor instance.

- It is not uncommon for regulatory controls to require operational data remains within originating geographies or countries.

- Regulatory requirements may stipulate the retention of critical data types for an extended period of time.
  - For example, in regulated banking, audit data must be retained for at least 7 years.

- Different operational data types may require different retention periods.
  - For example, security logs may need to be retained for a long period, while performance data is unlikely to require long-term retention outside the context of AIOps.

- Data can be [exported](https://docs.microsoft.com/azure/azure-monitor/logs/logs-data-export?tabs=portal) from Log Analytics Workspaces for long term retention and/or auditing purposes.

- [Azure Monitor Logs Dedicated Clusters](https://docs.microsoft.com/azure/azure-monitor/logs/logs-dedicated-clusters) provides a deployment option which enables Availability Zones for protection from zonal failures in supported Azure regions.
  - Dedicated Clusters require a minimum daily data ingest commitment.

- Azure Monitor for Logs resources, including underlying log and metrics storage, are deployed into a specified Azure region.

- To protect against loss of data from unavailability of an Azure Monitor for Logs workspace, resources can be configured with multiple Diagnostics configurations.
  - Each Diagnostic configuration can target metrics and logs at a separate Azure Monitor for Log workspace.
    - Each additional Azure Monitor for Logs workspace will incur extra costs.
    - The redundant Azure Monitor for Logs workspaces can be deployed into the same Azure region, or into separate Azure regions for additional regional redundancy.
      - Sending logs and metrics from an Azure resource to an Azure Monitor for Logs workspace in a different region will incur inter-region data egress costs.
      - Some Azure resources require an Azure Monitor for Logs workspace within the same region as the resource itself.

- Azure Monitor Logs workspace data [can be exported to Azure Storage or Azure Event Hubs on a continuous, scheduled, or one-time basis](https://docs.microsoft.com/azure/azure-monitor/logs/logs-data-export).
  - Data export allows for long-term data archiving and protects against possible operational data loss due to unavailability.
  - Available export destinations are Azure Storage or Azure Event Hub.
    - Azure Storage can be configured for different [redundancy levels](https://docs.microsoft.com/azure/storage/common/storage-redundancy) including zonal or regional.
    - Data export to Azure Storage stores the data within .json files.
  - Data export destinations must be within the same Azure region as the Azure Monitor Logs workspace.
    - An Event Hub data export destination to be within the same region as the Azure Monitor Logs workspace.
      - Azure Event Hubs geo-disaster recovery is not applicable for this scenario.
  - There are several [data export limitations](https://docs.microsoft.com/azure/azure-monitor/logs/logs-data-export?tabs=portal#limitations).

> Only specific Azure Monitor Logs [tables are supported](https://docs.microsoft.com/azure/azure-monitor/logs/logs-data-export#supported-tables) for data export.

- Azure Monitor Logs has [user query throttling limits](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#user-query-throttling) which may appear as reduced availability to clients, such as observability dashboards.
  - Five concurrent queries per user: if five queries are already running, additional queries are placed in a per-user concurrency queue until a running query ends.
  - Time in concurrency queue: if a query sits in the concurrency queue for over three minutes, it will be terminated and a 429 error code returned.
  - Concurrency queue depth limit: the concurrency queue is limited to 200 queries, and additional queries will be rejected with a 429 error code.
  - Query rate limit: there is a per-user limit of 200 queries per 30 seconds across all workspaces.

- [Query Packs](https://docs.microsoft.com/azure/azure-monitor/logs/query-packs) are Azure Resource Manager resources which can be used to protect and recover Azure Monitor Logs queries in the event of Azure Monitor Logs workspace unavailability.
  - Query Packs contain queries as JSON and can be stored external to Azure similar to other infrastructure-as-code assets.
    - Deployable through the Microsoft.Insights REST API.
    - If an Azure Monitor for Logs workspace must be re-created the Query Pack can be re-deployed from an externally stored definition.

- Application Insights can be deployed in a workspace-based deployment model, underpinned by a Log Analytics Workspace where all the data is stored.

- Sampling can be enabled within Application Insights to reduce the amount of telemetry sent and optimize data ingest costs.

- Log Analytics and Application Insights [charge based on the volume of data ingested and the duration that data is retained for](https://azure.microsoft.com/pricing/details/monitor/).
  - Data ingested into a Log Analytics Workspace can be retained at no additional charge up to first 31 days (90 days if Sentinel is enabled)
  - Data ingested into a Workspace-based Application Insights is retained for the first 90 days at no extra charge.

- The Log Analytics Commitment Tier pricing model provides a predictable approach to data ingest charges.
  - Any usage above the reservation level is billed at the same price as the current tier.

- Azure Monitor Log Analytics, Application Insights, and Azure Data Explorer use the Kusto Query Language (KQL).

- Log Analytics queries are saved as *functions* within Log Analytics (`savedSearches`).

### Design Recommendations

- Use Azure Monitor for Logs (Log Analytics) as a unified data sink to provide a 'single pane' across all operational data sets.
  - Decentralize Log Analytics Workspaces across all leveraged deployment regions. Each Azure region with an application deployment should consider a Log Analytics Workspace to gather all operational data originating from that region. All global resources should leverage a separate dedicated Log Analytics Workspace which should be deployed within a primary deployment region.
    - Sending all operational data to a single Log Analytics Workspace would create a single point of failure.
    - Requirements for data residency might prohibit data leaving the originating region, and federated workspaces solves for this requirement by default.
    - There is a substantial egress cost associated with transferring logs and metrics across regions.
  - All deployment stamps within the same region can leverage the same regional Log Analytics Workspace.

- Consider configuring resources with multiple diagnostic configurations pointing to different Azure Monitor for Logs workspaces to protect against Azure Monitor unavailability for applications with fewer regional deployment stamps.

- Use Application Insights as a consistent Application Performance Monitoring (APM) tool across all application components to collect application logs, metrics, and traces.
  - Deploy Application Insights in a workspace-based configuration to ensure each regional Log Analytics Workspaces contains logs and metrics from both application components and underlying Azure resources.

- Leverage [Cross-Workspace queries](https://docs.microsoft.com/azure/azure-monitor/logs/cross-workspace-query) to maintain a unified 'single pane' across the different workspaces.

- Leverage [Query Packs](https://docs.microsoft.com/azure/azure-monitor/logs/query-packs) to protect Azure Monitor Logs queries in the event of workspace unavailability.
  - Store query packs within the application git repository as infrastructure-as-code assets.

- All Log Analytics Workspaces should be treated as long-running resources with a different life-cycle to application resources within a regional deployment stamp.

- Export critical operational data from Log Analytics for long-term retention and analytics to facilitate AIOps and advanced analytics to refine the underlying the health model and inform predictive action.

- Carefully evaluate which data store should be used for long-term retention; not all data has to be stored in a hot and queryable data store.
  - It is strongly recommended to use Azure Storage in a GRS configuration for long-term operational data storage.
    - Use the Log Analytics Export capability to export all available data sources to Azure Storage.

- Select appropriate retention periods for operational data types within log analytics, configuring longer retention periods within the workspace where 'hot' observability requirements exist.

- Use Azure Policy to ensure all regional resources route operational data to the correct Log Analytics Workspace.

> In an Enterprise Scale environment, if there is a requirement for centralized storage of operational data, either a) [fork](https://docs.microsoft.com/azure/azure-monitor/logs/logs-data-export?tabs=portal) data at instantiation so it is ingested into both centralized tooling and Log Analytics Workspaces dedicated to the application, or b) expose access to application Log Analytics workspaces so that central teams can query application data. It is ultimately critical that operational data originating from the solution is available within Log Analytics Workspaces dedicated to the application.

> If SIEM integration is required, do not send raw log entries, but instead send critical alerts.

- Only configure sampling within Application Insights if it is required to optimize performance, or if not sampling becomes cost prohibitive.
  - Excessive sampling can lead to missed or inaccurate operational signals.

- Use correlation IDs for all trace events and log messages to tie them to a given request.
  - Return correlation IDs to the caller for all calls not just failed requests.

- Ensure application code incorporates proper instrumentation and logging to inform the health model and facilitate subsequent troubleshooting or root cause analysis when required.
  - Application code should leverage Application Insights to facilitate [Distributed Tracing](https://docs.microsoft.com/dotnet/core/diagnostics/distributed-tracing-concepts), by providing the caller with a comprehensive error message that includes a correlation ID when a failure occurs.

- Use [structured logging](https://stackify.com/what-is-structured-logging-and-why-developers-need-it/) for all log messages.

- Add meaningful health probes to all application components.
  - When using AKS, configure the health endpoints for each deployment (pod) so that Kubernetes can correctly determine when a pod is healthy or unhealthy.
  - When using Azure App Service, configure the [Health Checks](https://docs.microsoft.com/azure/app-service/monitor-instances-health-check) so that scale out operations will not cause errors by sending traffic to instances which are not-yet ready, and making sure unhealthy instances are recycled quickly.

> If the application is subscribed to Microsoft Mission-Critical Support, consider exposing key health probes to Microsoft Support, so application health can be modelled more accurately by Microsoft Support.

- Log successful health check requests, unless increased data volumes cannot be tolerated in the context of application performance, since they provide additional insights for analytical modelling.

- Do not configure production Log Analytics Workspaces to apply a daily cap, which limits the daily ingestion of operational data, since this can lead to the lose of critical operational data.
  - In lower environments, such as Development and Test, it can be considered as an optional cost saving mechanism.

- Provided operational data ingest volumes meet the minimum tier threshold, configure Log Analytics Workspaces to use Commitment Tier based pricing to drive cost efficiencies relative to the 'pay-as-you-go' pricing model.

- It is strongly recommended to store Log Analytics queries using source control and use CI/CD automation to deploy them to relevant Log Analytics instances.

## Dashboarding

Visually representing the health model alongside critical operational data is essential to achieve effective operations and maximize reliability. Dashboards should ultimately be utilized to provide near-real time insights into application health for DevOps teams, facilitating the swift diagnosis of deviations from steady state.

Microsoft provides several data visualization technologies, including Azure Dashboards, PowerBI, and Azure Managed Grafana (currently in-preview). Azure Dashboards is positioned to provide a tightly integrated out-of-the-box visualization solution for operational data within Azure Monitor. It therefore has a fundamental role to play in the visual representation of operational data and application health for an AlwaysOn solution. However, there are several limitations in terms of the positioning of Azure Dashboards as a holistic observability platform, and as a result consideration should be given to the supplemental use of market-leading observability solutions, such as Grafana which is also provided as a managed solution within Azure.

This section will therefore focus on the use of Azure Dashboards and Grafana to build a robust dashboarding experience capable of providing technical and business lenses into application health, enabling DevOps teams and effective operation.

>Robust dashboarding is essential to diagnose issues that have already occurred, and support operational teams in detecting and responding to issues as they happen.

### Design Considerations

- When visualizing the health model using Log Analytics queries, note that there are [Log Analytics limits on concurrent and queued queries, as well as the overall query rate](https://docs.microsoft.com/azure/azure-monitor/service-limits#user-query-throttling), with subsequent queries queued and throttled.

- Queries to retrieve operational data used to calculate and represent health scores can be written and executed in either Azure Monitor Log Analytics or Azure Data Explorer.
  - Sample queries are available [here](https://docs.microsoft.com/azure/azure-monitor/logs/examples).

- Log Analytics imposes several [query limits](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#action-groups) which must be designed for when designing operational dashboards.

- The visualization of raw resource metrics, such as CPU utilization or network throughput, requires manual evaluation by operations teams to determine health status impacts, and this can be challenging during an active incident.

- If multiple users leverage dashboards within a tool like Grafana, the number of queries sent to Log Analytics multiplies quickly. 
  - Reaching the concurrent query limit on Log Analytics will queue subsequent queries, making the dashboard experience feel 'slow'. 

### Design Recommendations

- Collect and present queried outputs from all regional Log Analytics Workspaces and the global Log Analytics Workspace to build a unified view of application health.

> When deploying into an Enterprise-Scale architecture, consideration should be given to also query the [central platform Log Analytics Workspace](https://docs.microsoft.com/azure/cloud-adoption-framework/ready/enterprise-scale/management-and-monitoring#plan-platform-management-and-monitoring) if key dependencies on platform resources exist, such as Express Route for scenarios involving on-premises communication.

- A ‘traffic light’ model should be used to visually represent 'healthy' and 'unhealthy' states, with green used to illustrate when key non-functional requirements are fully satisfied and resources are optimally utilized.
  - Use "Green", "Amber, and "Red" to represent "Healthy", "Degraded", and "Unavailable" states.

- Leverage Azure Dashboards to create operational lenses for global resources and regional deployment stamps, representing key metrics such as request count for Azure Front Door, server side latency for Cosmos DB, incoming/outgoing messages for Event Hub, and CPU utilization or deployment statuses for AKS.
  - Dashboards should be tailored to drive operational effectiveness, infusing learnings from failure scenarios to ensure DevOps teams have direct visibility into key metrics.

- If Azure Dashboards cannot be used to accurately represent the health model and requisite business requirements, then it is strongly recommended to consider Grafana as an alternative visualization solution, providing market-leading capabilities and an extensive open-source plugin ecosystem.
  - Evaluate the managed Grafana preview offering to avoid the operational complexities of managing Grafana infrastructure.

- When deploying self-hosted Grafana, employ a highly-available and geo-distributed design to ensure critical operational dashboards can be resilient to regional platform failures and cascading error scenarios.
  - Separate configuration state into an external datastore, such as Azure Database for Postgres or MySQL, to ensure Grafana application nodes remain stateless.
    - Configure database replication across deployment regions.
  - Deploy Grafana nodes to App Services in a highly-available configuration across ones within a region, using container based deployments.
    - Deploy App Service instances across considered deployment regions.

    > App Services provides a low-friction container platform which is ideal for low-scale scenarios such as operational dashboards, and isolating Grafana from AKS provides a clear separation of concern between the primary application platform and operational representations for that platform. Please refer to the Application Platform deign area for further configuration recommendations.

  - Use Azure Storage in a GRS configuration to host and manage custom visuals and plugins.
  - Deploy app service and database read-replica Grafana components to a minimum of 2 deployment regions, and consider employing a model where Grafana is deployed to all considered deployment regions.

> For scenarios targeting a >= 99.99% SLO, Grafana should be deployed within a minimum of 3 deployment regions to maximize overall reliability for key operational dashboards.

- Mitigate Log Analytics query limits by aggregating queries into a single or small number of queries, such as by using the KQL 'union' operator, and set an appropriate refresh rate on the dashboard.
  - An appropriate maximum refresh rate will depend on the number and complexity of dashboard queries; analysis of implemented queries is required.

- If the concurrent query limit of log analytics is being reached, consider optimizing the retrieval pattern by (temporarily) storing the data required for the dashboard in a high performance datastore such as Azure SQL.

## Automated Incident Response

While the visual representations of application health provides invaluable operational and business insights to support issue detection and diagnosis, it relies on the readiness and interpretations of operational teams, as well as the effectiveness of subsequent human-triggered responses. Therefore, to maximize reliability it is necessary to implement extensive alerting to detect proactively respond to issues in near real-time.

[Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/alerts/alerts-overview) provides an extensive alerting framework to detect, categorize, and respond to operational signals through [Action Groups](https://docs.microsoft.com/azure/azure-monitor/alerts/action-groups). This section will therefore focus on the use of Azure Monitor alerts to drive automated actions in response to current or potential deviations from a healthy application state.

>Alerting and automated action is critical to effectively detect and swiftly respond to issues as they happen, before greater negative impact can occur. Alerting also provides a mechanism to interpret incoming signals and respond to prevent issues before they occur.

### Design Considerations

- Alert rules are defined to fire when a conditional criteria is satisfied for incoming signals, which can include a variety of [data sources](https://docs.microsoft.com/azure/azure-monitor/agents/data-sources), such as metrics, log search queries, or availability tests.

- Alerts can be defined within Log Analytics or Azure Monitor on the specific resource.

- Some metrics are only interrogatable within Azure Monitor, since not all diagnostic data points are made available within Log Analytics.

- The Azure Monitor Alerts API can be leveraged to retrieve active and historic alerts.

- There are subscription limits related to alerting and action groups which must be designed for:
  - [Limits](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#alerts) exist for the number of configurable alert rules.
  - The Alerts API has [throttling limits](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#alerts-api) which should be considered for extreme usage scenarios.
  - Action Groups have [several hard limits](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#action-groups) for the number of configurable responses which must be designed for.
    - Each response type has a limit of 10 actions, apart from email which has a limit of 1,000 actions.

- Alerts can be integrated within a layered health model by creating an Alert Rule for a saved log search query from the model's 'root' scoring function.
  - For example, using 'WebsiteHealthScore' and alerting on a numeric value that represents an 'Unhealthy' state.

### Design Recommendations

- For resource-centric alerting, create alert rules within Azure Monitor to ensure all diagnostic data is available for the alert rule criteria.

- Consolidate automated actions within a minimal number of Action Groups, aligned with service teams to support a DevOps approach.

- Respond to excessive resource utilization signals through automated scale operations, leveraging Azure-native auto-scale capabilities where possible. Where built-in auto-scale functionality is not applicable, use the component health score to model signals and determine when to respond with automated scale operations.
  - Ensure automated scale operations are defined according to a capacity model which quantifies scale relationships between components, so that scale responses encompass components which need to be scaled in relation to other components.

- Model actions to accommodate a prioritized ordering which should be determined by business impact.

- Leverage the Azure Monitor Alerts API to gather historic alerts to incorporate within 'cold' operational storage for advanced analytics.

- For critical failure scenarios which cannot be met with an automated response, ensure operational 'runbook automation' is in-place to drive swift and consistent action once manual interpretation and sign-off is provided.
  - Leverage alert notifications to drive swift identification of issues requiring manual interpretation
  
- Create allowances within engineering sprints to drive incremental improvements in alerting to ensure new failure scenarios which have not previously been considered can be fully accommodated within new automated actions.

- Conduct operational readiness tests as part of CI/CD processes to validate key alert rules for deployment updates.

## Predictive Action and AIOps

Machine learning models can be applied to correlate and prioritize operational data, helping to gather critical insights related to filtering excessive alert 'noise' and predicting issues before they cause impact, as well as accelerating incident response when they do.

More specifically, an AIOps methodology can be applied to distil critical insights about the behavior of the system, users, and DevOps processes. These insights can include identifying a problem happening now (*detect*), quantifying why the problem is happening (*diagnose*), or signaling what will happen in the future (*predict*). Such insights can be used to drive actions which adjust and optimize the application to mitigate active or potential issues, leveraging key business metrics, system quality metrics, and DevOps productivity metrics, to prioritize according to business impact. Conducted actions can themselves be infused into the system though a feedback loop which further trains the underlying model to drive additional efficiencies.

[![AlwaysOn AIOps Methodologies](/docs/media/lwayson-aiops-methodology.png "AlwaysOn AIOps Methodologies")](./Health-Modeling.md)

There are multiple analytical technologies within Azure, such as Azure Synapse and Azure Databricks, which can be leveraged to build and train analytical models for AIOps. This section will therefore focus on how these technologies can be positioned within an AlwaysOn application design to accommodate AIOps and drive predictive action, focusing on Azure Synapse which reduces friction by bringing together the best of Azure's data services along with powerful new features.

>AIOps is used to drive predictive action, interpreting and correlating complex operational signals observed over a sustained period in order to better respond to and prevent issues before they occur.

### Design Considerations

- Azure Synapse Analytics offers multiple Machine Learning (ML) capabilities.
  - ML models can be trained and run on Synapse Spark Pools with libraries including MLLib, SparkML and MMLSpark, as well as popular open-source libraries, such as [Scikit Learn](https://scikit-learn.org/stable/).
  - ML models can be trained with common data science tools like PySpark/Python, Scala, or .NET.

- Synapse Analytics is integrated with Azure ML through Azure Synapse Notebooks, which enables ML models to be trained in an Azure ML Workspace using [Automated ML](https://docs.microsoft.com/azure/machine-learning/concept-automated-ml).

- Synapse Analytics also enables ML capabilities using [Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/what-are-cognitive-services) to solve general problems in various domains, such as [Anomaly Detection](https://docs.microsoft.com/azure/cognitive-services/anomaly-detector/). Cognitive Services can be used in Azure Synapse, Azure Databricks, and via SDKs and REST APIs in client applications.

- Azure Synapse natively integrates with [Azure Data Factory](https://docs.microsoft.com/azure/data-factory/introduction) tools to extract, transform, and load (ETL) or ingest data within orchestration pipelines.

- Azure Synapse enables external dataset registration to data stored in Azure Blob storage or Azure Data Lake Storage.
  - Registered datasets can be used in Synapse Spark pool data analytics tasks.

- Azure Databricks can be integrated into Azure Synapse Analytics pipelines for additional Spark capabilities.
  - Synapse orchestrates reading data and sending it to a Databricks cluster, where it can be transformed and prepared for ML model training.

- Source data typically needs to be prepared for analytics and ML.
  - Synapse offers various tools to assist with data preparation, including Apache Spark, Synapse Notebooks, and serverless SQL pools with T-SQL and built-in visualizations.

- ML models that have been trained, operationalized, and deployed can be used for _batch_ scoring in Synapse.
  - AIOps scenarios, such as running regression or degradation predictions in CI/CD pipelined, may require _real-time_ scoring.

- There are subscription limits for [Azure Synapse](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#azure-synapse-analytics-limits) which should be fully understood in the context of an AIOps methodology.

- To fully incorporate AIOps it is necessary to feed near real-time observability data into real-time ML inference models on an ongoing basis.
  - Capabilities such as anomaly detection should be evaluated within the observability data stream.

### Design Recommendations

- Ensure all Azure resources and application components are fully instrumented so that a complete operational dataset is available for AIOps model training.

- Ingest Log Analytics operational data from the global and regional Azure Storage Accounts into Azure Synapse for analysis.

- Use the Azure Monitor Alerts API to retrieve historic alerts and store it within cold storage for operational data to subsequently use within ML models. If Log Analytics data export is used, store historic alerts data in the same Azure Storage accounts as the exported Log Analytics data.

- After ingested data is prepared for ML training, write it back out to Azure Storage so that it is available for ML model training without requiring Synapse data preparation compute resources to be running.

- Ensure ML model operationalization supports both batch and real-time scoring.

- As AIOps models are created, implement MLOps and apply DevOps practices to [automate the ML lifecycle](https://docs.microsoft.com/azure/machine-learning/concept-model-management-and-deployment#automate-the-ml-lifecycle) for training, operationalization, scoring, and continuous improvement.
  - Create an iterative CI/CD process for AIOps ML models.

- Evaluate [Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/what-are-cognitive-services) for specific predictive scenarios due to their low administrative and integration overhead.
  - Consider [Anomaly Detection](https://docs.microsoft.com/azure/cognitive-services/anomaly-detector/) to quickly flag unexpected variances in observability data streams.

|Previous Page|Next Page|
|:--|:--|
|[Data Platform](./Data-Platform.md) |[Deployment and Testing](./Deployment-Testing.md) |

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

[AlwaysOn - Full List of Documentation](/docs/README.md)
