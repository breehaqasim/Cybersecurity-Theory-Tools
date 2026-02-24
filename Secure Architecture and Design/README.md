# Secure Architecture and OWASP-Based Threat Modeling of an Online Payment Processing Application
## Task 1. System Definition and Architecture 
### System Overview
This system helps customers pay merchants online in a way. It allows for payment checks, refunds and handling problems with payments. It also helps with getting payments settled with banks.

The design is layered. Can work with different cloud systems. It connects with a payment processor and a core banking system. The platform deals with money and personal info so it is built to make sure:
- Confidentiality (protection of sensitive data)
- Integrity (accurate transaction processing)
- Availability (continuous payment operations)
- Accountability (auditability of financial actions)

Both external threats (internet-based attacks) and insider threats (misuse of administrative privileges) are considered in the architecture.

### Application Components
**Frontend**
- Customer Web Application (checkout, order history, payment status)
- Merchant Portal (transaction viewing, refund requests, settlement reports)
- Admin Portal (refund approvals, dispute handling, monitoring dashboards)
All frontend applications communicate with backend services over HTTPS.

**Backend Services**
- API Gateway (authentication, authorization, rate limiting, request validation, entry point for all external traffic)
- API Backend (business logic, order management, validation rules)
- Payment Service (payment orchestration, token handling, communication with payment processor)
- Webhook Handler (receives payment confirmation callbacks, updates transaction state)
- Admin Service (refund approvals, dispute handling, financial adjustments)

**Data Storage (Private Zone)**
All databases are located in a restricted internal network zone.
- User Database (customer accounts, credentials)
- Merchant Database (merchant profiles, API credentials)
- Transaction/Billing Database (orders, payments, refunds records, settlement details)
- Audit Log Store (immutable logs, security events, administrative actions)

**External Integrations**
- Payment Processor API (authorization, capture, tokenization, payment confirmation callbacks)
- Core Banking System (settlement and reconciliation)
- Notification Service (email/SMS alerts)

**Admin Access Paths**
- Admin Portal accessible only through VPN
- Multi-factor authentication required
- Separate privilege plane from customer APIs
- Strict role-based access control

### Users and Roles
| Role                  | Description                                      |
|-----------------------|--------------------------------------------------|
| Customer              | Initiates payments and views order history       |
| Merchant              | Views transactions and requests refunds          |
| Support Agent         | Provides limited customer support assistance     |
| Finance Admin         | Approves refunds and manages settlements         |
| System Administrator  | Manages infrastructure and deployments           |
| External Attacker     | Internet-based adversary attempting exploitation |
| Insider Threat        | Malicious or compromised internal user           |

Administrative roles have elevated access and require additional controls.

### Data Types Handled
| Data Type               | Sensitivity Level        |
|-------------------------|--------------------------|
| Credit Card Data        | 🔴 Highly Sensitive       |
| Payment Tokens          | 🔴 Highly Sensitive       |
| Customer Personal Data  | 🟡 Sensitive              |
| Merchant Credentials    | 🔴 Highly Sensitive       |
| Transaction Records     | 🟡 Sensitive              |
| JWT / Session Tokens    | 🟡 Sensitive              |
| Audit Logs              | 🟠 Moderate to Sensitive  |

### High-Level Architecture Diagram (Logical View)
Below is the logical architecture of the system:
![High-Level Architecture Diagram](high%20level%20diagram.png)

### Trust Boundaries
**Trust Boundary 1 – Internet ↔ Payment Platform**
Description: This boundary seperates us from the rest. It is the line between public internet traffic and trusted internal production services. The people who use our website in our case are Merchant, Customer and Admin all start from outside where we make things. Any request that comes into our system has to go through the API Gateway. The API Gateway is like a door that everyone has to go through to get in. This is way to get into our system so we can keep an eye on what is happening. 

Why It Is a Trust Boundary? This is because external attackers might try to enter data. Credentials and tokens are transmitted. Traffic moves, from a domain we do not trust into one we do trust.

**Trust Boundary 2 – Frontend ↔ Backend Services**
Description: Frontend and Backend services are seperate from each other. Frontend can access Database, Payment Service and Admin Service through API Gateway only.  

Why It Is a Trust Boundary? Prevents direct database access. Enforces centralized routing and request validation. The presentation layer and the business logic are kept separate, from each other.

**Trust Boundary 3 – Admin Access Path**
Description: Administrative actions (refund approvals, disputes, financial adjustments) occur through a separate privilege plane. Although Admin Portal is part of frontend, it represents elevated access compared to Customer and Merchant.

Why It Is a Trust Boundary? Admin roles have higher privileges. Administrative actions can modify financial records. Compromise of this boundary can lead to data integrity issues.

**Trust Boundary 4 – Production Zone ↔ Data Storage Zone**
Description: All databases are located in a restricted internal zone. Frontend applications and external integrations have no direct database access. Backend services access databases via controlled internal connections.

Why It Is a Trust Boundary? Protects sensitive data. Limits exposure of storage layer. Prevents lateral movement from external actors.

**Trust Boundary 5 – Production Zone ↔ External Integration Zone**
Description: The Payment Processor and Core Banking System are external third-party systems not under direct organizational control. Communication occurs over secure APIs. Webhook callbacks re-enter the production environment via the Webhook Handler.

Why It Is a Trust Boundary? External systems may be compromised. API contracts must be validated. Callback endpoints introduce inbound risk.

**Trust Boundary 6 – Production Zone ↔ External Integration Zone**
Description: The Webhook Handler receives inbound callbacks from the Payment Processor. This creates a secondary external entry point into the production system.

Why It Is a Trust Boundary? Inbound traffic from external system. Potential spoofing or replay risks. Requires validation before updating transactions.

## Trust Boundaries – Online Payment Processing System

| Boundary ID | Between | Boundary Type | Description | Reasoning |
|-------------|----------|--------------|-------------|-----------|
| TB1 | Internet ↔ API Gateway | External Network Boundary | Separates untrusted public internet traffic from internal production services. All external requests must pass through the API Gateway. | Prevents direct access to backend systems and enforces authentication and validation at the entry point. |
| TB2 | Frontend ↔ Backend Services | Application Layer Boundary | Frontend applications (Customer, Merchant, Admin portals) communicate with backend services only via the API Gateway. | Prevents direct database access and maintains separation between presentation and business logic layers. |
| TB3 | Admin Portal ↔ Admin Service | Privilege Boundary | Administrative actions (refund approvals, dispute handling, financial adjustments) occur through a dedicated service layer. | Administrative roles have elevated privileges and can impact financial integrity. |
| TB4 | Backend Services ↔ Data Storage | Data Storage Boundary | Databases are located in a restricted internal zone accessible only by backend services. | Protects sensitive information such as credentials, tokens, and transaction data from unauthorized access. |
| TB5 | Production Zone ↔ Payment Processor | Third-Party / External System Boundary | Communication between internal Payment Service and external Payment Processor API occurs across organizational boundaries. | External systems are not under direct control and introduce integration risk. |
| TB6 | Payment Processor ↔ Webhook Handler | External Callback Boundary | The Webhook Handler receives inbound payment confirmation callbacks from the Payment Processor. | Represents a secondary external entry point that must validate inbound data before updating transactions. |


