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
![High-Level Architecture Diagram](high%20level%20diagram.png)

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
