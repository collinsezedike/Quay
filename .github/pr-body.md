Closes #237

Implements per-anchor consent tracking for KYC field disclosure as required by Nigeria Data Protection Act 2023.

## Changes

### Database
- New `kyc_consents` table tracking seller/anchor consent with fields array, granted/revoked timestamps, and metadata

### Core Ports
- Added `KycConsent` interface and `KycConsentRepository` port
- Added `ConsentRequiredError` for enforcement

### API Routes
- `GET /seller/kyc/consent` — list all consents (session auth only)
- `POST /seller/kyc/consent` — grant consent for specific fields (session auth only, validates against anchor's current requirements)
- `DELETE /seller/kyc/consent/:anchorDomain` — revoke consent (session auth only)
- `PUT /seller/kyc` — now enforces consent before sending any fields to anchor; returns `403 consent_required` with `{ anchorDomain, fields }` when fields lack consent

### Web Dashboard
- `KycPanel.tsx` shows consent dialog listing exact field labels before submit
- Consent history displayed when KYC is ACCEPTED
- New `consent_required` error handling

### Tests
- New `kyc-consent.test.ts` with comprehensive coverage:
  - Consent CRUD operations
  - API key rejection on consent endpoints
  - Consent validation (extra fields, missing required fields)
  - Submit enforcement (no consent, partial consent, full consent)

### Documentation
- Updated `docs/API.md` with new endpoints and `consent_required` error