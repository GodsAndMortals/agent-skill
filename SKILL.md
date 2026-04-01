---
name: gods-and-mortals
description: >
  Plays Gods & Mortals (RoOLZ) strategy RPG via MCP tools. Use when user asks to
  "play the game", "autoplay", "manage my kingdom", "do heists", "train stats",
  "check game status", "spend my tickets", or any Gods & Mortals / RoOLZ gameplay
  request. Orchestrates combat, economy, training, social, and casino systems with
  phase-aware strategy for a round-based mythological RPG.
metadata:
  author: PromptEra
  version: 1.0.0
  mcp-server: godsx-mortals
---

# Gods & Mortals Agent Skill

You are playing Gods & Mortals, a multiplayer strategy RPG on the TON blockchain. 8 Divine Kingdoms compete for power across 13-day rounds. You interact with the game via MCP tools from the `godsx-mortals` server.

## Session Start

1. Call `get_setup_status` — checks if class, kingdom, guild are chosen
2. If setup incomplete: `choose_class` (Slayer recommended for ranking), `choose_kingdom` (SHOGUNATE for Slayer)
3. Call `get_recommended_actions` — returns prioritized actions with pre-filled `params` and `paramHints`

## The Optimal Loop

Execute this decision tree every cycle. Actions are priority-ordered — do higher priorities first.

```
EVERY CYCLE:
  1. get_recommended_actions → read playerSummary + actions list
  2. IF jailed → bribe_jail {paymentType: 'gold'}
  3. IF hospitalized → heal_hospital {}
  3b. IF HP < 50% AND NOT hospitalized → tavern_heal {} (costs 500 + 200 x level^2 gold, 2 TC hour cooldown)
  4. IF goldOnHand > 0 → bank_deposit {amount: goldOnHand}
     ALWAYS bank before any risky action. No exceptions.
  5. IF training complete → collect_training {} → start_training {stat, duration: 30}
  5b. After XP-earning actions → check_level_up — level up if threshold reached
  6. IF staminaTickets > 10 AND addiction < 10:
       → use_all_stamina {heistType: from recommended params}
       → bank_deposit the earnings immediately
       → IF stamina depleted AND tickets > 10 → tavern_refill {} → repeat
  6b. collect_all_building_vaults {} (once per TC day — collect vault gold from buildings)
  7. IF staminaTickets <= 10 AND NOT training → start_training {stat, duration: 30}
  8. IF tcDay >= 15 AND HP >= 30% → get_assault_suggestions → preview_assault_target {targetId} → execute_assault {targetId}
  9. collect_companions {} (once per TC day — free passive gold)
  9b. get_achievements {} (periodic progress review — check badge/medal progress)
  10. WAIT for training to complete, then restart from 1
```

## Critical Resource Rules

### Banking (simplest, most impactful)
- Gold on hand is LOST on failed heists and STOLEN during assaults
- Banked gold is 100% safe and earns 2% interest per round
- **ALWAYS bank ALL gold before heists or assaults**

### Addiction (most common agent mistake)
- Each tavern refill adds `0.5 + (addiction x 0.04)` addiction (max 5.0 per refill)
- Penalties: heist success -0.5%/point (max -25%), combat -0.4%/point (max -20%)
- Decays -2.0 per TC day (6 real hours)
- **STOP refilling when addiction >= 10.** Wait for decay. Train instead.
- Below 5 = optimal (+10% training, 2x stamina regen)
- 5-10 = acceptable
- Above 10 = degraded (training and regen penalized)
- Above 20 = critical (stop all tavern use)

### Stamina & Tickets
- Tickets regenerate: 72/TC day (standard class), 120/TC day (Raider)
- Tavern refill: 1 ticket + gold (5,000 + 2,000 x addiction + 150 x addiction^2)
- **Spend tickets on heists FIRST. Train ONLY when tickets <= 10.**
- Keep 10 tickets as emergency buffer

### Buildings & Vaults
- Buildings produce gold into vaults over time
- Vaults must be collected manually with `collect_building_vault` or `collect_all_building_vaults`
- Collect once per TC day alongside companions

### Training
- 30-min sessions: 2,400 primary + 600 secondary stats
- While training: combat and heists are BLOCKED by the server
- **Train ONLY when tickets are depleted (<= 10)**
- **Never let training sit idle — collect and restart immediately**
- Best stat by class: Slayer = MIGHT, all others = WISDOM

### HP Management
- Passive regen: 10% max HP per TC day
- Hospitalized = cannot do ANYTHING. Heal immediately with `heal_hospital`
- **If HP below 30%, skip PvP assaults. Heal first.**

## Using Recommended Actions

`get_recommended_actions` returns:
- `playerSummary`: level, class, kingdom, gold, bank, HP, stamina, tickets, addiction, training status
- `recommendedActions`: priority-sorted array, each with:
  - `tool`: MCP tool name to call
  - `params`: pre-filled required parameters (use directly)
  - `paramHints`: for each param — type, validValues, howToGet
  - `reason`: explains why this action is recommended

**Follow the params directly.** They are computed from your current game state. If you need to adjust (e.g., keep a cash buffer), modify the param before calling.

## Tool Calling Rules

- All gold values are BigInt strings: `"4500000"` not `4500000`
- All tools return `{ success: true, data: ... }` or `{ success: false, error: { code, message, retryable, suggestion } }`
- Required scopes: read, combat, economy, casino, social, management

## Error Recovery

| Error | Recovery |
|-------|----------|
| JAILED | `bribe_jail {paymentType: 'gold'}` — costs 500 gold/sec remaining |
| HOSPITALIZED | `heal_hospital {}` — costs 300 gold/sec remaining |
| TRAINING | Wait, or `collect_training` if complete |
| INSUFFICIENT_GOLD | `bank_withdraw` or earn via heists |
| INSUFFICIENT_STAMINA | `tavern_refill` if addiction < 10, else wait for decay |
| LOW_HP | `tavern_heal {}` — costs 500 + 200 x level^2 gold, 2 TC hour cooldown |
| COOLDOWN | Do other actions, retry later |
| INVALID_INPUT | Check `paramHints` from `get_recommended_actions` |
| SCOPE_DENIED | Token lacks required scope — update in game UI |

## Game Clock

- 1 TC day = 6 real hours
- 1 round = 52 TC days = 13 real days
- Phases: Genesis (1-4), Expansion (5-14), Conquest (15-34), Endgame (35-45), Ragnarok (46-52)

For phase-specific strategy, consult `references/phase-strategy.md`.
For complete tool catalog with params, consult `references/tool-reference.md`.
For deep resource management mechanics, consult `references/resource-management.md`.

## Examples

### "Play the game for me"
1. `get_setup_status` — setup complete
2. `get_recommended_actions` — returns [bank_deposit, use_all_stamina, collect_companions]
3. `bank_deposit {amount: "4500000"}` — protect gold
4. `use_all_stamina {heistType: "VAULT_BREAK"}` — earned 2.3M
5. `bank_deposit {amount: "2300000"}` — bank earnings
6. `collect_companions {}` — earned 150K, bank it
7. Addiction at 3, tickets at 45: `tavern_refill`, repeat heists
8. Tickets at 8: `start_training {stat: "MIGHT", duration: 30}`

### "Check my status"
1. `get_player_status` — return formatted summary to user

### "Do heists until tickets run out"
1. `bank_deposit` all gold
2. Loop: `use_all_stamina` → `bank_deposit` → check addiction
3. If stamina = 0 and addiction < 10: `tavern_refill`, continue
4. If addiction >= 10: stop, report remaining tickets, suggest training

### "Manage my kingdom" (Kingdom Sovereign only)
1. `get_kingdom_info` — check members, treasury, upgrades
2. `set_kingdom_tax {rate: 5}` — adjust tax rate
3. `get_kingdom_upgrades` → `purchase_kingdom_upgrade {upgradeType: "..."}` — invest in upgrades
4. `get_kingdom_members` — review member activity

### "Manage my guild"
1. `get_guild_info` — check level, treasury, members
2. `get_guild_applications` — review pending applications
3. `approve_guild_application {applicationId: "..."}` or `reject_guild_application`
4. `upgrade_guild {upgradeType: "..."}` — invest in guild facilities
5. `set_guild_policy {policy: "APPLY"}` — set membership policy
