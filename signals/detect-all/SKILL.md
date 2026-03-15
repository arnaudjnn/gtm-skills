---
name: detect-all
description: Run all buying signal detections for a company domain in a single call. Returns only triggered signals covering social spikes, negative support reviews, and CX hiring.
---

# Detect All Signals

Run a full buying signal scan on a company domain with a single API call. This is the fastest way to check for signals:it runs 3 detections and returns only the ones that triggered.

## Tools Used

- `detect_signal` (15 tokens):runs all configured signal checks (default: `signal_socials_spike`, `signal_trustpilot_negative_support_reviews`, `signal_hiring_role`)
- `set_signals_order` (0 tokens):customise which signals run and in what order
- `get_signals_order` (0 tokens):see current signal order

## Workflow

1. **Call `detect_signal`**
   - Pass the target `domain` (e.g. `"gymshark.com"`)
   - See `references/tools-reference.md` for the exact command
   - The tool runs 3 signal checks in order and returns only triggered signals
   - Individual failures are silently skipped

2. **Interpret the results**
   - The response is an array of `{ signal_name, data }` objects
   - Empty array means no signals were detected
   - Each `data` object has the same shape as calling that signal tool directly

3. **Output a summary table**
   - For each triggered signal, summarize what was detected
   - Include key data points (e.g. spike %, review count, matching job titles)

## What It Checks

| Signal | Detection | Data Source |
|--------|-----------|-------------|
| `signal_socials_spike` | Follower spike on Instagram/TikTok | Instagram/TikTok |
| `signal_trustpilot_negative_support_reviews` | 1-star reviews mentioning "support" | Trustpilot |
| `signal_hiring_role` | CX/customer support hiring | LinkedIn |

## Use Cases

- "Scan acme.com for buying signals"
- "Are there any signals for gymshark.com?"
- "Quick signal check on notion.so"
- Batch scanning a list of prospect domains

## Customising Signal Order

Use `set_signals_order` to change which signals `detect_signal` runs and in what order:
- Available: `signal_socials_spike`, `signal_trustpilot_negative_support_reviews`, `signal_hiring_role`
- Omit a signal from the list to disable it
- Use `get_signals_order` to see the current configuration

## Limitations

- Uses hardcoded CX-focused job filter:for custom filters, use `hiring` sub-skill directly
- Only checks negative *support* reviews:for full Trustpilot analysis, use `reputation` sub-skill
- Does not include `signal_trustpilot_positive_reviews` or `signal_trustpilot_negative_reviews`
- `signal_technologies_identified` is not included in `detect_signal` — use it directly
