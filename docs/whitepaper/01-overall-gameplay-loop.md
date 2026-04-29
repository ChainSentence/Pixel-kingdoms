# Pixel Kingdoms Whitepaper

## Chapter 1 — Overall Gameplay Loop

**Status:** working design chapter  
**Purpose:** define how players move through Pixel Kingdoms day to day, how idle growth connects to active decisions, and how PvP pressure turns kingdom building into a living strategy game.

---

## Chapter Thesis

**Pixel Kingdoms is built around a simple but powerful loop: grow your kingdom, expose wealth, choose how to convert that wealth into power, and risk that power against nearby rivals. The game should be playable in short sessions, but deep enough that every kingdom slowly develops a recognizable identity.**

This chapter turns that thesis into the practical player loop.

Pixel Kingdoms is not meant to be a frantic clicker. It is a slow-burn kingdom game where players return throughout the day, make meaningful decisions with limited energy, and slowly shape their kingdom into something visible, dangerous, wealthy, defensive, or strange.

---

## The Core Loop

At the highest level, the loop is:

```text
generate gold -> spend gold + energy -> grow kingdom -> gain power -> target rivals -> attack/defend -> gain or lose value -> repeat
```

The player is always moving between two forces:

```text
growth and risk
```

Growth makes the kingdom stronger. Risk makes that growth meaningful.

Gold is the main engine of progression, but gold on hand is also the main PvP target. This means the player is never only asking, “How do I get richer?” They are also asking, “How much wealth am I willing to leave exposed?”

---

## The Player Session Loop

A normal player session should feel simple and repeatable.

When a player opens the game, they should be able to:

```text
1. Check kingdom status.
2. See live gold, income, energy, defense, and recent events.
3. Decide what kind of action matters most right now.
4. Spend gold and energy on buildings, training, banking, scouting, exploring, or PvP.
5. Review results and reports.
6. Leave the kingdom knowing it keeps growing while offline.
```

The important correction: Pixel Kingdoms should **not** rely on queueing buildings or actions as the main loop.

Instead, it should use an **energy/action system**. Players do not queue a long list and walk away. They receive a limited amount of action capacity over time and must decide how to spend it.

This makes the game more strategic because activity has an opportunity cost.

---

## Energy / Action Economy

Energy is the player’s active decision resource.

Initial proposed baseline:

```text
maxEnergy = 15
restoreRate = 1 energy per hour
```

Energy can be spent on important actions such as:

```text
building or upgrading
training units
scouting
spying
exploring
attacking
banking gold
defensive preparations
```

The exact costs can be balanced later, but the design intent is clear:

```text
players should not be able to do everything at once
```

Energy creates daily rhythm.

A player might ask:

- Do I spend energy upgrading my economy?
- Do I train defenders before logging out?
- Do I scout first before attacking?
- Do I bank gold now or save energy for a raid?
- Do I explore for captives/resources?
- Do I use my remaining energy defensively?

This creates light micromanagement without requiring constant attention.

---

## Idle Growth + Active Decisions

The kingdom should keep growing while the player is away.

Gold generation uses the passive backend model:

```text
liveGold = storedGold + elapsedTime * goldPerSecond
```

This gives the idle-builder feeling:

```text
My kingdom kept working while I was gone.
```

But energy controls meaningful actions.

So the full rhythm becomes:

```text
idle gold grows over time
energy restores over time
player returns
player spends limited energy to shape the kingdom
player leaves with new risks and rewards in motion
```

This is the heart of the game: idle kingdom growth, but active strategic spending.

---

## Main Player Decisions

Each session should push the player toward a few clean choices.

### Build Wealth

Spend gold and energy on economy buildings, workers, and upgrades that increase gold generation.

Strength:

```text
faster long-term growth
```

Risk:

```text
more exposed gold makes the kingdom more attractive to attackers
```

### Build Attack Power

Spend gold and energy on attackers, weapons, race bonuses, and scouting systems.

Strength:

```text
steal from other players and climb through aggression
```

Risk:

```text
even winning attacks can cost troops and equipment
```

### Build Defense

Spend gold and energy on defenders, fortifications, armor, and protection.

Strength:

```text
reduce losses and protect exposed wealth
```

Risk:

```text
slower economy or weaker offensive growth
```

### Gather Intel

Spend energy on scouting, spying, or exploration.

Strength:

```text
better target selection and hidden opportunities
```

Risk:

```text
uses energy that could have built, trained, attacked, or banked
```

### Protect Wealth

Spend energy on banking gold or defensive preparation.

Strength:

```text
reduces rage-quit losses and gives players control over risk
```

Risk:

```text
banking/protection should have limits or opportunity cost so exposed gold still matters
```

---

## The PvP Loop

PvP should be direct and understandable.

```text
open attack page
see players near your rank
compare live gold / power / risk
choose target
spend energy to attack
backend settles attacker and target gold
combat resolves automatically
both sides may lose troops and equipment
attacker steals gold if successful
experience/ranking updates
battle report is saved
```

The fun should come from preparation and target choice, not manual tactical combat.

The attacker chooses risk. The defender feels consequences but should not be destroyed so hard that they rage quit.

---

## Combat Loss Philosophy

Combat should create cost on both sides.

Important rule:

```text
Attackers can lose troops even when they win.
Defenders can lose troops even when they successfully defend.
```

This keeps combat from being free value extraction.

Winning should feel good, but not frictionless. A successful attack may steal gold but still burn units and equipment. A successful defense may protect gold but still cost defenders.

The goal is:

```text
painful enough to matter
not so punishing that players quit
```

This requires soft-loss design:

- losses scale with fight size and power gap
- protected/new players should not be farmed into zero
- repeated attacks may need limits
- defenders should retain enough kingdom identity to recover
- battle reports should clearly explain losses

Combat should create stories, not wipeouts.

---

## Attack Targeting Loop

Attack targets are based on rank proximity.

The attack list should show a small set of nearby rivals, likely around 20 max.

```text
getAttackTargets(player):
  find player rank by level + experience
  load nearby ranked players
  calculate live gold preview
  show attack-relevant stats
```

The key backend distinction remains:

```text
bulk read/calculate is acceptable
bulk write/update should be avoided
```

The target list can preview live gold for 20 players. The game only settles and writes real state when the player performs an update action, such as attacking the chosen target.

---

## The Building Loop

Buildings are long-term identity and visual progression.

```text
earn gold
restore energy
choose building or upgrade
pay gold + energy
building appears or improves in pixel kingdom
backend stats improve
new options may unlock
```

Buildings should not just be passive stat bumps. Each building should push a strategy.

Possible building roles:

- economy generation
- worker capacity
- unit training
- defense
- attack power
- equipment unlocks
- scouting/spying
- exploration
- storage/banking
- race-specific advantages

Buildings may be level-locked.

Example:

```text
Barracks level 1 unlocks basic attackers.
Player level 5 unlocks upgraded Barracks.
Armoury level 2 unlocks iron weapons.
Fortification upgrades require kingdom level thresholds.
```

Level locking gives progression structure and prevents players from rushing every powerful system immediately.

---

## The Army Loop

Minions and equipment are investments.

```text
train minions
unlock better equipment
upgrade equipment tiers
increase effective power
risk losing minions + equipment in combat
replace losses
improve again
```

The player should feel that an army is not just a number.

If 100 swordsmen carry 100 rusty swords, and 20 swordsmen die, those 20 swords and soldiers are gone. This makes every attack a real economic choice.

Training units likely should cost both gold and energy. This keeps army rebuilding from being instant and makes losses matter without requiring artificial waiting queues.

---

## Exploration / Scout / Spy Loop

Exploration and intel actions give players alternatives to direct attack.

Possible action types:

```text
Scout: low-risk information about nearby targets.
Spy: deeper information, possible failure or detection.
Explore: search for captives, resources, events, or race-specific bonuses.
```

These actions spend energy, so they compete with building, banking, training, and attacking.

This makes intel strategic:

```text
Do I attack now with imperfect information?
Or spend energy first to find a better opportunity?
```

---

## Banking / Protection Loop

Because exposed gold is central to PvP, the game needs a way to protect some value without removing risk completely.

Banking should likely spend energy.

This makes banking a real choice:

```text
I can protect some gold, but that energy could have been used to attack, scout, train, or build.
```

Banking should not make all wealth safe. It should reduce rage-quit risk while preserving the PvP economy.

Possible rules to explore in the economy chapter:

- bank only a percentage of gold
- banking costs energy
- banking has daily limits
- banked gold cannot be stolen but may have withdrawal limits
- some buildings improve storage/protection

---

## Progression Loop

Progression should happen across multiple layers:

```text
gold
energy efficiency
buildings
level
experience
race identity
minions
equipment
attack power
defense power
exploration power
spy power
rank
battle history
visual kingdom state
```

Level and experience matter, but they should not be the whole game. Their main roles are:

- unlock buildings or upgrades
- shape attack range
- give long-term progression
- help matchmaking/ranking

The deeper progression is kingdom identity.

A player should eventually be known as something:

```text
rich but vulnerable
hard to raid
brutal attacker
exploration goblin
spy-focused nuisance
balanced kingdom
late-game fortress
```

---

## Risk Loop

The central tension:

```text
gold helps me grow
gold makes me a target
energy lets me act
energy is limited
attacking can profit
attacking can cost troops and gear
defending protects wealth
defending slows aggression
banking protects value
banking consumes action capacity
```

A good session should involve at least one meaningful tradeoff.

If every action is obvious, the loop is too shallow.

---

## Short, Medium, and Long Sessions

Pixel Kingdoms should support different energy levels from players.

### Quick Session

```text
check gold
spend a few energy
upgrade or train
maybe bank gold
log out
```

### Medium Session

```text
review attack targets
scout or spy
attack once or twice
read reports
replace losses
make one building decision
```

### Deep Session

```text
study ranking bracket
optimize energy use
compare gold targets
rebalance army
plan race/building synergies
attack strategically
prepare defenses before logging out
```

The game should not punish casual players for not living inside it, but it should reward strategic players for thinking well.

---

## Full-Scope V1 Philosophy

The backend should be designed for the full intended scope from v1, even if systems are implemented chapter by chapter.

This does **not** mean rushing everything into a messy first build.

It means the first backend architecture should already understand the major systems:

- energy
- gold settlement
- buildings
- races
- minions
- equipment
- ranking
- attack targeting
- combat losses
- banking
- scouting/spying/exploration
- battle reports
- progression unlocks

The project is not in a rush. It can be built slowly and safely, but each piece should fit the final shape.

The right build approach:

```text
design full loop
build one stable slice
verify it
extend the next slice
keep the whitepaper updated
```

---

## V1 Gameplay Requirements

Because Pixel Kingdoms should aim for full-scope v1, the first complete playable version should eventually include:

```text
player registration
race selection
passive gold generation
energy restoration
building upgrades
level-locked progression
worker/economy loop
unit training
basic equipment upgrades
attack target list
PvP attack resolution
troop/equipment losses for both sides
stealable exposed gold
banking/protection action
battle reports
experience and ranking
```

Some systems can begin simple, but their hooks should exist early so the backend does not need a painful rewrite later.

---

## Chapter 1 Lock

The overall gameplay loop is:

```text
idle kingdom growth + limited energy decisions + exposed gold PvP
```

That combination is the engine.

Idle growth makes players return. Energy makes choices matter. Exposed gold makes PvP meaningful.

If future systems do not support one of those three pillars, they should wait.
