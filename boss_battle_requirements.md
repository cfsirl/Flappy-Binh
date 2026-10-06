# Flappy Binh - Boss Battle Stage Requirements

## Overview
This document outlines the requirements for a boss battle stage in Flappy Binh that's inspired by Mario's Bowser battles but with unique elements. The player controls Ethan (Binh) to fly over a boss character and destroy a bridge by touching a hatchet.

## Boss Battle Mechanics

### Core Gameplay
- **Objective**: Fly Binh over the boss and touch the hatchet to destroy the bridge
- **Safe Zone**: Binh can sit on the bridge without losing
- **Win Condition**: Successfully destroy the bridge by touching the hatchet
- **Lose Conditions**: 
  - Touch any laser beam from the boss
  - Touch the boss character itself
  - Fall off the level

### Boss Battle Progression
- **Frequency**: Boss battle occurs every 100 points (fixed interval, no frequency scaling).
- **Laser Speed Scaling**: Each subsequent boss battle features lasers that move 10% faster (relative to the base speed of 2 px/frame, capped at 3×).

### Five-Laser Special Attack
- The mole periodically fires a fan pattern of **five laser beams** in rapid succession (one after another, not simultaneously).
- Fan geometry (relative to horizontal, toward the player): `[-40°, -20°, 0°, +20°, +40°]`, fired 0.08 s apart (see Laser Mechanics for timing and warning rules).
- This fan pattern is distinct from normal single-laser shots which fire straight across the bridge.

### Player Movement During Boss Battle
- **Restricted movement**: Unlike normal gameplay (tap/click only to flap), the player has **left/right directional keyboard controls** in addition to flapping during the boss battle.
- **Controls**:
  - Keyboard: `A` / `←` = move left, `D` / `→` = move right, **1 logical px/frame**, clamped to `[0, canvasWidth]`; `Space` / `Enter` = flap (same physics as normal play).
  - Touch: hold the **left third** of the screen to steer left, **right third** to steer right, **center third** to flap.
- **On the bridge (safe zone)**: when the player's bottom edge ≥ `bridgeY` and the player is within the bridge's ±5 px tolerance band, gravity is disabled — the player may stand and walk. Moving past the ±5 px band beyond either edge → fall into the pit → loss.
- **Player hitbox**: the same 4 px inward forgiveness inset as normal play (`HITBOX_INSET = 4`) — the boss fight must not feel more brutal than the pipes.
- **Progressive Challenge**: Increasing difficulty with each boss encounter via laser speed only (cadence stays fixed at 3 s).

### Boss Character Design
- **Appearance**: A relatively slender girl with a huge mole above her left lip
- **Mole Features**:
  - Large mole positioned above the left side of her mouth
  - The mole shoots slow-moving laser beams
- **Movement Patterns**:
  - Regular idle behavior
  - Occasional jumping in an arc (like Bowser)
  - Laser shooting when in attack mode

### Environment
- **Bridge**: Solid platform spanning the **entire canvas width** (`bridgeWidth = canvasWidth = 288` logical px), **20 px thick**, top surface at `bridgeY = 0.62 * canvasHeight` (≈ 317 logical px) — the pit below is ≈ 195 px deep, a real "fall off the edge" threat (Bowser Castle III style).
- **Hatchet**: Positioned on the bridge that Binh must touch to destroy it
- **Laser Beams**: Slow-moving beams shot from the boss's mole
- **Level Boundaries**: Clear edges where falling results in game over

## Detailed Requirements

### 1. Boss Character Specifications
- **Visual Design**:
  - Slender female character with distinctive mole above left lip
  - Mole should be prominently visible and visually striking
  - Boss should have a distinct color scheme different from Binh
- **Behavioral Patterns**:
  - Idle state (normal movement)
  - Attack state (laser shooting)
  - Jumping state (arc trajectory like Bowser)

### 2. Laser Mechanics
- **Base Laser Speed**: Set the initial laser speed to `2 px/frame`. Speed per fight: `laserSpeed = min(2 * 1.1**bossCount, 6)` — +10% per successive battle, hard cap **6 px/frame** (3× base).
- **Normal Shot**: one beam fired **horizontally toward the player's side** (the mole aims straight across the bridge) at the current laser speed.
- **Five-Beam Special Attack**:
  - Five beams fired in rapid succession, **0.08 s (5 logical frames) apart**
  - Fan angles from the mole: `[-40°, -20°, 0°, +20°, +40°]` relative to the horizontal axis (0° = straight across toward the player)
  - Fires every **12 seconds (720 logical frames)** of active battle
- **Special Attack Warning**: A small on-screen flashing numeric countdown starts **3 s** before the special attack; turns red for the final 0.5 s; hidden after the barrage. The warning window is what makes a 5-beam spread fair — never fire the special without it.
- **Pre-Shot Cue (all shots)**: The mole outline flashes red for 0.5 s immediately before any laser is fired.

### 3. Boss Jumping Mechanics
- **Jump Animation**:
  - Parabolic arc trajectory (Bowser-style)
  - Apex: 1/3 of vertical screen height above the bridge
  - Total air time: **0.8 s (48 logical frames)**; `easeOutQuad` up, `easeInQuad` down
  - Landing: 4-frame dust puff + slight bridge shake
  - The boss hitbox follows the arc, so the player dodges by timing
  - Minimum jump frequency: Every 3 seconds (enforced by the tick below)
- **Timing**:
  - Boss takes actions (jump or shoot) at fixed **3-second intervals** — not per-frame random. At the top of each 3-second tick, randomly select between jumping and shooting (50/50).
  - The boss will never jump and shoot simultaneously; it is one action per tick.

### 4. Hatchet Mechanics
- **Hatchet Placement**:
  - 32×32 px, positioned on the bridge at `x = 0.75 * canvasWidth`, `y = bridgeY - 16` (just above the deck, on the far side of the boss)
  - Must be reachable by Binh when flying over the boss
  - The boss occupies the mid-bridge zone, so the hatchet is the only win path — it cannot be reached by walking straight across
- **Interaction**:
  - Touching hatchet destroys bridge
  - Bridge destruction = win condition
  - On touch: bridge collapses over **0.5 s** (`easeInBack`: deck sinks and cracks), plays optional `bridge_break.wav` (0.4 s), boss drops into the pit → Victory → normal play resumes at `score + 100`, `pipeSpeed` recalculated from the formula, `bossCount++`

### 5. Safe Zone Mechanics
- **Bridge Safety**:
  - Binh can land and stay on bridge without losing
  - Movement restrictions while on bridge (no falling)
- **Transition Rules**:
  - Must fly over boss to reach hatchet
  - Cannot simply walk across the bridge

## Game Flow

### Stage Setup
1. Player enters boss battle stage at the first score that is a multiple of 100, then every 100 points thereafter (fixed interval).
2. **Transition** (fixed ≈ 1.5 s timeline — the Bowser-castle entrance beat):
   - t = 0: `state = 'bossIntro'`; all active pipes are removed from the array; player physics is frozen for 0.5 s.
   - t = 0–1 s (60 logical frames): background continues scrolling at the current speed while the full-width bridge slides in from the right and the boss fades in at a random end (left or right).
   - t = 1 s: `state = 'bossActive'`; the boss 3-second tick clock and laser logic start; the 12 s special-attack countdown begins.
3. Boss character appears at one end of the bridge (left or right side, randomized).
4. Hatchet positioned on the bridge at a reachable location beyond the boss.
5. Bridge spans the **entire canvas width** with the pit below — Bowser Castle III style.

### Player Actions
1. **Fly Over**: Navigate over the boss to reach hatchet
2. **Safe Landing**: Land on bridge to avoid damage
3. **Hatchet Touch**: Activate bridge destruction mechanism

### Win/Lose Conditions
- **Win**: Successfully touch hatchet, destroy bridge, boss falls
- **Lose** (on any lose condition):
  - Touch laser beam
  - Touch boss character body or mole hitbox
  - Fall off the bridge into the pit below
  - **On loss** (intentional design choice — harsh by design): play the 1.6 s toilet death sequence (see `requirements.md`), then restart the entire game from the start screen: `score = 0`, `pipeSpeed = baseSpeed`, `laserSpeed = 2`, `bossCount = 0`, `theme = day`, all boss objects and timers cleared. **The `localStorage` high score is retained** — it is only overwritten by a new record.

### Multiple Boss Battles in a Row
- Boss battles can occur multiple times (every 100 points is fixed). Each consecutive boss battle increments the laser speed multiplier by +10% cumulatively (capped at 3× base speed).

- **Bridge Dimensions**: The bridge spans the **entire width** of the canvas (`bridgeWidth = canvasWidth`). A safety margin of ±5 px is allowed for player landing on each side. Picture a Bowser Castle III-style platform from Super Mario Bros. — a solid bridge stretching edge to edge with a drop into the pit below.
- **Laser Speed Cap**: Laser speed is limited to a maximum of 3× the base laser speed to prevent unplayable difficulty.
- **Laser Warning UI**: The mole outline flashes red for 0.5 s immediately before a laser is fired, giving the player a visual cue.
- **Hatchet Interaction Feedback**: When the hatchet is touched, the bridge collapses with a 0.5 s animation and plays an optional `bridge_break.wav` sound cue.
- **Hitbox Definitions** (anchors: rectangles are top-left `(x, y)`; world is 288×512 logical px):
  - **Boss Body**: AABB `60×80`, top-left at `(bossX, bossY)`; follows the jump arc.
  - **Mole**: circle, radius **10 px**, centered at `(bossX + 20, bossY + 45)` (left of the mouth, per the character design).
  - **Laser**: line segment from the mole center to the beam tip, **5 px** collision radius.
  - **Hatchet**: AABB `32×32`, top-left at `(0.75 * canvasWidth - 16, bridgeY - 32)`.
  - **Player**: same 4 px inward hitbox inset as normal play (`HITBOX_INSET = 4`).


### Boss Character Elements
- **Main Character**: Slender female with distinctive mole feature
- **Mole Details**: Large mole positioned above left lip (prominent visual)
- **Laser Effects**: 
  - Slow-moving beam projectiles
  - Clear visual trail/path of laser
  - Impact effects when hitting objects

### Environment Elements
- **Bridge**: Solid platform spanning the **entire canvas width** (Bowser Castle III style)
- **Hatchet**: Visual indicator that can be touched to activate
- **Background**: Distinctive theme different from regular gameplay

## Technical Implementation Details

### Physics and Movement
- **Boss Jumping**:
  - Implement arc trajectory physics with height limitation (1/3 screen height)
  - Boss takes actions at fixed **3-second intervals** — NOT per-frame random. At each 3-second tick, randomly select between jumping and shooting (50/50).
  - The boss will never jump and shoot simultaneously; one action per tick.
  - Landing with visual impact
- **Boss Idle Movement**: Boss drifts slowly back and forth along the bridge during idle periods for visual variety.

### Collision Detection
- **Laser Beams**: 
  - Line-based collision detection
  - Radius-based damage area
- **Boss Character**: 
  - Hitbox for body and mole area
  - Different hitboxes for laser vs. body contact

### Game States
1. **Boss Battle Intro**: Boss appears, hatchet visible
2. **Active Battle**: Player can fly over boss
3. **Bridge Destruction**: Hatchet touched, bridge collapses
4. **Boss Falls**: Boss falls due to broken bridge
5. **Victory Sequence**: Win screen or next stage

## Difficulty Progression

### Progressive Challenges
- **Early Boss Battles**: 
  - Slow lasers with wide margins
  - Infrequent jumps
- **Later Boss Battles** (laser speed is the primary scaling axis; the 3-second action cadence stays fixed to remain readable):
  - Lasers 10% faster each time (capped at 3× base speed)
  - From boss #2 onward, the jump/shoot tick split shifts from 50/50 toward shoot (recommended: 60/40)
  - The boss's idle drift range widens, occupying more of the mid-bridge zone and narrowing the safe passage

### Scaling System
- **Laser Speed Multiplier**: Increases by 10% for each boss battle
- **Action Frequency**: Boss takes actions every 3 seconds
- **Visual Feedback**: Clear indication of increasing difficulty

## User Interface Elements

### Boss Battle UI

- **Laser Warning**: Visual cues when boss is about to shoot
- **Hatchet Indicator**: Highlighting the hatchet for visibility
- **Timer/Score**: Optional timer or score for completion time
- **Difficulty Indicator**: Visual representation of current laser speed

## Integration Notes

### Stage Design
- Boss battle stage should be visually distinct from regular gameplay
- Clear transition from normal game to boss battle
- Different background theme for boss battle