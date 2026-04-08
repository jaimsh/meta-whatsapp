# meta-whatsapp

Send WhatsApp messages and manage Meta WhatsApp Business API credentials directly from Claude.

## Overview

This plugin gives Claude two skills for working with the Meta WhatsApp Business API:

- **send** — send text or template messages to one or more recipients
- **configure** — save, check, update, or clear your API credentials

Credentials are stored locally at `~/.claude/channels/whatsapp/.env` and never transmitted except as part of authorized API calls to Meta's Graph API.

## Setup

Before sending messages, you need a Meta for Developers account with a WhatsApp Business app configured.

1. Go to [developers.facebook.com](https://developers.facebook.com) → your WhatsApp Business app → **API Setup**
2. Note your **Phone Number ID** (numeric string under your phone number)
3. Generate a **temporary** (24 h) or **permanent** (system-user) **Access Token**
4. In Claude, say: *"Configure WhatsApp with phone number ID `<id>` and access token `<token>`"*

## Skills

### send

Trigger phrases: "send a WhatsApp", "send WhatsApp to", "message on WhatsApp"

**Text message:**
> "Send a WhatsApp to +919876543210 saying 'Your order has shipped!'"

**Multiple recipients:**
> "WhatsApp +919876543210, +14155551234 — 'Meeting starts in 10 minutes'"

**Template message:**
> "Send WhatsApp template hello_world to +919876543210"
> "Send WhatsApp template order_update to +919876543210 with params 'ORD-123' 'delivered'"

Phone number formats accepted: international (`+91...`), `00`-prefixed, or bare 10-digit Indian numbers (auto-prefixed with `+91`).

### configure

Trigger phrases: "configure WhatsApp", "set up WhatsApp", "check WhatsApp status", "clear WhatsApp credentials"

| Action | What to say |
|--------|-------------|
| Check status | "WhatsApp status" |
| Save credentials | "Configure WhatsApp `<phone_number_id>` `<access_token>`" |
| Update API version | "Set WhatsApp API version to v19.0" |
| Clear credentials | "Clear WhatsApp credentials" |

## Environment Variables

No environment variables need to be set manually. The configure skill writes credentials to `~/.claude/channels/whatsapp/.env` with permissions set to `600` (owner-read only).
