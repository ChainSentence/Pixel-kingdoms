# DarkThrone Foundation Research for Pixel Kingdoms

Status: working foundation draft. This file captures verified findings and direct source links found during the salvage pass after failed worker runs.

## Executive summary

DarkThrone was a 2004 browser-based fantasy strategy MMO by Lazarus Software. Its confirmed beta feature set was simple but sticky: choose a fantasy race, train citizens into economic/offensive/defensive/spy roles, equip armies with weapons/armor, form alliances, message other players, and compete in a persistent rankings environment.

For Pixel Kingdoms, the key reusable pattern is not the exact math; it is the browser-war loop:

1. passive population/resource growth,
2. train citizens into role-specialized units,
3. buy equipment that converts economic progress into combat power,
4. spend limited attack turns to raid rivals,
5. protect value through defense/banking/alliances,
6. climb public ranks while coordinating socially.

## Verified original-game source

### 2004 Wayback capture: official DarkThrone beta registration

Source: https://web.archive.org/web/20040701121716/http://darkthrone.com/

The archived official page states:

- title: `Dark Throne (beta)`
- copyright: `Copyright 2004 Lazarus Software`
- users could pre-register for the beta test
- races: `Undead`, `Humans`, `Goblins`, `Elves`
- train citizens as `miners`, `offensive or defensive soldiers`, and `spies`
- equip army with `weapons and armor`
- play with friends, create an alliance, communicate through an in-game message system
- create a character profile with custom avatar
- developer contact through DarkThrone forum

A similar capture exists at https://web.archive.org/web/20040603222359/http://www.darkthrone.com/ with the same feature list.

## Open-source / modern code findings

### DarkThrone Reborn

Source: https://github.com/MattGibney/DarkThrone

A public repository named `MattGibney/DarkThrone` describes itself as `DarkThrone Reborn`, an open-source Nx TypeScript monorepo containing:

- API backend
- main game client
- public website

The README describes it as an `Open source, test-based MMO` and gives local development instructions. This appears to be a modern rebuild/reimagining, not necessarily the original 2004 Lazarus Software source.

Relevant repo evidence inspected locally under:

`C:\Users\Jesse\OpenClawEvidence\research-then-resume-8a8b371457\DarkThrone`

Important implementation files inspected:

- `libs/game-data/src/index.ts`
- `apps/api/src/models/player.ts`
- `apps/api/src/controllers/attack.ts`
- `apps/api/src/controllers/training.ts`
- `apps/api/src/controllers/banking.ts`

### Similar inspired project: Stellar Dominion

Source: https://github.com/mungus451/Stellar-Dominion-Game

This repository explicitly says it is influenced by OpenThrone and DarkThrone Reborn and reimagines DarkThrone by Lazarus Software in a sci-fi setting. Its README describes a persistent turn-based multiplayer strategy game with resource management, tactical combat, alliances, banking, and a 10-minute turn cycle.

This is useful as a secondary design reference, not primary DarkThrone evidence.

## Game system model reconstructed from verified sources

### Player identity

Original official page confirms four races:

- Human
- Elf
- Goblin
- Undead

DarkThrone Reborn adds player classes:

- fighter
- cleric
- thief
- assassin

Reborn race/class mechanics from `apps/api/src/models/player.ts`:

- Humans and Undead: +5% attack strength
- Elves and Goblins: +5% defense strength
- Fighters: +5% attack strength
- Clerics: +5% defense strength
- Thieves: +5% gold per turn
- Assassin exists as a class type but no bonus was confirmed in the inspected code section

### Starting state in DarkThrone Reborn

From `libs/game-data/src/index.ts`:

- gold: 20,000
- gold in bank: 0
- attack turns: 1,000
- citizens: 100
- experience: 0
- fortification level: 0
- housing level: 0
- armoury level: 0

### Units

Original page confirms citizens can be trained as:

- miners
- offensive soldiers
- defensive soldiers
- spies

Reborn implementation uses:

- Citizen: support, no attack/defense, free, not trainable directly
- Worker: support, costs 1,000 gold, generates 50 gold/turn
- Soldier: offense, attack 3, costs 1,500 gold
- Guard: defense, defense 3, costs 1,500 gold

Training in Reborn consumes available citizens and gold, then creates or updates trained unit rows.

### Economy

Original game: confirmed economic role is `miners`. Reborn maps the economic worker role as `worker` producing gold per turn.

Reborn economy sources:

1. worker income: each worker produces 50 gold/turn
2. fortification income: fortification upgrades add flat gold/turn
3. thief class bonus: +5% gold per turn
4. raiding: successful attacks steal unbanked gold

Banking in Reborn:

- player can deposit gold into bank
- max deposit is 80% of current carried gold in inspected code
- banked gold is tracked separately as `gold_in_bank`
- withdraw restores banked gold to carried gold

Pixel Kingdom implication: use a split between exposed wallet/treasury and protected vault. Exposed resources create attack incentive; protected resources reduce rage-quits.

### Structures

Reborn has three structure tracks:

- Fortification
- Housing
- Armoury

Fortification:

- provides defense bonus percentage
- provides gold per turn
- long upgrade ladder from Manor to Empire tiers

Housing:

- increases citizens per day
- starts at Hovel, then Hut, Cottage, Longhouse, Manor House, Keep, Great Hall

Armoury:

- unlocks equipment access
- inspected early tier includes `Basic Armoury`

Pixel Kingdom implication: keep 3 clean upgrade lanes:

- Castle/Fortification = defense + kingdom status
- Housing/Villages = population growth
- Armoury/Forge = unit equipment and attack/defense scaling

### Equipment

Original page confirms armies can be equipped with weapons and armor.

Reborn equipment from `libs/game-data/src/index.ts`:

Offense examples:

- dagger: +25 offense, cost 12,500, sell 3,125
- padded hood: +6 offense, cost 3,000
- padded armor: +19 offense, cost 9,500
- padded boots: +6 offense, cost 3,000
- padded bracers: +3 offense, cost 1,500
- small wooden shield: +12 offense, cost 6,000

Defense examples mirror offense:

- sling: +25 defense, cost 12,500
- padded hood: +6 defense
- padded armor: +19 defense
- padded boots: +6 defense
- padded bracers: +3 defense
- small wooden shield: +12 defense

Implementation note: Reborn calculates item strength by combat unit type and item slot, assigning strongest applicable items up to the number of eligible units.

### Combat and defense

Original page confirms offensive soldiers, defensive soldiers, spies, weapons, armor.

Reborn attack controller rules:

- attacks require target ID and attack turns
- attack turns per attack must be 1 to 10
- player must have enough attack turns
- attacker must have non-zero attack strength
- target must exist
- attacker and target must be within 7 levels

Reborn strength formulas:

Attack strength:

- sum offensive unit attack strength
- plus offensive equipment bonuses
- apply race bonus if Human or Undead (+5%)
- apply fighter class bonus (+5%)
- floor result

Defense strength:

- sum defensive unit defense strength
- plus defensive equipment bonuses
- apply race bonus if Elf or Goblin (+5%)
- apply cleric class bonus (+5%)
- apply fortification defense percentage
- floor result

Victory condition:

- attacker wins if attack strength > defender strength

Reborn raid result:

- attack always spends requested turns
- if attacker loses: defender gains XP, no gold stolen
- if attacker wins: attacker steals a percentage of target carried gold
- inspected formula: total possible winnings = 80% of target carried gold; actual winnings = that amount * (0.1 * attackTurns)
- with 1 to 10 turns, this means attacks steal 8% to 80% of exposed carried gold if successful
- attacker gains randomized XP scaled by turns; defender gains XP when successfully defending

Pixel Kingdom implication: turns are the core anti-spam throttle and risk scaler. More turns should increase reward, but also create a strategic spend decision.

### Rankings / social

Original official page confirms:

- alliances
- in-game messaging
- custom profiles / avatars
- forum contact with developers

Reborn tracks overall rank and army size. The DAO comments indicate ranks are recalculated by cron/process eventually.

Pixel Kingdom implication: leaderboard should not be just net worth. Split rankings into:

- total kingdom power
- raid victories
- defense victories
- alliance power
- seasonal honor
- economy output

## What Pixel Kingdoms should adapt

1. Four race fantasy, but make each race readable at a glance.
2. Citizen conversion loop: idle population -> workers / attackers / defenders / spies.
3. Exposed-vs-protected economy: carried gold can be raided; vault/bank protects part of it.
4. Attack turns as the pacing throttle.
5. Equipment as a per-unit multiplier sink.
6. Fortification/housing/armoury as the three foundational building tracks.
7. Alliances as strategic social glue, not just chat.
8. Battle reports as content. Every attack should produce a shareable story.

## What Pixel Kingdoms should avoid

1. Pure spreadsheet UI. Pixel Kingdoms needs visual kingdom identity, not just numbers.
2. Overly harsh theft. Losing 80% of exposed gold can be exciting but also rage-inducing; consider caps, shields, cooldowns, or tiered exposure.
3. Hidden formulas. Players need understandable previews: attack power, defense power, potential loot, risk.
4. Unlimited snowballing. Add seasons, soft caps, level brackets, upkeep, or alliance balancing.
5. Pay-to-win credits. If there are premium credits, keep them cosmetic, convenience, season-pass, or capped boosts.

## First Pixel Kingdom system map

### Core resources

- Gold: primary spend/raid resource
- Citizens: population used for training
- Food or Energy: optional stabilizer for growth/upkeep
- Attack turns: raid stamina
- Honor: season/ranking score that cannot be directly bought

### Units

- Citizens: untrained population
- Workers/Farmers/Miners: generate resources
- Raiders/Knights: offense
- Guards/Wallsmen: defense
- Scouts/Spies: intel, sabotage, fog-of-war

### Buildings

- Castle/Fortification: defense multiplier, prestige, unlocks protection
- Housing/Village: population growth
- Forge/Armoury: equipment unlocks
- Market/Bank: deposit, trade, protection, fees
- Watchtower: spy defense / incoming attack visibility
- Alliance Hall: guild mechanics

### Combat loop

1. choose target in level/power range
2. choose attack-turn spend
3. preview expected attack power and possible loot range
4. resolve attack power vs defense power
5. produce battle report
6. transfer exposed loot only if attacker wins
7. grant XP/honor to winner; grant defense XP/honor to successful defender
8. update rankings and cooldowns

### ICP/Web3 angle

- Put season results, rare artifacts, land/kingdom identity, or alliance banners on-chain.
- Keep fast combat state efficient; avoid making every small action expensive.
- If tokens exist, avoid pay-to-win combat power. Use tokens for cosmetics, entry fees, governance, alliance treasuries, or seasonal prize pools.

## Open questions / next research targets

1. Find old player guides/forums that document the original DarkThrone formulas rather than Reborn formulas.
2. Find whether Lazarus Software ever published original code. So far: no verified original open-source release found.
3. Confirm whether `OpenThrone` was a clone/inspiration and whether its mechanics preserve original DarkThrone formulas.
4. Find screenshots of original UI for layout inspiration.
5. Determine whether original credits were premium credits, voting credits, referrals, or simply in-game gold/resources.

## Source list

- Official DarkThrone beta capture, 2004-07-01: https://web.archive.org/web/20040701121716/http://darkthrone.com/
- Official DarkThrone beta capture, 2004-06-03: https://web.archive.org/web/20040603222359/http://www.darkthrone.com/
- Wayback CDX capture index for darkthrone.com: https://web.archive.org/cdx?url=darkthrone.com/*&output=json&fl=timestamp,original,statuscode,mimetype,digest&filter=statuscode:200&collapse=digest&limit=50
- DarkThrone Reborn GitHub: https://github.com/MattGibney/DarkThrone
- DarkThrone Reborn README raw: https://raw.githubusercontent.com/MattGibney/DarkThrone/develop/README.md
- Stellar Dominion GitHub: https://github.com/mungus451/Stellar-Dominion-Game
