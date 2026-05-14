# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

WordPasta — single-file Hebrew word-search game. RTL UI (`<html lang="he" dir="rtl">`), pink "pasta" theme. Player drags across adjacent letter cells to form words listed under the grid; solved letters vanish from the grid; finishing all answers shows the win screen.

## Run / develop

No build, no deps, no tests. Open `index.html` directly in a browser, or serve the directory (e.g. `python3 -m http.server`) for live reload via your editor of choice. All code is inline in `index.html` — HTML, CSS, and a single IIFE `<script>` at the bottom.

## Architecture

Three screens swapped via `body[data-screen="levels|game|win"]` + CSS — no router, no framework. State lives in module-local vars inside the IIFE (`current`, `selection`, `solved`, `cellOwners`).

Level data model (`LEVELS` array, ~line 261):
- `grid`: 2D array of Hebrew letters, `rows`×`cols`.
- `answers[]`: each answer is defined by a `path` of `[row, col]` coordinates into `grid`; the displayed `word` is derived at load time by walking the path (line 304-306). **Never hand-author `word`** — edit the `path` and let it regenerate.
- Adjacency is 8-directional (Chebyshev distance == 1, see `adjacent()`).

Selection mechanics (`tryAddCell`, ~line 402):
- Pointer events drive a drag-selection across cells via `document.elementFromPoint`.
- Backtracking: dragging onto the second-to-last cell pops the last (lets the user "undo" mid-drag).
- Non-adjacent tap resets the selection to that cell rather than failing.
- On match, the answer's cells are removed only when **no other unsolved answer still needs them** — `cellOwners` is a `Map<"r,c", Set<answerIdx>>` tracking shared letters across overlapping answer paths. This is the non-obvious invariant: cells can belong to multiple answers and only vanish when their owner set empties (line 438-448).

Selection rendering: an overlay `<svg>` inside `.grid-wrap` draws a single stroked `<path>` through the centers of selected cells; stroke width is computed from cell size (`pts[0].w * 0.78`). `updateSelectionPath` recomputes on every selection change and on window resize.

## Adding a level

Append to `LEVELS` with a new `id`, `clue`, `rows`, `cols`, `grid`, and `answers[].path`. The level-select grid auto-populates from `LEVELS.length`. Verify each `path` is contiguous (8-directional) and that every cell in `grid` is reachable by at least one answer — otherwise stray letters remain when the level is solved.
