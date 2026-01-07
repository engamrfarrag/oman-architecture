```markdown
# FR-INT-07
Title: Integrate with ePayment gateway for invoice payment confirmation

Description:
The system shall integrate with the ePayment gateway to initiate payment for the issued invoice and receive the payment confirmation outcome.

Trigger:
Invoice generated and applicant proceeds to payment (SP08).

Outbound Data (to ePayment):
- Invoice reference
- Amount
- Service code/transaction metadata

Inbound Data (from ePayment):
- Payment status (Success/Failed/Cancelled/Expired)
- Receipt reference (on success)

Processing / Rules:
- Initiate payment.
- Correlate payment callback to the correct request/invoice.
- Persist payment outcome and receipt reference.
- Route process based on outcome (success → SP09; failure → release name reservation + notify).

Outputs:
- Payment outcome consumed by SP08/SP09

Failure Handling:
- Gateway timeout/unavailable: fail fast or retry per policy; notify applicant where applicable.

Related Capabilities:
- CAP-21, CAP-23, CAP-26

Related FRs:
- FR-SP08-004
```
