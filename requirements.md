# Flappy Binh - Game Requirements

## Project Overview
Flappy Binh is a parody of the popular Flappy Bird game with a Vietnamese cultural twist. The goal is to create an engaging, simple-to-play game that captures the essence of Flappy Bird while adding unique Vietnamese elements.

## Core Requirements

### Game Mechanics
- **Basic Gameplay**: Player controls Ethan that continuously falls and must flap to avoid pipes
- **Controls**: Click/tap to make Ethan flap upward
- **Collision Detection**: 
  - Pipe collisions (top and bottom)
  - Ground collision
  - Ceiling collision
- **Scoring System**: +1 point for each pipe successfully passed
- **Difficulty Progression**: Pipes move faster as score increases
- **Theme Changes**: Every 25 points, the theme changes between daytime and dusk
- **Speed Increase**: Every 25 points, pipe speed increases by 25% (pipeSpeed = baseSpeed * (1 + 0.25 * floor(score/25)))
- **High Score Persistence**: Store the highest score in `localStorage` under key `flappyBinhHighScore` and display it on the Game‑Over screen.
- **Accessibility Enhancements**: 
  - High‑contrast UI colors for color‑blind friendliness.
  - Keyboard shortcuts: Space/Enter to flap, `R` to restart.
  - Optional sound‑off toggle in the settings menu.

- **Special Ending**: When Ethan hits a pipe, ground, or ceiling, he falls into a toilet and gets pooped on by 2-3 random birds before game over
- **Boss Battle Stage**: Advanced stage with unique boss battle mechanics (inspired by Mario's Bowser battles) occurring every 100 points

- **Head Graphic Constraints**: Render `binh-head.png` at a height of 48 px (maintaining aspect ratio). The image must be cropped to remove any background before rendering.
- **Vietnamese Cultural Elements**: Include at least one of the following in the background:
  - Vietnamese flag colors (red background with a yellow star).
  - Traditional pattern overlay (e.g., a subtle lotus motif).
  - Optional background music snippet of traditional Vietnamese instruments.
- **Responsive Design Breakpoints**:\
  - Mobile: viewport width ≤ 480 px – UI elements stack vertically, score displayed at top center.\
  - Tablet: viewport width > 480 px and ≤ 1024 px – UI elements arranged horizontally, score on the right, with slightly larger touch targets.\
  - Desktop/Laptop: viewport width > 1024 px – UI elements arranged horizontally with additional margin, optional side panel for extra stats.\

- **Asset Loading Strategy**: Lazy‑load heavy assets (head image, boss sprites) after the initial game canvas is created to keep the initial page load < 200 KB.

- **Ethan (Binh) Character**: Cartoon character combining actual head graphic with cartoon body
- **Head Reference**: Use reference image `binh-head.png` for facial features (eyes, mouth, hair style) - resized and background removed
- **Body**: Cartoon body design (blue shirt, red pants) with simple limbs
- **Pipe Design**: Green pipes with brown caps
- **Background**: Sky gradient with clouds
- **Ground**: Brown ground with green grass
- **Vietnamese Themed Elements**:
  - Vietnamese flag elements in background
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
- No sound effects or music
- Limited to browser-based execution