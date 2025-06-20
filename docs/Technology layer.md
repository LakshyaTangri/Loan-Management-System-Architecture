# 04 - Technology Layer Architecture

The Technology Layer describes the underlying infrastructure software and hardware, including cloud services, that support the Application Layer. This section aligns with Phase C (Information Systems Architecture - Technology) and Phase D (Technology Architecture) of TOGAF ADM.

## 4.1 Azure Infrastructure Services

The LMS is deployed entirely on Microsoft Azure, leveraging a wide range of Platform-as-a-Service (PaaS) and Infrastructure-as-a-Service (IaaS) components.

### 4.1.1 Compute Services

* **Azure Kubernetes Service (AKS):** Hosts the microservices (Application Service, Credit Scoring Service, Decision Engine, Disbursement, Payment Processing, Collections, Document Management, Notification Services). Provides container orchestration, scaling, and self-healing capabilities.
* **Azure Functions:** Serverless compute for event-driven workflows (Background Check, Payment Scheduler, Report Generation, Data Synchronization). Eliminates server management and scales automatically based on demand.
* **Azure App Service:** Hosts the Customer Portal and Loan Officer Portal web applications. Provides a fully managed platform for web application deployment.
* **Azure Static Web Apps:** Hosts static content for the Customer Portal, offering global distribution and fast delivery.

### 4.1.2 Data Services

* **Azure SQL Database:** Managed relational database service for core transactional data (e.g., loan details, customer profiles, payment history). Provides high availability, backups, and scaling.
* **Azure Cosmos DB:** Globally distributed, multi-model database service (NoSQL) for high-performance and flexible data storage (e.g., audit logs, session data, document metadata).
* **Azure Storage Accounts (Blob Storage):** Scalable and cost-effective object storage for unstructured data such as uploaded customer documents (e.g., PDFs, images).
* **Azure Key Vault:** Secure storage for sensitive information like API keys, database connection strings, and certificates used by applications and services.

### 4.1.3 Networking and Connectivity

* **Azure Virtual Network (VNet):** Provides a secure and isolated network environment for all Azure resources.
    * **Subnets:** Dedicated subnets for AKS, Azure Functions, App Services, and databases to enforce network segmentation.
* **Azure Private Link:** Enables private connectivity to PaaS services (e.g., Azure SQL Database, Key Vault) from within the VNet, keeping traffic off the public internet.
* **Azure Application Gateway (with WAF):** Acts as a web traffic load balancer and application firewall (WAF) protecting the Customer Portal and API Gateway from common web vulnerabilities.
* **Azure Load Balancer:** Used internally by AKS to distribute traffic among pods.
* **Azure DNS:** Provides reliable and secure DNS services for domain resolution within Azure and for external access.

### 4.1.4 Integration and Messaging

* **Azure Service Bus:** Enterprise messaging broker for asynchronous communication between microservices (queues and topics). Ensures reliable message delivery and supports event-driven architectures.
* **Azure API Management:** Provides a centralized API gateway for publishing, securing, and managing APIs exposed by backend services. Facilitates integration with external systems.
* **Azure Event Grid:** Event routing service for reacting to events from other Azure services (e.g., new document uploaded to Blob Storage).

### 4.1.5 Security, Identity, and Management

* **Azure Active Directory (AAD):** Centralized identity and access management for users (loan officers, administrators) and applications. Supports Single Sign-On (SSO) and Multi-Factor Authentication (MFA).
* **Azure Policy:** Enforces organizational standards and assesses compliance at scale. Used to ensure naming conventions, resource tagging, and security configurations.
* **Azure Security Center / Microsoft Defender for Cloud:** Provides unified security management and advanced threat protection across hybrid cloud workloads.
* **Azure Monitor:** Collects and analyzes telemetry data (metrics, logs) from all Azure resources, providing insights into application performance and operational health.
* **Azure Log Analytics Workspace:** Centralized repository for all logs collected by Azure Monitor for querying and analysis.
* **Azure Backup:** Service for backing up and restoring data in Azure.

## 4.2 Disaster Recovery and High Availability

* **Zone Redundancy:** All critical PaaS services (Azure SQL DB, Service Bus, Cosmos DB) are deployed with zone redundancy where available, spreading instances across multiple availability zones within an Azure region.
* **Regional Paired Regions:** The overall architecture supports active-passive or active-active deployment across Azure paired regions for disaster recovery, with data replication and failover mechanisms in place.
* **Automated Backups:** Azure services provide automated backups with configurable retention policies.
* **Load Balancing:** Azure Load Balancer and Application Gateway distribute traffic, ensuring no single point of failure at the network ingress.
* **AKS Resilience:** AKS inherently provides high availability for control plane components and allows for replica sets for application pods, ensuring service continuity.

## 4.3 Network Security

* **Network Security Groups (NSGs):** Used to filter network traffic to and from Azure resources in a VNet.
* **Azure Firewall:** Provides centralized network security control for all outbound traffic from the VNet and can filter inbound traffic from external networks.
* **DDoS Protection:** Azure DDoS Protection Standard is enabled for the VNet to safeguard against large-scale denial-of-service attacks.
* **Service Endpoints / Private Endpoints:** Used to secure connectivity to Azure PaaS services, ensuring traffic flows directly over the Azure backbone network.