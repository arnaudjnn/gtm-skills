<img width="432" height="187" alt="Frame 5 (1)" src="https://github.com/user-attachments/assets/0dcad243-9abd-40ed-998d-04b20fdefd06" />

# GTM Skills

A collection of skills for AI coding agents following the Agent Skills format. These skills enable AI agents to run GTM outbound email workflows using an MCP server (IMAP/SMTP + Mailpool).

## Available Skills

### [`outbound`](./outbound)
Core outbound email operations. Send emails, read inbox/sent, tag messages, and manage audience segments via MCP tools. Use for cold email outreach and contact management.

### [`classify-replies`](./outbound/classify-replies)
Classify received reply emails by sentiment and tag them. Categories: interested, complained, out-of-office, unsubscribed, bounced. Tags both reply and original sent email.

### [`send-campaign`](./outbound/send-campaign)
Send outbound email campaigns to audience segments. Handles account rotation, rate limiting, and deliverability best practices.

### [`follow-up`](./outbound/follow-up)
Send follow-up emails to prospects who haven't replied. Identifies unreplied sent emails and sends contextual follow-ups for multi-touch outreach sequences.

### [`clean-bounces`](./outbound/clean-bounces)
Remove bounced and complained contacts from audience segments. Keeps lists clean and protects sender reputation.

### [`analytics`](./outbound/analytics)
Generate email campaign analytics and performance reports. Track bounce, complaint, and interest rates across accounts.

## Installation

```bash
npx skills add gtm-skills
```

## Usage

Skills are automatically activated when relevant tasks are detected. Example prompts:

- "Send a cold email campaign to our leads segment"
- "Classify the replies that came in today"
- "Follow up with prospects who haven't replied in 5 days"
- "Clean bounced contacts from all audiences"
- "Generate a campaign performance report"

## Prerequisites

- An MCP server (outbound-tools) deployed on Railway
- Mailpool API key stored as `MAILPOOL_API_KEY` environment variable

See the root [SKILL.md](./SKILL.md) for Railway deployment instructions.

## License

MIT
