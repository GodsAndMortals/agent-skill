# Resource Management Deep Dive

Exact formulas and thresholds for all resource systems.

## Addiction

### Increase per tavern refill
```
increase = 0.5 + (currentAddiction x 0.04)
capped at 5.0 per refill
```
Examples: addiction 0 → +0.5, addiction 10 → +0.9, addiction 30 → +1.7, addiction 50 → +2.5

### Penalties
- **Heist success rate**: -0.5% per point, max -25% at addiction 50
- **Combat power**: -0.4% per point, max -20% at addiction 50
- **Training effectiveness**: via Spirit level (see below)
- **Stamina passive regen**: via Spirit level (see below)

### Decay
- -2.0 per economy tick (1 TC day = 6 real hours)
- Full recovery from max: ~25 TC days (worst case)
- Floors at 0

### Spirit Levels (from addiction)

| Addiction | Spirit | Training Modifier | Stamina Regen/Tick |
|-----------|--------|-------------------|-------------------|
| 0-2 | VERY_HIGH | +10% | 2.0 |
| 3-5 | HIGH | +5% | 1.5 |
| 6-10 | NORMAL | 0% | 1.0 |
| 11-20 | LOW | -5% | 0.5 |
| 21+ | VERY_LOW | -10% | 0.25 |

### Strategy
- **Below 5**: Optimal zone. Refill freely.
- **5-10**: Acceptable. Refill cautiously.
- **Above 10**: STOP refilling. Train instead. Wait for decay.
- **Above 20**: Emergency. All activities penalized. Do nothing but train and wait.

### Divine Shield
- Cost: 20,000,000 gold from shop
- Duration: 4 TC hours (6 real hours)
- Effect: Blocks ALL addiction gain from tavern refills
- Worth it when planning 50+ refills in a session at high addiction levels

## Stamina & Tickets

### Ticket Income
- Standard classes: 72 tickets per TC day, cap 300
- Raider class: 120 tickets per TC day, cap 380
- Starting tickets: 200

### Tavern Refill Cost
```
gold = 5,000 + 2,000 x floor(addiction) + 150 x floor(addiction)^2
tickets = 1
```
Examples:
- Addiction 0: 5,000 gold
- Addiction 5: 18,750 gold
- Addiction 10: 40,000 gold
- Addiction 20: 105,000 gold
- Addiction 30: 200,000 gold

### Stamina Points
- Refill restores to 100 stamina
- Each heist costs stamina based on heist difficulty (1-10 per heist)
- `use_all_stamina` auto-spends until stamina = 0 or max 50 heists

### Strategy

The conventional pattern is "spend tickets on heists first, train on the remainder" — tickets have a ceiling, so unspent tickets past the cap are wasted regen. But this isn't universal:
- **Raider / Balanced**: spend tickets on the highest-tier heist you qualify for; train with ticket depletion as the natural break
- **Ranker / Warlord**: stats → respect fastest; training may outweigh heist gold once gear/buildings cover economy
- **Tycoon**: building vault activity gates may demand a minimum heist cadence regardless of ticket efficiency
- **Universal**: keep a small buffer (~10 tickets) so `bribe_jail` or heist fallback is affordable after unlucky runs

## HP System

### Passive Regen
- 10% of max HP per economy tick (6 real hours)
- Safety net: HP never drops below 10

### Hospitalization
- Triggered when HP reaches 0 in combat
- BLOCKS all actions (training, heists, assaults, everything)
- Early release: `heal_hospital` — costs 300 gold/sec remaining
- NFT tier discount: up to 90% off

### Tavern Healing (when not hospitalized)
- Cost: 500 + 200 x level^2 gold
- Cooldown: 2 TC hours (30 real minutes)
- Restores HP to full

### Strategy
- Below 30% HP: avoid PvP assaults (risk of hospitalization)
- Hospitalized: heal IMMEDIATELY — lost time is lost progression
- Budget for heal costs when planning assaults

## Training

### Session Types

| Duration | Primary Stats | Secondary Stats | Unlock |
|----------|--------------|----------------|--------|
| 30 min | 2,400 | 600 | Always available |
| 60 min | 4,800 | 1,200 | Level 5 + 20 sessions |
| 480 min | 24,000 | 6,000 | Level 10 + many sessions |

### Modifiers
- Spirit level: -10% to +10% (from addiction)
- Guild Training Protocol: +bonus per level
- Divine Favor: catch-up multiplier
- Beast Venom: -10% per stack (max -30%)

### What Training Blocks
- Heists (attempt_heist, use_all_stamina)
- Assaults (execute_assault)
- Raids
- All combat actions

### What You CAN Do While Training
- Bank deposit/withdraw
- Collect companions
- Buy/sell drugs
- Manage buildings
- Check status
- Social actions (chat, guild)

### Strategy
- 30-min sessions are the most flexible — more frequent collect cycles, compatible with any play pattern
- Longer sessions (60-min, 480-min) suit AFK play or overnight runs where you can't intervene
- Stat choice is archetype-driven — common pairings: Slayer → MIGHT (assault power, 2× respect/stat), Merchant/Tycoon → WISDOM (economic multipliers), Warlord → MIGHT or FORTITUDE (survivability), Trader → WISDOM, Oracle/Ranker → user preference per their plan
- Don't leave completed training uncollected — the session is over but stats aren't applied until `collect_training`
- "Train only when tickets are depleted" is a common default, not a rule — Rankers may train earlier to prioritize stat-respect

## Gold & Banking

### Interest
- 2% per round (52 TC days) on banked gold
- Compounds during economy ticks

### Risk on Hand
- Failed heists: lose gold based on heist difficulty
- Lost assaults: attacker steals portion of gold on hand
- Sabotage: can target gold on hand

### Strategy
- Bank ALL gold before heists or assaults
- Keep 0 gold on hand during risky play
- Optional: keep 50,000 buffer for emergency jail bribes
- Withdraw only when buying something specific

## Building Vaults

Buildings do not pay gold directly to your wallet. Instead, each economy tick (1 TC day), net production (production minus maintenance) accumulates in a per-building **vault**.

### How Vaults Work
```
vaultCap = netGoldPerTick x 3   (VAULT_CAP_TC_DAYS = 3)
```
- Production accumulates in the vault each TC day, clamped at the cap.
- Once the vault reaches capacity, production pauses until you collect.
- Vault gold is safe from theft (assaults, sabotage) -- only gold on hand is at risk.

### Collection
- **Single building**: `collect_building_vault` -- collects from one building, transfers vault gold to your wallet.
- **All buildings**: `collect_all_building_vaults` -- iterates all buildings with vault gold > 0 and collects each.
- Collection is atomic (transaction-protected against race conditions).

### Activity Requirements

Each building type requires recent themed activity within a **3 TC day lookback window** before you can collect. Thresholds are intentionally low -- "you must play, not grind."

| Building Type | Required Activity | Threshold |
|---------------|-------------------|-----------|
| Thieves' Den | Successful heists | 15 |
| Harbor | Dock sales | 1 |
| Pleasure Quarter | Companion collections | 1 |
| Temple | Completed training sessions | 2 |
| Forge | Assaults completed | 5 |
| Golden Exchange | Investments made | 1 |
| Merchant Row | Shop transactions | 3 |

If the activity requirement is not met, collection is blocked for that building (others can still be collected).

### Strategy
- Collect once per TC day alongside companion collections to maximize efficiency.
- Never let vaults sit at cap -- you lose production while capped.
- Keep activity thresholds met naturally through regular play (heists, training, companions).
- Vault gold is safe from robbery, so there is no urgency to collect before risky actions.

## Auction House

The auction house enables player-to-player equipment trading. It is the primary endgame gold sink -- every sale permanently removes gold from the economy via tax.

### Fee Structure
- **Listing fee**: 1% of starting price (deducted when listing; waived by premium item `AUCTION_LISTING_FEE_WAIVER`)
- **Sale tax**: 10% of final sale price (deducted from seller proceeds)
- **Minimum starting price**: 10,000 gold
- **Minimum bid increment**: 5% above current bid
- **Maximum active listings**: 5 per player
- **Auction duration**: 24 real hours

### How It Works
1. **List**: `create_auction_listing` -- choose equipment, set starting price, optional buy-now price. Equipment is escrowed (removed from inventory). Buy-now price must exceed starting price.
2. **Browse**: `list_auction_listings` -- filter by slot, rarity, price range. Paginated results.
3. **Bid**: `place_auction_bid` -- bid must meet minimum increment. Previous highest bidder is automatically refunded.
4. **Buy Now**: `auction_buy_now` -- instantly purchase at the buy-now price (if set). Tax applies.
5. **Cancel**: `cancel_auction_listing` -- cancel if no bids placed. Equipment is returned; listing fee is NOT refunded.
6. **My Listings**: `get_my_auction_listings` -- view your active listings.
7. **My Bids**: `get_my_auction_bids` -- view your active bids and whether you are the highest bidder.

### Strategy
- Set buy-now prices ~20-30% above starting price to attract impulse buyers.
- Monitor `get_my_auction_bids` -- you are refunded automatically if outbid, but you lose the item if you don't re-bid.
- The 10% tax is a significant cost at endgame. Factor it into pricing.
- Equipment must be unequipped and unlocked before listing.
- Maximum 5 active listings -- prioritize your best items.

## Academy

The Academy offers cross-class courses that grant permanent stat bonuses. Each of the 6 classes teaches a course that players of OTHER classes can take (you cannot study your own class's course).

### Course Structure
- **3 levels per course**: Apprentice, Journeyman, Master
- **Cost scales by level**: 5M gold (Apprentice), 25M gold (Journeyman), 100M gold (Master)
- **Progress required**: 100 (Apprentice), 250 (Journeyman), 500 (Master)
- **Bonuses are permanent** for the round (reset with round reset)

### Available Courses

| Course | Source Class | Bonus Type | Apprentice | Journeyman | Master |
|--------|-------------|------------|------------|------------|--------|
| Principles of Commerce | Merchant | Building production | +3% | +6% | +10% |
| Art of the Heist | Raider | Heist reward | +5% | +10% | +15% |
| Ways of Enchantment | Enchantress | Companion earnings | +5% | +10% | +15% |
| Secrets of Alchemy | Alchemist | Drug production | +5% | +10% | +15% |
| Path of the Blade | Slayer | Assault power | +3% | +6% | +10% |

### Flow
1. `get_academy_courses` -- view available courses, enrollment status, and progress.
2. `enroll_academy` -- pay gold to enroll in a course at the next available level.
3. **Earn progress through gameplay**: heists/raids progress Raider course, assaults progress Slayer course, building actions progress Merchant course, drug activities progress Alchemist course, sabotage progresses Enchantress course.
4. `complete_academy` -- finalize the course once progress requirement is met.

### Progress Events

| Event | Course Progressed |
|-------|-------------------|
| Successful heist / raid | Art of the Heist |
| Assault win | Path of the Blade |
| Equipment enhancement | Path of the Blade |
| Sabotage success | Ways of Enchantment |
| Building collect / upgrade | Principles of Commerce |
| Drug sell / potion brewed / potion used | Secrets of Alchemy |

### Strategy
- Enroll in courses that complement your playstyle early in the round.
- Progress accumulates passively through normal gameplay -- no special grinding needed.
- Prioritize Apprentice level across multiple courses before pushing to Master in one (broader bonuses first).
- Budget 130M gold total per course to reach Master (5M + 25M + 100M).
- You CANNOT take your own class's course -- choose from the other 5.

## Achievements & Medals

### Badges
Badges are permanent achievements tracked across rounds. They unlock when you hit specific gameplay milestones.

**Combat Badges:**
- Sicario I-VIII: Kill streaks of 75, 150, 300, 500, 750, 1000, 1500, 3000 in a single round
- First Blood: First kill in a round
- Warrior/Warlord/Legend of Elysium: 100/500/1000 cumulative kills across all rounds
- Blade of the Day / Martyr of the Day: Most kills / most deaths in a TC day

**Economy Badges:**
- Golden Mortal/Champion/God: Earn 1B/10B/100B gold in a single round
- Master Builder: All building slots filled with Level 13 buildings
- Greatest Heist: Largest single heist reward in a TC day
- Dockmaster/Harbor Lord/Merchant Prince: Top 3 boat sales in a round
- Shrewd Investor / Oracle of Fortune: 10 investments in a round / 50 across all rounds

**Social Badges:**
- Guild Commander: Lead a Mortal Guild
- Loyal Subject / Pillar of the Kingdom: 1M/10M guild points contributed to Kingdom
- War Veteran/Champion/Legend: 100/500/2000 points in Kingdom Wars
- Blessed of the Gods: Become a Blessed Guild Member
- Herald of Elysium: Recruit 5 players via referral

**Special Badges:**
- Throne Bearer / God-King: Member / Sovereign of #1 kingdom at round end
- Blood Champion: Top killer in your kingdom at round end
- Ruler of the Age: Win a round (highest respect)
- GodFather: Reach Level 14 for 5 consecutive rounds
- Supreme [Class]: Highest-respect player of each class in a round
- Eye of the Overseer / Voice of the Realm: Bug report / idea contribution

### Medals
Medals are awarded at round end in 5 categories. Only **Blessed Guild Members** are eligible.

| Category | Metric | Gold/Silver/Bronze |
|----------|--------|-------------------|
| RESPECT | Total respect at round end | Top 3 blessed players |
| WEALTH | Total gold (hand + bank) | Top 3 blessed players |
| COMBAT | Kill count | Top 3 blessed players |
| ECONOMY | Total stats (wisdom + fortitude + might + guile) | Top 3 blessed players |
| SOCIAL | Guild points contributed from raids | Top 3 blessed players |

Career stats aggregate medals across all rounds: total rounds played, cumulative kills/deaths, best respect, best rank, and medal counts by category.

### Tools
- `get_achievements` -- view all badges with unlock status, progress, and targets.
- `get_medals` -- view medals earned across all rounds.

### Strategy
- Check `get_achievements` periodically to see which badges you are close to unlocking.
- Align your activity toward the next threshold (e.g., if at 60 kills in a streak, push for Sicario I at 75).
- Becoming a Blessed Guild Member is a prerequisite for medal eligibility -- prioritize this early.
- Medals persist forever as cross-round prestige markers. They are the primary measure of long-term success.
- War badges require active Kingdom War participation -- contribute via heists, raids, assaults, and training during wars.
