---
name: configure
description: >
  This skill should be used when the user wants to "configure WhatsApp", "set up WhatsApp",
  "save a phone number ID", "save an access token", "check WhatsApp credential status",
  "update the API version", "clear WhatsApp credentials", or says "/meta-configure".
metadata:
  version: "0.1.0"
  allowed-tools:
    - Read
    - Write
    - Bash(ls *)
    - Bash(mkdir *)
    - Bash(chmod *)
    - Bash(cat *)
---

Manage Meta WhatsApp Business API credentials stored in `~/.claude/channels/whatsapp/.env`.

The `.env` file holds three keys:
```
META_WA_PHONE_NUMBER_ID=<value>
META_WA_ACCESS_TOKEN=<value>
META_WA_API_VERSION=v18.0
```

Determine the user's intent from their message and dispatch to the matching action below.

---

## Action: Status (no args, or "status")

Read `~/.claude/channels/whatsapp/.env` and display:

1. **Phone Number ID** — full value, or "not set"
2. **Access Token** — first 10 characters + `...`, or "not set"
3. **API Version** — current value, or "default: v18.0"

Follow with a next-step hint:
- If not configured: *"Run the configure skill with your Phone Number ID and Access Token to get started."*
- If configured: *"Ready. Ask me to send a WhatsApp message anytime."*

---

## Action: Save credentials (`<phone_number_id> <access_token>`)

1. Run `mkdir -p ~/.claude/channels/whatsapp`
2. Read existing `.env` if present, or start with an empty string.
3. Update or add these three lines (preserve any other keys already in the file):
   ```
   META_WA_PHONE_NUMBER_ID=<phone_number_id>
   META_WA_ACCESS_TOKEN=<access_token>
   META_WA_API_VERSION=v18.0
   ```
4. Write the file back.
5. Run `chmod 600 ~/.claude/channels/whatsapp/.env`
6. Confirm success, then show status (as above).

---

## Action: Update API version (`version <version_string>`)

Update `META_WA_API_VERSION` in the `.env` file to the provided value (e.g., `v19.0`).
Confirm the change.

---

## Action: Clear credentials (`clear`)

Remove the `.env` file entirely, or — if the file contains other keys beyond the three WhatsApp keys — remove only those three keys and leave the rest intact.
Confirm to the user that credentials have been cleared.

---

## Credential reference

Obtain credentials from **Meta for Developers → your WhatsApp Business app → API Setup**:

- **Phone Number ID** — numeric string shown under your test or production phone number
- **Access Token** — temporary (24 h) or permanent system-user token

Never print the full access token; always mask it as the first 10 characters + `...`.
