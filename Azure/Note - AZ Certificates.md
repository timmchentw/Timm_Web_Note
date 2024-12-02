# AZ Certificates

## AZ900

### Costs

#### [Azure reservations](https://portal.azure.com/#view/Microsoft_Azure_Reservations/ReservationsBrowseBlade/productType/Reservations)

* Discount
* One-year or three-year plans
* Multiple products, (such as virtual machines, SQL databases, Azure Cosmos DB, and others)
* Doesn’t affect the runtime state of resources
* Can be exchanged or refunded for another reservation of the same type, region, or term

#### [TCO Calculator](https://azure.microsoft.com/en-us/pricing/tco/calculator/)

* TCO: Total Cost of Ownership
* Estimate the cost savings you can achieve by migrating your application workloads to Microsoft Azure
* Provides a detailed breakdown of the costs and benefits of each scenario

#### Scale

* Scale vertically: adding RAM or CPUs to a virtual machine
* Scaling horizontally: adding instances of resources, such as adding virtual machines to the configuration.

### Manage

#### Azure Active Directory (AAD)

* Conditional Access: Microsoft Entra uses to allow or deny access to resources based on identity signals, such as the device being used
* Hybrid identity: create a common user identity for authentication and authorization to all resources, regardless of location.

#### Azure Policy

* Enforce organizational standards and assess compliance at-scale
* Restrict deployment location
* Require for MFA (a user is prompted during the sign-in process for an additional form of identification.)
* Respond to policy evaluation: deny, audit, append, modify, or deploy...

#### Azure Cloud Shell

* Azure CLI
* Azure PowerShell (interactive, browser-accessible)

#### Azure Monitor

* Trigger autoscaling
* Health advisories

#### Azure Resource Manager (ARM)

* Manage resources

#### [Purview governance portal](https://web.purview.azure.com/)

* Manage and govern data across organization

### Network

#### VPN Gateway

* Connecting an on-premises datacenter to an Azure virtual network

#### Virtual network peering

* Link virtual networks

#### ExpressRoute

* Create a connection between on-premises network and the Microsoft cloud in four different ways,
* CloudExchange Co-location, Point-to-point Ethernet Connection, Any-to-any (IPVPN) Connection, and ExpressRoute Direct

#### Azure Bastion

* Remotely administer Azure virtual machines by using SSH/RDP

#### Azure Firewall 

* Stateful firewall service used to protect virtual networks

### Storage

#### Storage type

* Archive storage: Stores data offline and offers the lowest storage costs, but also the highest costs to rehydrate and access data.
* Hot storage: Optimized for storing data that is accessed frequently.
* Cool access: tolerate slightly lower availability, but still requires high durability, retrieval latency, and throughput characteristics similar to hot data.

#### Azure Files

* Accessible via industry-standard SMB and NFS protocols.


### Cloud computing

#### Consumption-based model

* No upfront costs
* Stop paying for resources that are no longer used

#### On-premises Computing

* Physical machines on site

#### Nouns

* Agility: deploy and configure cloud-based resources quickly as app requirements change.
* Scalability: add RAM, CPU, or entire virtual machines to a configuration.
* Elasticity: configure cloud-based apps to take advantage of autoscaling, so apps always have the resources they need. 
* High availability: cloud-based apps can provide a continuous user experience with no apparent downtime, even when things go wrong.
* SLA: The service-level agreement for high availability