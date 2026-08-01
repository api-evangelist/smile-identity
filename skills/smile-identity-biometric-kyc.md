---
name: Run a Biometric KYC verification
description: Verify a user's identity against an ID authority combined with a selfie + liveness check using the Smile ID V3 Biometric KYC endpoint.
api: openapi/smile-identity-v3-openapi.json
operations: [getV3Token, v3BiometricKycEntry, getVerificationStatus]
environments:
  production: https://api.smileidentity.com
  sandbox: https://api.sandbox.smileidentity.com
---

# Run a Biometric KYC verification (Smile ID V3)

Combine a selfie + liveness capture with a KYC identity check against the
relevant ID authority. Results are returned asynchronously via your callback URL.

## Prerequisites
- A Smile ID partner account with Biometric KYC enabled and (for production) production access.
- Server-side API key + numeric partner ID (never expose the API key client-side).
- An allow-listed `callback_url` configured for the environment.
- Explicit user consent captured (granted, granted_at, notice_language, notice_privacy_policy_url).

## Steps

1. **Mint an access token** — `getV3Token` (`POST /v3/token`).
   Send headers `smileid-partner-id` and `smileid-api-key`. The response `token`
   is a JWT valid for 15 minutes. Mint a fresh token per request.

2. **Submit the verification** — `v3BiometricKycEntry` (`POST /v3/biometric_kyc`).
   - Headers: `SmileID-Partner-ID`, `SmileID-Token: <jwt>`. Content-Type `multipart/form-data`.
   - Body: `selfie_image` (JPEG), `liveness_images` (6-8 JPEGs), `consent`, `country`
     (ISO 3166-1 alpha-2), `id_type`, `id_number`, `user_details` (given_names,
     last_name, and at least one of email/phone_number), optional `callback_url`.
   - Keep the total payload under 4.5 MB (else HTTP 413).
   - A successful submit returns **HTTP 202** with `job_id`, `user_id`, `created_at`.

3. **Receive the result** — the authoritative verdict is POSTed to your
   `callback_url`. Verify the outgoing signature + timestamp before trusting it.
   The payload carries `status` (`clear` | `block` | `attention` | `error`),
   `message`, and `job_id`.

4. **(Optional) Poll** — `getVerificationStatus` (`GET /v3/status/{jobId}`) to
   fetch the result on demand, or `replayCallback` (`POST /v3/replay/{job_id}`)
   to resend the webhook.

## Rules
- **Sandbox** only accepts published test identities (match on last_name +
  given_names + email); see `sandbox/smile-identity-sandbox.yml`. Use e.g.
  `Clearwater / Amina Fatou / amina.clearwater@example.com` for a `clear` result.
- **Errors** use a `{ status, message }` envelope; handle 400/401/402/403/413/415/429/500
  and async webhook reasons (`spoof_detected`, `face_verification_failed`, etc.)
  per `errors/smile-identity-problem-types.yml`.
- **429** is rate-limited per ID number — back off exponentially.
- No idempotency-key: do not blindly retry a 202'd job; use `getVerificationStatus`.
