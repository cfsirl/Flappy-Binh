# Flappy Binh - Requirements Analysis

> **Status: illustrative / superseded.** This document predates the resolved requirements.
> Its sample code contains three known deviations from the authoritative requirements docs
> (`requirements.md`, `boss_battle_requirements.md`) and the decisions in
> `RECOMMENDATIONS.md` — treat the code below as a sketch, not a spec:
> 1. **Scoring** is awarded when a pipe leaves the screen (`p.x + p.width <= 0`); requirements
>    score when the player *passes* the pipe.
> 2. **Bridge width** is hard-coded to `300`; requirements specify the bridge spans the
>    **entire canvas width**.
> 3. **Boss state** changes per-frame via `Math.random() < 0.01`; requirements use a **fixed
>    3-second tick** (one action per tick, 50/50 jump/shoot).
>
> When implementing, follow the requirements docs and `RECOMMENDATIONS.md`, not this file.

## Understanding the Requirements

Looking at the requirements document, I can see this is a well-structured project with clear specifications. The key elements are:

### Core Game Mechanics
- Basic bird physics with gravity and flapping
- Pipe obstacles that move from right to left
- Collision detection for pipes, ground, and ceiling
- Score tracking system

### Visual Design Requirements  
- Bird character with eye and beak
- Green pipes with brown caps
- Sky background with clouds
- Brown ground with green grass
- Vietnamese cultural elements in the background

### Technical Requirements
- Single HTML file implementation
- Web-based (HTML5, CSS3, JavaScript)
- Responsive design for both desktop and mobile

## Implementation Plan Based on Requirements

Based on the updated requirements, here's what I would implement:

1. **Game Structure**:
   - Canvas element for rendering
   - Game state management (start, playing, game over)
   - Physics engine for Ethan (Binh) movement with special falling behavior
   - Theme and difficulty progression system
   - Boss battle trigger system

2. **Core Components**:
   - Ethan (Binh) object with position, velocity, and physics
   - Special falling logic when hitting obstacles
   - Toilet rendering when Ethan falls
   - Random bird generation for the poop scene
   - Pipe generation system with random gaps
   - Collision detection system
   - Score tracking with theme/speed progression
   - Theme switching between daytime and dusk every 25 points
   - Speed increase of 25% every 25 points
   - Boss battle trigger (every 100 points)
   - Boss battle progression system (laser speed increases by 10% each time)

3. **Visual Elements**:
   - Ethan character using actual head image (`binh-head.png`) resized and background removed
   - Cartoon body design (blue shirt, red pants) with simple limbs
   - Toilet rendering when Ethan falls
   - Bird characters with poop effects
   - Background elements that change between daytime and dusk themes
   - Boss battle environment (bridge, hatchet, boss character)
   - Laser beam effects for boss attacks

4. **User Interaction**:
   - Mouse/touch controls for flapping (Ethan jumping)
   - Start and restart buttons
   - Responsive design

## Sample Implementation Code

> ⚠️ **Do not use as-is.** This sketch has the three known deviations noted in the header
> (off-screen scoring, 300 px bridge, per-frame random boss state). It shows structure only.

Here's how I would approach the implementation based on the requirements:

```javascript
// Game initialization
const canvas = document.getElementById('game-canvas');
const ctx = canvas.getContext('2d');

// Game state with theme progression
let score = 0;
let frames = 0;
let theme = 'daytime'; // 'daytime' or 'dusk'
let pipeSpeedMultiplier = 1.0;
let bossBattleTriggered = false;
let bossBattleCount = 0; // Number of boss battles completed
let laserSpeedMultiplier = 1.0; // Laser speed increases by 10% each boss battle

// Ethan (Binh) object with physics properties
const ethan = {
    x: 50,
    y: canvas.height / 2 - 10,
    width: 34,
    height: 48, // Increased for full body
    gravity: 0.5,
    velocity: 0,
    jump: -10,
    falling: false,
    toiletX: 0,
    toiletY: 0,
    toiletWidth: 60,
    toiletHeight: 80,
    
    draw: function() {
        // Draw Ethan's head using the actual reference image
        // (Implementation would load and draw binh-head.png resized)
        
        // Draw cartoon body elements
        ctx.fillStyle = '#4169E1'; // Blue shirt
        ctx.fillRect(this.x, this.y + 30, this.width, 20);
        
        // Draw arms
        ctx.fillStyle = '#FFD700'; // Yellow skin tone for arms
        ctx.fillRect(this.x - 5, this.y + 30, 5, 10); // Left arm
        ctx.fillRect(this.x + 34, this.y + 30, 5, 10); // Right arm
        
        // Draw legs
        ctx.fillStyle = '#8B0000'; // Dark red pants
        ctx.fillRect(this.x + 5, this.y + 50, 8, 15); // Left leg
        ctx.fillRect(this.x + 21, this.y + 50, 8, 15); // Right leg
    },
    
    update: function() {
        if (gameRunning) {
            if (this.falling) {
                // Ethan falling into toilet
                this.velocity += this.gravity * 2; // Faster fall
                this.y += this.velocity;
                
                // Move toilet along with Ethan when falling
                if (!this.toiletX && !this.toiletY) {
                    this.toiletX = this.x + 10;
                    this.toiletY = this.y + 50;
                } else {
                    this.toiletY += this.velocity;
                }
            } else {
                // Normal game physics
                this.velocity += this.gravity;
                this.y += this.velocity;
                
                // Floor collision
                if (this.y + this.height >= canvas.height - ground.height) {
                    this.y = canvas.height - ground.height - this.height;
                    this.falling = true; // Trigger toilet fall
                    gameOver();
                }
                
                // Ceiling collision
                if (this.y <= 0) {
                    this.y = 0;
                    this.velocity = 0;
                }
            }
        }
    },
    
    flap: function() {
        if (!this.falling) {
            this.velocity = this.jump;
        }
    },
    
    reset: function() {
        this.y = canvas.height / 2 - 10;
        this.velocity = 0;
        this.falling = false;
        this.toiletX = 0;
        this.toiletY = 0;
    }
};

// Pipes array with speed progression
const pipes = {
    position: [],
    gap: 150,
    maxYPos: -150,
    baseDx: 2, // Base speed
    
    getSpeed: function() {
        return this.baseDx * pipeSpeedMultiplier;
    },
    
    draw: function() {
        for (let i = 0; i < this.position.length; i++) {
            let p = this.position[i];
            
            // Top pipe
            ctx.fillStyle = '#228B22';
            ctx.fillRect(p.x, p.y, p.width, p.height);
            
            // Pipe cap
            ctx.fillStyle = '#006400';
            ctx.fillRect(p.x - 5, p.y + p.height - 20, p.width + 10, 20);
            
            // Bottom pipe
            ctx.fillStyle = '#228B22';
            ctx.fillRect(p.x, p.y + p.height + this.gap, p.width, canvas.height);
            
            // Pipe cap
            ctx.fillStyle = '#006400';
            ctx.fillRect(p.x - 5, p.y + p.height + this.gap, p.width + 10, 20);
        }
    },
    
    update: function() {
        if (gameRunning) {
            if (frames % 100 === 0) {
                this.position.push({
                    x: canvas.width,
                    y: this.maxYPos * (Math.random() + 1),
                    width: 60,
                    height: 300
                });
            }
            
            for (let i = 0; i < this.position.length; i++) {
                let p = this.position[i];
                
                // Move pipe to the left at adjusted speed
                p.x -= this.getSpeed();
                
                // If pipe goes off screen, remove it
                if (p.x + p.width <= 0) {
                    this.position.shift();
                    score++;
                    
                    // Check for theme/speed changes every 25 points
                    if (score % 25 === 0 && score > 0) {
                        this.changeThemeAndSpeed();
                    }
                    
                    // Check for boss battle trigger every 100 points
                    if (score % 100 === 0 && score > 0 && !bossBattleTriggered) {
                        triggerBossBattle();
                    }
                    
                    scoreDisplay.innerHTML = `Score: ${score}`;
                }
                
                // Collision detection
                // Top pipe collision
                if (
                    ethan.x + ethan.width > p.x &&
                    ethan.x < p.x + p.width &&
                    ethan.y < p.y + p.height
                ) {
                    ethan.falling = true; // Trigger toilet fall on collision
                    gameOver();
                }
                
                // Bottom pipe collision
                if (
                    ethan.x + ethan.width > p.x &&
                    ethan.x < p.x + p.width &&
                    ethan.y + ethan.height > p.y + p.height + this.gap
                ) {
                    ethan.falling = true; // Trigger toilet fall on collision
                    gameOver();
                }
            }
        }
    },
    
    changeThemeAndSpeed: function() {
        // Switch theme between daytime and dusk
        theme = (theme === 'daytime') ? 'dusk' : 'daytime';
        
        // Increase speed by 25%
        pipeSpeedMultiplier *= 1.25;
    },
    
    reset: function() {
        this.position = [];
        this.baseDx = 2;
        pipeSpeedMultiplier = 1.0;
        theme = 'daytime';
    }
};

// Boss Battle System
const bossBattle = {
    // Boss properties
    bossX: 0,
    bossY: 0,
    bossWidth: 60,
    bossHeight: 80,
    bossHealth: 100,
    laserSpeedMultiplier: 1.0,
    
    // Laser properties
    lasers: [],
    baseLaserSpeed: 2, // Base speed for lasers
    
    getLaserSpeed: function() {
        return this.baseLaserSpeed * this.laserSpeedMultiplier;
    },
    
    // Hatchet properties
    hatchetX: 0,
    hatchetY: 0,
    hatchetWidth: 15,
    hatchetHeight: 20,
    
    // Bridge properties
    bridgeX: 0,
    bridgeY: 0,
    bridgeWidth: 300,
    bridgeHeight: 20,
    
    // Boss states
    state: 'idle', // 'idle', 'attacking', 'jumping'
    jumpTimer: 0,
    attackTimer: 0,
    
    // Boss drawing function
    draw: function() {
        // Draw bridge
        ctx.fillStyle = '#8B4513'; // Brown bridge
        ctx.fillRect(this.bridgeX, this.bridgeY, this.bridgeWidth, this.bridgeHeight);
        
        // Draw hatchet
        ctx.fillStyle = '#C0C0C0'; // Silver hatchet
        ctx.fillRect(this.hatchetX, this.hatchetY, this.hatchetWidth, this.hatchetHeight);
        
        // Draw boss character (slender girl with mole)
        ctx.fillStyle = '#FFB6C1'; // Pink body
        ctx.beginPath();
        ctx.arc(this.bossX + 30, this.bossY + 40, 30, 0, Math.PI * 2);
        ctx.fill();
        
        // Draw mole above left lip
        ctx.fillStyle = '#000000'; // Black mole
        ctx.beginPath();
        ctx.arc(this.bossX + 20, this.bossY + 45, 5, 0, Math.PI * 2);
        ctx.fill();
        
        // Draw laser beams if attacking
        for (let i = 0; i < this.lasers.length; i++) {
            let laser = this.lasers[i];
            ctx.fillStyle = '#FF0000'; // Red laser beam
            ctx.fillRect(laser.x, laser.y, 5, 10);
        }
    },
    
    update: function() {
        if (gameRunning && bossBattleTriggered) {
            // Boss movement and behavior
            this.updateBossBehavior();
            
            // Update lasers
            this.updateLasers();
        }
    },
    
    updateBossBehavior: function() {
        // Randomly change state
        if (Math.random() < 0.01) { // 1% chance per frame to change state
            const states = ['idle', 'attacking', 'jumping'];
            this.state = states[Math.floor(Math.random() * states.length)];
        }
        
        // Handle different boss states
        switch(this.state) {
            case 'attacking':
                // Shoot laser beam
                if (this.attackTimer <= 0) {
                    this.shootLaser();
                    this.attackTimer = 60; // Shoot every 60 frames
                } else {
                    this.attackTimer--;
                }
                break;
                
            case 'jumping':
                // Handle jumping animation
                if (this.jumpTimer <= 0) {
                    // Jump in arc trajectory
                    this.jumpTimer = 120; // Jump every 120 frames
                } else {
                    this.jumpTimer--;
                }
                break;
        }
    },
    
    shootLaser: function() {
        // Create a new laser beam from boss's mole
        this.lasers.push({
            x: this.bossX + 20, // Position at mole location
            y: this.bossY + 45,
            speed: this.getLaserSpeed()
        });
    },
    
    updateLasers: function() {
        // Update laser positions and remove off-screen lasers
        for (let i = this.lasers.length - 1; i >= 0; i--) {
            let laser = this.lasers[i];
            laser.x += laser.speed;
            
            // Remove if off screen
            if (laser.x > canvas.width) {
                this.lasers.splice(i, 1);
            }
        }
    },
    
    reset: function() {
        this.bossX = 0;
        this.bossY = 0;
        this.lasers = [];
        this.state = 'idle';
        this.jumpTimer = 0;
        this.attackTimer = 0;
        this.laserSpeedMultiplier = 1.0;
    }
};

// Draw background elements with theme changes
function drawBackground() {
    // Set background color based on theme
    if (theme === 'dusk') {
        ctx.fillStyle = '#8B4513'; // Dark orange/dusk sky
    } else {
        ctx.fillStyle = '#87CEEB'; // Daytime sky blue
    }
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    // Draw clouds
    if (theme === 'dusk') {
        ctx.fillStyle = 'rgba(105, 105, 105, 0.8)'; // Dark gray clouds in dusk
    } else {
        ctx.fillStyle = 'rgba(255, 255, 255, 0.8)'; // White clouds in day
    }
    
    ctx.beginPath();
    ctx.arc(100, 80, 30, 0, Math.PI * 2);
    ctx.arc(130, 70, 35, 0, Math.PI * 2);
    ctx.arc(160, 80, 30, 0, Math.PI * 2);
    ctx.fill();
    
    ctx.beginPath();
    ctx.arc(300, 120, 30, 0, Math.PI * 2);
    ctx.arc(330, 110, 35, 0, Math.PI * 2);
    ctx.arc(360, 120, 30, 0, Math.PI * 2);
    ctx.fill();
    
    // Draw Vietnamese flag elements (simplified)
    if (theme === 'dusk') {
        ctx.fillStyle = '#DA251D'; // Red
    } else {
        ctx.fillStyle = '#DA251D'; // Red (same for both themes)
    }
    ctx.fillRect(200, 200, 200, 100);
    
    if (theme === 'dusk') {
        ctx.fillStyle = '#FFFF00'; // Yellow
    } else {
        ctx.fillStyle = '#FFFF00'; // Yellow (same for both themes)
    }
    ctx.beginPath();
    ctx.moveTo(300, 250);
    ctx.lineTo(320, 270);
    ctx.lineTo(340, 250);
    ctx.lineTo(320, 230);
    ctx.closePath();
    ctx.fill();
}
```

## Next Steps

To implement the full game following these requirements, I would:

1. Create the complete HTML structure with canvas and UI elements
2. Implement all game objects (bird, pipes, ground)
3. Add physics and collision detection systems
4. Implement game states and user interaction
5. Polish visual design with Vietnamese cultural elements
6. Test across different devices and browsers

This approach directly addresses the requirements in the document while maintaining clean, readable code that can be easily extended.