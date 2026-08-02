---
name: Moderate content with a Virtual Trust and Safety Agent
description: Authenticate with Unitary and submit content to the Virtual Moderator endpoint for blended AI + human moderation decisions.
api: openapi/unitary-ai-content-classification-openapi-original.yaml
operations:
  - authenticate_authenticate_post
  - moderate_content_moderate_post
generated: '2026-07-21'
method: generated
---

# Moderate content with a Virtual Trust and Safety Agent

1. **Authenticate** — `POST /authenticate` (`authenticate_authenticate_post`) with form field `key={API_KEY}`; reuse the Bearer `api_token` (~24h TTL) via `Authorization: Bearer {api_token}`. `/authenticate` is rate-limited.
2. **Submit the case** — `POST /moderate` (`moderate_content_moderate_post`) with a VirtualModeratorRequest: main content as text (TextMainContent) or media (MediaMainContent), plus optional attachments (TextAttachment, LinkAttachment, MetadataAttachment, UserReportAttachment) for context. Payload limits: 200KiB total, 100KiB per attachment. Provide a `callback_url` for result delivery and an `external_id` to correlate with your own systems.
3. **Receive the decision** — the queued response (VirtualModeratorQueuedResponse) acknowledges the case; the moderation result is delivered to your `callback_url` (validate `X-Hub-Signature-256`, base64 HMAC-SHA256 with your preshared secret). The result carries `decision_type`, `escalate` + `escalate_reasons`, `policy_categories[]` ({name, risk_level}), `moderation_job_id`, `classification_job_id`, and your `external_id`.
4. **Handle errors** — `422` `HTTPValidationError` with `detail[]`; error results carry `is_error: true` with `error_message`.
5. **Route escalations** — when `escalate` is true, hand the case to your human review queue with `escalate_reasons` attached; Virtual Moderator blends AI decisions with human review for the ambiguous tail.
