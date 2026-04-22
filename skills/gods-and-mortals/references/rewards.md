# Rewards & What the Round Pays Out For

**Purpose:** help the user + agent co-design a strategy around *what actually pays out*, instead of defaulting to a generic meta. This file describes the reward **structure** — categories, metrics, eligibility gates. Specific prize amounts and category lists change each round; the authoritative source is the current patch notes (`get_patch_notes`) and round-results pages on https://roolzgods.com/patchnotes.

## Three layers of rewards

1. **Round-end TON payouts** — real money, paid to a connected TON wallet after round close. Most playstyle-differentiated layer.
2. **Medals** — on-chain prestige, round-end. Blessed Guild Members only. Aggregated into career stats across rounds.
3. **Badges** — permanent achievements, unlock whenever thresholds are hit; persist across rounds.

An agent helping a user chase rewards must know *which layer* the user cares about. They optimize for different things.

## Layer 1 — Round-end TON payouts (per-round, configurable)

This layer is the one most worth designing a strategy around, because the categories are diverse and announced *before* the round starts (from Round 3 onward). A user who reads the categories early can pick a niche that few others are competing for.

### Category archetypes (observed from Round 2 results)

| Tier | Structure | Example metrics |
|---|---|---|
| Overall champions | Top N by total respect | Top 3 respect at round end |
| Class champions | Best-in-class by respect, excluding higher tiers | Top Slayer, top Raider, etc. |
| Kingdom of the Round | Top war contributors in the winning kingdom | War points in kingdom wars |
| Banner Carriers | Top player of each top-N guild | Guild total respect → guild's highest-respect member |
| Activity Titles | One prize per play pattern | Most PvP kills, longest streak, biggest single heist, most heists, most gold from heists, most private ships won, longest login streak, most achievements, top war contribution, most deaths, most sabotages, top guild contributor, most gross gold, best K/D, most attacks on top-10 players, first PvP kill, last PvP kill |
| Active Mortals | Flat prize for clearing an activity floor | Level ≥ 5 AND (kills > 0 OR gold earned > 10M OR guild points > 0) |

### Announced for Round 3 (additive — stack on top of returning categories)

| Category | Metric |
|---|---|
| Building Baron | Highest sum of building levels (incl. prestige) |
| Grand Architect | First to reach a max-level building |
| Docks Tycoon | Most drugs sold via Docks |
| Casino King | Highest single casino win |
| High Roller | Most gold wagered |
| Lady Luck | Best casino net result |
| Master Scholar | Most academy courses completed |
| Market Mogul | Best % ROI on investments |
| Tycoon | Largest single investment |
| Gear Collector | Most Legendary / Divine equipment owned |
| Grand Enhancer | Highest enhancement level reached |
| Trial Conqueror | Hardest Divine Trial boss cleared |
| Raid Commander | Most successful raids led |
| Lottery Champion | Winner of each lottery draw |
| Mule Master | Most mule deliveries called |
| Alchemical Overlord | Most potions brewed |
| Drug Lord | Most drugs stockpiled at any snapshot |
| First Settler | First to hit each milestone (level 5, 10, 14, first heist, first kill) |

### Eligibility & integrity rules (as of Round 2)

- **TON wallet must be connected** before round close to receive payouts. Unclaimed prizes roll forward to the next round's pool.
- **Cascading**: if one player would claim multiple ranked tiers, the lower tier rolls down to the next eligible player so the pool spreads. An agent should not assume "I placed in Champions so I also auto-win Class Champion."
- **Fair-play review**: wash trading and self-farming are not paid. Redistributed prizes go to the next legitimate placer.
- **First-come categories** (First Blood, Last Stand, First Settler) are timestamp-races, not rank races — different strategy profile.
- **Activity floor** (for Active Mortals tier): level ≥ 5 AND (kills > 0 OR gold earned > 10M OR guild points > 0). Any user should clear this as a baseline.

### Strategy implication — how to co-design with the user

Before committing a playstyle, check the **current round's announced categories** via `get_patch_notes` or the round-results URL. Categories change round-to-round. The value of a niche category = prize ÷ likely competition. Some observations:

- **Narrow-metric titles** (most potions brewed, longest login streak, most mule deliveries) have few competitors by design. A user committing to Alchemical Overlord may win $1 with far less respect than a Champion runner-up.
- **First-X categories** reward being online at round start, not grinding.
- **Gross-value categories** (total gold earned, most wagered) reward volume, not efficiency.
- **Best-at-class** lets a minority-class player (Oracle, Alchemist, Merchant) win with lower absolute respect than the top Slayer.

Ask the user what they want *most* — money, prestige, or fun — and match them to the matching layer. Avoid auto-defaulting to "top 3 respect" just because that's the headline category.

## Layer 2 — Medals (round-end, Blessed only)

Medals are on-chain prestige, awarded at round end. **Only Blessed Guild Members are eligible.** Becoming Blessed is therefore a gate for this entire layer.

| Category | Metric |
|---|---|
| RESPECT | Total respect at round end |
| WEALTH | Total gold (hand + bank) |
| COMBAT | Kill count |
| ECONOMY | Total stats (wisdom + fortitude + might + guile) |
| SOCIAL | Guild points contributed from raids |

Top 3 Blessed players per category get Gold/Silver/Bronze. Career stats aggregate medals across rounds.

Strategy implication: if medals are the user's goal, Blessed status is load-bearing. Blessed comes from guild leadership granting it — check `get_guild_info` for current blessing state and the guild's Divine Mules / Mule-related upgrades.

## Layer 3 — Badges (permanent, threshold-based)

Badges unlock the moment you hit the threshold and persist across rounds. They're a *checklist*, not a race — the user can chase them at their own pace.

Categories (see `resource-management.md §Achievements & Medals` for full list):

- **Combat**: kill-streak tiers (Sicario I–VIII), cumulative kill tiers, per-day best
- **Economy**: round-gold tiers, building master, largest heist, dock sales rank, investment counts
- **Social**: guild leadership, guild-points contribution tiers, kingdom-war tiers, blessed status, referrals
- **Special**: throne-holder, top killer in kingdom, round winner, consecutive-max-level, class supreme, bug report / idea contribution

Use `get_achievements` to read current progress. A Completionist archetype works backward from the nearest unlocked-threshold.

## How to use this file

- **At session start**: if the user hasn't picked a reward target, summarize the three layers (money vs. prestige vs. checklist) and ask.
- **Before committing a play loop**: check `get_patch_notes` for the *current round's* category list. This file describes the template; the live data is authoritative.
- **Cross-reference with archetypes** in `SKILL.md`: a Trader archetype may win "Market Mogul" more reliably than "Champions"; a Gambler is the natural fit for Casino King / Lady Luck; a Completionist chases badges, not TON prizes.
- **Be skeptical of "the meta"**: Round 2's winners were heavy on Slayers/Shogunate because the category weights favored respect from PvP. Round 3's expanded categories re-weight the incentive landscape. The best strategy this round may be entirely different.

## Source of truth

This file is a structural reference. For specific prize amounts, current-round categories, and eligibility windows always check:

- `get_patch_notes {patchId: "round-N-results"}` — previous round payouts
- `get_patch_notes` (index) — pre-round announcements of upcoming categories
- https://roolzgods.com/patchnotes/round-N-results — public page per round
