# Test: Outbound Skill

**Skill under test:** `outbound` (and all sub-skills)
**Skill type:** Reference + Technique
**Test approach:** Retrieval and application scenarios:does the agent route correctly, use the right tools, and follow workflows?

## Setup

```
[Test scaffold] For each scenario below, commit to a specific answer with exact tool calls and parameters. You have access to all 12 tools from the outbound-tools server.
```

---

## Scenario 1: Router:Simple Send vs Campaign

**Tests:** Correct routing between outbound and send-campaign

```
Case A: "Send an email to alice@example.com introducing our product."
Case B: "Send our new product announcement to everyone in the leads segment."
```

**Expected:**
- Case A: Uses `outbound`:single `send_email` call, no campaign logic needed
- Case B: Uses `send-campaign`:audience lookup, exclusion checks, account rotation

**Failure indicators:**
- Uses send-campaign for a single email
- Sends to a full segment without exclusion checks

---

## Scenario 2: Send and Tag

**Tests:** Correct tool sequence for sending + tagging

```
Send an email from alice@acme.com to bob@prospect.com with subject "Quick question"
and plain text body "Hi Bob, are you the right person to talk to about X?".
Tag the sent email with "campaign_q1_outreach".
```

**Expected:**
1. `send_email` with from: "alice@acme.com", to: ["bob@prospect.com"], subject, text
2. `add_email_tag` on the sent email UID in SENT folder with tag "campaign_q1_outreach"

**Failure indicators:**
- Missing the `add_email_tag` step
- Tagging in INBOX instead of SENT
- Passing `to` as a string instead of array

---

## Scenario 3: Tag Filter Syntax

**Tests:** Correct boolean filter expressions

```
Find all sent emails that bounced or had complaints, but exclude any already classified.
What tag_filter do you use with list_sent_emails?
```

**Expected:** `(bounced OR complained) AND NOT classified` or equivalent valid boolean expression.

**Failure indicators:**
- Invalid syntax (e.g., `&&` or `||` instead of `AND`/`OR`)
- Missing the NOT classified condition
- Makes multiple API calls instead of using tag_filter

---

## Scenario 4: Classify:Clear Interest + Tagging Both Sides

**Tests:** Correct classification and dual tagging

```
Reply from bob@prospect.com (UID 42, INBOX):
Subject: Re: Quick question about your product
Body: "This sounds interesting! Can we set up a call next week?"

Original sent email UID: 101 (SENT). Account: alice@acme.com.
```

**Expected:**
- Classification: `interested`
- `add_email_tag` email=alice@acme.com, uid=42, tag="interested", folder="INBOX"
- `add_email_tag` email=alice@acme.com, uid=42, tag="classified", folder="INBOX"
- `add_email_tag` email=alice@acme.com, uid=101, tag="interested", folder="SENT"
- `add_email_tag` email=alice@acme.com, uid=101, tag="classified", folder="SENT"

**Failure indicators:**
- Missing tags on the sent email
- Missing the `classified` tag on either side
- Wrong category

---

## Scenario 5: Classify:Ambiguous OOO With Interest

**Tests:** Correct priority when categories overlap

```
Reply from carol@bigcorp.com (UID 60, INBOX):
Subject: Out of Office Re: Partnership Opportunity
Body: "I'm out of the office until March 15th. This looks really interesting
though:please reach out again after I'm back!"
```

**Expected:** Classification: `out-of-office` (OOO takes precedence:it's an auto-reply).

**Failure indicators:**
- Classifies as `interested` (the OOO nature takes precedence)
- Applies two category tags (should be exactly one)

---

## Scenario 6: Campaign:Pre-Send Exclusions

**Tests:** Filtering out bad contacts before sending

```
You're about to send a campaign to the "leads" segment (50 contacts).
What checks do you run before sending?
```

**Expected:**
- `list_sent_emails` with `tag_filter: "bounced OR complained OR unsubscribed"` per sender account
- Extract flagged recipient emails, cross-reference with segment to exclude
- Check for contacts already tagged with the campaign tag (avoid double-send)

**Failure indicators:**
- Sends to the full list without exclusion checks
- Only checks one type (e.g., bounced but not complained)

---

## Scenario 7: Campaign:Mid-Send Bounce Spike

**Tests:** Reacting to unhealthy metrics during a campaign

```
You're 40 emails into a 100-email campaign from alice@acme.com.
list_metrics shows: bounce_rate 3.5%, complain_rate 0%.
What do you do?
```

**Expected:**
- Stop sending from alice@acme.com (bounce rate > 2%)
- Investigate and run clean-bounces before continuing
- Switch remaining sends to other accounts if available

**Failure indicators:**
- Continues sending despite 3.5% bounce rate
- Doesn't mention the 2% threshold

---

## Scenario 8: Follow-Up:Identifying Unreplied Prospects

**Tests:** Correct workflow for finding who to follow up with

```
It's been 5 days since the initial outreach from alice@acme.com.
Walk through how you identify prospects who need a follow-up.
```

**Expected:**
1. `list_sent_emails` with `tag_filter: "NOT bounced AND NOT complained AND NOT unsubscribed AND NOT interested"`
2. `find_reply_threads` to see which sent emails got replies
3. Filter to sent emails with no reply and old enough (5+ days)
4. Check for existing `followup_1` tags to determine sequence position
5. `send_email` follow-up, then `add_email_tag` with `followup_1`

**Failure indicators:**
- Follows up with contacts tagged bounced/complained/unsubscribed
- Doesn't check for existing replies before sending
- Doesn't check age of original email

---

## Scenario 9: Clean Bounces:End-to-End

**Tests:** Correct cleanup workflow

```
After classify-replies ran, some sent emails are tagged "bounced".
Walk through how you clean the audience lists.
```

**Expected:**
1. `list_sent_emails` with `tag_filter: "bounced OR complained OR unsubscribed"` per account
2. Extract unique recipient emails from flagged sent emails
3. `remove_from_audience` for each bad contact
4. `list_metrics` to verify rates improved

**Failure indicators:**
- Removes from audience without checking sent tags first
- Doesn't verify metrics after cleanup

---

## Scenario 10: Analytics:Multi-Account Report

**Tests:** Correct aggregation across accounts

```
Generate a performance report across all accounts.
```

**Expected:**
1. `list_email_accounts`
2. `list_metrics` for each account
3. Aggregate totals (sum sent, bounced, complained, interested)
4. Compute overall rates
5. `list_audiences` for segment breakdown
6. Output as markdown table

**Failure indicators:**
- Only reports on one account
- Doesn't compute aggregate rates
- Missing audience segment data

---

## Scoring

| Scenario | Pass Criteria |
|----------|---------------|
| 1 - Router | Correct skill for single send vs campaign |
| 2 - Send and Tag | Correct tool sequence, SENT folder, array for `to` |
| 3 - Tag Filter | Valid boolean expression with AND/OR/NOT |
| 4 - Classify Interest | Correct category, tags on both INBOX + SENT, `classified` tag |
| 5 - Classify OOO | Picks `out-of-office` over `interested` |
| 6 - Campaign Exclusions | Checks bounced, complained, unsubscribed, campaign tag |
| 7 - Bounce Spike | Stops sending, mentions 2% threshold |
| 8 - Follow-Up | Correct filter + reply check + age check |
| 9 - Clean Bounces | Extract → remove → verify flow |
| 10 - Analytics | Multi-account aggregation with markdown output |

**Pass threshold:** 8/10 correct
