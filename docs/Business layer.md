# 02 - Business Layer Architecture

The Business Layer describes the organizational structure, business processes, and business services provided by the Loan Management System. This section aligns with Phase B (Business Architecture) of TOGAF ADM.

## 2.1 Business Actors and Roles

The following key business actors and their roles are involved in the LMS:

| Actor / Role          | Description                                                                 |
| :-------------------- | :-------------------------------------------------------------------------- |
| **Customer** | Applies for loans, submits documents, monitors application status, makes payments. |
| **Loan Officer** | Reviews applications, assesses risk, approves/rejects loans, manages customer interactions. |
| **Underwriter** | Performs detailed credit analysis and risk assessment for complex cases.      |
| **Collections Agent** | Manages overdue payments, communicates with customers, initiates recovery actions. |
| **Compliance Officer**| Ensures adherence to regulatory requirements, generates compliance reports.     |
| **Payment Gateway** | External system facilitating financial transactions (disbursement, repayments). |
| **Credit Bureau** | External system providing credit history and scores.                       |
| **Core Banking System**| External system for financial ledger and account management.             |

## 2.2 Business Processes

The LMS supports the full loan lifecycle, broken down into the following core business processes:

### 2.2.1 Loan Origination
* **Description:** The process of capturing and validating new loan applications.
* **Sub-processes:**
    * **Application Submission:** Customer fills out and submits loan application form via portal.
    * **Identity Verification (KYC):** System verifies customer identity and validates submitted documents.
    * **Initial Data Validation:** Automated checks on application completeness and basic eligibility.

### 2.2.2 Credit Assessment
* **Description:** Evaluating the applicant's creditworthiness and risk profile.
* **Sub-processes:**
    * **Credit Bureau Inquiry:** System queries external credit bureaus for applicant's credit score and history.
    * **Affordability Assessment:** System calculates the applicant's ability to repay the loan based on income and expenses.
    * **Risk Scoring:** Automated calculation of a risk score based on credit data and internal policies.

### 2.2.3 Loan Approval
* **Description:** Decision-making process leading to approval or rejection of a loan.
* **Sub-processes:**
    * **Automated Decisioning:** For low-risk, straightforward applications.
    * **Manual Underwriting Review:** For complex or high-risk applications, involving Loan Officers and Underwriters.
    * **Offer Generation:** If approved, system generates a loan offer with terms and conditions.
    * **Offer Acceptance:** Customer reviews and accepts/rejects the loan offer.

### 2.2.4 Disbursement
* **Description:** Transferring the approved loan funds to the customer.
* **Sub-processes:**
    * **Funding Request:** System initiates a request to the Core Banking System for fund transfer.
    * **Funds Transfer:** Core Banking System (via Payment Gateway) transfers funds to customer's account.
    * **Disbursement Confirmation:** System updates loan status upon successful transfer.

### 2.2.5 Loan Servicing
* **Description:** Managing the loan account throughout its lifecycle after disbursement.
* **Sub-processes:**
    * **Payment Processing:** Receiving and posting customer loan repayments.
    * **Interest Calculation:** Daily/monthly calculation of interest due.
    * **Balance Management:** Tracking outstanding principal, interest, and fees.
    * **Statement Generation:** Generating periodic loan statements for customers.
    * **Customer Inquiries:** Handling customer questions regarding their loan account.

### 2.2.6 Collections
* **Description:** Managing overdue loan payments and recovery actions.
* **Sub-processes:**
    * **Delinquency Detection:** System identifies overdue payments.
    * **Automated Reminders:** Sending automated notifications (SMS, email) to customers.
    * **Collections Workflow:** Assigning cases to collections agents for manual follow-up.
    * **Debt Recovery:** Initiating legal or external recovery processes if needed.

### 2.2.7 Compliance and Reporting
* **Description:** Ensuring regulatory adherence and providing analytical insights.
* **Sub-processes:**
    * **Regulatory Reporting:** Generating reports for financial authorities (e.g., anti-money laundering, credit reporting).
    * **Audit Trail Generation:** Maintaining immutable records of all loan transactions and decisions.
    * **Performance Reporting:** Generating business intelligence reports on loan portfolio performance.

## 2.3 Business Services

These are the capabilities exposed by the business layer to internal or external stakeholders:

* **Loan Application Service:** Allows customers to submit loan applications.
* **Credit Checking Service:** Provides creditworthiness assessment.
* **Loan Approval Service:** Manages the loan decision process.
* **Loan Disbursement Service:** Facilitates fund transfer to customers.
* **Payment Management Service:** Handles loan repayments and reconciliation.
* **Account Inquiry Service:** Allows customers to view their loan details.
* **Collections Management Service:** Manages overdue loans and recovery.
* **Compliance Reporting Service:** Generates required regulatory reports.
* **Customer Communication Service:** Manages notifications and correspondence.

docs/03_Application_Layer.md
# 03 - Application Layer Architecture

The Application Layer describes the applications, application components, and their interfaces that implement the business processes and provide services. This section aligns with Phase C (Information Systems Architecture - Application) of TOGAF ADM.

## 3.1 Application Components

The LMS application is designed as a set of microservices, primarily deployed on Azure Kubernetes Service (AKS) and Azure Functions, interacting through APIs and message queues.

* **Customer Portal:** A web and mobile application allowing customers to apply for loans, upload documents, track application status, view loan details, and make payments.
    * *Implementation:* Azure App Service (for web front-end), Azure Static Web Apps (for static content), Mobile App (iOS/Android).
    * *Integrates with:* API Gateway.
* **Loan Officer Portal:** A back-office web application for loan officers to manage applications, review documents, perform approvals, and interact with customers.
    * *Implementation:* Azure App Service.
    * *Integrates with:* API Gateway.
* **API Gateway (Azure API Management):** Acts as the single entry point for all application APIs, handling authentication, authorization, rate limiting, and routing requests to backend microservices.
* **Microservices (AKS Hosted):**
    * **Application Service:** Manages loan application state, data persistence, and workflows.
    * **Credit Scoring Service:** Calculates credit scores based on internal rules and external data.
    * **Decision Engine Service:** Implements business rules for automated loan approval/rejection.
    * **Disbursement Service:** Orchestrates fund transfers to the customer's bank account.
    * **Payment Processing Service:** Handles incoming loan repayments, reconciles transactions, and updates loan balances.
    * **Collections Service:** Manages overdue loans, dunning schedules, and collection agent workflows.
    * **Document Management Service:** Stores and retrieves application documents (e.g., ID, income proofs).
    * **Notification Service:** Manages SMS, email, and in-app notifications to customers and staff.
* **Azure Functions (Serverless):** Used for event-driven processing and scheduled tasks.
    * **Background Check Function:** Triggered by new applications to call external credit bureaus.
    * **Payment Scheduler Function:** Triggers payment reminders and processes scheduled payments.
    * **Report Generation Function:** Periodically generates compliance and operational reports.
    * **Data Synchronization Function:** Synchronizes data with the Core Banking System.
* **Data Services:**
    * **Azure SQL Database:** Relational database for core loan data, customer profiles, and transactional records (Application Service, Payment Processing Service, Collections Service).
    * **Azure Cosmos DB:** NoSQL database for flexible data storage, e.g., document management metadata, audit logs, user profiles for portals.
    * **Azure Storage Accounts:** Blob storage for storing raw documents (e.g., PDF, images) uploaded by customers.

## 3.2 Interfaces and APIs

All inter-service communication and external integrations are facilitated through well-defined APIs.

* **RESTful APIs:**
    * Exposed via Azure API Management for Customer and Loan Officer portals, and external partners (e.g., Credit Bureau for data push, Payment Gateway for status updates).
    * Internal APIs between microservices (e.g., Application Service calling Credit Scoring Service).
* **Event-Driven APIs (Azure Service Bus):**
    * Used for asynchronous communication between microservices to ensure loose coupling and resilience (e.g., 'Application Submitted' event, 'Payment Received' event).
    * Enables fan-out scenarios where multiple services react to the same event.
* **Integration Interfaces:**
    * **Credit Bureau Integration:**
        * Outbound: REST API calls from Background Check Function to Credit Bureau API.
        * Inbound: Webhook/Callback from Credit Bureau for real-time updates (via API Management).
    * **Payment Gateway Integration:**
        * Outbound: REST API calls from Disbursement Service and Payment Processing Service to initiate transactions.
        * Inbound: Webhook/Callback from Payment Gateway for transaction status updates (via API Management).
    * **Core Banking System Integration:**
        * Data Synchronization: Azure Function pushing/pulling data via secure API or file transfer (SFTP/Azure Data Factory).
        * Disbursement/Collections Ledger: Direct integration with Disbursement/Collections Services.

## 3.3 Application Cooperation

The applications and components cooperate as follows:

1.  **Customer Portal** submits loan application via **API Gateway** to **Application Service**.
2.  **Application Service** stores data in **Azure SQL DB** and publishes 'Application Submitted' event to **Azure Service Bus**.
3.  **Background Check Function** subscribes to 'Application Submitted' event, calls **Credit Bureau API**, and posts results back to **Application Service**.
4.  **Decision Engine Service** retrieves application data from **Application Service** and credit scores, then makes a decision.
5.  If approved, **Disbursement Service** receives an event, interacts with **Payment Gateway API** and **Core Banking System** to disburse funds.
6.  **Payment Processing Service** listens for payment events from **Payment Gateway** and updates **Azure SQL DB**.
7.  **Collections Service** monitors loan status in **Azure SQL DB** and initiates collection workflows based on delinquency.
8.  **Notification Service** sends communications based on events from various services.

This architecture emphasizes decoupling, asynchronous communication, and leveraging Azure's managed services for reliability and scalability.