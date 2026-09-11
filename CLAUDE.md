# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build process, no package.json.

## Running the game

There is no build/lint/test tooling. Just open or serve the file:

```bash
open index.html        # macOS — or double-click index.html
python3 -m http.server 8000   # or: npx serve .
```

Then visit `http://localhost:8000` if using a local server.

## Architecture

The whole project is three files that cooperate directly (no modules/bundler):

- `index.html` — DOM structure: the main `#board` canvas (300×600, 10×20 grid at `BLOCK=30`px), a `#next-canvas` preview, HUD spans (`#score`, `#lines`, `#level`), and an `#overlay` used for both Pause and Game Over.
- `style.css` — dark/retro arcade visual theme.
- `game.js` — all game logic, in one file, using global state (no classes/modules).

### Core state and flow (`game.js`)

Global mutable state: `board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, plus loop timing vars (`lastTime`, `dropAccum`, `dropInterval`, `animId`).

- **Board model**: `ROWS × COLS` matrix where each cell is `0` (empty) or a color index `1–7` identifying the locked piece type.
- **Pieces**: the 7 tetrominoes are defined as square matrices in `PIECES`, each cell holding its `COLORS` index. `randomPiece()` picks one and centers it at the top.
- **Rotation** (`rotateCW`): transpose + reverse rows. `tryRotate()` applies rotation then attempts wall kicks at offsets `[0, -1, 1, -2, 2]`, discarding the rotation if none succeed.
- **Collision** (`collide`): true if any occupied cell of a shape is out of bounds or overlaps a locked board cell.
- **Game loop** (`loop`, driven by `requestAnimationFrame`): accumulates elapsed time in `dropAccum`; once it exceeds `dropInterval`, the piece drops one row (or locks via `lockPiece` if blocked).
- **Locking** (`lockPiece` → `merge` + `clearLines` + `spawn`): merges the current piece into `board`, clears completed rows (top-to-bottom sweep with splice/unshift), then spawns the next piece.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level` on line clears; hard drop adds 2 pts/cell dropped, soft drop adds 1 pt/row.
- **Leveling/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level-1) * 90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row; drawn with `globalAlpha = 0.2`.
- **Spawn/Game Over**: `spawn()` promotes `next` to `current` and generates a new `next`; if the newly spawned piece immediately collides, `endGame()` fires and the Game Over overlay is shown.
- **Input** (`keydown` listener): arrows move/soft-drop, Up/X rotates, Space hard-drops, P toggles pause. Input is ignored while `paused` or `gameOver`.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `PIECES`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).
