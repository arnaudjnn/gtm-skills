---
name: outbound-sales
description: B2B outbound sales drafting — pick the right accounts using buying-intent signals, find the right decision-maker, verify their email, and draft both a cold email AND a LinkedIn DM keyed to the signal that fired. Use this skill whenever the user wants to do outbound prospecting, run a cold-outreach campaign, find leads and message them, prospect companies that show signs of buying intent, identify hot accounts to message this week, find Director of X at companies hiring SDRs (or any signal → persona → message combination), draft outreach copy that references a specific reason to reach out now, or repurpose detected signals into a prospect list with messaging — even if they don't say "outbound" or name the underlying tools. Drafts only — the skill returns copy-paste-ready email + LinkedIn DM content in a structured handoff format, intentionally agnostic of the sending tool (Apollo / Outreach / Salesloft / Lemlist / Smartlead / your own — none of them need to know about gtm-tools).
---

# Outbound Sales

Outbound that lands is **best message → right person → right time**. This skill turns that into a deterministic pipeline:

- **Right time** = a [signal](https://gtm-tools.sh/signals-tools) is firing on the domain (hiring, churn, follower spike, tech-stack match). Signals are the timing trigger — the company has a problem *right now* that you can speak to.
- **Best message** = the signal that fired *also* tells you which angle to write. "Hiring 3 SDRs" → talk about SDR ramp. "Negative Trustpilot support reviews" → talk about ticket resolution. Each signal maps 1-to-1 to a message hook.
- **Right person** = the title filter to use depends on which signal fired. SDR hiring → VP Sales / Head of Sales. Support reviews → VP CX / Director Support. Each signal has a default title filter (see [`references/signal-to-angle.md`](./references/signal-to-angle.md)).

The skill ties those three together: detect → qualify person → verify contact → draft outbound, multi-channel (email + LinkedIn DM). It **does not send** — companies use their own outreach platform (Apollo, Outreach, Salesloft, Lemlist, Smartlead, Instantly, Reply, internal CRM, etc.) and this skill stays agnostic by returning drafts in a structured format that pastes into any of them.

## When to engage

Trigger phrases — invoke for any of:

- "do outbound to <domains>" / "prospect <list>"
- "find hot leads this week"
- "which companies are hiring SDRs / scaling CX / etc?"
- "draft outreach for these accounts"
- "cold email <domain>"
- "LinkedIn DM the VP of Sales at <company>"
- "build me a prospect list with messages"
- "who should I reach out to at <company>?"
- "find the right Director of CX at companies with negative Trustpilot reviews"
- mentions of: outbound, cold email, sales prospecting, signal-based outreach, account-based selling, sales sequences

Don't invoke for: pure copy work on a single existing draft (use `linkedin-copywriter` for that), Reddit (use `reddit-community-manager`), or internal-comms (not outbound).

## Prerequisites

Before invoking any tool, confirm:

1. **gtm-tools is connected.** Endpoint: `https://api.gtm-tools.sh/api/v0`. Auth: `Authorization: Bearer $GTM_TOOLS_API_KEY`. (The bare `gtm-tools.sh` is the docs site — POSTing returns 405.)
2. **LinkedIn session is in the pool** (needed for `list_linkedin_company_employees`, `get_linkedin_profile`, etc.). Call `list_connected_linkedin_accounts`. If empty: `curl -fsSL https://api.gtm-tools.sh/extension/install.sh | bash`, then Connect.
3. **Token budget.** A single qualified lead end-to-end runs ~75 tokens (25 signal + 2 company URL + 30 employees + 5 email + ~15 LinkedIn context). Confirm via `get_token_balance` before fanning out across >10 accounts.

## The pipeline

```
domains
  ↓ detect_signal (cost: 0 dispatcher + 5/detector that fires)
qualified accounts (signals firing)
  ↓ map signal → message angle + title filter (references/signal-to-angle.md)
ranked accounts with angle
  ↓ get_linkedin_company_url (cost: 2)
LinkedIn company URLs
  ↓ list_linkedin_company_employees with title_filters (cost: 30)
decision-makers
  ↓ get_email (cost: 5 per candidate)
verified contacts
  ↓ Draft cold email + LinkedIn DM keyed to signal angle
copy-paste-ready output (handoff to user's outreach platform)
```

Skipping steps is fine — if the user already has a domain + a target persona, jump straight to step 4 (`list_linkedin_company_employees`). If they already have a name + domain, jump to step 5 (`get_email`). The full pipeline is for fresh prospecting from a target-account list.

## Stage 1 — Qualify with signals

Run `detect_signal` (free dispatcher) on each domain. Only spend tokens on people-lookup for accounts where at least one signal fires. This is the single biggest token-savings lever.

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/detect_signal" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"domain":"gymshark.com","techs":["zendesk.com","intercom.com"]}' | jq .
```

The response includes which signals fired. Sort accounts by how many signals fired and which ones — see [`references/signal-to-angle.md`](./references/signal-to-angle.md) for the priority table.

**If no signals fire, deprioritize the account.** The skill should output `SKIP — no signals on <domain>` rather than fall back to generic outreach. Generic outreach is what makes B2B inboxes hostile; signal-keyed outreach is what gets replies.

## Stage 2 — Map signal → angle + title

Look up the firing signal in [`references/signal-to-angle.md`](./references/signal-to-angle.md). Each entry gives you:

- **Hook** — the one-line opener the email/DM should lead with
- **Pain** — the specific pain the signal implies
- **Title filter** — boolean syntax for `list_linkedin_company_employees`
- **Persona** — who this hook lands with

Example mappings (full list in the reference file):

| Signal | Hook | Title filter |
|---|---|---|
| `signal_hiring_sales_rep_repost` | "Saw the SDR role got re-listed — usually means the last one didn't stick" | `"(VP OR Director OR Head) AND Sales NOT intern"` |
| `signal_trustpilot_negative_support_reviews` | "Noticed 6 negative Trustpilot reviews about support response time in the last 30 days" | `"(VP OR Director OR Head) AND (CX OR Support OR \"Customer Experience\") NOT intern"` |
| `signal_socials_spike` | "Saw the +18% follower spike on Instagram last week" | `"(VP OR Director OR Head) AND (Marketing OR Brand OR Growth)"` |
| `signal_hiring_support` | "Saw the 3 CX roles posted in the last 14 days" | `"(VP OR Director OR Head) AND (CX OR Support OR Operations)"` |

When multiple signals fire, prefer the one with the most specific hook over the broad one (Trustpilot review counts > "they're hiring").

## Stage 3 — Resolve company + employees

```bash
# Company URL
curl -s -X POST "https://api.gtm-tools.sh/api/v0/get_linkedin_company_url" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"domain":"gymshark.com"}' | jq .

# Decision-makers (30 tokens; the expensive step — title filter is mandatory)
curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_linkedin_company_employees" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{
    "domain":"gymshark.com",
    "title_filters":"(VP OR Director OR Head) AND (CX OR Support) NOT intern",
    "limit":5,
    "page":1
  }' | jq .
```

Use the **5-token search variant** `search_linkedin_company_employees` for cheaper discovery if you just need name+title+linkedin_url and don't need the full About / profile id. The full `list_` variant is worth the 30 tokens only when you'll use the structured data downstream (e.g. for AI-driven personalization).

## Stage 4 — Verify the email

For each candidate, call `get_email`. Skip candidates where the response sets `is_catch_all: true` unless the user explicitly approves catch-all targets.

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/get_email" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"name":"Camille Tichard","domain":"gymshark.com","input_parameters":{"signal":"trustpilot_negative_support","lead_id":"abc123"}}' | jq .
```

`input_parameters` is echoed back in the response — use `lead_id` to correlate to the user's CRM row, and `signal` to carry the firing-signal forward into the draft.

## Stage 5 — Draft email + LinkedIn DM

Both channels, keyed to the signal angle.

### Email structure

Cold email format that converts in B2B (validated by the templates in [`references/email-templates.md`](./references/email-templates.md)):

```
Subject: <signal-specific, 5-8 words, no salesy adjectives>

<First name>,

<Signal observation — one sentence, specific number / date / fact>.

<One-line implication / pain hypothesis> — <one-line social proof or
data point that makes it concrete>.

<Soft CTA — one specific yes-no question, not "do you have 30 mins">.

<Sign-off>
```

Hard rules:
- Subject ≤ 8 words. No "Quick question", "Following up", "Touching base", "Re:".
- Body ≤ 90 words. Anything longer reads as marketing.
- First sentence references the signal specifically, not the company generally. ("Saw the 3 CX roles you posted on Aug 12" beats "Noticed Gymshark is hiring".)
- One ask, one CTA. No "happy to share more / send a deck / book a call OR a Zoom".
- No mid-sentence em dashes. No "Hope this finds you well." No "I wanted to reach out because…".

### LinkedIn DM structure

Shorter than email — LinkedIn's read context is mobile, scroll-fast:

```
Hey <First name>,

<Signal observation — one sentence>.

<Two-sentence reason it matters + concrete reference>.

<One-line ask — even softer than the email CTA. "Worth a quick chat?" or "Want me to send the X piece?">.
```

Hard rules:
- ≤ 60 words total. LinkedIn truncates DMs at the top of the conversation pane.
- No emoji unless the recipient's recent posts use them.
- No "Hope this finds you well" / "Hope you're having a great week" — instant delete.
- Don't pitch a meeting in the first DM. The first DM exists to start a thread, not close one.

See [`references/dm-templates.md`](./references/dm-templates.md) for 5 reusable templates keyed to signal type.

### Personalization layer (optional, recommended)

For high-value accounts, pull the recipient's recent activity to layer in one specific reference:

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_user_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"username":"camilletichard","limit":5}' | jq .
```

If their recent post discusses a related topic, replace the second sentence of the email/DM with a callback to it. ("Saw your Aug 5 post on AI-driven ticket triage — the signal-side of that is exactly what we're seeing across CX leaders this quarter.")

## Stage 6 — Handoff (output format)

The skill **does not send.** It returns a structured handoff per lead, ready to paste into any outreach platform.

Default output format (one JSON object per lead, in a code block):

```json
{
  "lead_id": "abc123",
  "domain": "gymshark.com",
  "first_name": "Camille",
  "last_name": "Tichard",
  "title": "VP of Customer Experience",
  "linkedin_url": "https://linkedin.com/in/camilletichard",
  "email": "camille@gymshark.com",
  "is_catch_all": false,
  "signals_fired": ["signal_trustpilot_negative_support_reviews", "signal_hiring_support"],
  "primary_signal": "signal_trustpilot_negative_support_reviews",
  "angle": "Negative Trustpilot reviews about support response time — 6 in the last 30 days",
  "email_subject": "6 Trustpilot reviews about support",
  "email_body": "Camille,\n\nSaw 6 negative Trustpilot reviews about Gymshark's support response time in the last 30 days — the common thread was 48+ hour first-reply.\n\nUsually means ticket volume jumped before headcount did. The teams we work with at your scale cut first-reply to under 6h without doubling the CX headcount.\n\nWorth a 15-min compare on what changed for them?\n\n— <signature>",
  "linkedin_dm": "Hey Camille,\n\nSaw the spike in negative Trustpilot reviews about Gymshark's support response time over the past month.\n\nUsually means ticket volume outpaced headcount. The teams I work with at your scale cut first-reply to <6h without adding bodies.\n\nWorth a quick chat?",
  "ready_to_send": true
}
```

If the user asks for a markdown table instead (easier to skim across 20+ leads):

```markdown
| Lead | Signal | Email subject | Email body | LinkedIn DM |
|---|---|---|---|---|
| Camille Tichard, VP CX, gymshark.com — camille@gymshark.com | Trustpilot negative support reviews | 6 Trustpilot reviews about support | ... | ... |
```

Ask the user up-front which format they prefer if it's not obvious from context.

**Do not call `send_linkedin_message`, `send_linkedin_invitation`, or any email-send tool.** This skill stops at the handoff. The user pushes to their outreach platform manually or via a separate skill.

## Anti-patterns this skill prevents

- Drafting outreach for accounts with no signals firing
- "Hope this email finds you well" / "I wanted to reach out because…"
- Subject lines like "Quick question" / "Following up" / "Touching base"
- Multi-CTA emails ("happy to send a deck OR book a call OR have a quick chat")
- LinkedIn DMs longer than 60 words
- Identical email + LinkedIn DM copy (each channel has its own register)
- Pasting the same generic intro across accounts with different signals
- Sending without verifying email (`is_catch_all: true` without explicit user approval)

## Example — full pipeline end-to-end

User: "Find 3 hot accounts in the CX space and draft outreach."

Agent:

1. Ask for the target domain list (or use the user's saved list).
2. For each domain, call `detect_signal` with `techs:["zendesk.com","intercom.com"]`.
3. Filter to accounts where `signal_trustpilot_negative_support_reviews` OR `signal_hiring_support` fired.
4. For each qualified account, look up `references/signal-to-angle.md` to get the title filter + hook.
5. `get_linkedin_company_url` then `list_linkedin_company_employees` with the title filter, limit=3.
6. For top candidate per company, call `get_email`.
7. Pull `list_user_posts` for personalization if budget allows.
8. Draft email + DM per the structures above.
9. Return as JSON objects (or markdown table on request).
10. Show the user the leads with a closing line: "Push these to <their platform>. Want me to draft more, or refine the angle on any of these?"

## Reference

- Signal → angle + title filter mapping: [`references/signal-to-angle.md`](./references/signal-to-angle.md)
- Cold email templates (5 keyed to signal type): [`references/email-templates.md`](./references/email-templates.md)
- LinkedIn DM templates (5 keyed to signal type): [`references/dm-templates.md`](./references/dm-templates.md)
- gtm-tools docs: https://gtm-tools.sh/introduction
- Underlying tool reference for the API: https://gtm-tools.sh/signals-tools and https://gtm-tools.sh/socials-tools
- API: `https://api.gtm-tools.sh/api/v0` — auth via `Authorization: Bearer $GTM_TOOLS_API_KEY`
