# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A vanilla-JS Tetris game (HTML5 Canvas + CSS). No build step, no dependencies, no `package.json`, no test suite. Three files: `index.html`, `style.css`, `game.js`.

## Running

Open `index.html` directly, or serve statically and open the URL:

```bash
python3 -m http.server 8000   # or: npx serve .
```

There is nothing to build, lint, or test — changes to `game.js`/`style.css` are picked up on browser reload.

## Architecture (game.js)

All game logic lives in `game.js` as module-level functions sharing mutable `let` globals (`board`, `current`, `next`, `score`, etc.). Key conventions a new change must respect:

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7`. The same index drives both collision and rendering color via `COLORS[index]`.
- **Pieces as colored matrices**: each entry in `PIECES` is a square matrix whose non-zero cells already hold that piece's color index (e.g. the T piece uses `3`). So a piece's shape and its color are the same data — don't store color separately.
- **Rotation**: `rotateCW` transposes + reverses; `tryRotate` applies wall kicks by trying x-offsets `[0,-1,1,-2,2]` and committing the first that doesn't `collide`.
- **`collide(shape, ox, oy)`** is the single source of truth for legality (walls, floor, occupied cells). Every move/rotate/drop checks it before mutating `current`.
- **Game loop**: `requestAnimationFrame(loop)` accumulates `dt` into `dropAccum` and steps the piece down once `dropAccum >= dropInterval`. Pausing/game-over `cancelAnimationFrame(animId)`; `togglePause` restarts the loop and must reset `lastTime` to avoid a `dt` spike.
- **Lock cycle**: `lockPiece` → `merge` (write piece into board) → `clearLines` → `spawn`. `spawn` moves `next` into `current`, rolls a new `next`, and calls `endGame` if the fresh piece already collides.
- **Scoring/level coupling**: `clearLines` updates `score` (via `LINE_SCORES * level`), `lines`, `level` (`floor(lines/10)+1`), and recomputes `dropInterval` together. Keep these in sync if you touch difficulty.

## DOM coupling

`game.js` reads fixed element IDs from `index.html` (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`). Renaming any requires editing both files.

The board canvas is sized in `index.html` as `width=COLS*BLOCK` (300) `height=ROWS*BLOCK` (600). If you change `COLS`, `ROWS`, or `BLOCK` in `game.js`, update the `<canvas id="board">` attributes to match or rendering will clip/scale.
