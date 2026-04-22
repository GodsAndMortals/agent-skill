# Phase Strategy Guide

A round has 5 phases. Unlocks shift; priorities shift with the user's archetype (see SKILL.md §Strategy is the user's decision). Each phase below lists what *unlocks*, what most archetypes find *high-leverage*, and how archetypes *tilt* — without prescribing one loop.

## Genesis (TC Days 1-4)

**Unlocks:** heists, training, banking, tavern, early shop, basic equipment, hunting grounds, academy enrollment, companion purchases (at low tiers).

High-leverage for most archetypes:
1. Pick class + kingdom (irreversible — confirm with user first)
2. Start a training cycle (stat choice depends on archetype; restart immediately on collect)
3. Early heists (PETTY_THEFT, MARKET_HEIST tiers) to bootstrap gold
4. Bank everything; keep a small buffer for emergencies

Commonly deprioritized (but not forbidden):
- Buildings — usually locked until Expansion
- PvP assaults — protection still up for most
- Casino — volatile on a small bankroll; Gamblers may still exposure-test
- Aggressive tavern refills — early addiction compounds across the round

Archetype tilt:
- **Tycoon**: buy the first building the moment it unlocks; start companion cycle
- **Trader**: start tracking dealer prices across cycles to model spreads
- **Gambler**: small lottery buys at current jackpot; low-cost exposure to endgame payout
- **Diplomat**: scout guilds early (`search_guilds`); the best ones fill fast

## Expansion (TC Days 5-14)

**Unlocks:** buildings, companions at higher tiers, guilds (level 3+), higher heist tiers as you level, academy courses, lab, shadow market, auction house, investments (mid-phase).

Commonly high-leverage:
1. Continue training — collect/restart each session
2. Heists at the highest tier you qualify for
3. First buildings aligned with your activity pattern
4. Join/apply to a guild
5. Collect companions + building vaults each TC day
6. Enroll in academy courses aligned with your archetype

Archetype tilt:
- **Raider**: `hire_mercenaries` as soon as eligible (200+ tickets, TC day 3+); sell drug drops via dealer/ships
- **Warlord / Ranker**: consider `remove_protection` for early PvP access (irreversible — confirm)
- **Trader**: drug arbitrage starts paying; track `get_dealer_prices` spreads
- **Diplomat**: apply to guilds with strong raid/war upgrades; start chat presence
- **Tycoon**: prestige any level-maxed building; upgrade active producers
- **Gambler**: first blackjack / dice sessions with defined bankroll

## Conquest (TC Days 15-34)

**Unlocks:** PvP assaults (default, no protection-removal needed), raids, trials, guild wars, kingdom wars, district challenges, higher-tier auctions.

Commonly high-leverage:
1. Maintain training + heist engine
2. Route activity into raids / trials / wars if in an active guild
3. Daily: collect companions, building vaults, missions

Archetype tilt:
- **Warlord / Ranker**: 2-3 assaults/day (3 for Slayer); target weak-stat + high-gold; districts for permanent bonuses
- **Tycoon**: building upgrades + prestiges; investments maturing
- **Trader**: lab output peaks; premium private-ship sales
- **Diplomat**: coordinate raids in chat; lead kingdom/guild wars
- **Completionist**: many badge thresholds fall in this phase — read `get_achievements` and route actions
- **Gambler**: jackpot grows; medium-stakes blackjack if bankroll allows

Safety:
- Bank before every assault (gold-on-hand is the loss pool)
- HP ≥ 30% before committing to PvP
- Respond to jail/hospital immediately; `tavern_heal` has a 2 TC hour cooldown so time it

## Endgame (TC Days 35-45)

**Unlocks:** advanced training durations (60-min at lvl 5+20 sessions, 480-min at lvl 10). Nothing else new.

Commonly high-leverage:
1. Training weight increases — stats become the primary respect source
2. Investments if they'll mature before round end (check maturity window)
3. Optimize building production chains
4. Drug lab output peaks; stock potions for Ragnarok combat

Commonly deprioritized:
- New buildings — payback window may be shorter than remaining round
- Long-maturity investments — verify they finish before TC day 52
- Low-EV casino tiers (relative expected value shrinks)

Archetype tilt:
- **Tycoon**: investments + mature auction sales
- **Warlord**: assault efficiency peaks as you out-scale most targets
- **Gambler**: lottery jackpot is near its ceiling — most rational buy-in window
- **Completionist**: final-mile badge pushes
- **Trader**: liquidate drug stockpile; shift to gold-heavy position

## Ragnarok (TC Days 46-52)

**Unlocks:** nothing new. Everything converts to respect, wealth, and medal positioning.

Commonly high-leverage:
1. Maximize stat training
2. Aggressive PvP — stat-steal is the fastest remaining respect source
3. Collect all passive income — no reason to save for later
4. Final guild donations (donation respect)
5. Liquidate auctions + shadow market; gold-on-hand + bank both contribute to Wealth medal

Commonly deprioritized:
- New buildings / investments that won't mature
- Long speculative positions

Archetype tilt:
- **Ranker / Warlord**: full PvP push; break glass on scrolls, potions, Divine Shield
- **Gambler**: Hail-Mary lottery if behind on rank
- **Completionist**: last-call thresholds — work backward from `get_achievements`
- **Trader**: convert everything to gold
- **Diplomat**: coordinate your kingdom's final war contributions

## Class-specific context (descriptive, not prescriptive)

Pick based on what you want to optimize. Each class has bonuses; none is "best" universally.

### Slayer
- 3 assaults/day (+1 vs others), 2× respect per stat point
- Academy: Path of the Blade (+3/6/10% assault power)
- Common pairings (meta-driven, not forced): MIGHT primary stat, SHOGUNATE kingdom
- Fit: Ranker, Warlord

### Merchant
- 6 building slots (vs 4), 40% building discount
- Academy: Principles of Commerce (+3/6/10% building production)
- Fit: Tycoon, Trader (economy lean)

### Raider
- +20% heist rewards, 120 tickets/day (vs 72)
- Academy: Art of the Heist (+5/10/15% heist reward)
- Fit: Raider, Trader (drug byproducts)

### Enchantress
- +15% companion earnings, cheaper guild upgrades
- Academy: Ways of Enchantment (+5/10/15% companion earnings)
- Fit: Diplomat, sabotage-heavy Warlord

### Alchemist
- +20% drug profits, faster lab production
- Academy: Secrets of Alchemy (+5/10/15% drug production)
- Fit: Trader, potion-stacking Warlord

### Oracle
- +1% investment returns, cheaper academy tuition
- Patient, economy-adjacent
- Fit: Tycoon, Gambler (investment edge), patient Ranker

## Kingdom picks

All 8 kingdoms are viable: SHOGUNATE, DYNASTY_OF_RA, JADE_EMPIRE, OLYMPUS, SUN_EMPIRE, HOLY_ORDER, SHADOW_REALM, VALHALLA.

Recent round data shows SHOGUNATE clusters strong Slayer/Raider players, but meta varies by round and kingdom upgrades shift balance. The user's kingdom affects:
- Shared treasury and upgrades (Sovereign-managed)
- Kingdom war participation
- Tax rate on certain activities
- Access to kingdom chat + politics

Ask the user if they have a preference (e.g. aesthetic, friends in a kingdom, political ambitions) before defaulting.
