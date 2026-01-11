# FR-CC01 – Security & Access Control

> **Service:** REG001 – Temporary Vessel Registration  
> **Type:** Cross-Cutting Functional Requirements  
> **Last Updated:** 2026-01-09

---

## Overview

This document defines security and access control rules for REG001, extracted from official service specification documents. These rules are expressed in **business language only** and do not reference technical implementation details.

---

## 1. Authentication Rules

### FR-CC01-001
**Title:** Enforce ROP PKI authentication gate

**Description:**
The service requires verified identity through official government authentication before allowing access to REG001 subprocesses.

**Trigger:**
Applicant starts REG001 or resumes a draft requiring authentication.

**Inputs:**
- Applicant identity attributes (national ID / digital identity attributes)

**Processing / Rules:**
- Send authentication request to ROP PKI.
- Await PKI response within configured timeout.

**Outputs:**
- Authentication decision: Authenticated / Rejected

**Success Criteria:**
- Authenticated applicants can proceed to SP02.

**Failure Conditions:**
- PKI rejected
- PKI timeout/unavailable

**Related Gateways:** N/A (Cross-cutting CC01)  
**Related Capabilities:** CAP-02

---

### FR-CC01-002
**Title:** Notify and terminate on PKI rejection

**Description:**
The system shall notify the applicant and terminate the REG001 request when ROP PKI returns a rejection.

**Trigger:**
ROP PKI returns rejection status.

**Inputs:**
- PKI rejection status
- Rejection reason (if provided)

**Processing / Rules:**
- Log the rejection decision for audit.
- Send applicant notification with generic + reason (if available).
- Mark request as terminated.

**Outputs:**
- Rejection notification
- Terminated request state

**Success Criteria:**
- Rejected applicants cannot access SP02+.

**Failure Conditions:**
- Notification failure (must be logged and retried per policy)

**Related Gateways:** N/A (Cross-cutting CC01)  
**Related Capabilities:** CAP-02, CAP-23, CAP-26

---

## 2. Resource Ownership Rules

### FR-CC01-003
**Title:** Resource ownership eligibility

**Description:**
Actions on a vessel registration request are permitted only to its rightful owner or authorized representative.

**Business Rule:**
> مالك السفينة / الموكل من مالك السفينة  
> (Vessel owner / Authorized representative of vessel owner)

**Ownership Types:**

| Type | Arabic | Description |
|------|--------|-------------|
| **Individual Owner** | مالك (فرد) | Omani citizen or resident who owns the vessel directly |
| **Organizational Owner** | مالك (شركة) | Legal entity (company) that owns the vessel |
| **Authorized Representative** | موكل | Person with valid legal delegation (power of attorney) to act on behalf of owner |

**Eligibility Conditions:**
- Individual must be Omani citizen or resident (مواطن عماني / مقيم)
- Company must have valid commercial registry (سجل تجاري صالح)
- Representative must have valid delegation document (وكالة)

**Access Pattern:**
- Only the owner or their authorized representative may:
  - Submit a registration request
  - Upload documents
  - View request status
  - Receive certificates

**Related Capabilities:** CAP-01, CAP-03, CAP-04

---

### FR-CC01-004
**Title:** Commercial registry verification for organizations

**Description:**
When the applicant is a company (شركة), the system shall verify valid commercial registry before allowing access to vessel registration functions.

**Trigger:**
Applicant selects "Company" as applicant type.

**Inputs:**
- Commercial registry number (رقم السجل التجاري)
- Business activity type

**Processing / Rules:**
- Query Oman Business Platform (منصة عمان للأعمال) for registry validity
- Verify the applicant is authorized to act for the company

**Outputs:**
- Registry valid / invalid status
- Access granted / denied

**Success Criteria:**
- Only users with valid commercial registry can proceed for company registrations.

**Failure Conditions:**
- Invalid or expired commercial registry → Reject with notification

**Related Gateways:** G03  
**Related Capabilities:** CAP-03, CAP-04

---

### FR-CC01-005
**Title:** Business activity verification

**Description:**
The applicant must have a valid business activity matching the vessel type.

**Business Rule:**
> التحقق من النشاط طبقا لنوع الوحدة  
> (Verify activity according to vessel type)

**Inputs:**
- Applicant's business activity (from منصة عمان للأعمال)
- Selected vessel type

**Processing / Rules:**
- Query for active business activity
- Match activity type with vessel category

**Success Criteria:**
- نشاط تجاري ساري (Active business activity) → Allow continuation

**Failure Conditions:**
- نشاط غير ساري (Inactive activity) → Reject with notification

**Related Gateways:** G04  
**Related Capabilities:** CAP-04

---

## 3. API Access Rules (Organizational)

### FR-CC01-006
**Title:** Organizational access restrictions

**Description:**
Access to service functions is restricted to authorized organizational units within MTCIT.

**Organizational Units:**

| Department | Arabic | Permitted Functions |
|------------|--------|---------------------|
| Registration Department | إدارة التسجيل | Create, approve, issue certificates |
| Inspection Department | قسم المعاينات | View inspections, update results |
| Finance Department | قسم المالية | View invoices, confirm payments |
| MAFWR | وزارة الثروة الزراعية | Approve/reject fishing vessel requests |

**Business Rule:**
- Only authorized employees (موظف) within each department may access corresponding functions.
- External users (applicants) may only access submission and status viewing functions.

**Related Capabilities:** CAP-02, CAP-16, CAP-17

---

### FR-CC01-007
**Title:** Authority approval workflow access

**Description:**
Internal authority review functions are accessible only to designated staff within the relevant department.

**Roles by Department:**

| Function | Permitted By |
|----------|--------------|
| Review registration request | وزارة النقل - قسم التسجيل |
| Approve fishing vessel | وزارة الثروة الزراعية |
| Conduct inspection | هيئات التصنيف / الشركات الاستشارية |
| Confirm payment | MTCIT - Finance |

**Related Gateways:** G08, G09, G10  
**Related Capabilities:** CAP-16, CAP-17, CAP-18

---

## 4. Delegation & Representation Rules

### FR-CC01-008
**Title:** Legal delegation for vessel operations

**Description:**
A representative may act on behalf of the owner with valid legal authorization (power of attorney).

**Business Rule:**
> الموكل من مالك السفينة  
> (Authorized by vessel owner)

**Requirements:**
- Valid power of attorney document (وكالة)
- Delegation scope must include vessel registration activities
- Delegation must be current and not expired

**Permitted Actions for Representative:**
- Submit registration request on behalf of owner
- Upload documents
- Receive notifications
- Collect certificates (if authorized)

**Restrictions:**
- Representative cannot transfer ownership
- Representative cannot delegate to another party

**Related Capabilities:** CAP-01, CAP-03

---

## 5. Prohibited Access Rules

### FR-CC01-009
**Title:** Blocked vessels restriction

**Description:**
The system shall prevent registration of vessels that are prohibited from operating in Oman.

**Business Rule:**
> السفينة ليست من السفن المحظور استقبالها  
> (Vessel is not among prohibited vessels)

**Configuration:**
- REG-001-CONF-04: إعداد السفن المحظورة للتعامل في سلطنة عمان
- (Configuration of vessels prohibited from operating in Oman)

**Processing:**
- Check vessel against prohibited list before allowing registration
- Block registration if vessel is on prohibited list

**Related Capabilities:** CAP-07

---

## 6. Security Traceability

### Arabic Terms Mapping

| Arabic Term | English | Security Interpretation |
|-------------|---------|------------------------|
| مالك السفينة | Vessel Owner | Resource owner (individual or organization) |
| موكل | Authorized Representative | Legal delegation holder |
| وكالة | Power of Attorney | Delegation document |
| شركة | Company | Organizational owner type |
| فرد | Individual | Individual owner type |
| سجل تجاري | Commercial Registry | Legal entity identifier |
| موظف | Employee | Internal staff with API access |
| التصديق الالكتروني | Electronic Authentication | Identity verification |
| صلاحية | Permission/Validity | Access right or document validity |
| نشاط تجاري | Business Activity | Commercial activity license |

---

## 7. Acceptance Criteria Summary

### Authentication
- [ ] Authenticated applicants can access registration functions
- [ ] Rejected applicants receive notification and cannot proceed
- [ ] All authentication decisions are logged

### Ownership
- [ ] Only owners or authorized representatives can act on vessel requests
- [ ] Company applicants must have valid commercial registry
- [ ] Business activity must match vessel type

### Delegation
- [ ] Representatives with valid power of attorney can act for owners
- [ ] Delegation scope is validated before allowing actions

### Access Control
- [ ] Internal functions are restricted to authorized departments
- [ ] Prohibited vessels cannot be registered

---

**Document Control:**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-07 | System | Initial authentication rules |
| 2.0 | 2026-01-09 | System | Added ownership, delegation, and access control rules per updated BA security standards |
