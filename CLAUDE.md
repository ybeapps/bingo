# CLAUDE.md

## Project

Single-file Hebrew Bingo card generator — everything lives in `index.html`. No build tools, no dependencies.

## Architecture

One HTML file with inline CSS and JS:

- **`words[]`** — global array holding all entered words
- **`generateCards()`** — renders preview cards into `#cards-container` (inside `.container`, hidden on print)
- **`createBingoCard(index, gridSize, hasFreeSpace, cardWidth, cellSize)`** — builds a single `.bingo-card` DOM element; sets `data-grid-size` attribute used by print CSS
- **`preparePrint()`** — populates `#print-container` with 2-cards-per-page layout, then calls `window.print()`
- **`shuffleArray(array)`** — Fisher-Yates shuffle, returns a new array

## Print layout

Two separate containers:

| Container | Visible when |
|---|---|
| `#cards-container` | Screen only |
| `#print-container` | Print only (via `@media print`) |

### Cross-browser print fixes

**Chrome** — `#print-container` is `display:none` at print time; Chrome doesn't re-render hidden elements. Fix: `.pre-print` class positions it off-screen (`position:fixed; left:-99999px; visibility:hidden`) before `window.print()`, forcing layout. `onafterprint` removes the class.

**Chrome border thinning** — `createBingoCard()` applies staggered CSS animations (`j * 0.05s` delay per cell). Cells captured mid-animation render borders at varying sub-pixel weights. Fix: `preparePrint()` strips `animation` from all `.bingo-cell` elements before printing.

**Safari clipping** — `100vh` in `@media print` equals the full page size (e.g. 297mm), not the printable area after margins. Cards centered in 297mm get clipped. Fix: `@page { margin: 15mm }` + `padding: 20mm 0` on `.print-page` instead of `height: 100vh`.

**Safari overflow** — `.bingo-card` has `overflow:hidden` on screen (needed for `border-radius`). Print sets `border-radius:0` but left `overflow:hidden`, clipping the bottom border of the last row. Fix: `overflow:visible` in `@media print`.

### Cell sizing in print

Print CSS uses `data-grid-size` attribute to scale cells so 2 cards fit on one page:

| Grid | Cell size |
|---|---|
| 3×3 | 107px |
| 4×4 | 90px |
| 5×5 | 75px |

`width:auto !important` on `.bingo-card` overrides the JS-set inline width.
