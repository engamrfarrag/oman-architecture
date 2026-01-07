# UC-03 – Vessel Data Capture and Policy Validation

## Primary Actor
Applicant (Owner/Agent)

## Preconditions
- Applicant eligibility validated (SP02 completed).

## Main Flow
1. Applicant selects vessel/unit type and submits core attributes (build year, length/tonnage, activity, category).
2. System validates vessel age policy using DMN/config.
3. Applicant selects acquisition path (new build/purchase/flag change) and enters required dates.
4. System evaluates penalty trigger (e.g., >30 days after construction end) and adds penalty line item when applicable.
5. System checks prohibited/banned vessel policy list.
6. If all checks pass, system proceeds to dynamic documents collection (SP04).

## Alternate / Failure Flows
- **A1 – Age policy failed (G05)**: System blocks progression and allows re-entry where policy permits.
- **A2 – Vessel prohibited (G07)**: System rejects request, notifies applicant, and terminates.

## Postconditions
- Vessel data is validated and persisted OR the request is blocked/rejected.

## Traceability
- User Story: US-04
- BPMN: SP03 (G05, G06, G07)
- Capabilities: CAP-06, CAP-07, CAP-08, CAP-10
- DMN: reg001-vessel-age-validation.dmn, reg001-penalty-calculation.dmn
- Functional Requirements: [FR-SP03.md](../03-functional-requirements/FR-SP03.md) (FR-SP03-001…FR-SP03-004)
