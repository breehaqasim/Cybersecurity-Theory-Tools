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

<!--
Source - https://stackoverflow.com/q/11509830
Posted by Dave Dopson, modified by community. See post 'Timeline' for change history
Retrieved 2026-02-23, License - CC BY-SA 4.0
-->

<font color="green"> Some green text </font>

**Backend Services**
- API Gateway (authentication, rate limiting, request validation)
- API Backend (business logic)
- Payment Service (payment orchestration and processing)
- Webhook Handler (receives payment confirmation callbacks)
- Admin Service (refund approvals, dispute handling)

**Data Storage (Private Zone)**
- User Database (accounts, credentials)
- Merchant Database (roles, API credentials)
- Transaction/Billing Database (orders, payments, refunds)
- Audit Log Store (immutable logs)

**External Integrations**
- Payment Processor API (authorization, capture, tokenization)
- Core Banking System (settlement and reconciliation)
- Notification Service (email/SMS)

**Admin Access Paths**
- Admin Portal accessible only through VPN or Zero Trust Network Access (ZTNA)
- Multi-factor authentication required
- Separate privilege plane from customer APIs

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
