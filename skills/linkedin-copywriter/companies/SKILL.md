---
name: companies
description: Browse LinkedIn company data, employee directories, and find decision makers.
---

# LinkedIn Companies

Look up company data, browse employee directories, and find key contacts by title. Use this for account research, org mapping, and prospecting.

## Tools Used

- `get_linkedin_company` (2 tokens): full company data from a domain (industry, size, HQ, followers, funding info, company_id)
- `list_linkedin_company_employees` (30 tokens): paginated employee list with name, headline, and profile URL. Params: `company` (URL or slug), `start` (offset), `count` (max 49).
- `search_linkedin_company_contacts` (30 tokens): find decision makers by title keywords. Params: `company_id`, `title_keywords`, `limit`.

See `references/tools-reference.md` for exact commands.

## Workflow

### Company Lookup
1. Call `get_linkedin_company` with the domain to get company data and `company_id`
2. Extract industry, employee count, HQ location, and funding info

### Find Decision Makers (recommended)
1. Call `get_linkedin_company` with the domain to get `company_id`
2. Call `search_linkedin_company_contacts` with `company_id` and title keywords (e.g. ["CEO", "CTO", "VP Engineering"])
3. Use `get_linkedin_profile` on interesting contacts for full background

### Browse All Employees
1. Call `list_linkedin_company_employees` with the company URL or slug
2. Paginate as needed using `start` and `count` params
3. Scan headlines for relevant titles

## Use Cases

- "Tell me about Stripe as a company"
- "Who are the VPs at Vercel?"
- "Find the CTO and Head of Sales at acme.com"
- "List the first 20 employees at notion"
- "How big is this company and where are they based?"
