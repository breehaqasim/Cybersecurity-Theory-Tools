# Secure Architecture and STRIDE-Based Threat Modeling of an Online Payment Processing Application
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
![High-Level Architecture Diagram](high-level-diagram.png)

### Trust Boundaries
| Boundary ID | Between | Boundary Type | Description | Reasoning |
|-------------|----------|--------------|-------------|-----------|
| TB1 | Internet ↔ API Gateway | External Network Boundary | Separates untrusted public internet traffic from internal production services. All external requests must pass through the API Gateway. | Prevents direct access to backend systems and enforces authentication and validation at the entry point. |
| TB2 | Frontend ↔ Backend Services | Application Layer Boundary | Frontend applications (Customer, Merchant, Admin portals) communicate with backend services only via the API Gateway. | Prevents direct database access and maintains separation between presentation and business logic layers. |
| TB3 | Admin Portal ↔ Admin Service | Privilege Boundary | Administrative actions (refund approvals, dispute handling, financial adjustments) occur through a dedicated service layer. | Administrative roles have elevated privileges and can impact financial integrity. |
| TB4 | Backend Services ↔ Data Storage | Data Storage Boundary | Databases are located in a restricted internal zone accessible only by backend services. | Protects sensitive information such as credentials, tokens, and transaction data from unauthorized access. |
| TB5 | Production Zone ↔ Payment Processor | Third-Party / External System Boundary | Communication between internal Payment Service and external Payment Processor API occurs across organizational boundaries. | External systems are not under direct control and introduce integration risk. |
| TB6 | Payment Processor ↔ Webhook Handler | External Callback Boundary | The Webhook Handler receives inbound payment confirmation callbacks from the Payment Processor. | Represents a secondary external entry point that must validate inbound data before updating transactions. |

## Task 2. Asset Identification and Security Objectives 
### Asset Inventory Table
| Asset ID | Asset Name | Category | Description | Location | Sensitivity |
|----------|------------|----------|-------------|----------|------------|
| A1 | User Credentials | Credentials | Customer login data (username, hashed password) | User Database | High |
| A2 | Merchant API Credentials | Credentials | API keys and authentication data used by merchants | Merchant Database | High |
| A3 | Admin Credentials | Credentials | Administrative login credentials and session tokens | User Database | High |
| A4 | Credit Card Data | Financial Data | Card number, expiry, CVV (processed via payment service) | Payment Service / Transaction DB | Critical |
| A5 | Payment Tokens | Financial Data | Tokenized representation of card data | Payment Service | High |
| A6 | Transaction Records | Financial Data | Orders, payments, refunds, settlement data | Transaction/Billing Database | High |
| A7 | Customer Personal Data | Personal Data | Name, email, phone, billing address | User Database | Medium |
| A8 | Merchant Profile Data | Business Data | Merchant details, settlement preferences | Merchant Database | Medium |
| A9 | Business Logic | Application Logic | Order validation, refund logic, settlement rules | API Backend / Admin Service | High |
| A10 | Audit Logs | Logging Data | Immutable records of admin actions and system events | Audit Log Store | High |
| A11 | JWT / Session Tokens | Authentication Data | Session identifiers used for user authentication | API Gateway | High |
| A12 | Payment Processor API Contract | External Integration | Communication interface with third-party processor | Payment Service | High |

### Mapping of Assets to Security Objectives (CIAA)
| Asset ID | Asset Name | Confidentiality | Integrity | Availability | Accountability |
|----------|------------|----------------|-----------|-------------|---------------|
| A1 | User Credentials | ✔ Required | ✔ Required | ✔ Required | ✔ Required |
| A2 | Merchant API Credentials | ✔ Required | ✔ Required | ✔ Required | ✔ Required |
| A3 | Admin Credentials | ✔ Required | ✔ Required | ✔ Required | ✔ Required |
| A4 | Credit Card Data | ✔ Critical | ✔ Critical | ✔ Required | ✔ Required |
| A5 | Payment Tokens | ✔ Critical | ✔ Required | ✔ Required | ✔ Required |
| A6 | Transaction Records | ✔ Required | ✔ Critical | ✔ Critical | ✔ Required |
| A7 | Customer Personal Data | ✔ Required | ✔ Required | ✔ Required | Optional |
| A8 | Merchant Profile Data | ✔ Required | ✔ Required | ✔ Required | Optional |
| A9 | Business Logic | ✔ Required | ✔ Critical | ✔ Critical | Optional |
| A10 | Audit Logs | ✔ Required | ✔ Critical | ✔ Required | ✔ Critical |
| A11 | JWT / Session Tokens | ✔ Critical | ✔ Required | ✔ Required | ✔ Required |
| A12 | Payment Processor API Contract | ✔ Required | ✔ Critical | ✔ Critical | Optional |

## Task 3. Threat Modeling 
### Threat Model Table
| ID | Threat Area | STRIDE Category | Affected Component | Threat Title (From Tool) | Description (From Tool) | Impact | Risk Level | Risk Reasoning |
|----|------------|----------------|-------------------|--------------------------|--------------------------|--------|------------|---------------|
| 0 | Authentication | Spoofing | Customer (External Entity) | Spoofing the Customer External Entity | Customer may be spoofed by an attacker and this may allow unauthorized access to the system. | Unauthorized access to customer accounts and transactions. | High | Internet-facing authentication entry point; identity spoofing leads to financial compromise. |
| 1 | Authentication | Elevation Of Privilege | API Gateway | Elevation Using Impersonation | API Gateway may be able to impersonate the consumer. | Attackers may gain higher privileges through token/session misuse. | High | Gateway controls authentication flow; impersonation impacts entire system. |
| 2 | Authentication | Spoofing | Merchant (External Entity) | Spoofing the Merchant External Entity | Merchant may be spoofed by an attacker and this may allow unauthorized access. | Fraudulent refund or settlement requests. | High | Merchant APIs interact with financial operations; spoofing has direct financial impact. |
| 4 | Administrative Access | Spoofing | Admin (External Entity) | Spoofing the Admin External Entity | Admin may be spoofed by an attacker and this may allow unauthorized administrative control. | Full system compromise via admin takeover. | High | Admin plane has highest privilege level; compromise affects all zones. |
| 5 | Data Storage | Spoofing | User Database | Spoofing of Destination Data Store User Database | Destination data store User Database may be spoofed. | Attackers redirect queries to malicious database instance. | High | Database stores sensitive credentials and PII; spoofing breaks trust boundary. |
| 8 | Data Storage | Spoofing | Transaction/Billing Database | Spoofing of Destination Data Store Transaction/Billing Database | Destination data store Transaction/Billing Database may be spoofed. | Financial record manipulation. | High | Transaction integrity is critical for business continuity. |
| 10 | Logging & Monitoring | Spoofing | Audit Log Database | Spoofing of Destination Data Store Audit Log Database | Destination data store Audit Log Database may be spoofed. | Attackers hide traces of malicious activity. | High | Audit integrity is essential for accountability and forensic investigation. |
| 16 | API Communication | Tampering / Input Validation | API Gateway | Potential Lack of Input Validation for API Gateway | API Gateway may not validate input properly. | Injection attacks or malformed requests causing compromise. | High | Entry point for all traffic; weak validation exposes entire backend. |
| 18 | Logging & Monitoring | Repudiation | API Gateway | Potential Data Repudiation by API Gateway | API Gateway may not log actions sufficiently to prevent repudiation. | Inability to trace malicious activity. | High | Weak logging reduces accountability and regulatory compliance. |
| 19 | API Communication | Information Disclosure | Data Flow | Data Flow Sniffing | Data transmitted over the data flow may be sniffed by an attacker. | Exposure of tokens or sensitive financial data. | High | Cross-boundary communication increases exposure surface. |
| 20 | API Communication | Denial of Service | API Gateway | Potential Process Crash or Stop for API Gateway | API Gateway process may crash or stop. | Service unavailability for all users. | High | Single entry point; DoS affects entire platform availability. |
| 13 | API Communication | Denial of Service | Webhook Handler | Potential Excessive Resource Consumption for Webhook Handler or Transaction/Billing Database | Webhook handler may consume excessive resources. | Delayed payment confirmations and settlement issues. | High | Payment lifecycle depends on webhook reliability. |

### Threat diagram
![Threat Diagram](threat%20diagram.png)



