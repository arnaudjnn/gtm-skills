---
name: reddit-community-manager
description: Manage Reddit community engagement for a B2B company via Bash — hunt the best threads for the business (industry questions, competitor complaints, category buying-research), qualify the posters before replying, then bring real value in a comment that mentions your company *sparingly* and discloses affiliation *always*. Never cheesy, never pitch-first; the value-to-pitch ratio is roughly 9:1. Use this skill whenever the user mentions Reddit, mentions a subreddit (r/anything), shares a reddit.com or redd.it URL, asks to find / monitor / reply to / engage with Reddit threads, asks to post on Reddit, asks to DM a Reddit user, asks where their competitors are getting complained about, asks to find threads about <industry/category>, mentions Reddit outreach / marketing / community management / monitoring, asks "should I reply to this Reddit post", or asks to check a Reddit user's credibility — even if they don't explicitly say "skill" or name the underlying tools. Every comment that touches the business OR even adjacent to your product MUST open with a disclosure line ("<First> from <Company> here…") — non-negotiable.
---

# Reddit Community Manager

Engage Reddit communities for a B2B company without getting the account shadowbanned. Reddit is the highest-intent acquisition channel B2B teams ignore because the patterns that work on every other social network (broadcast the product, paste in a CTA, use upbeat marketing copy) are exactly what triggers Reddit's spam filters and community downvotes. This skill wires the **21 Reddit tools** at `api.gtm-tools.sh` into a five-stage loop, and bakes in the quality rules that actually work — the disclosure pattern, the no-links rule, the should-I-reply gate, and the anti-AI-writing scrubber.

## How to call tools

Use the Bash tool to run curl. Every call follows this pattern:

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/TOOL_NAME" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" \
  -d '{...arguments...}' | jq .
```

`$GTM_TOOLS_API_KEY` must be set. Get one at https://gtm-tools.sh via `get_api_key` (or `gtm-tools admin login` from the CLI). Per-tool curl examples are in [`references/tools-reference.md`](./references/tools-reference.md).

> Heads up: `gtm-tools.sh` is the docs site (POSTing to it returns 405). The tools live at `api.gtm-tools.sh`. Always use the `api.` subdomain.

## Prerequisites

Before invoking any tool, confirm:

1. **A Reddit session is in the pool.** Call `list_connected_reddit_accounts`. Each `ready` row has a `reddit_username` — pass that as `senderUsername` to every write tool. If the list is empty, the user needs to install the browser extension (`curl -fsSL https://api.gtm-tools.sh/extension/install.sh | bash`) and click Connect. Write tools fail until at least one session is pooled.
2. **Token balance.** Most reads cost 1, most writes cost 5. Check via `get_token_balance`. The minimum to run the full loop end-to-end is ~20 tokens.

## When to engage and when to skip

Reddit punishes over-engagement harder than under-engagement. The cheapest accounts to burn are new ones; even one rule-edge reply can shadowban an account. Before drafting any reply, run the five-question gate:

1. **Is this within our expertise?** If you can't answer from genuine knowledge, skip.
2. **Is there an actual question or answerable premise?** Rants without a question rarely deserve a reply — the OP isn't asking for one.
3. **Can you answer without speculating about the OP's personal situation?** "Should I do X?" questions usually need facts only the OP has.
4. **Is a reply on-brand?** Don't reply where your product wouldn't naturally come up — Reddit users smell promo from miles away.
5. **Does your reply add something the existing top comments don't?** If not, upvote them via `vote_reddit` and move on.

**If any answer is no, output `SKIP — <one-line reason>` instead of drafting a reply.** Skipping is the highest-leverage action on Reddit. The reason: a thoughtful no-reply costs nothing; a marginal reply burns karma that took weeks to build. The math is asymmetric, so treat the bar for replying as high.

## The five-stage loop

Map every Reddit request onto these five stages and pick the tool by stage:

| Stage | Goal | Tools |
|---|---|---|
| **1. Discover** | Find subreddits + threads worth engaging | `search_reddit_subreddits`, `list_subreddit_posts`, `search_reddit_posts`, `get_subreddit_about` |
| **2. Evaluate** | Decide whether this thread + poster is worth your time | `get_reddit_post`, `get_reddit_user`, `list_reddit_user_posts` |
| **3. Engage** | Reply / post / vote / DM — only after Stage 2 passes | `create_reddit_comment`, `create_reddit_post`, `vote_reddit`, `send_reddit_message` |
| **4. Follow up** | Get notified, queue work, read the inbox | `follow_reddit_post`, `save_reddit_thing`, `list_reddit_saved`, `list_reddit_inbox` |
| **5. Organize** | Build the persistent monitoring setup | `subscribe_reddit_subreddit`, `create_reddit_custom_feed`, `list_reddit_custom_feed_posts`, `list_reddit_subscriptions`, `list_reddit_custom_feeds` |

Don't skip stages. Specifically: never go to Stage 3 without Stage 2 ("is this poster credible?") and a Stage 1 compliance check (`get_subreddit_about` — see below). Both gates catch problems the reply itself can't fix.

## Stage 1 — Discover

The first time you work a new ICP, build a shortlist of 10–30 subreddits, bulk-subscribe so the account looks organic, and group them into a custom feed:

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/search_reddit_subreddits" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"query":"<ICP topic>","limit":25}' | jq .

curl -s -X POST "https://api.gtm-tools.sh/api/v0/subscribe_reddit_subreddit" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","subreddits":["sub1","sub2"],"action":"sub"}' | jq .

curl -s -X POST "https://api.gtm-tools.sh/api/v0/create_reddit_custom_feed" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","name":"icp_feed","subreddits":["sub1","sub2","sub3"]}' | jq .
```

Why subscribe before engaging: subreddit subscriptions are a credibility signal moderators check. A new account that posts in a subreddit it isn't subscribed to gets more scrutiny than one that's been a subscriber for weeks.

Naming constraint: `create_reddit_custom_feed`'s `name` must match `^[A-Za-z0-9_]+$` (letters, digits, underscores only; no dashes; max 50 chars). Use `display_name` for the human-readable label. The reason — Reddit's multireddit URL scheme reserves dashes for path semantics, so a dash-containing name returns a 400.

For the daily monitoring loop, read the custom feed sorted by `rising`:

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_reddit_custom_feed_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","feed_name":"icp_feed","sort":"rising","limit":50}' | jq .
```

Why `rising` and not `hot`: `rising` surfaces threads with early momentum — fewer comments, fresher conversation. Being one of the first three comments outperforms being the fiftieth by a wide margin, both for visibility and for the OP actually reading your reply.

### What threads are worth engaging with

Not every thread is good for the business. The ones that compound — both for reply credibility and for downstream brand mentions in LLM answers / SERP — fall into four categories. Hunt these specifically before anything else.

**1. Industry / category questions.** Someone in the buying-research phase asking "best <category> for <use-case>", "how does <X> work at <scale>", "anyone using <category>". These threads attract dozens of vendor-curious lurkers per upvote. Be the third comment with the substantive answer and you'll get read for years.

```bash
# Category research queries
curl -s -X POST "https://api.gtm-tools.sh/api/v0/search_reddit_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"query":"best <your category> for","time_window":"month","limit":25}' | jq .

curl -s -X POST "https://api.gtm-tools.sh/api/v0/search_reddit_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"query":"anyone using <category>","time_window":"month","limit":25}' | jq .
```

**2. Competitor complaints.** Someone openly frustrated with a tool you compete with — pricing change, feature regression, support failure, vendor lock-in. These are gold: the OP is already in market for an alternative, the thread attracts other unhappy customers, and a value-first comment (NOT a "you should switch to us" pitch) earns trust. Disclose your affiliation, then *only* contribute if you have a substantive non-pitch take.

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/search_reddit_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"query":"<competitor> alternatives","time_window":"month","limit":25}' | jq .

curl -s -X POST "https://api.gtm-tools.sh/api/v0/search_reddit_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"query":"<competitor> vs","time_window":"month","limit":25}' | jq .

# Frustration / churn signals
curl -s -X POST "https://api.gtm-tools.sh/api/v0/search_reddit_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"query":"fed up with <competitor>","time_window":"month","limit":25}' | jq .

curl -s -X POST "https://api.gtm-tools.sh/api/v0/search_reddit_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"query":"<competitor> pricing","time_window":"month","limit":25}' | jq .
```

**3. Pain-point posts that match your wedge.** Someone describing the problem your product solves, without naming any vendor. "Struggling with X", "our team can't handle Y at scale", "Z is broken — how do you handle it?". You don't need to pitch in these threads — just contribute the technique, then disclose at the bottom. The brand mention compounds via LLM training-data ingestion (see [Results Ranking Optimization](https://gtm-tools.sh/documentation/guides/results-ranking)).

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/search_reddit_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"query":"<pain phrase>","time_window":"week","limit":25}' | jq .
```

**4. Show-and-tell threads where your stack is part of the answer.** Someone shares their architecture / tooling / playbook and your product is genuinely part of how a real team solves the problem. The comment is "Here's how we use <X>, <Y>, and yours" — natural mention, no pitch.

```bash
# Show-and-tell / "what's your stack" threads
curl -s -X POST "https://api.gtm-tools.sh/api/v0/search_reddit_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"query":"my <category> stack","time_window":"month","limit":25}' | jq .
```

### What threads to SKIP

- Pure venting with no question — no actionable thing to reply to.
- Threads where 5+ vendors have already commented — saturation kills any further reply.
- Subs where self-promotion is banned by rule (run `get_subreddit_about` first; if rule fires on commercial mentions, skip).
- Threads older than 14 days unless the OP is still active in comments — dead threads don't compound.

**Compliance gate — call `get_subreddit_about` before posting in any new subreddit.** Rule violations are the single biggest shadowban trigger. The response includes both metadata (subscribers, allow_images, restrict_posting, etc.) and the actual rules array (each with `name`, `description`, `applies_to: all|link|comment`, `priority`). Read the rules, especially anything about self-promotion, affiliate links, or "no blog posts" — these are the rules new B2B accounts violate first.

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/get_subreddit_about" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"subreddit":"<sub>"}' | jq .
```

## Stage 2 — Evaluate

A real ICP poster looks like this:

- Account age > 90 days
- Balanced karma (link + comment, not lopsided one way)
- Posts in subreddits adjacent to the topic, not just promotional ones
- Active in the past 30 days

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/get_reddit_user" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"username":"<u>"}' | jq .

curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_reddit_user_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"username":"<u>","type":"overview","limit":25}' | jq .
```

Red flags that should drop you out of Stage 2 with no reply:

- Account under 30 days with disproportionately high karma (often purchased)
- Posts only in promotional or low-effort subs (r/promote, engagement-farm subs)
- Subreddit history contradicts the persona they're claiming in this post (e.g. "I'm a CTO" but only posts in r/cscareerquestions complaining about junior roles)

After the poster passes, evaluate the thread itself with `get_reddit_post` — read the full body and comment tree. If the question can be answered authoritatively in ≤ 300 words and the OP is engaging with replies, it's worth a reply. If the thread is already saturated with comments saying what you'd say, skip — upvote the best one with `vote_reddit` instead.

## Stage 3 — Engage

When the gate passes, the reply has to read like a human wrote it. Reddit punishes AI-tells harder than any other platform — algorithmically (mod tools and content classifiers flag templates) and socially (commenters call it out in replies, downvotes cascade).

### The value-to-pitch ratio (9:1, not negotiable)

Every comment is 90% useful information for the reader, 10% mention-of-your-company-at-most. This is not "value-first then pitch" — it's "value-only with a disclosure in the opener". The pitch *is* the disclosure; the body has zero sales language.

Concrete rules:

- **At least 90% of the body is genuinely useful to a reader who will never buy from you.** Teach the technique, share the data, explain the gotcha. If you removed every mention of your company, the comment should still be the best response in the thread.
- **One mention of your company, max.** And only when it's the natural example of how the technique gets applied. "We do X by doing Y, then Z" beats "Our product X solves this problem".
- **No CTA. No "happy to share more." No "DM me."** If they want more, they'll click your profile.
- **No marketing adjectives anywhere in the comment.** *Game-changer, transformative, robust, comprehensive* — instant downvote.
- **If the comment doesn't have a substantive non-pitch answer, don't post it.** Don't reply just to plant a brand mention. Reddit users notice every time.

### The disclosure rule (MANDATORY — non-negotiable)

**Every comment where your company / product comes up — even tangentially — opens with a disclosure line.** Not just when you're directly recommending it. Not just when you link to it. If you work at the company being discussed (or one in the same space), the disclosure goes first:

```
<First name> from <Company> here. <direct answer>. <reasoning>. <optional caveat>.
```

When the disclosure is required:

- You name your company anywhere in the body — even as "the team I work with"
- You discuss your category and one specific vendor's approach
- You reply in a thread about a competitor with substantive comparison
- You mention any vendor you have an affiliation with (employee, advisor, investor, contractor)
- The thread is about a problem your product solves and you're going to mention how you'd approach it

When the disclosure isn't required:

- You're commenting on something purely technical / informational with no commercial angle, AND you don't mention your company at all in the body. Even then, including the disclosure is usually better — it builds trust faster than zero-disclosure comments do.

Why disclosure works: Reddit's transparency norm is enforced socially (FTC + community both). Disclosure flips the reader's frame from "this is an ad" to "an expert is chiming in" — same content, opposite reception. Most B2B accounts that get shadowbanned never disclosed. Most that build karma always did. The 9-in-10 of your readers who weren't going to buy from you anyway — they don't penalize you for disclosing, they penalize you for *not* disclosing.

A comment without the required disclosure is a comment that fails the skill. Rewrite, don't post.

**Example 1 — disclosed reply:**

Input: A thread in r/selfhosted asks "Anyone using Matrix for self-hosted assistant bots? Push notifications working?"

Output:
```
Arnaud from gtm-tools here. Matrix push notifications work fine through Element on
mobile — the gotcha is the Megolm device verification on container restarts. If your
bot's olm store isn't persisted to a volume, every Docker restart drops the device
identity and you re-verify from your phone each time.

Fix is to mount the olm store on a host volume (or in S3 if you're paranoid). After
that, push works like any other Matrix room.
```

What this example demonstrates: opener with disclosure, direct answer in the first body sentence, technical reasoning, no link, no marketing copy, no AI tells. Notice that gtm-tools is named only in the disclosure — the body teaches a Megolm gotcha that's useful regardless of which product the reader uses.

### Don't be cheesy (the 4 patterns that kill comments)

These are the reflexes that vendor-side accounts default to and that the Reddit community has learned to recognize and downvote on sight. Catch and rewrite:

1. **The pitch-disguised-as-answer.** "Great question — at <Company>, we built X exactly for this!" The disclosure is buried inside a sales line. Rewrite: open with the disclosure, answer the actual question, mention the company once at most, no superlatives.

2. **The "let me know" close.** "Happy to share more about how we solved this — DM me anytime!" This reads as a sales-funnel CTA. Reddit replies don't end with CTAs. End with the takeaway, the caveat, or the question back to the OP. Never with an invitation to take it offline.

3. **The "we" overload.** "We see this all the time" / "We've helped hundreds of teams" / "We solved this by…". Three uses of "we" and the comment reads as a brochure. Use "we" sparingly (at most once, in the disclosure or a single specific anecdote) and "I" or third-person framing elsewhere.

4. **The competitor diss.** Showing up in a competitor-complaint thread to talk about how much better your product is. Even with the disclosure, this is the move that gets accounts banned. Instead: contribute the technique-level take, mention your company once as one of several solutions to consider (not "the solution"), and let the reader make the comparison themselves.

### Example — competitor complaint thread, value-first reply

Input: A thread in r/sales asks "Outreach is getting absurdly expensive — anyone moved off it?"

**Bad (cheesy / pitch-first):**
```
Great post! At <Company>, we built our outreach platform specifically to be a
better alternative to Outreach. Customers save 60% on average and love our
seamless workflow. Happy to DM you a demo link!
```

**Good (value-first, single mention, mandatory disclosure):**
```
Arnaud from gtm-tools here. The Outreach price-vs-value math broke at the same
point for us — somewhere around 80 seats, the per-seat tier jumped while the
features we actually used stayed the same.

What worked for the teams I've talked to who left wasn't the cheapest
alternative — it was unbundling the workflow. The actual jobs Outreach does
(sequence cadencing, deliverability tracking, reply classification, CRM sync)
are four different products' jobs. Outreach charges as if it's one. Replacing
it with three lighter tools is cheaper and the reps usually prefer it.

We use a mix of Lemlist for cadencing + Smartlead for warm-up + our own
classifier on inbound replies. Not advocating that exact mix — depends on
what your reps actually use Outreach for. Audit the actual feature usage
before picking the replacement.
```

What this demonstrates: disclosure in line 1, zero superlatives, one mention of our own classifier as part of a stack (not "the solution"), no DM-me, no pricing comparison, the actionable advice (audit usage before switching) is genuinely useful to anyone reading.

### The no-links rule

Don't include URLs in your reply — not to your blog, product, help center, calculators, landing pages, or competitors. Name primary authorities directly in prose (paper titles, RFC numbers, library names) so readers can search. The opener does the discovery work; the body just has to be useful.

Why the no-links rule: Reddit's link-spam heuristics weight new accounts and self-promotion subs heavily. A link to your own domain in the first ten comments of your account's life can shadowban silently — you keep posting, your replies go invisible. The single exception: OP explicitly asks for a link ("anyone got docs on…?"). Even then, prefer authoritative sources over your marketing surface.

### Length and tone

- **100–300 words.** Comments longer than that get ignored or read as ad copy.
- **Practitioner-to-practitioner**, not corporate. Write like you're explaining to a colleague over Slack.
- **Direct answer in the first body sentence.** Reddit readers scan; bury the answer and they bounce.
- **Vary sentence length.** Monotone rhythm is the strongest AI tell. Mix short with long.

### Anti-AI-writing pass (mandatory before posting)

Scrub the draft against this list. The reason these patterns matter: Reddit commenters and mod tools have learned them as signals, and once you're flagged the rest of your account's history gets re-scrutinized.

- **No em dashes** (—). Use regular dashes or rewrite.
- **No rule-of-three constructions.** "It's clean, fast, and reliable" reads as marketing copy.
- **No negative parallelisms.** "It's not just X, it's Y." "Not only A but B."
- **No `-ing` filler phrases.** "…highlighting the complexity", "…ensuring compliance", "…reflecting broader changes" are almost always deletable.
- **No copula avoidance.** Use *is* / *are* / *has*, not "serves as", "stands as", "represents", "functions as".
- **No AI vocabulary.** *Delve, landscape* (abstract), *tapestry, testament, pivotal, crucial, robust, seamless, intricate, navigate* (figurative), *unlock, foster, underscore, comprehensive, holistic, leverage* (verb).
- **No promotional language.** *Vibrant, powerful, cutting-edge, transformative, game-changer.* Reddit punishes this hardest.
- **No bold for emphasis, no emojis, no bulleted "header: description" lists.**
- **No sycophancy.** Don't open with "Great question!" or "Yeah, this is a common one!".
- **No generic upbeat closer.** Don't end with "hope this helps", "happy to clarify", "good luck out there".
- **No hedge stacking.** "It may potentially possibly be the case that…" → "It's".

**Example 2 — same reply, before and after the scrub:**

Before (AI-tell flavored):
```
Great question! Matrix is a robust, comprehensive solution for self-hosted assistant
bots — it's not just about push notifications, it's about unlocking the full
potential of decentralized chat. By leveraging Element's seamless mobile experience,
you can ensure your bot maintains a consistent presence across devices, highlighting
the power of open protocols.

Hope this helps! Happy to clarify anything 🚀
```

After (scrubbed, same content):
```
Arnaud from gtm-tools here. Matrix push notifications work fine through Element on
mobile — the gotcha is the Megolm device verification on container restarts. If your
bot's olm store isn't persisted to a volume, every Docker restart drops the device
identity and you re-verify from your phone each time.

Fix is to mount the olm store on a host volume. After that, push works like any
other Matrix room.
```

Diff in tells: dropped the sycophantic opener, the rule-of-three ("robust, comprehensive"), the negative parallelism ("not just… but…"), the AI vocabulary (*unlock, seamless, leverage, comprehensive*), the bold-for-emphasis, the emoji, the "hope this helps" closer. Same answer, half the words, doesn't read as bot.

### Posting workflow

```bash
# 1. Read the thread
curl -s -X POST "https://api.gtm-tools.sh/api/v0/get_reddit_post" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"url":"<reddit URL>"}' | jq .

# 2. Apply the should-I-reply gate (Stage 2). If any NO, output SKIP — <reason>.

# 3. Confirm rules (Stage 1 compliance gate)
curl -s -X POST "https://api.gtm-tools.sh/api/v0/get_subreddit_about" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"subreddit":"<sub>"}' | jq .

# 4. Draft. Scrub against the anti-AI list. Reread once.

# 5. Post — URL shape decides top-level vs threaded:
#    Post URL    → top-level comment (thing_id = t3_<post>)
#    Comment URL → threaded reply    (thing_id = t1_<comment>)
curl -s -X POST "https://api.gtm-tools.sh/api/v0/create_reddit_comment" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","url":"<URL>","body":"<First> from <Company> here. ..."}' | jq .

# 6. Upvote the OP and the top-quality comments — genuine engagement signal
curl -s -X POST "https://api.gtm-tools.sh/api/v0/vote_reddit" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","url":"<URL>","direction":"up"}' | jq .
```

## Stage 4 — Follow up

After each reply:

1. **Follow the post** so you know when OP or others respond. `follow_reddit_post` hits an undocumented Reddit endpoint that the mobile/web clients use; if it returns 404 (Reddit pulls or renames it), fall back to polling `get_reddit_post` on a schedule.
2. **Save with a category tag** so the engagement queue stays organized.

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/follow_reddit_post" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","url":"<URL>","follow":true}' | jq .

curl -s -X POST "https://api.gtm-tools.sh/api/v0/save_reddit_thing" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","url":"<URL>","action":"save","category":"engaged_2026w23"}' | jq .
```

**Constraint on `category`:** it's a Reddit Premium feature. Non-Premium accounts get a 403, with the tool translating to a clear "retry without category" error. On free accounts, drop the category argument and track stages out-of-band.

Read the inbox daily, the queue weekly:

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_reddit_inbox" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"username":"<u>","filter":"comment_replies","limit":20}' | jq .

curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_reddit_saved" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","limit":50}' | jq .
```

The reason `markRead` defaults to false on `list_reddit_inbox`: the agent often wants to peek without mutating Reddit's read state (so the human user can still see what's unread when they open Reddit). Pass `markRead: true` only when the user explicitly wants to clear the unread badge.

## Stage 5 — Organize

Three durable structures:

1. **Subscriptions** — `list_reddit_subscriptions` to audit, `subscribe_reddit_subreddit` to add. A real-looking account is subscribed to a mix of organic-interest subs and ICP subs — not only promotional ones.
2. **Custom feeds** — one per ICP / campaign / monitoring theme. `list_reddit_custom_feeds` to inventory, `create_reddit_custom_feed` to add. Retire old feeds (delete or just stop reading) when campaigns end.
3. **Saved categories (Premium only)** — periodically clear `engaged_*` items older than 30 days that didn't convert. Stops the queue from rotting.

## The minimal good-Reddit-agent loop

Putting all five stages together, this is the smallest end-to-end loop that runs daily without burning the account:

```bash
# 1. Read the merged ICP feed sorted by rising
curl ... list_reddit_custom_feed_posts (feed_name=<icp_feed>, sort=rising, limit=50)

# 2. For each promising post:
#    a. get_reddit_post(url)             # full thread
#    b. get_reddit_user(op_username)     # credibility
#    c. apply the 5-question gate        # if any no → SKIP — <reason>
#    d. if pass: get_subreddit_about(sub)  # rules
#    e. draft, scrub for AI patterns, post via create_reddit_comment
#    f. follow_reddit_post + save_reddit_thing with category

# 3. Check the inbox for replies on previous engagements
curl ... list_reddit_inbox (filter=comment_replies, limit=20)

# 4. Read the saved queue, work P0 follow-ups
curl ... list_reddit_saved
```

The hardest part of Reddit isn't the tooling — it's the discipline to skip nine threads to write one genuinely useful reply on the tenth.

## Reference

- Per-tool curl reference with parameter details: [`references/tools-reference.md`](./references/tools-reference.md)
- gtm-tools docs: https://gtm-tools.sh/documentation/core-concepts/reddit-tools
- Outreach guide (the long-form walkthrough this skill is condensed from): https://gtm-tools.sh/documentation/guides/reddit-outreach
- API base: `https://api.gtm-tools.sh/api/v0` (the bare `gtm-tools.sh` is the docs site — POSTing returns 405)
- Auth: `Authorization: Bearer $GTM_TOOLS_API_KEY` (get one via `get_api_key` on `gtm-tools.sh`, or `gtm-tools admin login` from the CLI)
