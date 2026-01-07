# SP06 – Inspection Workflow (Functional Requirements)

This file consolidates all SP06 functional requirements in a single document, while preserving the original FR IDs for traceability.

---

## FR-SP06-001
Title: Determine whether inspection is required using DMN/config

Description:
The system shall determine whether vessel inspection is required using configurable rules.

Trigger:
Inspection routing decision is evaluated.

Inputs:
- Vessel length
- Vessel category
- Acquisition type
- Construction phase indicator
- Additional inspection-related indicators (e.g., passengers, classification expiry)

Processing / Rules:
- Evaluate `reg001-inspection-required.dmn`.

Outputs:
- Inspection required (Y/N)
- Inspection type (if required)

Success Criteria:
- Requests requiring inspection are routed to inspection workflow.

Failure Conditions:
- Rules evaluation failure

Related Gateways:
- G11

Related Capabilities:
- CAP-14

Rule Source:
- DMN: `Technical_Design/Resources/dmn/reg001/reg001-inspection-required.dmn`

---

## FR-SP06-002
Title: Create inspection request and capture inspection outcome

Description:
When inspection is required, the system shall create an inspection request, send it to the inspection entity, and capture the inspection outcome/report.

Trigger:
Inspection required = Yes.

Inputs:
- Request ID
- Vessel details
- Inspection type

Processing / Rules:
- Send inspection request.
- Await response within configured timeout.
- Store inspection report reference and outcome.

Outputs:
- Inspection outcome (Passed/Failed/Cancelled/Timeout)
- Inspection report reference

Success Criteria:
- Inspection result is captured and linked to the request.

Failure Conditions:
- Inspection timeout/unavailable

Related Gateways:
- G12

Related Capabilities:
- CAP-15

---

## FR-SP06-003
Title: Block progression when inspection outcome is not passed

Description:
The system shall block progression when the inspection outcome is Failed/Cancelled/Timeout and notify the applicant.

Trigger:
Inspection outcome received or timeout occurs.

Inputs:
- Inspection outcome status
- Failure reason (if provided)

Processing / Rules:
- Update request state (Rejected or Pending Escalation) per policy.
- Notify applicant.

Outputs:
- Blocked request state
- Applicant notification

Success Criteria:
- Only Passed outcomes allow continuation.

Failure Conditions:
- None (business decision)

Related Gateways:
- G12

Related Capabilities:
- CAP-15, CAP-23

---

## Data Dictionary (SP06)
Aligned with the REG001 Data Dictionary template in `00-overview/Data-Dictionary-Template.md`.

| Data Element (Canonical) | Description | Source / Owner | Data Type | Format / Allowed Values | Validation / Rule Source | Required (Y/N) | Used By (FR IDs) | Example / Notes | Classification |
|---|---|---|---|---|---|---|---|---|---|
| inspectionRequired | Whether inspection is needed | LS-POLICY | boolean | true/false | `reg001-inspection-required.dmn` | Y | FR-SP06-001 |  | Internal |
| inspectionType | Inspection type/category | LS-POLICY/LS-INSPECTION | string/enum | config-driven | DMN/config | Conditional | FR-SP06-001, FR-SP06-002 |  | Internal |
| inspectionRequestId | Outbound inspection request reference | LS-INSPECTION | string |  | Inspection integration contract | Conditional | FR-SP06-002 | Used for correlation | Internal |
| inspectionOutcome | Inspection result | Inspection provider | enum | PASSED, FAILED, CANCELLED, TIMEOUT | Provider response | Conditional | FR-SP06-002, FR-SP06-003 | Only PASSED continues | Internal |
| inspectionReportRef | Report/document reference | LS-INSPECTION/Storage | string | URI/id | Storage service | N | FR-SP06-002 | May be a PDF | Internal |
| inspectionFailureReason | Failure reason (optional) | Inspection provider | string | free text/code | Provider response | N | FR-SP06-003 | Included in notification | Internal |
