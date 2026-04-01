# Gods & Mortals - Claude Code Skill

Play **Gods & Mortals (RoOLZ)** — a multiplayer strategy RPG on the TON blockchain — directly from Claude Code via MCP tools.

8 Divine Kingdoms compete for power across 13-day rounds. This skill teaches Claude how to play optimally: heisting, training, banking, assaulting, managing guilds, and climbing the leaderboard.

## Installation

### Claude Code
```bash
claude plugins install github:PromptEra/gods-and-mortals-skill
```

### Manual
Copy this directory into your Claude Code plugins folder.

## Setup

### 1. Get an Agent Token
1. Open the game at [roolzgods.com](https://roolzgods.com) via Telegram
2. Go to **Settings > Agent Access > Create Token**
3. Enable scopes: `read`, `combat`, `economy`, `casino`, `social`, `management`

### 2. Configure the MCP Server
The plugin auto-configures the MCP connection via `.mcp.json`. Set your token:

```bash
export GODSX_AGENT_TOKEN=your_token_here
```

Or add it to your `.claude/settings.json`:
```json
{
  "env": {
    "GODSX_AGENT_TOKEN": "your_token_here"
  }
}
```

### 3. Play

```
> Play Gods & Mortals for me
> Do heists until my tickets run out
> Check my game status
> Manage my kingdom
> Train my stats optimally
```

## What the Skill Does

- **Autoplay**: Full gameplay loop — bank, heist, train, assault, collect
- **Resource management**: Addiction tracking, stamina optimization, HP monitoring
- **Phase-aware strategy**: Adapts priorities across 5 round phases (Genesis through Ragnarok)
- **Error recovery**: Auto-handles jail, hospital, cooldowns, insufficient resources
- **Social management**: Guild applications, kingdom upgrades, war participation
- **145+ MCP tools**: Complete coverage of all game systems

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
| `skills/gods-and-mortals/SKILL.md` | Main skill — gameplay loop, rules, examples |
| `skills/gods-and-mortals/references/tool-reference.md` | Complete MCP tool catalog (145+ tools) |
| `skills/gods-and-mortals/references/resource-management.md` | Deep dive on addiction, stamina, HP, training, banking |
| `skills/gods-and-mortals/references/phase-strategy.md` | Phase-by-phase strategy with class-specific notes |

## Links

- **Game**: [roolzgods.com](https://roolzgods.com)
- **MCP Endpoint**: `https://roolzgods.com/mcp` (StreamableHTTP)
- **Author**: [PromptEra](https://github.com/PromptEra)

## License

MIT
