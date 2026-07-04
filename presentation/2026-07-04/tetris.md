# Tetris — Complete Game Specification

## Overview

Stand-alone HTML/CSS/JS Tetris game, **zero frameworks or dependencies**. All game logic, rendering, sound, and UI are contained in a single `index.html` file.

Designed for desktop (keyboard + mouse) and mobile/tablet (touch + on-screen buttons).

---

## 1. Game Board

- **Grid**: 10 columns × 20 rows.
- **Cell styling**: `border: 1px solid rgba(255,255,255,0.03)`, class `cell`.
- Board background: `#0d1117`, with border and shadow effects.
- The board is a CSS grid inside `<section id="board-section"> <div id="board">`.

---

## 2. Pieces & Randomizer

### 2.1 The 7 Standard Tetrominoes

Each piece has a shape matrix and a CSS `radial-gradient` color:

| Piece | Shape (matrix) | Color Gradient |
|-------|----------------|----------------|
| I (cyan)    | `[[1,1,1,1]]` | `radial-gradient(circle at 30% 30%, #e8ffff 0%, #7df9ff 35%, #00bcd4 100%)` |
| O (yellow)  | `[[1,1],[1,1]]` | `radial-gradient(circle at 30% 30%, #ffffa3 0%, #ffff00 35%, #ffd400 100%)` |
| T (purple)  | `[[0,1,0],[1,1,1]]` | `radial-gradient(circle at 30% 30%, #f3e5f5 0%, #ce93d8 35%, #7b1fa2 100%)` |
| L (orange)  | `[[1,0,0],[1,1,1]]` | `radial-gradient(circle at 30% 30%, #ffe0a3 0%, #ffb347 35%, #ff7a00 100%)` |
| J (blue)    | `[[0,0,1],[1,1,1]]` | `radial-gradient(circle at 30% 30%, #d0eaff 0%, #89cff0 35%, #0033ff 100%)` |
| S (green)   | `[[0,1,1],[1,1,0]]` | `radial-gradient(circle at 30% 30%, #d1ffd1 0%, #8fdd8f 35%, #228b22 100%)` |
| Z (red)     | `[[1,1,0],[0,1,1]]` | `radial-gradient(circle at 30% 30%, #ffe6e6 0%, #ff9494 35%, #d94545 100%)` |

### 2.2 Randomizer

- Uses a **bag system** (shuffle bag). When the bag empties, it is refilled with copies of all 7 shapes.
- **Repeat prevention**: Tracks the last 3 pieces drawn. If the next random candidate matches all of the last 3 (i.e. same piece 4 times in a row), retry up to 5 times.
- The game maintains an array `nextPieces` of **4 upcoming pieces**, pre-filled at game start and replenished one-at-a-time.

---

## 3. Game States & Flow

### 3.1 State Machine

The game tracks these boolean flags:

| Flag | Purpose |
|------|---------|
| `hasStarted` | Whether the player has begun (starts `false`) |
| `paused` | Game is paused |
| `isGameOver` | Game over state |
| `waitingLevel` | Level-complete overlay is shown |
| `clearing` | Line-clear flash animation is running |
| `locking` | Piece lock flash (100ms before merge) |

### 3.2 Flow

1. **Welcome screen** (on load): Shows game title, keyboard controls, "Tap / Press any key to start". `hasStarted=false`.
2. **Start**: On first keydown/tap/click on overlay, calls `startGame()` → `initGame()` → `newPiece()`, starts the gravity interval, plays `start` sound, hides overlay. `hasStarted=true`.
3. **Playing**: Gravity ticks, input handled.
4. **Pause**: Press P or pause button → overlay with "Paused", `paused=true`.
5. **Level complete**: When `levelLines >= requiredLines` → overlay "Level N Complete!", `waitingLevel=true`. Tap/any key to proceed.
6. **Game over**: When a new piece collides immediately upon spawning → overlay "Game Over", `isGameOver=true`. Tap/any key → back to welcome screen.

---

## 4. Input Controls

### 4.1 Keyboard

| Key | Action | Conditions |
|-----|--------|------------|
| `ArrowLeft` | Move piece left by 1 column | Not paused, not game over, not waiting, not clearing, not locking |
| `ArrowRight` | Move piece right by 1 column | Same |
| `ArrowDown` | Soft drop (move down 1 row) | Same |
| `ArrowUp` | Rotate piece clockwise | Same |
| `Space` | Hard drop (instant drop to bottom + lock) | Same |
| `p` / `P` | Toggle pause | Not waiting, not game over |

### 4.2 Mouse / Pointer

Uses **Pointer Events** (`pointerdown`, `pointermove`, `pointerup`, `pointercancel`).

- **Tap** (pointerdown + pointerup at same location, < 5px movement): Rotate piece.
- **Horizontal drag** (pointermove, accumulated `touchAccumX` tracked): Move piece 1 column per tile-width of drag. Direction determined by sign of delta.
- **Vertical drag** (pointermove, accumulated `touchAccumY` tracked): Soft drop 1 row per tile-height of drag.
- **Upward swipe** (pointerup, dy < 0 and |dy| > |dx| + 10): Rotate piece.

All pointer input is ignored on buttons, controls section, and overlay.

### 4.3 Touch (Mobile)

Basic `touchstart` / `touchmove` listeners with `passive: false` and `preventDefault()` prevent browser gestures. Same logic as pointer events above.

The game also uses pointer events for touch (unified input model). Touch is handled as pointer events. Additional touch listeners with `preventDefault()` block browser scrolling/zooming on the board area.

### 4.4 On-Screen Buttons

5 square buttons in `<section id="controls-section">`:

| Button | Symbol | Action | Hold-to-Repeat |
|--------|--------|--------|----------------|
| Left | ⬅️ | `move(-1)` | Yes (delay 250ms, repeat 120ms) |
| Rotate | ⬆️ | `rotate()` | Yes (delay 400ms, repeat 120ms) |
| Pause | ⏸️ / ▶️ | `togglePause()` | No (single tap) |
| Down | ⬇️ | `handleSoftDrop()` | Yes (delay 250ms, repeat 120ms) |
| Right | ➡️ | `move(1)` | Yes (delay 250ms, repeat 120ms) |

**Button styling**:
- Aspect-ratio 1 (square)
- Rounded corners 14px
- Gradient background: `linear-gradient(145deg, #2a2a4a, #1a1a3a)`
- Hover: `box-shadow: 0 4px 12px rgba(0,0,0,0.4), inset 0 1px 0 rgba(255,255,255,0.1)`
- Active: `background: linear-gradient(145deg, #1a1a3a, #0f0f2a)`, `box-shadow: 0 1px 4px rgba(0,0,0,0.5), inset 0 2px 6px rgba(0,0,0,0.4)`, `transform: scale(0.93)`
- `touch-action: manipulation` to prevent browser gesture interference.

**Hold-to-repeat mechanism (`holdStart` / `holdStop`)**:
- On mousedown/touchstart: execute action immediately, then start a timeout.
- After initial delay (250ms default, 400ms for rotate), repeats every 120ms.
- On mouseup/touchend/touchcancel/mouseleave: clear timeout.
- Debounce: ignores calls within 50ms of last call.

---

## 5. Scoring

| Event | Points |
|-------|--------|
| Piece lock (land & merge) | +1 |
| 1 line cleared | +5 |
| 2 lines cleared | +15 |
| 3 lines cleared | +30 |
| 4 lines cleared (Tetris) | +50 |

---

## 6. Levels & Difficulty

### 6.1 Level Progression

- Start at **level 1**.
- Each level requires **3 lines** cleared (`requiredLines = 3`, configurable constant; designed for 20 in production).
- When `levelLines >= requiredLines`:
  1. Play `levelup` sound.
  2. Show overlay "Level N Complete!".
  3. Wait for user tap/key.
  4. Increment level, reset `levelLines` to 0.
  5. **Clear the entire board**.
  6. Add **random white dots** (placed in rows 10-19, 20 attempts per dot) on empty cells: `level - 1` dots (level 2 = 1 dot, level 3 = 2 dots, etc.).
  7. Spawn new piece and continue.

### 6.2 Speed Formula

```
newSpeed = Math.max(200, 1000 - ((level - 1) * 100))
```

| Level | Speed (ms per tick) |
|-------|---------------------|
| 1     | 1000 |
| 2     | 900  |
| 3     | 800  |
| 4     | 700  |
| 5     | 600  |
| 6     | 500  |
| 7     | 400  |
| 8     | 300  |
| 9+    | 200 (minimum) |

When speed changes, the old interval is cleared and a new one is set in `update()`.

---

## 7. Game Mechanics (Detailed)

### 7.1 Gravity Tick (`tick()`)

1. If any blocking state (paused, waiting level, game over, clearing, locking) → return.
2. If no collision below → move piece down by 1.
3. If collision below → **lock sequence**:
   - Set `locking = true`.
   - Play `land` sound.
   - Draw piece in **white** (lock flash) for 100ms.
   - After 100ms: set `locking = false`, merge piece into board, add +1 score, check for line clears.
   - If lines found → start clear animation.
   - If no lines → check level up, spawn new piece, check game over.

### 7.2 Collision Detection (`collide(nx, ny, pieceShape)`)

Returns `true` if placing the shape at `(nx, ny)` would:
- Go out of bounds (x < 0 or x >= COLS or y >= ROWS).
- Overlap a filled cell on the board (for y >= 0).

Note: pieces can start partially above the board (y < 0 is allowed without collision).

### 7.3 Merge (`merge()`)

Copies the piece's color into the board array cells corresponding to its shape matrix.

### 7.4 Hard Drop (`drop()`)

Moves the piece down until collision, then immediately calls `tick()` to trigger lock sequence.

### 7.5 Soft Drop (`handleSoftDrop()`)

Moves piece down 1 row if no collision below.

### 7.6 Rotation (`rotate()`)

1. Computes rotated matrix: `r = s[0].map((_,i) => s.map(row => row[i]).reverse())` (clockwise rotation).
2. If shape is symmetric (rotated equals original), only test at current position → apply if no collision.
3. Otherwise, build a list of **wall kick offsets**:
   - If rotation would overhang right edge → negative kicks (and positive fallbacks).
   - If rotation would overhang left edge → positive kicks (and negative fallbacks).
   - Otherwise → try -1, -2, -3, +1, +2, +3.
   - Deduplicate the kick list.
4. Try each kick offset; first non-colliding position wins → apply shape and offset.

### 7.7 Line Clear (`clearLines()`)

Collects all row indices where every cell is non-null.

### 7.8 Clear Animation (`startClearAnimation()`)

- `clearing = true`, `clearingFlashOn` toggles every 80ms.
- Draws flash frames: white cells on clearing rows when `clearingFlashOn`, normal otherwise.
- After 6 flashes (480ms total) → `finishClearAnimation(numCleared)`.

### 7.9 Finish Clear (`finishClearAnimation()`)

1. Remove cleared rows (sorted descending) via `splice`.
2. Add empty rows at top to restore 20 rows.
3. Update score, play appropriate `clearN` sound.
4. Check level up.
5. If not waiting for level: spawn new piece, check game over.
6. `update()`.

---

## 8. Sound Effects

Uses **Web Audio API** (`AudioContext` / `webkitAudioContext`). Created lazily on first interaction (resume/init requirement).

### Sound Table

| Sound Name | Waveform | Frequency Pattern | Duration | Gain |
|-----------|----------|-------------------|----------|------|
| `start` | square | 400Hz → 800Hz (exponential ramp) | 0.25s | 0.15 → 0.001 |
| `land` | triangle | 600Hz → 200Hz (exponential ramp) | 0.12s | 0.25 → 0.001 |
| `pause` | sine | 600Hz | 0.10s | 0.20 → 0.001 |
| `resume` | sine | 800Hz | 0.12s | 0.20 → 0.001 |
| `clear1` | sine | 523Hz (C5) | 0.15s | 0.25 → 0.001 |
| `clear2` | triangle | 523Hz → 659Hz (step at 0.1s) | 0.30s | 0.25 → 0.001 |
| `clear3` | triangle | 523Hz → 659Hz → 784Hz (steps at 0.08s, 0.16s) | 0.40s | 0.25 → 0.001 |
| `clear4` | sine | 523Hz → 659Hz → 784Hz → 1047Hz (steps at 0.08s) | 0.60s | 0.35 → 0.001 |
| `levelup` | sine | 523Hz → 659Hz → 784Hz → 1047Hz (steps at 0.1s, 0.2s, 0.35s) | 0.70s | 0.30 (0.20 at 0.35s) → 0.001 |
| `gameover` | sawtooth | 400Hz → 60Hz (exponential ramp) | 1.50s | 0.20 (0.15 at 0.3s) → 0.001 |

Implementation note: Each sound creates a new `OscillatorNode` + `GainNode`, connects, schedules frequency/gain values using `setValueAtTime` and `exponentialRampToValueAtTime`, then starts and stops.

---

## 9. Visual Effects

### 9.1 Lock Flash
- When piece lands (before merge), piece renders in **white** for 100ms (`LOCK_FLASH_MS`).
- During this time, `locking = true` and user input is blocked.

### 9.2 Line Clear Flash
- Clearing rows flash **white** at 80ms interval for 6 toggles (3 on/off cycles).
- During this time, `clearing = true` and user input is blocked.

### 9.3 Piece Colors
All pieces use rich `radial-gradient(circle at 30% 30%, ...)` CSS backgrounds for a 3D/glossy look (see piece table above).

### 9.4 Board & Cell Styling
- Board: `background: #0d1117`, `border: 2px solid rgba(255,255,255,0.1)`, `border-radius: 3px`, `box-shadow` outer + inner.
- Cells: `border: 1px solid rgba(255,255,255,0.03)`.

---

## 10. UI Layout & Responsiveness

### 10.1 Overall Structure

```
<main id="game-main">  (CSS Grid)
  <section id="score-section">   — Score | Level | Lines
  <section id="board-section">   — 10×20 game board
  <section id="next-section">    — "Next" heading + 4 preview grids
  <section id="controls-section"> — 5 buttons
</main>
```

### 10.2 Portrait (default, width < height)

CSS grid: `2 columns × 3 rows`.

```
┌───────────────────┬────────────┐
│     SCORE         │            │  (spanning both columns)
├───────────────────┤  NEXT (4)  │
│     BOARD         │  (vertical)│
│     (10×20)       │            │
├───────────────────┴────────────┤
│          CONTROLS (5 buttons)  │  (spanning both columns)
└────────────────────────────────┘
```

- `grid-template-columns: 2fr 1fr`
- `grid-template-rows: 1fr 8fr 1fr`
- Areas: `score score / board next / controls controls`

### 10.3 Landscape (width >= height, class `.landscape` added to `<main>`)

CSS grid: `2 columns × 3 rows`.

```
┌──────────────┬─────────────────┐
│              │     SCORE       │
│    BOARD     ├─────────────────┤
│   (10×20)    │  NEXT (4 pieces)│
│              │  (horizontal)   │
│              ├─────────────────┤
│              │   CONTROLS      │
└──────────────┴─────────────────┘
```

- `grid-template-columns: 1fr 1fr`
- `grid-template-rows: 1fr 8fr 1fr`
- Areas: `board score / board next / board controls`
- Next pieces: `.landscape #next-section` uses `flex-direction: column`, preview width uses `clamp(10vh, 12vh, 15vh)`.

### 10.4 Score Section

- 3 stats in a horizontal flex row: **Score** (text-shadow gold `#ffd700`), **Level** (cyan `#7df9ff`), **Lines** (green `#7dff9e`).
- Each stat is a card with `rgba(255,255,255,0.06)` background, 10px border-radius, blur backdrop.
- Labels: uppercase, `clamp(8px, 2vw, 12px)`, color `rgba(255,255,255,0.5)`.
- Values: `clamp(16px, 4vw, 28px)`, weight 800, white with text-shadow.
- Lines display: `"{levelLines} / {requiredLines}"`.

### 10.5 Next Piece Preview

- "Next" heading: uppercase, `clamp(10px, 2.5vw, 16px)`, color `rgba(255,255,255,0.4)`.
- 4 preview grids, each a 4×4 CSS grid (`grid-template-columns: repeat(4, 1fr); grid-template-rows: repeat(4, 1fr)`).
- Preview size: `clamp(10vw, 15vw, 20vw)` in portrait, `clamp(10vh, 12vh, 15vh)` in landscape.
- Cells get the piece's gradient color where shape matrix is 1.

### 10.6 Controls Section

- 5-column CSS grid (`grid-template-columns: repeat(5, 1fr)`), `gap: 8px`.
- `touch-action: manipulation` on the section.
- Background gradient top-to-bottom similar to score section.

### 10.7 Overlay (Modal)

- Fixed position, full-screen, `z-index: 10`.
- Semi-transparent black background (`rgba(0,0,0,0.6)`) with `backdrop-filter: blur(8px)`.
- Centered card: gradient background, 20px border-radius, max-width 380px, 88% width on small screens.
- Title: gold/orange gradient text, `clamp(24px, 5vw, 36px)`, weight 800.
- Controls list (welcome screen): `clamp(13px, 2.5vw, 16px)`, line-height 2.2.
- Tap hint: smaller, uppercase, low opacity.

### 10.8 Resize Handling

- `window.addEventListener("resize", applyLayout)`.
- `applyLayout()`: if `innerWidth >= innerHeight` → add `landscape` class, else remove it.
- CSS media queries are not used; layout is toggled via class.

---

## 11. Responsive Sizing

- `body`: `height: 100dvh`, `overflow: hidden`, `user-select: none`, `touch-action: none`.
- `#game-main`: `width: 100dvw`, `height: 100dvh`.
- Board: `height: 100%`, `max-width: 100%`, `max-height: 100%`, `aspect-ratio: 10 / 20`, CSS grid with 10×20 cells.
- All font sizes use `clamp()` for fluid scaling.
- Preview grids use `clamp()` with `vw` and `vh` units.

---

## 12. Code Structure & Constants

```javascript
const COLS = 10;
const ROWS = 20;
const SHAPES = [ /* 7 pieces */ ];
const requiredLines = 3;    // lines needed per level (configurable)
const startSpeed = 1000;    // ms per gravity tick at level 1
const LOCK_FLASH_MS = 100;  // ms for lock flash animation
const CLEARING_FLASHES = 6; // number of flash toggles during line clear
const CLEARING_INTERVAL = 80; // ms between flash toggles
```

Key global variables:
- `board[][]` — 20×10 grid of color strings or `null`.
- `piece` — current active piece `{ shape, color }`.
- `nextPieces[]` — array of 4 upcoming pieces.
- `pos` — `{ x, y }` position of current piece.
- `score`, `level`, `levelLines` — current game state.
- `speed`, `interval` — current tick speed (ms) and interval ID.
- `bag[]` — shuffled bag of pieces.
- `lastPieces[]` — last 3 drawn pieces for repeat prevention.

---

## 13. DOM Reference

| ID | Element | Purpose |
|----|---------|---------|
| `#game-main` | `<main>` | Top-level grid container |
| `#board` | `<div>` | 10×20 CSS grid of `.cell` divs |
| `#score` | `<div>` | Score display |
| `#level` | `<div>` | Level display |
| `#lines` | `<div>` | Lines display (`{current}/{required}`) |
| `#next0`–`#next3` | `<div class="preview">` | 4×4 CSS grids for next pieces |
| `#btn-left` | `<button>` | Move left |
| `#btn-rotate` | `<button>` | Rotate |
| `#btn-pause` | `<button>` | Pause/Resume toggle |
| `#btn-down` | `<button>` | Soft drop |
| `#btn-right` | `<button>` | Move right |
| `#overlay` | `<div class="overlay">` | Full-screen modal overlay |
| `#overlay-content` | `<div class="overlay-card">` | Inner card for overlay text |

---

## 14. Initialization

On `window.load`:
1. Call `applyLayout()` to set initial orientation class.
2. Call `showWelcome()` which calls `initGame()` and renders the welcome overlay.

`initGame()`:
- Reset score to 0, level to 1, levelLines to 0.
- Reset all state flags.
- Clear bag, lastPieces, nextPieces.
- Call `resetBoard()` (fills board with null).
- Call `newPiece()`.
