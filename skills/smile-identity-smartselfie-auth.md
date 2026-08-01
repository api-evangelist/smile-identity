---
name: Enroll and authenticate a user with SmartSelfie
description: Register a user's biometrics once, then authenticate returning users against that enrollment using the Smile ID V3 SmartSelfie endpoints.
api: openapi/smile-identity-v3-openapi.json
operations: [getV3Token, v3BiometricEnrollment, v3BiometricAuthentication]
environments:
  production: https://api.smileidentity.com
  sandbox: https://api.sandbox.smileidentity.com
---

# Enroll and authenticate with SmartSelfie (Smile ID V3)

Establish a biometric identity for a user (registration), then verify returning
users with a fresh selfie + liveness check (authentication).

## Steps

1. **Mint a token** — `getV3Token` (`POST /v3/token`) → JWT (15 min).

2. **Enroll (once)** — `v3BiometricEnrollment` (`POST /v3/registration`),
   `multipart/form-data`, headers `SmileID-Partner-ID` + `SmileID-Token`.
   Provide `selfie_image`, `liveness_images` (6-8), `consent`, `user_details`,
   optional `User-ID` header (else a TypeID `user_id` is generated). Returns 202;
   the enrollment webhook message uses "Enrollment" wording.

3. **Authenticate (each return visit)** — `v3BiometricAuthentication`
   (`POST /v3/authentication`) with a new `selfie_image` + `liveness_images` and
   the enrolled `user_id`. Result `status`: `clear` | `block` | `error`.

4. **(Optional) Compare** — `v3SmartSelfieCompare` (`POST /v3/compare`) to compare
   a selfie to a supplied reference image without a stored enrollment.

## Rules
- Persist the `user_id` returned at enrollment — it is the key you authenticate against.
- Block reasons: `high_risk`, `face_verification_failed`, `spoof_detected`,
  `account_locked` (authentication only). Error reasons: `image_unavailable_or_invalid`,
  `internal_error`. See `errors/smile-identity-problem-types.yml`.
- Sandbox: reuse identities `Clearwater` (clear), `Twinley` (face_verification_failed),
  `Masquero` (spoof_detected), `Lockhart` (account_locked). See `sandbox/`.
