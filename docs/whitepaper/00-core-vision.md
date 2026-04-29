# Pixel Kingdoms Whitepaper

## Chapter 0 — Core Vision

**Status:** living design foundation  
**Purpose:** define the soul of Pixel Kingdoms before backend implementation begins. This chapter should be treated as the source-of-truth for the game’s high-level direction. Future backend chapters should extend this vision without losing the core loop.

---

## One-Sentence Vision

**Pixel Kingdoms is a slow-burn browser PvP kingdom builder that combines the addictive economy, army, ranking, and raiding loop of DarkThrone with the visual satisfaction of a Heroes of Might and Magic III-style kingdom-building screen.**

Players build a living pixel kingdom over time, choose a race with real strengths and weaknesses, grow wealth, train minions, upgrade equipment, scout rivals, attack nearby ranked players, steal exposed gold, and climb through persistent progression.

---

## Game Fantasy

The player fantasy is simple and strong:

```text
I choose a race.
I build a kingdom.
I watch buildings appear in my pixel city.
I grow my economy.
I train minions.
I upgrade weapons and armor.
I scout and spy on rivals.
I attack players near my rank.
I steal gold from exposed kingdoms.
I defend my own wealth.
I climb through level, experience, and power.
```

Pixel Kingdoms should feel like a kingdom that is always alive, even when the player is offline. Gold accumulates, troops stand guard, rivals become tempting targets, and every upgrade gives the player a clearer identity.

---

## Inspiration Pillars

### DarkThrone Foundation

The strategic foundation comes from classic browser war games like DarkThrone:

- persistent player kingdoms
- race selection
- passive economy growth
- role-based citizens or minions
- offensive and defensive armies
- spy/scout systems
- equipment-driven power scaling
- PvP attacks for exposed resources
- public ranking pressure
- simple pages with deep long-term progression

Pixel Kingdoms should preserve the part that made those games sticky: the backend strategy loop. The player is always choosing where to invest: wealth, offense, defense, scouting, or growth.

### HoMM3-Style Kingdom Building

The visual inspiration comes from Heroes of Might and Magic III’s building phase, not its tactical combat.

The player owns a flat pixel-art kingdom scene. As buildings are constructed and upgraded, they visibly appear or improve in the kingdom view. This makes progression feel tangible.

The visual layer is not just decoration; it reinforces identity and progress. A wealthy kingdom, a military kingdom, and a defensive kingdom should gradually look different.

### Backend-First Design

Combat is **automatic and text-based**, not turn-based tactical combat.

The backend resolves:

- gold generation
- troop training
- equipment allocation
- exploration outcomes
- spy reports
- attacks
- losses
- stolen gold
- experience gain
- ranking movement

The frontend shows the result clearly and beautifully, but the game must be designed so the backend can be built cleanly first.

---

## Core Gameplay Loop

The main loop should be easy to understand:

```text
build economy -> train minions -> upgrade equipment -> scout/explore -> attack/defend -> gain rewards -> improve kingdom
```

A more detailed loop:

1. **Choose a race** with a perk and weakness.
2. **Build the kingdom** through economy, military, defense, and utility buildings.
3. **Generate gold over time** through workers, buildings, and race bonuses.
4. **Train minions** into different roles.
5. **Upgrade equipment** through buildings like the Armoury.
6. **Scout, spy, or explore** to reveal opportunities and gather advantages.
7. **Attack nearby ranked players** to steal exposed gold.
8. **Lose or preserve units and gear** based on combat results.
9. **Gain experience and climb ranks** through successful play.
10. **Specialize over time** into wealth, attack power, defensive power, or intel/exploration.

---

## Player Identity

Every player controls one kingdom.

A kingdom is defined by:

- player account / principal
- race
- level
- experience
- rank
- gold on hand
- passive gold generation
- buildings
- minions
- equipment
- attack power
- defense power
- exploration strength
- spy strength
- battle history
- cooldowns or protection state

The player should always understand what kind of kingdom they are becoming:

- rich economy builder
- aggressive raider
- defensive fortress
- scout/explorer specialist
- race-synergy optimizer
- balanced kingdom

---

## Races

Races are not cosmetic skins. Each race should create a different backend strategy.

Every race should have:

```text
name
theme
perk
weakness
preferred playstyle
unique minion flavor
possible building or equipment synergy
```

Race design should follow a simple rule:

**Every strength must create a temptation, and every weakness must create a risk.**

Example direction:

```text
Goblins
Perk: 5% fewer losses while exploring; captives cannot run away.
Weakness: -5% defensive power; captives suffer heavier losses when the kingdom loses.
Playstyle: aggressive scavenger / captive economy / risky growth.
```

This is only an example, not final balance. The important rule is that race bonuses should change how players behave.

---

## Minions

Minions are the backbone of the army and economy. They are race-specific in flavor, but should share a clean backend structure.

Core minion categories:

```text
Attackers
Defenders
Spy type
Explorer type
Worker type
```

Higher-tier minions have higher base stats and can use better equipment.

Minions should support simple strategic questions:

- Do I train workers for gold?
- Do I train attackers to raid?
- Do I train defenders to protect my exposed gold?
- Do I invest in spies to choose better targets?
- Do I explore for captives, resources, or special outcomes?

---

## Equipment

Equipment should be simple to manage but meaningful in combat.

The core idea:

```text
minion type + equipment tier = effective power
```

Example:

```text
100 Swordsmen with Rusty Swords
Armoury upgrades swords
Some Swordsmen now use Iron Swords
Attack power increases automatically
```

Equipment should not require annoying manual micromanagement at the start. The backend can assign the best available equipment automatically based on minion tier and unlocked upgrades.

Important death rule:

**When a minion dies, the equipment carried by that minion is also lost.**

This makes combat costly. Attacking is exciting because it can win gold, but losing troops also burns invested resources.

---

## Buildings

Buildings are both visual progress and backend power.

A building should usually provide one or more of:

- gold generation
- population or worker capacity
- attack scaling
- defense scaling
- equipment unlocks
- minion unlocks
- spy or explorer bonuses
- storage/protection mechanics
- race-specific bonuses

The kingdom screen should visually reflect the backend state. When a player builds or upgrades something, the pixel kingdom should change.

This gives the game a strong emotional reward:

```text
I did not just increase a number.
My kingdom visibly grew.
```

---

## Gold and Wealth

Gold is the primary resource and the main PvP incentive.

The game revolves around **gold on hand**: the stealable wealth that makes kingdoms tempting targets.

Gold should be generated passively through workers, buildings, and bonuses, but it should not require constant writes just to remain accurate.

Core backend rule:

```text
queries preview live gold
updates settle real gold
```

A player may store:

```text
storedGold
goldPerSecond
lastSettledAt
```

Queries can calculate:

```text
liveGold = storedGold + elapsedTime * goldPerSecond
```

Update actions settle the value into real state.

This makes the game feel real-time without forcing expensive or risky bulk state writes.

---

## Ranking and PvP Targeting

Ranking determines who a player can attack.

The attack screen should show players near the attacker’s rank, with a maximum page size around 20 targets.

For those targets, the game can show live preview gold because PvP depends on choosing who has the most exposed wealth.

Important distinction:

```text
bulk read/calculate = acceptable
bulk write/update = avoid
```

Target list flow:

```text
getAttackTargets(player):
  find player rank by level + experience
  load nearby ranked players, max 20
  calculate live gold preview for each target
  return target list
```

Attack flow:

```text
attack(attacker, target):
  validate target is attackable
  settleGold(attacker)
  settleGold(target)
  resolve automatic combat
  steal from target's settled gold
  apply losses
  award experience
  save battle result
```

The target list is a preview. The attack update is the moment of truth.

---

## Combat

Combat is automatic and text-based.

Pixel Kingdoms should not copy HoMM3 tactical battlefield combat. The player prepares the kingdom, army, equipment, and target choice; the backend resolves the fight.

Combat should consider:

- attacker minions
- defender minions
- equipment tiers
- race modifiers
- building bonuses
- defensive power
- attack power
- possible spy/scout advantages
- random variance, if desired
- losses on both sides
- stolen gold
- experience rewards

The result should be shown as a clear battle report:

```text
You attacked GoblinKing.
You won.
You stole 12,450 gold.
You lost 8 Swordsmen and 8 Iron Swords.
The defender lost 11 Guards and 11 Rusty Shields.
You gained 34 experience.
```

---

## Strategic Paths

The game should support three main strategic paths from the start:

### Wealth Path

Focus on gold generation, workers, economy buildings, and fast upgrades.

Strength: grows quickly.  
Risk: becomes an attractive target.

### Attack Path

Focus on attackers, weapons, spy intel, and frequent raids.

Strength: steals value from others.  
Risk: loses units/equipment and may have weaker defense.

### Defense Path

Focus on guards, walls, fortifications, armor, and protection.

Strength: keeps wealth safer.  
Risk: may grow slower or attack less efficiently.

A good kingdom game lets players mix these paths, but each player should feel the tradeoff.

---

## Design Principles

Pixel Kingdoms should follow these principles:

1. **Backend truth first.** The backend must be clean enough to build safely.
2. **Visual progress matters.** Buildings should appear in the pixel kingdom as players progress.
3. **PvP needs exposed value.** Gold on hand creates tension and target selection.
4. **Queries should not mutate state.** Queries preview; updates settle.
5. **Death should matter.** Losing minions also loses their equipment.
6. **Races need identity.** A race must create a real playstyle.
7. **Start simple, leave room to expand.** MVP should be playable before deep balance complexity.
8. **Numbers live in balance chapters, not random code.** Formulas should be documented before implementation.
9. **Every system should create a decision.** If a system does not change player behavior, it should wait.
10. **Slow build is okay.** This project should be built chapter by chapter so good ideas are preserved instead of lost in chat.

---

## Initial Backend Implications

This vision implies the backend will eventually need:

- player registration
- race selection
- kingdom state storage
- building state storage
- minion and equipment state
- passive gold preview helper
- gold settlement helper
- ranking index
- attack target query
- attack update method
- battle report storage
- race modifier system
- building modifier system
- balance configuration

The first backend should not try to implement everything. The MVP should prove the main loop:

```text
register -> choose race -> generate gold -> build -> train -> view targets -> attack -> steal gold -> gain exp
```

Once that loop works, deeper systems can be added safely.

---

## Working Chapter Order

The Build Bible should continue in this order:

```text
0. Core Vision
1. Overall Gameplay Loop
2. Player + Kingdom State
3. Races
4. Buildings
5. Gold Economy
6. Minions + Equipment
7. Exploring / Captives / Workers
8. PvP + Combat Resolution
9. Ranking + Attack Targeting
10. Backend API + Storage Model
11. Balance Formulas
12. MVP Build Roadmap
```

Each chapter should become both a design guide and a backend implementation reference.

---

## Core Vision Lock

Pixel Kingdoms is not just a pixel art game and not just a backend numbers game.

It is a kingdom progression game where every backend decision should make the player feel one of these things:

```text
I am growing.
I am vulnerable.
I am dangerous.
I am choosing my identity.
I am building something visible.
I am risking something real.
```

That is the heart of the project.
