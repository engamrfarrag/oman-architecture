# UC-06 – Inspection Workflow (Conditional)

## Primary Actor
MTCIT System (REG001 Orchestrator)

## Supporting Actors
Inspection Entity (Classification Society / Consultancy / Internal)

## Preconditions
- Inspection required by policy decision (G11).

## Main Flow
1. System determines inspection requirement using DMN/config.
2. System sends an inspection request to the inspection entity.
3. Inspection entity performs inspection and returns outcome/report.
4. System evaluates inspection outcome.
5. If passed, the system proceeds to customs (if required) and/or payment.

## Alternate / Failure Flows
- **A1 – Inspection failed/cancelled/timeout (G12)**: System notifies applicant and terminates or escalates per policy.

## Postconditions
- Inspection outcome recorded; request proceeds or is terminated/escalated.

## Traceability
- User Story: US-07
- BPMN: SP06 (G11, G12)
- Capabilities: CAP-14, CAP-15
- DMN: reg001-inspection-required.dmn
- Functional Requirements: [FR-SP06.md](../03-functional-requirements/FR-SP06.md) (FR-SP06-001…FR-SP06-003)
