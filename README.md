# Gods & Mortals - Agent Skill

Play **Gods & Mortals (RoOLZ)** — a multiplayer strategy RPG on the TON blockchain — using AI agents via MCP tools.

8 Divine Kingdoms compete for power across 13-day rounds. This skill teaches any LLM how to play optimally: heisting, training, banking, assaulting, managing guilds, and climbing the leaderboard.

> **AI Agents:** See [AGENT_QUICKSTART.md](AGENT_QUICKSTART.md) for a deterministic onboarding runbook with boot sequence, safe operating rules, and troubleshooting.

## Prerequisites

### 1. Get an Agent Token
1. Open the game via Telegram at [t.me/GodsAndMortals_Bot/app](https://t.me/GodsAndMortals_Bot/app)
2. Go to **Settings > Agent Access > Create Token**
3. Enable scopes: `read`, `combat`, `economy`, `casino`, `social`, `management`, `feedback`
4. Copy the token

### 2. Set the Token

Make the token available as an environment variable. Choose your platform:

**Linux / macOS (bash/zsh)** — add to `~/.bashrc` or `~/.zshrc` for persistence:
```bash
export GODSX_AGENT_TOKEN=your_token_here
```

**Windows (PowerShell)**:
```powershell
$env:GODSX_AGENT_TOKEN = "your_token_here"
```

**Windows (CMD)**:
```cmd
set GODSX_AGENT_TOKEN=your_token_here
```

Or create a `.env` file in your project root:
```
GODSX_AGENT_TOKEN=your_token_here
```

## Installation

### Claude.ai (Web)
Upload the skill directly at [claude.ai](https://claude.ai):
1. Go to **Settings > Skills > Upload Skill**
2. Upload **one** of these:
   - **`gods-and-mortals.skill`** (recommended) — pre-built bundle with SKILL.md + all references
   - **`SKILL.md`** — standalone file (works but won't include reference docs)
3. Configure the MCP server separately in Claude.ai's MCP settings:
   - **URL**: `https://app.roolzgods.com/mcp`
   - **Auth Header**: `Authorization: Bearer ${GODSX_AGENT_TOKEN}`

### Claude Code (CLI)

**Recommended method** — register the MCP server directly:
```bash
claude mcp add godsx-mortals "https://app.roolzgods.com/mcp" \
  -t http -s project \
  -H "Authorization: Bearer $GODSX_AGENT_TOKEN"
```

Then clone this repo so Claude Code can read the skill:
```bash
git clone https://github.com/GodsAndMortals/agent-skill.git
```

**Alternative** — install as a plugin (also registers the MCP server via `.mcp.json`):
```bash
claude plugins install github:GodsAndMortals/agent-skill
```

> **Important: Restart Required**
> After adding the MCP server, **restart your Claude Code session**. MCP servers only connect at startup — tools won't appear until you restart.

### Verify It Works
```bash
claude mcp list
# Should show: godsx-mortals: https://app.roolzgods.com/mcp (HTTP) - Connected
```

Then try:
```
> Play Gods & Mortals for me
> Do heists until my tickets run out
> Check my game status
```

### Cursor
Copy the skill into your Cursor skills directory:
```bash
cp -r skills/gods-and-mortals ~/.cursor/skills-cursor/gods-and-mortals
```

Then configure the MCP server in Cursor settings:
```json
{
  "mcpServers": {
    "godsx-mortals": {
      "type": "http",
      "url": "https://app.roolzgods.com/mcp",
      "headers": {
        "Authorization": "Bearer ${GODSX_AGENT_TOKEN}"
      }
    }
  }
}
```

### Codex / OpenClaw
```bash
cp -r skills/gods-and-mortals ~/.codex/skills/gods-and-mortals
```

### Any MCP-Compatible LLM
Any LLM that supports MCP tools can play. Configure the MCP server:

| Setting | Value |
|---------|-------|
| **Type** | `http` (StreamableHTTP) |
| **URL** | `https://app.roolzgods.com/mcp` |
| **Auth Header** | `Authorization: Bearer ${GODSX_AGENT_TOKEN}` |

Then include the `SKILL.md` file in your system prompt or context to teach the LLM how to play.

### Manual (Any LLM Without MCP)
If your LLM doesn't support MCP but has tool/function calling:
1. Read `references/tool-reference.md` for the complete API
2. Map each MCP tool to an HTTP call to `https://app.roolzgods.com/mcp`
3. Include `SKILL.md` in the system prompt for gameplay strategy

## Prompts for Your Agent

Copy-paste one of these to get your agent playing immediately.

### Quick Prompt

```
Play Gods & Mortals for me. Use the godsx-mortals MCP tools.
Start with get_setup_status, complete any missing setup, then call
get_recommended_actions and execute the highest-priority actions.
Bank all gold before risky actions. If jailed or hospitalized, resolve
that first. Keep playing until I tell you to stop.
```

### Advanced Prompt

```
You are playing Gods & Mortals, a multiplayer strategy RPG, through MCP tools
from the godsx-mortals server.

Setup:
1. Read AGENT_QUICKSTART.md and SKILL.md from this repo for strategy guidance.
2. Follow the boot sequence: get_setup_status → complete setup → get_player_status → get_recommended_actions.
3. Use live MCP tool schema as the source of truth if documentation and schema differ.

Strategy:
- Follow the safe operating rules from AGENT_QUICKSTART.md. Bank gold before risky actions.
  Resolve jail/hospital before anything else. Stop tavern refills at addiction >= 10.
- If my preferred strategy is unclear, ask me what to optimize for:
  fastest gold growth, respect/leaderboard climbing, balanced progression, or low-risk play.
- If I don't answer, default to balanced progression with safe play.
- If a choice is irreversible (class, kingdom), ask me before choosing.
- If I say "just play," continue autonomously and explain your reasoning briefly.

Reporting:
- Tell me whether setup succeeded and what state my character is in.
- After each cycle, summarize actions taken and the next recommended action.
- Flag any blockers, errors, or decisions that need my input.
```

> The agent may ask a few strategy questions up front (class, playstyle, autonomy level), then continue autonomously using safe defaults if you don't respond.

## What the Skill Does

- **Autoplay**: Full gameplay loop — bank, heist, train, assault, collect
- **Resource management**: Addiction tracking, stamina optimization, HP monitoring
- **Phase-aware strategy**: Adapts priorities across 5 round phases (Genesis through Ragnarok)
- **Error recovery**: Auto-handles jail, hospital, cooldowns, insufficient resources
- **Social management**: Guild applications, kingdom upgrades, war participation
- **189 MCP tools**: Complete coverage of all game systems

## Game Overview

| Concept | Detail |
|---------|--------|
| **Platform** | Telegram Web App on TON blockchain |
| **Round length** | 52 TC days = 13 real days |
| **Time scale** | 1 TC day = 6 real hours |
| **Classes** | Slayer, Merchant, Raider, Enchantress, Alchemist, Oracle |
| **Kingdoms** | Olympus, Valhalla, Shogunate, Dynasty of Ra, Jade Empire, Sun Empire, Holy Order, Shadow Realm |
| **Currency** | Gold (in-game), $GODL (premium token) |

## Files

| File | Purpose |
|------|---------|
| **`gods-and-mortals.skill`** | **Ready-to-upload bundle for Claude.ai** (SKILL.md + references + MCP config) |
| `AGENT_QUICKSTART.md` | Deterministic agent onboarding — boot sequence, safe rules, troubleshooting |
| `SKILL.md` | Main skill — gameplay loop, resource rules, error recovery, examples |
| `references/tool-reference.md` | Complete MCP tool catalog (189 tools, 13 categories) |
| `references/resource-management.md` | Addiction, stamina, HP, training, banking, vaults, auctions, academy, achievements |
| `references/phase-strategy.md` | Phase-by-phase strategy with class-specific notes |

> Files are mirrored under `skills/gods-and-mortals/` for installable skill layouts used by supported clients (Cursor, Codex, etc.).

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Tools not appearing after config | **Restart Claude Code session** — MCP servers only connect at startup |
| `405 Not Allowed` | Ensure URL is `app.roolzgods.com/mcp` (not `roolzgods.com/mcp`) |
| `Maximum N concurrent sessions` | Close other sessions using the same token, or create a new token |
| `Missing or invalid agent token` | Check `GODSX_AGENT_TOKEN` env var is set and token hasn't expired |
| `settings.local.json` MCP not loading | Use `claude mcp add` CLI instead of manual JSON editing |
| `export` not recognized (Windows) | Use `$env:GODSX_AGENT_TOKEN = "..."` (PowerShell) or `set GODSX_AGENT_TOKEN=...` (CMD) |

## Links

- **Game**: [roolzgods.com](https://roolzgods.com)
- **MCP Endpoint**: `https://app.roolzgods.com/mcp` (StreamableHTTP)
- **Author**: [PromptEra](https://github.com/PromptEra)

## License

MIT
