# Actors and Roles (REG001)

> **Service:** REG001 – Temporary Vessel Registration  
> **Last Updated:** 2026-01-09

---

## 1. Primary Actors

### Applicant (Owner/Agent)
- Initiates request, provides data, uploads documents, selects vessel name, pays fees.

**Ownership Types:**

| Type | Arabic | Description | Eligibility |
|------|--------|-------------|-------------|
| **Individual Owner** | مالك (فرد) | Person who owns the vessel directly | Omani citizen or resident (مواطن عماني / مقيم) |
| **Organizational Owner** | مالك (شركة) | Legal entity that owns the vessel | Valid commercial registry (سجل تجاري صالح) |
| **Authorized Representative** | موكل | Person with valid power of attorney | Valid delegation document (وكالة) |

**Permitted Actions:**
- Submit registration request
- Upload required documents
- View request status
- Make payments
- Receive certificates

---

## 2. System Actors

### MTCIT System (REG001 Orchestrator)
- Orchestrates the end-to-end process, applies rules, routes to integrations, issues certificate.

---

## 3. External Actors / Integrations

| Actor | Arabic | Function |
|-------|--------|----------|
| **ROP PKI** | شرطة عمان السلطانية | Identity authentication (التصديق الالكتروني) |
| **Invest Easy** | منصة عمان للأعمال | Commercial registry & activity validation |
| **MAFWR** | وزارة الثروة الزراعية والسمكية | Fishing vessel approval |
| **Inspection Entity** | هيئات التصنيف / الشركات الاستشارية | Inspection outcomes and reports |
| **Bayan Customs** | نظام بيان | Import documents and customs clearance |
| **Payment Gateway** | بوابة الدفع الإلكتروني | Payment execution and confirmation |

---

## 4. Internal Authority Actors

| Department | Arabic | Responsibility |
|------------|--------|----------------|
| **Registration Department** | إدارة التسجيل | Create, review, approve, issue certificates |
| **Inspection Department** | قسم المعاينات | View inspections, update results |
| **Finance Department** | قسم المالية | View invoices, confirm payments |

---

## 5. Stakeholders (RACI Reference)

| # | Stakeholder | Arabic |
|---|-------------|--------|
| 1 | MTCIT – Maritime Affairs Directorate, Registration Section | وزارة النقل والاتصالات - قسم التسجيل |
| 2 | Royal Oman Police (ROP – NRS/Bayan) | شرطة عمان السلطانية |
| 3 | Oman Business Platform | منصة عمان للأعمال |
| 4 | MAFWR | وزارة الثروة الزراعية والسمكية وموارد المياه |
| 5 | Classification Societies | هيئات التصنيف |
| 6 | Consulting Companies | الشركات الاستشارية |
| 7 | Vessel Owners / Operators | ملاك السفن / المشغلين |

---

## 6. Access Control Summary

### Resource Ownership Rule
> Actions on a vessel registration request are permitted only to its rightful owner or authorized representative.

### Organizational Access Rule
> Access to internal service functions is restricted to authorized staff within the relevant department.

### Delegation Rule
> A representative may act on behalf of the owner with valid legal authorization (power of attorney).

---

## 7. Role Notes

- Applicant may be an **individual** (فرد) or a **company representative** (ممثل شركة).
- Company applicants must have valid commercial registry from منصة عمان للأعمال.
- Representatives must have valid power of attorney (وكالة).
- Some approvals are conditional based on **vessel category**, **activity**, and **routing rules** (DMN/config).
- Internal staff access is restricted to their departmental functions only.
