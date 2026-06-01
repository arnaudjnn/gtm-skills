---
name: posts
description: Research LinkedIn post content, engagement, and company employee activity.
---

# LinkedIn Posts

Research LinkedIn post content, analyze engagement, and monitor what a company's employees are talking about. Use this for content intelligence, prospect research, and engagement tracking.

## Tools Used

- `get_linkedin_post` (2 tokens): single post content with social counts (likes, comments, shares)
- `list_user_posts` (5 tokens): recent posts from a specific user
- `list_linkedin_saved_posts` (10 tokens): a user's saved posts (requires connected account username)
- `list_linkedin_post_reactions` (5 tokens): who reacted to a post and reaction types
- `list_linkedin_post_comments` (5 tokens): comments and replies on a post
- `list_linkedin_company_posts` (5 tokens): posts from a company's LinkedIn page with engagement metrics
- `list_linkedin_company_employees_posts` (80 tokens): recent posts from a company's employees

See `references/tools-reference.md` for exact commands.

## Workflow

The workflow depends on the use case:

### Single Post Analysis
1. Call `get_linkedin_post` with the post URL or URN
2. Summarize content, author, and social counts

### User Feed Analysis
1. Call `list_user_posts` with the username
2. Identify themes, posting frequency, and top-performing content

### Engagement Deep-Dive
1. Call `list_linkedin_post_reactions` and `list_linkedin_post_comments` in parallel
2. Analyze who is engaging: look for prospects, decision-makers, or competitors
3. Summarize engagement patterns

### Company Page Posts
1. Call `list_linkedin_company_posts` with the company URL or slug
2. See what the company is officially publishing: announcements, product updates, events

### Company Employee Content Intelligence
1. Call `list_linkedin_company_employees_posts` with the company URL or slug
2. Identify what employees are posting about: product launches, hiring, events, thought leadership
3. Surface notable posts with high engagement

## Use Cases

- "What did micheltricot post recently?"
- "Who engaged with this post?"
- "What are Vercel employees talking about?"
- "Analyze engagement on this post"
- "Show me saved posts from my connected account"
