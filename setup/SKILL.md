---
name: setup
description: Interactive setup wizard for deploying MCP server dependencies and configuring API keys for GTM skills.
---

# Setup Wizard

Interactive, table-driven setup for GTM skills. Guides the user through selecting skill groups, deploying dependencies, and configuring MCP servers.

## Dependency Registry

One row per MCP server dependency. This table is the source of truth for what needs to be deployed.

| ID | Name | Required By | Type | Endpoint |
|----|------|-------------|------|----------|
| `outbound-tools` | Outbound Tools | outbound, classify-replies, send-campaign, follow-up, clean-bounces, analytics | `railway-template` | User's Railway URL |
| `signals-tools` | Signals Tools | signals, detect-all, reputation, social-growth, hiring | `hosted-api-key` | `https://gtm-engine.sh` |

## Env Vars per Dependency

| Dependency ID | Var Name | Source | Required |
|---------------|----------|--------|----------|
| `outbound-tools` | `MAILPOOL_API_KEY` | Ask user | Yes |
| `outbound-tools` | `API_KEY` | Auto-generate (random 32-char hex) | Yes |
| `outbound-tools` | `ANTHROPIC_API_KEY` | Ask user | No |
| `signals-tools` | `API_KEY` | Call `get_api_key` tool or ask user | Yes |

## Skill Group Mapping

| Skill Group | Dependency IDs |
|-------------|---------------|
| Outbound | `outbound-tools` |
| Signals | `signals-tools` |

## Dependency Type Reference

| Type | Description | Procedure |
|------|-------------|-----------|
| `railway-template` | Self-hosted on Railway via one-click template. Requires Railway CLI for deployment. | See **Railway Template Procedure** below. |
| `hosted-api-key` | Hosted API that requires an API key for access. No deployment needed. | See **Hosted API Key Procedure** below. |

---

## Flow

Execute these 5 steps in order. Use the tables above to drive each step generically — do not hard-code dependency-specific logic.

### Step 1: Ask Which Skill Groups

Present the user with a choice:
- **Outbound** — cold email workflows (campaigns, replies, follow-ups, bounces, analytics)
- **Signals** — buying signal detection (reviews, social growth, hiring)
- **Both** — full GTM stack

### Step 2: Resolve Dependencies

Look up the selected skill group(s) in the **Skill Group Mapping** table. Collect the unique set of dependency IDs that need to be deployed.

### Step 3: Deploy Each Dependency

Iterate over the collected dependency IDs. For each one, look up its **Type** in the Dependency Registry and follow the matching procedure.

#### Railway Template Procedure

1. **Check Railway CLI**: Run `railway version`. If not installed, guide the user to install it (`brew install railway` on macOS, or `npm i -g @railway/cli`).
2. **Authenticate**: Run `railway whoami`. If not logged in, run `railway login` and wait for auth to complete.
3. **List workspaces**: Run `railway whoami` to see the current workspace. If the user has multiple workspaces, ask which one to use.
4. **Deploy from template**: Run `railway init --template outbound-tools` to create the project from the template directly via CLI.
5. **Set environment variables**: Iterate over the **Env Vars per Dependency** table for this dependency ID:
   - `Ask user` → prompt the user for the value
   - `Auto-generate` → generate a random 32-character hex string
   - `Ask user` with Required = No → ask, but allow skipping
6. **Get the domain**: Run `railway domain` to retrieve the public URL. Store it for Step 4.

#### Hosted API Key Procedure

1. **Connect to the MCP server**: Add a temporary MCP server entry for the dependency's endpoint.
2. **Get API key**: Call the `get_api_key` tool on the server. If the tool is unavailable or the user already has a key, ask them to provide it.
3. **Store the key** for use in Step 4.

### Step 4: Configure MCP Servers

For each deployed dependency:

1. Read the user's Claude MCP config (e.g., `~/.claude.json` or the project's `.mcp.json`).
2. Add or update the MCP server entry:
   - **URL**: the endpoint from Step 3 (Railway URL or hosted endpoint)
   - **Headers**: `Authorization: Bearer <API_KEY>` using the key from Step 3
3. Write the config back.

### Step 5: Verify

Smoke-test each configured server:

| Dependency ID | Test Action | Expected |
|---------------|-------------|----------|
| `outbound-tools` | Call `list_email_accounts` | Returns a list (even if empty) |
| `signals-tools` | Call `detect_signal` with a test domain | Returns signal data |

Present a summary table to the user:

| Dependency | Status | Endpoint |
|------------|--------|----------|
| Outbound Tools | OK / Failed | `https://...` |
| Signals Tools | OK / Failed | `https://...` |

---

## Extensibility

To add a new dependency:

1. **Add a row** to the **Dependency Registry** table with a unique ID, name, the skills that require it, its type, and endpoint.
2. **Add rows** to the **Env Vars per Dependency** table for any environment variables it needs.
3. **Add a row** to the **Skill Group Mapping** table (or create a new skill group).
4. **If the type is new**, add a row to the **Dependency Type Reference** table and write a new procedure section below Step 3.

No changes to the flow logic are needed — the 5-step process iterates over the tables generically.
