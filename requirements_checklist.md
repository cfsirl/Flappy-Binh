# Flappy Binh – Requirements Specification Checklist

> Use this checklist during implementation and QA. Mark each ✅ when the corresponding requirement is fully implemented, documented, and verified by automated or manual tests.

---

## 1. Core Gameplay (requirements.md)
| ✅ | Item | Details / Test Criteria |
|----|------|--------------------------|
| [ ] | Fixed **288×512 logical world**, DPR-aware canvas scaling; physics in logical px, UI governed by breakpoints. | Resize viewport; physics constants unaffected. |
| [ ] | Physics constants defined: `gravity 0.45`, `flapVelocity −8.0`, `maxFallSpeed 12`, `playerX = 72` (baseline — confirm in playtest). | Unit test: after N flaps/fall frames, position matches hand-computed trajectory. |
| [ ] | Fixed 60 fps dt-accumulator game loop; 120 Hz display behaves identically. | Run on 120 Hz + 60 Hz; compare pipe positions after 60 s. |
| [ ] | Collision detection: pipe/ground lethal, **ceiling non-lethal bonk** (`y=0, velocity=0`); checked each frame after position update. | Bonk the ceiling mid-run: no death, player continues. |
| [ ] | Player hitbox inset 4 px on all sides (`HITBOX_INSET = 4`) for all collision checks. | Debug hitbox overlay; graze test vs sprite edge. |
| [ ] | Scoring on **pipe pass** (`pipe.x + pipe.width < playerX`, once per pipe); off-screen exit never scores. | Kill the run on an unpassed pipe → no credit; pass all pipes on screen → correct count. |
| [ ] | `baseSpeed = 2.5 px/frame`, formula with **8 px/frame cap**. | Assert `pipeSpeed` ≤ 8 after score progression (cap reached ~score 100). |
| [ ] | Pipe design: 52 px body, 150 px gap (gap-center randomized, never narrowed), spawn every **90 logical frames** via dt accumulator. | Automated test runs 1000 frames: min gap ≥ 150, spawn interval ≈ 90 ± frame-tolerance. |

## 2. Boss‑Battle Stage (boss_battle_requirements.md)
| ✅ | Item | Details / Test Criteria |
|----|------|--------------------------|
| [ ] | Laser speed: base 2 px/frame, `min(2 * 1.1**bossCount, 6)` — cap 6 px/frame (3×). | Verify after 5 boss battles `laserSpeed ≤ 6`. |
| [ ] | Normal shot: single horizontal beam aimed at the player's side. | Visual inspection; beam travels horizontally across the bridge. |
| [ ] | Five-beam special: beams 0.08 s apart, fan `[-40°, -20°, 0°, +20°, +40°]`, fires every 12 s of active battle. | Record timestamps of the 5 beams; measure spread vs spec. |
| [ ] | Special warning: 3 s on-screen flashing countdown, red in final 0.5 s, hidden after barrage; never fires without it. | Visual test of timer; no special without full countdown. |
| [ ] | Pre-shot cue: mole outline flashes red 0.5 s before any laser fires. | Timing measured via console timestamps. |
| [ ] | Boss tick: fixed 3 s (180 logical frames), one action/tick, 50/50 jump/shoot, never both. | Log 20 ticks; exactly one action each, ≈ 50/50 over many runs. |
| [ ] | Boss #2+ difficulty: tick split shifts 50/50 → 60/40 toward shoot; idle drift range widens (narrower passage). | Observe drift range and shoot frequency in boss #2 vs #1. |
| [ ] | Boss jump: parabolic arc, apex ⅓ screen height, 0.8 s, easeOutQuad/easeInQuad, landing puff + bridge shake. | Capture animation frames; verify apex position and duration. |
| [ ] | Bridge: full canvas width (288), 20 px thick, `bridgeY = 0.62 * canvasHeight` (≈317). | Verify geometry in debug overlay. |
| [ ] | Boss-battle controls: A/D + arrows horizontal (1 px/frame, clamped), Space/Enter flap; touch thirds (left/right steer, center flap). | Press left/right during normal play – no effect; during boss – moves player; test touch. |
| [ ] | Safe zone: on bridge (bottom ≥ `bridgeY`, within ±5 px band) gravity off; beyond the band → pit → loss. | Walk off bridge past margin → loss. |
| [ ] | Player hitbox uses the same 4 px inset in the boss battle. | Debug overlay matches normal-play inset. |
| [ ] | Transition timeline: pipe clear + 0.5 s physics freeze → 1 s bridge slide-in + boss fade-in at random end → `bossActive` with tick clock and 12 s countdown started. | Automated script steps through frames; verifies state changes at t=0/1s. |
| [ ] | Hatchet: 32×32 at `(0.75 * canvasWidth, bridgeY - 16)`; only win path (boss blocks walking). | Collision test; attempt straight walk → blocked. |
| [ ] | Hatchet touch → 0.5 s `easeInBack` collapse + synthesized bridge-break cue (WebAudio, subject to sound toggle) → boss falls → Victory → resume at score+100, `bossCount++`. | Play animation; confirm duration, sound trigger (and that it mutes with the toggle), score continuity. |
| [ ] | Hitboxes: boss body AABB 60×80 at `(bossX, bossY)` (follows arc); mole r=10 at `(bossX+20, bossY+45)`; laser line 5 px radius; hatchet AABB 32×32. | Debug overlay shows correct shapes and anchors. |
| [ ] | Loss during boss: 1.6 s death sequence → full restart (score/speed/laser/bossCount/theme reset) → start screen; **localStorage high score retained**. | After loss, inspect `localStorage` (unchanged), `score`, `state`. |

## 3. All Decisions Finalized ✅
The previously pending items (A7 themes, A8 death-sequence art, B2 body sprite, B3 WebP, D1–D6 UI/accessibility) were all accepted on 2026-10-06 and are now specified in the requirements docs above. Remaining verification is implementation-level only.

## 4. Documentation & General
| ✅ | Item | Details / Test Criteria |
|----|------|--------------------------|
| [ ] | Correct `claude` run command path (`/mnt/hermes_data/claude/flappy-binh`). | Verify README shows exact command; test by running (dry‑run). |
| [ ] | Browser run instructions include static server note. | README contains `python -m http.server 8000` example. |
| [ ] | License section present (MIT). | License file exists and matches header. |
| [ ] | Future enhancement table with priority, feature, rationale added. | Table present in README under “Future Enhancement Ideas”. |
| [ ] | Side‑panel spec (content, width, background) documented in requirements. | Verify checklist item 2‑15 references side‑panel. |
| [ ] | No bundled audio/image assets beyond `binh-head.png` (+ its WebP); all other art is canvas-drawn and all sound is WebAudio-synthesized. | Grep for asset references; only the head image (png/webp) appears as a file. |

---

**How to use:**
1. Open `requirements_checklist.md` in your editor.
2. As you implement each feature, run the associated test(s).
3. Tick the checkbox (`[x]`) when the test passes and the implementation matches the spec.
4. Commit the updated checklist to the repository.

*Maintainer note:* Keep this checklist version‑controlled alongside the source code. When new requirements are added, extend the relevant section with a new row.
