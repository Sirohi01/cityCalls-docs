# Manish 11 — Third-Party Integration Plan

Implementation sequencing for the integrations abstracted in [14-integration-architecture.md](../14-integration-architecture.md). All are built behind their interface from day one (even before a real credential exists) using a `NoOp`/local adapter, so core workflow development never blocks on an integration account being provisioned.

## 1. Build Order

1. **File Storage** (Cloudinary + local fallback) — needed early since image capture appears in Phase 2-5 flows. Build the fallback adapter first (no external dependency), add Cloudinary adapter once credentials are available.
2. **SMTP/Email** — needed by Phase 1-2 for password reset; build with a `NoOp`/console-log adapter for early local dev, real SMTP adapter before Phase 6 (invoice emailing).
3. **AiSensy/WhatsApp** — needed from Phase 4 onward (status notifications); build interface + `NoOp` early, real adapter when Phase 8 begins.
4. **Gemini/OpenAI** — Phase 9, no earlier dependency.
5. **Payment Gateway** — interface only, no adapter until a provider is chosen (§6 of [14-integration-architecture.md](../14-integration-architecture.md)).
6. **SMS** — interface only, same as Payment Gateway.

## 2. Credential Provisioning Checklist (per integration, before enabling in any real environment)

- [ ] Account created, API key/credentials obtained.
- [ ] Sandbox/test mode verified working end-to-end before production credentials are used.
- [ ] Credential stored per §3 of [17-security-and-audit.md](../17-security-and-audit.md) (env var / encrypted config, never committed).
- [ ] Enabled flag flipped on only after the adapter is tested against sandbox.

## 3. Cloudinary Setup Specifics

Folder structure `{env}/{entityType}/{entityId}/{category}/`, signed-upload preset configured to enforce type/size limits server-side (not just client-side hints), webhook (if used) for upload-complete confirmation as an alternative to client confirm-back.

## 4. AiSensy Setup Specifics

Template registration/approval happens in the AiSensy dashboard (external, manual, business-side action) before `syncTemplates()` can pull them in — Manish's job is the sync + send integration, not template content creation (that's a Marketing Executive's admin-panel task once templates exist).

## 5. Gemini/OpenAI Setup Specifics

Both providers wired behind the same `AIProvider` interface (§5 of [14-integration-architecture.md](../14-integration-architecture.md)) so switching providers is a config change; cost/token tracking (`ai_requests` collection) built alongside the first real feature, not retrofitted later.

## 6. Testing Strategy for Integrations

Per §2 of [19-testing-strategy.md](../19-testing-strategy.md): every integration's disabled-path is unit tested; enabled-path integration tests run against sandbox credentials in CI where the provider supports it, or are manually verified per release per [coordination/11-release-checklist.md](../coordination/11-release-checklist.md) where sandbox testing isn't practical (e.g. real SMS costs money per send).
