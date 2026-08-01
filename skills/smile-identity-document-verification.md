---
name: Run a Document Verification
description: Classify and verify an identity document (with optional selfie match) using the Smile ID V3 Document Verification endpoint.
api: openapi/smile-identity-v3-openapi.json
operations: [getV3Token, v3DocumentVerificationEntry, getVerificationStatus]
environments:
  production: https://api.smileidentity.com
  sandbox: https://api.sandbox.smileidentity.com
---

# Run a Document Verification (Smile ID V3)

Verify an uploaded identity document — classify the document type, extract
fields, and (optionally) match it to a selfie. Results are asynchronous.

## Steps

1. **Mint a token** — `getV3Token` (`POST /v3/token`) with `smileid-partner-id`
   and `smileid-api-key` headers → JWT (15 min).

2. **Submit** — `v3DocumentVerificationEntry` (`POST /v3/document_verification`),
   `multipart/form-data`, headers `SmileID-Partner-ID` + `SmileID-Token`.
   Provide the document image(s), `country`, `id_type`, `user_details`, `consent`,
   and an allow-listed `callback_url`. Returns **HTTP 202** with a `job_id`.

3. **Consume the callback** — result POSTed to `callback_url` with `status`
   (`clear` | `block` | `attention` | `error`). Attention reasons include
   `document_copy_detected` and `document_expired` (still `clear`-ish success);
   block reasons include `document_check_failed`, `unsupported_document`.

4. **Poll if needed** — `getVerificationStatus` (`GET /v3/status/{jobId}`).

## Rules
- Sandbox test identities for this product include `Forgeman` (document_check_failed),
  `Oddpaper` (unsupported_document), `Puzzleton` (document_unclassifiable error),
  `Xeroxley` (document_copy_detected). See `sandbox/smile-identity-sandbox.yml`.
- Handle `document_unclassifiable` and `image_unavailable_or_invalid` by
  re-capturing a clearer image. Errors follow `errors/smile-identity-problem-types.yml`.
