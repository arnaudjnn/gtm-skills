# MCP Tools Reference

Complete reference for all 6 MCP tools provided by the signals-tools server.

---

## detect_signal

Runs multiple signal detections for a company domain and returns only the ones that triggered. Checks: `signal_socials_spike`, `signal_trustpilot_negative_support_reviews`, `signal_hiring_role` (CX filter hardcoded).

**Cost:** 15 tokens

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

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| domain | string | yes | The company domain to look up (e.g. "gymshark.com") |

**Returns:** Same shape as `signal_trustpilot_negative_reviews`.

---

## signal_trustpilot_positive_reviews

Detects positive Trustpilot reviews for a company in the last 30 days. A review is positive if its rating is 4 or 5 (out of 5).

**Cost:** 5 tokens

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

**Parameters:**
| Name | Type | Required | Description |
|------|------|----------|-------------|
| domain | string | yes | The company domain to look up (e.g. "gymshark.com") |
| job_title_filters | string | yes | Filter for job titles with OR and NOT operators (e.g. `"(cx OR customer support) NOT junior"`) |

**Filter syntax:**
- `OR` — match any of the terms: `"cx OR customer support"`
- `NOT` — exclude matches: `"NOT junior"`
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
