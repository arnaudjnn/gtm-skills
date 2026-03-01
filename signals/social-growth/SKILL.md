---
name: social-growth
description: Detect social media follower spikes on Instagram and TikTok for a company. Analyzes a 14-day window to flag unusual growth patterns.
---

# Social Growth Detection

Detect significant follower spikes or drops on Instagram and TikTok for a company. Use this skill when you need detailed social growth analysis beyond what `detect-all` provides.

## MCP Tools Used

- `signal_socials_spike` (5 tokens) — 14 days of daily follower stats

## Workflow

1. **Call `signal_socials_spike`**
   - Pass the target `domain` (e.g. `"gymshark.com"`)
   - The tool finds the company's Instagram and/or TikTok profiles from their website
   - Fetches 14 days of daily follower stats

2. **Interpret the results**
   - `spike_detected`: whether any threshold was met
   - `growth_percentage`: average growth across available platforms
   - `spike_reason`: human-readable explanation of why the spike was flagged (or null)
   - Per-platform history arrays for detailed day-by-day analysis

3. **Analyze the history data**
   - Look at `followers_gained` per day to identify the spike day(s)
   - Compare to `followers_total` to gauge relative significance
   - Check both platforms if available — a spike on one may not appear on the other

4. **Output a growth summary**
   - Whether a spike was detected and why
   - Growth percentage over the 14-day window
   - Platform-specific breakdown (Instagram vs TikTok)
   - Notable days with unusual activity

## Spike Detection Thresholds

| Condition | Threshold | Description |
|-----------|-----------|-------------|
| Overall growth | >= 10% | Total follower growth across the 14-day window |
| Single-day spike | > 3x daily average AND > 0.1% of total followers | One day significantly above normal |
| Day-over-day acceleration | > 2x previous day AND > 0.1% of total followers | Sudden jump compared to the day before |

## Use Cases

- "Is gymshark.com seeing a social media spike?"
- "Check Instagram and TikTok growth for acme.com"
- "Has notion.so had any unusual follower activity recently?"
- "What's the social media trend for this prospect?"
