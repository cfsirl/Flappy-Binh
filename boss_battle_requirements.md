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
- **Frequency**: Boss battle occurs every 100 points
- **Difficulty Scaling**: Each subsequent boss battle features lasers that move 10% faster
- **Progressive Challenge**: Increasing difficulty with each boss encounter

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
- **Bridge**: Main platform that spans across the level
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
- **Base Laser Speed**: Set the initial laser speed to `2 px/frame`. This value will be multiplied by the laser speed multiplier (max 3×) for subsequent boss battles.
- **Special Attack Countdown**: Display a small on‑screen timer (e.g., a flashing numeric countdown) that starts 3 seconds before the mole fires the five‑laser special attack. The timer resets after each special attack.

### 3. Boss Jumping Mechanics
- **Jump Animation**:
  - Arc trajectory like Bowser's jumping
  - Maximum jump height: 1/3 of vertical screen height
  - Minimum jump frequency: Every 3 seconds
- **Timing**: 
  - Boss takes actions (jump or shoot) every 3 seconds
  - Random selection between jumping and shooting

### 4. Hatchet Mechanics
- **Hatchet Placement**:
  - Positioned on the bridge at a strategic location
  - Must be reachable by Binh when flying over the boss
- **Interaction**:
  - Touching hatchet destroys bridge
  - Bridge destruction = win condition

### 5. Safe Zone Mechanics
- **Bridge Safety**:
  - Binh can land and stay on bridge without losing
  - Movement restrictions while on bridge (no falling)
- **Transition Rules**:
  - Must fly over boss to reach hatchet
  - Cannot simply walk across the bridge

## Game Flow

### Stage Setup
1. Player enters boss battle stage at 100 points, then every 100 points thereafter
2. Boss character appears at one end of the bridge
3. Hatchet positioned on bridge
4. Bridge spans between boss and edge of level

### Player Actions
1. **Fly Over**: Navigate over the boss to reach hatchet
2. **Safe Landing**: Land on bridge to avoid damage
3. **Hatchet Touch**: Activate bridge destruction mechanism

### Win/Lose Conditions
- **Win**: Successfully touch hatchet, destroy bridge, boss falls
- **Lose**: 
  - Touch laser beam
  - Touch boss character
  - Fall off level edges

- **Bridge Dimensions**: The bridge spans 60 % of the canvas width (`bridgeWidth = 0.6 * canvasWidth`). A safety margin of ±5 px is allowed for player landing.
- **Laser Speed Cap**: Laser speed is limited to a maximum of 3× the base laser speed to prevent unplayable difficulty.
- **Laser Warning UI**: The mole outline flashes red for 0.5 s immediately before a laser is fired, giving the player a visual cue.
- **Hatchet Interaction Feedback**: When the hatchet is touched, the bridge collapses with a 0.5 s animation and plays an optional `bridge_break.wav` sound cue.
- **Hitbox Definitions**:
  - **Boss Body**: Rectangular hitbox (`bossWidth` × `bossHeight`).
  - **Mole**: Circular hitbox radius = 10 px centered on the mole graphic.
  - **Laser**: Line segment with a 5 px radius for collision detection.
  - **Hatchet**: Rectangular hitbox (`hatchetWidth` × `hatchetHeight`).


### Boss Character Elements
- **Main Character**: Slender female with distinctive mole feature
- **Mole Details**: Large mole positioned above left lip (prominent visual)
- **Laser Effects**: 
  - Slow-moving beam projectiles
  - Clear visual trail/path of laser
  - Impact effects when hitting objects

### Environment Elements
- **Bridge**: Platform that spans the level width
- **Hatchet**: Visual indicator that can be touched to activate
- **Background**: Distinctive theme different from regular gameplay

## Technical Implementation Details

### Physics and Movement
- **Boss Jumping**:
  - Implement arc trajectory physics with height limitation (1/3 screen height)
  - Random jump intervals (minimum 3 seconds between actions)
  - Landing with visual impact

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
- **Later Boss Battles**:
  - 10% faster lasers each time
  - More frequent jumping and shooting
  - Narrower safe passage

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