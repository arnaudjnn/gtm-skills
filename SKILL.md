---
name: gtm-skills
description: GTM skills using Bash. Outbound email workflows (campaigns, reply classification, follow-ups, bounce cleanup, analytics) and buying signal detection (Trustpilot reviews, social media spikes, LinkedIn hiring).
metadata:
  author: arnaudjeannin
  version: "1.0.0"
  homepage: https://gtm-engine.sh
inputs:
  - name: OUTBOUND_TOOLS_URL
    description: Endpoint URL for the outbound-tools server (e.g. https://your-app.railway.app/mcp). Required for outbound email operations.
    required: false
  - name: OUTBOUND_API_KEY
    description: API key for authenticating with the outbound-tools server. Required for outbound email operations.
    required: false
  - name: GTM_ENGINE_API_KEY
    description: API key for the GTM Engine signals server at signals.gtm-engine.sh. Obtain one via the get_api_key tool (see signals sub-skill).
    required: false
  - name: MAILPOOL_API_KEY
    description: Mailpool API key for deploying the outbound-tools server on Railway. Only needed during setup.
    required: false
  - name: ANTHROPIC_API_KEY
    description: Anthropic API key to enable auto-classification of replies via the outbound-tools server. Only needed during setup.
    required: false
---

# GTM Skills

## Overview

Outbound email workflows and buying signal detection, powered by Bash. Covers the full cold outreach lifecycle (sending campaigns, classifying replies, following up, cleaning bounces, reporting analytics) plus buying signal detection across Trustpilot reviews, social media growth, and LinkedIn hiring activity.

All tools are called in the Bash tool. Just set the endpoint URL and API key as environment variables.

## Sub-Skills

| Skill                    | Description              | Use When                                                                                 |
| ------------------------ | ------------------------ | ---------------------------------------------------------------------------------------- |
| [`setup`](./setup)       | Interactive setup wizard | First-time setup, deploying servers, configuring API keys                                |
| [`outbound`](./outbound) | Outbound email workflows | Sending emails, campaigns, reply classification, follow-ups, bounce cleanup, analytics   |
| [`signals`](./signals)   | Buying signal detection  | Scanning domains for buying signals (Trustpilot reviews, social spikes, LinkedIn hiring) |

## Quick Routing

**First time using GTM skills?** → `setup`

**Anything related to cold email?** → `outbound`

**Looking for buying signals on a domain?** → `signals`

## Common Setup

Run the **setup** sub-skill to get started. It walks you through selecting skill groups (Outbound, Signals, or Both), deploying the required servers, configuring API keys as environment variables, and verifying everything works.

Just ask: "Set up GTM skills." or run `/setup`.

## Resources

- [mailpool.io](https://mailpool.io): Email infrastructure for outbound workflows
- [outbound-tools](https://github.com/arnaudjnn/outbound-tools): Outbound email server (self-hosted on Railway)
- [gtm-engine.sh](https://gtm-engine.sh): Hosted server for buying signal detection
