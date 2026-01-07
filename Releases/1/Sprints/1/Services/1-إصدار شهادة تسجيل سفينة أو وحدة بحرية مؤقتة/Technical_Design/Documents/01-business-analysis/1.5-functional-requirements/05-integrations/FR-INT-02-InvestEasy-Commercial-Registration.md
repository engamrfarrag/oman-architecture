```markdown
# FR-INT-02
Title: Integrate with Invest Easy for Commercial Registration (CR) validation

Description:
When the applicant is a company, the system shall integrate with Invest Easy to validate the company CR and return a valid/invalid result used to continue or reject the request.

Trigger:
Beneficiary type = Company (SP02).

Outbound Data (to Invest Easy):
- CR number
- Company identifiers (as required)

Inbound Data (from Invest Easy):
- CR status (Valid/Invalid)
- Optional validation details

Processing / Rules:
- Perform CR validation through Invest Easy.
- Persist the response and correlation identifiers.

Outputs:
- CR validation result used by SP02.

Failure Handling:
- Timeout/unavailable: block progression and report a technical failure per policy.

Related Capabilities:
- CAP-04, CAP-26

Related FRs:
- FR-SP02-002
```
