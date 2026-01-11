# REG001 – Sequence Diagrams

> **Service Code:** REG001  
> **Service Name (EN):** Issuing Temporary Registration Certificate for a Ship or Marine Unit  
> **Service Name (AR):** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة  
> **Document Type:** Sequence Diagrams  
> **Version:** 1.0  
> **Last Updated:** 2026-01-11

---

## 1. Purpose / الغرض

This document provides **sequence diagrams** for key service interactions in REG001, showing:
- Message flows between services
- Synchronous vs asynchronous communication
- External system integration patterns
- Error handling paths

---

## 2. Diagram Conventions

| Symbol | Meaning |
|--------|---------|
| `─────►` | Synchronous call |
| `- - -►` | Asynchronous message/event |
| `◄─────` | Synchronous response |
| `◄- - -` | Asynchronous callback |
| `[condition]` | Conditional branch |
| `loop` | Repeated action |
| `alt` | Alternative paths |
| `opt` | Optional action |

---

## 3. Main Process Flow Sequences

### 3.1 SD-01: Registration Request Initiation

**Scenario:** Applicant starts a new registration request.  
**Actors:** Applicant, API Gateway, Registration Service, Identity Adapter, Applicant Service

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Applicant  │     │ API Gateway │     │ Registration│     │  Identity   │     │  Applicant  │
│   Portal    │     │             │     │   Service   │     │   Adapter   │     │   Service   │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │                   │                   │
       │  1. Login (PKI)   │                   │                   │                   │
       │──────────────────►│                   │                   │                   │
       │                   │  2. Authenticate  │                   │                   │
       │                   │──────────────────►│                   │                   │
       │                   │                   │  3. Validate PKI  │                   │
       │                   │                   │──────────────────►│                   │
       │                   │                   │                   │  4. ROP PKI Call  │
       │                   │                   │                   │───────────────────┤
       │                   │                   │                   │◄──────────────────┤
       │                   │                   │◄──────────────────│                   │
       │                   │◄──────────────────│  AuthResult       │                   │
       │◄──────────────────│  Session Token    │                   │                   │
       │                   │                   │                   │                   │
       │  5. Start Registration               │                   │                   │
       │──────────────────►│                   │                   │                   │
       │                   │  6. POST /requests│                   │                   │
       │                   │──────────────────►│                   │                   │
       │                   │                   │  7. Get Profile   │                   │
       │                   │                   │──────────────────────────────────────►│
       │                   │                   │◄──────────────────────────────────────│
       │                   │                   │                   │                   │
       │                   │                   │  8. Start BPMN Process (Flowable)    │
       │                   │                   │─────────────────────────┐             │
       │                   │                   │◄────────────────────────┘             │
       │                   │                   │                   │                   │
       │                   │◄──────────────────│  RequestCreated   │                   │
       │◄──────────────────│  Request ID       │                   │                   │
       │                   │                   │                   │                   │
       │                   │                   │  9. Publish Event (async)             │
       │                   │                   │- - - - - - - - - - - - - - - - - - - -►
       │                   │                   │  RequestSubmittedEvent                │
       │                   │                   │                   │                   │
└──────┴──────┘     └──────┴──────┘     └──────┴──────┘     └──────┴──────┘     └──────┴──────┘
```

---

### 3.2 SD-02: Company CR and Activity Validation

**Scenario:** Validate company Commercial Registration and activity eligibility.  
**Actors:** Registration Service, Eligibility Adapter, Invest Easy, Policy Service

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Registration│     │ Eligibility │     │ Invest Easy │     │   Policy    │
│   Service   │     │   Adapter   │     │  (External) │     │   Service   │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │                   │
       │  1. Validate CR   │                   │                   │
       │──────────────────►│                   │                   │
       │                   │  2. GET /cr/{num} │                   │
       │                   │──────────────────►│                   │
       │                   │◄──────────────────│                   │
       │                   │  CR Details       │                   │
       │◄──────────────────│                   │                   │
       │  CRValidationResult                   │                   │
       │                   │                   │                   │
       ├───────────────────┤                   │                   │
       │ [If CR Valid]     │                   │                   │
       │                   │                   │                   │
       │  3. Check Activity│                   │                   │
       │──────────────────►│                   │                   │
       │                   │  4. GET /activity │                   │
       │                   │──────────────────►│                   │
       │                   │◄──────────────────│                   │
       │◄──────────────────│  Activity List    │                   │
       │                   │                   │                   │
       │  5. Validate Activity for Vessel Type │                   │
       │──────────────────────────────────────────────────────────►│
       │                   │                   │                   │
       │                   │                   │     DMN Evaluate  │
       │                   │                   │                   │
       │◄──────────────────────────────────────────────────────────│
       │  EligibilityResult                    │                   │
       │                   │                   │                   │
       ├───────────────────┤                   │                   │
       │ [If Eligible]     │                   │                   │
       │  Continue Flow    │                   │                   │
       │                   │                   │                   │
       ├───────────────────┤                   │                   │
       │ [If Not Eligible] │                   │                   │
       │  Reject Request   │                   │                   │
       │                   │                   │                   │
└──────┴──────┘     └──────┴──────┘     └──────┴──────┘     └──────┴──────┘
```

---

### 3.3 SD-03: Vessel Data Capture and Policy Validation

**Scenario:** Capture vessel data and validate against policy rules.  
**Actors:** Applicant Portal, Registration Service, Vessel Service, Policy Service

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Applicant  │     │ Registration│     │   Vessel    │     │   Policy    │
│   Portal    │     │   Service   │     │   Service   │     │   Service   │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │                   │
       │  1. Submit Vessel Data                │                   │
       │──────────────────►│                   │                   │
       │                   │  2. Create Vessel │                   │
       │                   │──────────────────►│                   │
       │                   │◄──────────────────│                   │
       │                   │  VesselId         │                   │
       │                   │                   │                   │
       │                   │  3. Validate Vessel Age               │
       │                   │──────────────────────────────────────►│
       │                   │                   │                   │
       │                   │                   │  DMN: vessel-age-validation
       │                   │                   │                   │
       │                   │◄──────────────────────────────────────│
       │                   │  AgeValidationResult                  │
       │                   │                   │                   │
       ├───────────────────┤                   │                   │
       │ [Age Exceeded - Warning]              │                   │
       │◄──────────────────│                   │                   │
       │  Warning + Disclaimer                 │                   │
       │                   │                   │                   │
       │  4. Accept Disclaimer                 │                   │
       │──────────────────►│                   │                   │
       │                   │                   │                   │
       ├───────────────────┤                   │                   │
       │ [Age Valid]       │                   │                   │
       │                   │                   │                   │
       │                   │  5. Check Prohibited List             │
       │                   │──────────────────────────────────────►│
       │                   │                   │  Check CONF-04    │
       │                   │◄──────────────────────────────────────│
       │                   │  ProhibitedCheckResult                │
       │                   │                   │                   │
       ├───────────────────┤                   │                   │
       │ [Prohibited]      │                   │                   │
       │◄──────────────────│                   │                   │
       │  Rejection        │                   │                   │
       │                   │                   │                   │
       ├───────────────────┤                   │                   │
       │ [Not Prohibited]  │                   │                   │
       │                   │                   │                   │
       │                   │  6. Check Penalty Trigger             │
       │                   │──────────────────────────────────────►│
       │                   │                   │  DMN: penalty-calculation
       │                   │◄──────────────────────────────────────│
       │                   │  PenaltyResult (may be 0)             │
       │                   │                   │                   │
       │◄──────────────────│                   │                   │
       │  Vessel Saved     │                   │                   │
       │                   │                   │                   │
└──────┴──────┘     └──────┴──────┘     └──────┴──────┘     └──────┴──────┘
```

---

### 3.4 SD-04: Dynamic Document Requirements and Upload

**Scenario:** Determine required documents and handle uploads.  
**Actors:** Applicant Portal, Registration Service, Policy Service, Document Service

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Applicant  │     │ Registration│     │   Policy    │     │  Documents  │
│   Portal    │     │   Service   │     │   Service   │     │   Service   │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │                   │
       │  1. Request Document List             │                   │
       │──────────────────►│                   │                   │
       │                   │  2. Get Required Docs                 │
       │                   │──────────────────►│                   │
       │                   │                   │                   │
       │                   │  DMN: documents-required              │
       │                   │  Inputs: vesselType, activity, length │
       │                   │                   │                   │
       │                   │◄──────────────────│                   │
       │                   │  DocumentList     │                   │
       │◄──────────────────│                   │                   │
       │  Required Documents                   │                   │
       │                   │                   │                   │
       │  3. Upload Document                   │                   │
       │──────────────────►│                   │                   │
       │                   │  4. Store Document│                   │
       │                   │──────────────────────────────────────►│
       │                   │◄──────────────────────────────────────│
       │                   │  DocumentId       │                   │
       │◄──────────────────│                   │                   │
       │  Upload Success   │                   │                   │
       │                   │                   │                   │
       │  (Repeat for each document)           │                   │
       │                   │                   │                   │
       │  5. Check Completeness                │                   │
       │──────────────────►│                   │                   │
       │                   │  6. Validate      │                   │
       │                   │──────────────────────────────────────►│
       │                   │◄──────────────────────────────────────│
       │                   │  CompletenessResult                   │
       │◄──────────────────│                   │                   │
       │  All Required Docs Uploaded           │                   │
       │                   │                   │                   │
└──────┴──────┘     └──────┴──────┘     └──────┴──────┘     └──────┴──────┘
```

---

### 3.5 SD-05: MAFWR Approval Flow (Fishing Vessels)

**Scenario:** Request and process MAFWR approval for fishing vessels.  
**Actors:** Registration Service, Policy Service, Approval Adapter, MAFWR, Notification Service

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Registration│     │   Policy    │     │  Approval   │     │   MAFWR     │     │ Notification│
│   Service   │     │   Service   │     │   Adapter   │     │  (External) │     │   Service   │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │                   │                   │
       │  1. Check MAFWR Required              │                   │                   │
       │──────────────────►│                   │                   │                   │
       │                   │  DMN: mafwr-required                  │                   │
       │◄──────────────────│                   │                   │                   │
       │  isRequired: true │                   │                   │                   │
       │                   │                   │                   │                   │
       │  2. Request MAFWR Approval            │                   │                   │
       │──────────────────────────────────────►│                   │                   │
       │                   │                   │  3. POST /approvals                   │
       │                   │                   │──────────────────►│                   │
       │                   │                   │◄──────────────────│                   │
       │                   │                   │  RequestAccepted  │                   │
       │◄──────────────────────────────────────│                   │                   │
       │  ApprovalRequestId│                   │                   │                   │
       │                   │                   │                   │                   │
       │  4. Update State: AWAITING_MAFWR      │                   │                   │
       │                   │                   │                   │                   │
       │  5. Notify Applicant (async)          │                   │                   │
       │- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -►
       │                   │                   │                   │                   │
       │                   │                   │                   │                   │
       │      ... Time passes (async) ...      │                   │                   │
       │                   │                   │                   │                   │
       │                   │                   │  6. Callback: Decision                │
       │                   │                   │◄──────────────────│                   │
       │                   │                   │                   │                   │
       │  7. Message Correlation (Flowable)    │                   │                   │
       │◄- - - - - - - - - - - - - - - - - - - │                   │                   │
       │  MAFWRApprovalReceived                │                   │                   │
       │                   │                   │                   │                   │
       ├───────────────────┤                   │                   │                   │
       │ [Approved]        │                   │                   │                   │
       │  Continue Flow    │                   │                   │                   │
       │                   │                   │                   │                   │
       ├───────────────────┤                   │                   │                   │
       │ [Rejected]        │                   │                   │                   │
       │  End Process      │                   │                   │                   │
       │                   │                   │                   │                   │
       │  8. Notify Applicant (async)          │                   │                   │
       │- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -►
       │                   │                   │                   │                   │
└──────┴──────┘     └──────┴──────┘     └──────┴──────┘     └──────┴──────┘     └──────┴──────┘
```

---

### 3.6 SD-06: Inspection Flow

**Scenario:** Request and process vessel inspection.  
**Actors:** Registration Service, Policy Service, Inspection Adapter, Inspection Entity

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Registration│     │   Policy    │     │ Inspection  │     │ Inspection  │
│   Service   │     │   Service   │     │   Adapter   │     │   Entity    │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │                   │
       │  1. Check Inspection Required         │                   │
       │──────────────────►│                   │                   │
       │                   │  DMN: inspection-required             │
       │                   │  Inputs: constructionPhase, length,   │
       │                   │          vesselType                   │
       │◄──────────────────│                   │                   │
       │  isRequired: true │                   │                   │
       │                   │                   │                   │
       │  2. Request Inspection                │                   │
       │──────────────────────────────────────►│                   │
       │                   │                   │  3. POST /inspections
       │                   │                   │──────────────────►│
       │                   │                   │◄──────────────────│
       │◄──────────────────────────────────────│  InspectionId     │
       │                   │                   │                   │
       │  Update State: AWAITING_INSPECTION    │                   │
       │                   │                   │                   │
       │      ... Time passes (async) ...      │                   │
       │                   │                   │                   │
       │                   │                   │  4. Callback: Result
       │                   │                   │◄──────────────────│
       │                   │                   │  InspectionReport │
       │  5. Message Correlation               │                   │
       │◄- - - - - - - - - - - - - - - - - - - │                   │
       │  InspectionResultReceived             │                   │
       │                   │                   │                   │
       ├───────────────────┤                   │                   │
       │ [Passed]          │                   │                   │
       │  Continue to Payment                  │                   │
       │                   │                   │                   │
       ├───────────────────┤                   │                   │
       │ [Failed]          │                   │                   │
       │  Reject Request   │                   │                   │
       │                   │                   │                   │
└──────┴──────┘     └──────┴──────┘     └──────┴──────┘     └──────┴──────┘
```

---

### 3.7 SD-07: Fee Calculation, Name Reservation, and Payment

**Scenario:** Calculate fees, reserve vessel name, generate invoice, and process payment.  
**Actors:** Registration Service, Policy Service, Vessel Service, Billing Service, Payment Adapter

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Registration│     │   Policy    │     │   Vessel    │     │   Billing   │     │  Payment    │
│   Service   │     │   Service   │     │   Service   │     │   Service   │     │   Adapter   │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │                   │                   │
       │  1. Calculate Fees│                   │                   │                   │
       │──────────────────►│                   │                   │                   │
       │                   │  DMN: fee-calculation                 │                   │
       │                   │  + DMN: penalty-calculation           │                   │
       │◄──────────────────│                   │                   │                   │
       │  FeeBreakdown     │                   │                   │                   │
       │                   │                   │                   │                   │
       │  2. Reserve Vessel Name               │                   │                   │
       │──────────────────────────────────────►│                   │                   │
       │                   │                   │  Create Reservation                   │
       │                   │                   │  (Hold for X minutes)                 │
       │◄──────────────────────────────────────│                   │                   │
       │  ReservationId    │                   │                   │                   │
       │                   │                   │                   │                   │
       │  3. Generate Invoice                  │                   │                   │
       │──────────────────────────────────────────────────────────►│                   │
       │                   │                   │                   │  Create Invoice   │
       │                   │                   │                   │  + Line Items     │
       │◄──────────────────────────────────────────────────────────│                   │
       │  InvoiceId        │                   │                   │                   │
       │                   │                   │                   │                   │
       │  4. Initiate Payment                  │                   │                   │
       │──────────────────────────────────────────────────────────►│                   │
       │                   │                   │                   │  5. Create Payment│
       │                   │                   │                   │──────────────────►│
       │                   │                   │                   │◄──────────────────│
       │                   │                   │                   │  PaymentUrl       │
       │◄──────────────────────────────────────────────────────────│                   │
       │  PaymentUrl       │                   │                   │                   │
       │                   │                   │                   │                   │
       │  (Applicant redirected to payment)    │                   │                   │
       │                   │                   │                   │                   │
       │      ... Payment processing ...       │                   │                   │
       │                   │                   │                   │                   │
       │                   │                   │                   │  6. Payment Callback
       │                   │                   │                   │◄──────────────────│
       │                   │                   │                   │  PaymentConfirmed │
       │  7. Payment Confirmed (event)         │                   │                   │
       │◄- - - - - - - - - - - - - - - - - - - - - - - - - - - - - │                   │
       │                   │                   │                   │                   │
       │  8. Confirm Name Reservation          │                   │                   │
       │──────────────────────────────────────►│                   │                   │
       │◄──────────────────────────────────────│                   │                   │
       │  NameConfirmed    │                   │                   │                   │
       │                   │                   │                   │                   │
       │  9. Continue to Certificate Issuance  │                   │                   │
       │                   │                   │                   │                   │
└──────┴──────┘     └──────┴──────┘     └──────┴──────┘     └──────┴──────┘     └──────┴──────┘
```

---

### 3.8 SD-08: Certificate Issuance

**Scenario:** Issue temporary registration certificate after payment.  
**Actors:** Registration Service, Certificate Service, Vessel Service, Notification Service, Audit Service

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Registration│     │ Certificate │     │   Vessel    │     │ Notification│     │    Audit    │
│   Service   │     │   Service   │     │   Service   │     │   Service   │     │   Service   │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │                   │                   │
       │  1. Issue Certificate                 │                   │                   │
       │──────────────────►│                   │                   │                   │
       │                   │  2. Get Vessel Data                   │                   │
       │                   │──────────────────►│                   │                   │
       │                   │◄──────────────────│                   │                   │
       │                   │  VesselProfile    │                   │                   │
       │                   │                   │                   │                   │
       │                   │  3. Generate PDF + QR                 │                   │
       │                   │─────────────────────────┐             │                   │
       │                   │◄────────────────────────┘             │                   │
       │                   │                   │                   │                   │
       │                   │  4. Update Vessel Registry            │                   │
       │                   │──────────────────►│                   │                   │
       │                   │◄──────────────────│                   │                   │
       │                   │  RegistryUpdated  │                   │                   │
       │                   │                   │                   │                   │
       │◄──────────────────│                   │                   │                   │
       │  CertificateId    │                   │                   │                   │
       │                   │                   │                   │                   │
       │  5. Update Request State: ISSUED      │                   │                   │
       │─────────────────────────┐             │                   │                   │
       │◄────────────────────────┘             │                   │                   │
       │                   │                   │                   │                   │
       │  6. Publish CertificateIssuedEvent    │                   │                   │
       │- - - - - - - - - - - - - - - - - - - - - - - - - - - - - -►- - - - - - - - - -►
       │                   │                   │                   │                   │
       │                   │                   │                   │  7. Send Notification
       │                   │                   │                   │─────────────────────┐
       │                   │                   │                   │◄────────────────────┘
       │                   │                   │                   │                   │
       │                   │                   │                   │                   │  8. Log Audit
       │                   │                   │                   │                   │───────────────┐
       │                   │                   │                   │                   │◄──────────────┘
       │                   │                   │                   │                   │
└──────┴──────┘     └──────┴──────┘     └──────┴──────┘     └──────┴──────┘     └──────┴──────┘
```

---

### 3.9 SD-09: Post-Issuance Monitoring and Penalties

**Scenario:** Monitor temporary registration and apply penalties if permanent registration delayed.  
**Actors:** Monitoring Service, Policy Service, Billing Service, Notification Service

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Monitoring │     │   Policy    │     │   Billing   │     │ Notification│
│   Service   │     │   Service   │     │   Service   │     │   Service   │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │                   │
       │  1. Scheduled Job: Check Overdue      │                   │
       │─────────────────────────┐             │                   │
       │◄────────────────────────┘             │                   │
       │  List of overdue registrations        │                   │
       │                   │                   │                   │
       │  (For each overdue registration)      │                   │
       │                   │                   │                   │
       │  2. Check Reminder Due                │                   │
       │──────────────────►│                   │                   │
       │                   │  Check CONF-02    │                   │
       │◄──────────────────│                   │                   │
       │  isReminderDue: true                  │                   │
       │                   │                   │                   │
       │  3. Send Reminder │                   │                   │
       │──────────────────────────────────────────────────────────►│
       │                   │                   │                   │  Send SMS/Email
       │◄──────────────────────────────────────────────────────────│
       │                   │                   │                   │
       │  4. Check Penalty Due                 │                   │
       │──────────────────►│                   │                   │
       │                   │  DMN: penalty-calculation             │
       │◄──────────────────│                   │                   │
       │  PenaltyAmount: X │                   │                   │
       │                   │                   │                   │
       ├───────────────────┤                   │                   │
       │ [Penalty > 0]     │                   │                   │
       │                   │                   │                   │
       │  5. Apply Penalty │                   │                   │
       │──────────────────────────────────────►│                   │
       │                   │                   │  Create Invoice   │
       │◄──────────────────────────────────────│                   │
       │  PenaltyInvoiceId │                   │                   │
       │                   │                   │                   │
       │  6. Notify Penalty Applied            │                   │
       │──────────────────────────────────────────────────────────►│
       │                   │                   │                   │  Send Notification
       │◄──────────────────────────────────────────────────────────│
       │                   │                   │                   │
       ├───────────────────┤                   │                   │
       │ [Permanent Reg Completed]             │                   │
       │  Stop Monitoring  │                   │                   │
       │                   │                   │                   │
└──────┴──────┘     └──────┴──────┘     └──────┴──────┘     └──────┴──────┘
```

---

### 3.10 SD-10: Customs Clearance Flow (Imported Vessels)

**Scenario:** Request and process customs clearance for imported vessels.  
**Actors:** Registration Service, Policy Service, Customs Adapter, Bayan

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Registration│     │   Policy    │     │   Customs   │     │    Bayan    │
│   Service   │     │   Service   │     │   Adapter   │     │  (External) │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │                   │
       │  1. Check Customs Required            │                   │
       │──────────────────►│                   │                   │
       │                   │  DMN: customs-required                │
       │                   │  Input: acquisitionPath               │
       │◄──────────────────│                   │                   │
       │  isRequired: true │                   │                   │
       │                   │                   │                   │
       │  2. Request Customs Clearance         │                   │
       │──────────────────────────────────────►│                   │
       │                   │                   │  3. POST /clearance
       │                   │                   │──────────────────►│
       │                   │                   │◄──────────────────│
       │◄──────────────────────────────────────│  RequestAccepted  │
       │  ClearanceRequestId                   │                   │
       │                   │                   │                   │
       │  Update State: AWAITING_CUSTOMS       │                   │
       │                   │                   │                   │
       │      ... Time passes (async) ...      │                   │
       │                   │                   │                   │
       │                   │                   │  4. Receive Import Doc
       │                   │                   │◄──────────────────│
       │  5. Import Document Received          │  ImportDocument   │
       │◄- - - - - - - - - - - - - - - - - - - │                   │
       │                   │                   │                   │
       │                   │                   │  6. Receive Final Clearance
       │                   │                   │◄──────────────────│
       │  7. Customs Clearance Confirmed       │  FinalClearance   │
       │◄- - - - - - - - - - - - - - - - - - - │                   │
       │                   │                   │                   │
       │  Continue to Certificate Issuance     │                   │
       │                   │                   │                   │
└──────┴──────┘     └──────┴──────┘     └──────┴──────┘     └──────┴──────┘
```

---

## 4. Error Handling Sequences

### 4.1 SD-ERR-01: Payment Timeout

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Registration│     │   Vessel    │     │   Billing   │     │ Notification│
│   Service   │     │   Service   │     │   Service   │     │   Service   │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │                   │
       │  Timer Event: Payment Timeout (Flowable)                  │
       │─────────────────────────┐             │                   │
       │◄────────────────────────┘             │                   │
       │                   │                   │                   │
       │  1. Release Name Reservation          │                   │
       │──────────────────►│                   │                   │
       │◄──────────────────│  Released         │                   │
       │                   │                   │                   │
       │  2. Cancel Invoice│                   │                   │
       │──────────────────────────────────────►│                   │
       │◄──────────────────────────────────────│  Cancelled        │
       │                   │                   │                   │
       │  3. Update State: PAYMENT_EXPIRED     │                   │
       │                   │                   │                   │
       │  4. Notify Applicant                  │                   │
       │──────────────────────────────────────────────────────────►│
       │◄──────────────────────────────────────────────────────────│
       │                   │                   │                   │
└──────┴──────┘     └──────┴──────┘     └──────┴──────┘     └──────┴──────┘
```

### 4.2 SD-ERR-02: External System Unavailable

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Registration│     │   Adapter   │     │  External   │
│   Service   │     │  (Any ACL)  │     │   System    │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │
       │  1. Request       │                   │
       │──────────────────►│                   │
       │                   │  2. Call External │
       │                   │──────────────────►│
       │                   │       X  Timeout / Error
       │                   │                   │
       │                   │  3. Retry (exponential backoff)
       │                   │──────────────────►│
       │                   │       X  Still failing
       │                   │                   │
       │◄──────────────────│                   │
       │  EXTERNAL_UNAVAILABLE                 │
       │                   │                   │
       │  4. Circuit Breaker Open              │
       │─────────────────────────┐             │
       │◄────────────────────────┘             │
       │  State: WAITING_EXTERNAL_RECOVERY     │
       │                   │                   │
       │  5. Scheduled Retry Later             │
       │                   │                   │
└──────┴──────┘     └──────┴──────┘     └──────┴──────┘
```

---

## 5. Traceability

| Sequence Diagram | BPMN Subprocess | Capabilities |
|------------------|-----------------|--------------|
| SD-01 | Entry, CC01 | CAP-01, CAP-02 |
| SD-02 | SP02 | CAP-03, CAP-04, CAP-05 |
| SD-03 | SP03 | CAP-06, CAP-07, CAP-08, CAP-09, CAP-10 |
| SD-04 | SP04 | CAP-11, CAP-12 |
| SD-05 | SP05 | CAP-13 |
| SD-06 | SP06 | CAP-14, CAP-15 |
| SD-07 | SP08 | CAP-18, CAP-19, CAP-20, CAP-21 |
| SD-08 | SP09 | CAP-22, CAP-23 |
| SD-09 | SP10 | CAP-24, CAP-25 |
| SD-10 | SP07 | CAP-16, CAP-17 |

---

## 6. Related Documents

- [01-C4_Architecture.md](01-C4_Architecture.md) – C4 model diagrams
- [02-Capability_to_Service_C4.md](02-Capability_to_Service_C4.md) – Service decomposition
- [04-Integration_Architecture.md](04-Integration_Architecture.md) – External system integrations
- BPMN Process Design files in `01-business-analysis/1.3-process/03-BPMN_Process_Design/`

---

**Last Updated:** 2026-01-11  
**Version:** 1.0  
**Author:** Solution Architecture Team
