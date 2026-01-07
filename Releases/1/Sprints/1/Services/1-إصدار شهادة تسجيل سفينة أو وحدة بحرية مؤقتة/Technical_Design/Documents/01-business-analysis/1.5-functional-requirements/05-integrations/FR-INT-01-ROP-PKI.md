```markdown
# FR-INT-01
Title: Integrate with ROP PKI for applicant authentication

Description:
The system shall integrate with ROP PKI to authenticate the applicant and allow the request to proceed only on successful authentication.

Trigger:
Applicant starts the service and initiates authentication (CC01).

Outbound Data (to ROP PKI):
- Applicant civil ID / identity token (as per PKI contract)

Inbound Data (from ROP PKI):
- Authentication result (Accepted/Rejected)
- Optional reason/details

Processing / Rules:
- Call ROP PKI using the approved integration contract.
- Enforce configured timeout and retry policy.
- Persist the authentication outcome for audit.

Outputs:
- Authenticated session/context OR rejected request path

Failure Handling:
- On PKI rejection: notify applicant and terminate request.
- On timeout/unavailability: fail fast or retry per policy; notify applicant if request cannot continue.

Related Capabilities:
- CAP-02, CAP-26

Related FRs:
- FR-CC01-001, FR-CC01-002
```
