# DINO-DRAGON QUEST — Project Planning & Architecture Log

## Overview
First-person raycasting Pacman game. Single HTML/CSS/JS file.
Theme: Lost 1994 shareware game aesthetic.
Font: "Press Start 2P" (Google Fonts).

---

## Build Phases
- [x] Phase 1: Raycasting engine + player movement
- [ ] Phase 2: Maze generation + pellet placement
- [ ] Phase 3: Enemy AI + collision
- [ ] Phase 4: Power-ups + combat
- [ ] Phase 5: HUD + minimap
- [ ] Phase 6: Screens + audio (combined to stay within token limits)

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

---

## Known Issues / Technical Debt
*(Tracked here as discovered)*
