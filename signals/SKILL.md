---
name: signals
description: Detect buying signals for a company domain via API calls. Scans Trustpilot reviews, social media follower spikes, and LinkedIn hiring activity. Use sub-skills for targeted detection or detect-all for a full scan.
---

# Buying Signals Detection

Detect buying signals for a target company using its domain. All operations use the signals-tools API via **Bash** (`https://signals.gtm-engine.sh/api/v0`).

## Getting an API Key

To use the signals tools, you need a `GTM_ENGINE_API_KEY`. Obtain one via the `get_api_key` tool:

**Step 1 — Request a verification code:**
```bash
curl -s -X POST "https://gtm-engine.sh/api/v0/get_api_key" \
  -H "Content-Type: application/json" \
  -d '{"email":"YOUR_EMAIL"}'
```

**Step 2 — Submit the 6-digit code received by email:**
```bash
curl -s -X POST "https://gtm-engine.sh/api/v0/get_api_key" \
  -H "Content-Type: application/json" \
  -d '{"email":"YOUR_EMAIL","code":"123456"}'
```

The response contains your API key. Export it:
```bash
export GTM_ENGINE_API_KEY="<your-key>"
```

## Architecture

- **Input**: a company domain string (e.g. `"gymshark.com"`)
- **Output**: triggered signals array with detection results
- **Data sources**: Trustpilot (reviews), Instagram/TikTok (follower stats), LinkedIn (job listings), website HTML (technology detection)
- **Token costs**: 5 tokens per individual signal tool, 15 tokens for `detect_signal` (runs all checks)

## How to Call Tools

Use the Bash tool to call the API. See `references/tools-reference.md` for the exact command for each tool.

**Pattern:**
```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/TOOL_NAME" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{...arguments...}' | jq .
```

Environment variable `GTM_ENGINE_API_KEY` must be set (see **Getting an API Key** above).

## Available Tools

See `references/tools-reference.md` for full commands and parameter details on all 13 tools.

### Meta
- `detect_signal`:Run all signal detections at once (order customisable). 15 tokens.
- `set_signals_order`:Set which signals `detect_signal` runs and in what order. 0 tokens.
- `get_signals_order`:Get the current signal execution order. 0 tokens.

### Reputation (Trustpilot)
- `signal_trustpilot_negative_reviews`:1-star reviews in the last 30 days. 5 tokens.
- `signal_trustpilot_negative_support_reviews`:1-star reviews mentioning "support" in the last 30 days. 5 tokens.
- `signal_trustpilot_positive_reviews`:4-5 star reviews in the last 30 days. 5 tokens.

### Social Growth
- `signal_socials_spike`:Follower spikes on Instagram/TikTok over 14 days. 5 tokens.

### Hiring (5 tokens each)
- `signal_hiring_role`:generic — job listings matching a custom title filter
- `signal_hiring_support`:preset — CX/customer support roles
- `signal_hiring_sales_rep`:preset — SDR/BDR roles (budget allocation signal)
- `signal_hiring_sales_leadership`:preset — Head of Sales/VP Sales/RevOps (new leadership signal)
- `signal_hiring_sales_rep_repost`:preset — SDR role reposted within 60 days (churn/failure signal)

### Technology
- `signal_technologies_identified`:Detect whether specific technologies are used on a website. 5 tokens.

## Sub-Skills

| Skill | Use When |
|-------|----------|
| `detect-all` | Quick full scan of a domain for all signals |
| `reputation` | Analyzing Trustpilot review sentiment (negative, positive, support-related) |
| `social-growth` | Checking for social media follower spikes |
| `hiring` | Checking if a company is hiring specific roles |
| `technology` | Detecting specific technologies on a company's website |

## Quick Routing

**Want a full signal scan of a domain?** → `detect-all`

**Need to analyze Trustpilot reviews (negative, positive, support)?** → `reputation`

**Checking for social media follower growth spikes?** → `social-growth`

**Looking for hiring activity on LinkedIn?** → `hiring`

**Checking if a company uses specific technologies?** → `technology`
