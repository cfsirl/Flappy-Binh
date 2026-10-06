# Flappy Binh - Game Requirements

## Project Overview
Flappy Binh is a parody of the popular Flappy Bird game with a Vietnamese cultural twist. The goal is to create an engaging, simple-to-play game that captures the essence of Flappy Bird while adding unique Vietnamese elements.

## Core Requirements

### Game Mechanics

**Logical world** — the simulation runs in a fixed **288×512 logical-pixel** world (portrait, classic Flappy Bird play area). The canvas is scaled to the viewport (devicePixelRatio-aware); the responsive breakpoints below govern the surrounding UI only, never the physics. All values in this document are logical px / logical frames (60 fps reference).

**Physics** (initial tuning baseline — confirm in playtest):

| Constant | Value | Notes |
|---|---|---|
| `gravity` | 0.45 px/frame² | applied to vertical velocity each frame |
| `flapVelocity` | −8.0 px/frame | set on each tap; overrides current velocity |
| `maxFallSpeed` | 12 px/frame | clamps downward velocity |
| `playerX` | ¼ × canvas width (72 px) | fixed horizontal position in normal play |

- **Basic Gameplay**: Player controls Ethan that continuously falls and must flap to avoid pipes
- **Controls**: Click/tap to make Ethan flap upward
- **Collision Detection**: 
  - Pipe collisions (top and bottom) — lethal
  - Ground collision — lethal
  - Ceiling collision — **not lethal**: on contact, clamp `y = 0` and `velocity = 0` (bonk), matching Flappy Bird
  - Checks are evaluated each frame after position update, using the inset player hitbox below
- **Player Hitbox (forgiveness)**: Collision box = drawn sprite **inset 4 px on all sides** (`HITBOX_INSET = 4`); the full 48 px head is still drawn. Primary feel control — the game should play as forgiving as Flappy Bird, not as its sprite outline.
- **Scoring System**: +1 point when Binh **passes a pipe** — awarded exactly once per pipe pair when the pipe's right edge crosses the player's left edge (`pipe.x + pipe.width < playerX`). A pipe that is never passed (death first) scores nothing; a pipe leaving the screen never scores by itself.
- **Difficulty Progression**: Pipes move faster as score increases
- **Theme Changes**: Every 25 points, the theme changes between daytime and dusk
- **Speed Increase**: Every 25 points, pipe speed increases by 25%: `pipeSpeed = min(baseSpeed * (1 + 0.25 * floor(score/25)), 8)`, with `baseSpeed = 2.5 px/frame` and a hard cap of **8 px/frame** (~3.2×). The cap is reached around score 100 — where the first boss battle begins.
- **High Score Persistence**: Store the highest score in `localStorage` under key `flappyBinhHighScore` and display it on the Game‑Over screen.
- **Accessibility Enhancements**: 
  - High‑contrast UI colors for color‑blind friendliness.
  - Keyboard shortcuts: Space/Enter to flap, `R` to restart.
  - Optional sound‑off toggle in the settings menu.

- **Special Ending (death sequence, ≈ 1.6 s total)**: When Ethan hits a **pipe or the ground** (ceiling is a non-lethal bonk), run this timeline before the Game Over screen:
  | t (s) | Event |
  |---|---|
  | 0.0 | Pipe scroll freezes; player velocity → `maxFallSpeed` |
  | 0.0–0.6 | Player tumbles (rotates ≈ −90°) to a toilet sprite that spawns at `(playerX, groundY)` |
  | 0.6–1.4 | 2–3 bird sprites (`2 + floor(rand * 2)`) arc in from the top, each dropping a "poop" sprite onto the player |
  | 1.4–1.6 | Screen flash / thud, then transition to the Game Over screen |
- **Boss Battle Stage**: Advanced stage with unique boss battle mechanics (inspired by Mario's Bowser battles) occurring every 100 points. Boss frequency is fixed at 100-point intervals; difficulty increases solely through faster laser beams (+10% per successive boss battle, capped at 3×). On loss during a boss battle, the **entire game restarts from scratch** — score, speed, laser multiplier, and boss state all reset — **except the persisted high score, which is retained** (see `boss_battle_requirements.md`, Win/Lose Conditions).

- **Head Graphic Constraints**: Render `binh-head.png` at a height of 48 px (maintaining aspect ratio → width ≈ 36.5 px; the source image is 276×363). The image **already has a transparent background** (verified: RGBA, 26.8% transparent pixels) — do NOT apply an additional background-removal/autocrop step, as it risks shaving the subject. Fall back to a placeholder silhouette if the image fails to load.
- **Vietnamese Cultural Elements**: Include at least one of the following in the background:
  - Traditional pattern overlay (e.g., a subtle lotus motif).
  - Optional background music snippet of traditional Vietnamese instruments.
- **Responsive Design Breakpoints**:\
  - Mobile: viewport width ≤ 480 px – UI elements stack vertically, score displayed at top center.\
  - Tablet: viewport width > 480 px and ≤ 1024 px – UI elements arranged horizontally, score on the right, with slightly larger touch targets.\
  - Desktop/Laptop: viewport width > 1024 px – UI elements arranged horizontally with additional margin, optional side panel for extra stats.\

- **Asset Loading Strategy**: Lazy‑load heavy assets (head image, boss sprites) after the initial game canvas is created to keep the initial page load < 200 KB.

- **Ethan (Binh) Character**: Cartoon character combining actual head graphic with cartoon body
- **Head Reference**: Use reference image `binh-head.png` for facial features (eyes, mouth, hair style), resized to the 48 px render height (background is already transparent — see Head Graphic Constraints)
- **Body**: Cartoon body design (blue shirt, red pants) with simple limbs
- **Pipe Design**: Green pipes with brown caps (cap 10 px wider than the body)
  - Pipe body width: 52 px
  - Pipe gap: 150 px minimum between top and bottom pipe openings (randomize the gap-center position; the gap is never narrowed to raise difficulty)
  - Pipe spawn rate: one pipe pair every **90 logical frames** (1.5 s at 60 fps), driven by a time accumulator (dt-based), not `frames % N`, so 120 Hz displays spawn identically
- **Background**: Sky gradient with clouds
- **Ground**: Brown ground with green grass
- **Vietnamese Themed Elements**:
  - Traditional pattern overlay (e.g., subtle lotus motif) in background
  - Cultural references in game aesthetics

### 3. User Interface Requirements
- **Start Screen**: 
  - Game title "Flappy Binh"
  - Brief description
  - Start button
- **Game Screen**:
  - Score display during gameplay
- **Game Over Screen**:
  - Final score display
  - Play again button
- **Responsive Design**: Works on both desktop and mobile devices

### 4. Technical Requirements
- **Platform**: Web-based (HTML5, CSS3, JavaScript)
- **Browser Compatibility**: Modern browsers (Chrome, Firefox, Safari, Edge)
- **Performance**: Smooth gameplay at 60fps
- **Game Loop**: Fixed logical 60 fps timestep via a dt accumulator; physics and timers tick in logical frames, rendering follows the display refresh
- **File Structure**: Single HTML file with embedded CSS and JS for simplicity

### 5. Additional Features
- **Touch Support**: Mobile-friendly touch controls
- **Accessibility**: Clear visual feedback and controls
- **Scalability**: Easy to extend with new features
- **Documentation**: Clear README with usage instructions

## Future Enhancement Ideas

### Visual Enhancements
- Animated bird sprites
- Particle effects for collisions
- Background parallax scrolling
- Different themed levels

### Game Features
- High score tracking (local storage)
- Power-ups (slow motion, invincibility)
- Sound effects and background music
- Multiple difficulty settings

### Cultural Elements
- More Vietnamese cultural references
- Localized names for game elements
- Traditional Vietnamese music integration

## Development Approach

### Phase 1: Core Game Mechanics
- Basic bird movement and physics
- Pipe generation and collision detection
- Score tracking system

### Phase 2: Visual Design
- Implement visual elements as described
- Polish UI components
- Add Vietnamese cultural touches

### Phase 3: Polish & Enhancement
- Add touch support
- Implement game states (start, playing, game over)
- Final testing and bug fixes

## Acceptance Criteria

1. Game runs smoothly in modern browsers
2. All core gameplay mechanics work as intended
3. Visual design matches requirements
4. Controls are responsive and intuitive
5. Game properly handles all game states
6. Source code is well-commented and organized
7. README provides clear usage instructions

## Tools & Technologies Used
- HTML5 Canvas for rendering
- JavaScript for game logic
- CSS3 for styling
- Git for version control

## Known Limitations
- Single HTML file implementation (no external dependencies)
- No bundled audio files; background music and the optional bridge-break sound cue are player-enabled via the settings sound toggle (see Accessibility Enhancements)
- Limited to browser-based execution