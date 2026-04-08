---
name: send
description: >
  This skill should be used when the user says "send a WhatsApp", "send WhatsApp to",
  "message someone on WhatsApp", "/meta", or wants to send a WhatsApp text or template
  message to one or more phone numbers via the Meta Business API.
metadata:
  version: "0.1.0"
  allowed-tools:
    - Read
    - Bash(curl *)
    - Bash(cat *)
---

Send WhatsApp messages via the Meta WhatsApp Business API.

Parse the user's request to extract:
- **Recipients**: one or more phone numbers (comma-separated)
- **Message type**: plain text OR a named template with optional parameters
- **Message content**: the text body, or template name + positional params

If no recipients or message are provided, print usage for both formats and show credential status (Step 1), then stop.

---

## Step 1 — Load credentials

```bash
cat ~/.claude/channels/whatsapp/.env
```

Extract:
- `META_WA_PHONE_NUMBER_ID`
- `META_WA_ACCESS_TOKEN`
- `META_WA_API_VERSION` (default: `v18.0` if unset)

If `META_WA_PHONE_NUMBER_ID` or `META_WA_ACCESS_TOKEN` are missing, stop and tell the user:
> "WhatsApp not configured. Use the **configure** skill (or say 'configure WhatsApp') to save your credentials first."

---

## Step 2 — Parse the request

**Format A — text message:**
```
<numbers> <message text...>
```
- First token(s) before the message: comma-separated phone numbers
- Remainder: message body

**Format B — template message:**
```
template <numbers> <template_name> [param1 param2 ...]
```
- First token: literal `template`
- Second token: comma-separated phone numbers
- Third token: template name
- Remaining tokens: positional params for `{{1}}`, `{{2}}`, etc.

---

## Step 3 — Normalize each phone number

Apply these rules in order:
1. Strip spaces, dashes, and parentheses.
2. Starts with `+` → use as-is.
3. Starts with `00` → replace leading `00` with `+`.
4. Exactly 10 digits → prepend `+91` (India default).
5. Otherwise → prepend `+`.

---

## Step 4 — Send messages

### Text message

For each normalized number, run:

```bash
curl -s -X POST \
  "https://graph.facebook.com/${META_WA_API_VERSION}/${META_WA_PHONE_NUMBER_ID}/messages" \
  -H "Authorization: Bearer ${META_WA_ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d "{
    \"messaging_product\": \"whatsapp\",
    \"recipient_type\": \"individual\",
    \"to\": \"NORMALIZED_NUMBER\",
    \"type\": \"text\",
    \"text\": {
      \"preview_url\": false,
      \"body\": \"MESSAGE_TEXT\"
    }
  }"
```

### Template message

Build a params array from positional args: `[{"type":"text","text":"param1"}, ...]`

For each normalized number, run:

```bash
curl -s -X POST \
  "https://graph.facebook.com/${META_WA_API_VERSION}/${META_WA_PHONE_NUMBER_ID}/messages" \
  -H "Authorization: Bearer ${META_WA_ACCESS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d "{
    \"messaging_product\": \"whatsapp\",
    \"to\": \"NORMALIZED_NUMBER\",
    \"type\": \"template\",
    \"template\": {
      \"name\": \"TEMPLATE_NAME\",
      \"language\": {\"code\": \"en_US\"},
      \"components\": [{
        \"type\": \"body\",
        \"parameters\": PARAMS_ARRAY
      }]
    }
  }"
```

Omit `components` entirely if no params were provided.

---

## Step 5 — Report results

For each number:
- Success (`messages[0].id` present in response): `✓ Sent to <number> — ID: <message_id>`
- Error: show `error.message` from response verbatim

Final summary line: `X of Y sent successfully.`

**Never print the access token. If it must appear in output, mask it as `EAAx...`.**

### Common errors

- **401** — Token expired. Tell the user to re-run the configure skill with a fresh token.
- **400** — Invalid number, unknown template, or wrong param count — display Meta's error message verbatim.
