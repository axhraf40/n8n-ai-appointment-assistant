# Template publication review

Reviewed the previously generated generic export. The original working workflow was not edited.

## Confirmed checks

- JSON parsed successfully.
- All embedded JavaScript Code/tool bodies passed JavaScript syntax compilation.
- The template is inactive.
- No matching Groq `gsk_` or Cal.com `cal_live_` secret pattern was detected by the targeted scan. This targeted scan is not a comprehensive secret audit.

## Issues requiring resolution before a paid release

1. **Credentials are only partly migrated.** Several Code nodes still make direct authenticated HTTP requests. Creating a similarly named credential in n8n does not make it available to that code automatically. The existing guide incorrectly implies that these credentials and code placeholders are connected.
2. **Cal.com instructions and code disagree.** The builder's setup guide instructs buyers to replace `YOUR_CALCOM_API_KEY`, but the original Cal.com tools use headers without that placeholder. Authentication must be implemented and verified for the buyer's account.
3. **Generic catalogue and retained rules disagree.** The catalogue was replaced with `service_1`/`service_2`, while legacy photography mappings remain elsewhere, including CRM extraction. Replacing brand text alone does not make these rules general-purpose.
4. **Booking-result propagation needs regression tests.** The previous conversion replaced the original response formatter with a shorter implementation. It no longer performs the same booking-result and CRM-state recovery as the original workflow.
5. **Anthropic authentication needs cleanup.** The follow-up HTTP node gained a Header Auth credential slot while retaining its explicit `x-api-key` placeholder header. Remove the duplicate header and verify credential resolution.
6. **Instance metadata removed in this publication copy.** The exported workflow ID, instance metadata and tags were removed before upload.
7. **Sandbox claims are unverified.** The test webhook feeds the main workflow; a test entry point alone does not guarantee that downstream messaging, CRM or booking side effects are suppressed.

## Publication status

The repository is intended as a private draft for the owner. Its preview screenshot shows a generated documentation overview, explicitly labelled as such. The local n8n server was unavailable, so no actual n8n editor screenshot or live execution verification is included. Resolve the functional findings above before listing this template as a finished product.
