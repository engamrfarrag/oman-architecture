# UC-04 – Dynamic Documents Collection

## Primary Actor
Applicant (Owner/Agent)

## Preconditions
- Vessel data validated (SP03 completed).

## Main Flow
1. System determines required documents dynamically based on vessel attributes and policy rules.
2. System displays the required documents list (mandatory/optional).
3. Applicant uploads documents.
4. System validates completeness and basic constraints.
5. If complete, system proceeds to conditional routing (SP05/SP06/SP07) and then payment.

## Alternate / Failure Flows
- **A1 – Documents incomplete (G08)**: System returns missing items and loops back to upload until complete.

## Postconditions
- Complete document set is attached to request OR request remains in rework state.

## Traceability
- User Story: US-05
- BPMN: SP04 (G08)
- Capabilities: CAP-11, CAP-12
- DMN: reg001-documents-required.dmn
- Functional Requirements: [FR-SP04.md](../03-functional-requirements/FR-SP04.md) (FR-SP04-001…FR-SP04-003)
