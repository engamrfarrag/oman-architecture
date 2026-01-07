# UC-07 – Bayan Customs Clearance (Conditional)

## Primary Actor
MTCIT System (REG001 Orchestrator)

## Supporting Actors
Bayan Customs

## Preconditions
- Customs clearance required by policy decision (G13).

## Main Flow
1. System determines if Bayan is required.
2. System sends customs clearance request to Bayan.
3. Bayan returns clearance status and any import documents.
4. System evaluates clearance status.
5. If cleared, the system proceeds to payment.

## Alternate / Failure Flows
- **A1 – Customs rejected (G14)**: System notifies applicant with reason and terminates request.
- **A2 – Bayan timeout**: System holds request in pending customs state and notifies applicant as needed.

## Postconditions
- Clearance recorded as cleared/rejected.

## Traceability
- User Story: US-08
- BPMN: SP07 (G13, G14)
- Capabilities: CAP-16, CAP-17
- DMN: reg001-customs-required.dmn
- Functional Requirements: [FR-SP07.md](../03-functional-requirements/FR-SP07.md) (FR-SP07-001…FR-SP07-003)
