# SP02 – Applicant Profiling & Eligibility (Functional Requirements)

This file consolidates all SP02 functional requirements in a single document, while preserving the original FR IDs for traceability.

---

## FR-SP02-001
Title: Capture beneficiary type and applicant identifiers

Description:
The system shall capture the beneficiary type (Individual/Company) and required applicant identifiers to determine eligibility routing.

Trigger:
Applicant proceeds after successful PKI authentication.

Inputs:
- Beneficiary type (Individual/Company)
- Applicant identity attributes
- Company CR number (if Company)

Processing / Rules:
- If beneficiary type = Individual, bypass CR/activity validations.
- If beneficiary type = Company, enforce CR + activity validations.

Outputs:
- Beneficiary profile state: Individual or Company

Success Criteria:
- The beneficiary type and identifiers are stored and available for SP03.

Failure Conditions:
- Missing mandatory fields (e.g., CR number when Company)

Related Gateways:
- G02

Related Capabilities:
- CAP-03

---

## FR-SP02-002
Title: Validate company Commercial Registration (CR) via Invest Easy

Description:
When the beneficiary type is Company, the system shall validate the company Commercial Registration (CR) using Invest Easy.

Trigger:
Beneficiary type = Company.

Inputs:
- CR number
- Company identifiers (as required by integration)

Processing / Rules:
- Send validation request to Invest Easy.
- Await response within configured timeout.

Outputs:
- CR validation status: Valid/Invalid

Success Criteria:
- Valid CR allows the request to proceed to activity eligibility validation.

Failure Conditions:
- Invest Easy timeout/unavailable

Related Gateways:
- G03

Related Capabilities:
- CAP-04

---

## FR-SP02-004
Title: Reject request when CR is invalid

Description:
The system shall reject the request when Invest Easy returns CR validation status = Invalid.

Trigger:
CR validation response received.

Inputs:
- CR validation status
- Reason (if provided)

Processing / Rules:
- Update request status to Rejected.
- Notify applicant.

Outputs:
- Rejected request
- Rejection notification

Success Criteria:
- The request cannot proceed to SP03.

Failure Conditions:
- None (business rejection)

Related Gateways:
- G03

Related Capabilities:
- CAP-04, CAP-23

---

## FR-SP02-003
Title: Validate company activity eligibility aligned to vessel type/activity

Description:
When the beneficiary type is Company and CR is valid, the system shall validate that the company has an active/eligible activity aligned to the selected vessel/unit type and intended activity.

Trigger:
CR status = Valid.

Inputs:
- CR number
- Selected vessel category/type
- Intended activity classification

Processing / Rules:
- Request activity status/eligibility from Invest Easy.
- Evaluate eligibility mapping policy (config/policy).

Outputs:
- Activity eligibility status: Eligible / Not eligible

Success Criteria:
- Eligible activity allows the request to proceed to SP03.

Failure Conditions:
- Invest Easy timeout/unavailable

Related Gateways:
- G04

Related Capabilities:
- CAP-05

---

## FR-SP02-005
Title: Reject request when activity is not eligible

Description:
The system shall reject the request when activity eligibility is Not eligible.

Trigger:
Activity validation response received.

Inputs:
- Activity eligibility status
- Reason (if available)

Processing / Rules:
- Update request status to Rejected.
- Notify applicant.

Outputs:
- Rejected request
- Rejection notification

Success Criteria:
- The request cannot proceed to SP03.

Failure Conditions:
- None (business rejection)

Related Gateways:
- G04

Related Capabilities:
- CAP-05, CAP-23

---

## Data Dictionary (SP02)
Aligned with the REG001 Data Dictionary template in `00-overview/Data-Dictionary-Template.md`.

| Data Element (Canonical) | Description | Source / Owner | Data Type | Format / Allowed Values | Validation / Rule Source | Required (Y/N) | Used By (FR IDs) | Example / Notes | Classification |
|---|---|---|---|---|---|---|---|---|---|
| beneficiaryType | Applicant classification | Applicant UI / LS-APPLICANT | enum | INDIVIDUAL, COMPANY | UI validation | Y | FR-SP02-001 | Drives eligibility routing | Internal |
| applicantCivilId | Applicant national/civil id used for identification | IAM / Applicant UI | string | per ROP PKI | ROP PKI contract | Y | FR-SP02-001 | Store masked where applicable | Confidential/PII |
| companyCrNumber | Commercial Registration number | Applicant UI | string | Invest Easy format | Required if `beneficiaryType=COMPANY` | Conditional | FR-SP02-001, FR-SP02-002 |  | Internal |
| crValidationStatus | CR validation outcome | LS-CR-ADAPTER | enum | VALID, INVALID | Invest Easy response | Conditional | FR-SP02-002, FR-SP02-004 |  | Internal |
| crRejectionReason | CR rejection reason | Invest Easy | string | free text/code | Invest Easy response | N | FR-SP02-004 | Optional | Internal |
| activityCode | Business activity code/classification | Applicant UI / Invest Easy | string | Invest Easy activity code | Invest Easy contract + mapping config | Conditional | FR-SP02-003 |  | Internal |
| activityEligibilityStatus | Eligibility outcome | LS-ACTIVITY-VALIDATION | enum | ELIGIBLE, NOT_ELIGIBLE | Policy mapping (config) + Invest Easy | Conditional | FR-SP02-003, FR-SP02-005 |  | Internal |
| requestStatus | Overall case state | LS-REG-ORCH | enum | IN_PROGRESS, REJECTED, ... | Orchestrator policy | Y | FR-SP02-004, FR-SP02-005 | Rejected blocks SP03 | Internal |
