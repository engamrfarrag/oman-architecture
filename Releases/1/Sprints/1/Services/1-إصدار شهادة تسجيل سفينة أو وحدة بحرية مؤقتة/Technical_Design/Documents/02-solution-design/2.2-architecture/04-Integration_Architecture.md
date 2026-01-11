# REG001 – Integration Architecture

> **Service Code:** REG001  
> **Service Name (EN):** Issuing Temporary Registration Certificate for a Ship or Marine Unit  
> **Service Name (AR):** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة  
> **Document Type:** Integration Architecture  
> **Version:** 1.0  
> **Last Updated:** 2026-01-11

---

## 1. Purpose / الغرض

This document defines the **integration architecture** for REG001, detailing:
- External system integrations and contracts
- Anti-Corruption Layer (ACL) patterns
- Message formats and protocols
- Error handling and resilience patterns
- Security considerations for integrations

---

## 2. Integration Overview

### 2.1 External Systems Summary

| # | External System | Provider | Purpose | Pattern | Protocol |
|---|-----------------|----------|---------|---------|----------|
| 1 | **ROP PKI** | Royal Oman Police | Identity authentication | Sync + ACL | REST/SOAP |
| 2 | **Invest Easy** | MOFA | CR and activity validation | Sync + ACL | REST |
| 3 | **MAFWR** | Ministry of Agriculture | Fishing vessel approval | Async + ACL | REST + Callback |
| 4 | **Bayan** | Customs | Customs clearance | Async + ACL | REST + Message |
| 5 | **Inspection Entities** | Classification Societies | Vessel inspection | Async + ACL | REST + Callback |
| 6 | **ePayment Gateway** | Multiple Banks | Payment processing | Sync + Callback | REST |
| 7 | **SMS Provider** | Telecom | SMS notifications | Async | REST |
| 8 | **Email Provider** | SMTP/API | Email notifications | Async | SMTP/REST |

### 2.2 Integration Landscape

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    INTEGRATION LANDSCAPE                                             │
├─────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                      │
│   SYNCHRONOUS INTEGRATIONS                          ASYNCHRONOUS INTEGRATIONS                        │
│   ┌─────────────────────────────────────┐          ┌─────────────────────────────────────┐          │
│   │                                     │          │                                     │          │
│   │  ┌───────────┐    ┌───────────┐    │          │  ┌───────────┐    ┌───────────┐    │          │
│   │  │  ROP PKI  │    │Invest Easy│    │          │  │   MAFWR   │    │   Bayan   │    │          │
│   │  │ (Identity)│    │ (CR/Act)  │    │          │  │ (Fishing) │    │ (Customs) │    │          │
│   │  └─────┬─────┘    └─────┬─────┘    │          │  └─────┬─────┘    └─────┬─────┘    │          │
│   │        │                │          │          │        │                │          │          │
│   │        ▼                ▼          │          │        ▼                ▼          │          │
│   │  ┌───────────┐    ┌───────────┐    │          │  ┌───────────┐    ┌───────────┐    │          │
│   │  │ Identity  │    │Eligibility│    │          │  │ Approval  │    │  Customs  │    │          │
│   │  │  Adapter  │    │  Adapter  │    │          │  │  Adapter  │    │  Adapter  │    │          │
│   │  └─────┬─────┘    └─────┬─────┘    │          │  └─────┬─────┘    └─────┬─────┘    │          │
│   │        │                │          │          │        │                │          │          │
│   │        └───────┬────────┘          │          │        └───────┬────────┘          │          │
│   │                │                   │          │                │                   │          │
│   │                ▼                   │          │                ▼                   │          │
│   │  ┌───────────────────────────┐     │          │  ┌───────────────────────────┐     │          │
│   │  │    Request-Response       │     │          │  │   Message Correlation     │     │          │
│   │  │    (< 30 seconds)         │     │          │  │   (minutes to days)       │     │          │
│   │  └───────────────────────────┘     │          │  └───────────────────────────┘     │          │
│   │                                     │          │                                     │          │
│   └─────────────────────────────────────┘          └─────────────────────────────────────┘          │
│                                                                                                      │
│   CALLBACK-BASED INTEGRATIONS                       NOTIFICATION INTEGRATIONS                        │
│   ┌─────────────────────────────────────┐          ┌─────────────────────────────────────┐          │
│   │                                     │          │                                     │          │
│   │  ┌───────────┐    ┌───────────┐    │          │  ┌───────────┐    ┌───────────┐    │          │
│   │  │Inspection │    │ ePayment  │    │          │  │    SMS    │    │   Email   │    │          │
│   │  │  Entity   │    │  Gateway  │    │          │  │ Provider  │    │ Provider  │    │          │
│   │  └─────┬─────┘    └─────┬─────┘    │          │  └─────┬─────┘    └─────┬─────┘    │          │
│   │        │                │          │          │        │                │          │          │
│   │        ▼                ▼          │          │        ▼                ▼          │          │
│   │  ┌───────────┐    ┌───────────┐    │          │  ┌─────────────────────────────┐    │          │
│   │  │Inspection │    │ Payment   │    │          │  │    Notification Adapter     │    │          │
│   │  │  Adapter  │    │  Adapter  │    │          │  │     (Fire-and-Forget)       │    │          │
│   │  └───────────┘    └───────────┘    │          │  └─────────────────────────────┘    │          │
│   │                                     │          │                                     │          │
│   └─────────────────────────────────────┘          └─────────────────────────────────────┘          │
│                                                                                                      │
└─────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Integration Patterns

### 3.1 Anti-Corruption Layer (ACL)

All external integrations use the ACL pattern to:
- Isolate internal domain models from external system specifics
- Translate between external and internal data formats
- Handle external system errors gracefully
- Enable external system replacement without internal impact

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ANTI-CORRUPTION LAYER PATTERN                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────┐         ┌─────────────────────────────────────────┐   │
│   │                 │         │              ACL ADAPTER                 │   │
│   │    External     │         │  ┌─────────────────────────────────┐   │   │
│   │     System      │◄───────►│  │         Translator              │   │   │
│   │                 │         │  │  - External DTO → Internal VO   │   │   │
│   │  (Own Model)    │         │  │  - Internal VO → External DTO   │   │   │
│   │                 │         │  └─────────────────────────────────┘   │   │
│   └─────────────────┘         │  ┌─────────────────────────────────┐   │   │
│                               │  │         Error Handler           │   │   │
│                               │  │  - External Error → Domain Err  │   │   │
│                               │  │  - Retry Logic                  │   │   │
│                               │  │  - Circuit Breaker              │   │   │
│                               │  └─────────────────────────────────┘   │   │
│                               │  ┌─────────────────────────────────┐   │   │
│                               │  │         Client                  │   │   │
│                               │  │  - HTTP Client                  │   │   │
│                               │  │  - Connection Pooling           │   │   │
│                               │  │  - Timeout Configuration        │   │   │
│                               │  └─────────────────────────────────┘   │   │
│                               └─────────────────────────────────────────┘   │
│                                              │                               │
│                                              ▼                               │
│                               ┌─────────────────────────────────────────┐   │
│                               │           INTERNAL DOMAIN               │   │
│                               │                                         │   │
│                               │   Uses internal domain model only       │   │
│                               │   No knowledge of external specifics    │   │
│                               │                                         │   │
│                               └─────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Integration Styles

| Style | When to Use | REG001 Usage |
|-------|-------------|--------------|
| **Request-Response (Sync)** | Immediate response needed, < 30s | ROP PKI, Invest Easy |
| **Callback (Async)** | Long-running external process | MAFWR, Inspection, Payment |
| **Message Correlation** | Event-driven, multi-step | Bayan customs |
| **Fire-and-Forget** | No response needed | SMS, Email |

---

## 4. External System Specifications

### 4.1 ROP PKI – Identity Authentication

**Purpose:** Authenticate applicants using national PKI infrastructure.

#### Integration Details

| Attribute | Value |
|-----------|-------|
| **System Owner** | Royal Oman Police |
| **Protocol** | SOAP / REST |
| **Pattern** | Request-Response |
| **Timeout** | 30 seconds |
| **Retry Policy** | 3 retries, exponential backoff |

#### Request/Response Contract

**Authentication Request:**
```json
{
  "pkiToken": "base64-encoded-certificate",
  "timestamp": "2026-01-11T10:30:00Z",
  "clientId": "MTCIT-REG"
}
```

**Authentication Response:**
```json
{
  "success": true,
  "userId": "OMN12345678",
  "fullNameAr": "محمد عبدالله",
  "fullNameEn": "Mohammed Abdullah",
  "nationalId": "12345678",
  "nationality": "OMANI_CITIZEN",
  "sessionToken": "jwt-token-here",
  "expiresAt": "2026-01-11T12:30:00Z"
}
```

#### Error Mapping

| External Error | Internal Error | HTTP Status |
|----------------|----------------|-------------|
| CERT_EXPIRED | SESSION_EXPIRED | 401 |
| CERT_REVOKED | SESSION_INVALID | 403 |
| INVALID_SIGNATURE | AUTHENTICATION_FAILED | 401 |
| SERVICE_UNAVAILABLE | EXTERNAL_UNAVAILABLE | 503 |

---

### 4.2 Invest Easy – CR & Activity Validation

**Purpose:** Validate Commercial Registration and business activity eligibility.

#### Integration Details

| Attribute | Value |
|-----------|-------|
| **System Owner** | Ministry of Commerce |
| **Protocol** | REST API |
| **Pattern** | Request-Response |
| **Timeout** | 30 seconds |
| **Retry Policy** | 3 retries, exponential backoff |

#### CR Validation Contract

**Request:**
```
GET /api/v1/commercial-registrations/{crNumber}
Authorization: Bearer {service-token}
```

**Response:**
```json
{
  "crNumber": "1234567",
  "companyNameAr": "شركة النقل البحري",
  "companyNameEn": "Maritime Transport Company",
  "status": "ACTIVE",
  "expiryDate": "2027-06-15",
  "activities": [
    {
      "code": "50101",
      "nameAr": "النقل البحري",
      "nameEn": "Maritime Transport"
    }
  ]
}
```

#### Activity Validation Contract

**Request:**
```
POST /api/v1/activity-eligibility/validate
Authorization: Bearer {service-token}
Content-Type: application/json

{
  "crNumber": "1234567",
  "requiredActivity": "VESSEL_REGISTRATION",
  "vesselType": "FISHING"
}
```

**Response:**
```json
{
  "eligible": true,
  "matchedActivities": ["50101", "50102"],
  "reason": null
}
```

---

### 4.3 MAFWR – Fishing Vessel Approval

**Purpose:** Obtain approval for fishing vessel registration.

#### Integration Details

| Attribute | Value |
|-----------|-------|
| **System Owner** | Ministry of Agriculture, Fisheries & Water Resources |
| **Protocol** | REST API + Webhook Callback |
| **Pattern** | Async with Callback |
| **SLA** | 3-5 business days |
| **Retry Policy** | N/A (async) |

#### Approval Request Contract

**Request:**
```
POST /api/v1/vessel-approvals
Authorization: Bearer {service-token}
Content-Type: application/json

{
  "requestId": "REG001-2026-00123",
  "vesselDetails": {
    "name": "السفينة العمانية",
    "type": "FISHING",
    "length": 15.5,
    "grossTonnage": 25,
    "buildYear": 2020
  },
  "applicantDetails": {
    "nationalId": "12345678",
    "name": "محمد عبدالله",
    "crNumber": "1234567"
  },
  "callbackUrl": "https://mtcit.gov.om/api/v1/callbacks/mafwr"
}
```

**Acknowledgment Response:**
```json
{
  "approvalRequestId": "MAFWR-2026-00456",
  "status": "RECEIVED",
  "estimatedCompletionDate": "2026-01-16"
}
```

#### Callback Contract

**Callback (from MAFWR to MTCIT):**
```
POST {callbackUrl}
Content-Type: application/json

{
  "approvalRequestId": "MAFWR-2026-00456",
  "originalRequestId": "REG001-2026-00123",
  "decision": "APPROVED",
  "decisionDate": "2026-01-14T14:30:00Z",
  "remarks": "Approved for fishing activities",
  "approvedBy": "MAFWR-INSPECTOR-001"
}
```

**Decision Values:**
- `APPROVED` – Proceed with registration
- `REJECTED` – Cannot register; reason provided
- `CONDITIONAL` – Approved with conditions
- `PENDING_INFO` – Additional information required

---

### 4.4 Bayan – Customs Clearance

**Purpose:** Process customs clearance for imported vessels.

#### Integration Details

| Attribute | Value |
|-----------|-------|
| **System Owner** | Royal Oman Customs |
| **Protocol** | REST API + Message Queue |
| **Pattern** | Async with Message Correlation |
| **SLA** | 1-3 business days |

#### Clearance Request Contract

**Request:**
```
POST /api/v1/vessel-clearances
Authorization: Bearer {service-token}
Content-Type: application/json

{
  "requestId": "REG001-2026-00123",
  "vesselDetails": {
    "previousFlag": "PANAMA",
    "imoNumber": "IMO1234567",
    "name": "Sea Explorer",
    "type": "CARGO",
    "grossTonnage": 5000
  },
  "importDetails": {
    "portOfEntry": "SOHAR",
    "importDate": "2026-01-10",
    "purchaseValue": {
      "amount": 500000,
      "currency": "OMR"
    }
  },
  "applicantDetails": {
    "crNumber": "1234567",
    "companyName": "Maritime Transport Company"
  }
}
```

#### Message Events

**Import Document Received:**
```json
{
  "eventType": "IMPORT_DOCUMENT_READY",
  "requestId": "REG001-2026-00123",
  "bayanReference": "BAYAN-2026-00789",
  "documentUrl": "https://bayan.gov.om/documents/import-doc-xxx.pdf",
  "timestamp": "2026-01-11T15:00:00Z"
}
```

**Final Clearance:**
```json
{
  "eventType": "CUSTOMS_CLEARED",
  "requestId": "REG001-2026-00123",
  "bayanReference": "BAYAN-2026-00789",
  "clearanceNumber": "CLR-2026-12345",
  "clearanceDate": "2026-01-12T10:00:00Z",
  "dutiesPaid": {
    "amount": 25000,
    "currency": "OMR"
  }
}
```

---

### 4.5 Inspection Entities – Vessel Inspection

**Purpose:** Request and receive vessel inspection results.

#### Integration Details

| Attribute | Value |
|-----------|-------|
| **System Owner** | Classification Societies / MTCIT Inspectors |
| **Protocol** | REST API + Webhook |
| **Pattern** | Async with Callback |
| **SLA** | 5-10 business days |

#### Inspection Request Contract

**Request:**
```
POST /api/v1/vessel-inspections
Authorization: Bearer {service-token}
Content-Type: application/json

{
  "requestId": "REG001-2026-00123",
  "inspectionType": "INITIAL_REGISTRATION",
  "vesselDetails": {
    "name": "السفينة العمانية",
    "type": "CARGO",
    "length": 28.5,
    "grossTonnage": 450,
    "buildYear": 2019,
    "buildMaterial": "STEEL"
  },
  "location": {
    "port": "MUSCAT",
    "berth": "B12"
  },
  "callbackUrl": "https://mtcit.gov.om/api/v1/callbacks/inspection"
}
```

#### Callback Contract

**Inspection Result Callback:**
```json
{
  "inspectionId": "INSP-2026-00321",
  "originalRequestId": "REG001-2026-00123",
  "result": "PASSED",
  "inspectionDate": "2026-01-15T09:00:00Z",
  "inspector": "CLASS-INSPECTOR-001",
  "reportUrl": "https://inspection.mtcit.gov.om/reports/INSP-2026-00321.pdf",
  "findings": [],
  "recommendations": [],
  "validUntil": "2027-01-15"
}
```

**Result Values:**
- `PASSED` – Vessel meets requirements
- `FAILED` – Vessel does not meet requirements
- `CONDITIONAL` – Passed with conditions
- `RESCHEDULED` – Inspection rescheduled

---

### 4.6 ePayment Gateway – Payment Processing

**Purpose:** Process fee and penalty payments.

#### Integration Details

| Attribute | Value |
|-----------|-------|
| **System Owner** | Government Payment Gateway |
| **Protocol** | REST API + Redirect + Webhook |
| **Pattern** | Sync initiation + Async confirmation |
| **Timeout** | 15 minutes (payment session) |

#### Payment Initiation Contract

**Request:**
```
POST /api/v1/payments/initiate
Authorization: Bearer {service-token}
Content-Type: application/json

{
  "merchantId": "MTCIT-REG001",
  "orderId": "INV-2026-00456",
  "amount": {
    "value": 150.000,
    "currency": "OMR"
  },
  "description": "Temporary Vessel Registration - REG001-2026-00123",
  "returnUrl": "https://mtcit.gov.om/registration/payment-complete",
  "callbackUrl": "https://mtcit.gov.om/api/v1/callbacks/payment",
  "expiryMinutes": 15,
  "metadata": {
    "requestId": "REG001-2026-00123",
    "invoiceId": "INV-2026-00456"
  }
}
```

**Response:**
```json
{
  "paymentId": "PAY-2026-00789",
  "paymentUrl": "https://epayment.gov.om/pay/PAY-2026-00789",
  "status": "PENDING",
  "expiresAt": "2026-01-11T11:00:00Z"
}
```

#### Payment Callback Contract

**Callback (from Payment Gateway):**
```json
{
  "paymentId": "PAY-2026-00789",
  "orderId": "INV-2026-00456",
  "status": "COMPLETED",
  "transactionReference": "TXN-2026-12345",
  "paidAmount": {
    "value": 150.000,
    "currency": "OMR"
  },
  "paidAt": "2026-01-11T10:45:00Z",
  "paymentMethod": "CREDIT_CARD",
  "cardLastFour": "4321"
}
```

**Status Values:**
- `COMPLETED` – Payment successful
- `FAILED` – Payment failed
- `CANCELLED` – User cancelled
- `EXPIRED` – Payment session expired

---

### 4.7 Notification Providers

**Purpose:** Deliver SMS and Email notifications to applicants.

#### SMS Integration

**Request:**
```
POST /api/v1/sms/send
Authorization: Bearer {service-token}
Content-Type: application/json

{
  "to": "+968xxxxxxxx",
  "templateId": "REG001_STATUS_UPDATE",
  "variables": {
    "requestNumber": "REG001-2026-00123",
    "status": "تم إصدار الشهادة",
    "actionRequired": "لا يوجد"
  },
  "language": "ar"
}
```

#### Email Integration

**Request:**
```
POST /api/v1/emails/send
Authorization: Bearer {service-token}
Content-Type: application/json

{
  "to": "applicant@example.com",
  "templateId": "REG001_CERTIFICATE_ISSUED",
  "variables": {
    "applicantName": "محمد عبدالله",
    "requestNumber": "REG001-2026-00123",
    "vesselName": "السفينة العمانية",
    "certificateNumber": "TEMP-REG-2026-00456",
    "downloadLink": "https://mtcit.gov.om/certificates/TEMP-REG-2026-00456"
  },
  "attachments": [
    {
      "filename": "certificate.pdf",
      "contentType": "application/pdf",
      "content": "base64-encoded-content"
    }
  ],
  "language": "ar"
}
```

---

## 5. Resilience Patterns

### 5.1 Circuit Breaker Configuration

| Integration | Failure Threshold | Recovery Timeout | Half-Open Calls |
|-------------|-------------------|------------------|-----------------|
| ROP PKI | 5 failures in 60s | 30 seconds | 3 |
| Invest Easy | 5 failures in 60s | 30 seconds | 3 |
| Payment Gateway | 3 failures in 30s | 60 seconds | 2 |
| Notification | 10 failures in 60s | 120 seconds | 5 |

### 5.2 Retry Configuration

| Integration | Max Retries | Initial Delay | Max Delay | Backoff |
|-------------|-------------|---------------|-----------|---------|
| ROP PKI | 3 | 1 second | 10 seconds | Exponential |
| Invest Easy | 3 | 1 second | 10 seconds | Exponential |
| Payment Gateway | 2 | 2 seconds | 8 seconds | Exponential |
| Notification | 5 | 5 seconds | 60 seconds | Exponential |

### 5.3 Timeout Configuration

| Integration | Connection Timeout | Read Timeout | Total Timeout |
|-------------|-------------------|--------------|---------------|
| ROP PKI | 5 seconds | 25 seconds | 30 seconds |
| Invest Easy | 5 seconds | 25 seconds | 30 seconds |
| Payment Gateway | 5 seconds | 10 seconds | 15 seconds |
| Notification | 5 seconds | 10 seconds | 15 seconds |

### 5.4 Fallback Strategies

| Integration | Fallback Strategy |
|-------------|-------------------|
| ROP PKI | Reject with user-friendly message; no fallback |
| Invest Easy | Cache recent validations for 24h (read-only) |
| MAFWR | Queue for retry; notify applicant of delay |
| Payment Gateway | Redirect to alternative payment method |
| Notification | Queue for retry; mark for manual follow-up |

---

## 6. Security Considerations

### 6.1 Authentication Methods

| Integration | Method | Details |
|-------------|--------|---------|
| ROP PKI | Mutual TLS | Client certificate required |
| Invest Easy | OAuth 2.0 | Client credentials flow |
| MAFWR | API Key + JWT | Signed requests |
| Bayan | OAuth 2.0 | Client credentials flow |
| Payment Gateway | HMAC Signature | Request signing |
| Notification | API Key | Simple key authentication |

### 6.2 Data Protection

| Data Type | Protection | Storage |
|-----------|------------|---------|
| PKI Tokens | Never stored | Memory only |
| National IDs | Encrypted at rest | Masked in logs |
| Payment Data | PCI-DSS compliant | Not stored |
| Documents | Encrypted at rest | Object storage |

### 6.3 Network Security

- All external calls over HTTPS/TLS 1.2+
- IP whitelisting for callback endpoints
- Rate limiting on callback endpoints
- WAF protection on exposed APIs

---

## 7. Monitoring & Observability

### 7.1 Integration Metrics

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| `integration.latency` | Response time by integration | > 5 seconds |
| `integration.error_rate` | Error percentage | > 5% in 5 minutes |
| `integration.circuit_breaker` | Circuit breaker state changes | Any open |
| `integration.retry_count` | Retry attempts | > 10 in 1 minute |
| `integration.callback_latency` | Time to receive callback | > SLA |

### 7.2 Alerting Rules

| Alert | Condition | Severity |
|-------|-----------|----------|
| Integration Down | Circuit breaker OPEN for > 5 min | Critical |
| High Latency | P99 latency > 10s for 5 min | Warning |
| Callback Delayed | No callback received within SLA | Warning |
| Authentication Failure | > 10 auth failures in 1 min | Critical |

### 7.3 Logging Requirements

All integration calls must log:
- Request ID (correlation)
- External system name
- Request timestamp
- Response timestamp
- HTTP status code
- Error code (if any)
- Payload (masked sensitive fields)

---

## 8. Traceability

| Integration | BPMN Subprocess | Capabilities |
|-------------|-----------------|--------------|
| ROP PKI | CC01 | CAP-02 |
| Invest Easy | SP02 | CAP-04, CAP-05 |
| MAFWR | SP05 | CAP-13 |
| Inspection | SP06 | CAP-15 |
| Bayan | SP07 | CAP-17 |
| Payment Gateway | SP08 | CAP-21 |
| Notifications | SP02-SP10 | CAP-23 |

---

## 9. Related Documents

- [01-C4_Architecture.md](01-C4_Architecture.md) – C4 model diagrams
- [02-Capability_to_Service_C4.md](02-Capability_to_Service_C4.md) – Service decomposition
- [03-Sequence_Diagrams.md](03-Sequence_Diagrams.md) – Service interaction flows
- [03-Context_Map_ACL_SharedKernel.md](../2.1-domain-model/03-Context_Map_ACL_SharedKernel.md) – ACL patterns

---

**Last Updated:** 2026-01-11  
**Version:** 1.0  
**Author:** Solution Architecture Team
