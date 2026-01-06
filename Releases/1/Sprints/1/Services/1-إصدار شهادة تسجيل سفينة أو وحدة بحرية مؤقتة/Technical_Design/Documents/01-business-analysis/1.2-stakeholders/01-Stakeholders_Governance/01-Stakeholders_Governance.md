# REG001 – Stakeholders & Governance

## Service Identification
- **Service Code (Project):** REG001
- **Service Name (EN):** Issuing Temporary Registration Certificate for a Ship or Marine Unit
- **Service Name (AR):** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة
- **System Service Code:** MTCIT-REG-001
- **Release / Sprint:** 1 / 1

---

## 1. Purpose / الغرض
This document identifies the key stakeholders for REG001 and defines the governance, engagement, and decision rights required to deliver and operate the service.

تهدف هذه الوثيقة إلى تحديد أصحاب المصلحة الرئيسيين لخدمة REG001 وتحديد آليات الحوكمة وإدارة المشاركة وحقوق اتخاذ القرار اللازمة لتطوير وتشغيل الخدمة.

---

## 2. Stakeholder Scope / نطاق أصحاب المصلحة
### In Scope
- Business ownership and operational roles within the maritime registration domain.
- External entities involved in identity validation, commercial registration validation, approvals, inspections, customs clearance, and payments.
- Delivery stakeholders (BA, Architecture, Development, QA, DevOps) for implementation and handover.

### Out of Scope
- Stakeholders for the **permanent registration** lifecycle (handled by the permanent registration service/process).

---

## 3. Stakeholder Map / خريطة أصحاب المصلحة
The stakeholders below are derived from:
- The service source `.docx` (converted to Markdown)
- Service README (Technical Design)
- Scope artifacts (Executive Summary + Scope & Objectives)

### 3.1 Primary Beneficiaries / المستفيدون الرئيسيون
- **Ship/Vessel Owners & Operators / ملاك السفن والمشغلون**: Primary applicants.
- **Authorized Agents / الوكلاء**: Submit requests on behalf of owners.

### 3.2 Service Owner & Internal Business Stakeholders / مالك الخدمة وأصحاب المصلحة الداخليين
- **MTCIT – Directorate General of Maritime Affairs – Registration Department / وزارة النقل والاتصالات وتقنية المعلومات – المديرية العامة للشؤون البحرية – قسم التسجيل**
  - Service owner, policy owner, and final accountable authority for registration decisions.

### 3.3 External Stakeholders (Integrations & Decisions) / أصحاب المصلحة الخارجيون
- **Royal Oman Police (ROP) – PKI / شرطة عمان السلطانية – التصديق الإلكتروني**
  - Identity verification (approve/reject authentication).
- **Oman Business Platform (Invest Easy) / منصة عمان للأعمال**
  - Commercial Registration (CR) validity and activity validation.
- **MAFWR / وزارة الثروة الزراعية والسمكية وموارد المياه**
  - Conditional approval for fishing vessels.
- **Inspection / Classification Entities / جهات المعاينة وهيئات التصنيف**
  - Technical inspection outcomes and compliance confirmation.
- **Bayan Customs / نظام بيان (الجمارك)**
  - Import documentation and customs clearance where applicable.
- **Payment Gateway / بوابة الدفع الإلكتروني**
  - Payment confirmation for fees and penalties.

### 3.4 Delivery & Operations Stakeholders / أصحاب المصلحة للتسليم والتشغيل
- **Product/Service Management (Business Product Owner)**: Prioritization and acceptance.
- **Solution Architecture**: Target architecture, integrations, and NFR alignment.
- **Development Team(s)**: Implementation of orchestration, integration adapters, APIs/events.
- **QA/UAT**: Test strategy, UAT coordination, acceptance sign-off.
- **DevOps/Operations**: Environments, CI/CD, monitoring, incident management.
- **Security/Compliance**: Security controls and audit readiness.

---

## 4. Stakeholder Register / سجل أصحاب المصلحة
> Names and contacts can be completed during discovery workshops.

| Stakeholder / الجهة | Category / التصنيف | Role in REG001 / الدور | Interest / الاهتمام | Influence / التأثير | Notes / ملاحظات |
|---|---|---|---|---|---|
| MTCIT Maritime Affairs – Registration Dept | Internal (Business Owner) | Owns service rules, approves policy, accountable for issuance | High | High | Owner of configuration (documents, validity, bans) |
| Applicant (Owner/Agent) | External (Beneficiary) | Submits request, provides data/docs, pays fees | High | Medium | Primary UX audience |
| ROP PKI | External (Authority) | Identity authentication decision | High | High | Hard gate: success/failure |
| Invest Easy | External (Authority) | CR and activity validation | High | High | Hard gate for company path |
| MAFWR | External (Authority) | Approval for fishing vessels | Medium–High | High | Conditional gate |
| Inspection / Classification | External (Service Provider) | Inspection results, compliance confirmation | Medium–High | High | Triggered by rules (length, phase, type) |
| Bayan Customs | External (Authority) | Customs clearance / import documentation | Medium | High | Conditional gate for imported paths |
| Payment Gateway | External (Platform) | Payment processing and confirmation | High | High | Gate before issuance |
| DevOps/Operations | Internal (Run) | Operates service, monitoring, incident response | High | Medium | SLA/SLO ownership |
| Security/Compliance | Internal (Control) | Security requirements, auditability | High | Medium | Mandatory for government submission |

---

## 5. Roles & Responsibilities / الأدوار والمسؤوليات
### 5.1 Business Ownership
- **Service Owner (MTCIT Registration Dept)**
  - Owns business policy and regulatory interpretation.
  - Approves key rules: vessel age threshold, banned list policy, certificate validity, reminder cadence, penalty triggers.
  - Owns service KPIs and business acceptance.

### 5.2 Data Ownership (High-Level)
- **Applicant-provided Data**: Vessel/unit details, ownership/sales/construction dates, documents.
- **Authoritative External Data**:
  - Identity verification: ROP PKI
  - Company registration/activity: Invest Easy
  - Fishing approval: MAFWR
  - Customs clearance: Bayan
  - Payment confirmation: Payment gateway

### 5.3 Operational Responsibilities
- Operational monitoring of integrations and workflow completion.
- Support handling for rejected requests (with traceable reason) and payment issues.
- Post-issuance reminders and penalty application aligned with configured rules.

---

## 6. Governance Model / نموذج الحوكمة
### 6.1 Decision Rights / حقوق اتخاذ القرار
This section clarifies **who makes each key decision** for REG001.

- **A (Accountable / المسؤول النهائي):** Final decision owner
- **C (Consulted / مستشار):** Provides input before decision
- **I (Informed / مُبلّغ):** Notified after decision

**Policy & regulatory decisions / قرارات السياسات والامتثال**
- **Eligibility policy (citizen/resident) + banned vessels list policy / سياسة الأهلية + قائمة الحظر**
  - **A:** MTCIT Registration
  - **C:** Legal/Compliance, Security
  - **I:** Delivery teams

**Operational rules (configuration) / قرارات الإعدادات التشغيلية**
- **Vessel age threshold + inspection triggers / حد عمر السفينة + محفزات المعاينة**
  - **A:** MTCIT Registration
  - **C:** Inspection entities, Architecture
  - **I:** QA, DevOps
- **Required documents rules (dynamic configuration) / قواعد المستندات المطلوبة (إعدادات ديناميكية)**
  - **A:** MTCIT Registration
  - **C:** BA, Operations
  - **I:** Applicant support
- **Certificate validity (months), reminders cadence, penalty timing / صلاحية الشهادة، دورية التذكير، توقيت المخالفات**
  - **A:** MTCIT Registration
  - **C:** Operations, BA
  - **I:** QA, DevOps

**Delivery & integration decisions / قرارات التنفيذ والتكامل**
- **Integration contract changes (payloads, SLAs) / تغييرات عقود التكامل (البيانات، اتفاقيات مستوى الخدمة)**
  - **A:** Architecture / Integration Owner
  - **C:** External entities, Security
  - **I:** MTCIT Registration, DevOps
- **Release scope and acceptance criteria / نطاق الإصدار ومعايير القبول**
  - **A:** Product/Service Owner + MTCIT Registration
  - **C:** BA, QA
  - **I:** All stakeholders

### 6.2 Governance Cadence / دورية الحوكمة
- **Weekly Working Session (BA + Product + Tech leads):** requirements clarification, backlog grooming, decision log updates.
- **Architecture/Integration Review (bi-weekly or as needed):** integration changes, NFRs, security controls.
- **UAT/Operational Readiness Review (per release):** test readiness, monitoring, runbooks, support model.

### 6.3 Escalation Path / مسار التصعيد
1. BA/Delivery lead → Product/Service Owner
2. Product/Service Owner → MTCIT Registration Service Owner
3. If external dependency block: escalate to the relevant external entity focal point (ROP/Invest Easy/MAFWR/Bayan/Payment)

---

## 7. RACI (Scope-Level) / مصفوفة المسؤوليات (على مستوى النطاق)
**R**: Responsible (executes) | **A**: Accountable (final ownership) | **C**: Consulted | **I**: Informed

| Key Activity / النشاط | Applicant | MTCIT System/Platform | MTCIT Registration (Business Owner) | ROP PKI | Invest Easy | MAFWR | Inspection / Classification | Bayan Customs | Payment Gateway |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Start request, enter data, upload docs | R | A | I | I | I | I | I | I | I |
| Identity authentication | I | A | I | R | - | - | - | - | - |
| CR validity check (company path) | I | A | I | - | R | - | - | - | - |
| Business activity validation | I | A | I | - | R | - | - | - | - |
| Dynamic document rules setup (configuration) | I | C | A/R | - | - | - | C | C | - |
| Fishing vessel approval | - | A | I | - | - | R/A | - | - | - |
| Inspection request and result | - | A | I | - | - | - | R | - | - |
| Customs/import documentation (conditional) | - | A | I | - | - | - | - | R | - |
| Fees/penalties payment | R | A | I | - | - | - | - | - | R |
| Issue temporary certificate | - | A/R | A | - | - | - | - | - | - |
| Post-issuance reminders and overdue penalties | I | A/R | A | - | - | - | - | - | - |

---

## 8. Communication Plan / خطة التواصل
### Core Notifications to Applicant (High-Level)
- Authentication success/failure
- Validation failure reasons (CR/activity/customs/approval/inspection)
- Invoice issued and payment status
- Certificate issued (PDF + QR)
- Reminders to complete permanent registration (periodic)
- Penalty applied (if overdue)

### Internal Communications
- Integration incident alerts (PKI/Invest Easy/MAFWR/Bayan/Payment)
- Workflow failure monitoring dashboards and daily operational review (as needed)

---

## 9. Assumptions & Open Points / افتراضات ونقاط مفتوحة
### Assumptions
- Key thresholds and document lists are configuration-driven (as indicated in the source document).
- External entities provide stable response contracts and availability that meet service needs.

### Open Points to Confirm
1. Named focal points and escalation contacts per external entity (ROP, Invest Easy, MAFWR, Bayan, Payment).
2. Final policy values: vessel age limit, inspection criteria, certificate validity, reminder schedule, penalty schedule.
3. Whether name reservation is an internal capability or separate integrated system (and its owner).

---

## 10. Change Log / سجل التعديلات
| Version | Date | Author | Changes |
|---|---|---|---|
| 1.0 | 2026-01-06 | Copilot | Created Stakeholders & Governance document for REG001 |
