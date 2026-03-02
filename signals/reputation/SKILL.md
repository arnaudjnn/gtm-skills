---
name: reputation
description: Analyze Trustpilot review sentiment for a company. Run negative, positive, and support-related review detections to build a full reputation picture.
---

# Reputation Analysis

Analyze a company's Trustpilot review sentiment using three specialized API calls. Use this skill when you need a detailed breakdown of review sentiment rather than a quick signal scan.

## Tools Used

- `signal_trustpilot_negative_reviews` (5 tokens):1-star reviews in the last 30 days
- `signal_trustpilot_negative_support_reviews` (5 tokens):1-star reviews mentioning "support"
- `signal_trustpilot_positive_reviews` (5 tokens):4-5 star reviews in the last 30 days

See `references/tools-reference.md` for exact commands.

## Workflow

1. **Run all 3 Trustpilot tools**
   - Call each tool with the target `domain` using the Bash tool
   - All three can be called in parallel (same data source, independent filters)

2. **Analyze the sentiment balance**
   - Compare `positive_reviews_count` vs `negative_reviews_count`
   - Check if negative support reviews are a subset of all negative reviews
   - Note whether the company replies to reviews (`has_company_reply`)

3. **Identify patterns**
   - Look for recurring themes in negative review titles/messages
   - Check if support complaints are a significant portion of negative reviews
   - Note review dates:are problems recent or ongoing?

4. **Output a reputation report**
   - Total positive vs negative reviews (last 30 days)
   - Support-specific complaint count
   - Company response rate (count of `has_company_reply: true`)
   - Key themes from review content
   - Trustpilot URL for reference

## Review Rating Thresholds

| Tool | Rating Filter | Time Window |
|------|---------------|-------------|
| `signal_trustpilot_negative_reviews` | Rating = 1 | Last 30 days |
| `signal_trustpilot_negative_support_reviews` | Rating = 1 AND "support" in title/message | Last 30 days |
| `signal_trustpilot_positive_reviews` | Rating >= 4 | Last 30 days |

## Use Cases

- "What's the Trustpilot sentiment for acme.com?"
- "Are customers complaining about support at gymshark.com?"
- "Compare positive vs negative reviews for notion.so"
- "Is this company's reputation getting better or worse?"
