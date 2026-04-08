# Signals Tools API Reference

Complete reference for all 13 tools provided by the signals-tools server.

---

## How to Call Tools

Use the Bash tool to run curl commands. Every call follows this pattern:

```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/TOOL_NAME" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{...arguments...}'
```

- The API base URL is `https://signals.gtm-engine.sh/api/v0`
- `$GTM_ENGINE_API_KEY`: the API key for authentication (see the signals SKILL.md for how to obtain one)
- The response is direct JSON (parse with `jq .`)

### Tip: parse responses with jq

```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/detect_signal" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"domain":"gymshark.com"}' | jq .
```

---

## detect_signal

Runs multiple signal detections for a company domain and returns only the ones that triggered. Checks: `signal_socials_spike`, `signal_trustpilot_negative_support_reviews`, `signal_hiring_role` (CX filter hardcoded).

**Cost:** 15 tokens

```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/detect_signal" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"domain":"gymshark.com"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| domain | string | yes | The company domain to look up (e.g. "gymshark.com") |

**Returns:** Array of detected signals:
```json
[
  {
    "signal_name": "signal_socials_spike",
    "data": { ... }
  }
]
```

Each element has:
| Field | Type | Description |
|-------|------|-------------|
| signal_name | string | Name of the triggered signal tool |
| data | object | Full response from that signal tool |

Returns an empty array `[]` if no signal is detected. Individual signal failures are silently skipped.

---

## signal_trustpilot_negative_reviews

Detects negative Trustpilot reviews for a company in the last 30 days. A review is negative if its rating is 1 (out of 5).

**Cost:** 5 tokens

```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/signal_trustpilot_negative_reviews" \
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
| trustpilot_url | string | Trustpilot page URL for the company |
| is_detected | boolean | True if at least one 1-star review was posted in the last 30 days |
| negative_reviews_count | number | Count of matching reviews |
| negative_reviews | TrustpilotReview[] | Array of matching review objects |

---

## signal_trustpilot_negative_support_reviews

Detects negative Trustpilot reviews that mention customer support. Same as `signal_trustpilot_negative_reviews` but also requires "support" in the review title or message (case-insensitive).

**Cost:** 5 tokens

```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/signal_trustpilot_negative_support_reviews" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"domain":"gymshark.com"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| domain | string | yes | The company domain to look up (e.g. "gymshark.com") |

**Returns:** Same shape as `signal_trustpilot_negative_reviews`.

---

## signal_trustpilot_positive_reviews

Detects positive Trustpilot reviews for a company in the last 30 days. A review is positive if its rating is 4 or 5 (out of 5).

**Cost:** 5 tokens

```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/signal_trustpilot_positive_reviews" \
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
| trustpilot_url | string | Trustpilot page URL for the company |
| is_detected | boolean | True if at least one 4-5 star review was posted in the last 30 days |
| positive_reviews_count | number | Count of matching reviews |
| positive_reviews | TrustpilotReview[] | Array of matching review objects |

---

## TrustpilotReview (shared shape)

All Trustpilot tools return reviews with this structure:

| Field | Type | Description |
|-------|------|-------------|
| reviewer | string | Reviewer name |
| rating | number | Star rating (1-5) |
| date | string | Review date |
| title | string | Review title |
| message | string | Review body text |
| has_company_reply | boolean | Whether the company replied to the review |

---

## signal_socials_spike

Detects significant follower spikes on Instagram and/or TikTok over a 14-day window.

**Cost:** 5 tokens

```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/signal_socials_spike" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"domain":"gymshark.com"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| domain | string | yes | The company domain to look up (e.g. "gymshark.com") |

**Spike detection thresholds:**
- Overall growth >= 10% across the 14-day window
- Single-day gain > 3x daily average AND > 0.1% of total followers
- Day-over-day acceleration > 2x previous day AND > 0.1% of total followers

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| domain | string | The queried domain |
| spike_detected | boolean | True if any spike threshold was met |
| growth_percentage | number | Average growth % across available platforms |
| spike_reason | string \| null | Explanation of why a spike was flagged, or null |
| instagram_username | string | (optional) Instagram handle found |
| instagram_history | DailyStats[] | (optional) 14 days of Instagram stats |
| tiktok_username | string | (optional) TikTok handle found |
| tiktok_history | DailyStats[] | (optional) 14 days of TikTok stats |

### DailyStats shape

| Field | Type | Description |
|-------|------|-------------|
| date | string | Date (YYYY-MM-DD) |
| day | string | Day of week (Mon, Tue, ...) |
| followers_gained | number \| null | Followers gained that day |
| followers_total | number \| null | Total follower count |
| following_change | number \| null | Following count change |
| following_total | number \| null | Total following count |
| media_change | number \| null | Media/posts count change |
| media_total | number \| null | Total media/posts count |

---

## signal_hiring_role

Detects whether a company is hiring for roles matching given job title filters via LinkedIn.

**Cost:** 5 tokens

```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/signal_hiring_role" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{
    "domain":"gymshark.com",
    "job_title_filters":"(cx OR customer support) NOT junior"
  }'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| domain | string | yes | The company domain to look up (e.g. "gymshark.com") |
| job_title_filters | string | yes | Filter for job titles with OR and NOT operators (e.g. `"(cx OR customer support) NOT junior"`) |

**Filter syntax:**
- `OR`:match any of the terms: `"cx OR customer support"`
- `NOT`:exclude matches: `"NOT junior"`
- Parentheses for grouping: `"(cx OR customer support) NOT junior"`
- Case-insensitive matching

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| domain | string | The queried domain |
| linkedin_url | string | LinkedIn company page URL |
| all_jobs_url | string | URL to all job listings |
| total_jobs | number | Total job listings for the company |
| is_detected | boolean | True if at least one job matches the filter |
| matching_jobs_count | number | Count of matching jobs |
| matching_jobs | LinkedInJob[] | Array of matching job objects |

### LinkedInJob shape

| Field | Type | Description |
|-------|------|-------------|
| id | string | LinkedIn job ID |
| title | string | Job title |
| url | string | Job listing URL |
| location | string | Job location |
| posted_at | string | When the job was posted |

---

## signal_hiring_support

Detects whether a company is hiring for customer support / CX roles. Preset filter: CX, customer support, customer experience, customer service, support engineer, support specialist (NOT junior).

**Cost:** 5 tokens

```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/signal_hiring_support" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"domain":"gymshark.com"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| domain | string | yes | The company domain to look up |

**Returns:** Same shape as `signal_hiring_role`.

---

## signal_hiring_sales_rep

Detects whether a company is actively hiring SDRs or BDRs. Preset filter: SDR, BDR, sales development, business development representative.

**Cost:** 5 tokens

```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/signal_hiring_sales_rep" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"domain":"gymshark.com"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| domain | string | yes | The company domain to look up |

**Returns:** Same shape as `signal_hiring_role`.

---

## signal_hiring_sales_leadership

Detects whether a company is hiring sales leadership or RevOps roles. Preset filter: Head of Sales, VP Sales, Director of Sales, RevOps, Revenue Operations, CRO.

**Cost:** 5 tokens

```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/signal_hiring_sales_leadership" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"domain":"gymshark.com"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| domain | string | yes | The company domain to look up |

**Returns:** Same shape as `signal_hiring_role`.

---

## signal_hiring_sales_rep_repost

Detects whether a company has reposted SDR/BDR roles within the last 60 days. Triggers only if 2+ postings are found, indicating ramp failure, turnover, or underperformance.

**Cost:** 5 tokens

```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/signal_hiring_sales_rep_repost" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{"domain":"gymshark.com"}'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| domain | string | yes | The company domain to look up |

**Returns:** Same shape as `signal_hiring_role`. Note: `is_detected` is true only when 2+ postings are found (indicating a repost).

---

## signal_technologies_identified

Detects whether specific technologies are used on a website by crawling HTML and JS bundles.

**Cost:** 5 tokens

```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/signal_technologies_identified" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{
    "domain":"mammaly.de",
    "technologies":["zendesk.com","intercom.io","digitalgenius.com"]
  }'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| domain | string | yes | The company domain to look up (e.g. "mammaly.de") |
| technologies | string[] | yes | Array of technology domains to detect (e.g. ["zendesk.com", "intercom.io"]) |

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| domain | string | The queried domain |
| is_detected | boolean | True if at least one technology was found |
| technologies_found | string[] | List of matched technology domains |
| matches | TechnologyMatch[] | Per-technology results |

### TechnologyMatch shape

| Field | Type | Description |
|-------|------|-------------|
| technology | string | Technology domain searched for |
| found | boolean | Whether it was found |
| sample_references | string[] | Snippets showing where it was found in the source |

---

## set_signals_order

Sets the order in which `detect_signal` runs signal checks for your workspace. Signals omitted from the list will be skipped.

**Cost:** 0 tokens

```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/set_signals_order" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{
    "order":["signal_socials_spike","signal_hiring_role"]
  }'
```

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| order | string[] | yes | Ordered list of signal names. Available: `signal_socials_spike`, `signal_trustpilot_negative_support_reviews`, `signal_hiring_role` |

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| message | string | Confirmation message |
| order | string[] | The new signal order |

---

## get_signals_order

Returns the current signal execution order for your workspace.

**Cost:** 0 tokens

```bash
curl -s -X POST "https://signals.gtm-engine.sh/api/v0/get_signals_order" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_ENGINE_API_KEY" \
  -d '{}'
```

**Parameters:** None.

**Returns:**
| Field | Type | Description |
|-------|------|-------------|
| order | string[] | Current signal execution order |
| available_signals | string[] | All available signal names |
