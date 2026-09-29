Closes #233

Implements the `offramp.transfer_required` webhook so API-key integrators can drive the send leg of a cash-out.

## Changes

### Database
- Added `transfer_notified_at` column to `offramp_jobs` table to track when transfer instructions were first surfaced

### Core Ports
- Added `transferNotifiedAt` to `StoredOffRampJob` interface
- Updated `OffRampStateRepository.updateJob` to accept `transferNotifiedAt` in patch

### API Logic
- In `triggerCashOut()`: emit `offramp.transfer_required` when `initiation.kind === "transfer"` (immediate instructions)
- In `pollCashOuts()`: emit `offramp.transfer_required` on first poll when `status()` returns a `transfer` (late-arriving instructions)
- Both paths mark the job with `transferNotifiedAt` to ensure exactly-once delivery per job

### Off-ramp Adapters
- Updated `MockAnchorOffRamp` and `TestAnchorOffRamp` to include `transferNotifiedAt` in saved jobs

### Tests
- Added comprehensive tests covering:
  - Webhook emitted on `initiate` with `kind: "transfer"`
  - No webhook emitted on `initiate` with `kind: "fields"`
  - Webhook emitted on first poll when `status()` returns `transfer`
  - No re-emission on subsequent polls
  - No webhook emitted when job status is `settled`

### Documentation
- Added `offramp.transfer_required` to events table in `docs/API.md`
- Added example payload with `transfer` object containing SEP-6 deposit instructions
- Noted sensitivity considerations for integrators