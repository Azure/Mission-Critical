# Networking and connectivity

Networking is a fundamental design topic for a Mission-Critical application, particularly given the recommended globally distributed active-active design approach.

This section will therefore explore various network topology topics at an application level, considering requisite connectivity and redundant traffic management. More specifically, it will highlight critical considerations and recommendations intended to inform the design of a secure and scalable global network topology for a Mission-Critical application.

- [Global Traffic Routing](#global-traffic-routing)
- [Application Delivery Services](#application-delivery-services)
- [Virtual Network Integration](#virtual-network-integration)
- [Internet Egress](#internet-egress)
- [Inter-Zone and Inter-Region Connectivity](#inter-zone-and-inter-region-connectivity)
- [Micro-Segmentation and Kubernetes Network Policies](#micro-segmentation-and-kubernetes-network-policies)

## Global traffic routing

The use of multiple active regional deployment stamps necessitates a global routing service to distribute traffic to each active stamp.

Within Azure, [Azure Front Door](https://azure.microsoft.com/services/frontdoor/), [Azure Traffic Manager](https://azure.microsoft.com/services/traffic-manager/), and [Azure Standard Load Balancer](https://docs.microsoft.com/azure/load-balancer/cross-region-overview) provide requisite routing capabilities to manage global traffic across a multi-region application.

> There are also a variety of third-party globally routing technologies, such as those offered by CDN providers, which can almost seamlessly be swapped in to replace or extend the use of Azure-native global routing services.

This section will therefore explore key differences between Azure Front Door, Azure Traffic Manager, and Azure Standard Load Balancer to define how each can can be leveraged to optimize different scenarios.

### Design considerations

- A routing service bound to a single region represents a single-point-of-failure and a significant risk with regards to regional outages.

- If the application workload scenario encompasses client control, such as with mobile or desktop client applications, it is possible to provide service redundancy within client routing logic.
  - Multiple global routing technologies, such as Azure Front Door and Azure Traffic Manager, can be considered in parallel for redundancy, with clients configured to failover to an alternative technology when certain failure conditions are met.
    - The introduction of multiple global routing services introduces significant complexities around edge caching and Web Application Firewall capabilities, as well as certificate management for SSL offload and application validation for ingress paths.
  - Third-party technologies can also be considered, providing global routing resiliency to all levels of Azure platform failures.

- Capability disparity between Azure Front Door and Traffic Manager means that if the two technologies are positioned alongside one another for redundancy, a different ingress path or design changes would be required to ensure a consistent and acceptable level of service is maintained.

- Azure Front Door and Azure Traffic Manager are globally distributed services with built-in multi-region redundancy and availability.
  - Hypothetical failure scenarios of a scale large enough to threaten the global availability of these resilient routing services presents a broader risk to the application in terms of cascading and correlated failures.
    - Failure scenarios of this scale are only feasibly caused by shared foundational services, such as Azure DNS or Azure AD which serve as global platform dependencies for almost all Azure services.
      - If a redundant Azure technology is applied it is likely that the secondary service will also be experiencing unavailability or a degraded service.
    - Global routing service failure scenarios are highly likely to significantly impact many other services used for key application components through interservice dependencies.
      - Even if a third-party technology is used, the application will likely be in an un-heathy state due to the broader impact of the underlying issue, meaning that routing to application endpoints on Azure will provide little value anyway.

- Global routing service redundancy provides mitigation for an extremely small number of hypothetical failure scenarios, where the impact of a global outage is constrained to the routing service itself.
  - To provide broader redundancy to global outage scenarios, a multi-cloud active-active deployment approach can be considered.
    - A multi-cloud active-active deployment approach introduces significant operational complexities which pose significant resiliency risks, likely far outweighing the hypothetical risks of a global outage.

- For scenarios where client control is not possible, a dependency must be taken on a single global routing service to provide a unified entry point for all active deployment regions.
  - When used in isolation they represent a single-point-of-failure at a service level due to global dependencies, even though built-in multi-region redundancy and availability is provided.
  - The SLA provided by the selected global routing service represents the maximum attainable composite SLA, regardless of how many deployment regions are considered.

- When client control is not possible, operational mitigations can be considered to define a process for migrating to a secondary global routing service in the event that a global outage disables the primary service.
  - Migrating from one global routing service to another is typically a lengthy process lasting several hours, particularly where DNS propagation is considered.

- Some third-party global routing services provide a 100% SLA. However, the historic and attainable SLA provided by these services is typically lower than 100%.
  - While these services provide financial reparations for unavailability, it comes of little significance when the impact of unavailability is significant, such as with safety-critical scenarios where human life is ultimately at stake.
    - Technology redundancy or sufficient operational mitigations should therefore still be considered even when the advertised legal SLA is 100%.

**Azure Front Door**

- Azure Front Door provides global HTTP/S load balancing and optimized connectivity using the Anycast protocol with split TCP to take advantage of the Microsoft global backbone network.
  - A number of connections are maintained for each of the backend endpoints.
  - Incoming client requests are first terminated at the edge node closest to the originating client.
  - After any required traffic inspection, requests are either forwarded over the Microsoft backbone to the appropriate backend using existing connections, or served from the internal cache of an edge node.
  - This approach is very efficient in spreading high traffic volumes over the backend connections.

- Provides a built-in cache that serves static content from edge nodes.
  - In many use cases this can also eliminate the need for a dedicated Content Delivery Network (CDN).

- Azure Web Application Firewall (WAF) can be used on Azure Front Door, and since it is deployed to Azure network edge locations around the globe, every incoming request delivered by Front Door in inspected at the network edge.

- Azure Front Door protects application endpoints against DDoS attacks using [Azure DDoS protection Basic](https://docs.microsoft.com/azure/frontdoor/front-door-ddos#integration-with-azure-ddos-protection-basic).
  - Azure DDoS Standard provides additional and more advanced protection and detection capabilities and can be added as an additional layer to Azure Front Door.

- Azure Front Door offers a fully managed certificate service.
  - Enables TLS connection security for endpoints without having to manage the certificate lifecycle.

- Azure Front Door Premium supports private endpoints, enabling traffic to flow from the internet directly onto Azure virtual networks.
  - This would eliminate the need of using public IPs on the VNet for making the backends accessible via Azure Front Door Premium.

- Azure Front Door relies on health probes and backend health endpoints (URLs) which are called on an interval basis to return a HTTP status code reflecting if the backend is operating normally, with a HTTP 200 (OK) response reflecting a healthy status.
  - As soon as a backend reflects an unhealthy status, from the perspective of a certain edge node, that edge node will stop sending requests there. Unhealthy backends are therefore transparently removed from traffic circulation without any delay.

- Supports HTTP/S protocols only.

- The Azure Front Door WAF and Application Gateway WAF provide a slightly different feature set, though both support built-in and custom rules and can be set to operate in either detection mode or prevention mode.

- The Front Door backend IP space may change, but Microsoft will ensure integration with [Azure IP Ranges and Service Tags](https://www.microsoft.com/download/details.aspx?id=56519)
  - It is possible to subscribe to Azure IP Ranges and Service Tags to receive notifications about any changes or updates.

- Azure Front Door supports various [load distribution configurations](https://docs.microsoft.com/azure/frontdoor/front-door-routing-methods):
  - Latency-based: the default setting which routes traffic to the "closest" backend from the client; based on request latency.
  - Priority-based: useful for active-passive setups, where traffic must always be sent to a primary backend unless it is not available.
  - Weighted: applicable for canary deployments in which a certain percentage of traffic is sent to a specific backend.
    - If multiple backends have the same weights assigned, latency-based routing is used.

- By default Azure Front Door uses latency-based routing which can lead to situations where some backends get a lot more incoming traffic then others, depending on where clients originate from.

- If a series of client requests must be handled by the same backend, [Session Affinity](https://docs.microsoft.com/azure/frontdoor/front-door-routing-methods#session-affinity) can be configured on the frontend.
  - It uses a client-side cookie to send subsequent requests to the same backend as the first request, provided the backend is still available.

**Azure Traffic Manager**

- Azure Traffic Manager is a DNS redirection service.
  - The actual request payload is not processed, but instead Traffic Manager returns the DNS name of one of the backends it the pool, based on configured rules for the selected traffic routing method.
  - The backend DNS name is then resolved to its final IP address which is subsequently directly called by the client.

- The DNS response is cached and re-used by the client for a specified Time-To-Live (TTL) period, and requests made during this period will go directly to the backend endpoint without Traffic Manager interaction.
  - Eliminates the extra connectivity step which provides cost benefits compared to Front Door.

- Since the request is made directly from the client to the backend service, any protocol supported by the backend can be leveraged.

- Similar to Azure Front Door, Azure Traffic Manager also relies on health probes to understand if a backend is healthy and operating normally. If another value is returned or nothing is returned, the routing service recognizes ongoing issues and will stop routing requests to that specific backend.
  - However, unlike with Azure Front Door this removal of unhealthy backends is not instantaneous since clients will continue to create connections to the unhealthy backend until the DNS TTL expires and a new backend endpoint is requested from the Traffic Manager service.
  - In addition, even when the TTL expires, there is no guarantee that public DNS servers will honor this value, so DNS propagation can actually take much longer to occur.
    - This means that traffic may continue to be sent to the unhealthy endpoint for a sustained period of time.  

**Azure Standard Load Balancer**

> As of December 2021, the Cross-Region Standard Load Balancer is available in preview with [technical limitations](https://docs.microsoft.com/azure/load-balancer/cross-region-overview#limitations) that prevent consideration in a Mission-Critical context.

> - Integration with AKS is no available: loss of connectivity will occur when deploying a cross-region load balancer with an AKS cluster as a backend.
> - Frontend IP configurations have to be public: internal frontend endpoints are not supported.

- Azure Standard Load Balancer provides a cross-region load distribution capability, which enables geo-redundant high-availability scenarios.
  - Offers instant global failover, load distribution across regions to the closest Azure region, and the ability to scale up/down behind a single endpoint.

- Cross-region load balancer is configured with a global frontend public IP address and uses health probes to monitor availability of regional load balancers.
  - If there is a regional failure, the affected load balancer is taken out of rotation until it becomes available again.

### Design recommendations

- Use Azure Front Door as the primary global traffic routing service for HTTP/S scenarios.
  - Azure Front Door is strongly advocated for HTTP/S workloads as it provides optimized traffic routing, transparent failover, private backend endpoints (with the Premium SKU), edge caching and integration with Web Application Firewall (WAF).

- For application scenarios where client control is possible, apply client side routing logic to consider failover scenarios where the primary global routing technology fails.
  - Two or more global routing technologies should be positioned in parallel for added redundancy, if single service SLA isn't sufficient. Client logic is required to route to the redundant technology in the event of a global service failure.
    - Two distinct URLs should be used, with one applied to each of the different global routing services to simplify the overall certificate management experience and routing logic in the event of a failover.
    - Prioritize the use of third-party routing technologies as the secondary failover service, since this will mitigate the largest number of global failure scenarios and the capabilities offered by industry leading CDN providers will allow for a consistent design approach.
    - Consideration should also be given to directly routing to a single regional stamp rather than a separate routing service. While this will result in a degraded level of service, it represents a far simpler design approach.

Following diagram illustrates a redundant global load balancer configuration with client failover using Azure Front Door as primary global load balancer.

[![Azure Mission-Critical Global Load Balancer Configuration](/docs/media/alwayson-global-routing.gif "Azure Mission-Critical Global Load Balancer Configuration")](./Networking.md)

> To truly mitigate the risk of global failures within the Azure platform, a multi-cloud active-active deployment approach should be considered, with active deployment stamps hosted across two or more cloud providers and redundant third-party routing technologies used for global routing.
> However, it is strongly recommended not to apply a multi-cloud approach since it introduces significant operational complexity, with different deployment stamp definitions and representations of operational health across the different cloud platforms. This complexity in-turn introduces numerous resiliency risks within the normal operation of the application, which far outweigh the hypothetical risks of a global platform outage.

- Whilst it is not recommended, for HTTP(s) workloads using Azure Traffic Manager for global routing redundancy to Azure Front Door, consider if WAF execution can be offloaded to Application Gateway for 'happy path traffic' flowing through Azure Front Door.
  - This will introduce an additional failure point to the standard ingress path, an additional critical-path component to manage and scale, and will also incur additional costs to ensure global high-availability. It will, however, greatly simplify the failure scenario by providing consistency between the 'happy' and 'un-happy' ingress paths through Azure Front Door and Azure Traffic Manager, both in terms of WAF execution but also private application endpoints.
  - The loss of edge caching in a failure scenario will impact overall performance, and this must be aligned with an acceptable level of service or mitigating design approach.
    - To ensure a consistent level of service, consider offloading edge caching to a third-party CDN provider for both 'happy' and 'un-happy' paths.

> Whilst it deviates from the Azure-native design principle, it is recommended to consider a third-party global routing service in place of two Azure global routing services, since this provides the maximum level of fault mitigation and the a more simple design approach since most industry leading CDN providers offer edge capabilities largely consistent with that offered by Azure Front Door.

**Azure Front Door**

- Use the Azure Front Door managed certificate service to enable TLS connections, and remove the need to manage certificate lifecycles.

- Use the Azure Front Door Web Application Firewall (WAF) to provide protection at the edge from common web exploits and vulnerabilities, such as SQL injection.

- Use the Azure Front Door built-in cache to serve static content from edge nodes.
  - In most cases this will also eliminate the need for a dedicated Content Delivery Network (CDN).

- Configure the application platform ingress points to [validate incoming requests through header based filtering](https://docs.microsoft.com/azure/frontdoor/front-door-faq#how-do-i-lock-down-the-access-to-my-backend-to-only-azure-front-door-) using the *X-Azure-FDID* to ensure all traffic is flowing through the configured Front Door instance.
  - Consider also configuring IP ACLing using Front Door Service Tags to validate traffic originates from the Azure Front Door backend IP address space and Azure infrastructure services. This will ensure traffic flows through Azure Front Door at a service level, but header based filtering will still be required to ensure the use of a configured Front Door instance.

- Define a custom TCP health endpoint to validate critical downstream dependencies within a regional deployment stamp, including data platform replicas, such as Cosmos DB in the example provided by the foundational reference implementation.
  - If one or more dependencies becomes unhealthy, the health probe should reflect this in the response returned so that the entire regional stamp can be taken out of circulation.

- Ensure health probe responses are logged and ingest all operational data exposed by Azure Front Door into the global Log Analytics workspace to facilitate a unified data synch and single operational view across the entire application.

- Unless the workload is extremely latency sensitive, spread traffic evenly across all considered regional stamps to most effectively use deployed resources.
  - To achieve this, set the ["Latency Sensitivity (Additional Latency)"](https://docs.microsoft.com/azure/frontdoor/front-door-backend-pool#load-balancing-settings) parameter to a value that is high enough to cater for latency differences between the different regions of the backends.
    - Ensure a tolerance that is acceptable to the application workload regarding overall client request latency.

- Do not enable Session Affinity unless it is required by the application, since it can have a negative impact the balance of traffic distribution.
  - With a fully stateless application, if the recommended Mission-Critical application design approach is followed, any request could be handled by any of the regional deployments.

**Azure Traffic Manager**

- Use Traffic Manager for non HTTP/S scenarios as a replacement to Azure Front Door.
  - Capability differences will drive different design decisions for cache and WAF capabilities, as well as TLS certificate management.

- WAF capabilities should be considered within each region for the Traffic Manager ingress path, using Azure Application Gateway.

- Configure a suitably low TTL value to optimize the time required to remove an unhealthy backend endpoint from circulation in the event that backend becomes unhealthy.

- Similar to with Azure Front Door, a custom TCP health endpoint should be defined to validate critical downstream dependencies within a regional deployment stamp, which should be reflected in the response provided by health endpoints.
  - However, in the case of Traffic Manager additional consideration should be given to service level regional failover (i.e. 'dog legging') to mitigate the potential delay associated with the removal of an unhealthy backend due to dependency failures, particularly if it is not possible to set a low TTL for DNS records.

- Consideration should be given to third-party CDN providers in order to achieve edge caching when using Azure Traffic Manager as a primary global routing service.
  - Where edge WAF capabilities are also offered by the third-party service, consideration should be given to simplify the ingress path and potentially remove the need for Application Gateway.

## Application delivery services

In addition to redundant global traffic routing, the network ingress path for a Mission-Critical application must also consider requisite application delivery services to ensure secure, reliable, and scalable ingress traffic.

This section will therefore build on recommendations from the previous global routing section by exploring key application delivery capabilities, considering relevant services such as Azure Standard Load Balancer, Azure Application Gateway, and Azure API Management.

### Design considerations

- TLS encryption is critical to ensure the integrity of inbound user traffic to an Mission-Critical application, with **TLS Offloading** applied only at the point of a stamp's ingress to decrypt incoming traffic.
  - TLS Offloading Requires the private key of the TLS certificate to decrypt traffic.

- A **Web Application Firewall** provides protection against common web exploits and vulnerabilities, such as SQL injection or cross site scripting, and is essential to achieve the maximum reliability aspirations of a Mission-Critical application.

- Azure WAF provides out-of-the-box protection against the top 10 OWASP vulnerabilities using managed rule sets.
  - Custom rules can also be added to extend the managed rule set.
  - Azure WAF can be enabled within either Azure Front Door, Azure Application Gateway, or Azure CDN (currently in public preview).
    - The features offered on each of the services differ slightly. For example, the Azure Front Door WAF provides rate limiting, geo-filtering and bot protection, which are not yet offered within the Application Gateway WAF. However, they all support both built-in and custom rules and can be set to operate in detection mode or prevention mode.
    - The roadmap for Azure WAF will ensure a consistent WAF feature set is provided across all service integrations.

- Third-party WAF technologies such as NVAs and advanced ingress controllers within Kubernetes can also be considered to provide requisite vulnerability protection.

- Optimal WAF configuration typically require fine tuning, regardless of the technology used.

  **Azure Front Door**

- Azure Front Door only accepts HTTP and HTTPS traffic, and will only process requests with a known `Host` header.
  - This protocol blocking helps to mitigate volumetric attacks spread across protocols and ports, as well as DNS amplification and TCP poisoning attacks.

- Azure Front Door is a global Azure resource so configuration is deployed globally to all [edge locations](https://docs.microsoft.com/azure/frontdoor/edge-locations-by-region).
  - Resource configuration can be distributed at a massive scale to handle hundreds of thousands of requests per second.
  - Updates to configuration, including routes and backend pools, are seamless and will not cause any downtime during deployment.

- Azure Front Door provides both a fully managed certificate service as well as a bring-your-own-certificate method for the client-facing SSL certificates.
  - The fully managed certificate service provides a simplified operational approach and helps to reduce complexity in the overall design by performing certificate management within a single area of the solution.

- Azure Front Door auto-rotates "Managed" certificates at least 60 days ahead of certificate expiration to protect against expired certificate risks.
  - If self-managed certificates are used, updated certificates should be deployed no later than 24 hours prior to expiration of the existing certificate, otherwise clients may receive expired certificate errors.

- Certificate updates will only result in downtime if Azure Front Door is switched between "Managed" and "Use Your Own Certificate".

- Azure Front Door is protected by Azure DDoS Protection Basic, which is integrated into Front Door by default. This provides always-on traffic monitoring, real-time mitigation, and also defends against common Layer 7 DNS query floods or Layer 3/4 volumetric attacks.
  - These protections help to maintain Azure Front Door availability even when faced with a DDoS attack.
    - Distributed Denial of Service (DDoS) attacks can render a targeted resource unavailable by overwhelming it with illegitimate traffic.

- Azure Front Door also provides WAF capabilities at a global traffic level, while Application Gateway WAF must be provided within each regional deployment stamp.
  - Capabilities include firewall rulesets to protect against common attacks, geo-filtering, address blocking, rate limiting, and signature matching.

  **Azure Load Balancer**

- The Azure Basic Load Balancer SKU is not backed by an SLA and has several capability constraints compared to the Standard SKU.

### Design Recommendations

- Perform TLS Offloading in as few places as possible in order to maintain security whilst simplifying the certificate management lifecycle.

- Use encrypted connections (e.g. HTTPS) from the point where TLS offloading occurs to the actual application backends.
  - Application endpoints will not be visible to end users, so Azure-managed domains, such as `azurewebsites.net` or `cloudapp.net`, can be used with managed certificates.

- For HTTP(S) traffic, ensure WAF capabilities are applied within the ingress path for all publicly exposed endpoints.

- Enable WAF capabilities at a single service location, either globally with Azure Front Door or regionally with Azure Application Gateway, since this simplifies configuration fine tuning and optimizes performance and cost.
  - Configure WAF in Prevention mode to directly block attacks.
    - Only use WAF in Detection mode (i.e. only logging but not blocking suspicious requests) when the performance penalty of Prevention mode is too high.
      - The implied additional risk must be fully understood and aligned to the specific requirements of the workload scenario.

- Prioritize the use of Azure Front Door WAF since it provides the richest Azure-native feature set and applies protections at the global edge, which simplifies the overall design and drives further efficiencies.

- Use Azure API Management only when exposing a large number of APIs to external clients or different application teams.

- Use the Azure Standard Load Balancer SKU for any internal traffic distribution scenario within micros-service workloads.
  - Provides an SLA of 99.99% when deployed across Availability Zones.
  - Provides critical capabilities such as diagnostics or outbound rules.

- Use Azure DDoS Protection Standard protection plans to help protect public endpoints hosted within each application virtual network.

## Virtual Network Integration

A Mission-Critical application will typically encompass requirements for integration with other applications or dependent systems, which could be hosted on Azure, another public cloud, or on-premises data centers. This application integration can be accomplished using public-facing endpoints and the internet, or private networks through network-level integration. Ultimately, the method by which application integration is achieved will have a significant impact on the security, performance, and reliability of the solution, as well as strongly impacting design decisions within other Mission-Critical design areas.

A Mission-Critical application can be deployed within one of three overarching network configurations, which determines how application integration can occur at a network level.

1. **Public** application **without** corporate network connectivity.
1. **Public** application **with** corporate network connectivity.
1. **Private** application **with** corporate network connectivity.

> Within an Enterprise-Scale context, configuration 1) should be deployed within an Online Landing Zone, while both 2) and 3) should be deployed within a Corp. Connected Landing Zone to facilitate network-level integration.

This section will therefore explore these network integration scenarios, layering in the appropriate use of Azure Virtual Networks and surrounding Azure networking services to ensure integration requirements are optimally satisfied.

### Design Considerations

**No Virtual Networks**

- The simplest design approach is to not deploy the application within a virtual network.
  - Connectivity between all considered Azure services will be provided entirely through public endpoints and the Microsoft Azure backbone.
    - Connectivity between public endpoints hosted on Azure will only traverse the Microsoft backbone and will not go over the public internet.
  - Connectivity to any external systems outside Azure will be provided by the public internet.

- This design approach adopts "identity as a security perimeter" to provide access control between the various service components and dependent solution.
  - While this may be an acceptable solution for scenarios which are less sensitive to security, having all application services and dependencies accessible via a public endpoint leaves them vulnerable to additional attack vectors orientated around gaining unauthorized access.

- This design approach is also not applicable for all Azure services, since many services, such as AKS, have a hard requirement for an underlying virtual network.

**Isolated Virtual Networks**

- To mitigate the risks associated with unnecessary public endpoints, a Mission-Critical solution can be deployed within a standalone network that is not connected to other networks.

- Incoming client requests will still require a public endpoint to be exposed to the internet, however, all subsequent communication can be within the virtual network using private endpoints.
  - When using Azure Front Door Premium, it is possible to route directly from edge nodes to private application endpoints.

- While private connectivity between application components will occur over virtual networks, all connectivity with external dependencies will still rely on public endpoints.
  - Connectivity to Azure platform services can be established via Private Endpoints if supported. If other external dependencies exist on Azure, such as another downstream application, connectivity will be provided provided through public endpoints and the Microsoft Azure backbone.
  - Connectivity to any external systems outside Azure would be provided by the public internet.
  
- For scenarios where there are no network integration requirements for external dependencies, deploying the solution within an isolated network environment provides maximum design flexibility.
  - No addressing and routing constraints associated with broader network integration.

- Azure Bastion is a fully platform-managed PaaS service that can be deployed on an virtual network and provides secure RDP/SSH connectivity to Azure VMs.
  - When you connect via Azure Bastion, virtual machines do not need a public IP address.

- The use of application virtual networks introduces significant deployment complexities within CI/CD pipelines, since both data plane and control plane access to resources hosted on private networks is required to facilitate application deployments.
  - Secure private network path must be established to allow CI/CD tooling to perform requisite actions.
  - Private build agents can be deployed within application virtual networks to proxy access to resources secured by the virtual network.

**Connected Virtual Networks**

- For scenarios with external network integration requirements, application virtual networks can be connected to other virtual networks within Azure, another cloud provider, or on-premises networks using a variety of connectivity options.
  - For example, some application scenarios might consider application-level integration with other line-of-business applications hosted privately within an on-premises corporate network.

- The application network design must align with the broader network architecture, particularly concerning topics such as addressing and routing.

- Overlapping IP address spaces across Azure regions or on-premises networks will create major contention when network integration is considered.
  - A virtual network resource can be updated to consider additional address space, however, when a virtual network address space of a peered network changes a [sync on the peering link is required](https://azure.microsoft.com/blog/how-to-resize-azure-virtual-networks-that-are-peered-now-in-preview/) which will temporarily disable peering.
  - Azure reserves five IP addresses within each subnet, which should be considered when determining appropriate sizes for application virtual networks and encompassed subnets.
  - Some Azure services require dedicated subnets, such as Azure Bastion, Azure Firewall, or Azure Virtual Network Gateway.
    - The size of these service subnets is very important, since they should be big enough to support all current instances of the service considering future scale requirements, but not so large as to unnecessarily waste addresses.

- When on-premises or cross-cloud network integration is required, Azure offers two different solutions to establish a secure connection.
  - An ExpressRoute circuit can be sized to provide bandwidths up to 100Gbps.
  - A Virtual Private Network (VPN) can be sized to provide aggregated bandwidth up to 10Gbps in hub and spoke networks, and up to 20Gbps in Azure Virtual WAN.

> In an Enterprise-Scale environment, the foundational platform will provide requisite connectivity to on-premises networks using ExpressRoute as well as other virtual networks in Azure using either Virtual WAN or a hub-and-spoke network design.

- The inclusion of additional network paths and resources introduces additional reliability and operational considerations for the Mission-Critical application to ensure health is maintained.

### Design Recommendations

- It is recommended that Mission-Critical solutions be deployed within Azure virtual networks where possible to remove unnecessary public endpoints, limiting the application attack surface to maximize security and reliability.
  - Use Private Endpoints for connectivity to Azure platform services.
    - Service Endpoints can be considered for services which do not support Private Link, provided data exfiltration risks are acceptable or mitigated through alternative controls.

- For application scenarios which do not require corporate network connectivity, treat all virtual networks as ephemeral resources that are replaced when a new regional deployment is conducted.

- When connecting to other Azure or on-premises networks, application virtual networks should not be treated as ephemeral since it creates significant complications where virtual network peering and virtual network gateway resources are concerned.
  - All relevant application resources within the virtual network should continue to be ephemeral, with parallel subnets used to facilitate blue-green deployments of updated regional deployment stamps.

- In scenarios where corporate network connectivity is required to facilitate application integration over private networks, ensure that the IPv4 address space used for regional application virtual networks does not overlap with other connected networks and is properly sized to facilitate required scale without needing to update the virtual network resource and incur downtime.
  - It is strongly recommended to only use IP addresses from the address allocation for private internets (RFC 1918).
    - For environments with a limited availability of private IP addresses (RFC 1918), consider using IPv6.
    - If the use of public IP address is required, ensure that only owned address blocks are used.
  - Align with organization plans for IP addressing in Azure to ensure that application network IP address space does not overlap with other networks across on-premises locations or Azure regions.
  - Do not create unnecessarily large application virtual networks to ensure that IP address space is not wasted.

- Prioritize the use Azure CNI for AKS network integration, since it [supports a richer feature set](https://docs.microsoft.com/azure/aks/concepts-network#compare-network-models).
  - Consider Kubenet for scenarios with a limited rage available IP addresses to fit the application within a constrained address space.

- For scenarios requiring on-premises network integration, prioritize the use Express Route to ensure secure and scalable connectivity.
  - Ensure the reliability level applied to the Express Route or VPN fully satisfies application requirements.
  - Multiple network paths should be considered to provide additional redundancy when required, such as cross connected ExpressRoute circuits or the use of VPN as a failover connectivity mechanism.

- Ensure all components on critical network paths are in line with the reliability and availability requirements of associated user flows, regardless of whether the management of these paths and associated component is delivered by the application team of central IT teams.

> When integrating with a broader organizational network topology, careful consideration should be given to the [Enterprise Scale Landing Zones Network Guidance](https://docs.microsoft.com/azure/cloud-adoption-framework/ready/enterprise-scale/network-topology-and-connectivity) to ensure the foundational network is aligned with Microsoft best-practices.

- Use [Azure Bastion](https://docs.microsoft.com/azure/bastion/bastion-overview) or proxied private connections to access the data plane of Azure resources or perform management operations.

## Internet Egress

Internet egress is a foundational network requirement for a Mission-Critical application to facilitate external communication in the context of:

1. Direct application user interaction.
1. Application integration with external dependencies outside Azure.
1. Access to external dependencies required by the Azure services leveraged by the application.

This section will therefore explore how internet egress can be achieved while ensuring security, reliability, and sustainable performance are maintained, highlighting key egress requirements for services recommended in a Mission-Critical context.

### Design Considerations

- Many Azure services require access to public endpoints for various management and control plane functions to operate as intended.

- Azure provides different direct internet outbound [connectivity methods](https://docs.microsoft.com/azure/load-balancer/load-balancer-outbound-connections#scenarios), such as Azure NAT gateway or Azure Load Balancer, for virtual machines or compute instances on a virtual network.

- When traffic from inside a virtual network travels out to the Internet, Network Address Translation (NAT) must take place.
  - This is a compute operation that occurs within the networking stack and that can therefore impact system performance.

- When NAT takes place at a small scale the performance impact should be negligible, however, if there are a large number of outbound requests network issues may occur.
  - These issues typically come in the form of 'Source NAT (or SNAT) port exhaustion'.

- In a multi-tenant environment, such as Azure App Service, there is a limited number of outbound ports available to each instance. If these ports run out, no new outbound connections can be initiated.
  - This issue can be mitigated by reducing the number of private/public edge traversals or by using a more scalable NAT solution such as the [Azure NAT Gateway](https://docs.microsoft.com/azure/virtual-network/nat-gateway/nat-overview).

- In addition to NAT limitations, outbound traffic may also be subject to requisite security inspections.
  - Azure Firewall provides appropriate security capabilities to secure network egress.

  - [Azure Firewall](https://docs.microsoft.com/azure/firewall/protect-azure-kubernetes-service) (or an equivelent NVA) can be used to secure Kubernetes egress requirements by providing granular control over outbound traffic flows.

- Large volumes of internet egress will incur [data transfer charges](https://azure.microsoft.com/pricing/details/bandwidth/).

**Azure NAT Gateway**

- Azure NAT Gateway supports 64,000 connections for TCP and UDP per assigned outbound IP address.
  - Up to 16 IP addresses can be assigned to a single NAT gateway.
  - A default TCP idle timeout of 4 minutes.
    - If idle timeout is altered to a higher value, flows will be held for longer which will increase the pressure on the SNAT port inventory.

- NAT gateway cannot provide zonal isolation out-of-the-box.
  - To get zonal redundancy, a subnet containing zonal resources must be aligned with corresponding zonal NAT gateways.

### Design Recommendations

- Minimize the number of outgoing Internet connections as this will impact NAT performance.
  - If large numbers of internet-bound connections are required, consider using [Azure NAT Gateway](https://docs.microsoft.com/azure/virtual-network/nat-gateway/nat-overview) to abstract outbound traffic flows.

- Use Azure Firewall where requirements to control and inspect outbound internet traffic exist.
  - Ensure Azure Firewall is not used to inspect traffic between Azure services.

> In an Enterprise-Scale context, consider using the foundational platform Azure Firewall resource (or equivalent NVA).
> - If a dependency is taken on a central platform resource for internet egress, then the reliability level of that resource and associated network path should be closely aligned with application requirements. Operational data from the resource should also be made available to the application in order to inform potential operational action in failure scenarios.

> If there are high-scale requirements associated with outbound traffic, consideration should be given to a dedicated Azure Firewall resource for a Mission-Critical application, to mitigate risks associated with using a centrally shared resource, such as noisy neighbor scenarios.
> - When deployed within a Virtual WAN environment, consideration should be given to Firewall Manager to provide centralized management of dedicated application Azure Firewall instances to ensure organizational security postures are observed through global firewall policies.
>   - Ensure incremental firewall policies are delegated to application security teams via role-based access control to allow for application policy autonomy.

## Inter-Zone and Inter-Region Connectivity

While the Azure Mission-Critical application design strongly advocates independent regional deployment stamps, many application scenarios may still require network integration between application components deployed within different zones or Azure regions, even if only under degraded service circumstances. The method by which inter-zone and inter-region communication is achieved has a significant bearing on overall performance and reliability, which will be explored through the considerations and recommendations within this section.

### Design Considerations

- The application design approach for a Mission-Critical application endorses the use of independent regional deployments with zonal redundancy applied at all component levels within a single region.

- An [Availability Zone (AZ)](https://docs.microsoft.com/azure/availability-zones/az-overview#availability-zones) is a physically separate data center location within an Azure region, providing physical and logical fault isolation up to the level of a single data center.
  - A round-trip latency of less than 2ms is guaranteed for inter-zone communication.
    - Zones will have a small latency variance given varied distances and fiber paths between zones.

- Availability Zone connectivity depends on regional characteristics, and therefore traffic entering a region via an edge location may need to be routed between zones to reach its destination.
  - This will add a ~1ms-2ms latency given inter-zone routing and 'speed of light' constraints, but this should only have a bearing for hyper sensitive workloads.

- Availability Zones are treated as logical entities within the context of a single subscription, so different subscriptions might have a different zonal mapping for the same region.
  - For example, zone 1 in Subscription A could correspond to the same physical data center as zone 2 in subscription B.

- Communication between zones within a region incurs a [data transfer charge](https://azure.microsoft.com/pricing/details/bandwidth/) per GB of bandwidth.

- With application scenarios that are extremely chatty between application components, spreading application tiers across zones can introduce significant latency and increased costs.
  - It is possible to mitigate this within the design by constraining a deployment stamp to a single zone and deploying multiple stamps across the different zones.

- Communication between different Azure regions incurs a larger [data transfer charge](https://azure.microsoft.com/pricing/details/bandwidth/) per GB of bandwidth.
  - The applicable data transfer rate depends on the continent of the considered Azure regions.
  - Data traversing continents is charged at a considerably higher rate.

- Express Route and VPN connectivity methods can also be used to directly connect different Azure regions together for certain scenarios, or even different cloud platforms.

- For services to service communication Private Link can be used for direct communication using private endpoints.

- Traffic can be hair-pinned through Express Route circuits used for on-premise connectivity in order to facilitate routing between virtual networks within an Azure region as well as across different Azure regions within the same geography.
  - Hair-pinning traffic through Express Route will bypass data transfer costs associated with virtual network peering, so can be used as a way to optimize costs.
  - This approach necessitates additional network hops for application integration within Azure, which introduces latency and reliability risks.
    - Expands the role of Express Route and associated gateway components from Azure/on-premises to also encompass Azure/Azure connectivity.

- When sub-millisecond latency are required between services, [Proximity Placement Groups](https://docs.microsoft.com/azure/virtual-machines/co-location) can be used when supported by the services used.

### Design Recommendations

- Use virtual network peering to connect networks within a region and across different regions.
  - It is strongly recommended to avoid hair-pinning within Express Route.

- Use Private Link to establish communication directly between services in the same region or across regions (service in Region A communicating with service in Region B.

- For application workloads which are extremely chatty between components, consider constraining a deployment stamp to a single zone and deploying multiple stamps across the different zones.
  - This ensures zonal redundancy is maintained at the level of an encapsulated deployment stamp rather than a single application component.

- Where possible, treat each deployment stamp as independent and disconnected from other stamps.
  - Use data platform technologies to synchronize state across regions rather than achieving consistency at an application level with direct network paths.
  - Avoid 'dog legging' traffic between different regions unless necessary, even in a failure scenario.
    - Use global routing services and end-to-end health probes to take an entire stamp out of circulation in the event that a single critical component tier fails, rather than routing traffic at that faulty component level to another region.

- For hyper latency sensitive application scenarios, prioritize the use of zones with regional network gateways to optimize network latency for ingress paths.

## Micro-Segmentation and Kubernetes Network Policies

Micro-segmentation is a network security design pattern used to isolate and secure individual application workloads, with policies applied to limit network traffic between workloads based on a Zero Trust model. It is typically applied to reduce network attack surface, improve breach containment, and strengthen security through policy-driven application-level network controls.

A Mission-Critical application can enforce application-level network security using Network Security Groups (NSG) at either a subnet or network interface level, service Access Control Lists (ACL), and network policies when using Azure Kubernetes Service (AKS). This section will therefore explore the optimal use of these capabilities, providing key considerations and recommendations to achieve application-level micro-segmentation.

### Design Considerations

- AKS can be deployed in two different [networking models](https://docs.microsoft.com/azure/aks/concepts-network#azure-cni-advanced-networking).
  - Kubenet networking: AKS nodes are integrated within an existing virtual network, but pods exist within a virtual overlay network on each node.
  - Azure Container Networking Interface (CNI) networking: the AKS cluster is integrated within an existing virtual network resources and treated as a single large network, with pods attributed their own IP address.

- By default, pods are non-isolated and accept traffic from any source and can send traffic to any destination; a pod can communicate with every other pod in a given Kubernetes cluster; Kubernetes does not ensure any network level isolation, and does not isolate namespaces at the cluster level.

- Pods can be isolated using [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/).
  - AKS provides two ways to implement Network Policy, and both implementations use Linux IPTables to enforce specified policies.
    - _Azure Network Policies_
    - _Calico Network Policies_
  - Network policies do not conflict since they are additive.
  - For a network flow between two pods to be allowed, both the egress policy on the source pod and the ingress policy on the destination pod need to allow the traffic.
  - The network policy feature can only be enabled at cluster instantiation time.
    - It is not possible to enable network policy on an existing AKS cluster.

The delivery of network policies is consistent regardless of whether Azure or Calico is used.
  - Calico provides a [richer feature set](https://docs.microsoft.com/azure/aks/use-network-policies#differences-between-azure-and-calico-policies-and-their-capabilities), including support for windows-nodes.
  - Calico introduces another party within the support ecosystem; either Calico community or paid support.
    - 1st-party Azure support is provided for Azure network policies.

> Network Policy is a Kubernetes specification that defines access policies for communication between Pods. Using Network Policies, an ordered set of rules can be defined to control how traffic is sent/received, and applied to a collection of pods that match one or more label selectors.

- AKS supports the creation of different node pools to separate different workloads using nodes with different hardware and software characteristics, such as nodes with and without GPU capabilities.
  - Using node pools does not provide any network-level isolation.
  - All node pools must reside within the same virtual network and separating node pools in different subnets is [currently in preview](https://github.com/Azure/AKS/issues/1338).
    - NSGs at the subnet can be applied to implement micro-segmentation between node pools.

### Design Recommendations

- Configure an NSG on all considered subnets to provide an IP ACL to secure ingress paths and isolate application components based on a Zero Trust model.
  - Use Front Door Service Tags within NSGs on all subnets containing application backends defined within Azure Front Door, since this will validate traffic originates from a legitimate Azure Front Door backend IP address space.
    - This will ensure traffic flows through Azure Front Door at a service level, but header based filtering will still be required to ensure the use of a particular Front Door instance and to also mitigate 'IP spoofing' security risks.
  - Public internet traffic should be disabled on RDP and SSH ports across all applicable NSGs.

- Enable [Network Policy](https://docs.microsoft.com/azure/aks/use-network-policies) for Azure Kubernetes Service at deployment time.
  - The Network Policy feature in Kubernetes should be used to define rules for ingress and egress traffic between pods in a cluster.
    - Define granular Network Policies to restrict and limit cross-pod communication.

- Prioritize the use of _Calico Network Policies_ because it provides a richer feature set with broader community adoption and support.

---

|Previous Page|Next Page|
|:--|:--|
 [Deployment and Testing](./Deployment-Testing.md) |[Security](./Security.md)
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
