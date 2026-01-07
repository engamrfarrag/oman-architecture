# FR-00 – Overview (REG001)

## 1. Scope of This FR Package
This Functional Requirements package covers the end-to-end REG001 flow:
- Authenticate applicant (ROP PKI)
- Profile applicant (Individual / Company) + validate CR/activity where applicable
- Capture vessel/unit data + policy checks (age, prohibited list) + penalty triggers
- Determine and collect required documents dynamically
- Conditional approvals and routing: MAFWR approval, Inspection, Bayan/customs
- Fees calculation + name reservation + invoicing + payment
- Temporary certificate issuance + notifications
- Post-issuance reminders and overdue penalties until permanent registration completes

## 2. Out of Scope
- Detailed UI/UX design (only functional behaviors and validations)
- Detailed data model (handled under Solution Design)
- Permanent registration service implementation (only status dependency in SP10)

## 3. Terminology
- **Applicant:** Owner or authorized agent (individual/company).
- **Request/Case:** A REG001 application instance.
- **SPxx:** BPMN subprocess document (source of process-level traceability).
- **CAP-xx:** Capability ID from capability mapping.
- **DMN:** Decision Model and Notation rules used for configurable business decisions.

## 4. Key Business Rules (Source Anchors)
- Vessel age policy validation: `reg001-vessel-age-validation.dmn` (SP03 / G05)
- Penalty calculation (timing-based): `reg001-penalty-calculation.dmn` (SP03 / G06)
- Dynamic document requirements: `reg001-documents-required.dmn` + config (SP04 / G08)
- MAFWR approval requirement: `reg001-mafwr-required.dmn` (SP05 / G09)
- Inspection requirement: `reg001-inspection-required.dmn` (SP06 / G11)
- Customs requirement: `reg001-customs-required.dmn` (SP07 / G13)
- Fee calculation: `reg001-fee-calculation.dmn` (SP08)

## 5. Traceability Rules
- Each **Use Case** references: User Story + BPMN SP + gateways (Gxx) + CAP-xx.
- Each **FR** references: SP + gateway(s) (if applicable) + CAP-xx + DMN/config/integration rule source.

## 6. Assumptions
- Payment/name reservation expiry durations are configuration-driven.
- Notifications (SMS/Email/in-app) are delivered by a shared notifications capability.
- Where integration payload fields are not finalized, requirements specify minimum mandatory fields and behaviors; detailed schemas move to API Contracts later.
