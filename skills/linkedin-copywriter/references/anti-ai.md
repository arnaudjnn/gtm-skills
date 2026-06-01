# Anti-AI scrub list

Run every draft against this list before output. Composed from the union of three independent skill catalogs (assafkip/linkedin-brand `voice-check.md`, sergebulaev/linkedin-skills `linkedin-humanizer` rules, kvsdileep/linkedin-writer banned-words). LinkedIn has the most aggressive AI-detection community on social — one tell can kill the post.

## Hard zeros (never allowed)

- **Em dashes** (—). Use commas, periods, or rewrite. Single biggest AI tell.
- **Rule-of-three constructions.** "Clean, fast, and reliable" reads as marketing copy. Use two or four.
- **Negative parallelisms.** "Not just X, it's Y." "Not only A but B." Forbidden.
- **Bold-for-emphasis** in the middle of sentences. LinkedIn's editor strips it; the formatter `**word**` syntax in your output is a tell.
- **Emojis at the end of bullets or as sentence-starters** (unless the user's voice fingerprint uses them).
- **Curly quotes** ("smart quotes"). Straight quotes only.

## Banned vocabulary

### Corporate tells (~40)

leverage, utilize, robust, paradigm, synergy, streamline, empower, delve, comprehensive, crucial, pivotal, innovative, transformative, cutting-edge, groundbreaking, unprecedented, tapestry, realm, catalyst, testament, optimize, foster, underscore, bolster, enhance, revolutionize, spearhead, seamlessly, meticulously, effectively, strategically, ecosystem, landscape, holistic, scalable, disruptive, next-gen, seamless, intricate, navigate (figurative), unlock

### Intensifiers (delete on sight)

deeply, truly, fundamentally, inherently, profoundly, genuinely, absolutely, completely, entirely, thoroughly

### Promotional language

vibrant, powerful, transformative, game-changer, game-changing, paradigm shift, mission-critical, best-in-class, world-class, industry-leading, state-of-the-art

### Filler / hedge

a wide range of, in today's [X], at the end of the day, when it comes to, with that being said, that being said, that said (overused), it's worth noting, it goes without saying

### `-ing` filler phrases

"…highlighting the complexity", "…ensuring compliance", "…reflecting broader changes", "…demonstrating value", "…enabling efficient", "…driving outcomes" — almost always deletable.

### Copula avoidance — use *is* / *are* / *has*

Avoid: "serves as", "stands as", "represents", "functions as", "constitutes", "amounts to".

## Banned phrases

- "Here's the thing:"
- "Let that sink in."
- "In today's [X]" / "In today's fast-paced world"
- "Hope this helps"
- "Happy to clarify"
- "Let me know your thoughts"
- "Game-changer" / "Game-changing"
- "Deep dive" (as noun)
- "Great question!"
- "Yeah, this is a common one!"
- "Thanks for sharing!"
- "What a great point!"
- "Couldn't agree more"
- "100% agree"
- "This!"
- "🚀" / "🔥" / "💯" as a closer

## Sycophancy ban

Don't open replies / comments with praise of the OP. Skip "Great question!", "Love this!", "Yeah, this is a common one!", "Brilliant point!". Get straight to substance.

## Closing ban

Don't end posts/comments with generic upbeat closers:
- "hope this helps"
- "happy to clarify"
- "good luck out there"
- "thoughts?" (as the only ending)
- "what do you think?" (as the only ending — fine if it's a real question)

## Hedge stacking

Replace "It may potentially possibly be the case that…" → "It's".

Single hedge OK; stacked hedges are a tell.

## Mass-noun / abstract overuse

"the AI space", "the LinkedIn landscape", "the marketing ecosystem", "the SaaS realm" — all read as filler. Name the specific thing instead: "Claude", "B2B post engagement", "Hubspot's outbound module".

## Burstiness (force on every draft)

Mix short sentences (<8 words) with long ones (>20 words). All-short reads choppy. All-long reads like AI. Target a Flesch Reading Ease score above 55.

## "Specifics over generics" rule

Every 100 words of draft should contain:
- One specific number (revenue, percentage, count, date)
- One named entity (company, person, tool, product)
- One first-person sensory detail or anecdote
- Either one contradiction OR one vulnerability moment

If the draft is generic enough to fit any company in any industry, it's AI-flavored. Rewrite with concrete details from the user's voice fingerprint.

## Self-check pass

Before output, ask:
1. Could this be confidently posted by anyone in this industry? If yes — too generic, rewrite.
2. Is there a single em dash? If yes — replace.
3. Are any banned words present? Run a regex pass.
4. Does the first sentence under 8 words exist? If no — break a long one.
5. Is there at least one first-person specific detail? If no — add one.

If any answer is no, the draft fails. Rewrite.
