<img width="432" height="187" alt="Frame 5 (1)" src="https://github.com/user-attachments/assets/0dcad243-9abd-40ed-998d-04b20fdefd06" />

# GTM Skills

[![Install with npx skills](https://img.shields.io/badge/npx_skills-add_arnaudjnn/gtm--skills-blue?logo=npm)](https://skills.sh) [![Agent Skills](https://img.shields.io/badge/agent_skills-SKILL.md-8A2BE2)](https://skills.sh) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

A collection of skills for AI coding agents following the [Agent Skills](https://skills.sh) format. These skills teach an agent how to run GTM workflows via Bash + curl — outbound email, buying-intent signal detection, LinkedIn intelligence, and Reddit community engagement — against the [gtm-tools.sh](https://gtm-tools.sh) API.

## Installation

Install the whole bundle, or any single skill by name:

```bash
# Install the catalog
npx skills add arnaudjnn/gtm-skills

# Install a single skill
npx skills add arnaudjnn/gtm-skills --skill reddit-community-manager
npx skills add arnaudjnn/gtm-skills --skill outbound
npx skills add arnaudjnn/gtm-skills --skill signals
npx skills add arnaudjnn/gtm-skills --skill linkedin-copywriter
```

Or via Claude Code's plugin marketplace (uses `.claude-plugin/marketplace.json`):

```
/plugin marketplace add arnaudjnn/gtm-skills
/plugin install gtm-skills@gtm-tools-skills
```

## Available Skills

| Skill | Description | Source |
|---|---|---|
| [`setup`](./skills/setup) | Interactive setup wizard. First-time setup, deploying servers, configuring API keys. | `./skills/setup` |
| [`outbound`](./skills/outbound) | Outbound email workflows. Send campaigns, classify replies, follow up with non-responders, clean bounces, generate analytics. Powered by the outbound-tools server (IMAP/SMTP + Mailpool). | `./skills/outbound` |
| [`signals`](./skills/signals) | Buying-intent signal detection. Scan domains for Trustpilot sentiment, social-media follower spikes, and LinkedIn hiring activity. Powered by gtm-tools.sh. | `./skills/signals` |
| [`linkedin-copywriter`](./skills/linkedin-copywriter) | Ghostwrite LinkedIn posts, comments, connection-request notes, and DMs in the user's voice without AI tells. Voice ingestion + anti-AI scrub + draft-then-paste, with four modes (post / comment / invitation / dm) and char-limit + banned-word guards. | `./skills/linkedin-copywriter` |
| [`reddit-community-manager`](./skills/reddit-community-manager) | Reddit community engagement for B2B. Discover relevant threads, qualify posters, draft and post comments that don't read as AI spam, send DMs, follow up. 21 Reddit tools wired into a five-stage loop with the should-I-reply gate, disclosure pattern, no-links rule, and anti-AI-writing scrubber. | `./skills/reddit-community-manager` |

## Usage

Skills auto-activate when relevant prompts come in. Examples:

- "Send a cold email campaign to our leads segment" → `outbound`
- "Classify the replies that came in today" → `outbound`
- "Scan acme.com for buying signals" → `signals`
- "What's the Trustpilot sentiment for gymshark.com?" → `signals`
- "Find a profile on LinkedIn for Justin Mares at Kettle & Fire" → `linkedin-copywriter`
- "Send a LinkedIn invitation to ..." → `linkedin-copywriter`
- "Find Reddit threads about <pain point>" → `reddit-community-manager`
- "Should I reply to this Reddit post?" → `reddit-community-manager`
- "DM this Reddit user" → `reddit-community-manager`

## Prerequisites

- `GTM_TOOLS_API_KEY` set in your environment. Obtain via `get_api_key` at https://gtm-tools.sh or `gtm-tools admin login` from the [`gtm-tools` CLI](https://gtm-tools.sh/documentation/integrations/cli).
- The [GTM Tools browser extension](https://gtm-tools.sh/extension) connected if you'll use LinkedIn write tools or Reddit write tools (both need a pooled browser session).
- For outbound: the outbound-tools server deployed on Railway + `MAILPOOL_API_KEY` set. See [`./skills/setup`](./skills/setup).

## License

MIT
