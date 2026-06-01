# Signal → Angle + Title-Filter Mapping

The signal that fires is the timing trigger AND the message angle. Each gtm-tools signal maps 1-to-1 to:

- **Hook** — the one-line opener
- **Pain** — the specific implied problem
- **Title filter** — boolean syntax for `list_linkedin_company_employees` / `search_linkedin_company_employees`
- **Persona** — who this hook lands with
- **Priority** — when multiple signals fire on the same account, prefer higher-priority

## Priority order (when picking among multiple firing signals)

1. **`signal_hiring_sales_rep_repost`** — strongest. A re-listed SDR role means the last hire churned. Highest urgency, narrowest pain.
2. **`signal_trustpilot_negative_support_reviews`** — second strongest. Customer-side evidence of an operational problem.
3. **`signal_hiring_support`** — concrete budget unlock (the team is actively scaling).
4. **`signal_hiring_sales_leadership`** — strategic moment (the leadership layer is being rebuilt).
5. **`signal_socials_spike`** — momentum signal, broader pain hypothesis.
6. **`signal_trustpilot_negative_reviews`** — generic Trustpilot negative (not support-specific).
7. **`signal_hiring_sales_rep`** — generic SDR hiring (not a repost).
8. **`signal_hiring_role`** — generic role hiring (least specific).
9. **`signal_technologies_identified`** — qualification only, not urgency.
10. **`signal_trustpilot_positive_reviews`** — congratulatory hook, low conversion.

When multiple fire, pick the most specific. Two firing > one firing (more reasons to talk now), but the lead-hook still uses the highest-priority single signal — don't try to mention all of them in one sentence.

## Full mapping

### signal_hiring_sales_rep_repost

- **Hook:** "Saw the SDR role got re-listed — usually means the last hire didn't make it past ramp."
- **Pain:** SDR churn, ramp-time problem, sales-ops infrastructure gap.
- **Title filter:** `"(VP OR Director OR Head OR Chief) AND (Sales OR Revenue OR \"Go to Market\") NOT (intern OR junior OR associate)"`
- **Persona:** VP Sales / CRO / Head of Sales Development
- **Notes:** This signal includes the role URL + the original posting date in the response. Reference both for credibility.

### signal_trustpilot_negative_support_reviews

- **Hook:** "Noticed <N> negative Trustpilot reviews about support response time in the last 30 days — the common thread was <theme>."
- **Pain:** Ticket volume outpacing headcount; first-reply SLA broken; CS team under-resourced.
- **Title filter:** `"(VP OR Director OR Head OR Chief) AND (CX OR Support OR \"Customer Experience\" OR \"Customer Success\" OR Operations) NOT intern"`
- **Persona:** VP CX / Head of Support / Director of Operations
- **Notes:** The signal response includes the reviews (titles + dates) — surface 1-2 specific complaint themes in the hook, not just the count.

### signal_hiring_support

- **Hook:** "Saw the <N> CX roles you posted in the last 14 days."
- **Pain:** Scaling team, no clear playbook for what to give the new hires, looking for tooling that compounds.
- **Title filter:** `"(VP OR Director OR Head OR Chief) AND (CX OR Support OR Operations) NOT intern"`
- **Persona:** VP CX / Head of Support
- **Notes:** This is the budget-unlock signal — they have headcount approved. The pitch should be "tooling that lets the new hires ramp in 2 weeks instead of 6", not "tooling that replaces them".

### signal_hiring_sales_leadership

- **Hook:** "Saw the <Head of Sales / VP Revenue> role posted — usually means the GTM motion is being rebuilt."
- **Pain:** Org transition, playbook reset, often coincides with a tooling re-evaluation.
- **Title filter:** `"(CEO OR Founder OR Chief OR President) NOT intern"`
- **Persona:** Founder / CEO who's about to hire the new sales leader and is evaluating the stack the new hire will inherit.
- **Notes:** Timing is sensitive — the window is the 4-8 weeks before the new leader starts. Lead with founder-pain, not seller-pain.

### signal_socials_spike

- **Hook:** "Saw the +<X>% follower spike on <Instagram/TikTok> in the last 30 days."
- **Pain:** Marketing momentum without the operational backbone — usually CX/CS or fulfillment breaks first.
- **Title filter:** `"(VP OR Director OR Head OR Chief) AND (Marketing OR Brand OR Growth OR Acquisition) NOT intern"`
- **Persona:** VP/Head of Marketing or Brand
- **Notes:** The pain isn't "they need more marketing" (they're killing it) — it's "what scales next when the marketing keeps working".

### signal_trustpilot_negative_reviews

- **Hook:** "Noticed the Trustpilot rating dropped from <X> to <Y> in the last month."
- **Pain:** Generic reputation risk. Less specific than the support-reviews variant.
- **Title filter:** `"(VP OR Director OR Head OR Chief) AND (Marketing OR Brand OR \"Customer Experience\" OR Operations) NOT intern"`
- **Persona:** VP Brand / VP CX
- **Notes:** Weaker than the support-specific variant. If both fire, lead with the support-specific one.

### signal_hiring_sales_rep

- **Hook:** "Saw you're hiring <N> SDR(s) — the first month is usually where ramp time gets decided."
- **Pain:** Ramp speed, onboarding playbook, sales enablement gap.
- **Title filter:** `"(VP OR Director OR Head OR Chief OR Manager) AND (Sales OR Revenue OR \"Sales Operations\") NOT intern"`
- **Persona:** VP Sales / Head of Sales Ops / Sales Enablement
- **Notes:** Weaker than the `repost` variant — they're hiring, but no evidence of churn (yet).

### signal_hiring_role

- **Hook:** "Saw you're hiring for <role> — usually means <implication for the team>."
- **Pain:** Depends on the role. Generic hiring signal.
- **Title filter:** Match the role being hired. Use boolean syntax that brackets the relevant function.
- **Persona:** Hiring manager for that function.
- **Notes:** The lowest-precision signal. Use only if it's the only signal that fires AND the role title implies a concrete pain you can speak to.

### signal_technologies_identified

- **Hook:** "Saw <Tech X> in your stack — most teams I work with came over from <Tech X> when <reason>."
- **Pain:** Qualification only. Not urgent on its own.
- **Title filter:** Match the function that owns the tech. (Zendesk → CX; Hubspot → Marketing; Salesforce → RevOps.)
- **Persona:** Owner of that tool inside the company.
- **Notes:** Don't lead an outbound with this alone — it's a qualification gate, not a timing trigger. Pair with one of the time-based signals above.

### signal_trustpilot_positive_reviews

- **Hook:** "Noticed the Trustpilot rating jumped to <X> last month — most companies that hit that point start asking <next question>."
- **Pain:** None directly. The pitch is forward-looking: what comes after the customer-satisfaction win.
- **Title filter:** `"(VP OR Director OR Head OR Chief) AND (Marketing OR Brand)"`
- **Persona:** VP Marketing — they own the case-study and amplification motion.
- **Notes:** Low-priority signal. Convert most effectively as a "congrats + here's-the-next-mountain" pitch, not as a pain-pitch.

## How to combine signals on a single account

When 2+ signals fire on the same domain:

- **Two highest-priority firing.** Lead with the higher one. Mention the second briefly: "Also saw <second signal>, but the bigger thing is <primary>."
- **A timing signal + a qualification signal.** Use the qualification signal to frame credibility (Zendesk users came to us from X). Use the timing signal to drive the CTA.
- **Three or more firing.** That's a hot account. The hook gets one sentence; the social proof gets one sentence; the ask gets one sentence. Don't try to mention all three signals — pick the strongest.

## When NO signals fire

Output `SKIP — no signals on <domain>` and move on. Do not fall back to a generic "noticed your company" intro. Signal-keyed outreach gets replies; generic outreach is what makes B2B inboxes hostile.

If the user explicitly overrides and wants outreach on a no-signal account, say so in the handoff JSON (`primary_signal: null`, `angle: "user_override"`) so they can audit which sends were signal-keyed vs. forced.
