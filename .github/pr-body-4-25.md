Closes #236

Implements field mapping between the seller's reusable SEP-9 profile and each anchor's requested fields (issue 4.25).

## Changes

### Core
- New `selectFieldsForAnchor` function in `packages/core/src/kyc/select.ts`:
  - Only fields the anchor requested are included in the send body
  - SEP-9 aliases resolved to anchor's field name (e.g., `family_name` ← `last_name`)
  - Unknown (anchor-specific) fields only come from overrides, never from profile
  - Binary fields excluded (handled separately in 3.13)
  - Optional fields without values are not marked missing

### Ports & Schema
- `KycRecord` now includes `providedFieldStatus` and `sentFields`
- `Sep12CustomerResult` includes `providedFieldStatus` parsed from anchor's `provided_fields`
- New `provided_field_status` and `sent_fields` columns on `seller_kyc` table

### TestAnchorKyc
- `submit()` now uses `selectFieldsForAnchor` to determine what to send
- First submission (no customer record yet) sends all provided fields to discover requirements
- Subsequent submissions only send fields the anchor currently requests
- Records `sentFields` and `providedFieldStatus` on each submission
- Profile repository provides reusable SEP-9 fields from seller's payout fields

### Database
- New columns: `provided_field_status` (JSON), `sent_fields` (JSON string[])
- Additive migrations for existing databases

### Tests
- New `packages/core/test/select-fields.test.ts` with comprehensive alias mapping tests
- New `packages/offramp/test/kyc.test.ts` for submit field mapping
- Updated existing tests for new KycRecord fields