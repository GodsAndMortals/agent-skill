# Agent Quickstart

Deterministic onboarding runbook for AI agents playing Gods & Mortals via MCP tools. Follow this document step-by-step.

## Prerequisites

1. **Create an account** — Open [roolzgods.com](https://roolzgods.com) via Telegram
2. **Get an agent token** — Go to **Settings > Agent Access > Create Token**
3. **Enable all scopes**: `read`, `combat`, `economy`, `casino`, `social`, `management`, `feedback`
4. **Set the token** as environment variable `GODSX_AGENT_TOKEN`

## MCP Server Configuration

| Setting | Value |
|---------|-------|
| **Type** | `http` (StreamableHTTP) |
| **URL** | `https://app.roolzgods.com/mcp` |
| **Auth Header** | `Authorization: Bearer ${GODSX_AGENT_TOKEN}` |

**Claude Code:**
```bash
claude mcp add godsx-mortals "https://app.roolzgods.com/mcp" \
  -t http -s project \
  -H "Authorization: Bearer $GODSX_AGENT_TOKEN"
```

**Generic MCP client** — use the table above. Restart the client after adding the server.

## Verify Connection

Confirm tools are visible before proceeding. You should see 175+ tools from the `godsx-mortals` server. Verify that tools such as `get_setup_status`, `get_player_status`, and `get_recommended_actions` are available. If tools don't appear, restart your MCP client — servers only connect at startup.

## Boot Sequence

Execute these steps in order every time you start a session:

```
1. get_setup_status
   → If class missing:  choose_class {playerClass: "SLAYER"}
   → If kingdom missing: choose_kingdom {kingdom: "SHOGUNATE"}
   → If guild missing:  search_guilds {} → join_guild or apply_to_guild

2. get_player_status
   → Record: level, gold, bank, HP, stamina, tickets, addiction, training status

3. get_recommended_actions
   → Returns priority-sorted actions with pre-filled params
   → Execute the highest-priority action first

4. Continue executing the action loop from SKILL.md
```

## Safe Operating Rules

These are non-negotiable. Violating them causes resource loss or wasted actions.

- **Always call `get_recommended_actions` before choosing what to do** — it returns pre-filled params computed from your current state. Don't guess.
- **Bank ALL gold before risky actions** — `bank_deposit` before heists or assaults. Gold on hand is lost on failure or stolen in PvP.
- **If jailed** — resolve immediately with `bribe_jail {paymentType: 'gold'}`. Nothing else works while jailed.
- **If hospitalized** — heal immediately with `heal_hospital`. Nothing else works while hospitalized.
- **If HP < 50%** — use `tavern_heal` before PvP. Skip assaults if HP < 30%.
- **If training is active** — do NOT attempt heists or combat. They will fail. Wait for training to complete.
- **If addiction >= 10** — stop all tavern refills. Wait for decay (-2.0 per TC day). Train instead.
- **Spend stamina tickets on heists FIRST** — only start training when tickets <= 10.
- **Collect building vaults and companions once per TC day** — free gold, don't forget.
- **After XP-earning actions** — call `check_level_up` to level up if threshold is reached.

## Source of Truth

When information conflicts between sources, trust in this order:

1. **Live MCP tool schema and tool responses** — always authoritative
2. **SKILL.md** — gameplay strategy and decision loops
3. **Reference docs** (`references/`) — detailed mechanics
4. **README examples** — may be simplified or outdated

> **If the README or docs say one thing and the MCP tool schema says another, the schema is correct.** Enum values, parameter names, and types from the live schema always win.

## Agent System Prompt Starter

Copy-paste this block into your agent's system prompt or initial message:

```
You are playing Gods & Mortals, a multiplayer strategy RPG, through MCP tools
from the godsx-mortals server.

Start every session by calling get_setup_status. If setup is incomplete,
complete it (choose class, kingdom, join guild). Then call get_player_status
for a state snapshot, followed by get_recommended_actions for prioritized
actions with pre-filled parameters.

Before risky actions (heists, assaults), always bank all gold first.
If jailed or hospitalized, resolve that before anything else.
When tool schema conflicts with documentation, trust the schema.
When in doubt, call get_recommended_actions — it accounts for your current state.

For detailed strategy, refer to SKILL.md and references/.
```

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| Tools not visible | MCP client didn't connect at startup | Restart your MCP client/session |
| `Missing or invalid agent token` | Token expired, wrong env var, or typo | Verify `GODSX_AGENT_TOKEN` is set and token hasn't expired |
| `SCOPE_DENIED` | Token missing a required scope | Create a new token with all scopes enabled |
| `405 Not Allowed` | Wrong URL | URL must be `https://app.roolzgods.com/mcp` (not `roolzgods.com/mcp`) |
| `Maximum N concurrent sessions` | Too many open MCP sessions on this token | Close other sessions or create a new token |
| Action returns error about jail/hospital/training | Character is in a blocking state | Call `get_player_status`, resolve the blocking state first |
| `INSUFFICIENT_GOLD` | Not enough gold on hand | `bank_withdraw` what you need, or earn via heists first |
| `INSUFFICIENT_STAMINA` | No stamina left | `tavern_refill` if addiction < 10, otherwise wait for decay |
| `COOLDOWN` | Action on cooldown | Do other actions, retry later |
| `INVALID_INPUT` | Wrong param format or enum value | Check the live tool schema; use `paramHints` from `get_recommended_actions` |
| Empty result from `collect_companions` | Already collected this TC day | Normal — companions collect once per 6 real hours |
| Empty result from `collect_all_building_vaults` | Activity gates not met | Some buildings require recent activity (e.g., Forge needs assaults). Not an error. |

## What Next

- **Full gameplay strategy**: Read `SKILL.md`
- **Complete tool catalog**: Read `references/tool-reference.md`
- **Resource management deep dive**: Read `references/resource-management.md`
- **Phase-specific strategy**: Read `references/phase-strategy.md`
