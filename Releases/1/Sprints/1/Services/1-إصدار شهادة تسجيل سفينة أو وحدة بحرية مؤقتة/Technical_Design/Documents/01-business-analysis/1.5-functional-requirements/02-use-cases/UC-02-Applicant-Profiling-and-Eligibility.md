# UC-02 – Applicant Profiling and Eligibility (CR/Activity)

## Primary Actor
Applicant (Owner/Agent)

## Supporting Actors
Invest Easy (Oman Business Platform)

## Preconditions
- Applicant authenticated successfully (CC01).

## Main Flow
1. Applicant selects beneficiary type (Individual/Company).
2. If Individual, system proceeds to vessel data capture (SP03).
3. If Company, applicant submits company CR.
4. System validates CR via Invest Easy.
5. If CR valid, system validates company activity eligibility aligned to vessel type/activity.
6. If eligible, system proceeds to SP03.

## Alternate / Failure Flows
- **A1 – CR invalid (G03)**: System rejects request and notifies applicant.
- **A2 – Activity not eligible (G04)**: System rejects request and notifies applicant.
- **A3 – Invest Easy timeout**: System informs applicant and holds the request in a pending state per policy.

## Postconditions
- Beneficiary profile is confirmed OR the request is rejected.

## Traceability
- User Story: US-03
- BPMN: SP02 (G02, G03, G04)
- Capabilities: CAP-03, CAP-04, CAP-05
- Functional Requirements: [FR-SP02.md](../03-functional-requirements/FR-SP02.md) (FR-SP02-001…FR-SP02-005)
