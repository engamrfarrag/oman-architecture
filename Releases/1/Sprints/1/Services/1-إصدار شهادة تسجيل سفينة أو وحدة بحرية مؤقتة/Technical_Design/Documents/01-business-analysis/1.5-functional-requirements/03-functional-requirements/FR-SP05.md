# SP05 – MAFWR Approval (Fishing) (Functional Requirements)

This file consolidates all SP05 functional requirements in a single document, while preserving the original FR IDs for traceability.

---

## FR-SP05-001
Title: Determine whether MAFWR approval is required

Description:
The system shall determine whether MAFWR approval is required based on vessel category/activity and policy.

Trigger:
SP05 routing decision is evaluated.

Inputs:
- Vessel category
- Activity type
- Fishing equipment indicator (if applicable)

Processing / Rules:
- Evaluate `reg001-mafwr-required.dmn` (or equivalent policy mapping).

Outputs:
- MAFWR required (Y/N)

Success Criteria:
- Requests requiring MAFWR approval are routed to approval integration.

Failure Conditions:
- Rules evaluation failure

Related Gateways:
- G09

Related Capabilities:
- CAP-13

Rule Source:
- DMN: `Technical_Design/Resources/dmn/reg001/reg001-mafwr-required.dmn`

---

## FR-SP05-002
Title: Send approval request to MAFWR and track response

Description:
When MAFWR approval is required, the system shall send an approval request to MAFWR and track the response within configured timeout policy.

Trigger:
MAFWR required = Yes.

Inputs:
- Request ID
- Applicant identifiers
- Vessel details relevant to approval

Processing / Rules:
- Send approval request to MAFWR.
- Await decision response.
- Persist approval status and correlation identifiers.

Outputs:
- Approval decision (Approved/Rejected)
- Stored approval reference

Success Criteria:
- MAFWR decision is captured and linked to the request.

Failure Conditions:
- MAFWR timeout/unavailable

Related Gateways:
- G10

Related Capabilities:
- CAP-13

---

## FR-SP05-003
Title: Reject request when MAFWR approval is rejected

Description:
The system shall reject the request when MAFWR returns a rejection decision.

Trigger:
MAFWR decision received.

Inputs:
- Approval status
- Rejection reason (if provided)

Processing / Rules:
- Mark request as Rejected.
- Notify applicant with rejection reason (if provided).

Outputs:
- Rejected request
- Rejection notification

Success Criteria:
- Rejected requests cannot proceed to downstream routing.

Failure Conditions:
- None (business rejection)

Related Gateways:
- G10

Related Capabilities:
- CAP-13, CAP-23

---

## Data Dictionary (SP05)
Aligned with the REG001 Data Dictionary template in `00-overview/Data-Dictionary-Template.md`.

| Data Element (Canonical) | Description | Source / Owner | Data Type | Format / Allowed Values | Validation / Rule Source | Required (Y/N) | Used By (FR IDs) | Example / Notes | Classification |
|---|---|---|---|---|---|---|---|---|---|
| mafwrRequired | Whether approval is needed | LS-APPROVALS/LS-POLICY | boolean | true/false | `reg001-mafwr-required.dmn` | Y | FR-SP05-001 |  | Internal |
| mafwrRequestId | Outbound approval request reference | LS-APPROVALS | string |  | Integration contract | Conditional | FR-SP05-002 | Used for correlation | Internal |
| mafwrDecision | Approval decision | MAFWR/LS-APPROVALS | enum | APPROVED, REJECTED | MAFWR response | Conditional | FR-SP05-002, FR-SP05-003 |  | Internal |
| mafwrDecisionReason | Rejection reason (optional) | MAFWR | string | free text/code | MAFWR response | N | FR-SP05-003 | Included in notification | Internal |
| approvalCorrelationId | Correlation identifier between request and response | LS-APPROVALS | string |  | Orchestrator correlation policy | Conditional | FR-SP05-002 | Should be unique | Internal |
