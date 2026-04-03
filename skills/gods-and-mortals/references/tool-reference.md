# Tool Reference

Complete catalog of MCP tools organized by scope. Required params shown with types.

## Status (read scope)

| Tool | Params | Notes |
|------|--------|-------|
| `get_player_status` | none | Full player snapshot: level, stats, gold, bank, HP, timers, NFT tier |
| `get_round_info` | none | Round phase, TC day, unlocked features |
| `get_jail_status` | none | Jail status, remaining time, bribe cost |
| `get_hospital_status` | none | Hospital status, remaining time, heal cost (with NFT discount) |
| `get_active_events` | none | Active divine events with effects |
| `get_event_effects` | none | Aggregated event modifier map (faster than get_active_events) |
| `get_activity_log` | none | Recent assault and sabotage history |
| `view_public_profile` | `playerId: string` | Another player's public profile |

## Combat (combat scope)

| Tool | Params | Notes |
|------|--------|-------|
| `list_heists` | none (read) | All heist targets with difficulty, success rate, rewards, level requirements |
| `attempt_heist` | `heistType: string` | Single heist attempt. Prefer `use_all_stamina` for bulk. |
| `use_all_stamina` | `heistType: string` | Bulk heist (max 50). Stops on jail/hospital. Returns aggregate results. |
| `hire_mercenaries` | `heistType: string` | Bulk heist alternative. Requires 200+ tickets, TC day 3+. |
| `search_assault_target` | `query: string` | Find PvP opponent by username. Returns target info + estimated outcome. |
| `get_assault_suggestions` | none (read) | Suggested assault targets at similar respect level |
| `preview_assault_target` | `targetId: string` (read) | Preview target stats and estimated outcome before committing |
| `get_assault_detail` | `assaultId: string` (read) | Single assault log with full combat details |
| `execute_assault` | `targetPlayerId: string` | PvP attack. 2-3/day max. Transfers stats from loser. |
| `list_hunting_grounds` | none (read) | Hunting locations with rewards |
| `fight_hunting_ground` | `tavernId: string` | Combat at hunting location for gold + components (auto enter+fight) |
| `list_beasts` | `category: string` (read) | List beasts in a specific hunting tavern category |
| `preview_beast_fight` | `beastId: string` (read) | Preview fight outcome for a specific beast |
| `deploy_sabotage` | `victimId: string, agentType: enum` | Sabotage a player. agentType: SHADOW_SPY, DARK_ASSASSIN, VOODOO_PRIEST, DIVINE_WRAITH, PHANTOM_THIEF, SIREN_AGENT, CHAOS_ALCHEMIST |
| `get_sabotage_guard_status` | none (read) | Guard status on your districts |
| `buy_sabotage_guard` | `guardType: string` | Buy protection on district |
| `sell_sabotage_guard` | none | Dismiss current defense guard (required before buying a different one) |
| `list_sabotage_agents` | none (read) | Available sabotage agent types with costs, effects, and cooldowns |
| `list_districts` | none (read) | Conquerable districts with rewards |
| `conquer_district` | `districtId: string (uuid)` | Conquer a district for permanent bonuses |
| `challenge_district_street` | `districtId: string, streetId: string` | Challenge a street owner in a district for PvP district combat |
| `get_my_districts` | none (read) | Your currently owned districts and streets |
| `remove_protection` | none | Remove divine protection to enable PvP (irreversible for the round) |
| `bribe_jail` | `paymentType: 'gold'` | Escape jail. Cost: 500 gold/sec remaining. |
| `heal_hospital` | none | Leave hospital. Cost: 300 gold/sec remaining. |

## Economy (economy scope)

| Tool | Params | Notes |
|------|--------|-------|
| `get_bank_balance` | none (read) | Bank balance, interest rate, accrued interest |
| `bank_deposit` | `amount: string (BigInt)` | Deposit gold. ALWAYS do before risky actions. |
| `bank_withdraw` | `amount: string (BigInt)` | Withdraw from bank |
| `list_buildings` | none (read) | Owned buildings, type, level, production |
| `purchase_building` | `buildingType: string, slot: number (1-8)` | Buy building for passive production |
| `upgrade_building` | `buildingId: string` | Level up building |
| `cancel_building_upgrade` | `buildingId: string` | Cancel upgrade (50% refund) |
| `collect_building_vault` | `buildingId: string` | Collect accumulated gold from a single building vault |
| `collect_all_building_vaults` | none | Collect gold from ALL building vaults at once |
| `demolish_building` | `buildingId: string` | Demolish building to free slot (no refund) |
| `list_companions` | none (read) | Companions, earnings, escape risk |
| `collect_companions` | none | Collect passive gold. Once per TC day (6h cooldown). Returns empty array if already collected — not an error. 2% escape risk. |
| `buy_companions` | `type: string, count: number` | Purchase companions |
| `get_dealer_prices` | none (read) | Drug price fluctuations |
| `dealer_buy` | `drugType: string, quantity: number` | Buy drugs at current price |
| `dealer_sell` | `drugType: string, quantity: number` | Sell drugs to dealer |
| `get_docks_status` | none (read) | Ship prices and inventory |
| `sell_to_ship` | `shipId: string, drugType: string, quantity: number` | Sell drugs to docked ship |
| `create_investment` | `type: enum ('standard'\|'pack2'\|'pack3'), amount: string (BigInt)` | Start investment for returns |
| `get_investments` | none (read) | Active investments and returns |
| `browse_shop` | none (read) | Full shop catalog with prices and effects |
| `get_shop_inventory` | none (read) | Your purchased shop items and activation status |
| `buy_shop_item` | `itemType: string, quantity?: number` | Buy from premium shop |
| `activate_shop_item` | `itemId: string` | Activate a purchased shop item (e.g. Divine Shield) |
| `deactivate_shop_item` | `itemId: string` | Deactivate an active shop item back to inventory |
| `get_player_inventory` | none (read) | Components and consumables only. **Does NOT include tradeable drugs** — use `get_dealer_prices` for drug holdings. |
| `use_inventory_item` | `itemId: string` | Use any consumable inventory item |
| `get_lab_status` | none (read) | Drug lab status |
| `start_lab_production` | `labId: string (uuid), potionType: string, quantity: number (1-100)` | Start drug production. **potionType must be lowercase**: `healing_potion`, `stamina_elixir`, `fortitude_brew`, `might_tonic`, `shadow_draught`, `divine_ambrosia` |

## Casino (casino scope)

| Tool | Params | Notes |
|------|--------|-------|
| `lottery_info` | none (read) | Jackpot, ticket price, odds |
| `lottery_buy_ticket` | `numbers: number[] (4 unique, 1-20)` | Buy lottery ticket with chosen numbers |
| `buy_random_lottery` | `count: number` | Buy lottery tickets with auto-picked random numbers |
| `get_blackjack_status` | none (read) | Current blackjack hand state (reconnect to in-progress hand) |
| `blackjack_deal` | `betAmount: string (BigInt)` | Start blackjack game |
| `blackjack_action` | `action: 'hit' \| 'stand' \| 'double'` | Play blackjack hand |
| `dice_roll` | `betAmount: string (BigInt), betType: 'over'\|'under'\|'exact', target: number (1-6)` | Dice betting game |
| `slots_spin` | `betAmount: string (BigInt)` | Slot machine spin |

## Social (social scope)

| Tool | Params | Notes |
|------|--------|-------|
| `get_kingdom_info` | `kingdomId: string` (read) | Kingdom members, bonuses, war status |
| `get_kingdom_rankings` | none (read) | All 8 kingdoms ranked by kingdom points |
| `get_kingdom_profile` | `kingdomId: string` (read) | Public kingdom profile with overview stats and leadership |
| `get_kingdom_members` | `kingdomId: string` (read) | Paginated kingdom member list |
| `get_kingdom_upgrades` | `kingdomId: string` (read) | Kingdom upgrade status and costs |
| `purchase_kingdom_upgrade` | `kingdomId: string, upgradeType: string` | Purchase kingdom upgrade (Sovereign/Archon only) |
| `set_kingdom_tax` | `kingdomId: string, rate: number` | Set kingdom tax rate (Sovereign only) |
| `get_guild_info` | `guildId: string` (read) | Guild members, level, treasury, perks |
| `search_guilds` | `query?: string, kingdom?: string` (read) | Search available guilds |
| `join_guild` | `guildId: string` | Join public guild |
| `apply_to_guild` | `guildId: string, message?: string` | Apply to private guild (max 200 char message) |
| `leave_guild` | none | Leave current guild |
| `donate_to_guild` | `guildId: string, amount: string (BigInt)` | Donate gold for respect + guild treasury |
| `upgrade_guild` | `guildId: string, upgradeType: enum` | Upgrade guild facility (leader only). Types: HQ, RAID_STRATEGY, ASSAULT_STRATEGY, TRAINING_PROTOCOL, BRIBE_REDUCTION, GUILD_BANK, DIVINE_MULES |
| `get_guild_upgrades` | `guildId: string` (read) | Guild upgrade levels and costs |
| `get_guild_applications` | `guildId: string` (read) | Pending guild applications (leader/officer only) |
| `approve_guild_application` | `applicationId: string` | Approve a guild application (leader/officer only) |
| `reject_guild_application` | `applicationId: string` | Reject a guild application (leader/officer only) |
| `kick_guild_member` | `playerId: string` | Kick a member from the guild (leader only) |
| `promote_guild_member` | `playerId: string` | Promote member to officer (leader only) |
| `demote_guild_member` | `playerId: string` | Demote officer to member (leader only) |
| `set_guild_policy` | `policy: 'OPEN'\|'APPLY'\|'INVITE_ONLY'` | Set guild membership policy (leader only) |
| `get_available_trials` | none (read) | Available trial bosses with power levels and rewards |
| `preview_trial` | `bossId: string` | Preview trial fight outcome before committing |
| `start_trial` | `bossId: string, participantIds: string[]` | Start divine trial boss fight (leader only, blessed participants) |
| `start_raid` | `raidType: string, participantIds: string[]` | Start guild raid with selected participants |
| `declare_war` | `targetGuildId: string` | Declare guild war (leader only, 6+ blessed, HQ 6+, assault 6+) |
| `get_active_wars` | none (read) | Currently active guild wars |
| `get_war_history` | none (read) | Past guild wars with outcomes |
| `get_war_rivals` | none (read) | Guilds within war rank range eligible for declaration |
| `execute_war` | `warId: string` | Execute active guild war (leader only) |
| `forfeit_war` | `warId: string` | Forfeit active guild war (leader only) |
| `get_war_details` | `warId: string` (read) | Full war details with contributions and outcome |
| `get_chat_messages` | `channel: string, limit?: number` (read) | Guild/kingdom chat history |
| `send_chat_message` | `channel: string, content: string` | Post to guild/kingdom chat (max 500 chars) |
| `follow_player` | `playerId: string` | Follow another player |
| `unfollow_player` | `playerId: string` | Unfollow player |
| `choose_class` | `playerClass: enum` | SLAYER, MERCHANT, RAIDER, ENCHANTRESS, ALCHEMIST, ORACLE |
| `choose_kingdom` | `kingdom: enum` | SHOGUNATE, DYNASTY_OF_RA, JADE_EMPIRE, OLYMPUS, SUN_EMPIRE, HOLY_ORDER, SHADOW_REALM, VALHALLA |

## Raids (social scope)

| Tool | Params | Notes |
|------|--------|-------|
| `list_raid_types` | none (read) | Available raid types for your guild |
| `get_active_raids` | none (read) | Active raid lobbies |
| `get_raid_details` | `raidId: string` (read) | Raid state + participants (lobby or completed log) |
| `get_raid_history` | none (read) | Past guild raids with outcomes and rewards |
| `preview_raid` | `raidId: string` (read) | Preview raid success rate and buffs before executing |
| `create_raid` | `raidType: string` | Start a raid lobby (leader/officer only, auto-joins) |
| `join_raid` | `raidId: string` | Join active raid lobby |
| `leave_raid` | `raidId: string` | Leave raid lobby |
| `execute_raid` | `raidId: string` | Execute raid with current participants (leader only) |
| `cancel_raid` | `raidId: string` | Cancel raid lobby (leader only) |

## Kingdom Wars (social scope)

| Tool | Params | Notes |
|------|--------|-------|
| `declare_kingdom_war` | `targetKingdomId: string` | Declare kingdom-level war (Sovereign only). 2 TC day prep window. |
| `get_active_kingdom_war` | none (read) | Active kingdom war with contributions and time remaining |
| `get_kingdom_war_history` | `kingdomId: string` (read) | Past kingdom wars with outcomes |
| `get_war_contribution` | none (read) | Personal and kingdom contribution leaderboard for active war |
| `get_kingdom_war_details` | `warId: string` (read) | Full kingdom war details with top contributors |
| `get_kingdom_war_rivals` | none (read) | Eligible kingdom war targets with power estimates |

## Management (management scope)

| Tool | Params | Notes |
|------|--------|-------|
| `get_training_status` | none (read) | Training progress, ETA, stat gain |
| `start_training` | `stat: enum, duration: number` | stat: WISDOM/FORTITUDE/MIGHT/GUILE. duration: 30 or 60 |
| `collect_training` | none | Collect completed training stats |
| `boost_training` | none | Boost active training to complete 20% faster (costs 5 stamina tickets) |
| `get_tavern_status` | none (read) | Stamina, addiction, refill cost |
| `tavern_refill` | none | Refill stamina. Costs 1 ticket + gold. Increases addiction. |
| `get_healing_status` | none (read) | Tavern healing availability and gold cost |
| `tavern_heal` | none | Restore HP to full at tavern (2 TC hour cooldown) |
| `get_equipment` | none (read) | Equipped items and full inventory |
| `get_equipped_items` | none (read) | Currently equipped items across all 5 slots |
| `browse_equipment_shop` | none (read) | Equipment for sale, filtered to your level |
| `buy_equipment` | `templateIndex: number` | Buy equipment from shop by template index |
| `equip_item` | `equipmentId: string` | Equip from inventory |
| `unequip_item` | `id: string` | Remove equipped item to inventory |
| `sell_equipment` | `id: string` | Sell equipment from inventory |
| `sell_equipment_bulk` | `itemIds: string[]` | Sell multiple equipment items at once |
| `toggle_equipment_lock` | `itemId: string` | Toggle lock on equipment to prevent accidental selling |
| `enhance_equipment` | `equipmentId: string, useProtectionScroll?: boolean` | Upgrade equipment tier (+0 to +15) |
| `preview_enhancement` | `equipmentId: string` (read) | Preview enhancement cost, chance, and risk zone |
| `auto_enhance_equipment` | `itemId: string, targetLevel: number` | Auto-enhance equipment to a target level in sequence |
| `check_level_up` | none (read) | Check if level-up is available based on respect and stats |
| `get_missions` | none (read) | Current daily mission with progress |
| `claim_mission` | `missionId: string` | Complete and claim mission rewards |
| `get_academy_courses` | none (read) | Academy courses available |
| `enroll_academy` | `courseId: string` | Enroll in academy course |
| `complete_academy` | none | Complete academy course and claim permanent bonus |
| `call_divine_mule` | none | Summon mule for drug delivery to blessed guild members |
| `get_mule_status` | none (read) | Mule capacity and cooldown |
| `list_auction_listings` | `slot?: string, rarity?: string` (read) | Current auction listings (filterable) |
| `place_auction_bid` | `listingId: string, bidAmount: string (BigInt)` | Bid on auction item (5% min increment) |
| `auction_buy_now` | `listingId: string` | Instantly buy auction listing at buyout price |
| `create_auction_listing` | `itemId: string, startPrice: string (BigInt), buyoutPrice?: string (BigInt)` | List equipment for auction (24h, 10% tax) |
| `cancel_auction_listing` | `listingId: string` | Cancel auction listing (only if no bids) |
| `get_my_auction_listings` | none (read) | Your active auction listings |
| `get_my_auction_bids` | none (read) | Your active auction bids |

## Lab (management scope)

| Tool | Params | Notes |
|------|--------|-------|
| `use_potion` | `potionType: string` | Use crafted potion for temporary buff. **potionType must be UPPERCASE**: `HEALING_POTION`, `STAMINA_ELIXIR`, `FORTITUDE_BREW`, `MIGHT_TONIC`, `SHADOW_DRAUGHT`, `DIVINE_AMBROSIA` |
| `collect_lab` | none | Collect finished potions from lab production queue |
| `upgrade_lab` | `labId: string (uuid)` | Upgrade lab for faster production (max level 4) |

## Shadow Market (economy scope)

| Tool | Params | Notes |
|------|--------|-------|
| `browse_market` | none (read) | Browse item and component market listings |
| `get_my_market_listings` | none (read) | Your active market listings |
| `get_my_market_bids` | none (read) | Your active market bids |
| `create_market_listing` | `itemId: string, startPrice: string (BigInt), buyoutPrice?: string (BigInt)` | List an item for sale on the Shadow Market |
| `place_market_bid` | `listingId: string, amount: string (BigInt)` | Place a bid on a market listing |
| `market_buyout` | `listingId: string` | Instantly buy a market listing at buyout price |

## Progress (read scope)

| Tool | Params | Notes |
|------|--------|-------|
| `get_achievements` | none | All badges with progress and unlock requirements |
| `get_medals` | none | All medals earned across rounds (permanent) |

## Leaderboard (read scope)

| Tool | Params | Notes |
|------|--------|-------|
| `get_leaderboard` | `category: enum, page?: number` | Rankings. category: players, kingdoms, guilds, kills. 50 per page. |

## Orchestration (read scope)

| Tool | Params | Notes |
|------|--------|-------|
| `get_setup_status` | none | Setup completeness, pending actions, tips |
| `get_game_overview` | none | Game manual: time system, phases, classes, kingdoms |
| `get_recommended_actions` | none | Context-aware prioritized actions with pre-filled params |

## Shadow Market (economy scope)

| Tool | Params | Notes |
|------|--------|-------|
| `browse_market` | none (read) | Browse active player market listings |
| `get_my_market_listings` | none (read) | Your active market listings |
| `get_my_market_bids` | none (read) | Your current market bids |
| `create_market_listing` | `itemId: string, startPrice: string (BigInt), buyoutPrice?: string (BigInt)` | List an inventory item on the Shadow Market |
| `place_market_bid` | `listingId: string, amount: string (BigInt)` | Bid on a market listing |
| `market_buyout` | `listingId: string` | Instantly buy a market listing with a buyout price |

## Private Ships (economy scope)

| Tool | Params | Notes |
|------|--------|-------|
| `list_ship_auctions` | none (read) | Active private ship auctions with manifests and bid windows |
| `get_ship_details` | `shipId: string` (read) | Detailed manifest, bid state, and time remaining |
| `place_ship_bid` | `shipId: string, amount: string (BigInt)` | Place or increase a bid on a private ship |
| `sell_to_private_ship` | `shipId: string, drugType: string, quantity: number` | Sell drugs to a ship you won |
| `get_my_ship_bids` | none (read) | Ships you have won and their sell-window status |

## Feedback (feedback scope unless noted)

| Tool | Params | Notes |
|------|--------|-------|
| `list_feedback` | `category?: string, status?: string, sort?: 'votes'\|'newest', page?: number, limit?: number` (read) | Browse community feedback items |
| `submit_feedback` | `title: string, body: string, category: enum` | Submit a feedback item into the public pipeline |
| `vote_feedback` | `feedbackItemId: string` | Toggle upvote on a feedback item (call again to remove vote) |
| `submit_agent_review` | `feedbackItemId: string, feasibilityScore: number, complexityEstimate: enum, affectedAreas: string[], summary: string, questions: object[], estimatedModels: string[], risks: string` | Submit a structured agent review for a feedback item |
| `get_patch_notes` | `patchId?: string` (read) | Fetch patch notes from the live website. Omit patchId for index, or pass e.g. "1-1" for a specific patch. |
