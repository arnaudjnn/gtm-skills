# Reddit Tools API Reference

Complete reference for all 21 tools at `https://api.gtm-tools.sh/api/v0`.

## Common pattern

Every call is a Bash + curl POST:

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/TOOL_NAME" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" \
  -d '{...arguments...}' | jq .
```

`$GTM_TOOLS_API_KEY` must be set — get one via `get_api_key` on `gtm-tools.sh` or `gtm-tools admin login` from the CLI.

> The bare `gtm-tools.sh` is the docs site (returns 405 on POST). Use the `api.` subdomain. ✓

---

## Account

### list_connected_reddit_accounts

List Reddit accounts pooled by your workspace via the browser extension. **0 tokens.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_connected_reddit_accounts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" -d '{}' | jq .
```

Response: `{accounts: [{reddit_username, connected_by, last_synced, status}]}`. Use `reddit_username` as `senderUsername` for write tools.

---

## Discover

### search_reddit_posts

Search posts by keyword, optionally restricted to a subreddit. **2 tokens.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/search_reddit_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"query":"matrix self hosted","subreddit":"selfhosted","sort":"relevance","time_window":"month","limit":25}' | jq .
```

Params: `query` (required), `subreddit?`, `sort?` (`relevance|hot|new|top|comments`, default `relevance`), `time_window?` (`hour|day|week|month|year|all`, default `all`), `limit?` (1–100, default 25), `after?` (pagination cursor).

### list_subreddit_posts

List posts from a subreddit by sort order. **1 token.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_subreddit_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"subreddit":"SaaS","sort":"rising","limit":25}' | jq .
```

Params: `subreddit` (required), `sort?` (`hot|new|top|rising|controversial|best`, default `hot`), `time_window?` (only for `top`/`controversial`), `limit?`, `after?`.

### search_reddit_subreddits

Search subreddits by name or description. **1 token.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/search_reddit_subreddits" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"query":"agent automation","sort":"relevance","limit":25}' | jq .
```

Params: `query` (required), `sort?` (`relevance|activity`), `limit?`, `include_nsfw?` (default false), `after?`.

### get_subreddit_about

Get a subreddit's metadata + rules. **Call before posting in any new subreddit.** **1 token.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/get_subreddit_about" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"subreddit":"selfhosted"}' | jq .
```

Response: `{about: {subscribers, active_user_count, submission_type, allow_images, allow_polls, restrict_posting, ...}, rules: [{name, description, applies_to: "all|link|comment", priority}]}`.

---

## Evaluate

### get_reddit_post

Get a Reddit post and its full comment tree. **1 token.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/get_reddit_post" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"url":"https://www.reddit.com/r/SaaS/comments/1abcde/some_slug/"}' | jq .
```

Accepted URLs: `/r/<sub>/comments/<post>/<slug>/`, `/r/<sub>/comments/<post>/<slug>/<commentId>/` (comment URL resolves to the same post), `old.reddit.com`, `redd.it/<post>`. Each comment includes a `url` permalink ready for `create_reddit_comment`.

### get_reddit_user

Get a user's karma, account age, and verified flags (credibility check). **1 token.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/get_reddit_user" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"username":"rlnerd"}' | jq .
```

Response: `{user: {username, account_age_days, link_karma, comment_karma, total_karma, has_verified_email, is_employee, is_gold, is_mod, verified, bio}}`.

### list_reddit_user_posts

List a user's recent posts and/or comments. **1 token.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_reddit_user_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"username":"rlnerd","type":"overview","sort":"new","limit":25}' | jq .
```

Params: `username` (required), `type?` (`submitted|comments|overview`, default `overview`), `sort?` (`new|hot|top`, default `new`), `time_window?`, `limit?`, `after?`.

---

## Engage

### create_reddit_comment

Reply to a Reddit post or comment. **5 tokens.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/create_reddit_comment" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","url":"https://www.reddit.com/r/...","body":"<First> from <Company> here. ..."}' | jq .
```

URL shape decides reply type: **post URL** → top-level comment (`thing_id = t3_<post>`); **comment URL** → threaded reply (`thing_id = t1_<comment>`).

### create_reddit_post

Submit a new post to a subreddit (text or link). **5 tokens.**

```bash
# Text post
curl -s -X POST "https://api.gtm-tools.sh/api/v0/create_reddit_post" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","subreddit":"SaaS","title":"<title>","body":"<markdown>"}' | jq .

# Link post
curl -s -X POST "https://api.gtm-tools.sh/api/v0/create_reddit_post" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","subreddit":"SaaS","title":"<title>","linkUrl":"<url>"}' | jq .
```

Params: `senderUsername`, `subreddit` (`r/` prefix stripped), `title` (max 300), `body?` XOR `linkUrl?`, `nsfw?`, `spoiler?`, `sendReplies?` (default true). To post on your own profile, use `subreddit: "u_<username>"`.

### vote_reddit

Upvote, downvote, or clear a vote. **1 token.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/vote_reddit" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","url":"https://www.reddit.com/r/...","direction":"up"}' | jq .
```

`direction`: `up` (+1), `down` (-1), `none` (clear). Works on both post and comment URLs.

### send_reddit_message

DM via Reddit Chat (Matrix-based — works for every account). **5 tokens.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/send_reddit_message" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","recipientUsername":"<u2>","message":"<text>"}' | jq .
```

Response: `{status: "ok", event_id, room_id, recipient_matrix_id}`. Reddit Chat doesn't render Markdown — newlines work, most other formatting renders literally.

---

## Follow up

### follow_reddit_post

Follow / unfollow a post for new-comment notifications. **Best-effort — uses Reddit's undocumented `/api/follow_post` endpoint.** **1 token.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/follow_reddit_post" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","url":"<URL>","follow":true}' | jq .
```

If Reddit returns 404, fall back to polling `get_reddit_post` on a schedule.

### save_reddit_thing

Save (or unsave) a post / comment to the engagement queue. **1 token.**

```bash
# Save (free)
curl -s -X POST "https://api.gtm-tools.sh/api/v0/save_reddit_thing" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","url":"<URL>","action":"save"}' | jq .

# Save with category (Reddit Premium only — non-Premium accounts get 403)
... -d '{"senderUsername":"<u>","url":"<URL>","action":"save","category":"p0_reply"}' | jq .
```

### list_reddit_saved

List saved posts and comments. **1 token.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_reddit_saved" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","limit":50}' | jq .
```

Params: `senderUsername`, `type?` (`links|comments`, default both), `limit?`, `after?`.

### list_reddit_inbox

Read DMs, comment replies, post replies, mentions. **1 token.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_reddit_inbox" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"username":"<u>","filter":"comment_replies","limit":20}' | jq .
```

`filter`: `all` (default), `unread`, `messages` (DMs), `comment_replies`, `post_replies`, `mentions`. `markRead?` defaults to `false` so reads are non-mutating; pass `true` to flip items to read on fetch.

---

## Organize

### subscribe_reddit_subreddit

Bulk subscribe / unsubscribe to subreddits. **1 token.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/subscribe_reddit_subreddit" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","subreddits":["selfhosted","privacy","Matrix"],"action":"sub"}' | jq .
```

`action`: `sub` or `unsub`. Accepts array of subreddit names.

### list_reddit_subscriptions

List subscribed subreddits. **1 token.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_reddit_subscriptions" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","limit":100}' | jq .
```

### list_reddit_custom_feeds

List the connected account's custom feeds (multireddits). **1 token.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_reddit_custom_feeds" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>"}' | jq .
```

### list_reddit_custom_feed_posts

List posts from a custom feed, merged across subreddits. **1 token.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/list_reddit_custom_feed_posts" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{"senderUsername":"<u>","feed_name":"icp_feed","sort":"rising","limit":50}' | jq .
```

Params: `senderUsername`, `feed_name` (required), `sort?` (`hot|new|top|rising`), `time_window?`, `limit?`, `after?`.

### create_reddit_custom_feed

Create a custom feed from a list of subreddits. **5 tokens.**

```bash
curl -s -X POST "https://api.gtm-tools.sh/api/v0/create_reddit_custom_feed" \
  -H "Authorization: Bearer $GTM_TOOLS_API_KEY" -H "Content-Type: application/json" \
  -d '{
    "senderUsername":"<u>",
    "name":"icp_feed",
    "subreddits":["selfhosted","privacy","Matrix"],
    "visibility":"private",
    "description":"ICP monitoring feed"
  }' | jq .
```

**`name` constraint:** `^[A-Za-z0-9_]+$` (letters, digits, underscores only — no dashes; max 50 chars). Reddit's multireddit URL scheme reserves dashes. Use `display_name?` for the human-readable label.

`visibility?`: `private` (default), `public`, `hidden`. `description?`: Markdown.
