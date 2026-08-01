---
name: Run an Enhanced KYC (no-biometrics) verification
description: Verify a user's identity by ID number against an ID authority (no selfie) using the Smile ID V3 Enhanced KYC endpoint.
api: openapi/smile-identity-v3-openapi.json
operations: [getV3Token, submitEnhancedKyc, getVerificationStatus]
environments:
  production: https://api.smileidentity.com
  sandbox: https://api.sandbox.smileidentity.com
---

# Run an Enhanced KYC verification (Smile ID V3)

Look up and validate a user against a government/institutional ID authority by
ID number, returning the authority's biographic record — no biometrics required.

## Steps

1. **Mint a token** — `getV3Token` (`POST /v3/token`) with `smileid-partner-id`
   and `smileid-api-key` headers → JWT (15 min).

2. **Submit** — `submitEnhancedKyc` (`POST /v3/enhanced_kyc`), headers
   `SmileID-Partner-ID` + `SmileID-Token`. Provide `country`, `id_type`,
   `id_number`, `user_details` (given_names, last_name, and at least one of
   email/phone_number), `consent`, and an allow-listed `callback_url`.

3. **Read the result** — synchronous or via `callback_url`. `status` is
   `clear` | `block` | `error`; block reasons include `high_risk`,
   `identifier_not_found`, `age_requirement_not_met`.

4. **Poll if needed** — `getVerificationStatus` (`GET /v3/status/{jobId}`).

## Rules
- Provide at least one of `email` or `phone_number` (else HTTP 400).
- `id_number` must match the country/id_type regex (else HTTP 400) — check
  `getSupportedIdTypes` (`GET /v3/services/supported_id_types`).
- Sandbox test identities: `Clearwater` (clear), `Dangerfield` (high_risk),
  `Ghostwell` (identifier_not_found), `Youngblood` (age_requirement_not_met).
