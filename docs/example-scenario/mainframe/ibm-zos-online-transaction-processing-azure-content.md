Online transaction processing (OLTP) systems interact directly with users and are the face of the business. With a dynamically adaptable infrastructure, businesses can realize and launch their products quickly to delight their users.

## Architecture

The following diagram shows the architecture of the workload to be migrated, an OLTP system running on a z/OS mainframe:

:::image type="content" source="media/ibm-zos-online-transaction-processing-on-zos.svg" alt-text="OLTP architecture on z/OS" lightbox="media/ibm-zos-online-transaction-processing-on-zos.svg" border="false":::

*Download a [Visio file](https://arch-center.azureedge.net/ibm-zos-online-transaction-processing-on-zos.vsdx) of this architecture.*

### Workflow

The following workflow corresponds to the preceding diagram:

1. Users connect to the mainframe over TCP/IP using standard mainframe protocols like TN3270 and HTTPS.
1. The transaction managers interact with the users and invoke the application to satisfy user requests.
1. In the front end of the application layer, users interact with the CICS/IMS screens or with web pages.
1. The transaction managers use the business logic written in COBOL or PL/1 to implement the transactions.
1. Application code uses storage capabilities of the data layer, typically DB2, IMS DB, or VSAM.
1. Along with transaction processing, other services provide authentication, security, management, monitoring, and reporting. These services interact with all other services in the system.

Here, we see how to migrate this architecture to Azure.

:::image type="content" source="media/ibm-zos-online-transaction-processing-on-azure.svg" alt-text="Diagram that shows an architecture for migrating a z/OS OLTP workload." lightbox="media/ibm-zos-online-transaction-processing-on-azure.svg" border="false":::

*Download a [Visio file](https://arch-center.azureedge.net/ibm-zos-online-transaction-processing-on-azure.vsdx) of this architecture.*

1. Mainframe users are familiar with 3270 terminals and on-premises connectivity. In the migrated system, they interact with Azure applications via public internet or via a private connection implemented with Azure ExpressRoute. Microsoft Entra ID provides authentication.
1. Input requests go to a global load balancer service, like Azure Front Door or Azure Traffic Manager. The load balancer can serve a geographically spread user base. It routes the requests according to rules defined for the supported workloads. These load balancers can coordinate with Azure Application Gateway or Azure Load Balancer for load balancing of the application layer. The Azure Content Delivery Network service caches static content in edge servers for quick response, secured using the Web Application Firewall (WAF) service.
1. The front end of the application layer uses Azure services like Azure App Service to implement application screens and to interact with users. The screens are migrated versions of the mainframe screens.
1. COBOL and PL/1 code in the back end of the application layer implements the business logic. The code can use services like Azure Functions, WebJobs, and Azure Spring Apps microservices. Applications can run in an Azure Kubernetes Service (AKS) container.
1. An in-memory data store accelerates high-throughput OLTP applications. One such store is In-Memory OLTP, a feature of Azure SQL Database and Azure SQL Managed Instance. Another is Azure Cache for Redis.
1. The data layer can include, for example:

   1. Files, tables, and blobs implemented using Azure Storage services.
   1. Relational databases from the Azure SQL family.
   1. Azure implementations of the PostgreSQL and MySQL open-source databases.
   1. Azure Cosmos DB, a NoSQL database.

   These stores hold data migrated from the mainframe for use by the application layer.
1. Azure native services like Application Insights and Azure Monitor proactively monitor the health of the system. You can integrate the Monitor logs using an Azure dashboard.

### Components

This architecture consists of several Azure cloud services and is divided into four categories of resources: networking and identity, application, storage, and monitoring. The services for each and their roles are described in the following sections.

#### Networking and identity

When designing application architecture, it is crucial to prioritize networking and identity components to ensure security, performance, and manageability during interactions over public internet or private connections. The following components in the architecture are essential to addressing this requirement effectively.

- [Azure ExpressRoute](/azure/well-architected/service-guides/azure-expressroute) carries private connections between on-premises infrastructure and Azure datacenters.
- [Microsoft Entra ID](/entra/fundamentals/whatis) is an identity and access management service that can synchronize with an on-premises directory.
- [Azure Front Door](/azure/well-architected/service-guides/azure-front-door) provides global HTTP load balancing with instant failover. Its caching option can quicken delivery of static content.
- [Azure Traffic Manager](/azure/well-architected/service-guides/traffic-manager/reliability) directs incoming DNS requests based on your choice of traffic routing methods.
- [Azure Web Application Firewall](/azure/web-application-firewall/overview) helps protect web apps from malicious attacks and common web vulnerabilities, such as SQL injection and cross-site scripting.
- [Azure Content Delivery Network (CDN)](/azure/cdn/cdn-overview) caches static content in edge servers for quick response, and uses network optimizations to improve response for dynamic content. CDN is especially useful when the user base is global.
- [Azure Application Gateway](/azure/well-architected/service-guides/azure-application-gateway) is an application delivery controller service. It operates at layer 7, the application layer, and has various load-balancing capabilities.
- [Azure Load Balancer](/azure/well-architected/service-guides/azure-load-balancer/reliability) is a layer 4 (TCP, UDP) load balancer. In this architecture, it provides load balancing options for Spring Apps and AKS.

#### Application

Azure offers a managed to support the secure, scalable, and efficient deployment of applications. The application tier services cited in the above architecture can contribute to achieving optimal application architecture.

- [Azure API Management](/azure/well-architected/service-guides/api-management/reliability) supports the publishing, routing, securing, logging, and analytics of APIs. You can control how the data is presented and extended, and which apps can access it. You can restrict access to your apps, or allow third parties.
- [Azure App Service](/azure/well-architected/service-guides/app-service-web-apps) is a fully managed service for building, deploying, and scaling web apps. You can build apps using .NET, .NET Core, Node.js, Java, Python, or PHP. The apps can run in containers or on Windows or Linux. In a mainframe migration, the front-end screens or web interface can be coded as HTTP-based REST APIs. They can be segregated as per the mainframe application, and can be stateless to orchestrate a microservices-based system.
- WebJobs is a feature of Azure App Service that runs a program or script in the same instance as a web app, API app, or mobile app. A web job can be a good choice for implementing sharable and reusable program logic. For technical information, see [Run background tasks with WebJobs in Azure App Service](/azure/app-service/webjobs-create).
- [Azure Kubernetes Service (AKS)](/azure/well-architected/service-guides/azure-kubernetes-service) is a fully managed Kubernetes service for deploying and managing containerized applications. AKS simplifies deployment of a managed AKS cluster in Azure by offloading the operational overhead to Azure.
- [Azure Spring Apps](/azure/spring-apps/basic-standard/overview) is a fully managed Spring service, jointly built and operated by Microsoft and VMware. With it, you can easily deploy, manage, and run Spring microservices, and write Spring applications using Java or .NET.
- [Azure Service Bus](/azure/well-architected/service-guides/service-bus/reliability) is a reliable cloud messaging service for simple hybrid integration. Service Bus and Storage queues can connect the front end with the business logic in the migrated system.
- [Azure Functions](/azure/well-architected/service-guides/azure-functions-security) provides an environment for running small pieces of code, called functions, without having to establish an application infrastructure. You can use it to process bulk data, integrate systems, work with IoT, and build simple APIs and microservices. With microservices, you can create servers that connect to Azure services and are always up to date.
- [Azure Cache for Redis](/azure/well-architected/service-guides/azure-cache-redis/reliability) is a fully managed in-memory caching service for sharing data and state among compute resources. It includes both the open-source Redis (OSS Redis) and a commercial product from Redis Labs (Redis Enterprise) as a managed service. You can improve performance of high-throughput OLTP applications by designing them to scale and to make use of an in-memory data store such as Azure Cache for Redis.

#### Storage and Database

The architecture addresses scalable and secure cloud storage as well as managed databases for flexible and intelligent data management.

- [Azure Storage](/azure/well-architected/service-guides/storage-accounts/reliability) is a set of massively scalable and secure cloud services for data, apps, and workloads. It includes [Azure Files](/azure/well-architected/service-guides/azure-files), [Azure Table Storage](/azure/storage/tables/table-storage-overview), and [Azure Queue Storage](https://azure.microsoft.com/services/storage/queues). Azure Files is often an effective tool for migrating mainframe workloads.
- [Azure SQL](/azure/azure-sql/) is a family of SQL cloud databases that provides flexible options for application migration, modernization, and development. The family includes:
  - [SQL Server on Azure Virtual Machines](/azure/azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview)
  - [Azure SQL Managed Instance](/azure/well-architected/service-guides/azure-sql-managed-instance/reliability)
  - [Azure SQL Database](/azure/well-architected/service-guides/azure-sql-database-well-architected-framework)
  - [Azure SQL Edge](/azure/azure-sql-edge/overview)
- [Azure Cosmos DB](/azure/well-architected/service-guides/cosmos-db) is a fully managed NoSQL database service with open-source APIs for MongoDB and Cassandra. A possible application is to migrate mainframe non-tabular data to Azure.
- [Azure Database for PostgreSQL](/azure/well-architected/service-guides/postgresql) is a fully managed, intelligent, and scalable PostgreSQL that has native connectivity with Azure services.
- [Azure Database for MySQL](/azure/well-architected/service-guides/azure-db-mysql-cost-optimization) is a fully managed, scalable MySQL database.
- In-Memory OLTP is a feature of [Azure SQL Database](/azure/well-architected/service-guides/azure-sql-database-well-architected-framework) and [Azure SQL Managed Instance](/azure/well-architected/service-guides/azure-sql-managed-instance/reliability) that provides fast in-memory data storage. For technical information, see [Optimize performance by using in-memory technologies in Azure SQL Database and Azure SQL Managed Instance](/azure/azure-sql/in-memory-oltp-overview).

#### Monitoring

The monitoring tools outlined below provide comprehensive data analysis and valuable insights into application performance.

- [Azure Monitor](/azure/azure-monitor/overview) collects, analyzes, and acts on personal data from your Azure and on-premises environments.
- [Log Analytics](/azure/well-architected/service-guides/azure-log-analytics) is a tool in the Azure portal used to query Monitor logs using a powerful query language. You can work with the results of your queries interactively or use them with other Azure Monitor features such as log query alerts or workbooks. For more information, see [Overview of Log Analytics in Azure Monitor](/azure/azure-monitor/logs/log-analytics-overview).
- [Application Insights](/azure/well-architected/service-guides/application-insights) is a feature of Monitor that provides code-level monitoring of application usage, availability, and performance. It monitors the application, detects application anomalies such as mediocre performance and failures, and sends personal data to the Azure portal. You can also use Application Insights for logging, distributed tracing, and custom application metrics.
- Azure Monitor Alerts are a feature of Monitor. For more information, see [Create, view, and manage metric alerts using Azure Monitor](/azure/azure-monitor/alerts/alerts-metric).

## Scenario details

With ever-evolving business needs and data, applications must produce and scale without creating infrastructure issues. This example workload shows how you can migrate a z/OS mainframe OLTP application to a secure, scalable, and highly available system in the cloud, by using Azure platform as a service (PaaS) services. Such a migration helps businesses in finance, health, insurance, and retail to minimize application delivery timelines, and it helps reduce the costs of running the applications.

### Potential use cases

This architecture is ideal for OLTP workloads that have these characteristics:

- They serve an international user base.
- Their usage varies greatly over time, so they benefit from flexible scaling and usage-based pricing.

## Considerations

These considerations implement the pillars of the Azure Well-Architected Framework, which is a set of guiding tenets that can be used to improve the quality of a workload. For more information, see [Microsoft Azure Well-Architected Framework](/azure/well-architected/).

### Reliability

Reliability ensures your application can meet the commitments you make to your customers. For more information, see [Design review checklist for Reliability](/azure/well-architected/reliability/checklist).

- This OLTP architecture can be deployed in multiple regions and can have a geo-replicated data layer.
- The Azure database services support zone redundancy and can fail over to a secondary node if an outage occurs, or to allow for maintenance activities.

### Security

Security provides assurances against deliberate attacks and the abuse of your valuable data and systems. For more information, see [Design review checklist for Security](/azure/well-architected/security/checklist).

- ExpressRoute creates a private connection to Azure from an on-premises environment. You can also use site-to-site VPN.
- Microsoft Entra ID can authenticate resources and control access using Azure role-based access control.
- Database services in Azure support various security options like data encryption at rest.
- For general guidance on designing secure solutions, see [Overview of the security pillar](/azure/architecture/framework/security/overview).

### Cost Optimization

Cost Optimization is about looking at ways to reduce unnecessary expenses and improve operational efficiencies. For more information, see [Design review checklist for Cost Optimization](/azure/well-architected/cost-optimization/checklist).

Use the [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator) to estimate costs for your implementation.

### Operational Excellence

Operational Excellence covers the operations processes that deploy an application and keep it running in production. For more information, see [Design review checklist for Operational Excellence](/azure/well-architected/operational-excellence/checklist).

- This scenario uses Azure Monitor and Application Insights to monitor the health of the Azure resources. You can set alerts for proactive management.
- For guidance on resiliency in Azure, see [Designing reliable Azure applications](/azure/architecture/framework/resiliency/app-design).

### Performance Efficiency

Performance Efficiency is the ability of your workload to scale to meet the demands placed on it by users in an efficient manner. For more information, see [Design review checklist for Performance Efficiency](/azure/well-architected/performance-efficiency/checklist).

- This architecture uses Azure PaaS services like App Service, which has autoscaling capabilities.
- For guidance on autoscaling in Azure, see [Autoscaling](../../best-practices/auto-scaling.md).

## Contributors

*This article is maintained by Microsoft. It was originally written by the following contributors.*

Principal author:

- [Ashish Khandelwal](https://www.linkedin.com/in/ashish-khandelwal-839a851a3/) | Principal Engineering Architecture Manager
- [Nithish Aruldoss](https://www.linkedin.com/in/nithish-aruldoss-b4035b2b) | Engineering Architect

*To see non-public LinkedIn profiles, sign in to LinkedIn.*

## Next steps

- For more information, contact [datasqlninja@microsoft.com](mailto:datasqlninja@microsoft.com).
- [Azure Database Migration Guides](/data-migration)

## Related resources

See the following related architectures and related technical information:

### Related architectures

- [High-volume batch transaction processing](./process-batch-transactions.yml)
- [IBM z/OS mainframe migration with Avanade AMT](./avanade-amt-zos-migration.yml)
- [Micro Focus Enterprise Server on Azure VMs](./micro-focus-server.yml)
- [Refactor IBM z/OS mainframe Coupling Facility (CF) to Azure](../../reference-architectures/zos/refactor-zos-coupling-facility.yml)
- [Replicate and sync mainframe data in Azure](../../reference-architectures/migration/sync-mainframe-data-with-azure.yml)
- [Migrate IBM mainframe applications to Azure with TmaxSoft OpenFrame](../../solution-ideas/articles/migrate-mainframe-apps-with-tmaxsoft-openframe.yml)

### Related technical information

- [Run background tasks with WebJobs in Azure App Service](/azure/app-service/webjobs-create)
- [Optimize performance by using in-memory technologies in Azure SQL Database and Azure SQL Managed Instance](/azure/azure-sql/in-memory-oltp-overview)
- [Azure Monitor overview](/azure/azure-monitor/overview)
- [Create, view, and manage metric alerts using Azure Monitor](/azure/azure-monitor/alerts/alerts-metric)
- [Create and share dashboards of Log Analytics data](/azure/azure-monitor/visualize/tutorial-logs-dashboards)
- [Overview of the security pillar](/azure/architecture/framework/security/overview)
