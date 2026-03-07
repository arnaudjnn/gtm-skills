---
name: socials
description: LinkedIn social intelligence via API calls. Look up profiles, browse posts, list jobs, map company employees, and send direct messages. Use sub-skills for specific tasks or browse the tools list below.
---

# LinkedIn Social Intelligence

Research LinkedIn profiles, posts, jobs, and companies, plus send direct messages. All operations use the socials-tools server via **Bash** (`https://socials.gtm-engine.sh/mcp`).

## LinkedIn Account Setup

Most tools require a connected LinkedIn account. Two setup tools (0 tokens each) manage this:

- `connect_linkedin`:returns a curl command to run on the user's Mac. It extracts the LinkedIn cookie from Chrome/Brave/Edge/Comet and sets up hourly auto-refresh.
- `list_connected_linkedin_accounts`:lists all connected accounts with username, name, and last sync time.

Run `connect_linkedin` first, then verify with `list_connected_linkedin_accounts`.

## How to Call Tools

Use the Bash tool to call the API. See `references/tools-reference.md` for the exact command for each tool.

**Pattern:**
```bash
curl -s -X POST "https://socials.gtm-engine.sh/mcp" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"TOOL_NAME","arguments":{...}},"id":1}' \
  | jq -r '.result.content[0].text' | jq .
```

Environment variable `GTM_ENGINE_API_KEY` must be set (see the **setup** skill or **Getting an API Key** in the signals skill).

## Available Tools

See `references/tools-reference.md` for full commands and parameter details on all tools.

### Setup (0 tokens)
- `ping`:health check
- `connect_linkedin`:get the setup command for connecting a LinkedIn account
- `list_connected_linkedin_accounts`:list connected LinkedIn accounts

### Profiles (2 tokens)
- `get_linkedin_profile`:structured profile data (experiences, education, headline)
- `find_linkedin_url`:find a company's LinkedIn page from its domain

### Posts (2-20 tokens)
- `get_linkedin_post`:content and social counts for a single post. 2 tokens.
- `list_user_posts`:recent posts from a LinkedIn user. 5 tokens.
- `list_linkedin_saved_posts`:a user's saved posts with content. 10 tokens.
- `list_linkedin_post_reactions`:who reacted to a post. 5 tokens.
- `list_linkedin_post_comments`:comments on a post. 5 tokens.
- `list_linkedin_company_employees_posts`:recent posts from a company's employees. 20 tokens.

### Jobs (2-5 tokens)
- `get_linkedin_job`:full job description. 2 tokens.
- `list_linkedin_jobs`:all job openings for a company, with optional title filter. 5 tokens.

### Companies (5 tokens)
- `list_linkedin_company_employees`:paginated employee directory with name, headline, profile URL. 5 tokens.

### Messaging (5 tokens)
- `send_linkedin_dm`:send a direct message from a connected account. 5 tokens.

## Sub-Skills

| Skill | Use When |
|-------|----------|
| `profiles` | Looking up a person's LinkedIn profile or finding a company's LinkedIn page |
| `posts` | Researching post content, engagement, or what a company's employees are posting |
| `jobs` | Checking job listings or hiring activity for a company |
| `companies` | Browsing a company's employee directory or finding specific roles |
| `messaging` | Sending a LinkedIn direct message |

## Quick Routing

**Need someone's LinkedIn profile or background?** -> `profiles`

**Researching posts, engagement, or content activity?** -> `posts`

**Checking job listings or hiring signals?** -> `jobs`

**Browsing a company's employees?** -> `companies`

**Sending a LinkedIn DM?** -> `messaging`
