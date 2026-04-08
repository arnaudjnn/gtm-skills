---
name: socials
description: LinkedIn social intelligence via API calls. Look up profiles, browse posts, list jobs, map company employees, and send direct messages. Use sub-skills for specific tasks or browse the tools list below.
---

# LinkedIn Social Intelligence

Research LinkedIn profiles, posts, jobs, and companies, plus send direct messages. All operations use the socials-tools API via **Bash** (`https://socials.gtm-engine.sh/api/v0`).

## LinkedIn Account Setup

Messaging tools (`send_linkedin_message`, `send_linkedin_invitation`, `list_linkedin_conversations`, `list_linkedin_saved_posts`, `list_user_posts`) require a connected LinkedIn account. Two setup tools (0 tokens each) manage this:

- `connect_linkedin`:returns a curl command to run on the user's Mac. It extracts the LinkedIn cookie from Chrome/Brave/Edge/Comet and sets up hourly auto-refresh.
- `list_connected_linkedin_accounts`:lists all connected accounts with username, name, and last sync time.

All other tools (profiles, companies, posts, jobs) work without a connected account.

## How to Call Tools

Use the Bash tool to call the API. See `references/tools-reference.md` for the exact command for each tool.

**Pattern:**
```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/TOOL_NAME" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{...arguments...}' | jq .
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
- `get_linkedin_company_url`:find a company's LinkedIn page from its domain
- `get_linkedin_profile_url`:find a person's LinkedIn profile from name + company domain

### Posts (2-80 tokens)
- `get_linkedin_post`:content and social counts for a single post. 2 tokens.
- `list_user_posts`:recent posts from a LinkedIn user. 5 tokens.
- `list_linkedin_saved_posts`:a user's saved posts with content. 10 tokens.
- `list_linkedin_post_reactions`:who reacted to a post. 5 tokens.
- `list_linkedin_post_comments`:comments on a post. 5 tokens.
- `list_linkedin_company_posts`:posts from a company's LinkedIn page with engagement metrics. 5 tokens.
- `list_linkedin_company_employees_posts`:recent posts from a company's employees. 80 tokens.

### Jobs (2-5 tokens)
- `get_linkedin_job`:full job description. 2 tokens.
- `list_linkedin_jobs`:all job openings for a company, with optional title filter. 5 tokens.

### Companies (2-30 tokens)
- `get_linkedin_company`:full company data from a domain (industry, size, HQ, followers, funding). 2 tokens.
- `list_linkedin_company_employees`:paginated employee directory with name, headline, profile URL. 30 tokens.
- `search_linkedin_company_contacts`:find decision makers by title keywords (CEO, CTO, VP, etc.). 30 tokens.

### Messaging (5 tokens)
- `send_linkedin_message`:send a message from a connected account. 5 tokens.
- `send_linkedin_invitation`:send a connection request with optional message. 5 tokens.
- `list_linkedin_conversations`:list 25 most recent conversations with up to 100 messages each. 5 tokens.

## Sub-Skills

| Skill | Use When |
|-------|----------|
| `profiles` | Looking up a person's LinkedIn profile or finding a company's LinkedIn page |
| `posts` | Researching post content, engagement, or what a company's employees are posting |
| `jobs` | Checking job listings or hiring activity for a company |
| `companies` | Looking up company data, browsing employees, or finding decision makers |
| `messaging` | Sending or reading LinkedIn messages |

## Quick Routing

**Need someone's LinkedIn profile or background?** -> `profiles`

**Researching posts, engagement, or content activity?** -> `posts`

**Checking job listings or hiring signals?** -> `jobs`

**Looking up a company or finding decision makers?** -> `companies`

**Sending or reading LinkedIn messages?** -> `messaging`
