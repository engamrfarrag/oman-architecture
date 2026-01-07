# UC-01 – Authenticate Applicant (ROP PKI)

## Primary Actor
Applicant (Owner/Agent)

## Preconditions
- Applicant selected REG001 and started a new request OR is resuming a draft requiring authentication.

## Main Flow
1. Applicant initiates authentication.
2. System sends authentication request to ROP PKI.
3. System receives PKI response.
4. If authenticated, the system allows the applicant to proceed to applicant profiling (SP02).

## Alternate / Failure Flows
- **A1 – PKI rejected**: System notifies applicant (with reason if provided) and terminates the request.
- **A2 – PKI timeout/unavailable**: System informs applicant and prevents progression until successful authentication.

## Postconditions
- Applicant is authenticated OR the request is terminated/blocked.

## Traceability
- User Story: US-02
- BPMN: CC01
- Capability: CAP-02
- Functional Requirements: [FR-SP02.md](../03-functional-requirements/FR-SP02.md) (handoff to SP02 after authentication)
