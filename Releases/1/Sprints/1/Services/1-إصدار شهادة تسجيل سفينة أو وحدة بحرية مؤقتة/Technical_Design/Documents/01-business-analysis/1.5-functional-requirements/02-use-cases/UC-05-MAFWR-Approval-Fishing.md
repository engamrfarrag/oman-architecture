# UC-05 – MAFWR Approval (Fishing Vessels)

## Primary Actor
MTCIT System (REG001 Orchestrator)

## Supporting Actors
MAFWR

## Preconditions
- Documents complete (SP04 completed).
- Vessel category/activity indicates fishing approval is required (G09).

## Main Flow
1. System determines if MAFWR approval is required.
2. System sends approval request to MAFWR.
3. System receives approval decision.
4. If approved, system proceeds to downstream routing/payment.

## Alternate / Failure Flows
- **A1 – MAFWR rejected (G10)**: System rejects request, notifies applicant with reason, terminates.
- **A2 – MAFWR timeout**: System holds request in pending approval state and notifies applicant as needed.

## Postconditions
- Approval recorded as approved/rejected.

## Traceability
- User Story: US-06
- BPMN: SP05 (G09, G10)
- Capability: CAP-13
- DMN: reg001-mafwr-required.dmn
- Functional Requirements: [FR-SP05.md](../03-functional-requirements/FR-SP05.md) (FR-SP05-001…FR-SP05-003)
