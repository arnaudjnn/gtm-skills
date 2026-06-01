---
name: profiles
description: Look up LinkedIn profiles and find company LinkedIn URLs from domains.
---

# LinkedIn Profiles

Look up structured LinkedIn profile data or find a company's LinkedIn page URL from its domain. Use this for prospect research, background checks, and enrichment.

## Tools Used

- `get_linkedin_profile` (2 tokens): structured profile with experiences, education, skills, and headline
- `get_linkedin_company_url` (2 tokens): find a company's LinkedIn page URL from a domain
- `get_linkedin_profile_url` (5 tokens): find a person's LinkedIn profile URL from name + company domain

See `references/tools-reference.md` for exact commands.

## Workflow

1. **Look up profile or find company URL**
   - For a person by username: call `get_linkedin_profile` with the LinkedIn username
   - For a person by name: call `get_linkedin_profile_url` with name + company domain to find their URL, then `get_linkedin_profile`
   - For a company domain: call `get_linkedin_company_url` with the domain

2. **Extract key information**
   - Person: current role, company, tenure, past experience, education, headline
   - Company: LinkedIn page URL for use with other socials sub-skills

3. **Output a structured summary**
   - For profiles: name, headline, current role, notable past experience, education
   - For company lookups: the LinkedIn URL and confirmation of the match

## Use Cases

- "Get the LinkedIn profile for rauchg"
- "Find the LinkedIn page for vercel.com"
- "Find John Doe who works at Stripe on LinkedIn"
- "What's the background of this prospect?"
