# FR-CC01-001
Title: Enforce ROP PKI authentication gate

Description:
The system shall authenticate the applicant using ROP PKI before allowing access to REG001 subprocesses.

Trigger:
Applicant starts REG001 or resumes a draft requiring authentication.

Inputs:
- Applicant identity attributes (national ID / digital identity attributes)

Processing / Rules:
- Send authentication request to ROP PKI.
- Await PKI response within configured timeout.

Outputs:
- Authentication decision: Authenticated / Rejected

Success Criteria:
- Authenticated applicants can proceed to SP02.

Failure Conditions:
- PKI rejected
- PKI timeout/unavailable

Related Gateways:
- N/A (Cross-cutting CC01)

Related Capabilities:
- CAP-02

---

# FR-CC01-002
Title: Notify and terminate on PKI rejection

Description:
The system shall notify the applicant and terminate the REG001 request when ROP PKI returns a rejection.

Trigger:
ROP PKI returns rejection status.

Inputs:
- PKI rejection status
- Rejection reason (if provided)

Processing / Rules:
- Log the rejection decision for audit.
- Send applicant notification with generic + reason (if available).
- Mark request as terminated.

Outputs:
- Rejection notification
- Terminated request state

Success Criteria:
- Rejected applicants cannot access SP02+.

Failure Conditions:
- Notification failure (must be logged and retried per policy)

Related Gateways:
- N/A (Cross-cutting CC01)

Related Capabilities:
- CAP-02, CAP-23, CAP-26
