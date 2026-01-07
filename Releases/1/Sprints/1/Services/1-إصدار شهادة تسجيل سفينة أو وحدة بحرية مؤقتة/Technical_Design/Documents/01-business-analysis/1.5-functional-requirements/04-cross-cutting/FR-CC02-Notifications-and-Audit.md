# FR-CC02-001
Title: Send applicant notifications on key outcomes

Description:
The system shall send applicant notifications for key outcomes across SP02–SP10 (rejection, approval, payment, certificate issuance, reminders, penalties).

Trigger:
A key outcome occurs (e.g., rejection, approval received, invoice generated, payment status, certificate issued, reminder due, penalty applied).

Inputs:
- Request ID
- Applicant contact details
- Outcome type
- Outcome reason (if applicable)

Processing / Rules:
- Select notification template by outcome type.
- Deliver via configured channels (SMS/Email/in-app).

Outputs:
- Delivered notification records (status per channel)

Success Criteria:
- Applicant receives timely notification for each key outcome.

Failure Conditions:
- Notification provider unavailable

Related Gateways:
- SP02: G03, G04
- SP03: G07
- SP05: G10
- SP06: G12
- SP07: G14
- SP08: G15
- SP10: G16

Related Capabilities:
- CAP-23

---

# FR-CC02-002
Title: Maintain auditable trail of decisions and integrations

Description:
The system shall record an auditable trail for all critical decisions and external integration calls, including inputs, decision outcome, and timestamps.

Trigger:
- A gateway decision is evaluated (G02–G16)
- An external integration call is executed (ROP PKI, Invest Easy, MAFWR, Inspection, Bayan, Payment)

Inputs:
- Request ID
- Decision/integration identifier
- Inputs and outcomes

Processing / Rules:
- Persist audit log entries.

Outputs:
- Audit record set

Success Criteria:
- Audit records are available for compliance review and traceability.

Failure Conditions:
- Audit persistence failure (must be treated as high severity)

Related Gateways:
- G02–G16

Related Capabilities:
- CAP-26

---

# FR-CC02-003
Title: Correlate events and external responses to the correct request

Description:
The system shall correlate inbound responses (Invest Easy, MAFWR, Inspection, Bayan, Payment) to the correct REG001 request using unique correlation identifiers.

Trigger:
External response/callback is received.

Inputs:
- Correlation ID
- Request ID (if provided)
- External reference ID

Processing / Rules:
- Match response to request.
- Reject or quarantine unmatched responses for manual review.

Outputs:
- Updated request state
- Correlation/audit record

Success Criteria:
- No response updates the wrong request.

Failure Conditions:
- Missing/invalid correlation identifiers

Related Gateways:
- G03, G04, G10, G12, G14, G15

Related Capabilities:
- CAP-26
