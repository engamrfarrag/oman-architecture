## Optimization: Minimal changes to enable clean FR generation

Below are **surgical improvements** — not new documents.

---

## Optimization #1 — Add a “Functional Requirement Anchor” to each SP

### What to add (very small change)

At the top of **each SPxx document**, add this section:

```md
## Functional Responsibility
This subprocess is responsible for:
- <Responsibility 1>
- <Responsibility 2>
- <Responsibility 3>
```

### Example (SP03)

```md
## Functional Responsibility
This subprocess is responsible for:
- Capturing vessel/unit master data
- Validating vessel age against configurable policy
- Detecting penalty triggers based on timing rules
- Blocking prohibited or banned vessels
```

**Why this matters**

* Each bullet becomes **1–2 Functional Requirements**
* Zero ambiguity during FR writing
* Perfect alignment with CAP mapping

---

## Optimization #2 — Convert Gateways into Explicit Decision Rules

You already have gateways like:

* G05 – Vessel age policy passed?
* G11 – Inspection required?

### What to add (small enhancement)

Under each Gateway section, add:

```md
### Decision Rule (Business)
- Inputs:
- Rule Source (DMN / Config):
- Outcomes:
```

### Example (G05)

```md
### Decision Rule (Business)
- Inputs: vessel type, build year, material
- Rule Source: reg001-vessel-age-validation.dmn
- Outcomes:
  - Pass → continue
  - Fail → block issuance; allow re-entry if policy permits
```

**Why this matters**

* FRs become **unambiguous**
* QA can derive test cases
* DMN ownership is explicit

---





