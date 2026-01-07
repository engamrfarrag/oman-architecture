```markdown
# FR-INT-03
Title: Integrate with Invest Easy for company activity eligibility validation

Description:
The system shall integrate with Invest Easy (or the authoritative activity source) to validate that the company activity is active/eligible for the selected vessel/unit type.

Trigger:
CR validation successful and activity eligibility validation required (SP02).

Outbound Data (to Invest Easy):
- CR number
- Activity identifier(s)
- Vessel/unit type context (as required)

Inbound Data (from Invest Easy):
- Activity status (Active/Inactive)
- Eligibility indicator (Eligible/Not eligible) (if provided)

Processing / Rules:
- Request activity status/eligibility.
- If activity is not active/eligible, reject the request and notify applicant.
- Persist response and correlation identifiers.

Outputs:
- Activity eligibility outcome used by SP02.

Failure Handling:
- Timeout/unavailable: block progression and report a technical failure per policy.

Related Capabilities:
- CAP-05, CAP-26

Related FRs:
- FR-SP02-003
```
