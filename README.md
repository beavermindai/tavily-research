# tavily-research

Adaptive web research plugin for Claude Code and Claude Cowork. Adds a `/research` skill with mode-based depth control, backed by an isolated `tavily-researcher` subagent so heavy research never bloats your main context.

## What's inside

- **`/research` skill** — slash trigger that parses your query, picks a mode (or auto-detects), and delegates to the subagent.
- **`tavily-researcher` subagent** — runs in isolated context, dispatches the right Tavily tool tier per mode, returns synthesized findings with citations.

## Modes

| Mode | Latency | Use case |
|---|---|---|
| `quick` | 5-15s | single fact, one entity |
| `medium` | 30-90s | structured findings, vendor lookup (default) |
| `deep` | 3-15min | full audit, saved to disk as markdown |
| `compare` | 1-5min | head-to-head matrix |
| `news` | 5-15s | recency-filtered, last day/week |

## Requires

- Tavily MCP server connected: `tavily-remote-mcp` (configured in your Claude environment, not shipped by this plugin).
- Get a Tavily API key at https://app.tavily.com/

## Install — Claude Code

User-level install (fastest):

```bash
git clone https://github.com/beavermindai/tavily-research.git ~/tavily-research
mkdir -p ~/.claude/agents ~/.claude/skills/research
cp ~/tavily-research/agents/tavily-researcher.md ~/.claude/agents/
cp ~/tavily-research/skills/research/SKILL.md ~/.claude/skills/research/
```

Then restart Claude Code. Verify with `/help` — `research` should be listed.

## Install — Claude Cowork

In a Cowork session, install plugin from this repo URL via the Cowork plugin manager. Tavily MCP must be added separately in Cowork's MCP settings.

## Usage

```
/research quick current gold spot price USD
/research medium Loyverse POS UAE
/research deep UAE jewelry POS market scan
/research compare Lightspeed Retail vs Lightspeed X-Series
/research news UAE e-invoicing rules
```

Plain phrasing also triggers: "research X", "deep dive on X", "compare A and B", "what's new with X".

## Output

- Quick / medium / compare / news → inline reply.
- Deep → markdown file saved to `~/research/<slug>-<YYYY-MM-DD>.md` plus a short summary in chat.

## Customizing

Edit `agents/tavily-researcher.md` to tune:
- Default `country` (set to your primary geography for fewer mis-targeted results).
- Domain allow/exclude lists.
- Length targets per mode.
- Recency cutoffs.

Edit `skills/research/SKILL.md` to tune:
- Auto-detection rules.
- Output expectations per mode.
- Default save paths.

## License

MIT — use, fork, ship.
