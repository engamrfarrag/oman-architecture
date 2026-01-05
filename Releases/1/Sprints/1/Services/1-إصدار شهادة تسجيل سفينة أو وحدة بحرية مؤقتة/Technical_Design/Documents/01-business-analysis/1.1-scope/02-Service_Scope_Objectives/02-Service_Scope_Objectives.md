# REG001 – Service Scope & Objectives

## Service Identification
- **Service Code (Project):** REG001
- **Service Name (EN):** Issuing Temporary Registration Certificate for a Ship or Marine Unit
- **Service Name (AR):** إصدار شهادة تسجيل سفينة أو وحدة بحرية مؤقتة
- **System Service Code:** MTCIT-REG-001
- **Release / Sprint:** 1 / 1

---

## 1. Scope Statement / بيان النطاق
REG001 covers the end-to-end business capability of **issuing a temporary vessel/marine unit registration certificate**, including required validations, document intake, approvals/inspections (where applicable), fee/penalty handling, payment, and issuance/notification.

تشمل الخدمة دورة عمل **إصدار شهادة تسجيل مؤقتة** للوحدة البحرية بما في ذلك التحقق من الهوية والسجل التجاري، جمع المستندات، الحصول على الموافقات/المعاينات عند الحاجة، احتساب الرسوم/المخالفات، السداد، ثم إصدار الشهادة وإرسال الإشعارات.

---

## 2. Objectives / الأهداف
### Business Objectives
- Provide a legally compliant temporary registration instrument to enable timely vessel operation.
- Standardize validations and approvals to reduce manual processing and variability.
- Ensure traceability and auditability across all decision points.

### Operational Objectives
- Automate routing to approvals/inspection paths based on configured rules.
- Integrate with external systems to validate identity, CR, customs, and fishing approvals.
- Provide proactive reminders to complete permanent registration and enforce penalties when overdue.

---

## 3. In-Scope Capabilities / القدرات ضمن النطاق
1. **User initiation and guidance**: start request, view instructions.
2. **Identity authentication (ROP PKI)**: proceed/terminate based on verification result.
3. **Applicant type handling**: Individual vs Company.
4. **Company validations**:
   - Commercial Registration validity check (Invest Easy)
   - Activity validation aligned to vessel/unit type
5. **Vessel/unit data capture and validation**:
   - Unit type selection
   - Year built capture and age policy validation (config-driven)
   - Construction/sale path selection and date capture
6. **Penalty rule (example from source)**: if more than **30 days** after construction end date at submission, add a penalty line item.
7. **Dynamic document intake**: required documents vary based on configuration, vessel type, and path.
8. **MAFWR approval** (conditional): required for fishing vessels.
9. **Inspection orchestration** (conditional): invoked for certain criteria (e.g., construction phase, length > 24m, tug boat, imported vessel path).
10. **Name reservation** (conditional): temporary name reservation during payment window.
11. **Fees and payment**: invoice generation, payment confirmation, receipt.
12. **Certificate issuance**: temporary certificate output (e.g., PDF + QR), notification.
13. **Customs/Bayan integration** (conditional): request and receive import document/customs clearance.
14. **Notifications & reminders**:
   - Rejection/approval/payment/certificate issued
   - Reminder every configured interval (example: every 7 days) to complete permanent registration
   - Stop reminders when permanent registration is completed
   - Apply overdue penalty per configuration

---

## 4. Out of Scope / خارج النطاق
- Permanent registration lifecycle (treated as a separate service or downstream process).
- Detailed technical API design (OpenAPI), event schema design, and integration payload definitions (Solution Design phase).
- Full BPMN diagrams and all exception paths (covered in Process/BPMN deliverables).
- UI/UX design artifacts beyond what is needed to describe scope.

---

## 5. Actors & Stakeholders / الجهات والأطراف
### Primary Actors
- Applicant (Owner/Agent) / المستفيد

### Internal Stakeholders
- Maritime Affairs Registration Department (Service Owner) / قسم التسجيل

### External Stakeholders & Integrations
- **ROP PKI**: identity authentication
- **Invest Easy (Oman Business Platform)**: CR and activity validation
- **MAFWR**: fishing vessel approval
- **Inspection / Classification entities**: technical inspection outcomes
- **Bayan Customs**: customs clearance and import documents
- **Payment gateway**: fee/penalty payment

---

## 6. Preconditions & Eligibility / الشروط المسبقة والأهلية
- Applicant is an owner or authorized agent.
- Applicant is an Omani citizen or resident (per source).
- Vessel/unit is not on a banned list (policy managed via configuration).

---

## 7. High-Level Process (Scope View) / نظرة عامة على سير العمل
1. Applicant selects service and starts request.
2. System performs ROP PKI authentication.
3. Applicant indicates Individual/Company; system validates CR/activity for companies.
4. Applicant captures vessel/unit data; system validates policies (age, banned list, etc.).
5. Applicant uploads required documents (dynamic rules).
6. Conditional routing:
   - Fishing vessel → MAFWR approval
   - Requires inspection → inspection workflow and validation
   - Imported (<24m scenario in source) → Bayan integration for import docs/clearance
7. Invoice generation and payment.
8. Certificate issuance and notifications.
9. Post-issuance reminders to complete permanent registration; penalties for overdue.

---

## 8. Key Business Rules (Scope-Level) / القواعد الرئيسية
- **Identity verification is mandatory**; failure terminates the request.
- **Company path requires CR validity** and appropriate commercial activity.
- **Vessel age policy is configurable**; exceeding the threshold blocks issuance (with warning/re-entry path in source).
- **Document requirements are configurable** and may change without code changes.
- **Penalty timing and certificate validity are configurable** (months, reminder intervals).

---

## 9. Deliverables (This Scope Package) / المخرجات
- Executive Summary
- Service Scope & Objectives (this document)

---

## 10. Open Points / نقاط بحاجة لتأكيد
1. Confirm official thresholds: vessel age limit, inspection triggers, penalty schedule.
2. Confirm the exact certificate validity period (months) and reminder cadence.
3. Confirm required documents per vessel type/activity (configuration source of truth).
4. Confirm whether name reservation is a separate integrated service or internal capability.

---

## 11. Source & Traceability / المصادر
- Service README: `Technical_Design/Documents/README.md`
- Converted service source document (from service `.docx`)
- Note: `Technical_Design/Documents/00-proposal/offer.md` is currently missing in the service folder.
