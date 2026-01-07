# Actors and Roles (REG001)

## Primary Actors
- **Applicant (Owner/Agent)**
  - Initiates request, provides data, uploads documents, selects vessel name, pays fees.

## System Actors
- **MTCIT System (REG001 Orchestrator)**
  - Orchestrates the end-to-end process, applies rules, routes to integrations, issues certificate.

## External Actors / Integrations
- **ROP PKI**
  - Provides identity authentication decision.
- **Invest Easy (Oman Business Platform)**
  - Provides CR validation and company activity status/eligibility inputs.
- **MAFWR**
  - Provides approval decision for fishing-related requests.
- **Inspection Entity (Classification Society / Consultancy / Internal)**
  - Provides inspection outcomes and reports.
- **Bayan Customs**
  - Provides import documents and clearance status.
- **Payment Gateway (ePayment)**
  - Executes payment and sends payment confirmation status.

## Secondary / Optional Actors
- **MTCIT Registration Department (Backoffice)**
  - Handles escalations, exceptional cases, operational overrides (if allowed by policy).

## Role Notes
- Applicant may be an **individual** or a **company representative**.
- Some approvals are conditional based on **vessel category**, **activity**, and **routing rules** (DMN/config).
