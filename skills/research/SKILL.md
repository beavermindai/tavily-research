---
name: research
description: Adaptive web research via Tavily. Triggers on "/research", "research this", "research X", "deep dive on", "audit X", "market scan", "compare A vs B", "what's new with X", "find me sources on X". Supports modes - quick (5-15s single fact), medium (30-90s structured findings), deep (3-15min full report saved to disk), compare (head-to-head matrix), news (recency-filtered). Auto-detects mode from query if not specified. Delegates heavy work to tavily-researcher subagent to keep main context clean.
---

# Research Skill

Wrapper for the `tavily-researcher` subagent. Provides flexible web research with mode-based depth control.

## Usage patterns

The user invokes this skill via:
- `/research <topic>` — auto-detect mode
- `/research quick <topic>` — fast single fact
- `/research medium <topic>` — structured findings (default)
- `/research deep <topic>` — full audit, saved to disk
- `/research compare <A> vs <B> [vs <C>...]` — comparison matrix
- `/research news <topic>` — recent items only
- Plain phrasing: "research X", "deep dive on X", "compare A and B", "what's new with X"

## How to invoke

When this skill triggers, you MUST delegate to the `tavily-researcher` subagent via the `Agent` tool. Do not run Tavily calls inline in the main thread — that bloats context.

### Step 1: Parse the request

Extract:
- **Topic(s)** — the entity/entities or question to research
- **Mode** — explicit (`quick`/`medium`/`deep`/`compare`/`news`) or `auto` if not specified
- **Output language** — match the user's language (default match conversation)
- **Output path** — for deep mode, default `~/research/<slug>-<YYYY-MM-DD>.md` unless user specifies
- **Constraints** — recency, geography, included/excluded domains, length target

### Step 2: Build the subagent prompt

The subagent has no memory of this conversation. The prompt must be self-contained. Include:
- Mode (or "infer mode")
- Topic + scope
- Any specific sub-questions to answer
- Output language
- Length target
- Output path (if deep)
- Any domain preferences/exclusions
- Any known constraints (e.g., "client is in UAE — prioritize MENA sources")

### Step 3: Call Agent

Use:
```
Agent(
  description: "<short description>",
  subagent_type: "tavily-researcher",
  prompt: "<full self-contained brief>"
)
```

For background-friendly deep research where you can keep working in parallel, use `run_in_background: true`.

### Step 4: Relay result

Subagent returns synthesis. You forward the substance to the user. For deep mode, confirm the file path and word count.

## Mode auto-detection rules

If the user did not specify a mode:

| Query shape | Mode |
|---|---|
| Single fact, single entity, ≤10 words | `quick` |
| "Tell me about X", "research X", "what is X with detail" | `medium` |
| "Audit X", "deep dive on X", "build a report on X", "market scan", "comprehensive research" | `deep` |
| "Compare A vs B", "A or B", "head-to-head" | `compare` |
| "What's new with X", "recent X", "latest on X", "X news this week" | `news` |
| Multiple entities + multi-criteria | `deep` or `compare` (pick `compare` if user explicitly wants ranking) |

When ambiguous, ask the user one clarifying question before spawning the subagent — it's cheaper than running the wrong mode.

## Output expectations by mode

- **quick**: inline 2-4 sentences + 3-5 URLs in your reply.
- **medium**: structured 400-800 word markdown in your reply.
- **deep**: short summary in your reply + confirmation of file path + word count. Full report on disk.
- **compare**: comparison matrix in your reply.
- **news**: bulleted list of recent items in your reply.

## Anti-patterns

- DO NOT run Tavily calls inline. Always delegate to the subagent.
- DO NOT default to `deep` for everything — it's slow and wasteful for simple questions.
- DO NOT skip mode detection — wrong mode wastes time and tokens.
- DO NOT relay raw subagent output verbatim if it's huge — extract the answer.

## Examples

### Example 1 — quick
User: "What's the current price of gold per oz in USD?"
→ Mode: `quick`
→ Subagent prompt: "Quick mode. Find current gold spot price per troy ounce in USD as of today. Cite source. Inline answer only."

### Example 2 — medium
User: "Research Loyverse POS for a UAE retail client."
→ Mode: `medium`
→ Subagent prompt: "Medium mode. Research Loyverse POS specifically for UAE retail use case. Cover: features, pricing, MENA presence, Arabic UI, API/integrations, multi-location, VAT support, known limitations. 400-800 words structured markdown with citations."

### Example 3 — deep
User: "Build me a market scan of jewelry POS systems available in the UAE."
→ Mode: `deep`
→ Subagent prompt: "Deep mode. Comprehensive market scan of jewelry POS systems available to UAE retailers as of 2026. Cover: 8-15 vendors, pricing, MENA presence, Arabic UI, gold-specific features (weight/karat/gold-rate fix-unfix), goAML/DPMSR integration, 5% VAT compliance, e-invoicing 2026 readiness, multi-location, API quality, customer base. Comparison matrix + ranked top 3 + risk register. 2,000-4,000 words. Save to ~/research/uae-jewelry-pos-scan-<today>.md."

### Example 4 — compare
User: "Compare Lightspeed Retail vs Lightspeed X-Series for a 5-shop jewelry chain."
→ Mode: `compare`
→ Subagent prompt: "Compare mode. Lightspeed Retail vs Lightspeed X-Series (formerly Vend) for a 5-location jewelry retail chain in UAE. Criteria: pricing, multi-location, jewelry features (weight/karat), API, MENA support, Arabic UI, VAT, integrations. Comparison matrix + winner per criterion + overall verdict."

### Example 5 — news
User: "What's new in UAE e-invoicing rules?"
→ Mode: `news`
→ Subagent prompt: "News mode. Recent UAE e-invoicing announcements, FTA updates, ASP accreditation news, deadline changes — last 30 days. Bulleted list with dates + URLs."
