---
name: Classify a video against content policy
description: Authenticate with Unitary, queue a video for policy classification, and collect the result by webhook or polling.
api: openapi/unitary-ai-content-classification-openapi-original.yaml
operations:
  - authenticate_authenticate_post
  - video_policy_classify_classify_policy_video_post
  - video_policy_classification_results_classify_policy_video__job_id__get
generated: '2026-07-21'
method: generated
---

# Classify a video against content policy

1. **Authenticate** — `POST /authenticate` (`authenticate_authenticate_post`) with form field `key={API_KEY}` (content type `application/x-www-form-urlencoded`) against `https://api.unitary.ai/v1`. Cache the returned `api_token` (valid ~24h) and send it as `Authorization: Bearer {api_token}` on every other call. Do NOT re-authenticate per request — `/authenticate` is rate-limited.
2. **Queue the video** — `POST /classify/policy/video` (`video_policy_classify_classify_policy_video_post`) with a `url` form field pointing at the video (pre-signed object-storage URLs supported). Prefer passing `callback_url` so results arrive by webhook; append query params to `callback_url` to round-trip your own correlation data. Limits: max 200MB, first 3 minutes processed, formats .mp4/.mpeg/.webm/.mov/.mkv/.gif/.m4v. Send any user-submitted text as `caption` in the same request for better accuracy.
3. **Collect the result** — preferred: receive the webhook `POST` on your `callback_url`; validate the `X-Hub-Signature-256` header (base64 HMAC-SHA256 of the raw body with your preshared secret) before trusting it. Fallback: poll `GET /classify/policy/video/{job_id}` (`video_policy_classification_results_classify_policy_video__job_id__get`) — processing can take from sub-second up to 24 hours, so poll sparingly.
4. **Handle errors** — `422` returns `HTTPValidationError` (`detail[].loc/msg/type`); `404` on an unknown `job_id`; async failures arrive with `is_error: true` and `result.error` (e.g. "Failed to decode video.").
5. **Act on the decision** — `result.policy_categories[]` entries carry `name`, `description`, and `risk_level` (e.g. ARMS_AMMO / HIGH); `result.metadata` reports width/height/fps/duration/seconds_processed.

For synchronous needs, the add-on `POST /classify/policy/video/now` (`video_classify_policy_now_classify_policy_video_now_post`) returns results immediately.
