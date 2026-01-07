# SP04 – Dynamic Documents Collection (Functional Requirements)

This file consolidates all SP04 functional requirements in a single document, while preserving the original FR IDs for traceability.

---

## FR-SP04-001
Title: Determine required documents dynamically using DMN/config

Description:
The system shall determine the required documents (mandatory/optional) dynamically based on vessel category, acquisition type, activity, and length.

Trigger:
SP04 starts after SP03 completion.

Inputs:
- Vessel category
- Acquisition type (NEW_BUILD/PURCHASE/FLAG_CHANGE)
- Activity type
- Vessel length

Processing / Rules:
- Evaluate `reg001-documents-required.dmn`.
- Apply dynamic configuration overrides (REG-001-CONF-01).

Outputs:
- Required documents list (document code, name, mandatory flag)

Success Criteria:
- Applicant receives an accurate required documents list.

Failure Conditions:
- Rules evaluation failure (must be logged and handled)

Related Gateways:
- G08

Related Capabilities:
- CAP-11

Rule Source:
- DMN: `Technical_Design/Resources/dmn/reg001/reg001-documents-required.dmn`
- Config: REG-001-CONF-01

---

## FR-SP04-002
Title: Accept document uploads and enforce basic file constraints

Description:
The system shall allow the applicant to upload documents and enforce basic file constraints (format, size, required metadata) per policy.

Trigger:
Applicant uploads a document in SP04.

Inputs:
- Document code/type
- File (e.g., PDF)
- File metadata

Processing / Rules:
- Validate file meets configured constraints (allowed types, size limits).
- Store document and link to request.

Outputs:
- Stored document reference(s)

Success Criteria:
- Uploaded documents are stored and retrievable for review and downstream integrations.

Failure Conditions:
- Invalid file type/size
- Storage failure

Related Gateways:
- Supports G08

Related Capabilities:
- CAP-12

---

## FR-SP04-003
Title: Validate documents completeness and return rework list

Description:
The system shall validate that all mandatory documents are uploaded and, when incomplete, return a rework list of missing items.

Trigger:
Applicant completes an upload attempt or requests validation.

Inputs:
- Required documents list (mandatory flags)
- Uploaded documents set

Processing / Rules:
- Compare uploaded set to mandatory requirements.
- If incomplete, generate missing-items list and loop.

Outputs:
- Completeness decision (complete/incomplete)
- Missing-items rework list

Success Criteria:
- The request proceeds to routing only when documents are complete.

Failure Conditions:
- None (business rework)

Related Gateways:
- G08

Related Capabilities:
- CAP-12

---

## Data Dictionary (SP04)
Aligned with the REG001 Data Dictionary template in `00-overview/Data-Dictionary-Template.md`.

| Data Element (Canonical) | Description | Source / Owner | Data Type | Format / Allowed Values | Validation / Rule Source | Required (Y/N) | Used By (FR IDs) | Example / Notes | Classification |
|---|---|---|---|---|---|---|---|---|---|
| requiredDocuments | Computed required docs list | LS-DOCS | array | list of doc requirements | `reg001-documents-required.dmn` + REG-001-CONF-01 | Y | FR-SP04-001, FR-SP04-003 | Contains mandatory flag | Internal |
| documentCode | Document identifier/code | LS-DOCS | string | config/DMN driven | DMN/config | Y | FR-SP04-001, FR-SP04-002 |  | Internal |
| documentName | Display name | LS-DOCS / Config | string |  | Config | Y | FR-SP04-001 |  | Internal |
| isMandatory | Mandatory indicator | LS-DOCS | boolean | true/false | DMN/config | Y | FR-SP04-001, FR-SP04-003 |  | Internal |
| uploadedDocuments | Uploaded docs set | LS-DOCS / Storage | array | list of stored docs | Storage policy | Y | FR-SP04-002, FR-SP04-003 |  | Internal |
| fileMimeType | Uploaded file type | Client/Storage | string | e.g., application/pdf | Config constraints | Y | FR-SP04-002 |  | Internal |
| fileSizeBytes | Uploaded file size | Client/Storage | number | >=0 | Config constraints | Y | FR-SP04-002 |  | Internal |
| documentStorageRef | Storage reference/id | Document storage | string | URI/id | Storage service | Y | FR-SP04-002 |  | Internal |
| missingDocuments | Rework list of missing required docs | LS-DOCS | array | list of documentCode | Derived by comparison | N | FR-SP04-003 | Returned to applicant UI | Internal |
