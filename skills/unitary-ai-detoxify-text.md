---
name: Detect toxic language in text
description: Authenticate with Unitary and run user-generated text through the Detoxify endpoints to score harmful or offensive language.
api: openapi/unitary-ai-content-classification-openapi-original.yaml
operations:
  - authenticate_authenticate_post
  - text_detoxify_classify_detoxify_text__post
  - text_results_detoxify_text__job_id__get
generated: '2026-07-21'
method: generated
---

# Detect toxic language in text

1. **Authenticate** — `POST /authenticate` (`authenticate_authenticate_post`) with form field `key={API_KEY}`; reuse the returned Bearer `api_token` (~24h TTL) as `Authorization: Bearer {api_token}`. The endpoint is rate-limited — never authenticate per request.
2. **Submit the text** — `POST /detoxify/text/` (`text_detoxify_classify_detoxify_text__post`) with the text to score (max 256KB). Include `callback_url` to get the result pushed by webhook, or capture the returned `job_id`.
3. **Collect the result** — webhook preferred (validate `X-Hub-Signature-256`, base64 HMAC-SHA256 with your preshared secret); otherwise poll `GET /detoxify/text/{job_id}` (`text_results_detoxify_text__job_id__get`).
4. **Interpret scores** — Detoxify returns per-category toxicity scores (LanguageToxicityCategory); choose thresholds per Unitary's guidance (https://docs.unitary.ai/api-references/how-to-select-thresholds-for-items-and-characteristics).
5. **Handle errors** — `422` `HTTPValidationError` with `detail[]`; `404` for unknown `job_id`; async failures deliver `is_error: true` with `result.error`.
