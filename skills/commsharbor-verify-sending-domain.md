---
name: commsharbor-verify-sending-domain
description: Register a sending domain, publish its DKIM records, verify it against live SES and DNS, and smoke-test it.
api: CommsHarbor API
generated: '2026-09-05'
method: generated
source: openapi/commsharbor-openapi.json + https://commsharbor.com/llms.txt
operations:
  - commsharbor_domain_create
  - commsharbor_domain_get
  - commsharbor_domain_verify
  - commsharbor_domain_smoke
  - commsharbor_domain_report
---

# Verify a sending domain

1. Register the domain with `commsharbor_domain_create` (`POST /api/organizations/{organization_id}/domains`). SES provisioning is queued idempotently — asking twice does not provision twice. The response carries the DKIM records to publish.
2. Publish the returned DKIM records (and any custom MAIL FROM / DMARC records) in the domain's DNS.
3. Call `commsharbor_domain_verify` (`POST .../domains/{domain_id}/verify`). This observes SES, DKIM, DMARC and custom MAIL FROM state **right now** and stores what was seen — active state is only ever granted from these live observations. `commsharbor_domain_get` deliberately never infers current DNS state; re-run verify when DNS changes.
4. Smoke-test with `commsharbor_domain_smoke` (`POST .../domains/{domain_id}/smoke`). It queues one controlled message to the server-side QA recipient; the request never accepts a recipient address, so a smoke can never reach a real customer.
5. Watch the domain's health with `commsharbor_domain_report` (`GET .../domains/{domain_id}/report`) — delivery and normalized feedback aggregates. SES tenant or reputation risk pauses organization marketing until a healthy observation lets an operator resume it.
