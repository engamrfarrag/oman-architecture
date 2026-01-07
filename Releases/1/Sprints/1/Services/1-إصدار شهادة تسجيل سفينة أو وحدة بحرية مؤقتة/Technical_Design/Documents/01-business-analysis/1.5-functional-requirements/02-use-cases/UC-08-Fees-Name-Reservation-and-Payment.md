# UC-08 – Fees, Name Reservation, Invoice and Payment

## Primary Actor
Applicant (Owner/Agent)

## Supporting Actors
Payment Gateway (ePayment)

## Preconditions
- All required validations and conditional approvals completed.

## Main Flow
1. System calculates fees and penalties using DMN/config.
2. Applicant selects the desired vessel name.
3. System reserves the vessel name for a configured payment window.
4. System generates an invoice and initiates payment via the payment gateway.
5. Applicant completes payment.
6. System receives payment result.
7. If successful, system proceeds to certificate issuance.

## Alternate / Failure Flows
- **A1 – Payment failed/cancelled/expired (G15)**: System releases name reservation, notifies applicant, and terminates or allows retry per policy.

## Postconditions
- Payment recorded as successful/failed and name reservation confirmed or released.

## Traceability
- User Story: US-09
- BPMN: SP08 (G15)
- Capabilities: CAP-18, CAP-19, CAP-20, CAP-21
- DMN: reg001-fee-calculation.dmn, reg001-penalty-calculation.dmn
- Functional Requirements: [FR-SP08.md](../03-functional-requirements/FR-SP08.md) (FR-SP08-001…FR-SP08-004)
