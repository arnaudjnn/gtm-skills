---
name: outbound
description: Send and manage outbound emails via API calls. List accounts, send emails, read inbox/sent, tag messages, manage audience segments. Use for cold email outreach and contact management.
---

# Outbound Email Operations

Core skill for sending and managing outbound emails. All operations use the outbound-tools server via **Bash** (IMAP/SMTP + Mailpool).

## Architecture

- **No external database**: IMAP keywords (flags) act as the tagging/state layer
- Tags are stored directly on email messages as IMAP keywords
- Audience segments are encoded as IMAP keywords on messages (e.g., `audience_vip`)
- Fully portable: all state lives in the mailbox itself

## How to Call Tools

Use the Bash tool to call the API. See `references/tools-reference.md` for the exact command for each tool.

**Pattern:**
```bash
curl -s -X POST "$OUTBOUND_TOOLS_URL/api/v0/TOOL_NAME" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OUTBOUND_API_KEY" \
  -d '{...arguments...}' | jq .
```

Environment variables `OUTBOUND_TOOLS_URL` and `OUTBOUND_API_KEY` must be set (see `setup` skill).

## Available Tools

See `references/tools-reference.md` for full commands and parameter details on all 12 tools.

### Account Management
- `list_email_accounts`:List all registered mailbox accounts

### Email Operations
- `send_email`:Send an email (plain text or HTML, with CC/BCC)
- `list_received_emails`:Read inbox with pagination and tag filtering
- `list_sent_emails`:Read sent folder with pagination and tag filtering

### Tagging
- `add_email_tag`:Add an IMAP keyword to a message (INBOX or SENT)
- `remove_email_tag`:Remove an IMAP keyword from a message

### Thread Matching
- `find_reply_threads`:Match received replies to original sent emails

### Audiences
- `add_to_audience`:Add a contact email to audience segments
- `remove_from_audience`:Remove a contact from audience segments
- `list_audiences`:List all audience segments with contacts

### Metrics
- `list_metrics`:Get bounce, complaint, and interest rates for an account

## Key Workflows

### Send an email
1. Call `list_email_accounts` → pick sender
2. Call `send_email` with from, to, subject, and text/html body
3. Optionally call `add_email_tag` on the sent message with a campaign tag

### Check for responses
1. Call `list_received_emails` for an account
2. Call `find_reply_threads` to match replies to sent emails
3. Call `add_email_tag` to classify each reply

### Manage audiences
1. Call `add_to_audience`:add contacts to segments (e.g., `["leads", "saas"]`)
2. Call `list_audiences`:see all segments and their contacts
3. Call `remove_from_audience`:remove contacts from segments

## Tag Filter Syntax

The `tag_filter` parameter on `list_received_emails` and `list_sent_emails` supports boolean expressions:

- Single tag: `interested`
- AND: `classified AND interested`
- OR: `bounced OR complained`
- NOT: `NOT classified`
- Combined: `NOT bounced AND NOT complained AND NOT unsubscribed`

## Sub-Skills

| Skill | Use When |
|-------|----------|
| `classify-replies` | Categorize incoming replies by sentiment |
| `send-campaign` | Send outreach to an audience segment |
| `follow-up` | Follow up with non-responders |
| `clean-bounces` | Remove bounced/complained from audiences |
| `analytics` | Generate campaign performance reports |
