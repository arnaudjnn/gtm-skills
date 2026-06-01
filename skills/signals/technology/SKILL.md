---
name: technology
description: Detect whether specific technologies are used on a company's website.
---

# Technology Detection

Detect whether specific technologies (e.g. Zendesk, Intercom, Salesforce) are used on a company's website by crawling their HTML and JS bundles.

## Tools Used

- `signal_technologies_identified` (5 tokens): crawls a website and searches for technology domain references in HTML and JS bundles

See `references/tools-reference.md` for exact commands.

## Workflow

1. **Call `signal_technologies_identified`**
   - Pass the target `domain` and an array of `technologies` to search for
   - Technologies should be domain names (e.g. `"zendesk.com"`, `"intercom.io"`)
   - The tool crawls the site HTML, tries locale variants (/en-us/, /en-gb/), accepts cookie consent, and inspects JS bundles as fallback

2. **Interpret the results**
   - `is_detected` is true if at least one technology was found
   - `technologies_found` lists which ones matched
   - `matches` array has per-technology details with `sample_references` showing where in the source it was found

## Use Cases

- "Does acme.com use Zendesk or Intercom?"
- "Check if this company uses Salesforce"
- "Detect what helpdesk tool mammaly.de is running"
- "Is gymshark.com using DigitalGenius?"
