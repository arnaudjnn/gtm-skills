---
name: companies
description: Browse LinkedIn company employee directories.
---

# LinkedIn Company Employees

Browse a company's employee directory on LinkedIn. Use this to find specific people by role, map out team structures, or identify decision-makers.

## Tools Used

- `list_linkedin_company_employees` (5 tokens): paginated employee list with name, headline, and profile URL. Params: `company` (URL or slug), `start` (offset), `count` (max 49).

See `references/tools-reference.md` for exact commands.

## Workflow

1. **List employees**
   - Call `list_linkedin_company_employees` with the company URL or slug
   - Paginate as needed using `start` and `count` params (max 49 per page)

2. **Filter by role or seniority**
   - Scan headlines for relevant titles (VP, Director, Manager, Engineer, etc.)
   - Narrow down to the function or level you need

3. **Deep-dive on individuals**
   - Use the `profiles` sub-skill to get full profile details for interesting people
   - Look at experience, tenure, and background

## Use Cases

- "Who works at Vercel?"
- "Find the VP of Engineering at acme"
- "List the first 20 employees at notion"
- "Who are the senior leaders at this company?"
