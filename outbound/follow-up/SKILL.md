---
name: follow-up
description: Send follow-up emails to prospects who haven't replied. Identifies unreplied sent emails and sends contextual follow-ups. Use when running multi-touch outreach sequences.
---

# Follow-Up Sequences

Automatically follow up with prospects who haven't replied to your outreach. Identifies unreplied sent emails and sends contextual follow-ups.

## Workflow

1. **List accounts**
   - Call `list_email_accounts` to get all active sender accounts

2. **Find eligible sent emails** (for each account)
   - Call `list_sent_emails` with `tag_filter: "NOT bounced AND NOT complained AND NOT unsubscribed AND NOT interested"`
   - This gives you sent emails to contacts that haven't been flagged as problematic or already engaged

3. **Identify which got replies**
   - Call `find_reply_threads` to match received replies to sent emails
   - Filter these out — only keep sent emails with no matching reply

4. **Filter by age**
   - Only follow up on emails old enough (e.g., 3+ days since sent)
   - Skip emails that are too recent — give prospects time to respond

5. **Check sequence position**
   - Check tags on each sent email for existing follow-up tags (e.g., `followup_1`, `followup_2`)
   - Determine the next step in the sequence
   - Skip if max follow-ups reached (see best practices)

6. **Send follow-ups**
   - For each eligible unreplied email:
     - Call `send_email` as a follow-up (same thread context, new message)
     - Call `add_email_tag` on the new sent email with the sequence step tag (e.g., `followup_1`)
     - Call `add_email_tag` on the original sent email with `followed_up`

## Best Practices

### Sequence Length
- **Maximum 4-7 emails** per sequence (including the original)
- Diminishing returns after 5 touches
- Final email should be a polite "break-up" email

### Timing
- **Follow-up 1**: 3 days after original
- **Follow-up 2**: 5 days after follow-up 1
- **Follow-up 3**: 7 days after follow-up 2
- **Follow-up 4+**: 7-10 days between each

### Content
- Each follow-up should add new value, not just "checking in"
- Reference the previous email briefly
- Keep follow-ups shorter than the original
- Vary the angle: social proof, case study, different benefit, urgency
- Final email: acknowledge no response, leave the door open

### Tags
- `followup_1`, `followup_2`, etc. — track sequence position
- `followed_up` — mark original email as having a follow-up sent
- `sequence_complete` — mark when all follow-ups are exhausted
