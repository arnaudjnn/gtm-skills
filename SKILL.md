---
name: gtm-skills
description: GTM skills using MCP servers. Outbound email workflows (campaigns, reply classification, follow-ups, bounce cleanup, analytics) and buying signal detection (Trustpilot reviews, social media spikes, LinkedIn hiring).
---

# GTM Skills

Outbound email workflows and buying signal detection, powered by MCP servers.

## Sub-Skills

| Skill | Description | Use When |
|-------|-------------|----------|
| [`outbound`](./outbound) | Core email operations | Sending emails, reading inbox/sent, tagging, managing audiences |
| [`classify-replies`](./outbound/classify-replies) | Reply classification | Categorizing replies by sentiment (interested, bounced, etc.) |
| [`send-campaign`](./outbound/send-campaign) | Campaign sending | Sending cold outreach to audience segments |
| [`follow-up`](./outbound/follow-up) | Follow-up sequences | Auto follow-up unreplied prospects |
| [`clean-bounces`](./outbound/clean-bounces) | Bounce cleanup | Removing bounced/complained contacts from audiences |
| [`analytics`](./outbound/analytics) | Campaign analytics | Generating performance reports and metrics |
| [`signals`](./signals) | Buying signal detection | Scanning a domain for buying signals (reviews, social spikes, hiring) |
| [`detect-all`](./signals/detect-all) | Full signal scan | Quick scan of a domain for all signals in one call |
| [`reputation`](./signals/reputation) | Trustpilot analysis | Analyzing positive/negative review sentiment |
| [`social-growth`](./signals/social-growth) | Social media spikes | Checking Instagram/TikTok follower growth |
| [`hiring`](./signals/hiring) | LinkedIn hiring | Checking if a company is hiring specific roles |

## Deploy the MCP Servers

### Outbound Tools

The outbound-tools server must be running before using outbound sub-skills. Deploy to Railway using the one-click template:

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/outbound-tools)

Set `MAILPOOL_API_KEY` during deployment. Optionally set `ANTHROPIC_API_KEY` to enable auto-classification via `/api/classify`.

Store the Railway URL — you'll configure it as your MCP server endpoint.

### Signals Tools

The signals-tools server is hosted at `gtm-engine.sh`. You need an API key to use signal skills.

1. Get an API key from the signals-tools MCP server by calling the `get_api_key` tool
2. Configure it as your MCP server endpoint

### Configure MCP Servers

Point your agent to the deployed server URLs:
- **Outbound**: Your Railway URL — exposes 12 MCP tools for email operations
- **Signals**: `gtm-engine.sh` — exposes 6 MCP tools for buying signal detection

## Quick Routing

**Need to send a single email or manage contacts?** → `outbound`

**Running a cold email campaign to a list?** → `send-campaign`

**Replies came in and need classification?** → `classify-replies`

**Want to follow up with non-responders?** → `follow-up`

**Bounce rate too high or need list cleanup?** → `clean-bounces`

**Need a performance report?** → `analytics`

**Want a quick signal scan of a domain?** → `detect-all`

**Need detailed Trustpilot review analysis?** → `reputation`

**Checking for social media follower spikes?** → `social-growth`

**Looking for specific hiring activity?** → `hiring`
