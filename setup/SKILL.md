---
name: setup
description: Interactive setup wizard for deploying server dependencies and configuring environment variables for GTM skills.
---

# Setup Wizard

Interactive, table-driven setup for GTM skills. Guides the user through selecting skill groups, deploying dependencies, and configuring environment variables for API calls.

## Dependency Registry

One row per server dependency. This table is the source of truth for what needs to be deployed.

| ID | Name | Required By | Type | Endpoint |
|----|------|-------------|------|----------|
| `outbound-tools` | Outbound Tools | outbound, classify-replies, send-campaign, follow-up, clean-bounces, analytics | `railway-template` | User's Railway URL |
| `signals-tools` | Signals Tools | signals, detect-all, reputation, social-growth, hiring | `hosted-api-key` | `https://signals.gtm-engine.sh/mcp` |

## Env Vars per Dependency

| Dependency ID | Var Name | Source | Required |
|---------------|----------|--------|----------|
| `outbound-tools` | `MAILPOOL_API_KEY` | Ask user | Yes |
| `outbound-tools` | `API_KEY` | Auto-generate (random 32-char hex) | Yes |
| `outbound-tools` | `ANTHROPIC_API_KEY` | Ask user | No |
| `signals-tools` | `GTM_ENGINE_API_KEY` | Ask user | Yes |

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

Execute these 5 steps in order. Use the tables above to drive each step generically:do not hard-code dependency-specific logic.

### Step 1: Ask Which Skill Groups

Present the user with a choice:
- **Outbound**:cold email workflows (campaigns, replies, follow-ups, bounces, analytics)
- **Signals**:buying signal detection (reviews, social growth, hiring)
- **Both**:full GTM stack

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

1. **Ask the user for their email**: Prompt them for the email to register with.
2. **Request a verification code** via the `get_api_key` tool:
   ```bash
   curl -s -X POST "https://signals.gtm-engine.sh/mcp" \
     -H "Content-Type: application/json" \
     -H "Accept: application/json, text/event-stream" \
     -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"get_api_key","arguments":{"email":"USER_EMAIL"}},"id":1}'
   ```
3. **Ask the user for the 6-digit code** they received by email.
4. **Submit the code** to get the API key:
   ```bash
   curl -s -X POST "https://signals.gtm-engine.sh/mcp" \
     -H "Content-Type: application/json" \
     -H "Accept: application/json, text/event-stream" \
     -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"get_api_key","arguments":{"email":"USER_EMAIL","code":"CODE"}},"id":1}'
   ```
5. **Store the key** for use in Step 4.

### Step 4: Configure Environment Variables

For each deployed dependency, export the required environment variables so they persist for tool calls.

**For outbound-tools:**
```bash
export OUTBOUND_TOOLS_URL="https://<railway-domain>/mcp"
export OUTBOUND_API_KEY="<the API_KEY from Step 3>"
```

**For signals-tools:**
```bash
export GTM_ENGINE_API_KEY="<the key from Step 3>"
```

Write these exports to the user's shell profile (`~/.zshrc`, `~/.bashrc`, or `~/.profile`) so they persist across sessions. Ask the user which file to use, or detect their shell.

Also add them to the project's `.env` file if one exists.

### Step 5: Verify

Smoke-test each configured server:

**For outbound-tools:**
```bash
curl -s -X POST "$OUTBOUND_TOOLS_URL" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OUTBOUND_API_KEY" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"list_email_accounts","arguments":{}},"id":1}' \
  | jq -r '.result.content[0].text'
```

**For signals-tools:**
```bash
curl -s -X POST "https://signals.gtm-engine.sh/mcp" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"detect_signal","arguments":{"domain":"gymshark.com"}},"id":1}' \
  | jq -r '.result.content[0].text'
```

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

No changes to the flow logic are needed:the 5-step process iterates over the tables generically.
