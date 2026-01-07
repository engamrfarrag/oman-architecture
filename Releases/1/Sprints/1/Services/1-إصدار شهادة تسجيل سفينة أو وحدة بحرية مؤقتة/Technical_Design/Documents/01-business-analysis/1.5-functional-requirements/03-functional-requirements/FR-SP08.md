# SP08 – Fees, Name Reservation, Invoice & Payment (Functional Requirements)

This file consolidates all SP08 functional requirements in a single document, while preserving the original FR IDs for traceability.

---

## FR-SP08-001
Title: Calculate fees using DMN/config and include penalties

Description:
The system shall calculate applicable fees for temporary registration using DMN/config and include any penalty line items.

Trigger:
SP08 starts after all required validations/approvals are complete.

Inputs:
- Vessel category
- Vessel length
- Gross tonnage
- Activity type
- Registration type = TEMPORARY
- Penalty line items (if any)

Processing / Rules:
- Evaluate `reg001-fee-calculation.dmn`.
- Aggregate base fee, surcharges, and penalties.

Outputs:
- Fee breakdown and total amount

Success Criteria:
- Fee summary is presented to applicant and used to generate an invoice.

Failure Conditions:
- Rules evaluation failure

Related Gateways:
- N/A (supports SP08 steps and payment)

Related Capabilities:
- CAP-18, CAP-20

Rule Source:
- DMN: `Technical_Design/Resources/dmn/reg001/reg001-fee-calculation.dmn`

---

## FR-SP08-002
Title: Reserve vessel name during payment window and release on failure

Description:
The system shall reserve the applicant-selected vessel name for a configured payment window and release the reservation if payment fails/cancels/expires.

Trigger:
Applicant selects a vessel name prior to payment.

Inputs:
- Requested vessel name
- Request ID
- Name reservation hold duration (config)

Processing / Rules:
- Create name reservation hold.
- Confirm reservation on successful payment.
- Release reservation on payment failure/expiry.

Outputs:
- Name reservation status (Held/Confirmed/Released)

Success Criteria:
- The name is protected from conflicting requests during the payment window.

Failure Conditions:
- Name not available
- Reservation service failure

Related Gateways:
- G15 (release on failure)

Related Capabilities:
- CAP-19

---

## FR-SP08-003
Title: Generate invoice prior to initiating payment

Description:
The system shall generate an invoice containing the calculated fees and penalties before initiating payment.

Trigger:
Fee calculation completed.

Inputs:
- Fee breakdown and total amount
- Request ID
- Applicant identifiers

Processing / Rules:
- Create invoice with unique reference.
- Send invoice reference to payment gateway.

Outputs:
- Invoice reference
- Invoice status (Issued)

Success Criteria:
- Invoice exists and can be reconciled with payment confirmation.

Failure Conditions:
- Invoice creation failure

Related Gateways:
- N/A

Related Capabilities:
- CAP-20

---

## FR-SP08-004
Title: Process payment and handle success/failure outcomes

Description:
The system shall initiate payment via the payment gateway, receive payment confirmation, and route the request based on payment outcome.

Trigger:
Invoice issued and applicant proceeds to payment.

Inputs:
- Invoice reference
- Amount
- Payment callback status
- Receipt reference (if success)

Processing / Rules:
- Initiate payment with the payment gateway.
- Receive callback/confirmation.
- If success, persist receipt and proceed to SP09.
- If failed/cancelled/expired, release name reservation, notify applicant, and terminate or allow retry per policy.

Outputs:
- Payment status (Success/Failed/Cancelled/Expired)
- Receipt reference (on success)

Success Criteria:
- Successful payment unlocks certificate issuance.

Failure Conditions:
- Gateway timeout/unavailable

Related Gateways:
- G15

Related Capabilities:
- CAP-21, CAP-23

---

## Data Dictionary (SP08)
Aligned with the REG001 Data Dictionary template in `00-overview/Data-Dictionary-Template.md`.

| Data Element (Canonical) | Description | Source / Owner | Data Type | Format / Allowed Values | Validation / Rule Source | Required (Y/N) | Used By (FR IDs) | Example / Notes | Classification |
|---|---|---|---|---|---|---|---|---|---|
| feeBreakdown | Itemized fees/penalties | LS-FEES | array | line items | `reg001-fee-calculation.dmn` | Y | FR-SP08-001, FR-SP08-003 | Used for invoice | Internal |
| totalAmount | Total payable amount | LS-FEES | number | >=0 | Sum of breakdown | Y | FR-SP08-001, FR-SP08-003, FR-SP08-004 |  | Internal |
| vesselName | Requested vessel name | Applicant UI / LS-NAME | string | policy-defined | Uniqueness + availability | Y | FR-SP08-002 | Held during payment | Internal |
| nameReservationStatus | Reservation state | LS-NAME | enum | HELD, CONFIRMED, RELEASED | Reservation service policy | Y | FR-SP08-002, FR-SP08-004 | Release on failure | Internal |
| invoiceReference | Invoice identifier | LS-FEES/LS-PAYMENTS | string |  | Invoice service | Y | FR-SP08-003, FR-SP08-004 | Sent to gateway | Internal |
| paymentStatus | Payment outcome | Payment gateway / LS-PAYMENTS | enum | SUCCESS, FAILED, CANCELLED, EXPIRED | Gateway callback | Y | FR-SP08-004 | Drives SP09 | Internal |
| receiptReference | Payment receipt reference | Payment gateway | string |  | Gateway response | N | FR-SP08-004 | Stored on success | Internal |
| paymentCallbackPayload | Callback data for reconciliation | Payment gateway | object | per contract | Signature/validation policy | Y | FR-SP08-004 | Must be auditable | Internal |
