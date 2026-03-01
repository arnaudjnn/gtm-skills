# Test: Signals Skill

**Skill under test:** `signals` (and all sub-skills)
**Skill type:** Reference + Technique
**Test approach:** Retrieval and application scenarios — does the agent route correctly, use the right MCP tools, and follow workflows?

## Setup

```
[Test scaffold] For each scenario below, commit to a specific answer with exact MCP tool calls and parameters. You have access to all 6 MCP tools from the signals-tools server.
```

---

## Scenario 1: Router — Detect-All vs Specific Sub-Skill

**Tests:** Correct routing between detect-all and targeted sub-skills

```
Case A: "Scan acme.com for buying signals."
Case B: "What are the Trustpilot reviews like for acme.com?"
```

**Expected:**
- Case A: Uses `detect-all` — single `detect_signal` call, returns triggered signals array
- Case B: Uses `reputation` — runs all 3 Trustpilot tools for a full sentiment breakdown

**Failure indicators:**
- Uses 3 individual tools when detect-all was appropriate
- Uses only detect_signal when detailed Trustpilot analysis was requested

---

## Scenario 2: Detect-All — Interpreting Results

**Tests:** Correct interpretation of detect_signal output

```
detect_signal for "gymshark.com" returns:
[
  { "signal_name": "signal_socials_spike", "data": { "spike_detected": true, "growth_percentage": 12.5, "spike_reason": "[Instagram] +12.5% overall growth" } },
  { "signal_name": "signal_hiring_role", "data": { "is_detected": true, "matching_jobs_count": 3, "matching_jobs": [{ "title": "Customer Support Lead" }, { "title": "CX Manager" }, { "title": "Head of Customer Experience" }] } }
]
```

**Expected:**
- Identifies 2 out of 3 signals triggered (socials spike + hiring, not support reviews)
- Summarizes the Instagram growth spike (12.5%)
- Lists the 3 matching CX hiring roles
- Notes that negative support reviews were NOT detected

**Failure indicators:**
- Misses that signal_trustpilot_negative_support_reviews was not in the results (i.e. not triggered)
- Fabricates data not present in the response

---

## Scenario 3: Reputation — Full Analysis

**Tests:** Running all 3 Trustpilot tools and analyzing sentiment

```
Analyze Trustpilot reputation for "acme.com".

signal_trustpilot_negative_reviews returns: is_detected: true, negative_reviews_count: 4
signal_trustpilot_negative_support_reviews returns: is_detected: true, negative_reviews_count: 2
signal_trustpilot_positive_reviews returns: is_detected: true, positive_reviews_count: 15
```

**Expected:**
- Calls all 3 Trustpilot tools with domain "acme.com"
- Reports the sentiment balance: 15 positive vs 4 negative
- Notes that 2 out of 4 negative reviews specifically mention support
- Includes the Trustpilot URL

**Failure indicators:**
- Only calls one or two Trustpilot tools
- Doesn't compare positive vs negative counts
- Ignores the support-specific subset

---

## Scenario 4: Social Spike — Interpreting Thresholds

**Tests:** Understanding spike detection logic

```
signal_socials_spike for "newbrand.com" returns:
{
  "spike_detected": true,
  "growth_percentage": 3.2,
  "spike_reason": "[Instagram] Single-day spike on 2026-02-25: +8,500 followers (4.2x the daily average)",
  "instagram_username": "newbrand",
  "instagram_history": [14 days of data]
}
```

**Expected:**
- Explains the spike was a single-day event, not overall growth (3.2% < 10% threshold)
- Identifies the spike day (Feb 25) and magnitude (4.2x average, 8,500 followers)
- Notes this was Instagram only (no TikTok data)

**Failure indicators:**
- Claims 3.2% overall growth triggered the spike (it didn't — it was the single-day threshold)
- Ignores the spike_reason explanation

---

## Scenario 5: Hiring — Filter Syntax

**Tests:** Correct job_title_filters construction

```
"Check if acme.com is hiring for VP of Engineering or Head of Engineering, but not junior roles."
```

**Expected:**
- Calls `signal_hiring_role` with domain: "acme.com"
- Uses filter: `"(vp engineering OR head of engineering OR vp of engineering OR head of engineering) NOT junior"` or equivalent valid expression
- Uses OR for alternatives and NOT for exclusions

**Failure indicators:**
- Passes plain text without OR/NOT operators
- Uses AND instead of OR for alternative titles
- Omits the NOT junior exclusion

---

## Scenario 6: Cost Awareness

**Tests:** Understanding token costs and choosing efficient tools

```
"I need to check 20 domains for any buying signals. What's the most efficient approach?"
```

**Expected:**
- Recommends `detect_signal` for each domain (15 tokens each, 300 total)
- Notes this is cheaper than running all 6 tools individually (30 tokens each, 600 total)
- Mentions that detect_signal covers 3 of the 6 tools (socials spike, negative support reviews, CX hiring)
- If full Trustpilot analysis is also needed, explains the additional cost

**Failure indicators:**
- Suggests running all 6 tools per domain without mentioning the detect_signal shortcut
- Gets the token costs wrong

---

## Scenario 7: Batch Scanning — Multiple Domains

**Tests:** Handling a list of domains efficiently

```
"Scan these 3 domains for signals: acme.com, bigcorp.com, startup.io"
```

**Expected:**
- Calls `detect_signal` for each of the 3 domains
- Presents results in a summary table (domain → triggered signals)
- Domains with no signals show as "no signals detected"
- Total cost: 45 tokens (3 x 15)

**Failure indicators:**
- Runs individual tools instead of detect_signal
- Only scans one domain
- Doesn't present a comparative summary

---

## Scoring

| Scenario | Pass Criteria |
|----------|---------------|
| 1 - Router | Correct sub-skill for general scan vs Trustpilot analysis |
| 2 - Detect-All | Identifies triggered vs non-triggered signals, summarizes correctly |
| 3 - Reputation | All 3 Trustpilot tools called, sentiment comparison, support subset noted |
| 4 - Social Spike | Explains correct threshold trigger, identifies spike day |
| 5 - Hiring Filter | Valid OR/NOT syntax, correct parameter name |
| 6 - Cost Awareness | Correct token costs, recommends detect_signal for batch |
| 7 - Batch Scanning | detect_signal per domain, summary table, correct total cost |

**Pass threshold:** 5/7 correct
