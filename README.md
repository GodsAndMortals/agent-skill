# Gods & Mortals - Agent Skill

Play **Gods & Mortals (RoOLZ)** — a multiplayer strategy RPG on the TON blockchain — using AI agents via MCP tools.

8 Divine Kingdoms compete for power across 13-day rounds. This skill teaches any LLM how to play optimally: heisting, training, banking, assaulting, managing guilds, and climbing the leaderboard.

## Prerequisites

### 1. Get an Agent Token
1. Open the game at [roolzgods.com](https://roolzgods.com) via Telegram
2. Go to **Settings > Agent Access > Create Token**
3. Enable scopes: `read`, `combat`, `economy`, `casino`, `social`, `management`
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
   - **Auth Header**: `Authorization: Bearer <your_token>`

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
| **Auth Header** | `Authorization: Bearer <your_token>` |

Then include the `SKILL.md` file in your system prompt or context to teach the LLM how to play.

### Manual (Any LLM Without MCP)
If your LLM doesn't support MCP but has tool/function calling:
1. Read `references/tool-reference.md` for the complete API
2. Map each MCP tool to an HTTP call to `https://app.roolzgods.com/mcp`
3. Include `SKILL.md` in the system prompt for gameplay strategy

## What the Skill Does

- **Autoplay**: Full gameplay loop — bank, heist, train, assault, collect
- **Resource management**: Addiction tracking, stamina optimization, HP monitoring
- **Phase-aware strategy**: Adapts priorities across 5 round phases (Genesis through Ragnarok)
- **Error recovery**: Auto-handles jail, hospital, cooldowns, insufficient resources
- **Social management**: Guild applications, kingdom upgrades, war participation
- **175 MCP tools**: Complete coverage of all game systems

## Game Overview

| Concept | Detail |
|---------|--------|
| **Platform** | Telegram Web App on TON blockchain |
| **Round length** | 52 TC days = 13 real days |
| **Time scale** | 1 TC day = 6 real hours |
| **Classes** | Slayer, Merchant, Raider, Enchantress, Alchemist, Oracle |
| **Kingdoms** | Olympus, Asgard, Shogunate, Pharaoh, Aztec, Celtic, Roman, Persian |
| **Currency** | Gold (in-game), $GODL (premium token) |

## Files

| File | Purpose |
|------|---------|
| **`gods-and-mortals.skill`** | **Ready-to-upload bundle for Claude.ai** (SKILL.md + references + MCP config) |
| `SKILL.md` | Main skill — gameplay loop, resource rules, error recovery, examples |
| `references/tool-reference.md` | Complete MCP tool catalog (175 tools, 13 categories) |
| `references/resource-management.md` | Addiction, stamina, HP, training, banking, vaults, auctions, academy, achievements |
| `references/phase-strategy.md` | Phase-by-phase strategy with class-specific notes |

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
