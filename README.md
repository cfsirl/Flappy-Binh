# Flappy Binh - Parody Edition

A parody of the popular Flappy Bird game, reimagined with a Vietnamese twist!

## Game Features
- Simple touch/click controls
- Increasing difficulty
- Score tracking
- Parody elements inspired by Vietnamese culture
- Boss battle stage (inspired by Mario's Bowser battles)

## Controls
- Click or tap to make the bird flap
- Avoid pipes and obstacles
- Try to achieve the highest score!

## Development
This game is built using HTML5, CSS3, and JavaScript with a focus on simplicity and fun.

## How to Play
1. Click "Start Game" to begin
2. Click or tap to make the bird flap upward
3. Navigate through the pipes without hitting them
4. Each pipe you pass gives you 1 point
5. Game ends when you hit a pipe or the ground

## Boss Battle Stage
An advanced stage featuring:
- Unique boss character (slender girl with large mole)
- Laser shooting attacks from the mole
- Bridge destruction mechanics
- Bowser-style jumping patterns
- Safe zone on bridge for strategic gameplay

## Running with Claude
To run this project with Claude:
1. Ensure you're logged into Claude via the command line: `claude login`
2. Run: `claude /mnt/hermes_data/claude/flappy-binh`

## Running in Browser
To play directly in your browser:
1. Serve the project folder over HTTP (avoids `file://` asset/CORS issues):
   ```bash
   cd /mnt/hermes_data/claude/flappy-binh
   python3 -m http.server 8000
   ```
2. Open `http://localhost:8000` in any modern web browser
3. Click "Start Game"
4. Click or tap to make the bird flap

## Ethan (Binh) Character
The main character is Ethan, also known as Binh, a Vietnamese name. 
A reference image of his head is included in this project as `binh-head.png`.

## License
MIT