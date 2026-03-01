---
name: gtm-skills
description: GTM outbound email skills using an MCP server. Deploy on Railway, then use sub-skills for campaigns, reply classification, follow-ups, bounce cleanup, and analytics.
---

# GTM Skills

Outbound email workflow skills powered by an MCP server (outbound-tools). All email operations go through MCP tools backed by IMAP/SMTP + Mailpool on Railway.

## Sub-Skills

| Skill | Description | Use When |
|-------|-------------|----------|
| [`outbound`](./outbound) | Core email operations | Sending emails, reading inbox/sent, tagging, managing audiences |
| [`classify-replies`](./outbound/classify-replies) | Reply classification | Categorizing replies by sentiment (interested, bounced, etc.) |
| [`send-campaign`](./outbound/send-campaign) | Campaign sending | Sending cold outreach to audience segments |
| [`follow-up`](./outbound/follow-up) | Follow-up sequences | Auto follow-up unreplied prospects |
| [`clean-bounces`](./outbound/clean-bounces) | Bounce cleanup | Removing bounced/complained contacts from audiences |
| [`analytics`](./outbound/analytics) | Campaign analytics | Generating performance reports and metrics |

## Deploy the MCP Server

The outbound-tools server must be running before using any sub-skills. Deploy to Railway:

```bash
# 1. Install Railway CLI
npm i -g @railway/cli

# 2. Authenticate
railway login
railway whoami

# 3. Initialize project (run from the outbound-tools directory)
railway init

# 4. Set required environment variables
railway variables set MAILPOOL_API_KEY=<your-mailpool-api-key>

# 5. Deploy
railway up

# 6. Get your public URL
railway domain
```

Store the Railway URL — you'll configure it as your MCP server endpoint.

### Configure MCP Server

Point your agent to the deployed server URL. The server exposes 12 MCP tools for email operations (send, read, tag, audiences, metrics, thread matching).

## Quick Routing

**Need to send a single email or manage contacts?** → `outbound`

**Running a cold email campaign to a list?** → `send-campaign`

**Replies came in and need classification?** → `classify-replies`

**Want to follow up with non-responders?** → `follow-up`

**Bounce rate too high or need list cleanup?** → `clean-bounces`

**Need a performance report?** → `analytics`
