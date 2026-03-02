---
name: send-campaign
description: Send outbound email campaigns to audience segments. Handles account rotation, rate limiting, and deliverability best practices. Use when sending cold outreach or campaigns to a list.
---

# Send Campaign

Send outbound email campaigns to audience segments with account rotation and deliverability safeguards. See `references/tools-reference.md` for exact commands.

## Workflow

1. **Select target audience**
   - Call `list_audiences` to see available segments
   - Pick the target segment (e.g., `leads`, `saas-founders`)

2. **Select sender accounts**
   - Call `list_email_accounts` to get available accounts
   - Choose one or more accounts for rotation

3. **Pre-send checks** (for each contact in the segment)
   - Call `list_sent_emails` with `tag_filter: "bounced OR complained OR unsubscribed"` for each sender account
   - Skip any contact already tagged as bounced, complained, or unsubscribed
   - Skip contacts already emailed in this campaign (check for campaign tag)

4. **Send emails**
   - For each eligible contact:
     - Pick the next sender account (round-robin rotation)
     - Call `send_email` with the campaign content
     - Call `add_email_tag` on the sent email with a campaign identifier tag (e.g., `campaign_launch_mar2026`)

5. **Post-send verification**
   - Call `list_metrics` for each sender account
   - Verify bounce rate stays below 2%
   - Verify complaint rate stays below 0.1%

## Best Practices

### Volume Limits
- **New accounts** (< 30 days): max 50 emails/day per account
- **Warmed accounts** (30-90 days): max 100 emails/day per account
- **Established accounts** (90+ days): max 200 emails/day per account

### Warm-Up Schedule
- Week 1: 10-20 emails/day
- Week 2: 20-40 emails/day
- Week 3: 40-80 emails/day
- Week 4+: Gradually increase to account limit

### Deliverability
- Rotate across multiple sender accounts to spread volume
- Space emails 30-60 seconds apart (don't blast all at once)
- Stop sending from an account if bounce rate exceeds 2%
- Stop sending from an account if complaint rate exceeds 0.1%
- Use plain text or minimal HTML:avoid heavy formatting
- Personalize subject lines and opening lines

### Campaign Tags
Use a consistent naming convention for campaign tags:
- Format: `campaign_<name>_<date>` (e.g., `campaign_launch_mar2026`)
- This enables filtering and analytics per campaign later
