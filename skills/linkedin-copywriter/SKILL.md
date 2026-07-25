---
name: linkedin-copywriter
description: Ghostwrite LinkedIn posts, comments, connection-request notes, and DMs in the user's voice without AI tells. Use this skill whenever the user asks to write / draft / ghostwrite a LinkedIn post, comment on a specific LinkedIn post URL, react or reply to a LinkedIn post, write a connection-request note, DM a LinkedIn user, repurpose a blog post / talk / podcast into LinkedIn content, write in someone's voice, write a "personal brand" post, plan a week of LinkedIn content, or mentions LinkedIn copywriting / ghostwriting / personal branding — even if they don't say "skill" or name the underlying tools. Reads the user's recent posts to learn their voice, fetches context for the target post or profile, drafts in the user's style, scrubs AI tells (em dashes, banned vocab, intensifiers), and returns copy-paste-ready output. Never auto-publishes posts or comments (LinkedIn's write API is gated); only DMs and connection invitations send through the API after explicit approval.
---

# LinkedIn Copywriter

Draft LinkedIn posts, comments, connection-request notes, and DMs that read like the user wrote them — not like AI. This skill exists because LinkedIn has the most aggressive AI-detection community on social: a single em dash, one rule-of-three triplet, or the word "leverage" gets your post downranked and your account quietly marked as spam. The competing skills surveyed (sergebulaev/linkedin-skills, kvsdileep/linkedin-writer, assafkip/linkedin-brand) all converge on the same three pillars: voice ingestion, anti-AI scrub, draft-then-paste. This skill bakes all three in.

## When to engage

Trigger phrases — invoke for any of:

- "write a LinkedIn post about <topic>"
- "draft a LinkedIn comment for <URL>" / "react to this LinkedIn post"
- "ghostwrite for me" / "write in my voice"
- "comment on this LinkedIn post"
- "write a connection request to <person>"
- "DM <person> about <topic>"
- "repurpose this blog/talk into a LinkedIn post"
- "plan my LinkedIn week"
- mentions of LinkedIn copywriting, ghostwriting, personal branding, content strategy

Don't invoke for: pure prospecting (use the LinkedIn lookup tools at gtm-tools.sh directly), CRM updates, or non-LinkedIn channels.

## The four modes

Every request maps to exactly one `mode`. Determine it from the user's request before doing anything else; if ambiguous, ask. Each mode has its own char budget, structure, and tool fan-out:

| Mode | What it produces | Char limit (hard / sweet) | Publishes via |
|---|---|---|---|
| `post` | Top-level post on the user's own feed | 3,000 / 1,800–2,800 | Copy-paste (user pastes into LinkedIn) |
| `comment` | Comment on someone else's post | 1,250 / 200–350 | Copy-paste |
| `invitation` | Connection-request note | 300 hard | `send_linkedin_invitation` after approval |
| `dm` | Direct message | No hard cap, target <800 | `send_linkedin_message` after approval |

Why posts and comments are copy-paste only: LinkedIn's write API is gated to approved Marketing Developer Platform partners, and browser-extension automation is fragile under LinkedIn's anti-automation heuristics. Every mature competitor ships draft-only for posts/comments. Friction is minimal — the user is already in LinkedIn. The value of "I shipped a perfect 2,400-char post" is identical whether posted via API or pasted.

## Prerequisites

Before invoking any tool, confirm:

1. **gtm-tools is connected.** API endpoint: `https://api.gtm-tools.sh/api/v0`. Auth: `Authorization: Bearer $GTM_TOOLS_API_KEY`. (The bare `gtm-tools.sh` is the docs site — POSTing returns 405. Always use the `api.` subdomain.)
2. **A LinkedIn session is in the pool** if the mode will send (invitation/dm) or if voice ingestion needs to read personal-data tools. Call `list_connected_linkedin_accounts`. Each `ready` row has a `linkedin_username` — pass as `senderUsername` to write tools. If empty: `curl -fsSL https://api.gtm-tools.sh/extension/install.sh | bash`, then Connect in the popup.

## Universal workflow

Regardless of mode, follow this loop:

### 1. Voice ingestion (once per session)

Pull the user's last 15–30 LinkedIn posts to learn their voice. Cache the fingerprint for the conversation.

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_user_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"username":"<user_linkedin_username>","limit":25}' | jq .
```

Extract from the response: average sentence length, opener patterns, signature phrasings, content pillars, formatting habits (use of line breaks, bullet markers, emojis or not), characteristic words and word avoidance. Do not copy specific phrases — capture the register.

If the user can't be read (no LinkedIn session connected), ask the user for 3–5 pasted reference posts before drafting.

### 2. Mode-specific context fetch

`post` mode — optional. If the user gave a URL to react to or named a company to research:

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/get_linkedin_post" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"post_url":"<URL>"}' | jq .

curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_linkedin_company_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"domain":"<company.com>","limit":10}' | jq .
```

`comment` mode — REQUIRED. Read the OP post + existing top comments so the draft doesn't duplicate what's already there:

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/get_linkedin_post" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"post_url":"<URL>"}' | jq .

curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_linkedin_post_comments" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"post_url":"<URL>","limit":15}' | jq .
```

`invitation` / `dm` mode — REQUIRED. Fetch the recipient's profile + recent posts so the message has a specific hook:

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/get_linkedin_profile" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"profile_url":"https://linkedin.com/in/<username>"}' | jq .

curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_user_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"username":"<target>","limit":5}' | jq .
```

### 3. Reaction-Mode Gate (comment mode only)

Before drafting a comment, extract the OP's claims verbatim and show them to the user for confirmation. This catches misreadings before they become a public, downvotable comment.

```
Claims from <OP's name>'s post:
1. "<verbatim claim 1>"
2. "<verbatim claim 2>"
3. "<verbatim claim 3>"

Reacting to which one(s)? Or all of them?
```

Wait for the user to confirm before continuing to step 4. Do not draft on inferred claims.

### 4. Draft using the user's voice + mode structure

`post` structure — Hook (≤210 chars, fits above LinkedIn's "…see more" fold) → Problem → Framework / insight → Action → Engagement question. Use the 3-step hook formula (Context Lean-In → Scroll Stop Interjection → Contrarian Snapback). See [`references/hooks.md`](./references/hooks.md) for 10 templates.

`comment` structure — One of seven templates (see [`references/comment-templates.md`](./references/comment-templates.md)):
1. Missing-Piece — "The piece I'd add: ..."
2. Answer-the-Closing-Question — directly answer what OP closed with
3. Data-First — "We ran this in production at scale X. Result Y."
4. Practitioner Observation — "What this matches in my work: ..."
5. Counter-with-Concession — "Agreed on A; the place I'd push back is B."
6. Quotable Reframe — restate OP's idea in a sharper one-liner
7. Ask-a-Sharper-Question — pose the next-level question OP's post doesn't answer

`invitation` structure — One sentence acknowledging a specific thing in their profile or recent activity; one sentence on why connecting matters. 300 chars hard.

`dm` structure — Hook from their recent post or profile detail → specific thing you noticed → one-sentence ask (call, intro, opinion). Target <800 chars.

### 5. Anti-AI scrub (mandatory before output)

Run the draft against [`references/anti-ai.md`](./references/anti-ai.md). LinkedIn has the most aggressive AI-detection community on social — one tell can kill the post.

Hard zeros:
- Zero em dashes (—). Use commas, periods, or rewrite.
- Zero banned words from `references/anti-ai.md` (~55 corporate-tells: *leverage, utilize, robust, paradigm, synergy, streamline, empower, delve, comprehensive, crucial, pivotal, innovative, transformative, cutting-edge, groundbreaking, unprecedented, tapestry, realm, catalyst, testament, optimize, foster, underscore, bolster, enhance, revolutionize, spearhead, seamlessly, meticulously, effectively, strategically, ecosystem, landscape, holistic, scalable, disruptive, next-gen* + intensifiers *deeply, truly, fundamentally, inherently*).
- Zero rule-of-three constructions. "Clean, fast, and reliable" reads as marketing copy.
- Zero "Here's the thing:" / "Let that sink in." / "In today's [X]" / "Game-changer" / "Deep dive".

Then run the break + add pass:
- **Break:** force burstiness. Mix short sentences (<8 words) with longer ones. Monotone rhythm is the strongest AI tell.
- **Add:** one specific number per 100 words; one named entity; one first-person sensory detail; one contradiction or vulnerability moment.

### 6. Char count check

Count the draft. Show the count next to it: `"2,847 / 3,000"`. If over the hard limit, cut — don't trim adjectives, cut whole sentences. Prefer ~1,800–2,800 for posts (sweet spot for the algorithm).

### 7. Approval card → publish path

Present the draft as a fenced plain-text block. No "Here's your post:" preamble. Just the draft + char count + the approval prompt.

```
[draft]

2,847 / 3,000 characters

Ship this, edit, or rewrite?
```

On approval:

- `post` / `comment` → **copy-paste mode.** Output the draft as a fenced code block ready for pasting. Optionally save to `linkedin-<mode>-<YYYYMMDD-HHMMSS>.txt` in the working directory.
- `invitation` → call `send_linkedin_invitation` only after the user says yes explicitly:
  ```bash
  curl -s -X POST "https://api.gtm-tools.sh/api/v0/send_linkedin_invitation" \
    -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
    -d '{"senderUsername":"<u>","recipientUsername":"<target>","message":"<draft>"}' | jq .
  ```
- `dm` → call `send_linkedin_message` only after the user says yes explicitly:
  ```bash
  curl -s -X POST "https://api.gtm-tools.sh/api/v0/send_linkedin_message" \
    -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
    -d '{"senderUsername":"<u>","recipientUsername":"<target>","message":"<draft>"}' | jq .
  ```

Never auto-call a send tool without an explicit "yes, send" / "ship it" / "send it" from the user. "Looks good" is not approval to send — it's approval to copy-paste.

## Output format conventions

- Plain text only. No markdown headings, no bold, no italics. LinkedIn's editor strips most of it anyway, and AI-tell detectors flag formatted output.
- Double line-breaks between paragraphs (LinkedIn renders single newlines as no-break).
- No emojis unless the user's voice fingerprint shows they use them. If they do, match their frequency, not maximize.
- 0–2 hashtags max, at the end, lowercase, no spaces. Higher hashtag counts hurt reach in 2026.
- Links go in the first comment, not the post body. Posts with external links are downranked.

## Anti-patterns this skill prevents

- "Hope this helps!" / "Happy to clarify" / "Let me know your thoughts" closers
- "Great question!" / "Yeah, this is a common one!" sycophantic openers
- Heavy bold for emphasis
- "Here's the thing:" → "Three things matter:" → 1. 2. 3. → emoji
- The em dash sandwich ("Matrix — open source — wins")
- Marketing tagline closers ("The future is open source 🚀")

If any of these survive the scrub, the draft fails. Rewrite.

## Example — `comment` mode end-to-end

User: "Comment on this for me: https://linkedin.com/posts/some-vp-of-cx_..."

Agent flow:

1. Voice ingestion: `list_user_posts(username="arnaudjnn", limit=25)`. Notes: short paragraphs, contrarian openers, no emojis, signature phrase "the place I'd push back".
2. Context fetch: `get_linkedin_post(post_url)` + `list_linkedin_post_comments(post_url, limit=15)`.
3. Reaction-Mode Gate:
   ```
   Claims from <OP>'s post:
   1. "AI agents will replace tier-1 support by 2027."
   2. "Companies still using human-first CX will lose market share."
   3. "Knowledge bases are now a liability, not an asset."

   Reacting to which one(s)?
   ```
   User: "the third one."
4. Draft against Counter-with-Concession template:
   ```
   Agreed AI agents reshape tier-1, but calling knowledge bases a liability
   inverts the cause. Bases aren't dead, they're just no longer the
   user-facing surface. The agent reads from one. The KB stops being a
   help-center page and starts being the agent's spine. That's a promotion,
   not a death sentence.
   ```
5. Anti-AI scrub: no em dashes ✓. No banned words ✓. Vary sentence length ✓.
6. Char count: 312 — well under 350 sweet spot ✓.
7. Approval card:
   ```
   312 / 1,250 characters

   Ship this, edit, or rewrite?
   ```
8. User: "Ship it." → Output as fenced plain-text for copy-paste. (Comments are copy-paste mode, not API send.)

## Reference

- Anti-AI scrub list (banned words + phrases): [`references/anti-ai.md`](./references/anti-ai.md)
- Hook templates (10 patterns): [`references/hooks.md`](./references/hooks.md)
- Comment templates (7 patterns): [`references/comment-templates.md`](./references/comment-templates.md)
- gtm-tools LinkedIn docs: https://gtm-tools.sh/socials-tools
- API: `https://api.gtm-tools.sh/api/v0` — auth via `Authorization: Bearer $GTM_TOOLS_API_KEY`
