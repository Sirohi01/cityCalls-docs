# 14 — Integration Architecture

Every third-party integration is optional, independently enableable/disableable via `config.integrations.{name}.enabled`, and sits behind an internal interface so the rest of the codebase never imports a vendor SDK directly. This document defines each integration's interface contract and, critically, its **disabled/failure behavior**.

## 1. General Pattern

```
Domain module → Internal interface (e.g. FileStorage, MessageSender, AIProvider) → Adapter (Cloudinary | Local, AiSensy | NoOp, Gemini | OpenAI | NoOp) → External API
```

Adapter selection reads `config.integrations.{name}.enabled` and provider choice at call time (not baked in at boot), so an admin flipping a toggle takes effect without a redeploy. Every adapter call is wrapped so a thrown/rejected error is caught, logged, and surfaces as a `FAILED` state on the relevant record (notification, file, AI request) — never an unhandled exception that aborts the calling business operation.

## 2. Cloudinary / File Storage

**Interface**: `FileStorage.getSignedUploadParams(category, entityType, entityId)`, `FileStorage.confirmUpload(fileId, providerRef)`, `FileStorage.delete(fileId)`.

**When enabled**: signed-upload flow per [10-api-standards.md](10-api-standards.md) §9 — API issues short-lived signed params, client uploads directly to Cloudinary, confirms back to API which stores the returned secure URL + metadata.

**When disabled (fallback)**: same interface, adapter swaps to local/S3-compatible bucket storage — API issues a pre-signed PUT URL (S3-style) instead of Cloudinary signature, same confirm-back flow. Neither client code nor the calling module changes; only the adapter differs.

**Restrictions** (enforced server-side regardless of adapter): file-type allowlist per category (images: jpg/png/webp; video: mp4; documents: pdf/jpg/png), max size per category (configurable, e.g. 10MB images, 100MB video), folder naming `{env}/{entityType}/{entityId}/{category}/{filename}`.

## 3. SMTP / Email

**Interface**: `EmailSender.send(to, templateId, variables, attachments?)`.

**When enabled**: routes through the configured SMTP account (org default or branch-level override if set), records an `email_logs` entry with open/click tracking pixel/links where supported.

**When disabled**: `send()` still records the intended email as a `notifications` document with `status: SKIPPED_INTEGRATION_DISABLED` (not `FAILED` — this is an intentional, not erroneous, non-send) so reporting can distinguish "we didn't try" from "we tried and failed." The triggering business action (e.g. invoice generation) completes normally either way.

**Bounce/unsubscribe handling**: bounce webhook (provider-dependent) marks the customer's `consent.email` toward a soft-bounce counter; repeated bounces flip consent to a `BOUNCED` sub-state (extends the `ConsentState` enum for email specifically) surfaced to marketing before further sends.

## 4. AiSensy / WhatsApp

**Interface**: `WhatsAppSender.send(to, templateName, variables)`, `WhatsAppSender.syncTemplates()`.

**When enabled**: templates are synced from AiSensy's approved template list into `notification_templates` (channel: `WHATSAPP`) via a scheduled job; sending uses only approved, synced templates — free-form messages outside the 24-hour session window are not attempted since WhatsApp Business API disallows them.

**When disabled**: same `SKIPPED_INTEGRATION_DISABLED` behavior as email.

**Consent enforcement**: `WhatsAppSender.send()` checks `customer.consent.whatsapp === GRANTED` before attempting any marketing-category send; transactional-category sends (order/service updates) are permitted under a narrower default-consent assumption per WhatsApp Business policy, but the exact transactional-vs-marketing template categorization is confirmed with AiSensy at integration time — flagged in [21-open-decisions-and-clarifications.md](21-open-decisions-and-clarifications.md).

## 5. Gemini / OpenAI

**Interface**: `AIProvider.summarize(text)`, `AIProvider.classify(text, categories)`, `AIProvider.generateContent(prompt, context)`.

**When enabled**: provider (Gemini or OpenAI, admin-selected) and model are read from config; every call is logged (`ai_requests` collection: input reference, output, tokenUsage, cost estimate, requestedBy, timestamp) for cost/audit visibility; per-feature enable flags gate which triggers may call AI at all (e.g. "call-note summarization" can be on while "auto-reply suggestions" is off).

**When disabled**: features that depend on AI degrade to their non-AI default (e.g. complaint classification falls back to manual category selection by the call executive) — no AI-dependent feature is ever the *only* way to complete a workflow step.

**Irreversibility rule**: AI output is always presented for human review/approval before it changes system state (e.g. AI-suggested lead score is a suggestion field the sales exec can override, not an auto-write that silently changes lead priority).

**Usage limits**: token/cost caps configurable globally and per-role; exceeding a cap disables further AI calls for that scope until reset, surfaced as a clear "AI limit reached" state, not a silent failure.

## 6. Payment Gateway (interface-ready, provider TBD)

**Interface**: `PaymentGateway.createOrder(amount, referenceId)`, `PaymentGateway.verifyPayment(orderId, signature)`, `PaymentGateway.refund(paymentId, amount)`.

No provider is wired for v1 (see [21-open-decisions-and-clarifications.md](21-open-decisions-and-clarifications.md)); cash/UPI-manual/bank-transfer/cheque/credit payment methods (manually recorded, not gateway-verified) are fully functional without this integration, so payment recording is never blocked on gateway selection.

## 7. SMS (interface-ready, provider TBD)

Same `MessageSender`-family interface as WhatsApp, `channel: SMS`. No provider selected yet; the `notifications` pipeline already treats SMS as a first-class channel value so wiring a provider later is adapter work only, not a schema change.

## 8. Integration Settings UI (Admin)

`GET/PATCH /api/v1/config/integrations` exposes enabled flags, provider selection (where applicable), and masked API-key fields (never returned in full once saved — write-only, matching standard secret-handling practice) for each integration in this document. Full settings-screen field spec in [rohit/07-form-field-specifications.md](rohit/07-form-field-specifications.md).
