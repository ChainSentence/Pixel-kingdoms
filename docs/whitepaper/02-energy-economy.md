# Pixel Kingdoms Whitepaper

## Chapter 2 — Energy Economy

**Status:** working design chapter  
**Purpose:** define the action economy that controls how players interact with the game, how often they can make meaningful decisions, and how energy creates strategic pressure without relying on action/building queues.

---

## Chapter Thesis

**Energy is the active decision currency of Pixel Kingdoms. Gold grows while the kingdom idles, but energy determines what the player can actually do with that growth.**

Gold creates resources. Energy creates choices.

Without energy, the game risks becoming a simple gold-spending ladder. With energy, every important action competes against every other important action.

```text
Do I build?
Do I train?
Do I attack?
Do I scout?
Do I bank?
Do I defend?
```

That pressure is what makes the loop strategic.

---

## Core Energy Rules

Initial baseline:

```text
maxEnergy = 15
energyRestoreRate = 1 energy per hour
```

Energy restores passively over time until it reaches the maximum.

A player cannot store infinite action power. If they are at max energy, additional restored energy is wasted until they spend some.

This creates a natural daily rhythm:

```text
log in
spend energy
let energy recover
return later
make new decisions
```

The goal is not to force constant play. The goal is to make each return meaningful.

---

## Why Energy Instead of Queues

Pixel Kingdoms should not use long action/building queues as the core loop.

Queues often turn the game into:

```text
click upgrades -> wait -> click upgrades -> wait
```

Energy creates a better pattern:

```text
limited action capacity -> choose priority -> accept tradeoff
```

A player with 7 energy has to decide what those 7 actions are worth. They might build, attack, bank, train, scout, or defend — but not all at once.

This makes every session a small strategy puzzle.

---

## Energy as Opportunity Cost

Energy should be attached to meaningful actions.

If an action changes player power, safety, income, information, or PvP position, it probably costs energy.

Energy turns every major decision into an opportunity cost:

```text
Energy spent banking cannot be spent attacking.
Energy spent attacking cannot be spent building.
Energy spent building cannot be spent scouting.
Energy spent defending cannot be spent training.
```

This helps prevent dominant strategies because even strong actions compete for the same limited resource.

---

## Actions That Cost Energy

The exact numbers belong in balance configuration, but the design categories are clear.

### Build / Upgrade

Buildings and upgrades should cost gold and energy.

```text
upgradeBuilding(buildingId):
  cost gold
  cost energy
  apply building level increase
  update kingdom visuals/stats
```

Building with energy means players cannot instantly convert all accumulated gold into a full kingdom. They must choose which improvements matter most.

---

### Train Units

Training units should likely cost gold and energy.

```text
trainUnits(unitType, amount):
  cost gold
  cost energy
  add or reinforce minions
```

This keeps army rebuilding from being instant after losses and makes combat consequences matter.

Training can still be simple. It does not need a queue at first. The energy cost itself is the pacing system.

---

### Attack

Attacking should cost energy.

```text
attack(target):
  cost energy
  settle attacker gold
  settle target gold
  resolve combat
  apply losses
  steal gold if successful
  save battle report
```

Energy limits attack spam and gives raiding a real cost.

A player has to decide:

```text
Is this target worth one of my limited actions?
```

---

### Scout / Spy

Scouting and spying should cost energy because information is power.

```text
scout(target):
  cost energy
  return rough target information

spy(target):
  cost energy
  roll success/failure
  return deeper intel or trigger consequences
```

This creates a useful decision:

```text
attack now with imperfect information
or spend energy first to improve target choice
```

---

### Explore

Exploration should cost energy.

```text
explore(areaOrMode):
  cost energy
  resolve event/reward/risk
```

Exploration can support race identity, captives, resources, special events, and alternative growth paths.

It should compete with PvP and building so exploration is a real strategic route, not free side content.

---

### Bank Gold

Banking should cost energy.

```text
bankGold(amount):
  cost energy
  move some exposed gold into protected storage
```

This is important because protected gold reduces PvP risk. If banking is free, players will hide value too easily. If banking costs energy, it becomes a meaningful defensive choice.

Banking should protect against rage-quit losses, but it should not remove exposed gold from the game entirely.

---

### Defensive Stance

Defensive stance is a temporary protection action funded by current available energy.

It is **not permanent** and should not be a free toggle.

Core rule:

```text
1 energy committed = 1 hour defensive stance
```

Example:

```text
player has 7 energy
player commits all 7 energy to defensive stance
energy becomes 0
defensive stance lasts 7 hours
```

While active, defensive stance can provide:

- small defense bonus
- increased defense against spies
- increased defense against scouts

Exact bonuses belong in balance configuration.

Defensive stance creates a clean logout decision:

```text
Do I spend my remaining energy protecting my kingdom while I am away?
Or do I use it now for growth, training, scouting, banking, or attacks?
```

---

## Defensive Stance Anti-Abuse

Players should not be rewarded for sitting permanently in defensive mode.

Short defensive stance should be smart protection. Long defensive stance should become turtle mode with downside.

Possible anti-abuse rule:

```text
short stance = full defensive value
long stance = reduced value or penalty
```

The exact thresholds belong in balance files, but the design direction is:

```text
If a player tries to stay buffed defensively for too long, penalties begin.
```

Possible penalties:

- reduced gold generation during long stance
- reduced defense bonus after a threshold
- reduced energy restoration while exhausted
- cannot attack while stance is active
- attacking cancels stance
- long stance increases action cost afterward

A simple conceptual model:

```text
0-8 hours: normal defensive stance
9-15 hours: reduced stance efficiency or minor penalty
16+ hours: stronger exhaustion penalty, if max energy ever expands beyond 15
```

Since the starting max energy is 15, a 24-hour defensive stance should not be possible unless later systems increase max energy. If future upgrades raise max energy, penalties become more important.

---

## Energy Restoration and Settlement

Energy can use the same lazy-settlement philosophy as gold.

Stored fields may look like:

```text
storedEnergy
lastEnergySettledAt
maxEnergy
energyRestoreRate
```

Queries can preview current energy:

```text
liveEnergy = min(maxEnergy, storedEnergy + elapsedHours * restoreRate)
```

Updates settle energy before spending it:

```text
settleEnergy(player)
validate player has enough energy
spend energy
save new energy state
```

This keeps energy feeling real-time without requiring scheduled backend jobs.

---

## Energy and Gold Together

Gold and energy are the two main pacing resources.

```text
gold = economic resource
energy = action resource
```

Gold answers:

```text
Can I afford this?
```

Energy answers:

```text
Is this worth one of my limited actions?
```

Most important actions should require both.

Examples:

```text
Building upgrade = gold + energy
Training units = gold + energy
Attack = energy + army risk
Banking = energy + maybe fee/limit
Defensive stance = energy over time
Scouting/spying = energy + success/failure risk
```

This prevents a player with lots of gold from instantly doing everything.

---

## Energy and Player Types

Energy should support different playstyles.

### Casual Player

Logs in once or twice per day, spends accumulated energy on major actions, logs out.

Needs:

- clear best available actions
- simple defensive options
- no heavy punishment for not checking constantly

### Active Player

Logs in often enough to avoid wasting capped energy.

Needs:

- meaningful target choices
- tactical scouting/attacking decisions
- satisfying energy optimization

### Strategic Player

Plans energy around timing, defense, banking, attacks, and rank movement.

Needs:

- enough depth that energy sequencing matters
- visible tradeoffs
- battle/intel reports worth studying

Energy should reward attention and planning without turning the game into a chore.

---

## Balance Boundaries

This chapter defines concepts, not final numbers.

The following belong in balance files or balance chapters:

```text
exact energy cost per action
defensive stance bonus percentages
anti-spy bonus
anti-scout bonus
banking limits
training cost by unit type
building cost by level
attack energy cost
exploration cost and rewards
penalties for long defensive stance
max energy upgrades, if any
energy restoration modifiers
```

The whitepaper should define what the system is for. Balance files should define exact values.

---

## Backend Requirements

The backend should eventually support:

```text
getEnergyPreview(player)
settleEnergy(player)
spendEnergy(player, amount)
activateDefensiveStance(hours)
getDefensiveStanceStatus(player)
applyDefensiveStanceModifiers(player)
expireOrSettleDefensiveStance(player)
```

Energy should be checked before any energy-costing update method.

Common update flow:

```text
settleGold(player)
settleEnergy(player)
validate action requirements
spend energy
apply action
save state
return result
```

For target-based actions such as attack, scout, or spy:

```text
settle actor energy
spend actor energy
settle relevant target state if needed
resolve action
save both states if mutated
```

---

## Design Rules

Energy economy should follow these rules:

1. **Energy is active decision power.**
2. **No permanent defensive stance.**
3. **No free protection.** Banking and defensive posture cost energy.
4. **No action queues as the core loop.** Energy is the pacing mechanic.
5. **Most powerful actions should cost energy.**
6. **Energy should restore slowly enough to create choice, not frustration.**
7. **Exact numbers belong in balance files.**
8. **Energy should support casual and active players.**
9. **Long defensive turtling needs penalties.**
10. **Energy should make players ask what matters most right now.**

---

## Chapter 2 Lock

The Energy Economy is the action-pressure system of Pixel Kingdoms.

```text
Gold grows the kingdom.
Energy decides what the player can do.
Exposed gold creates PvP pressure.
Defensive energy choices control risk.
```

The best version of this system makes every login feel like a small but meaningful kingdom decision.
