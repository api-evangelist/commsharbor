---
name: commsharbor-import-contacts-with-consent
description: Import a consent-declared contact CSV safely - preview first, confirm exactly once, then fix rejected rows.
api: CommsHarbor API
generated: '2026-09-05'
method: generated
source: openapi/commsharbor-openapi.json + https://commsharbor.com/llms.txt
operations:
  - commsharbor_contact_import_preview
  - commsharbor_contact_import_confirm
  - commsharbor_contact_import_get
  - commsharbor_contact_import_errors
---

# Import contacts with consent

1. Upload the consent-declared CSV with `commsharbor_contact_import_preview` (`POST /api/organizations/{organization_id}/contact-imports`). Nothing is imported yet — the response is a safe preview.
2. Confirm with `commsharbor_contact_import_confirm` (`POST .../contact-imports/{import_id}/confirm`). The `Idempotency-Key` header is **mandatory** here: reusing it returns the same import and never creates a second queue message, which is what keeps a retry from double-importing.
3. Poll `commsharbor_contact_import_get` for the durable import's current counts.
4. List row-numbered rejections with `commsharbor_contact_import_errors` (codes like `invalid_email`, with the 1-based CSV row) so the source file can be fixed and re-imported.

Import and export files expire after seven days; durable audit and row-level result records remain. Export the CRM back out with `commsharbor_contacts_export`, also retained seven days.
