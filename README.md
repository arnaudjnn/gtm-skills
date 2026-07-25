<img width="432" height="187" alt="Frame 5 (1)" src="https://github.com/user-attachments/assets/0dcad243-9abd-40ed-998d-04b20fdefd06" />

# GTM Skills

[![Install with npx skills](https://img.shields.io/badge/npx_skills-add_arnaudjnn/gtm--skills-blue?logo=npm)](https://skills.sh) [![Agent Skills](https://img.shields.io/badge/agent_skills-SKILL.md-8A2BE2)](https://skills.sh) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)

A collection of skills for AI coding agents following the [Agent Skills](https://skills.sh) format. These skills teach an agent how to run GTM workflows via Bash + curl — signal-keyed B2B outbound, LinkedIn copywriting, and Reddit community engagement — against the [gtm-tools.sh](https://gtm-tools.sh) API.

## Installation

Install the whole bundle, or any single skill by name:

```bash
# Install the catalog
npx skills add arnaudjnn/gtm-skills

# Install a single skill
npx skills add arnaudjnn/gtm-skills --skill gtm-tools
npx skills add arnaudjnn/gtm-skills --skill outbound-sales
npx skills add arnaudjnn/gtm-skills --skill linkedin-copywriter
npx skills add arnaudjnn/gtm-skills --skill reddit-community-manager
```

Or via Claude Code's plugin marketplace (uses `.claude-plugin/marketplace.json`):

```
/plugin marketplace add arnaudjnn/gtm-skills
/plugin install gtm-skills@gtm-skills
```

## Available Skills

| Skill | Description | Source |
|---|---|---|
| [`gtm-tools`](./skills/gtm-tools) | **Start here.** Base capability skill: get an API key (including autonomous `auth.md` self-registration), connect the browser-session pool, then call all 63 LinkedIn / Reddit / email / signal / geocoding tools. Carries the full tool catalog with token costs, the metering rules (what a failed call costs), and the error contract. | `./skills/gtm-tools` |
| [`outbound-sales`](./skills/outbound-sales) | Signal-keyed B2B outbound. Run buying-intent signals on target domains, map each firing signal to a message angle + title filter, find the decision-maker, verify the email, and draft both a cold email AND a LinkedIn DM keyed to the signal. Returns structured drafts in a tool-agnostic format ready to paste into Apollo / Outreach / Salesloft / Lemlist / Smartlead / your CRM. **Does not send** — handoff only. | `./skills/outbound-sales` |
| [`linkedin-copywriter`](./skills/linkedin-copywriter) | Ghostwrite LinkedIn posts, comments, connection-request notes, and DMs in the user's voice without AI tells. Voice ingestion + anti-AI scrub + draft-then-paste, with four modes (post / comment / invitation / dm) and char-limit + banned-word guards. | `./skills/linkedin-copywriter` |
| [`reddit-community-manager`](./skills/reddit-community-manager) | Reddit community engagement for B2B. Discover relevant threads, qualify posters, draft and post comments that don't read as AI spam, send DMs, follow up. 21 Reddit tools wired into a five-stage loop with the should-I-reply gate, disclosure pattern, no-links rule, and anti-AI-writing scrubber. | `./skills/reddit-community-manager` |

## Usage

Skills auto-activate when relevant prompts come in. Examples:

- "Do outbound to these 20 domains" → `outbound-sales`
- "Find hot CX accounts this week and draft outreach" → `outbound-sales`
- "Scan acme.com for buying signals and draft messages" → `outbound-sales`
- "Build me a prospect list with signal-keyed messages" → `outbound-sales`
- "Write a LinkedIn post about <topic>" → `linkedin-copywriter`
- "Comment on this LinkedIn post" → `linkedin-copywriter`
- "Draft a connection request to <person>" → `linkedin-copywriter`
- "Find Reddit threads about <pain point>" → `reddit-community-manager`
- "Should I reply to this Reddit post?" → `reddit-community-manager`
- "DM this Reddit user" → `reddit-community-manager`
- "Set up gtm-tools" / "how do I get an API key?" / "what's my token balance?" → `gtm-tools`

## Prerequisites

- `GTM_TOOLS_API_KEY` set in your environment. Obtain via `get_api_key` at https://gtm-tools.sh or `gtm-tools admin login` from the [`gtm-tools` CLI](https://gtm-tools.sh/documentation/integrations/cli).
- The [GTM Tools browser extension](https://gtm-tools.sh/extension) connected if you'll use LinkedIn write tools or Reddit write tools (both need a pooled browser session). `outbound-sales`, `linkedin-copywriter`, and `reddit-community-manager` all rely on this for the relevant write paths.

## License

MIT
