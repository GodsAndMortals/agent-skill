# Phase Mechanics

**What this is:** the fixed game physics of each round phase — unlocks, gates, decay, maturity windows. What the agent can't figure out from tools alone.

**What this is not:** a playbook. This file deliberately does not say "do X then Y." Strategy is the user's decision (see `SKILL.md §Strategy is the user's decision` and `references/rewards.md` for what the round pays out for). Use this reference to reason about *what is possible* in the current phase, not to follow a prescribed loop.

A round is 52 TC days (13 real days). 1 TC day = 6 real hours.

## Phase boundaries

| Phase | TC Days | Real days (approx) |
|---|---|---|
| Genesis | 1–4 | 0–1 |
| Expansion | 5–14 | 1–3.5 |
| Conquest | 15–34 | 3.5–8.5 |
| Endgame | 35–45 | 8.5–11.25 |
| Ragnarok | 46–52 | 11.25–13 |

## Unlocks by phase

Each row is *newly available* in that phase; prior-phase features carry forward.

### Genesis (TC 1–4)
- Class + kingdom selection (irreversible — `choose_class`, `choose_kingdom` confirm first)
- Heists — all tiers you qualify for by level
- Training — 30-min sessions
- Banking (interest accrues from tick 1)
- Tavern refill + tavern heal
- Shop (basic tier items) + equipment shop
- Companions (low tiers)
- Hunting grounds
- Academy enrollment
- Starting PvP **protection** is active for most players — assaults blocked until level 7 or `remove_protection` (irreversible this round)

### Expansion (TC 5–14)
- Buildings unlock (all 7 types, level requirements still apply per building)
- Guilds open for joining (level 3+ to join; guild creation has its own gate)
- Higher-tier companions
- Academy courses become worthwhile (apprentice → journeyman progression viable)
- Drug lab
- Shadow market + auction house
- Investments — **check maturity window** before creating; maturity time is in the tool response
- Divine Mule (leader-triggered for blessed members)

### Conquest (TC 15–34)
- **PvP assaults available by default** (protection auto-drops; `remove_protection` only matters pre-Conquest)
- Raids (guild-coordinated; requires active guild)
- Trials (guild boss fights, blessed members only)
- Guild wars (declaration prerequisites: HQ 6+, assault upgrade 6+, 6+ blessed members)
- Kingdom wars (2 TC day prep window, auto-resolves via economy tick)
- District challenges (permanent round bonuses)
- Higher-tier auctions populate as endgame gear circulates

### Endgame (TC 35–45)
- **Training 60-min sessions** at level 5 + 20 sessions completed
- **Training 480-min sessions** at level 10 + many sessions completed
- Nothing else structurally new — this phase is defined by maturity windows closing, not by unlocks

### Ragnarok (TC 46–52)
- Nothing new unlocks. All remaining systems convert into final respect, wealth, and medal positioning.

## Maturity & decay windows — the Endgame/Ragnarok math

These are the mechanical constraints that shift what's *possible* late-round. Not prescriptions — inputs for the user's decision.

- **Investments** — each has a maturity time (see `get_investments`). Investments that mature after TC day 52 return **principal only** (no profit). Verify before creating.
- **Buildings** — payback period = build cost ÷ net gold per tick. If payback > remaining ticks, the building is a net loss this round. Prestige + demolish rules differ; `upgrade_building` may be cheaper ROI.
- **Academy courses** — bonuses apply only for the current round. Master-level Academy completion near round end may not recoup its 130M gold cost in remaining play.
- **Addiction decay** — −2.0 per economy tick (6 real hours). Decaying from addiction 20 → 0 takes ~10 TC days. Refilling heavily in Ragnarok means carrying penalties to round end.
- **HP regen** — 10% max HP per tick. Floor 10. In Ragnarok, every hospital stay is a full recovery cycle unless you pay `heal_hospital` (300 gold/sec remaining).
- **Drug stockpile** — persists only to round end. Private ship auctions and dealer spreads are the liquidation routes.
- **Gold on hand** — only gold on hand counts toward the Wealth medal in some prize categories and is exposed to assault loss; banked gold is safe and counts for other wealth metrics. Check the current rewards criteria (`references/rewards.md`) before deciding whether to bank or hold.
- **Stat gains** — training applies to respect weights at a multiplier that increases late-round (Divine Favor catch-up). See `resource-management.md`.
- **Cooldowns at round end** — Sacred Mist / Sacred Springs have 4 TC day cooldowns; Divine Shield lasts 4 TC hours. Time purchases so the duration actually covers the intended window.

## Irreversible decisions (always confirm with user)

- `choose_class` — locked for the round.
- `choose_kingdom` — locked for the round.
- `remove_protection` — cannot be restored this round. Only matters in Genesis/early Expansion; protection drops automatically in Conquest.
- `demolish_building` — refund is partial; the building slot reopens but the build cost is sunk.
- `forfeit_war` — war ends and rewards forfeit.

## What phase you're in — how to check

- `get_player_status` → `tcDay` field → cross-reference the table above.
- `get_game_overview` → includes current phase.
- Don't hard-code phase from real-world dates — rounds can be manually advanced or paused, and PENDING rounds may block actions.

## How to reason about phase without prescribing a loop

The user decides the strategy (see `SKILL.md` archetypes and `references/rewards.md`). The agent's job per cycle:

1. Establish what the user is optimizing for (once, at session start — persist the answer).
2. Read current state: `get_player_status`, `get_recommended_actions`, phase.
3. Filter the system map (`SKILL.md §System map`) to what's **unlocked now** (this file) and **relevant to the user's goal** (rewards.md + archetype).
4. Resolve blockers (jail/hospital/training complete) and bank before risk — these are game physics, not strategy.
5. Pick the action from the filtered set. Explain the choice if non-obvious.

This keeps decisions the user's call, while still guaranteeing the agent doesn't waste cycles on features that aren't available yet or that have closed their payback window.
