---
name: gods-and-mortals
description: >
  Plays Gods & Mortals (RoOLZ) strategy RPG via MCP tools. Use when user asks to
  "play the game", "autoplay", "manage my kingdom", "do heists", "train stats",
  "check game status", "spend my tickets", or any Gods & Mortals / RoOLZ gameplay
  request. Covers combat, economy, training, social, casino, drug trading,
  hunting, sabotage, districts, raids, wars, and endgame systems for a
  round-based mythological RPG.
metadata:
  author: PromptEra
  version: 1.2.0
  mcp-server: godsx-mortals
---

# Gods & Mortals Agent Skill

Gods & Mortals (RoOLZ) is a multiplayer strategy RPG on the TON blockchain. 8 Divine Kingdoms compete for power across 13-day rounds (52 TC days, 1 TC day = 6 real hours). Phases: Genesis (1-4), Expansion (5-14), Conquest (15-34), Endgame (35-45), Ragnarok (46-52). You interact with the game through the `godsx-mortals` MCP server.

## Strategy is the user's decision

This skill does not tell the user how to play. Before committing to a playstyle, **ask what they want to optimize for** — unless they've already said, or you're continuing a previous plan.

A good first step is to check `references/rewards.md` (structure) and `get_patch_notes` (live round categories) together. The round pays out for many different niches — money, prestige medals, or permanent badges — and different categories reward wildly different play. Pointing the user at the actual prize list often changes what they want to optimize for.

Example archetypes users might pick:

| Archetype | Focus | Natural class fit |
|-----------|-------|-------------------|
| **Ranker** | Leaderboard respect — stats, PvP, medals | Slayer; any class works |
| **Tycoon** | Gold accumulation — buildings, companions, investments, auctions | Merchant |
| **Raider** | Heist volume — tickets, mercenaries, drug byproducts | Raider |
| **Warlord** | PvP assaults, guild/kingdom wars, district capture | Slayer / high-might |
| **Trader** | Dealer arbitrage, private ships, shadow market, auction flips | Alchemist / Merchant |
| **Diplomat** | Guild leadership, kingdom politics, chat, policy, wars | Any |
| **Gambler** | Casino EV hunting — lottery, blackjack, dice, slots | Oracle (investment bonus) |
| **Completionist** | Badge hunting — work backward from `get_achievements` thresholds | Any |
| **Balanced** | Do a bit of everything, follow `get_recommended_actions` | Any |

Users can blend these or invent their own. If they say "just play" with no preference, default to **Balanced** and tell them the default so they can override it.

Also ask about:
- **Risk tolerance** — aggressive PvP, high-addiction heisting, casino exposure
- **Autonomy** — do they want confirmation before large gold spends? Irreversible choices? Every action?
- **Session length** — a single burst, or until a natural break (training start, tickets out)?

**Always confirm irreversible actions** before calling them: `choose_class`, `choose_kingdom`, `remove_protection`, `demolish_building`, `forfeit_war`. These cannot be undone this round.

## Universal safety rules (mechanics, not strategy)

These apply to every playstyle — they're game physics, not preference:

1. **Bank gold before any risky action.** Gold on hand is lost on failed heists, stolen in lost assaults, exposed to sabotage. Banked gold is safe and earns ~2% interest per round.
2. **Resolve blocking states first.** Jailed → `bribe_jail`. Hospitalized → `heal_hospital`. Training active → combat tools fail until complete. Nothing else works while these are set.
3. **Addiction above 10 degrades everything.** Heist success, combat power, training, stamina regen all drop. Wait for decay (-2.0/TC day) or buy Divine Shield before pushing further.
4. **Trust the live tool schema over these docs.** If a param or enum has changed, the schema is authoritative.
5. **BigInt values are strings.** Gold: `"4500000"`, not `4500000`.

## Decision framework

Don't hard-code a loop. Think in four layers every cycle:

```
LAYER 1 — Unblock (always): jailed → bribe; hospitalized → heal; training ready → collect
LAYER 2 — De-risk (always): bank gold on hand
LAYER 3 — Produce (archetype-driven): the actions below, weighted by archetype
LAYER 4 — Maintain (periodic): daily collections, missions, level-ups, achievements
```

**Balanced default sequence** (adapt for other archetypes):

```
BOOT:
  get_setup_status → complete setup with user confirmation
  get_recommended_actions → read playerSummary
  get_player_status for any detail not in summary

CYCLE:
  1. Resolve any blocking state (jail / hospital / training-complete)
  2. Bank gold on hand
  3. Pick a productive action by state:
     - tickets > 10, addiction < 10 → use_all_stamina (heist)
     - training idle → start_training (stat depends on archetype)
     - vault/companions ready → collect_all_building_vaults / collect_companions
     - tcDay >= 15, HP >= 30%, PvP archetype → assault flow
     - in a guild with role → raids / trials / wars
     - training active → shift to non-blocking work (see below)
  4. If a mission progressed → claim_mission
  5. After XP-earning actions → check_level_up
  6. Periodic (every few cycles): get_achievements, get_patch_notes, get_active_events
```

**Archetype overrides** — examples, not mandates:
- **Ranker**: Prioritize PvP windows and stat-training over heist volume once safe. Pick the archetype's primary stat (Slayer: MIGHT, others typically WISDOM, but user preference wins).
- **Tycoon**: Weight buildings, companions, investments, auctions. Heists mainly serve building activity gates.
- **Raider**: `hire_mercenaries` every 2 TC days. Sell drug drops via dealer or private ships.
- **Warlord**: Target rival guild/kingdom contributors during wars. District challenges for permanent bonuses.
- **Trader**: Each cycle starts with `get_dealer_prices`, `get_docks_status`, `list_ship_auctions`, `browse_market`. Arbitrage until spreads close.
- **Gambler**: Set a session bankroll cap before the first bet. Stop at the cap, win or lose.
- **Completionist**: Read `get_achievements` each cycle; bias toward the nearest threshold.

**Efficiency rules:**
- Don't re-poll `get_recommended_actions` between micro-steps. Once per cycle — or after an error / state-changing call (tavern_refill, execute_assault, start_training) — is enough.
- Prefer `use_all_stamina` over single `attempt_heist` (batches ~50 heists, auto-stops on jail/hospital).
- Training blocks heists/assaults/raids/hunting — use the time for non-blocking work.

## System map — full game surface

Every major subsystem and its tools. Use this to discover features the user may want to exploit. **Deep-dive details in `references/systems-overview.md`.** For exact params see `references/tool-reference.md`.

| System | Key tools | Purpose |
|---|---|---|
| Heists | `list_heists`, `use_all_stamina`, `attempt_heist`, `hire_mercenaries` | Core gold + some drug drops |
| Training | `start_training`, `collect_training`, `boost_training`, `get_training_status` | Stats → combat power + respect |
| Banking | `bank_deposit`, `bank_withdraw`, `get_bank_balance` | Safe storage + interest |
| Buildings | `list_buildings`, `purchase_building`, `upgrade_building`, `prestige_building`, `collect_building_vault`, `collect_all_building_vaults`, `demolish_building`, `cancel_building_upgrade` | Passive gold, activity-gated |
| Companions | `list_companions`, `buy_companions`, `collect_companions` | Daily passive gold |
| Investments | `create_investment`, `get_investments` | Mid-round gold multipliers (standard / pack2 / pack3) |
| Dealer & ships | `get_dealer_prices`, `dealer_buy`, `dealer_sell`, `get_docks_status`, `sell_to_ship`, `list_ship_auctions`, `get_ship_details`, `place_ship_bid`, `sell_to_private_ship`, `get_my_ship_bids` | Drug price arbitrage |
| Drug lab | `get_lab_status`, `start_lab_production`, `collect_lab`, `upgrade_lab`, `use_potion` | Craft consumables; Alchemy academy |
| Shop | `browse_shop`, `buy_shop_item`, `activate_shop_item`, `deactivate_shop_item`, `get_shop_inventory` | Divine Shield, scrolls, premium items |
| Inventory | `get_player_inventory`, `use_inventory_item` | Components + consumables (NOT drugs) |
| Equipment | `browse_equipment_shop`, `buy_equipment`, `equip_item`, `unequip_item`, `get_equipment`, `get_equipped_items`, `sell_equipment`, `sell_equipment_bulk`, `toggle_equipment_lock`, `enhance_equipment`, `auto_enhance_equipment`, `preview_enhancement`, `get_enhancement_breakdown` | 5-slot gear, +0…+15 |
| Auction house | `list_auction_listings`, `create_auction_listing`, `place_auction_bid`, `auction_buy_now`, `cancel_auction_listing`, `get_my_auction_listings`, `get_my_auction_bids` | Player-to-player gear trade |
| Shadow market | `browse_market`, `create_market_listing`, `place_market_bid`, `market_buyout`, `get_my_market_listings`, `get_my_market_bids` | Item + component trade |
| PvP assaults | `get_assault_suggestions`, `search_assault_target`, `preview_assault_target`, `execute_assault`, `get_assault_detail` | Steal stats/gold, kill streaks |
| Hunting | `list_hunting_grounds`, `list_hunting_beasts`, `preview_hunting_beast`, `fight_hunting_beast` | Alt PvE — gold + components |
| Sabotage | `list_sabotage_agents`, `deploy_sabotage`, `get_sabotage_guard_status`, `buy_sabotage_guard`, `sell_sabotage_guard` | Attack / defend economies |
| Districts | `list_districts`, `get_my_districts`, `conquer_district`, `challenge_district_street` | Permanent round bonuses |
| Guild | `get_guild_info`, `search_guilds`, `join_guild`, `apply_to_guild`, `leave_guild`, `donate_to_guild`, `upgrade_guild`, `get_guild_upgrades`, `set_guild_policy`, `get_guild_applications`, `approve_guild_application`, `reject_guild_application`, `kick_guild_member`, `promote_guild_member`, `demote_guild_member` | Social base unit |
| Guild wars | `declare_war`, `get_active_wars`, `get_war_history`, `get_war_rivals`, `execute_war`, `forfeit_war`, `get_war_details` | Guild-vs-guild PvP |
| Raids | `list_raid_types`, `get_active_raids`, `preview_raid`, `create_raid`, `join_raid`, `leave_raid`, `execute_raid`, `cancel_raid`, `get_raid_details`, `get_raid_history` | Coordinated guild combat |
| Trials | `get_available_trials`, `preview_trial`, `start_trial` | Guild boss fights (blessed only) |
| Kingdom | `get_kingdom_info`, `get_kingdom_profile`, `get_kingdom_rankings`, `get_kingdom_members`, `get_kingdom_upgrades`, `purchase_kingdom_upgrade`, `set_kingdom_tax` | Nation layer; Sovereign/Archon tools |
| Kingdom wars | `declare_kingdom_war`, `get_active_kingdom_war`, `get_war_contribution`, `get_kingdom_war_rivals`, `get_kingdom_war_details`, `get_kingdom_war_history` | Week-long kingdom PvP |
| Casino | `lottery_info`, `lottery_buy_ticket`, `buy_random_lottery`, `get_blackjack_status`, `blackjack_deal`, `blackjack_action`, `dice_roll`, `slots_spin` | Gambling for gold / jackpots |
| Missions | `get_missions`, `claim_mission` | Daily free reward |
| Academy | `get_academy_courses`, `enroll_academy`, `complete_academy` | Cross-class permanent bonuses |
| Chat & social | `get_chat_messages`, `send_chat_message`, `follow_player`, `unfollow_player`, `view_public_profile`, `get_kingdom_profile` | Coordination, scouting |
| Leaderboards | `get_leaderboard` (players / kingdoms / guilds / kills) | Positioning intel |
| Tavern | `get_tavern_status`, `tavern_refill`, `get_healing_status`, `tavern_heal` | Stamina + HP services |
| Jail/hospital | `get_jail_status`, `bribe_jail`, `get_hospital_status`, `heal_hospital` | Blocking state resolution |
| Divine Mule | `get_mule_status`, `call_divine_mule` | Leader-triggered drug + gold drops to blessed members |
| Protection | `remove_protection` | Irreversibly enable PvP this round |
| Events | `get_active_events`, `get_event_effects` | Round-wide modifiers |
| Setup orchestration | `get_setup_status`, `get_game_overview`, `get_recommended_actions` | Onboarding + per-cycle priority |
| Status | `get_player_status`, `check_level_up`, `get_activity_log` | State snapshots |
| Achievements | `get_achievements`, `get_medals` | Badges + medals |
| Feedback | `list_feedback`, `submit_feedback`, `vote_feedback`, `submit_agent_review`, `get_patch_notes` | Community + patch awareness |

## Resource rule quick reference

Full formulas in `references/resource-management.md`.

- **Addiction**: +`0.5 + 0.04×addiction` per refill (cap +5). Decay −2.0/TC day. Penalties start stacking at 6; bad above 10; critical at 21+.
- **Stamina tickets**: +72/TC day (Raider +120). Refill = 1 ticket + `5,000 + 2,000×addiction + 150×addiction²` gold. Keep ~10-ticket buffer for emergencies.
- **HP**: +10% max/TC day, floor 10. Tavern heal: `500 + 200×level²` gold, 2 TC hour cooldown. `heal_hospital`: 300 gold/sec remaining.
- **Training**: 30 min always; 60 min at lvl 5 + 20 sessions; 480 min at lvl 10. Modifier stack: Spirit (±10%), Guild Training Protocol, Divine Favor catch-up, Beast Venom (−10%/stack, max −30%).
- **Bank**: +2%/round interest on balance. Gold-on-hand is the only pool exposed to combat loss.
- **Enhancement**: +0…+15, risk zone kicks in at +11. Use `get_enhancement_breakdown` before high attempts; Protection Scrolls from shop reduce tier-drop risk.

## Required scopes

`read`, `combat`, `economy`, `casino`, `social`, `management`, `feedback`. Create the token with all scopes from **Settings > Agent Access** in-game.

## Response envelope

```
{ success: true, data: {...} }
{ success: false, error: { code, message, retryable, suggestion } }
```
If `retryable`, follow `suggestion` and retry. Otherwise surface the failure to the user.

## Known gotchas

### Drug inventory is NOT in `get_player_inventory`
`get_player_inventory` returns components and consumables only. For drug holdings (moonleaf, shadow_dust, crimson_vial, void_essence, titan_ichor, divine_nectar) call `get_dealer_prices` and read the per-drug `playerInventory` field.

### Potion casing differs by tool
- `start_lab_production` expects **lowercase**: `healing_potion`, `stamina_elixir`, `fortitude_brew`, `might_tonic`, `shadow_draught`, `divine_ambrosia`
- `use_potion` expects **UPPERCASE**: `HEALING_POTION`, `STAMINA_ELIXIR`, `FORTITUDE_BREW`, `MIGHT_TONIC`, `SHADOW_DRAUGHT`, `DIVINE_AMBROSIA`

### `collect_companions` empty result isn't a failure
Companions collect once per TC day (6h). Already-collected returns empty silently.

### Building vault activity gates
`collect_all_building_vaults` skips buildings that haven't met their activity threshold (e.g. Forge needs 5 assaults / 3 TC days). See resource-management.md for the full table.

### Hunting tool names
The canonical flow is `list_hunting_grounds` → `list_hunting_beasts {tavernCategory}` → `preview_hunting_beast {beastId}` → `fight_hunting_beast {beastId}`. Pass beast IDs, not tavern IDs.

## Error recovery

| Error | Recovery |
|---|---|
| JAILED | `bribe_jail {paymentType: 'gold'}` — 500 gold/sec remaining |
| HOSPITALIZED | `heal_hospital {}` — 300 gold/sec remaining |
| TRAINING | Non-blocking work, or `collect_training` if complete |
| INSUFFICIENT_GOLD | `bank_withdraw` or earn first |
| INSUFFICIENT_STAMINA | `tavern_refill` if addiction safe, else wait |
| LOW_HP | `tavern_heal` (2 TC hour cooldown) |
| COOLDOWN | Other work, retry later |
| INVALID_INPUT | Check `paramHints` or live schema |
| SCOPE_DENIED | Token lacks scope — add it in-game |

## Reference docs

- `references/systems-overview.md` — deep dive per system with strategic levers
- `references/phase-strategy.md` — phase mechanics: unlocks, gates, maturity/decay windows (no prescribed loops)
- `references/rewards.md` — what the round pays out for: TON categories, medals, badges (for co-designing strategy with the user)
- `references/resource-management.md` — exact formulas and thresholds
- `references/tool-reference.md` — full 189-tool catalog with params

## Community & feedback

### Patch notes
- `get_patch_notes` — index
- `get_patch_notes {patchId: "..."}` — full detail
- Check when user asks what's new or after confusing results

### Feedback & bug reports
- `list_feedback` — browse
- `submit_feedback` — ask user first; include system affected and repro
- `vote_feedback` — toggle upvote
- `submit_agent_review` — structured feasibility analysis for UNDER_REVIEW items

Don't submit without user agreement. Vote on items the user cares about or bugs you've personally hit.

## Examples

### "Play the game for me" (no archetype stated)
1. Ask what they want to optimize for (Ranker, Tycoon, Raider, Warlord, Trader, Diplomat, Gambler, Completionist, Balanced). Default to Balanced if they say "just play".
2. `get_setup_status` → complete setup if missing. Confirm `choose_class` / `choose_kingdom` with the user before calling.
3. `get_recommended_actions` → execute highest-priority action per the decision framework, weighted by archetype.
4. Loop. Stop at a natural break (training started, tickets exhausted with high addiction, major irreversible decision) or on user signal.

### "Do heists until tickets run out"
1. `bank_deposit` all gold
2. Loop: `use_all_stamina` → `bank_deposit` → check addiction
3. Stamina = 0 and addiction < 10 → `tavern_refill`, continue
4. Addiction ≥ 10 → stop; report tickets remaining; suggest training or non-blocking work

### "What class should I pick?"
Don't prescribe. Present the tradeoffs:
- Slayer: 3 assaults/day, 2× respect/stat — ranking/PvP
- Merchant: +slots, 40% discount — economy/Tycoon
- Raider: +20% heist, 120 tickets/day — heist volume
- Enchantress: +15% companions, cheap guild upgrades — social/sabotage
- Alchemist: +20% drug profit, faster lab — trading
- Oracle: +1% investments, cheap academy — patient economy

Ask what they want to optimize for, then summarize the 1-2 best fits.

### "Manage my kingdom" (Sovereign/Archon)
1. `get_kingdom_info`, `get_kingdom_members`, `get_kingdom_upgrades`
2. `set_kingdom_tax` — balance revenue vs. member retention
3. `purchase_kingdom_upgrade` by priority and treasury
4. Wars: `get_kingdom_war_rivals` → confirm with members via chat before `declare_kingdom_war`
5. `send_chat_message {channel: 'kingdom'}` for announcements

### "Manage my guild" (Leader)
1. `get_guild_info`, `get_guild_upgrades`, `get_guild_applications`
2. Applications: approve/reject; set `set_guild_policy`
3. Upgrades: HQ, RAID_STRATEGY, ASSAULT_STRATEGY, TRAINING_PROTOCOL, BRIBE_REDUCTION, GUILD_BANK, DIVINE_MULES
4. Raids: `create_raid` → members `join_raid` → `execute_raid`
5. Trials (blessed only): `get_available_trials` → `preview_trial` → `start_trial`
6. Wars: `get_war_rivals` → `declare_war` when eligible (HQ 6+, assault 6+, 6+ blessed)
