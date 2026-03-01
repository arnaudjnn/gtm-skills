---
name: hiring
description: Detect if a company is hiring for specific roles via LinkedIn job listings. Supports custom job title filters with OR/NOT operators for targeted role detection.
---

# Hiring Detection

Detect whether a company is hiring for specific roles by searching their LinkedIn job listings. Use this skill when you need custom job title filters — `detect-all` only checks for CX roles.

## MCP Tools Used

- `signal_hiring_role` (5 tokens) — LinkedIn job listings filtered by title

## Workflow

1. **Determine the job title filter**
   - Ask the user what roles to look for, or use context from the conversation
   - Build a filter string using OR/NOT operators

2. **Call `signal_hiring_role`**
   - Pass `domain` and `job_title_filters`
   - The tool finds the company's LinkedIn page and searches their job listings

3. **Interpret the results**
   - `is_detected`: whether any jobs matched the filter
   - `matching_jobs_count` and `matching_jobs`: the matches with title, URL, location
   - `total_jobs`: total job count for context (are they on a hiring spree?)

4. **Output a hiring summary**
   - Whether matching roles were found
   - Job titles, locations, and links
   - Total jobs vs matching jobs ratio
   - LinkedIn company page URL for further research

## Filter Syntax

The `job_title_filters` parameter supports boolean operators for flexible matching:

| Operator | Example | Description |
|----------|---------|-------------|
| OR | `"cx OR customer support"` | Match any of the terms |
| NOT | `"NOT junior"` | Exclude matches |
| Parentheses | `"(cx OR customer support) NOT junior"` | Group terms |

Matching is case-insensitive.

## Common Filter Examples

| Looking For | Filter |
|-------------|--------|
| CX / support roles | `"(cx OR customer support OR customer experience OR customer service) NOT junior"` |
| Engineering leaders | `"(vp engineering OR head of engineering OR engineering director)"` |
| Sales roles | `"(account executive OR sales OR business development) NOT intern"` |
| Product roles | `"(product manager OR product owner OR head of product)"` |
| Marketing leaders | `"(vp marketing OR head of marketing OR cmo OR marketing director)"` |

## Use Cases

- "Is acme.com hiring CX reps?"
- "Check if gymshark.com is looking for engineering leaders"
- "Are they hiring sales people at notion.so?"
- "Find companies from this list that are hiring product managers"
