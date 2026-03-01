---
name: gtm-skills
description: GTM skills using MCP servers. Outbound email workflows (campaigns, reply classification, follow-ups, bounce cleanup, analytics) and buying signal detection (Trustpilot reviews, social media spikes, LinkedIn hiring).
metadata:
  author: arnaudjeannin
  version: "1.0.0"
  homepage: https://gtm-engine.sh
inputs:
  - name: MAILPOOL_API_KEY
    description: Mailpool API key for outbound email operations. Required for sending emails, managing audiences, and campaign workflows.
    required: false
  - name: ANTHROPIC_API_KEY
    description: Anthropic API key to enable auto-classification of replies via the outbound-tools server.
    required: false
  - name: SIGNALS_API_KEY
    description: API key for the signals-tools MCP server at gtm-engine.sh. Obtain one by running the setup sub-skill or calling the get_api_key tool.
    required: false
---

# GTM Skills

## Overview

Outbound email workflows and buying signal detection, powered by MCP servers. Covers the full cold outreach lifecycle — sending campaigns, classifying replies, following up, cleaning bounces, and reporting analytics — plus buying signal detection across Trustpilot reviews, social media growth, and LinkedIn hiring activity.

## Sub-Skills

| Skill                    | Description              | Use When                                                                                 |
| ------------------------ | ------------------------ | ---------------------------------------------------------------------------------------- |
| [`setup`](./setup)       | Interactive setup wizard | First-time setup, deploying MCP servers, configuring API keys                            |
| [`outbound`](./outbound) | Outbound email workflows | Sending emails, campaigns, reply classification, follow-ups, bounce cleanup, analytics   |
| [`signals`](./signals)   | Buying signal detection  | Scanning domains for buying signals (Trustpilot reviews, social spikes, LinkedIn hiring) |

## Quick Routing

**First time using GTM skills?** → `setup`

**Anything related to cold email?** → `outbound`

**Looking for buying signals on a domain?** → `signals`

## Common Setup

Run the **setup** sub-skill to get started. It walks you through selecting skill groups (Outbound, Signals, or Both), deploying the required MCP servers, configuring API keys, and verifying everything works.

Just ask: "Set up GTM skills." or run `/setup`.

## Resources

- [mailpool.io](https://mailpool.io) — Email infrastructure for outbound workflows
- [outbound-tools](https://github.com/arnaudjnn/outbound-tools) — MCP server for outbound email operations (self-hosted on Railway)
- [gtm-engine.sh](https://gtm-engine.sh) — Hosted MCP server for buying signal detection
