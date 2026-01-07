# SP03 – Vessel/Unit Data, Policy Validations & Penalty Trigger (Functional Requirements)

This file consolidates all SP03 functional requirements in a single document, while preserving the original FR IDs for traceability.

---

## FR-SP03-001
Title: Capture vessel/unit attributes required for routing and billing

Description:
The system shall capture and persist vessel/unit attributes required for policy validation, routing decisions, and fee calculation.

Trigger:
Applicant proceeds to SP03.

Inputs:
- Vessel/unit category/type
- Build year
- Build material (if applicable)
- Vessel length (m)
- Gross tonnage (GT)
- Intended activity
- Acquisition type (NEW_BUILD / PURCHASE / FLAG_CHANGE)
- Construction/acquisition dates (as applicable)

Processing / Rules:
- Validate mandatory fields are present.
- Persist attributes for downstream SP04–SP10.

Outputs:
- Stored vessel/unit record linked to request

Success Criteria:
- Downstream SP decisions can evaluate with complete data.

Failure Conditions:
- Missing mandatory attributes

Related Gateways:
- N/A (supports G05–G07 and later G11/G13 and SP08)

Related Capabilities:
- CAP-06, CAP-09

---

## FR-SP03-002
Title: Validate vessel age policy using DMN/config

Description:
The system shall validate vessel age against configurable policy rules and block progression when the vessel age is not acceptable.

Trigger:
Applicant submits build year (and material/category where applicable).

Inputs:
- Build year
- Current year
- Build material (optional)
- Vessel category

Processing / Rules:
- Evaluate `reg001-vessel-age-validation.dmn`.
- If invalid, display a clear message and prevent progression to SP04.
- Where policy permits, allow applicant to re-enter/correct inputs.

Outputs:
- Age validation result (valid/invalid)
- Validation message

Success Criteria:
- Valid age allows continuation to acquisition path inputs and SP04.

Failure Conditions:
- Age invalid per policy (business rejection/block)

Related Gateways:
- G05

Related Capabilities:
- CAP-07

Rule Source:
- DMN: `Technical_Design/Resources/dmn/reg001/reg001-vessel-age-validation.dmn`

---

## FR-SP03-003
Title: Evaluate late submission penalty trigger and add penalty line item

Description:
The system shall evaluate whether a late submission penalty applies based on configured timing rules and add a penalty line item when applicable.

Trigger:
Applicant submits construction end date / acquisition date.

Inputs:
- Construction end date (or relevant acquisition completion date)
- Submission date/time
- Grace period configuration (e.g., 30 days)
- Base fee (if required by DMN)
- Vessel category (if required by DMN)

Processing / Rules:
- Calculate days late.
- Evaluate `reg001-penalty-calculation.dmn`.
- If penalty applies, add penalty line item to the request fee basket.

Outputs:
- Penalty applies (Y/N)
- Penalty amount and reason
- Updated fee basket

Success Criteria:
- Penalty is included in invoice when applicable.

Failure Conditions:
- Missing required dates (cannot evaluate)

Related Gateways:
- G06

Related Capabilities:
- CAP-10

Rule Source:
- DMN: `Technical_Design/Resources/dmn/reg001/reg001-penalty-calculation.dmn`
- Config: REG-001-CONF-02 (timing/thresholds)

---

## FR-SP03-004
Title: Reject prohibited/banned vessels based on configured policy list

Description:
The system shall check the vessel/unit against the configured prohibited/banned vessel list and reject the request when a match is found.

Trigger:
Vessel/unit identifiers and core attributes are captured.

Inputs:
- Vessel identifiers/attributes needed to match banned list

Processing / Rules:
- Evaluate banned/prohibited list policy (config).
- If prohibited, reject request and notify applicant.

Outputs:
- Rejected request (prohibited)
- Rejection notification

Success Criteria:
- Prohibited vessels cannot proceed to SP04.

Failure Conditions:
- None (business rejection)

Related Gateways:
- G07

Related Capabilities:
- CAP-08, CAP-23

Rule Source:
- Config: REG-001-CONF-04 (prohibited/banned vessels)

---

## Data Dictionary (SP03)
Aligned with the REG001 Data Dictionary template in `00-overview/Data-Dictionary-Template.md`.

| Data Element (Canonical) | Description | Source / Owner | Data Type | Format / Allowed Values | Validation / Rule Source | Required (Y/N) | Used By (FR IDs) | Example / Notes | Classification |
|---|---|---|---|---|---|---|---|---|---|
| vesselCategory | Vessel/unit category/type used for routing | Applicant UI / LS-VESSEL | string/enum | config-driven | Config (REG-001-SET-04) | Y | FR-SP03-001 |  | Internal |
| buildYear | Year of build | Applicant UI | integer | YYYY | DMN age validation | Y | FR-SP03-001, FR-SP03-002 | 2015 | Internal |
| buildMaterial | Hull/material indicator (if applicable) | Applicant UI | string/enum | policy-defined | DMN age validation (optional input) | N | FR-SP03-002 |  | Internal |
| vesselLengthM | Length in meters | Applicant UI | number | >=0 | DMN/inspection rules use this | Y | FR-SP03-001 | 23.5 | Internal |
| grossTonnage | Gross tonnage | Applicant UI | number | >=0 | Fee calculation DMN input | N | FR-SP03-001 |  | Internal |
| acquisitionType | How the vessel is acquired | Applicant UI / LS-VESSEL | enum | NEW_BUILD, PURCHASE, FLAG_CHANGE | UI validation | Y | FR-SP03-001 | Drives customs/inspection logic | Internal |
| constructionEndDate | Construction completion date | Applicant UI | date | ISO-8601 | Penalty timing config + DMN | Conditional | FR-SP03-003 | Required for NEW_BUILD | Internal |
| submissionDateTime | When the request is submitted/advanced | LS-REG-ORCH | datetime | ISO-8601 | Used for penalty calculation | Y | FR-SP03-003 |  | Internal |
| penaltyAmount | Late penalty amount | LS-PENALTIES | number | >=0 | `reg001-penalty-calculation.dmn` | N | FR-SP03-003 | Stored as line item | Internal |
| bannedVesselMatch | Flag indicating banned/prohibited match | LS-POLICY | boolean | true/false | Config list (REG-001-CONF-04) | Y | FR-SP03-004 | If true → reject | Internal |
