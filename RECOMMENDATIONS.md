# Flappy Binh — Recommendation Addendum

Concrete proposals for every requirement that was flagged as under‑specified. Values are
anchored to the two reference games this is a mashup of:

* **Flappy Bird** (2013) — single‑tap physics, right‑to‑left pipe gauntlet, +1/pipe.
* **Super Mario Bros. Bowser castle fights** (1985) — a solid bridge spanning the whole
  level, a boss who jumps on a fixed cadence and hurls projectiles, standing on the bridge
  is safe.

Every number below is a **proposal**, not a discovered fact. Treat as the baseline to
implement against and tweak in playtesting. Anything marked *(confirm)* should be a quick
design decision rather than a guess at build time.

---

## A. Core gameplay (Flappy Bird fidelity)

### A1. Physics constants
The analysis code and the requirements never pin gravity/flap. For a 512‑px‑tall canvas
Flappy Bird plays at roughly these numbers:

| Constant | Value | Notes |
|---|---|---|
| `gravity` | `0.45 px/frame²` | applied each frame to velocity |
| `flapVelocity` | `-8.0 px/frame` | set on each tap; overrides current velocity |
| `maxFallSpeed` | `12 px/frame` | clamp downward velocity so death falls read clearly |
| `playerX` | `1/4 of canvas width` | fixed; only vertical (and boss‑battle horizontal) motion |

**Recommendation:** define a `PHYSICS` object with these four and clamp velocity. The
analysis code already uses `gravity:0.5, jump:-10` — keep that but **add the max‑fall
clamp** so the toilet‑fall death animation is legible.

### A2. Canvas / logical resolution
Requirements say "responsive" but never define the world the game runs in.

**Recommendation:** run the simulation in a **fixed logical 288×512** world (the classic
Flappy Bird play area, portrait) and letterbox/scale it to the viewport with `devicePixelRatio`
awareness. All constants (pipe speed, gaps, hitboxes, laser speeds) are then in *logical*
pixels and the responsive breakpoints in the requirements control the surrounding UI, not
the physics. This kills an entire class of "it's different on mobile" bugs.

### A3. Scoring trigger (bug in current spec)
`requirements.md` says "+1 per pipe passed," but the analysis code awards the point when a
pipe goes **off‑screen** (`p.x + p.width <= 0`). That is ~2 screens late and lets a player
die on a pipe and still be credited for it.

**Recommendation:** score when the player **passes the pipe's right edge**:
`if (p.scored === false && p.x + p.width < playerX) { p.scored = true; score++; }`.
This is how Flappy Bird scores and it is unambiguous.

### A4. Pipe speed base + cap
`baseSpeed` is referenced but never set. Analysis uses `baseDx = 2`.

**Recommendation:** `baseSpeed = 2.5 px/frame` (logical), **cap at `8 px/frame`** (~3.2×).
Keep the stated formula `pipeSpeed = baseSpeed * (1 + 0.25 * floor(score/25))` but write the
cap in as `Math.min(pipeSpeed, 8)`. At score 25 → 3.125, 50 → 3.75, …, capped at 8 around
score 100 — which is exactly when the first boss fight starts, so difficulty handoff is
clean.

### A5. Pipe spawn cadence
"one pipe pair every ~100 frames (adjustable)" — vague and frame‑rate dependent.

**Recommendation:** spawn every `90` **logical frames** (1.5 s at 60 fps) with a fixed
gap of **150 px** (already spec'd) and pipe width **52 px** (classic). If a difficulty knob
is wanted, scale spawn interval, *not* gap, so the game never becomes geometrically
impossible. Use a time accumulator (`dt`‑based), not a raw `frames % 90`, so 60 fps and
120 Hz displays spawn identically.

### A6. Forgiving hitbox (important for feel)
Flappy Bird is *much* easier than its sprite suggests because the collision box is smaller
than the sprite.

**Recommendation:** inflate the player hitbox **inward by 4 px** on each side when testing
collisions, while drawing the full 48 px head. This is the single biggest factor in
"fun vs. unfair." Make it a named constant `HITBOX_INSET = 4`.

### A7. Theme + speed change event order
Both happen at every 25 points and the doc doesn't say which runs first or what "theme"
values are.

**Recommendation:** on the *same* score increment, run in this order:
1. `speed = Math.min(baseSpeed * (1 + 0.25 * floor(score/25)), 8)`
2. `theme = (score/25) % 2 === 0 ? 'dusk' : 'day'`

Define the two themes as objects so the switch is data, not scattered `if`s:

```js
const THEMES = {
  day:  { sky:'#87CEEB', cloud:'rgba(255,255,255,0.85)', ground:'#3CB371' },
  dusk: { sky:'#2C3E50', cloud:'rgba(120,120,140,0.8)',  ground:'#2E7D32' }
};
```
Crossfade over **30 frames** (0.5 s) rather than a hard snap — reads as "night is falling,"
not a glitch.

### A8. Death / toilet sequence (the "poop" ending)
Spec: "hits a pipe, ground, or ceiling → falls into a toilet, pooped on by 2‑3 random birds."
No timing, no assets, and note that in real Flappy Bird a **ceiling hit is not death** (you
just bonk and fall).

**Recommendation — pin the timeline (total ~1.6 s) and keep ceiling a non‑lethal bonk:**

| t (s) | Event |
|---|---|
| 0.0 | Collision detected. Freeze pipe scroll. Player velocity → `maxFallSpeed`. |
| 0.0–0.6 | Player tumbles (rotate ~‑90°) down to a toilet sprite that spawns at `playerX, groundY`. |
| 0.6–1.4 | 2–3 bird sprites (pick `2 + floor(rand*2)`) arc in from the top, each dropping a small "poop" sprite onto the player. |
| 1.4–1.6 | Screen flash / thud, then transition to Game‑Over card. |

Ceiling: on hit, set `y=0, velocity=0` (bonk) — **no death**. Only pipe/ground are lethal.
Document this explicitly because the current spec lists "ceiling collision" under collision
detection without saying it's lethal, and the analysis code treats it as non‑lethal.

---

## B. Assets

### B1. Head graphic — background already removed ✅
Measured `binh-head.png`: **276×363, RGBA, corners fully transparent, 26.8% of pixels
transparent, 70.7% opaque.** The "crop to remove any background" requirement is already
satisfied — do **not** run an auto‑crop/matting step that could shave the subject.

**Recommendation:**
* Render at **height = 48 px** → width = `48 * (276/363) ≈ 36.5 px` (preserve aspect).
* The source is *portrait* (0.76 ratio), so on a 512‑px‑tall play field a 48 px‑tall head
  is small and correct — the body sits below it.
* **Remove** the "background removed" step from the asset pipeline; instead add a
  **fallback** `head_placeholder.svg` used if the PNG fails to load (offline / 404), so the
  game never draws a broken image.
* Preload with `requestIdleCallback` after canvas init to keep first paint < 200 KB
  (the PNG is 124 KB, so this is the binding constraint — see B3).

### B2. Cartoon body
"blue shirt, red pants, simple limbs" is fine but no dimensions.

**Recommendation:** body sprite **36 × 32 px** (matches head width 36.5), drawn directly
under the head so total character ≈ 48 + 32 = 80 px tall. Legs 8×15, arms 5×10 as in the
analysis code. Animate a 2‑frame idle bob (±1 px, 8 fps) and a 3‑frame flap so it feels
alive. Keep it as canvas vector drawing (no extra PNG) to respect the no‑dependencies rule.

### B3. Initial payload < 200 KB
Only `binh-head.png` (124 KB) + HTML/JS/CSS. This is achievable but tight.

**Recommendation:** lazy‑load the PNG with `requestIdleCallback` as spec'd, and keep all
other art (pipes, bridge, boss, hatchet, poop, birds) as **canvas‑drawn vectors**. That
keeps the mandatory first payload to `index.html` (target < 50 KB) + fonts, with the 124 KB
head arriving in the idle window. If the PNG can be exported as a lossy‑less **WebP**, it
drops to ~60 KB and the budget becomes trivial — worth a one‑line export.

---

## C. Boss battle (Bowser‑castle fidelity)

The current spec is 80% right. The gaps are mostly *timing and coordinates*. Below are the
proposals that make it buildable.

### C1. Bridge geometry
Spec is contradictory: "spans the **entire canvas width**" but the analysis code uses
`bridgeWidth = 300`.

**Recommendation:** `bridgeWidth = canvasWidth` (logical 288), **20 px thick**, top surface
at `bridgeY = 0.62 * canvasHeight` (~317 px on a 512 field) so the pit below is ~195 px —
a real, deadly gap, matching the Bowser‑castle "fall off the edge" threat. The ±5 px
landing margin is then a *forgiveness band* past each screen edge.

### C2. Boss cadence — replace per‑frame random (bug in analysis)
Analysis does `Math.random() < 0.01` per frame to change state — that's ~0.6 state changes
/sec, unpredictable and not "Bowser‑like." The requirements doc already states the fix
(fixed 3‑second ticks) but the analysis code ignores it.

**Recommendation:** drive the boss from a single **tick counter** in frame units:
```
tick = (frame % 180) === 0        // 180 frames = 3 s at 60 fps
on tick:  action = (Math.random() < 0.5) ? 'jump' : 'shoot'
```
One action per tick, never both — exactly as the requirements doc says. This is the SMB
cadence: Bowser acts on a metronome, which players can *read*.

### C3. Boss jump
"max jump height 1/3 screen, min frequency 3 s." Pin the animation:

**Recommendation:** parabolic arc, apex `= canvasHeight/3` above the bridge, total air time
**0.8 s** (48 frames), `easeOutQuad` up / `easeInQuad` down, land with a 4‑frame dust puff
+ slight bridge shake. Boss is **invulnerable to lasers but its hitbox moves with the
arc**, so the player can dodge by timing.

### C4. Laser — normal + five‑beam special
Two things are underspecified: normal shot direction, and the fan geometry of the 5‑beam
special.

**Recommendation:**
* **Normal shot:** one beam, fired **horizontally toward the player's side** (the mole
  "aims" straight across the bridge), speed `2 px/frame * laserMultiplier`. Mole outline
  flashes red for 0.5 s before fire (already spec'd) — keep it.
* **Laser speed:** base `2 px/frame`, `+10%` per successive boss, **hard cap 6 px/frame**
  (3×). Write the cap in the formula: `laser = Math.min(2 * 1.1**bossCount, 6)`.
* **Five‑beam special:** fire 5 beams in **rapid succession** (0.08 s apart = 5 frames each),
  fanned around the horizontal axis. Angles from the mole: `[-40°, -20°, 0°, +20°, +40°]`
  (0° = straight across toward the player). This is a readable "spread shot," clearly
  distinct from the single horizontal beam.
* **Special cadence + warning:** the mole fires the 5‑beam special every **12 seconds**
  (720 frames), and a **3‑s on‑screen countdown** (flashing, around the mole) starts at
  `t-3 s`. That warning window is what makes a 5‑beam barrage fair — without it the player
  can't react. Reset the countdown after each special.

### C5. Player movement in the boss stage
Spec: left/right **plus** flap during the battle, but no key map or bridge‑walking rules.

**Recommendation:**
* Keys: **A/←** = left, **D/→** = right (1 logical px/frame each, clamped to `[0, canvasWidth]`),
  **Space/Enter** = flap (same physics as normal play). Touch: hold left/right third of screen
  to steer, center third to flap.
* **On the bridge** (player bottom ≥ `bridgeY` and within the ±5 px band): gravity is
  disabled, player can walk and stand. This is the Bowser‑castle safe‑zone rule.
* **Off the bridge edges** (past the ±5 px band) or into the pit: fall, then loss.

### C6. Hatchet + win
Spec: "touch hatchet → bridge breaks → win." No placement or sizes.

**Recommendation:**
* Hatchet **32×32**, placed on the bridge at `x = 0.75 * canvasWidth`, `y = bridgeY - 16`
  (just above the deck, on the far side of the boss).
* Touching it triggers: **0.5 s** bridge‑collapse (`easeInBack`, deck sinks + cracks),
  optional `bridge_break.wav`, boss drops into the pit, then **Victory** → resume normal
  play at `score + 100`, `pipeSpeed` recalculated, `bossCount++`.
* **Hatchet is the only win path** — you must fly past/over the boss to reach it; walking
  straight across is blocked by the boss occupying the mid‑bridge zone. (Bowser always
  stands between you and the objective.)

### C7. Boss hitboxes
Spec gives shapes but no anchors/sizes.

**Recommendation:**
* Boss body: AABB `60×80`, top‑left at `(bossX, bossY)`.
* Mole: circle, **r = 10 px**, centered at `(bossX + 20, bossY + 45)` (left‑of‑mouth, per
  the character design).
* Laser: line segment, **5 px** collision radius, from mole center to beam tip.
* Hatchet: AABB `32×32`.
* Player: same `HITBOX_INSET = 4` forgiveness as normal play, so the boss fight isn't
  spike‑harsher than the pipes.

### C8. Transition into the boss stage
"pipes disappear, scroll briefly, bridge appears" — pin the duration.

**Recommendation:** on the score that is a multiple of 100:
1. `state = 'bossIntro'`; clear the pipe array; **freeze** player physics for 0.5 s.
2. Keep scrolling the background **60 frames (1 s)** as the full‑width bridge slides in
   from the right and the boss fades in at a random end (left/right).
3. `state = 'bossActive'`; enable boss tick clock and laser logic.
This "bridge slides in" beat is the Bowser‑castle entrance and gives a clear read.

### C9. Loss during boss = full restart
Spec: lose the boss fight → whole game restarts. This is harsh; document it as a *design
choice* so it's intentional, not a bug.

**Recommendation:** on any boss‑fight loss (laser, boss body, pit): play the 1.6 s toilet
death sequence (reuse A8), then reset **everything** — `score=0`, `speed=base`,
`laserMult=1`, `bossCount=0`, `theme=day`, clear boss objects, → Start screen. **Keep the
`localStorage` high score** (don't wipe the player's best). Persist the high score only on
a new record.

### C10. Difficulty progression
Spec: "later battles = faster lasers, more frequent jumping, narrower passage." But the
cadence is fixed at 3 s and only laser speed scales. Reconcile:

**Recommendation:** keep the **3‑s cadence fixed** (that's what makes it readable and
Bowser‑like) and scale difficulty through **laser speed only** (+10%/boss, cap 3×), plus a
soft ramp: from boss #2 onward, the 50/50 jump/shoot split shifts to **60/40 toward shoot**
(more beams to dodge). Do **not** shorten the cadence — a faster, unreadable boss is worse
than a slow, fair one. "Narrower passage" is achieved by the boss occupying more of the
mid‑bridge zone in later fights (wider idle drift range), not by shrinking the gap.

---

## D. UI / accessibility / responsive

### D1. Score display
**Recommendation:** big white outlined number, top‑center during play (matches the mobile
breakpoint already spec'd). On increment, scale 1.3× → 1.0 over 100 ms (the analysis code
has the hook).

### D2. High score
**Recommendation:** `localStorage` key `flappyBinhHighScore`, shown on the Game‑Over card as
"Best: N." Update only when the final score beats it.

### D3. Sound toggle
**Recommendation:** a `#soundToggle` in a small settings (gear) button on the start screen.
Default **on**, persisted in `localStorage` as `flappyBinhSoundEnabled`. This resolves the
direct contradiction in the docs (requirements list "optional music" under features *and*
"no sound" under known limitations) — the resolution: **no bundled audio files required**,
a **player‑enabled** optional music track + the optional `bridge_break.wav` cue. Update the
"Known Limitations" line accordingly.

### D4. Keyboard
**Recommendation:** `Space`/`Enter` flap (ignore `e.repeat`, call `preventDefault`), `R`
restart — `R` works in *playing* and *game‑over* states only, does **not** clear the high
score.

### D5. Breakpoints
The spec's mobile/tablet/desktop layout is fine. Pin the desktop side panel:
`#statsPanel`, width **280 px**, `grid-template-columns: 1fr 280px`, shows High Score /
Bosses defeated / Laser level, hidden below 1024 px.

### D6. Color‑blind palette
"high‑contrast" is not measurable. **Recommendation:** state WCAG AA (≥ 4.5:1) for text and
give the palette:
```
--fg:#FFFFFF  --bg:#1E88E5  --accent:#FFD500  --danger:#FF5252
```
and ship a `?cb=1` query toggle (or a settings option) that swaps the pipe green for a
**blue/orange** pair and adds a **shape cue** (chevron) on the top vs bottom pipe cap so
the gap is identifiable without color.

---

## E. Documentation fixes (quick wins)

* **README `claude` command:** path is `/mnt/hermes/data/claude/flappy-binh` — the real
  path is `/mnt/hermes_data/claude/flappy-binh`. Fix the slash.
* **README browser run:** add "serve the folder (`python3 -m http.server 8000`) to avoid
  `file://` asset/CORS issues."
* **requirements.md "Known Limitations: No sound effects"** — reword to "No bundled audio;
  optional music + bridge cue are player‑enabled."
* **requirements_analysis.md sample code** — it is the source of three real bugs (off‑screen
  scoring, 300‑px bridge, per‑frame random boss). Either fix it to match the decisions in
  this addendum or clearly mark it as *illustrative, superseded by the requirements docs*.
* Add a **Future Enhancements** table (Priority / Feature / Rationale) so power‑ups and
  parallax have an owner and a rank.

---

## F. What to implement *first* to de‑risk

1. Fixed 288×512 logical world + `PHYSICS` constants + time‑accumulator loop (A1, A2, A5).
2. Correct scoring (pass the pipe, not off‑screen) (A3).
3. Forgiving hitbox (A6) — this changes the *feel* more than anything.
4. Boss tick clock (fixed 3 s) instead of per‑frame random (C2) — this changes the fight
   from "unfair noise" to "readable pattern."

These four are the highest‑leverage fixes and are all unambiguous with the values above.
