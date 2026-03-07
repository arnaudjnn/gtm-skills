---
name: profiles
description: Look up LinkedIn profiles and find company LinkedIn URLs from domains.
---

# LinkedIn Profiles

Look up structured LinkedIn profile data or find a company's LinkedIn page URL from its domain. Use this for prospect research, background checks, and enrichment.

## Tools Used

- `get_linkedin_profile` (2 tokens): structured profile with experiences, education, skills, and headline
- `find_linkedin_url` (2 tokens): find a company's LinkedIn page URL from a domain

See `references/tools-reference.md` for exact commands.

## Workflow

1. **Look up profile or find company URL**
   - For a person: call `get_linkedin_profile` with the LinkedIn username
   - For a company domain: call `find_linkedin_url` with the domain

2. **Extract key information**
   - Person: current role, company, tenure, past experience, education, headline
   - Company: LinkedIn page URL for use with other socials sub-skills

3. **Output a structured summary**
   - For profiles: name, headline, current role, notable past experience, education
   - For company lookups: the LinkedIn URL and confirmation of the match

## Use Cases

- "Get the LinkedIn profile for rauchg"
- "Find the LinkedIn page for vercel.com"
- "What's the background of this prospect?"
- "Look up the current role and experience for johndoe on LinkedIn"
