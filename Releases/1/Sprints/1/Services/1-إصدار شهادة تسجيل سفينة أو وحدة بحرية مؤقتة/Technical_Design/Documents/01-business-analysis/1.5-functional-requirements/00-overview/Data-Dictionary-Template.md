# REG001 – Data Dictionary Template (Aligned to FRs)

Use this template inside each SP Functional Requirements file (e.g., `03-functional-requirements/FR-SP02.md`) to define a consistent, implementation-ready data dictionary.

## How to use
- Keep names **canonical** (camelCase) and stable across SPs.
- For each FR, list the data elements it reads/writes and any validation rules.
- Reference DMN/config/integration contracts when rules are not hard-coded.

## Data Classification (recommended)
- **Public**: non-sensitive
- **Internal**: internal operational data
- **Confidential/PII**: personal/identity data (e.g., Civil ID)

## Template
| Data Element (Canonical) | Description | Source / Owner | Data Type | Format / Allowed Values | Validation / Rule Source | Required (Y/N) | Used By (FR IDs) | Example / Notes | Classification |
|---|---|---|---|---|---|---|---|---|---|
|  |  |  |  |  |  |  |  |  |  |
