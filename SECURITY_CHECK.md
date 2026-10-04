# Publication secret check

Checked 4 October 2026.

Scope: workflow.json, README.md, BUYER_SETUP.md, REVIEW.md, preview.html and the documentation preview image in the initial GitHub commit.

- No matching Meta, Anthropic/OpenAI, Groq, GitHub, Airtable or Telegram API token patterns were found in the text files.
- No PEM private-key markers were found.
- The workflow contains no staticData or pinData, so no saved workflow client memory or pinned execution payloads are included.
- Credential references contain empty IDs and setup labels, not credential values.
- The repository file list contains no .env, private key, database, credentials export or original production workflow file.
- The long Telegram destination ID found during initial preparation was replaced before the initial upload.

This is a local static inspection of the uploaded package, not an API-key validity test or a guarantee against every possible secret format. Credentials must remain in the buyer's own n8n account. Functional release issues are documented separately in REVIEW.md.
