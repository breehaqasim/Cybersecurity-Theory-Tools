# Secure Architecture and OWASP-Based Threat Modeling of an Online Payment Processing Application
## System Overview
I made this system so customers can pay merchants online. It also helps with other tasks like giving refunds solving problems and sorting out money issues.

The system is set up in layers and can work with any cloud it also connects to a payment processor and a core banking system inside. Since the system deals with financial information I make sure that it is private, accurate, available and responsible.

I look at what could happen if someone outside tries to attack the system and what could happen if someone inside tries to cause trouble so the system is safe and secure and I think about the payment system a lot to make sure it is really secure.

## Task 1 — System Definition and Architecture
### Application Components
**Frontend**
- Customer Web Application (checkout, order history)
- Merchant Portal (transaction and settlement viewing)

**Backend Services**
- API Gateway (authentication, rate limiting, request validation)
- API Backend (business logic)
- Payment Service (payment orchestration and processing)
- Webhook Handler (receives payment confirmation callbacks)
- Admin Service (refund approvals, dispute handling)
