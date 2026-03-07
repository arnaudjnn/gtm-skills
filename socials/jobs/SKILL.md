---
name: jobs
description: Research LinkedIn job listings for hiring intelligence.
---

# LinkedIn Jobs

Research LinkedIn job listings for a company to understand hiring patterns, team growth, and technology investments. Use this for hiring signals and competitive intelligence.

## Tools Used

- `get_linkedin_job` (2 tokens): full job description, requirements, and metadata for a specific listing
- `list_linkedin_jobs` (5 tokens): all open positions for a company, with optional title keyword filter

See `references/tools-reference.md` for exact commands.

## Workflow

1. **List jobs for a company**
   - Call `list_linkedin_jobs` with the company domain
   - Optionally filter by title keyword (e.g. "engineer", "customer success")

2. **Get full details for interesting listings**
   - Call `get_linkedin_job` with the job ID for positions worth investigating
   - Extract requirements, seniority, tech stack, and team context

3. **Summarize hiring patterns**
   - Count openings by department or function
   - Identify seniority levels being hired (IC vs leadership)
   - Note technology or tool mentions in job descriptions
   - Flag roles that indicate strategic priorities (e.g. new market, new product)

## Use Cases

- "What jobs does gymshark.com have open?"
- "Is acme.com hiring CX roles?"
- "Get the full description for job 4356405688"
- "What technology stack is this company investing in based on their job posts?"
