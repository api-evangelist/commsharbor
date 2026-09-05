---
name: commsharbor-launch-marketing-campaign
description: Launch a permission-based marketing campaign exactly once, from consent to reconciliation, with pause/cancel reversal.
api: CommsHarbor API
generated: '2026-09-05'
method: generated
source: openapi/commsharbor-openapi.json + https://commsharbor.com/llms.txt
operations:
  - commsharbor_contact_marketing_put
  - commsharbor_audience_create
  - commsharbor_template_publish
  - commsharbor_campaign_create
  - commsharbor_campaign_launch
  - commsharbor_campaign_report
  - commsharbor_campaign_update
---

# Launch a marketing campaign

Marketing requires explicit consent or a documented relationship; global, organization and SES tenant suppressions are checked before quota and enqueue. Campaign writes need the `campaign:write` permission.

1. Record consent per contact with `commsharbor_contact_marketing_put` (`PUT .../crm/contacts/{contact_id}/marketing`). Being in the CRM is not permission to email.
2. Build the audience with `commsharbor_audience_create` (`POST .../audiences`) — a static audience or a saved segment with an allowlisted filter; add members with `commsharbor_audience_member_add`.
3. Publish the marketing template with `commsharbor_template_publish` (`POST .../templates/{template_id}/publish`) — an immutable, content-hashed version.
4. Create the draft with `commsharbor_campaign_create` from an **active** domain, the audience, and the published marketing template.
5. Launch with `commsharbor_campaign_launch` (`POST .../campaigns/{campaign_id}/launch`). `Idempotency-Key` is required: the launch freezes the eligible recipients **exactly once**, and a replay never launches twice. A `429` means the frozen recipient set exceeds remaining capacity (the global bootstrap cap is 1,000 real recipients per month).
6. Reconcile with `commsharbor_campaign_report` — delivery counts and normalized feedback counts.

**Reversal:** `commsharbor_campaign_update` (`PATCH .../campaigns/{campaign_id}`) pauses, resumes or cancels a campaign that already launched; `commsharbor_messaging_settings_update` pauses all organization marketing. Messages already dispatched cannot be recalled.
