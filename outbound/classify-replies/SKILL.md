---
name: classify-replies
description: Classify received reply emails by sentiment and tag them. Categories: interested, complained, out-of-office, unsubscribed, bounced. Tags both reply and original sent email.
---

# Classify Replies

Automatically classify incoming reply emails by sentiment and tag them with the appropriate category. This keeps your mailbox organized and feeds into downstream skills (clean-bounces, follow-up, analytics).

## Categories

| Category | Tag | Description |
|----------|-----|-------------|
| Interested | `interested` | Positive reply, wants to learn more or continue conversation |
| Complained | `complained` | Negative reply, asked to stop, reported spam |
| Out of Office | `out-of-office` | Auto-reply, vacation, OOO message |
| Unsubscribed | `unsubscribed` | Explicitly asked to be removed from future emails |
| Bounced | `bounced` | Delivery failure, invalid address, mailbox full |

## Workflow

1. **List accounts**
   - Call `list_email_accounts` to get all active accounts

2. **Find unclassified replies** (for each account)
   - Call `find_reply_threads` with `unclassifiedOnly=true`
   - This returns matched pairs (reply ↔ original sent) and unmatched UIDs

3. **Classify each reply**
   - Read the reply content and determine which category it belongs to
   - Use the subject line, body text, and headers to decide
   - Each reply gets exactly one category

4. **Tag the reply** (INBOX)
   - `add_email_tag` with the category tag (e.g., `interested`) on the reply in INBOX
   - `add_email_tag` with `classified` on the reply in INBOX

5. **Tag the original sent email** (SENT)
   - For matched replies: `add_email_tag` with the same category tag on the sent email in SENT
   - `add_email_tag` with `classified` on the sent email in SENT

6. **Handle unmatched replies**
   - For replies that couldn't be matched to a sent email, still classify the reply
   - Tag the reply with the category + `classified` in INBOX

7. **Print summary**
   - Output a table showing: account, total classified, and count per category

## Auto vs Manual Classification

- **Auto**: If `ANTHROPIC_API_KEY` is set on the outbound-tools server, replies are classified automatically via `GET /api/classify`. Set up a Railway cron or external scheduler to hit the endpoint periodically.
- **Manual**: If `ANTHROPIC_API_KEY` is not set, use this agent skill to classify replies on demand.

## Classification Guidelines

- **interested**: Contains positive language, asks questions, requests a meeting/call, engages with the offer
- **complained**: Negative tone, "stop emailing me", "this is spam", threatens to report
- **out-of-office**: Auto-generated, mentions vacation/OOO/away, includes return date
- **unsubscribed**: "Remove me", "unsubscribe", "take me off your list"
- **bounced**: Delivery status notification, "undeliverable", "mailbox not found", "550" error codes

When ambiguous, prefer the most specific category. A bounce notification that also mentions spam → `bounced`. An OOO that also shows interest → `out-of-office` (classify on re-engagement later).
