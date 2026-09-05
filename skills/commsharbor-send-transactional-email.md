---
name: commsharbor-send-transactional-email
description: Send one idempotent transactional email through CommsHarbor and confirm its delivery.
api: CommsHarbor API
generated: '2026-09-05'
method: generated
source: openapi/commsharbor-openapi.json + https://commsharbor.com/llms.txt
operations:
  - commsharbor_context
  - commsharbor_domains
  - commsharbor_templates
  - commsharbor_message_send
  - commsharbor_delivery_get
  - commsharbor_delivery_events
---

# Send a transactional email

Base URL `https://commsharbor.com`. Authenticate with `Authorization: Bearer` (session + `X-Organization-Id`, or a scoped API key that determines its own organization). Sending requires the `messages:send` permission.

1. Resolve the active tenant with `commsharbor_context` (`GET /api/context`) — confirm the organization and role your credential acts as.
2. Pick an active sending domain with `commsharbor_domains` (`GET /api/organizations/{organization_id}/domains`). Only a domain that a verify observation marked active can send.
3. Pick a published template with `commsharbor_templates` (`GET /api/organizations/{organization_id}/templates`); published versions are immutable and content-hashed.
4. Send with `commsharbor_message_send` (`POST /api/organizations/{organization_id}/messages`). The `Idempotency-Key` header is **mandatory**: an equivalent replay returns the same delivery with `replayed: true` and never produces a second message; the same key with a different payload answers `409` before any quota or queue work. A `400` means a missing key, unknown template, or an unset required variable; `409` also covers a suppressed recipient; `429` means the organization's quota or the global monthly cap is exhausted — read the `capacity` object (`remaining`, `global_remaining`, `hard_cap`) in every send response.
5. Confirm with `commsharbor_delivery_get` (`GET /api/organizations/{organization_id}/deliveries/{delivery_id}`) and read normalized SES feedback with `commsharbor_delivery_events`. Delivery responses never carry recipient addresses or message content; Open and Click are additive events that never overwrite delivery state.

A send cannot be recalled after dispatch. To stop future sends to an address, create a suppression (`commsharbor_suppression_create`).
