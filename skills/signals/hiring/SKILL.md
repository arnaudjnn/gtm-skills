---
name: hiring
description: Detect if a company is hiring for specific roles. Includes preset signals for support, sales reps, sales leadership, and SDR repost detection, plus a generic tool with custom filters.
---

# Hiring Detection

Detect whether a company is hiring for specific roles by searching 20+ startup job boards and ATS platforms (free), with LinkedIn as fallback.

## Tools Used

- `signal_hiring_role` (5 tokens): generic — takes custom `job_title_filters` with OR/NOT operators
- `signal_hiring_support` (5 tokens): preset — CX, customer support/service/experience roles
- `signal_hiring_sales_rep` (5 tokens): preset — SDR, BDR, sales development roles
- `signal_hiring_sales_leadership` (5 tokens): preset — Head of Sales, VP Sales, RevOps, CRO roles
- `signal_hiring_sales_rep_repost` (5 tokens): preset — SDR/BDR roles reposted within 60 days (≥2 postings required)

See `references/tools-reference.md` for exact commands.

## Workflow

### Using preset signals (recommended for common use cases)

1. Pick the right preset tool based on the use case
2. Call it with just the `domain` — no filter needed
3. Interpret `is_detected` and `matching_jobs`

### Using the generic tool (custom filters)

1. Build a filter string using OR/NOT operators (see syntax below)
2. Call `signal_hiring_role` with `domain` and `job_title_filters`
3. Interpret the results

## Filter Syntax (for `signal_hiring_role`)

| Operator | Example | Description |
|----------|---------|-------------|
| OR | `"cx OR customer support"` | Match any of the terms |
| NOT | `"NOT junior"` | Exclude matches |
| Parentheses | `"(cx OR customer support) NOT junior"` | Group terms |

Matching is case-insensitive.

## Preset Filters Reference

| Tool | What it searches for |
|------|---------------------|
| `signal_hiring_support` | CX, customer support, customer experience, customer service, support engineer, support specialist (NOT junior) |
| `signal_hiring_sales_rep` | SDR, BDR, sales development, business development representative |
| `signal_hiring_sales_leadership` | Head of Sales, VP Sales, Director of Sales, RevOps, Revenue Operations, CRO |
| `signal_hiring_sales_rep_repost` | SDR, BDR, sales development rep — 60-day window, requires 2+ postings |

## Signal Interpretation

| Signal | What it means |
|--------|--------------|
| `signal_hiring_support` detected | Company investing in CX — potential need for CX tooling |
| `signal_hiring_sales_rep` detected | Budget allocated for outbound — immediate sales need |
| `signal_hiring_sales_leadership` detected | New leadership = permission to change systems and processes |
| `signal_hiring_sales_rep_repost` detected | Ramp failed, turnover, or no results — pain point signal |

## Use Cases

- "Is acme.com hiring CX reps?" → `signal_hiring_support`
- "Are they building a sales team at notion.so?" → `signal_hiring_sales_rep`
- "Is gymshark hiring a Head of Sales?" → `signal_hiring_sales_leadership`
- "Has this company been struggling to fill SDR roles?" → `signal_hiring_sales_rep_repost`
- "Check if they're hiring product managers" → `signal_hiring_role` with custom filter
