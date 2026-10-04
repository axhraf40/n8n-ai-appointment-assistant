# Generic AI Appointment Lead Assistant

## What the buyer receives

- `Generic_AI_Appointment_Lead_Agent_Template.json` — an inactive n8n workflow template.
- This setup guide.

The workflow receives Instagram Direct Messages and WhatsApp messages, transcribes audio, replies through an AI assistant, stores leads in Airtable, checks availability through Cal.com, creates/reschedules/cancels bookings, records escalations, and sends daily lead follow-ups.

It contains no usable API tokens, no live Airtable data, no customer data, no studio name, no address, and no business prices.

## Before importing

You need:

1. An n8n account or self-hosted n8n instance.
2. Meta Business access for Instagram Messaging and/or WhatsApp Cloud API.
3. An Anthropic API key for the AI assistant.
4. A Groq API key if you want audio transcription.
5. An Airtable Personal Access Token and an Airtable base.
6. A Cal.com account and API key if you want bookings.
7. A public HTTPS webhook URL. n8n Cloud provides this; local n8n needs Cloudflare Tunnel or an equivalent.
8. Telegram bot credentials only if you want escalation notifications.

## 1. Import the workflow

In n8n:

1. Open **Workflows**.
2. Select **Import from File**.
3. Select `Generic_AI_Appointment_Lead_Agent_Template.json`.
4. Do not activate the workflow yet.
5. Find the three setup items on the canvas:
   - `SETUP — Start Here`
   - `SETUP — Credentials Required`
   - `SETUP — Business Configuration`

The workflow is intentionally imported inactive. It cannot send messages until you configure it and activate it yourself.

## 2. Create credentials in n8n

Open **Credentials** in n8n and create the following items. For header credentials, use n8n's **Header Auth** credential type.

| Credential name used by the template | Header name | Header value | Used by |
| --- | --- | --- | --- |
| Meta Instagram Page Token | `Authorization` | `Bearer YOUR_META_ACCESS_TOKEN` | Send Instagram DM, Follow-Up IG DM |
| Meta WhatsApp Cloud Token | `Authorization` | `Bearer YOUR_META_ACCESS_TOKEN` | Send WhatsApp Message, Follow-Up WhatsApp |
| Anthropic API Key | `x-api-key` | Your Anthropic API key | Anthropic Chat Model, Generate Follow-Up Claude |
| Groq API Key | `Authorization` | `Bearer YOUR_GROQ_API_KEY` | Whisper Groq |
| Airtable Personal Access Token | `Authorization` | `Bearer YOUR_AIRTABLE_PERSONAL_ACCESS_TOKEN` | Airtable code nodes |
| Cal.com API Key | `Authorization` | `Bearer YOUR_CALCOM_API_KEY` | Cal.com tool code nodes |
| Telegram Bot Token (optional) | Not a header credential in this template | Replace URL placeholder | Notify Telegram (optional) |

After importing, open each HTTP Request node whose credential is blank and select the matching credential. The workflow contains named credential slots for the main HTTP nodes.

Some Code nodes call API endpoints directly. n8n does not export secret values with a workflow. For those nodes, replace the `YOUR_...` placeholder in the code after creating the credential. Search the workflow for `YOUR_` to find every required replacement.

## 3. Configure your business

Open the disabled code node `SETUP — Business Configuration`. It lists the places you must adapt.

At minimum, update these nodes:

| Node | What to customize |
| --- | --- |
| `Get_catalogue` | Your services, packages, prices, inclusions, optional extras and currency. |
| `Normalize & Filter` | Your Cal.com username, event type slugs, packages and durations. |
| `Prep Agent Input` | Business name, assistant name, city, tone, languages, policies and what the AI may promise. |
| `Format Agent Reply` | Escalation words, fallback response and business-specific safety rules. |
| `Get_slots`, `Create_booking`, `Update_booking`, `Delete_booking` | Cal.com API key and any event type mapping required by your Cal.com account. |
| `Airtable Upsert Lead`, `Airtable Log Escalation`, `CRM Sync Relational`, relance nodes | Airtable base ID, table IDs/names and API token placeholder. |
| Instagram and WhatsApp HTTP nodes | Page ID, WhatsApp phone-number ID and Meta credentials. |
| `Notify Telegram (optional)` | Telegram bot token and destination chat ID, or leave the node unused. |

## 4. Configure Airtable

Create an Airtable base and change all `YOUR_AIRTABLE_...` placeholders to your IDs. You can obtain a base ID from the Airtable API/documentation or from tools that show the base metadata.

The minimum recommended tables are:

### Leads

Create fields that match the workflow's CRM output:

```text
sender_id
sender_id_num
prenom
channel
langue
intention
prestation
pack
dernier_message
reponse_bot
historique
escalade
mis_a_jour
appointment_id
scheduled_at
reservation_dates
annule
reporte
followup_count
last_followup_at
last_message_at
status
client_email
lien_reservation
```

### Escalations

```text
sender_id
sender_id_num
channel
intention
langue
message
reponse_bot
prestation
date
raison
ig_user_id
```

### Optional relational CRM tables

The template also supports `Clients`, `Enfants` and `Séances`. You can use these if you want a relational CRM, or disable `CRM Sync Relational` if your business only needs the `Leads` and `Escalations` tables.

Field types matter. Use text fields for IDs, long text for messages/history, checkbox or boolean fields for `escalade`, `annule`, `reporte`, number for `followup_count`, and date/time fields for timestamp values.

## 5. Configure Cal.com

1. Create the event types in Cal.com first.
2. Copy each event type slug.
3. In `Normalize & Filter`, replace `YOUR_EVENT_TYPE_...` values with those slugs.
4. Set the correct duration for every service/package.
5. Add your Cal.com API key where `YOUR_CALCOM_API_KEY` appears in the Cal.com tool code.
6. Test `Get_slots` before allowing the assistant to create a booking.

The agent must find a real free slot with `Get_slots` before calling `Create_booking`. This protects against confirming a time that is already unavailable.

## 6. Configure Instagram and WhatsApp

### Instagram

1. Connect your Instagram professional account to a Facebook Page.
2. Create a Meta app with Instagram Messaging permissions.
3. Replace `YOUR_META_PAGE_ID`, `YOUR_INSTAGRAM_ACCOUNT_ID` and `YOUR_META_ACCESS_TOKEN` in the workflow.
4. Attach the **Meta Instagram Page Token** credential to `Send Instagram DM` and `Follow-Up IG DM`.
5. In Meta, set the Callback URL to your n8n production webhook URL plus `/webhook/your-business-instagram`.

### WhatsApp

1. Set up WhatsApp Cloud API in your Meta app.
2. Replace `YOUR_WHATSAPP_PHONE_NUMBER_ID` and `YOUR_META_ACCESS_TOKEN`.
3. Attach the **Meta WhatsApp Cloud Token** credential to `Send WhatsApp Message` and `Follow-Up WhatsApp`.
4. In Meta, set the callback URL to your n8n production webhook URL plus `/webhook/your-business-whatsapp`.

The imported paths are generic placeholders. Replace the `YOUR_BUSINESS_NAME-...` webhook paths with simple, lowercase paths such as `acme-instagram` and `acme-whatsapp` before using Meta.

## 7. Configure local webhooks with Cloudflare Tunnel

When running local n8n, start n8n and then use a separate terminal:

```powershell
cloudflared tunnel --url http://localhost:5678
```

Cloudflare prints a public `https://...trycloudflare.com` URL. Use that URL with your webhook path in Meta. Keep the tunnel terminal open.

Before Meta validation, test the local webhook challenge using the workflow's final chosen path. For example:

```powershell
Invoke-WebRequest "http://127.0.0.1:5678/webhook/acme-instagram?hub.mode=subscribe&hub.verify_token=YOUR_VERIFY_TOKEN&hub.challenge=12345" -UseBasicParsing
```

The response must be `12345`.

## 8. Test checklist

Run these tests before activating production webhooks:

1. POST a text message to `Test Webhook`.
2. Confirm the run reaches `AI Booking Assistant` and `Format Agent Reply`.
3. Confirm a lead appears in Airtable after `Airtable Upsert Lead`.
4. Ask for a price and verify the answer matches your rewritten `Get_catalogue` data.
5. Ask for availability and verify `Get_slots` checks Cal.com.
6. Create a test booking using a future free slot and a test email.
7. Send a complaint or “I want a human” and verify the `Escalations` table receives a record.
8. Send an audio message and verify the transcript reaches the AI.
9. Test one Instagram and one WhatsApp message only after the sandbox tests pass.
10. Activate the workflow only after all required nodes are green.

## 9. What the buyer must never do

- Never paste a secret token into a public screenshot, support request or product review.
- Never use the seller's old IDs; every `YOUR_...` placeholder must be changed.
- Never activate the workflow before selecting credentials and testing a sandbox request.
- Never let the assistant invent prices or availability; keep the catalog and Cal.com checks enabled.

## 10. Known setup responsibilities

This is a reusable template, not a hosted service. The buyer owns and pays for their n8n account, Meta account, Airtable, AI usage, Cal.com account and any external services. API permissions, Meta app review and webhook availability depend on the buyer's own accounts.
