# Socials Tools API Reference

Complete reference for all 20 tools provided by the socials-tools server.

---

## How to Call Tools

Use the Bash tool to run curl commands. Every call follows this pattern:

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/TOOL_NAME" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{...arguments...}'
```

- The API base URL is `https://socials.gtm-engine.sh/api/v0`
- `$GTM_ENGINE_API_KEY`: the API key for authentication (see the socials SKILL.md for how to obtain one)
- The response is direct JSON (parse with `jq .`)

### Tip: parse responses with jq

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/get_linkedin_profile" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"username":"rauchg"}' | jq .
```

---

## ping

Health check for the API server.

**Cost:** 0 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/ping" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{}'
```

**Parameters:** None.

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| result | string | Always `"pong"` |
| timestamp | string | Current server timestamp |
| message | string | Always `"API server is healthy"` |

---

## connect_linkedin

Returns setup instructions with a curl command for Mac that extracts LinkedIn cookie from Chrome/Brave/Edge/Comet. Use this to connect a new LinkedIn account.

**Cost:** 0 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/connect_linkedin" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{}'
```

**Parameters:** None.

**Returns:** Setup instructions with a curl command to extract and register the LinkedIn cookie.

---

## list_connected_linkedin_accounts

Lists all LinkedIn accounts currently connected to the API.

**Cost:** 0 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/list_connected_linkedin_accounts" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{}'
```

**Parameters:** None.

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| status | string | `"ok"` |
| accounts | Account[] | Array of connected accounts |

### Account shape

| Field | Type | Description |
|-------|------|-------------|
| username | string | LinkedIn username |
| firstName | string | First name |
| lastName | string | Last name |
| updatedAt | string | Last updated timestamp |

---

## get_linkedin_company_url

Finds the LinkedIn company page URL for a given domain.

**Cost:** 2 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/get_linkedin_company_url" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"domain":"gymshark.com"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| domain | string | yes | The company domain to look up (e.g. "gymshark.com") |

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| domain | string | The queried domain |
| linkedin_url | string | LinkedIn company page URL |

---

## get_linkedin_profile_url

Finds a person's LinkedIn profile URL given their name and company domain.

**Cost:** 5 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/get_linkedin_profile_url" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"name":"Arnaud Jeannin","domain":"gorgias.com"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| name | string | yes | The person's full name (e.g. "Arnaud Jeannin") |
| domain | string | yes | The company domain to match against (e.g. "gorgias.com") |

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| name | string | The queried name |
| domain | string | The queried domain |
| linkedin_url | string | LinkedIn profile URL found |
| location | string | Person's location (if available from search results) |

---

## get_linkedin_profile

Retrieves a LinkedIn user's full profile including experiences and education.

**Cost:** 2 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/get_linkedin_profile" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"username":"rauchg"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| username | string | yes | LinkedIn username or profile URL (e.g. "rauchg") |

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| name | string | Full name |
| headline | string | Profile headline |
| location | string | Location |
| summary | string | Profile summary/about |
| experiences | Experience[] | Work experience entries |
| educations | Education[] | Education entries |

---

## get_linkedin_post

Retrieves a single LinkedIn post by URL or activity URN.

**Cost:** 2 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/get_linkedin_post" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"url":"urn:li:activity:7654321098765432100"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| url | string | yes | Post URL or activity URN (e.g. "urn:li:activity:..." or "https://www.linkedin.com/posts/...") |

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| activity_urn | string | Activity URN identifier |
| author_name | string | Post author's name |
| content | string | Post text content |
| published_at | string | Publication date (YYYY-MM-DD) |
| url | string | Post URL |
| total_reactions | number | Total reaction count |
| total_comments | number | Total comment count |

---

## get_linkedin_job

Retrieves a single LinkedIn job posting by ID.

**Cost:** 2 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/get_linkedin_job" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"job_id":"4356405688"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| job_id | string | yes | LinkedIn job ID (e.g. "4356405688") |

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| id | string | LinkedIn job ID |
| title | string | Job title |
| url | string | Job listing URL |
| location | string | Job location |
| posted_at | string | When the job was posted |
| content | string | Full job description |

---

## list_user_posts

Lists recent posts from a LinkedIn user.

**Cost:** 5 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/list_user_posts" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"username":"rauchg","limit":10}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| username | string | yes | LinkedIn username |
| limit | number | no | Number of posts to return (default: 10, max: 100) |

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| status | string | `"ok"` |
| posts | Post[] | Array of posts (same shape as `get_linkedin_post` return) |

---

## list_linkedin_jobs

Lists job postings for a company, optionally filtered by title keywords.

**Cost:** 5 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/list_linkedin_jobs" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{
    "domain":"gymshark.com",
    "filter":"(cx OR customer support) NOT junior"
  }'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| domain | string | yes | The company domain to look up (e.g. "gymshark.com") |
| filter | string | no | Filter for job titles with OR/NOT syntax (e.g. "(cx OR customer support) NOT junior") |

**Filter syntax:**
- `OR`: match any of the terms: `"cx OR customer support"`
- `NOT`: exclude matches: `"NOT junior"`
- Parentheses for grouping: `"(cx OR customer support) NOT junior"`
- Case-insensitive matching

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| domain | string | The queried domain |
| linkedin_url | string | LinkedIn company page URL |
| all_jobs_url | string | URL to all job listings |
| total_jobs | number | Total job listings for the company |
| filter | string | The filter used (if provided) |
| matched_jobs | number | Count of matching jobs (if filter provided) |
| jobs | LinkedInJob[] | Array of job objects |

### LinkedInJob shape

| Field | Type | Description |
|-------|------|-------------|
| id | string | LinkedIn job ID |
| title | string | Job title |
| url | string | Job listing URL |
| location | string | Job location |
| posted_at | string | When the job was posted |

---

## list_linkedin_company_employees

Lists employees of a LinkedIn company page with pagination.

**Cost:** 30 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/list_linkedin_company_employees" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"company":"vercel","start":0,"count":10}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| company | string | yes | LinkedIn company URL or slug (e.g. "vercel") |
| start | number | no | Pagination offset (default: 0) |
| count | number | no | Number of results per page (default: 10, max: 49) |

**Returns:** Paginated employee list with name, headline, and profile URL.

---

## list_linkedin_post_reactions

Lists reactions on a LinkedIn post with pagination.

**Cost:** 5 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/list_linkedin_post_reactions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"post":"urn:li:activity:7654321098765432100","start":0,"count":10}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| post | string | yes | Post URL or activity URN |
| start | number | no | Pagination offset (default: 0) |
| count | number | no | Number of results per page (default: 10, max: 49) |

**Returns:** Array of reactors with:
| Field | Type | Description |
|-------|------|-------------|
| name | string | Reactor's name |
| headline | string | Reactor's headline |
| reaction_type | string | Reaction type (LIKE, CELEBRATE, LOVE, INSIGHTFUL, FUNNY, SUPPORT) |
| profile_url | string | Reactor's LinkedIn profile URL |

---

## list_linkedin_post_comments

Lists comments on a LinkedIn post with pagination and sorting.

**Cost:** 5 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/list_linkedin_post_comments" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"post":"urn:li:activity:7654321098765432100","start":0,"count":10,"sort":"RELEVANCE"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| post | string | yes | Post URL or activity URN |
| start | number | no | Pagination offset (default: 0) |
| count | number | no | Number of results per page (default: 10, max: 49) |
| sort | string | no | Sort order: `RELEVANCE` (default) or `REVERSE_CHRONOLOGICAL` |

**Returns:** Array of comments with:
| Field | Type | Description |
|-------|------|-------------|
| author_name | string | Commenter's name |
| headline | string | Commenter's headline |
| content | string | Comment text |
| published_at | string | Comment date |
| reaction_count | number | Number of reactions on the comment |
| profile_url | string | Commenter's LinkedIn profile URL |

---

## send_linkedin_message

Sends a message on LinkedIn from a connected account.

**Cost:** 5 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/send_linkedin_message" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{
    "senderUsername":"myaccount",
    "recipientUsername":"rauchg",
    "message":"Hello! I wanted to reach out about..."
  }'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| senderUsername | string | yes | LinkedIn username of the connected sender account |
| recipientUsername | string | yes | LinkedIn username of the recipient |
| message | string | yes | Message content (1-8000 characters) |

**Returns:** Send result confirmation.

---

## send_linkedin_invitation

Sends a LinkedIn connection request (invitation) from a connected account, with an optional personal message.

**Cost:** 5 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/send_linkedin_invitation" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{
    "senderUsername":"myaccount",
    "recipientUsername":"rauchg",
    "message":"Happy to connect!"
  }'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| senderUsername | string | yes | LinkedIn username of the connected sender account |
| recipientUsername | string | yes | LinkedIn username of the person to invite |
| message | string | no | Optional personal note (max 300 characters) |

**Returns:** Invitation result confirmation.

---

## list_linkedin_conversations

Lists the 25 most recent LinkedIn conversations for a connected account. Returns conversations with participant info, last message preview, and up to 100 messages per conversation (both sent and received).

**Cost:** 5 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/list_linkedin_conversations" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"username":"myaccount"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| username | string | yes | Connected LinkedIn account username (inbox owner) |

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| status | string | "ok" or "error" |
| conversations | LinkedInConversation[] | Array of conversations |
| error | string | Error message (when status is "error") |

### LinkedInConversation shape

| Field | Type | Description |
|-------|------|-------------|
| conversationUrn | string | LinkedIn conversation URN identifier |
| participant | string | Display name of the other participant |
| participantProfileId | string | LinkedIn profile ID of the other participant |
| participantHeadline | string | Headline of the other participant |
| lastMessage | string | Text of the most recent message |
| lastMessageSender | string | Name of who sent the last message |
| lastMessageAt | string | ISO 8601 timestamp of the last message |
| messages | ConversationMessage[] | Full message history for the conversation |

### ConversationMessage shape

| Field | Type | Description |
|-------|------|-------------|
| sender | string | Display name of the message sender |
| senderProfileId | string | LinkedIn profile ID of the sender |
| body | string | Message text content |
| deliveredAt | string | ISO 8601 timestamp of when the message was delivered |

---

## list_linkedin_saved_posts

Lists saved posts for a connected LinkedIn account, enriched with full post data.

**Cost:** 10 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/list_linkedin_saved_posts" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"username":"myaccount","limit":10}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| username | string | yes | Connected LinkedIn username |
| limit | number | no | Number of saved posts to return (default: 10) |

**Returns:** Array of enriched saved posts with:
| Field | Type | Description |
|-------|------|-------------|
| content | string | Post text content |
| published_at | string | Publication date |
| total_reactions | number | Total reaction count |
| total_comments | number | Total comment count |

---

## list_linkedin_company_employees_posts

Lists recent posts from employees of a given company. Finds active employees and collects their recent posts.

**Cost:** 80 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/list_linkedin_company_employees_posts" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"company":"vercel","max_employees":5,"days_back":7}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| company | string | yes | LinkedIn company URL or slug (e.g. "vercel") |
| max_employees | number | no | Maximum employees to scan (default: 5, max: 20) |
| days_back | number | no | How many days back to look for posts (default: 7, max: 90) |

**Returns:** Employees who posted recently with their posts, including author info, post content, publication dates, and social counts (reactions, comments).

---

## get_linkedin_company

Gets structured company data from a domain name. Returns company name, description, industry, HQ location, employee count, follower count, LinkedIn URL, and funding info.

**Cost:** 2 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/get_linkedin_company" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"domain":"stripe.com"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| domain | string | yes | Company domain (e.g. "stripe.com", "vercel.com") |

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| company_id | string | LinkedIn company ID (use with `search_linkedin_company_contacts`) |
| company_name | string | Company name |
| description | string | Company description |
| domain | string | Company domain |
| industries | string[] | Industry list |
| employee_count | number | Number of employees |
| employee_range | string | Employee range (e.g. "501-1000") |
| follower_count | number | LinkedIn follower count |
| hq_city | string | HQ city |
| hq_region | string | HQ state/region |
| hq_country | string | HQ country code |
| linkedin_url | string | LinkedIn company page URL |
| specialties | string[] | Company specialties |
| year_founded | number | Year founded |

---

## list_linkedin_company_posts

Lists posts from a company's LinkedIn page with engagement metrics.

**Cost:** 5 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/list_linkedin_company_posts" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"company":"stripe","start":0,"sort":"recent"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| company | string | yes | LinkedIn company URL or slug (e.g. "stripe") |
| start | number | no | Pagination offset (0 for page 1, 50 for page 2, etc.) |
| sort | string | no | Sort order: `recent` (default) or `top` |

**Returns:** Array of posts with:
| Field | Type | Description |
|-------|------|-------------|
| text | string | Post content |
| posted | string | Publication date (YYYY-MM-DD HH:MM:SS) |
| post_url | string | Post URL |
| urn | string | Post activity URN |
| num_reactions | number | Total reactions |
| num_comments | number | Total comments |
| num_reposts | number | Total reposts |
| num_likes | number | Like count |
| num_empathy | number | Empathy reaction count |
| images | array | Post images |
| video | object | Post video (if any) |

---

## search_linkedin_company_contacts

Searches for key contacts (decision makers) at a company by title keywords. Use `get_linkedin_company` first to get the `company_id` from a domain.

**Cost:** 30 tokens

```bash
curl -s -X POST "https://socials.gtm-engine.sh/api/v0/search_linkedin_company_contacts" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{
    "company_id":"2135371",
    "title_keywords":["CEO","CTO","VP Engineering"],
    "limit":5
  }'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| company_id | string | yes | LinkedIn company ID (from `get_linkedin_company`) |
| title_keywords | string[] | yes | Title keywords to match (e.g. ["CEO", "CTO", "VP"]) |
| limit | number | no | Max contacts to return (default: 10, max: 25) |

**Returns:** Array of matching contacts with:
| Field | Type | Description |
|-------|------|-------------|
| full_name | string | Contact's full name |
| job_title | string | Current job title |
| company | string | Company name |
| location | string | Location |
| linkedin_url | string | LinkedIn profile URL |
| about | string | Profile about/summary |
