# Acceptance Criteria (REG001)

This document provides acceptance criteria (high-level) mapped to core functional requirements. Detailed test cases can be derived directly from the FR files.

Note: FR IDs referenced below (e.g., `FR-SP09-001`) are preserved, and are located inside the consolidated subprocess files under `03-functional-requirements/FR-SPxx.md`.

## AC-01 – PKI authentication gate
- **Given** an applicant starts REG001
- **When** the system calls ROP PKI
- **Then** the request proceeds only if PKI returns authenticated
- **And** on rejection the system notifies the applicant and terminates the request
- **Traceability:** FR-CC01-001, FR-CC01-002

## AC-02 – Company CR validation
- **Given** the applicant selects beneficiary type = Company
- **When** the system validates CR through Invest Easy
- **Then** invalid CR results in rejection + notification
- **Traceability:** FR-SP02-002, FR-SP02-004

## AC-03 – Activity eligibility validation
- **Given** the applicant is a Company and CR is valid
- **When** the system validates activity eligibility aligned to vessel type
- **Then** the request proceeds only if activity is active/eligible
- **Traceability:** FR-SP02-003, FR-SP02-004

## AC-04 – Vessel age policy validation
- **Given** the applicant submits build year and material
- **When** the system evaluates vessel age policy using DMN/config
- **Then** policy failure blocks progression to document collection
- **Traceability:** FR-SP03-002

## AC-05 – Late submission penalty trigger
- **Given** the applicant provides construction end date (or acquisition date)
- **When** the system evaluates days late against configured grace period
- **Then** a penalty line item is added when the rule applies
- **Traceability:** FR-SP03-003

## AC-06 – Dynamic document requirements and completeness loop
- **Given** vessel attributes are captured and validated
- **When** the system calculates required documents dynamically
- **Then** the applicant must upload all mandatory documents before proceeding
- **And** missing items are returned as a rework list
- **Traceability:** FR-SP04-001, FR-SP04-003

## AC-07 – MAFWR approval (conditional)
- **Given** a fishing vessel (or fishing activity) is detected
- **When** the system requests MAFWR approval
- **Then** approval is required to proceed and rejection terminates the request
- **Traceability:** FR-SP05-001, FR-SP05-003

## AC-08 – Inspection (conditional)
- **Given** inspection is required by policy
- **When** inspection outcome is received
- **Then** only a “passed” outcome allows the request to continue
- **Traceability:** FR-SP06-001, FR-SP06-003

## AC-09 – Bayan customs clearance (conditional)
- **Given** Bayan is required by policy
- **When** Bayan responds with clearance status
- **Then** only “cleared” allows the request to proceed to payment
- **Traceability:** FR-SP07-001, FR-SP07-003

## AC-10 – Payment and name reservation
- **Given** fees are calculated and the applicant selects a vessel name
- **When** the system reserves the name and generates an invoice
- **Then** payment success proceeds to certificate issuance
- **And** payment failure releases the name reservation and notifies the applicant
- **Traceability:** FR-SP08-002, FR-SP08-004

## AC-11 – Certificate issuance
- **Given** payment is successful
- **When** the system issues the temporary certificate
- **Then** the applicant receives a certificate (PDF + QR) and an issuance notification
- **Traceability:** FR-SP09-001, FR-SP09-002

## AC-12 – Post-issuance reminders and penalties
- **Given** a temporary certificate is issued
- **When** permanent registration is not completed
- **Then** the system sends reminders at the configured cadence
- **And** applies overdue penalties per configured thresholds
- **Traceability:** FR-SP10-001, FR-SP10-003
