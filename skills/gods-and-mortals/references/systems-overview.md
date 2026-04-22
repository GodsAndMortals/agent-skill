# Systems Overview

Breadth map of every major game system: purpose, tools, strategic levers, and gotchas. For exact formulas see `resource-management.md`. For phase timing see `phase-strategy.md`. For full tool params see `tool-reference.md`.

## Core gold loop

### Heists
The primary gold generator. Stamina-gated, level-gated by heist type.

- `list_heists` — all heist types with difficulty, success rate, rewards, level requirements
- `use_all_stamina {heistType}` — batch up to 50 heists; auto-stops on jail/hospital. **Default choice.**
- `attempt_heist {heistType}` — single heist; use for probing a new tier
- `hire_mercenaries {heistType}` — bulk alternative, needs 200+ tickets and TC day 3+

Strategic levers:
- Pick the highest tier you qualify for — reward scales faster than difficulty
- Raider class: +20% heist reward, 120 tickets/day — natural fit
- Academy "Art of the Heist" adds +5/10/15% reward
- Addiction penalty is -0.5% success/pt (max -25%)

### Training
Converts time into stats, which feed combat power and respect.

- `get_training_status`, `start_training {stat, duration}`, `collect_training`, `boost_training` (+20% speed, costs 5 tickets)
- Durations: 30 min (always) / 60 min (lvl 5 + 20 sessions) / 480 min (lvl 10)
- Blocks heists, assaults, raids, hunting, trials

Strategic levers:
- Stat choice should match archetype: MIGHT for PvP power, WISDOM for heist/economy multipliers, FORTITUDE for HP/survivability, GUILE for crit/evasion
- Spirit modifier ranges ±10% based on addiction — train at addiction ≤ 5 for +10%
- Guild Training Protocol adds per-level bonus
- Use training time for non-blocking economy work (market, lab, investments)

### Banking
Safe gold storage.

- `get_bank_balance`, `bank_deposit {amount}`, `bank_withdraw {amount}`
- ~2% interest per round on balance
- Always bank before heists/assaults/sabotage windows

## Passive economy

### Buildings & vaults
8 slots (6 for Merchant +40% discount). Production accumulates in per-building vaults, capped at 3 TC days' production. Activity gates required to collect.

- `list_buildings`, `purchase_building {buildingType, slot}`, `upgrade_building {buildingId}`, `cancel_building_upgrade {buildingId}` (50% refund)
- `prestige_building {buildingId}` — max-level reset for permanent production bonus
- `demolish_building {buildingId}` — free slot (no refund)
- `collect_building_vault {buildingId}`, `collect_all_building_vaults`

Activity thresholds (3 TC day window):
- Thieves' Den: 15 successful heists
- Harbor: 1 dock sale
- Pleasure Quarter: 1 companion collection
- Temple: 2 completed training sessions
- Forge: 5 assaults completed
- Golden Exchange: 1 investment made
- Merchant Row: 3 shop transactions

Strategic levers:
- Tycoons prioritize; everyone benefits from 1-2 buildings aligned with their normal activity
- Don't build past your play pattern — Forges only pay Warlords
- Prestige when a building maxes and you're continuing the same build

### Companions
Passive per-TC-day gold generators with escape risk.

- `list_companions`, `buy_companions {type, count}`, `collect_companions`
- Collect once per TC day (6h). Returns empty if already collected — normal.
- 2% escape risk per collection

Strategic levers:
- Enchantress +15% companion earnings
- Pair with Pleasure Quarter building (activity = collection)
- Academy "Ways of Enchantment" adds +5/10/15%

### Investments
Mid-round gold multiplier.

- `create_investment {type, amount}` — types: `standard`, `pack2`, `pack3`
- `get_investments`

Strategic levers:
- Mature over many TC days — only start if enough round remains
- Oracle +1% returns
- Academy-adjacent for economy builds

## Drug economy

### Dealer & docks & ships
Drug prices fluctuate; profit from the spread between buy and sell venues.

- Dealer (primary): `get_dealer_prices` (shows prices + your holdings), `dealer_buy {drugType, quantity}`, `dealer_sell {drugType, quantity}`
- Docks: `get_docks_status`, `sell_to_ship {shipId, drugType, quantity}` — premium prices for matched drugs
- Private ships: `list_ship_auctions`, `get_ship_details {shipId}`, `place_ship_bid {shipId, amount}`, `get_my_ship_bids`, `sell_to_private_ship {shipId, drugType, quantity}` — win auction for exclusive sell window

Drug tiers (rising margin): moonleaf → shadow_dust → crimson_vial → void_essence → titan_ichor → divine_nectar

Strategic levers:
- Trader archetype: check prices every cycle; arbitrage until spreads close
- Raider: heists drop drugs — sell rather than hoard
- Alchemist: +20% drug profit class bonus
- **Drug holdings are NOT in `get_player_inventory`** — use `get_dealer_prices` to see them

### Drug lab
Produce consumable potions with combat/economy effects. Tied to Alchemy academy.

- `get_lab_status`
- `upgrade_lab {labId}` — max level 4, each tier speeds production
- `start_lab_production {labId, potionType, quantity}` — **lowercase** potion names
- `collect_lab`
- `use_potion {potionType}` — **UPPERCASE** potion names

Potions: healing_potion / HEALING_POTION, stamina_elixir / STAMINA_ELIXIR, fortitude_brew, might_tonic, shadow_draught, divine_ambrosia.

Strategic levers:
- Stock stamina_elixir + might_tonic before big PvP/raid windows
- Healing potions bypass tavern cooldown
- Always have production running — lab is a free activity during training

## Equipment

5 slots. Power scales with enhancement (+0 to +15). High-level enhancement can drop tier or destroy the item; Protection Scrolls from shop mitigate.

- Shop: `browse_equipment_shop`, `buy_equipment {templateIndex}`
- Manage: `get_equipment`, `get_equipped_items`, `equip_item {equipmentId}`, `unequip_item {id}`, `toggle_equipment_lock {itemId}` (prevent accidental sale)
- Dispose: `sell_equipment {id}`, `sell_equipment_bulk {itemIds}`
- Enhance: `preview_enhancement {equipmentId}`, `get_enhancement_breakdown {equipmentId}`, `enhance_equipment {equipmentId, useProtectionScroll?}`, `auto_enhance_equipment {itemId, targetLevel}`
- Trade: auction house or shadow market (see below)

Strategic levers:
- Lock best items before any bulk-sell
- Risk zone starts at +11 — check `get_enhancement_breakdown` before attempting
- Enhance during training (no combat block)
- Drops from heists / raids / trials augment shop purchases

### Auction house
Player-to-player equipment market. 24h listings, 10% sale tax, 1% listing fee.

- `list_auction_listings {slot?, rarity?}`, `get_my_auction_listings`, `get_my_auction_bids`
- `create_auction_listing {itemId, startPrice, buyoutPrice?}` (max 5 active)
- `place_auction_bid {listingId, bidAmount}` (5% min increment)
- `auction_buy_now {listingId}`
- `cancel_auction_listing {listingId}` — only if no bids; fee not refunded

Strategic levers:
- Buyout ~20-30% above start for impulse buyers
- `get_my_auction_bids` refunds on outbid, but the item is lost without re-bidding
- Endgame 10% tax is a real consideration

### Shadow market
Item + component trade (non-equipment).

- `browse_market`, `get_my_market_listings`, `get_my_market_bids`
- `create_market_listing {itemId, startPrice, buyoutPrice?}`
- `place_market_bid {listingId, amount}`, `market_buyout {listingId}`

Use for sourcing components you need (lab, enhancement) or liquidating surplus drops.

## Shop (premium store)

- `browse_shop`, `get_shop_inventory`
- `buy_shop_item {itemType, quantity?}`
- `activate_shop_item {itemId}` / `deactivate_shop_item {itemId}`

Key items:
- **Divine Shield** — 20M gold, 4 TC hours of zero-addiction tavern refills
- **Protection Scrolls** — reduce enhancement failure penalties
- Seasonal premium items rotate

## Combat

### PvP assaults
Direct player attack. Wins transfer stats and gold on hand.

- `get_assault_suggestions` — matched-respect targets
- `search_assault_target {query}` — find by username
- `view_public_profile {playerId}` — scout
- `preview_assault_target {targetId}` — estimate outcome
- `execute_assault {targetPlayerId}`
- `get_assault_detail {assaultId}` — full log

Limits: 2-3/day (Slayer +1). Requires `remove_protection` (irreversible) or Conquest phase by default.

Strategic levers:
- Target weak-stat + high-gold-on-hand
- HP ≥ 30% before committing (hospitalization costs more time than one assault earns)
- Stat theft is Ranker/Warlord fastest late-round respect source
- Forge building needs 5 assaults / 3 TC days — routine assaulting feeds it

### Hunting
Alt PvE combat. No ticket cost; rewards gold + components.

- `list_hunting_grounds` — categories (beginner / fixed / dynamic)
- `list_hunting_beasts {tavernCategory}` — returns beast IDs like `cave_imp`
- `preview_hunting_beast {beastId}` — odds and rewards
- `fight_hunting_beast {beastId}` — pass beast ID, NOT tavern ID

Strategic levers:
- Component farming when lab/enhancement starved
- Lower-risk combat XP than PvP
- Training blocks hunting

### Sabotage
Asymmetric attack/defense for rival economies.

- Offense: `list_sabotage_agents` → `deploy_sabotage {victimId, agentType}`
  - Agents: SHADOW_SPY, DARK_ASSASSIN, VOODOO_PRIEST, DIVINE_WRAITH, PHANTOM_THIEF, SIREN_AGENT, CHAOS_ALCHEMIST (different effects, costs, cooldowns)
- Defense: `get_sabotage_guard_status`, `buy_sabotage_guard {guardType}`, `sell_sabotage_guard`

Strategic levers:
- Cheap way to tilt rivals without committing to PvP combat
- Enchantress archetype/academy synergy
- Must `sell_sabotage_guard` before buying a different guard type

### Districts
Conquerable map zones with permanent round bonuses.

- `list_districts`, `get_my_districts`
- `conquer_district {districtId}` — claim (stat-gated)
- `challenge_district_street {districtId, streetId}` — PvP a specific street owner

Strategic levers:
- Early claims in uncontested districts for cheap bonuses
- Warlord focus mid-/late-round

### Jail & hospital
Blocking states from failed actions or lost combat.

- `get_jail_status`, `bribe_jail {paymentType: 'gold'}` — 500 gold/sec remaining
- `get_hospital_status`, `heal_hospital` — 300 gold/sec remaining (NFT tier: up to 90% off)

Never leave either untreated — idle time is lost progression.

### Tavern services
- `get_tavern_status`, `tavern_refill` — restore stamina; +addiction
- `get_healing_status`, `tavern_heal` — HP to full; 2 TC hour cooldown; `500 + 200×level²` gold

## Guild layer

### Guild basics
All players can lead a guild.

- Discover: `get_guild_info {guildId}`, `search_guilds {query?, kingdom?}`
- Join: `join_guild {guildId}` (open policy) or `apply_to_guild {guildId, message?}` (private)
- Leave: `leave_guild`
- Contribute: `donate_to_guild {guildId, amount}` — respect + treasury

### Guild leadership
- Members: `kick_guild_member {playerId}`, `promote_guild_member {playerId}`, `demote_guild_member {playerId}`
- Applications: `get_guild_applications {guildId}`, `approve_guild_application {applicationId}`, `reject_guild_application {applicationId}`
- Policy: `set_guild_policy {policy}` — OPEN / APPLY / INVITE_ONLY
- Upgrades: `get_guild_upgrades {guildId}`, `upgrade_guild {guildId, upgradeType}` — HQ, RAID_STRATEGY, ASSAULT_STRATEGY, TRAINING_PROTOCOL, BRIBE_REDUCTION, GUILD_BANK, DIVINE_MULES

### Guild wars
Guild-vs-guild PvP.

- Discover: `get_war_rivals`
- Declare: `declare_war {targetGuildId}` — leader only, requires HQ 6+, assault 6+, 6+ blessed members
- Execute: `execute_war {warId}` (leader)
- Abandon: `forfeit_war {warId}` (leader, irreversible loss)
- Track: `get_active_wars`, `get_war_history`, `get_war_details {warId}`

### Raids
Coordinated guild combat.

- Discover: `list_raid_types`, `get_active_raids`, `get_raid_history`
- Preview: `preview_raid {raidId}` — success rate, buffs
- Lifecycle: `create_raid {raidType}` (leader/officer, auto-joins) → members `join_raid {raidId}` → `execute_raid {raidId}` (leader) OR `cancel_raid` OR `leave_raid`
- Detail: `get_raid_details {raidId}`

### Trials
Boss fights with blessed members only.

- `get_available_trials`
- `preview_trial {bossId}`
- `start_trial {bossId, participantIds}` — leader-initiated; blessed-only participants

### Divine Mule
Free gold/drug delivery to blessed guild members.

- `get_mule_status`, `call_divine_mule`
- Leader triggers; upgrades via `upgrade_guild {upgradeType: 'DIVINE_MULES'}`

## Kingdom layer

### Kingdom basics
- `get_kingdom_info {kingdomId}`, `get_kingdom_profile {kingdomId}`, `get_kingdom_rankings`, `get_kingdom_members {kingdomId}`
- `get_kingdom_upgrades {kingdomId}`

### Kingdom leadership (Sovereign/Archon)
- `set_kingdom_tax {kingdomId, rate}` — Sovereign only
- `purchase_kingdom_upgrade {kingdomId, upgradeType}` — Sovereign/Archon

### Kingdom wars
Week-long kingdom-vs-kingdom war. Individual players contribute via normal gameplay.

- Discover: `get_kingdom_war_rivals` — eligible targets + power estimate
- Declare: `declare_kingdom_war {targetKingdomId}` — Sovereign only, 2 TC day prep
- Track: `get_active_kingdom_war`, `get_war_contribution` (personal + kingdom leaderboard), `get_kingdom_war_details {warId}`, `get_kingdom_war_history {kingdomId}`

Contribution flows from 7 services: heist, raid, assault, training, hunting, trial, guild war. Winner: +kingdom points, CONQUEROR buff (+5%), treasury plunder (capped). Loser: WAR_WEARY (-3%).

Badges: War Veteran (100pts) / War Champion (500pts) / War Legend (2000pts).

## Casino

Four games. House edge on each; treat as entertainment or positioning tool, not primary income.

### Lottery
- `lottery_info` — jackpot, ticket price, odds
- `lottery_buy_ticket {numbers}` — 4 unique numbers from 1-20
- `buy_random_lottery {count}` — auto-picked

Strategic use: low-cost exposure to tail-risk jackpot; most rational near round end for Wealth/Ranker positioning.

### Blackjack
- `get_blackjack_status` — reconnect to in-progress hand
- `blackjack_deal {betAmount}` — start
- `blackjack_action {action}` — hit / stand / double

### Dice
- `dice_roll {betAmount, betType, target}` — over / under / exact on d6

### Slots
- `slots_spin {betAmount}` — pure RNG

### Casino strategy
- Set a session bankroll cap before the first bet
- Stop at the cap, win or lose
- Never borrow from bank "just for one more spin"

## Missions & achievements

### Missions
Daily free-reward pipeline.

- `get_missions` — active mission + progress
- `claim_mission {missionId}` — complete

Always claim. Some missions tick passively during normal play.

### Achievements
Permanent badges across rounds. See resource-management.md §Achievements for the full catalog.

- `get_achievements` — all badges with progress + unlock requirements
- `get_medals` — round-end medals (blessed-only)

### Academy
Cross-class permanent round bonuses.

- `get_academy_courses`, `enroll_academy {courseId}`, `complete_academy`
- Costs: 5M / 25M / 100M per level. 130M total to master one course.
- Progress passive through gameplay — event map in resource-management.md
- Cannot take your own class's course

## Social & communication

### Chat
- `get_chat_messages {channel, limit?}` — channel: `guild`, `kingdom`, etc.
- `send_chat_message {channel, content}` — 500 char max

Use for coordinating raids/wars, announcing, diplomatic channels.

### Follow / profile
- `follow_player {playerId}`, `unfollow_player {playerId}` — track specific rivals/allies
- `view_public_profile {playerId}` — scout stats before assault
- `get_kingdom_profile {kingdomId}` — kingdom overview

### Leaderboards
`get_leaderboard {category, page?}` — categories: `players`, `kingdoms`, `guilds`, `kills`.

Rankers check each cycle. Completionists find reachable thresholds. Warlords identify kingdom/guild targets.

## Protection & events

### Protection
`remove_protection` — irreversibly ends divine protection, enabling PvP against you for the rest of the round. Default protection usually covers Genesis + early Expansion.

Always confirm with the user before calling. Irreversible.

### Events
- `get_active_events` — current active events
- `get_event_effects` — aggregated modifier map (use when optimizing multi-system actions)

Events can boost XP, shift drug markets, amplify casino payouts, etc. Read once per cycle or two.

## Status & orchestration

### Status
- `get_player_status` — full player snapshot
- `check_level_up` — threshold check
- `get_activity_log` — recent assault + sabotage history

### Orchestration (built for agents)
- `get_setup_status` — setup completeness + pending actions
- `get_game_overview` — game manual (time system, phases, classes, kingdoms)
- `get_recommended_actions` — context-aware prioritized actions with pre-filled params

Use `get_recommended_actions` at the start of each cycle, then execute from the returned list. Don't re-poll between steps.

## Feedback & patches

- `get_patch_notes {patchId?}` — fetch patch content from the live site
- `list_feedback {category?, status?, sort?, page?, limit?}` — browse community submissions
- `submit_feedback {title, body, category}` — submit (ask user first)
- `vote_feedback {feedbackItemId}` — toggle upvote
- `submit_agent_review {feedbackItemId, ...}` — structured feasibility analysis for UNDER_REVIEW items
