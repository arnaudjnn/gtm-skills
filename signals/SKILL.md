---
name: signals
description: Detect buying signals for a company domain via MCP tools. Scans Trustpilot reviews, social media follower spikes, and LinkedIn hiring activity. Use sub-skills for targeted detection or detect-all for a full scan.
---

# Buying Signals Detection

Detect buying signals for a target company using its domain. All operations use MCP tools from the signals-tools server (gtm-engine.sh).

## Architecture

- **Input**: a company domain string (e.g. `"gymshark.com"`)
- **Output**: triggered signals array with detection results
- **Data sources**: Trustpilot (reviews), Instagram/TikTok (follower stats), LinkedIn (job listings)
- **Token costs**: 5 tokens per individual signal tool, 15 tokens for `detect_signal` (runs 3 checks)

## MCP Tools Available

See `references/tools-reference.md` for full parameter details on all 6 tools.

### Meta
- `detect_signal` — Run all signal detections at once (socials spike + negative support reviews + CX hiring). 15 tokens.

### Reputation (Trustpilot)
- `signal_trustpilot_negative_reviews` — 1-star reviews in the last 30 days. 5 tokens.
- `signal_trustpilot_negative_support_reviews` — 1-star reviews mentioning "support" in the last 30 days. 5 tokens.
- `signal_trustpilot_positive_reviews` — 4-5 star reviews in the last 30 days. 5 tokens.

### Social Growth
- `signal_socials_spike` — Follower spikes on Instagram/TikTok over 14 days. 5 tokens.

### Hiring
- `signal_hiring_role` — LinkedIn job listings matching a title filter. 5 tokens.

## Sub-Skills

| Skill | Use When |
|-------|----------|
| `detect-all` | Quick full scan of a domain for all signals |
| `reputation` | Analyzing Trustpilot review sentiment (negative, positive, support-related) |
| `social-growth` | Checking for social media follower spikes |
| `hiring` | Checking if a company is hiring specific roles |

## Quick Routing

**Want a full signal scan of a domain?** → `detect-all`

**Need to analyze Trustpilot reviews (negative, positive, support)?** → `reputation`

**Checking for social media follower growth spikes?** → `social-growth`

**Looking for hiring activity on LinkedIn?** → `hiring`
