<img width="432" height="187" alt="Frame 5 (1)" src="https://github.com/user-attachments/assets/0dcad243-9abd-40ed-998d-04b20fdefd06" />

# GTM Skills

[![Install with npx skills](https://img.shields.io/badge/npx_skills-add_arnaudjnn/gtm--skills-blue?logo=npm)](https://skills.sh) [![Agent Skills](https://img.shields.io/badge/agent_skills-SKILL.md-8A2BE2)](https://skills.sh) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

A collection of skills for AI coding agents following the Agent Skills format. These skills enable AI agents to run GTM workflows using MCP servers — outbound email operations and buying signal detection.

## Available Skills

### [`outbound`](./outbound)
Outbound email workflows. Send campaigns, classify replies, follow up with non-responders, clean bounced contacts, and generate analytics reports. Powered by the outbound-tools MCP server (IMAP/SMTP + Mailpool).

### [`signals`](./signals)
Buying signal detection. Scan company domains for Trustpilot review sentiment, social media follower spikes (Instagram/TikTok), and LinkedIn hiring activity. Powered by the signals-tools MCP server (gtm-engine.sh).

## Installation

```bash
npx skills add arnaudjnn/gtm-skills
```

## Usage

Skills are automatically activated when relevant tasks are detected. Example prompts:

- "Send a cold email campaign to our leads segment"
- "Classify the replies that came in today"
- "Follow up with prospects who haven't replied in 5 days"
- "Clean bounced contacts from all audiences"
- "Generate a campaign performance report"
- "Scan acme.com for buying signals"
- "What's the Trustpilot sentiment for gymshark.com?"
- "Is notion.so hiring CX reps?"
- "Check Instagram growth for newbrand.com"

## Prerequisites

- An MCP server (outbound-tools) deployed on Railway for outbound skills
- Mailpool API key stored as `MAILPOOL_API_KEY` environment variable
- An API key for gtm-engine.sh for signal skills — see the [signals skill](./signals) for setup

See the root [SKILL.md](./SKILL.md) for deployment instructions.

## License

MIT
