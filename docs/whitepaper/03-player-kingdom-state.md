# Pixel Kingdoms Whitepaper

## Chapter 3 — Player + Kingdom State

**Status:** approved technical design chapter  
**Purpose:** define the stable backend state for a player kingdom while keeping gameplay effects, race flavor, rank names, balance tables, ads, and subscriptions outside the core stable records.

---

## Chapter Thesis

**A kingdom should store simple facts, not every rule that can ever apply to those facts.**

The stable state should answer:

```text
Who owns this kingdom?
What race/class did they choose?
How many citizens and resources do they currently have?
Which fixed buildings exist, and what rank are they?
Which generic minions exist, and what rank/amount are they?
What daily events are waiting to be shown?
```

The rule layer should answer:

```text
What does rank 3 forge do?
What is a Goblin rank 2 attacker called?
How many citizens does castle_core rank 4 generate today?
How does the Undead bone_pit convert captives?
What does an active ad boost or subscription automate?
```

This split protects stable state. New ranks, names, race effects, cosmetic options, balance changes, and monetization rules should usually be added by updating definitions and formulas, not by migrating every player record.

---

## Stable State Design Rule

Pixel Kingdoms should avoid embedding balance logic directly into player records.

Preferred pattern:

```text
stable player state = type/kind + rank + amount
rule definitions = names, effects, unlocks, formulas, restrictions
```

Examples:

```text
Building { kind = "forge", rank = 2 }
```

The building record does not need to store what forge rank 2 does. The building definition table decides that.

```text
MinionStack { kind = "attacker", rank = 2, amount = 50 }
```

The minion stack does not need to store that Goblin attacker rank 2 displays as Hobgoblin. The race/class definition table decides that.

This is especially important for ICP/Motoko stable memory because gameplay is expected to evolve after launch.

---

## Global Game Calendar

Pixel Kingdoms should have a global day counter that starts at launch.

```text
GameClock {
  launchTimestamp
  secondsPerDay
}
```

The current day should be derived from time rather than manually updated forever:

```text
currentDay = ((now - launchTimestamp) / secondsPerDay) + 1
```

Day 1 begins when the game goes live. The day count then runs forever.

The day system powers:

```text
citizen generation
passive gold generation
captive escape rolls
attacks received summaries
mercenary returns
leaderboard seasons later
login reports
```

Each kingdom should track its own last resolved day.

```text
KingdomDailyState {
  lastResolvedDay
  lastReportSeenDay
  pendingDailyReport
}
```

This lets the backend catch a kingdom up when the owner returns instead of requiring every kingdom to be processed every second.

---

## Kingdom State

A kingdom is the main owned game object.

```text
Kingdom {
  kingdomId
  ownerPrincipal
  profile
  lord
  economy
  resources
  buildings
  army
  captives
  daily
  stats
  createdAt
  updatedAt
}
```

The kingdom owns the records needed to render and resolve gameplay, but each subsystem stays focused.

---

## Player Profile

The profile is the public identity layer.

```text
PlayerProfile {
  playerId
  ownerPrincipal
  name
  bio
  race
  level
  rank
  experience
  createdAt
  lastActiveAt
}
```

### Name

The display name of the player or kingdom ruler.

Names should fit leaderboards, battle reports, and kingdom cards.

### Bio

A short public one-liner.

```text
bioMaxLength = 140 characters
```

Examples:

```text
"Gold sleeps in my vault, not yours."
"Undead kingdom. No rest. No mercy."
"Builder first, raider second. Usually."
```

Bio text should be filtered and sanitized before display.

### Race / Class

Race belongs on the profile because it affects identity, available visuals, class building behavior, rank names, bonuses, and weaknesses.

Initial v1 races/classes:

```text
Undead
Wood Elves
Humans
Goblins
```

Chapter 4 should define the race-specific playstyles and class buildings in detail. Chapter 3 only stores the selected race and provides the neutral structures those rules will act on.

### Level / Rank / Experience

These are progression fields.

```text
level = long-term growth marker
rank = competitive position
experience = raw progression points
```

Rank may later be calculated from power, PvP performance, wealth, seasons, or another formula.

---

## Lord / Mascot State

Each player should have a small pixel character that represents the ruler of the kingdom.

The Lord/Mascot is both profile identity and race/class expression.

```text
Lord {
  race
  skinTone
  hairType
  hairColor
  shirt
  pants
}
```

The stable state stores selected option keys only. The allowed cosmetic options live in definitions.

Examples:

```text
Human skin tones = multiple human skin palettes
Goblin skin tones = green-based goblin palettes only
Undead skin tones = bone, grey, pale, spectral palettes
Wood Elf skin tones = elf/nature palettes
```

This keeps cosmetics expandable. New shirts, pants, hair, palettes, and profile icons can be added without changing the Lord struct.

Design split:

```text
race/class = gameplay identity
appearance = visual identity
```

---

## Population / Citizens

Pixel Kingdoms should use large full-number citizen counts.

```text
PopulationState {
  citizens
  lastCitizenClaimAt
}
```

Citizens should be displayed as full numbers, not hidden thousand-units.

Examples:

```text
12,450 citizens
125 citizens lost
10,000 citizens gained
```

Large numbers make percentage losses easier to understand and balance:

```text
1% of 1,000 citizens = 10 citizens
1% of 50,000 citizens = 500 citizens
```

This avoids awkward fractional-person logic.

### Castle Core Citizen Generation

The `castle_core` building should drive daily citizen growth.

Example definition shape:

```text
castle_core rank 1 = +1,000 citizens/day
castle_core rank 2 = +2,500 citizens/day
castle_core rank 3 = +5,000 citizens/day
castle_core rank 4 = +10,000 citizens/day
castle_core rank 5 = +25,000 citizens/day
```

Exact values are balance data and can change later.

Ads and subscriptions should modify or automate claims from outside the building state.

---

## Resources

The v1 economy should stay simple.

Core resources:

```text
gold
captives
weapons by rank
armour by rank
```

Pixel Kingdoms should not add wood, stone, ore, or other construction materials in v1 unless a later chapter deliberately reopens that decision.

Gold should pay for the main economy:

```text
building upgrades
training
equipment
banking
repairs
mercenaries
future market actions
```

This keeps the game readable and reduces early balance complexity.

---

## Economy State

Economy state tracks the kingdom's gold position.

```text
EconomyState {
  goldOnHand
  goldInBank
  goldPerDay
  lastGoldUpdate
}
```

### Gold On Hand

Gold on hand is usable and risky.

```text
goldOnHand = exposed treasury
```

This is the gold enemies can potentially steal through attacks.

### Gold In Bank / Vault

Banked gold is safer storage.

```text
goldInBank = protected or partially protected wealth
```

Protection may depend on the vault building, race modifiers, attack rules, and future upgrades.

### Gold Per Day

Gold generation should align with the daily report system.

```text
goldPerDay = passive income from buildings, workers, modifiers, bonuses, and penalties
```

This may be cached for fast reads, but the source of truth should remain definitions + current state.

---

## Equipment Resources

Weapons and armour are resources divided by rank.

```text
EquipmentInventory {
  weaponsByRank
  armourByRank
}
```

Example conceptual representation:

```text
weaponsByRank[1] = 120 basic weapons
weaponsByRank[2] = 35 improved weapons
armourByRank[1] = 90 basic armour
armourByRank[2] = 10 improved armour
```

Weapons and armour should be generic in stable state. Race-specific names and visuals should live in definitions.

Example:

```text
rank 1 weapon = Rusted Sword / Bone Blade / Goblin Shiv depending race/flavor
rank 2 weapon = Steel Sword / Cursed Blade / Hobgoblin Cleaver depending race/flavor
```

Equipping enough ranked weapons/armour can upgrade minion stacks.

Example:

```text
attacker rank 1 + required rank 2 swords -> attacker rank 2
```

The stable minion stays generic. The display layer resolves the class/race name.

---

## Captive State

Captives should have their own state because they are not just a number.

```text
CaptiveState {
  workerCaptives
  soldierCaptives
  totalCapturedLifetime
  totalRanAwayLifetime
  ranAwaySinceLastSeen
}
```

### Captive Types

```text
workerCaptives = captives used for labor / mines / production
soldierCaptives = captives taken through combat or trained for riskier uses
```

This split gives the game room to make captives useful without overcomplicating v1.

### Escape Events

Captives should escape through daily events, not constant real-time leakage.

Daily resolution is easier to understand, easier to balance, and cleaner for UI.

Example report line:

```text
Day 17: 14 captive workers ran away.
```

Combat can still show captive changes directly in combat text:

```text
You captured 22 workers and 4 soldiers.
7 captives escaped during the retreat.
```

For non-combat workers/miners, escape should normally resolve during the daily tick.

Escape rate can later depend on:

```text
race
vault/prison/watchtower/walls
kingdom defense
recent attacks
class building effects
special statuses
```

Undead are the major exception planned for Chapter 4: they can sacrifice captives into loyal skeleton units through the Bone Pit class building.

---

## Building State

Buildings should be fixed HOMM3-style kingdom slots.

Every kingdom has the same canonical building positions. The player does not freely place arbitrary building types in arbitrary slots for v1.

Stable building state should be simple:

```text
Building {
  kind
  rank
}
```

Optional if needed for indexing:

```text
slot
```

The meaning of rank lives in building definitions.

```text
BuildingDefinition {
  kind
  displayName
  maxRank
  rankEffects
  upgradeCosts
  unlockRules
}
```

### V1 Fixed Building Set

Recommended core building slots:

```text
castle_core
forge
armoury
training_grounds
mine
class_hall
vault
watchtower_or_walls
```

### Castle Core

The kingdom growth building.

Primary role:

```text
increases citizens/day
may unlock higher building ranks
may increase population capacity later
```

### Forge

Weapon production and weapon rank progression.

### Armoury

Armour production and defensive equipment progression.

### Training Grounds

Minion upgrades, training throughput, and combat readiness.

### Mine

Gold production. Since v1 uses gold-only economy, the mine does not need to produce ore.

### Class Hall

Race/class-specific rule-breaker building.

Examples to define in Chapter 4:

```text
Undead = bone_pit
Wood Elves = living grove / elder grove
Humans = mercenary hall / guild hall
Goblins = plunder den / scrap pit
```

### Vault

Gold protection and possibly captive containment later.

### Watchtower / Walls

Anti-siege defense, attack warning, and defensive modifiers.

Exact naming can be decided in the UI/theme pass.

---

## Army / Minion State

Minions should also stay generic in stable state.

```text
MinionStack {
  kind
  rank
  amount
}
```

Recommended v1 minion kinds:

```text
worker
attacker
defender
```

### Worker

Used for mines, production, building support, and non-combat work.

### Attacker

Used for raids, siege, offensive combat, and scavenging danger.

### Defender

Used to protect the kingdom, vault, captives, and citizens.

### Race-Specific Names

Minion names should be resolved by race + kind + rank.

Example Goblin attacker names:

```text
rank 1 attacker = Goblin Warrior
rank 2 attacker = Hobgoblin
rank 3 attacker = Goblin Champion / Bugbear / TBD
```

Example Human attacker names:

```text
rank 1 attacker = Footman
rank 2 attacker = Knight
rank 3 attacker = Champion
```

The stored state remains:

```text
{ kind = "attacker", rank = 2, amount = 50 }
```

This lets the team rename, rebalance, or extend race flavor without migrating army state.

---

## Daily Kingdom Report

When a user logs in for the first time after a new day, the backend should resolve missed daily events and return a popup/report.

The report is the daily "what happened while you were gone" moment.

```text
DailyReport {
  fromDay
  toDay
  citizensGained
  goldEarned
  captivesCaptured
  captivesRanAway
  attacksReceived
  goldLost
  defendersLost
  attackersLost
  mercenaryIncome
  notes
  createdAt
  seenAt
}
```

Example popup:

```text
Day 43 Report
+12,000 citizens
+3,400 gold earned
-8 captives escaped
3 attacks received
-900 gold lost
-14 defenders lost
+500 mercenary income
```

The popup should be shown once, then marked seen.

Recommended state:

```text
KingdomDailyState {
  lastResolvedDay
  lastReportSeenDay
  pendingDailyReport
}
```

If a player misses multiple days, the game can either show one aggregated report or multiple day-by-day reports. For v1, one aggregated report is simpler.

---

## Ads and Subscriptions Stay Outside Core State

Ads and subscriptions should not be embedded into buildings.

Free player pattern:

```text
manual claim
optional ad watch for x2 citizen/gold claim
```

Subscriber pattern:

```text
auto-claim
ad-equivalent boost automation
possibly quality-of-life report automation
```

These benefits should live in account/benefit state or entitlement checks, not in building records.

```text
AccountBenefits {
  subscriptionTier
  adBoostEquivalent
  autoClaimEnabled
  boostUntil
}
```

This lets the game launch the core economy before monetization is finalized.

---

## Stats

Stats should remain separate from current resources.

```text
EconomyStats {
  goldEarnedLifetime
  goldStolenLifetime
  goldLostTotal
  goldSpentLifetime
  goldBankedLifetime
  goldWithdrawnLifetime
}
```

```text
WarfareStats {
  attacksMade
  attacksReceived
  wins
  defeats
  defensesWon
  defensesLost
  soldiersKilled
  soldiersLost
  goldStolenFromRaids
  goldLostToRaids
  captivesTaken
  captivesLost
  lastAttackAt
  lastDefendedAt
}
```

```text
ScavengeStats {
  attempts
  successful
  failed
  goldFound
  weaponsFound
  armourFound
  captivesFound
  soldiersLost
  lastScavengeAt
}
```

Important naming distinction:

```text
EconomyStats.goldLostTotal = all gold lost from any source
WarfareStats.goldLostToRaids = gold lost specifically from PvP raids
```

This keeps accounting clean for leaderboards and achievements.

---

## Public vs Private State

Not every field should be visible to everyone.

### Public by Default

```text
name
bio
race
level
rank
lord/mascot appearance
selected public stats
rough kingdom strength
```

### Owner Only

```text
exact gold calculations
exact army composition unless scouted
captives
private cooldowns
unseen daily reports
account benefits/subscription status
```

### Scout / Spy Reveal Later

```text
estimated gold on hand
estimated army strength
estimated defense strength
building hints
recent activity hints
```

Information should become part of strategy. A player should not automatically know everything about a target.

---

## Initial Kingdom Bundle

A newly created kingdom should initialize with:

```text
PlayerProfile
Lord
PopulationState
EconomyState
ResourceState
CaptiveState
BuildingState
ArmyState
KingdomDailyState
EconomyStats
WarfareStats
ScavengeStats
```

Suggested initial structures:

```text
ResourceState {
  equipment
}

BuildingState {
  buildingsByKind
}

ArmyState {
  minionStacks
}
```

Exact starting values belong in balance/config, not this chapter.

---

## Chapter 3 Decisions

Chapter 3 locks these technical directions:

```text
1. Keep stable state generic and rank-based.
2. Buildings are fixed HOMM3-style slots.
3. Building stable state is kind + rank.
4. Minion stable state is kind + rank + amount.
5. Race-specific minion names live in definitions.
6. Core v1 resources are gold, captives, ranked weapons, and ranked armour.
7. Skip wood/stone/ore for v1.
8. Captives have their own state and daily escape reporting.
9. Population uses large full-number citizen counts.
10. castle_core drives citizens/day.
11. The world starts at Day 1 on launch and runs forever.
12. Login after a new day shows a Daily Kingdom Report popup.
13. Ads and subscriptions are benefit/boost systems, not building state.
14. Lord/Mascot appearance stores selected option keys; allowed cosmetics live in definitions.
```

---

## What Moves to Chapter 4

Chapter 4 should define race identity and class building behavior.

Initial race direction:

```text
Undead = offensive snowball; sacrifice captives into loyal skeleton units; brittle bones defensive debuff
Wood Elves = defensive living-tree style protection; peaceful/no captive scavenging
Humans = balanced; possible mercenary/economic flexibility
Goblins = plunderers; more gold/weapons from attacks and scavenging; debuff TBD
```

Chapter 4 should also define:

```text
class_hall names per race
race-specific minion names per rank
race bonuses and debuffs
captives and scavenging rules per race
initial balance constraints
```

Chapter 3 is now the neutral kingdom skeleton. Chapter 4 is where the races get personality.
