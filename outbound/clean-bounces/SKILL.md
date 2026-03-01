---
name: clean-bounces
description: Remove bounced and complained contacts from audience segments. Keeps lists clean and protects sender reputation. Use after classify-replies or when bounce rate exceeds 2%.
---

# Clean Bounces

Remove bounced, complained, and unsubscribed contacts from audience segments. Protects sender reputation and keeps lists deliverable.

## Workflow

1. **List accounts**
   - Call `list_email_accounts` to get all active accounts

2. **Find flagged sent emails** (for each account)
   - Call `list_sent_emails` with `tag_filter: "bounced OR complained OR unsubscribed"`
   - Collect all matching emails

3. **Extract unique recipients**
   - From each flagged sent email, extract the recipient email addresses
   - Deduplicate across all accounts

4. **Remove from audiences**
   - For each bounced/complained/unsubscribed contact:
     - Call `remove_from_audience` with all segments they belong to
   - This removes the audience segment tags from all messages involving that contact

5. **Verify improvement**
   - Call `list_metrics` for each account
   - Check that bounce rate is below 2% and complaint rate is below 0.1%

6. **Alert on high rates**
   - If bounce rate > 2%: flag the account — may need to pause sending
   - If complaint rate > 0.1%: flag the account — immediate action needed
   - Output a summary table with rates per account and flagged accounts

## When to Run

- **After classify-replies** — once replies are tagged, clean bounces removes bad contacts
- **Before send-campaign** — ensure lists are clean before sending
- **When bounce rate exceeds 2%** — immediate cleanup needed
- **Weekly maintenance** — regular hygiene keeps deliverability high

## Rate Thresholds

| Metric | Healthy | Warning | Critical |
|--------|---------|---------|----------|
| Bounce rate | < 1% | 1-2% | > 2% |
| Complaint rate | < 0.05% | 0.05-0.1% | > 0.1% |

At critical levels, pause outbound sending from the affected account until the rate drops.
