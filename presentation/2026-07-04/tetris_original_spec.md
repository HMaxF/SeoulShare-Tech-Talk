# Tetris — Complete Game Specification

## Overview

Stand-alone HTML/CSS/JS Tetris game, no frameworks.

Designed for desktop (keyboard + mouse) and mobile/tablet (touch + on-screen buttons).

---

# Game Control

1. Desktop using Keyboard and Mouse
2. Mobile/Tablet using Touch and On-screen Buttons


## Keyboard control
p/P = Pause or Resume
Left arrow = to left
Right arrow = to right
Down arrow = soft drop
Up arrow = rotate

## Mouse control
Tap = (a quick mouse button click) Rotate
Click and Drag = (a continuous mouse button press) Move to left/right or soft drop.

## Touch control
This is exactly like mouse operation.

Tap = rotate
Drag = move left/right


## On-screen control
There are 5 buttons == HTML <button>

Layout => Left arrow + Rotate + Pause/Resume + Soft Drop + Right Arrow


# Game Area
There are 4 areas ==> using HTML tag <section>

## Section A: Game board
Consisted of 10x20 square tiles
- `touch-action: none` to allow full touch control.

## Section B: Score area
Score + Level + Remaining Lines (eg: 4/10)
- `touch-action: none` to allow full touch control.

## Section C: Next piece preview
Display next pieces
- `touch-action: none` to allow full touch control.
## Section D: Game Control
On-screen buttons ==> HTML tag <button>

5-column CSS grid, 8px gap. Each button:

- Aspect-ratio 1 (square)
- Rounded corners (14px)
- Gradient background (`#2a2a4a` → `#1a1a3a`)
- Box-shadow: outer drop shadow + thin inset top highlight
- Active state: darker gradient, inset shadow, `scale(0.93)`
- `touch-action: manipulation` to prevent browser gesture interference

Layout order: 

⬅️ (left) | ⬆️ (rotate) | ⏸️ (pause) or ▶️ (resume) | ⬇️ (down) | ➡️ (right)


# UI Layout & Responsiveness

Layout change depending on the viewport (browser) size.

Landscape orientation: Width >= Height
Portrait orientation: Width < Height

Listen to window's resize event and change layout accordingly.


## Landscape orientation


- <main> is grid with 2 columns
- Column 1:
    * Game board (Section A)
- Column 2:
    * Score area (Section B)
    * Next pieces preview (Section C) --> horizontally arranged
    * Game Control (Section D)
    * Stacked vertically.

## Portrait orientation

- <main> is Flex
- Row 1:
    * Score area (Section B)
- Row 2: <grid 2 columns> 
    * Game board (Section A)
    * Next pieces preview (Section C) --> vertically arranged.
- Row 3:
    * Game Control (Section D)


# Game logic


## Levels

1. Start at level 1
2. Each level requires **3 lines** to complete (configurable via `requiredLines` constant; designed for 20 in production)
3. On level complete: show overlay, wait for user tap/key. Then increment level, reset lines to 0, clear board, spawn new piece
4. On each level-up a **random white dot** is placed on the board:
   - Quantity: `level-1` dots (level 1 = 0 dots, level 2 = 1 dot, level 3 = 2 dots, etc.)
   - Speed formula per level:
     newSpeed = Math.max(200, 1000 - ((level - 1) * 100))
     Level 1: 1000ms, Level 2: 900ms, Level 3: 800ms, ... minimum 200ms at level 9+
5. `update()` function must recalculate interval each time it's called after a level change. If the speed changes, clear any existing interval and start a new one.


