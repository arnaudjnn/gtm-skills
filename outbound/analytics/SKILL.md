---
name: analytics
description: Generate email campaign analytics and performance reports. Track bounce, complaint, and interest rates across accounts. Use when reviewing campaign health or generating outreach reports.
---

# Campaign Analytics

Generate performance reports across all email accounts. Track deliverability, engagement, and audience health.

## Workflow

1. **List accounts**
   - Call `list_email_accounts` to get all active accounts

2. **Collect metrics** (for each account)
   - Call `list_metrics` to get bounce, complaint, and interest rates
   - Record: totalSent, bounced, complained, interested, and computed rates

3. **Aggregate across accounts**
   - Sum totals across all accounts
   - Compute overall rates (total bounced / total sent, etc.)

4. **Audience breakdown**
   - Call `list_audiences` to get all segments
   - Record segment names, contact counts

5. **Optional drill-down**
   - Use `list_sent_emails` with `tag_filter` to break down by campaign tag
   - Use `list_received_emails` with `tag_filter` to see reply categories
   - Example: `tag_filter: "campaign_launch_mar2026"` to see all emails from a specific campaign

6. **Output report**

Format as a markdown report:

```markdown
## Email Campaign Report

### Account Performance

| Account | Sent | Bounced | Complained | Interested | Bounce % | Complaint % | Interest % |
|---------|------|---------|------------|------------|----------|-------------|------------|
| alice@domain.com | 500 | 5 | 1 | 45 | 1.00% | 0.20% | 9.00% |
| bob@domain.com | 300 | 2 | 0 | 30 | 0.67% | 0.00% | 10.00% |
| **Total** | **800** | **7** | **1** | **75** | **0.88%** | **0.13%** | **9.38%** |

### Health Status

- Bounce rate: 0.88% ✓ (healthy < 2%)
- Complaint rate: 0.13% ⚠ (warning > 0.1%)

### Audience Segments

| Segment | Contacts |
|---------|----------|
| leads | 250 |
| saas-founders | 85 |
| replied | 75 |
```

## Metrics Reference

| Metric | Source | Meaning |
|--------|--------|---------|
| totalSent | Sent folder count | Total emails sent from account |
| bounced | `bounced` tag in Sent | Emails that failed delivery |
| complained | `complained` tag in Sent | Recipients who marked as spam |
| interested | `interested` tag in Sent | Recipients who showed interest |
| bounce_rate | bounced / totalSent | Delivery failure rate |
| complain_rate | complained / totalSent | Spam complaint rate |
| interest_rate | interested / totalSent | Positive engagement rate |
