# DINO-DRAGON QUEST — Project Planning & Architecture Log

## Overview
First-person raycasting Pacman game. Single HTML/CSS/JS file.
Theme: Lost 1994 shareware game aesthetic.
Font: "Press Start 2P" (Google Fonts).

---

## Build Phases
- [x] Phase 1: Raycasting engine + player movement
- [x] Phase 2: Maze generation + pellet placement
- [x] Phase 3: Enemy AI + collision
- [x] Phase 4: Power-ups + combat
- [x] Phase 5: HUD + minimap
- [x] Phase 6: Screens + audio (combined to stay within token limits)

---

## Architecture Decisions

### Canvas / Rendering
- Single HTML5 Canvas element
- Wolfenstein-style raycasting with fish-eye correction
- Target: 60fps via requestAnimationFrame
- Canvas size: TBD (awaiting user input; tentative 800×600)
- Wall textures: TBD (prompt cut off — awaiting clarification; tentative = solid colored rectangles per level)

### Player
- Controls: Arrow keys OR WASD
- Movement: forward/back/turn left/turn right
- No strafing (classic Wolfenstein style)

### Characters (Title Screen Select)
- DINO WARRIOR — fast, low attack
- DRAGON KNIGHT — slow, high attack
- PALEO PACMAN — balanced
- Stat differentiation: TBD (awaiting clarification on mechanical vs cosmetic)

### Maze
- Recursive backtracker generation
- 3 levels of increasing size/difficulty
- 2 SAFE ROOMS per level (ghosts cannot enter, glow green on minimap)
- Level 1: CRETACEOUS CAVERNS
- Level 2: MESOZOIC MINES
- Level 3: DRAGON'S LAIR (tight maze, boss ghost)

### Collectibles
- Small pellets — collect all to win level
- Power pellets — activate ghost-fear mode (ghosts turn blue, can be defeated)

### Enemies
- 3–4 ghost types as colored billboards
- Behaviors: chase, patrol, random
- Level 3 Boss: 2 hits to kill (color change after 1st hit — awaiting clarification)

### Power-ups (floating world icons)
- FIRE BREATH — ranged kill, 5s
- TAIL WHIP — close AOE stun
- FOSSIL SHIELD — absorb one hit
- SPEED BOOST — 8s sprint
- ANCIENT RAGE — all ghosts frightened 10s (rare)

### HUD
- Top: Score | Level | Pellets left | Lives (3)
- Bottom left: Character avatar + HP + active power-up + timer bar
- Bottom right: 2D minimap (player=arrow, enemies=red dots, safe rooms=green)

### Audio (Web Audio API, no external files)
- Footstep, pellet collect, power-up pickup
- Ghost frighten, ghost defeat, player death, level complete

### Persistence
- localStorage high score

---

## Clarifications Resolved
1. Wall textures — colored rectangles (no bitmaps) ✓
2. GitHub commits — after each phase ✓
3. Boss ghost — changes color after 1st hit ✓
4. Canvas — 800×600 ✓
5. Character stats — simple multipliers (moveSpeed, hitsToKill) ✓
6. No mobile/touch controls ✓
7. 3 enemy types (not 4) ✓
8. Phases 6+7 combined into one phase to stay under token limits ✓

---

## Architecture Decisions (confirmed)
- Canvas: 800×600, image-rendering: pixelated
- Raycasting: DDA algorithm, fish-eye corrected via perpendicular distance
- Coordinate system: +x = right, +y = down, angle 0 = facing right
- Map: 2D array, 1=wall, 0=open
- Wall shading: side=0 (vertical faces) bright, side=1 (horizontal faces) dark
- Collision: X and Y axes checked independently → wall sliding
- Wall palettes: Level 1 warm brown, Level 2 slate blue-grey, Level 3 deep red
- Characters stored in CHARACTERS dict, active char in `char` variable

---

## QA Notes

### Phase 1
- Raycasting + DDA: implemented correctly with perpendicular distance fish-eye fix
- Collision margin: 0.25 units — prevents clipping
- Wall shading: side differentiation gives convincing lighting at zero perf cost
- Key events: only prevent default on game control keys (ArrowKeys, WASD, Space)
- Map/MAP_W/MAP_H: use `let` so maze generator can reassign in Phase 2
- KNOWN: No frame-rate cap — runs at monitor refresh rate. Acceptable for now.

### Phase 2
- Maze gen: recursive backtracker (DFS) + 12% extra wall removals for loop passages
- Map values: 0=open, 1=wall, 2=safe room (walkable but ghosts excluded in Phase 3)
- Pellet grid: separate array (pelletGrid[y][x]) — keeps collision map clean
- Power pellets: max 6 per level, placed >3 cells from spawn
- Small pellets: all open cells except 3×3 spawn area (~80% of corridors)
- Safe rooms: 2 per level, random interior logical cells, not at spawn
- ZBuffer (Float32Array[800]): filled during wall pass, used for sprite depth test
- Sprite projection: Lode's raycasting camera transform (invDet method)
- Distance fog: walls darken by distance for depth cues (max 75% darkening)
- Delta time: capped at 50ms/frame to prevent physics tunnelling on tab-out
- Pellet collection: checked per-frame at player's current map cell

### Phase 3
- Ghost movement: cell-to-cell (targetCX/targetCY), prevents getting stuck
- prevCX/prevCY tracks last cell — prevents immediate U-turns (ghosts feel more purposeful)
- 3 ghost types: SHADOW_RAPTOR(chase), STONE_WYVERN(patrol/random), BONE_SPECTER(random)
- Boss (level 3): hp=2, scale=1.3, turns pink (#ff00cc) after 1st hit
- Frighten: global STATE.frightenTimer, ghosts slow 50%, run away from player
- Frighten blink: ghosts flash white/blue in last 3 seconds (warns player it's ending)
- Ghost drawing: column-by-column with ZBuffer test; body + eyes + pupils + 3-bump skirt
- Attack: Space key rising-edge only (no auto-repeat on hold), range 1.2–1.5 units
- Kill messages: per ghost type + boss stagger/vanquish messages
- Player invincibility: 2s after being hit, 2s on spawn
- Death flash: red screen overlay fades over 0.4s
- isGhostWalkable: ghosts cannot enter map value 2 (safe rooms)
- BUG NOTED fixed: respawnGhost g.isboss → g.isBoss

### Phase 4
- 5 power-up types: FIRE_BREATH, TAIL_WHIP, FOSSIL_SHIELD, SPEED_BOOST, ANCIENT_RAGE
- Power-ups distributed across maze (1 of each per level), staggered bob cycles
- FIRE_BREATH: Space key kills any ghost within 5 units (5s, no frighten required)
- TAIL_WHIP: instant AOE stun all ghosts within 2 units (2s stun, cell move halted)
- FOSSIL_SHIELD: absorbs next hit (small blue flash confirms shield used), no damage taken
- SPEED_BOOST: 1.5x moveSpeed for 8s (applied in updatePlayer as spdMult)
- ANCIENT_RAGE: frightenTimer=10, same global mechanism as power pellet
- ANCIENT_RAGE timer stays in sync with frightenTimer (updated each frame)
- Power-up sprites: colored square billboard with border highlight, bob animation, 2D label when close
- State tracking: STATE.activePowerUp, STATE.activePowerUpTimer, STATE.shieldActive
- Ghost stun: stunTimer field on ghost, ghosts skip movement while stunTimer > 0
- Respawn on ghost stun resets stunTimer to 0

### Phase 5
- Top bar (26px): score (7-digit zero-padded), level name, colored life squares, pellet count
- Bottom-left (206×78): avatar + HP squares + invincibility bar + power-up name + timer bar
- Avatar: 32×32 canvas-drawn icon per character (rects only, no images)
- FOSSIL_SHIELD timer bar: pulsing solid bar (no countdown, lasts until hit)
- Frighten timer bar: blue sub-bar under top bar, blinks in last 3s (matches ghost blink)
- Minimap (120×120): variable cell size (max(3, floor(120/MAP_W))), whole map visible
- Minimap: walls=dark gray, safe rooms=animated green, open=transparent, power-ups=colored dots
- Minimap: ghosts=red(normal)/blue(frightened) 3px dots, player=white triangle arrow
- SANCTUARY overlay: pulsing green text at H/2+28 when player cell is in safeRoomSet
- ctx.save/restore around renderHUD to prevent textAlign/textBaseline leakage
- Lives displayed as 10×12 colored rectangles (avoids font glyph issues with ♥)

### Phase 6
- Audio: Web Audio API, lazy AudioContext (unlocked on first keydown per browser policy)
- 8 sounds: footstep (70Hz square 0.04s), pellet (880→1200Hz 0.07s), pelletPwr (layered),
  powerup (ascending arpeggio 440/660/880Hz), frighten (800→200 sawtooth 0.5s),
  ghostDefeat (220→600 sawtooth + 900 sine), death (440→55 sawtooth 0.9s),
  levelDone (4-note fanfare), win (5-note ascending)
- Footstep: plays every 0.18s while any forward/back key is held
- Title screen: star field + blinking title + 3 character select boxes with 48px avatars
- drawAvatar now parameterized: drawAvatar(charObj, x, y, sz=32) — scales with r=sz/32
- Character stats shown per box; selected box glows gold with pulse
- LEFT/RIGHT arrows cycle character on title (rising edge), Enter starts game
- AudioContext unlocked on first keydown
- LEVEL_COMPLETE screen: level name cleared, score, next level prompt (or all-done)
- WIN screen: animated color burst, star field, new high score detection
- GAME_OVER screen: blinking red title, score vs high score, enter to retry
- localStorage key: 'dino-dragon-hs'
- updateHighScore() called at: game over, level complete → win, pelletsLeft hits 0
- CRT scanline overlay (rgba 0,0,0,0.07-0.08, every 3px) on all game screens
- pulse moved from renderSprites to main loop (prevents double-increment)
- enterPressed rising-edge cleared after each frame's screen handler uses it
- WIN phase added: all 3 levels cleared, returns to TITLE on Enter
- SANCTUARY overlay: pulsing green text moved into renderHUD (works from renderScene call)

---

## Feature Log 1 — Changelog

### Feature A: Age Selection Screen + Difficulty Scaling
- AGE_SELECT phase added before TITLE; shown once per session, age persists
- DIFF_PARAMS drives ghost speed, safe rooms (1/2/3), ghost count, maze size, PU durations, frighten duration
- AGE_KEYS: ['young','mid','old'] — maps to selectedAge cursor index
- initQuestionQueue() called on age confirm to pre-shuffle the correct question bank

### Feature B: Pellet Progress Bar
- totalPellets snapshotted in initLevel(); bar drains as pelletsLeft decreases
- Color: green >50%, yellow 25-50%, red <25%
- Positioned at y=32 (below top bar + frighten sub-bar), 14px height

### Feature C: ANCIENT EYE Power-Up
- 6th power-up type; PU_TYPES now has 6 entries; bob phase uses TWO_PI/6
- ancientEyeTimer tracked independently (like frightenTimer) — persists if other PU collected on top
- Minimap: pulsing gold dots at pellet cells; power pellet dots brighter/larger
- 3D view: ZBuffer-tested arc sprites for pellets within 10 units of player
- powerUpMult applies to ANCIENT_EYE duration like other timed PUs

### Feature D: Educational Life-Save Modal
- playerHit() now enters LIFE_SAVE phase instead of decrementing lives directly
- Modal appears on EVERY hit; player may lose life or attempt a question
- Question banks: 22 young MC, 22 mid MC, 30 old (12 fill-in + 12 vocab MC + 6 match)
- Fill-in answers: exact case-sensitive match (trains accurate spelling for 11+)
- Match answers: normalized (strip spaces, uppercase) before compare
- keydown handler returns early during LIFE_SAVE to block game input through modal
- chooseLoseLife() / submitMCAnswer() / submitFillAnswer() handle life decrement

### Feature E: Three Camera Perspectives — Edge Cases (logged pre-implementation)
1. TAB disabled during: AGE_SELECT, TITLE, LEVEL_COMPLETE, WIN, GAME_OVER, LIFE_SAVE
2. player.angle persists across switches; used for minimap arrow in all modes
3. Controls identical in all modes: W/S = forward/back (along angle), A/D = turn; no strafe
4. Ghost cell-to-cell movement uses world coords; works identically in all perspectives
5. All timers (frightenTimer, ancientEyeTimer, activePowerUpTimer, playerInvincTimer) persist globally
6. Death flash red overlay rendered in all three view renderers
7. LIFE_SAVE renders the current perspective as frozen background + modal overlay on top
8. Fire breath animation is view-specific: first-person=orange screen flash, top-down=beam line, side-scroll=horizontal projectile
9. Safe rooms: green-tinted in all views
10. Minimap always rendered via renderHUD() call (last, on top), in all views
11. ANCIENT EYE pellets: always visible in top-down/side-scroll; only glow in first-person when active
12. WASD turning (A/D) still updates angle in all views; top-down arrow reflects facing direction
13. Camera snap (no lerp) in side-scroll; clamped to maze bounds to prevent over-scroll
14. For very large mazes (old, level 3: 33×33 cells), top-down cell size adapts (min ~24px)

### Feature E: Three Camera Perspectives — QA Notes
- TAB cycles: first → top → side → first (rising-edge in keydown, only during PLAYING)
- renderCurrentView() helper in loop() dispatches to correct renderer; used by both PLAYING and LIFE_SAVE
- fireProjTimer: set 0.30s in handleAttack on FIRE_BREATH; first-person=screen flash, top-down=beam line, side-scroll=horizontal projectile rect
- Top-down: cs = min(floor(W/MAP_W), floor(H/MAP_H)); maze centered via ox/oy offset; pellets always visible
- Side-scroll: cs=48px, camera snaps to player (no lerp), clamped to [0, MAP*cs-viewport] per axis
- Side-scroll player: white/character-colored rect sprite at screen center; 2-frame walk via floor(pulse*5)%2
- Side-scroll flip: player sprite mirrored via ctx.scale(-1,1) when cos(angle)<0
- Minimap [TAB]1P/TD/SS indicator added to top-left of top bar (5px font)
- perspective persists across level transitions (fireProjTimer reset per level, perspective not reset)

---

## Known Issues / Technical Debt
*(Tracked here as discovered)*
