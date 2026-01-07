# UC-09 – Certificate Issuance and Notifications

## Primary Actor
MTCIT System (REG001 Orchestrator)

## Preconditions
- Payment successful.

## Main Flow
1. System confirms eligibility for issuance.
2. System generates temporary certificate (PDF + QR).
3. System persists certificate metadata and links it to the request.
4. System notifies applicant that certificate is issued and provides access.

## Alternate / Failure Flows
- **A1 – Issuance eligibility failure**: System stops issuance and notifies operations/applicant as applicable.

## Postconditions
- Temporary certificate issued and accessible.

## Traceability
- User Story: US-09
- BPMN: SP09
- Capability: CAP-22, CAP-23
- Functional Requirements: [FR-SP09.md](../03-functional-requirements/FR-SP09.md) (FR-SP09-001…FR-SP09-002)
