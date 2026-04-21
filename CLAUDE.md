# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository

Single-file Tetris game: `tetris.html`. Vanilla HTML + inline CSS + inline JS. No build system, no dependencies, no tests. Open `tetris.html` directly in a browser to run.

UI strings (controls panel, start button, overlay) are in Korean — preserve the language when editing user-facing text.

## Architecture

All game logic lives in the `<script>` block of `tetris.html`. Key structure:

- **Board state**: module-level `let` vars (`board`, `piece`, `next`, `score`, `lines`, `level`, `dropInterval`, `dropCounter`, `lastTime`, `paused`, `over`) mutated in place by the game functions. `start()` resets all of them.
- **Rendering**: two canvases — `#board` (main 10×20 grid, `SIZE=24`px cells) and `#next` (preview). `draw()` / `drawNext()` repaint from scratch each call via `drawCell()`.
- **Game loop**: `loop(time)` runs via `requestAnimationFrame`, accumulates `delta` into `dropCounter`, and calls `drop()` when it exceeds `dropInterval`. Exits when `over` is true.
- **Piece data**: `SHAPES[1..7]` are 2D arrays of color-index ints; `COLORS[1..7]` maps the index to a hex color. `SHAPES[0]` is an empty placeholder so ids line up 1–7.
- **Rotation**: `rotate()` does a pure 90° CW matrix rotation. `tryRotate()` applies it then nudges `piece.x` with an alternating kick sequence (0, -1, +1, -2, +2) — simpler than SRS, gives up after |kick| > 2 and reverts.
- **Line clears**: `clearLines()` scans bottom-up, splices full rows, unshifts empties. Scoring table `[0,100,300,500,800][cleared] * level`; level ticks every 10 lines; `dropInterval` shortens by 80ms per level, floored at 100ms.
- **Input**: single `keydown` listener on `document` handles move/rotate/soft-drop/hard-drop/pause. Pause uses the overlay `<div>`, which is also repurposed for `GAME OVER`.

When editing, keep in mind that `collide()` is the single source of truth for all boundary + overlap checks — move/rotate/drop all rely on it.
