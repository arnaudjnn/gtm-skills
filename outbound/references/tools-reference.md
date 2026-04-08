# Outbound Tools API Reference

Complete reference for all 12 tools provided by the outbound-tools server.

---

## How to Call Tools

Use the Bash tool to run curl commands. Every call follows this pattern:

```bash
curl -s -X POST "$OUTBOUND_TOOLS_URL/api/v0/TOOL_NAME" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OUTBOUND_API_KEY" \
  -d '{...arguments...}'
```

- `$OUTBOUND_TOOLS_URL`:the server base URL (set during setup)
- `$OUTBOUND_API_KEY`:the API key for authentication (set during setup)
- The response is direct JSON (parse with `jq .`)

### Tip: parse responses with jq

```bash
curl -s -X POST "$OUTBOUND_TOOLS_URL/api/v0/list_email_accounts" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OUTBOUND_API_KEY" \
  -d '{}' | jq .
```

---

## list_email_accounts

List all registered mailbox accounts.

```bash
curl -s -X POST "$OUTBOUND_TOOLS_URL/api/v0/list_email_accounts" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OUTBOUND_API_KEY" \
  -d '{}'
```

**Returns:** Array of account objects:
| Field | Type | Description |
|-------|------|-------------|
| id | string | Mailbox ID |
| email | string | Email address |
| firstName | string | Account first name |
| lastName | string | Account last name |
| status | string | Account status |
| domain | string | Email domain |

---

## send_email

Send an email via SMTP. The sent message is automatically copied to the Sent folder.

```bash
curl -s -X POST "$OUTBOUND_TOOLS_URL/api/v0/send_email" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OUTBOUND_API_KEY" \
  -d '{
    "from":"sender@domain.com",
    "to":["recipient@example.com"],
    "subject":"Hello",
    "text":"Plain text body"
  }'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| from | string | yes | Sender email address (must be a registered mailbox) |
| to | string[] | yes | Recipient email addresses |
| subject | string | yes | Email subject |
| text | string | no | Plain text body |
| html | string | no | HTML body |
| cc | string[] | no | CC recipients |
| bcc | string[] | no | BCC recipients |

At least one of `text` or `html` is required.

**Returns:** `{ messageId, accepted, rejected }`

---

## list_received_emails

Fetch received emails from the INBOX with pagination and optional tag filtering.

```bash
curl -s -X POST "$OUTBOUND_TOOLS_URL/api/v0/list_received_emails" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OUTBOUND_API_KEY" \
  -d '{
    "email":"user@domain.com",
    "limit":50,
    "page":1
  }'
```

**Parameters:**
| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| email | string | yes |:| Email account |
| limit | number | no | 50 | Emails per page |
| page | number | no | 1 | Page number (1-indexed, most recent first) |
| tag_filter | string | no |:| Boolean tag filter expression |

**Tag filter examples:** `"interested"`, `"classified AND interested"`, `"NOT classified"`, `"complained OR unsubscribed"`

**Returns:** `{ emails: [...], total, page, limit }`

---

## list_sent_emails

Fetch sent emails with pagination and optional tag filtering.

```bash
curl -s -X POST "$OUTBOUND_TOOLS_URL/api/v0/list_sent_emails" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OUTBOUND_API_KEY" \
  -d '{
    "email":"user@domain.com",
    "limit":50,
    "page":1,
    "tag_filter":"NOT bounced"
  }'
```

**Parameters:**
| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| email | string | yes |:| Email account |
| limit | number | no | 50 | Emails per page |
| page | number | no | 1 | Page number (1-indexed, most recent first) |
| tag_filter | string | no |:| Boolean tag filter expression |

**Returns:** `{ emails: [...], total, page, limit }`

---

## find_reply_threads

Match received inbox emails to their original sent emails. Useful for identifying which outbound emails got replies.

```bash
curl -s -X POST "$OUTBOUND_TOOLS_URL/api/v0/find_reply_threads" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OUTBOUND_API_KEY" \
  -d '{
    "email":"user@domain.com",
    "unclassifiedOnly":true
  }'
```

**Parameters:**
| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| email | string | yes |:| Email account to analyze |
| receivedLimit | number | no | 50 | Max received emails to check |
| sentLimit | number | no | 200 | Max sent emails to match against |
| unclassifiedOnly | boolean | no | true | Only check emails without the 'classified' flag |

**Returns:** `{ matches: [{ received, sent }], unmatchedUids: [...], totalChecked }`

- `matches`:pairs of received emails matched to their original sent email
- `unmatchedUids`:UIDs of received emails that couldn't be matched to a sent email

---

## list_metrics

Get bounce, complaint, and interest rate metrics for an email account.

```bash
curl -s -X POST "$OUTBOUND_TOOLS_URL/api/v0/list_metrics" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OUTBOUND_API_KEY" \
  -d '{"email":"user@domain.com"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| email | string | yes | Email account to get metrics for |

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| totalSent | number | Total emails in Sent folder |
| bounced | number | Count of emails tagged `bounced` |
| complained | number | Count of emails tagged `complained` |
| interested | number | Count of emails tagged `interested` |
| bounce_rate | number | Bounce percentage (2 decimal places) |
| complain_rate | number | Complaint percentage (2 decimal places) |
| interest_rate | number | Interest percentage (2 decimal places) |

---

## add_email_tag

Add an IMAP keyword (tag) to an email message.

```bash
curl -s -X POST "$OUTBOUND_TOOLS_URL/api/v0/add_email_tag" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OUTBOUND_API_KEY" \
  -d '{
    "email":"user@domain.com",
    "uid":123,
    "tag":"interested",
    "folder":"INBOX"
  }'
```

**Parameters:**
| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| email | string | yes |:| Email account that owns the mailbox |
| uid | number | yes |:| UID of the email message |
| tag | string | yes |:| IMAP keyword to add |
| folder | "INBOX" \| "SENT" | no | "INBOX" | Folder containing the message |

**Returns:** Confirmation message.

---

## remove_email_tag

Remove an IMAP keyword (tag) from an email message.

```bash
curl -s -X POST "$OUTBOUND_TOOLS_URL/api/v0/remove_email_tag" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OUTBOUND_API_KEY" \
  -d '{
    "email":"user@domain.com",
    "uid":123,
    "tag":"interested",
    "folder":"INBOX"
  }'
```

**Parameters:**
| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| email | string | yes |:| Email account that owns the mailbox |
| uid | number | yes |:| UID of the email message |
| tag | string | yes |:| IMAP keyword to remove |
| folder | "INBOX" \| "SENT" | no | "INBOX" | Folder containing the message |

**Returns:** Confirmation message.

---

## add_to_audience

Add a contact email to one or more audience segments. Scans all accounts and folders (INBOX + SENT) for messages involving that contact and tags them.

```bash
curl -s -X POST "$OUTBOUND_TOOLS_URL/api/v0/add_to_audience" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OUTBOUND_API_KEY" \
  -d '{
    "email":"contact@example.com",
    "segments":["leads","vip"]
  }'
```

**Parameters:**
| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| email | string | yes |:| Contact email to segment |
| segments | string[] | no | ["general"] | Audience segments to add (e.g., `["leads", "vip"]`) |

**Returns:** Confirmation with count of tagged messages across accounts.

---

## remove_from_audience

Remove a contact email from audience segments.

```bash
curl -s -X POST "$OUTBOUND_TOOLS_URL/api/v0/remove_from_audience" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OUTBOUND_API_KEY" \
  -d '{
    "email":"contact@example.com",
    "segments":["leads"]
  }'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| email | string | yes | Contact email to remove from segments |
| segments | string[] | yes | Audience segments to remove |

**Returns:** Confirmation with count of untagged messages across accounts.

---

## list_audiences

List all audience segments with their contacts, aggregated across all accounts and folders.

```bash
curl -s -X POST "$OUTBOUND_TOOLS_URL/api/v0/list_audiences" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OUTBOUND_API_KEY" \
  -d '{}'
```

**Parameters:** None

**Returns:**
```json
{
  "segments": [
    {
      "name": "leads",
      "count": 42,
      "contacts": ["alice@example.com", "bob@example.com"]
    }
  ]
}
```
