# Klondike Solitaire — Design Document

## Overview

A browser-based Klondike solitaire game built with Next.js, React 19, and Tailwind CSS 4. Hosted on GitHub Pages. Features four selectable visual themes, light/dark mode, synthesized sound effects, drag-and-drop plus double-click-to-auto-move, unlimited undo, and rotating win celebrations.

## Game Rules

Standard Klondike solitaire:
- 7 tableau columns (1 card in first column, 2 in second, etc.; top card face-up)
- 4 foundation piles, built up by suit from Ace to King
- Stock pile with draw-1 or draw-3 mode (player chooses at game start)
- Tableau builds: descending rank, alternating colors
- Only Kings may fill empty tableau columns
- No scoring — play for fun

## Architecture

### Approach: DOM-based rendering (Option A)

All cards are styled DOM elements. CSS handles animations and theming. No canvas, no game framework, no extra dependencies beyond what's in the project.

### Game Engine

Pure TypeScript reducer, no UI dependencies.

**State:**
- `stock: Card[]` — undealt cards (face down)
- `waste: Card[]` — drawn cards (face up)
- `tableau: Card[7][]` — 7 columns, each card has `faceUp` boolean
- `foundations: Card[4][]` — 4 piles, one per suit
- `drawMode: 1 | 3`
- `history: GameState[]` — previous states for undo

**Actions:** `NEW_GAME`, `DRAW`, `MOVE`, `AUTO_MOVE`, `UNDO`, `AUTO_COMPLETE`

**Validation per move:**
- Tableau: alternating colors, descending rank
- Foundation: same suit, ascending rank (Ace first)
- Empty tableau column: Kings only

**Double-click:** Auto-moves card to best legal destination (foundations first, then tableau).

### Theming

Four themes selectable at any time, each a set of ~15 CSS custom properties:

| Theme | Background | Card back | Surface | Accent |
|-------|-----------|-----------|---------|--------|
| Classic Casino | Deep green felt (CSS gradient) | Ornate red/gold pattern | Green baize | Gold |
| Modern Minimal | Slate gray gradient | Geometric muted blue | Cool gray | Electric blue |
| Cozy Warm | Rich wood grain (CSS gradient) | Burgundy with subtle floral | Warm brown | Amber/copper |
| Playful | Coral-to-violet gradient | Bright multicolor | Soft white/cream | Rainbow |

- Light/dark mode is an additional layer adjusting brightness, contrast, shadows
- Card back designs are CSS-only (gradients + repeating patterns)
- Theme + mode persisted to `localStorage`
- 300ms transition on theme switch

### Card Rendering & Animation

**Cards:**
- DOM elements with front/back faces via CSS 3D transforms
- Suit symbols as text (unicode characters)
- Themed box shadows for depth
- Tableau cards overlap vertically (face-down show less, face-up show more)

**Animations:**
- Deal: cards fly from stock to tableau, staggered ~50ms each
- Move: smooth translate ~200ms ease-out
- Flip: 3D Y-axis rotation ~300ms
- Draw: slide from stock to waste
- Invalid move: snap back with shake
- Auto-complete: rapid cascade to foundations

**Interaction:**
- Drag-and-drop via pointer events (cross-device)
- Dragged cards scale up 1.05x with elevated shadow
- Valid drop zones glow on hover
- Double-click auto-moves to best destination

### Sound Effects

All synthesized via Web Audio API — zero audio file downloads:
- Shuffle: rapid noise bursts (card riffle)
- Deal: "thwip" per card, ascending pitch
- Card place: soft thud
- Card flip: paper-snap
- Draw: swish
- Invalid move: low buzz
- Win: ascending chime melody

Controls: volume slider + mute toggle, persisted to localStorage.

### Win Celebrations

Three animations, rotating 1-2-3-1... across wins. Each runs for 10 seconds.

1. **Cascading Cards:** All 52 cards bounce around with gravity physics, colorful trails, bouncing off edges.
2. **Fireworks/Confetti:** Particle bursts from center, multiple waves, "You Win!" with glow.
3. **Card Fly-Away:** Cards spiral outward in a vortex, fading. "Congratulations" message.

After celebration: "New Game" prompt.

### Layout

- Full viewport height, responsive
- Top row: stock/waste (left), 4 foundations (right)
- Main area: 7 tableau columns, cards fan downward
- Bottom bar: New Game, Undo, Settings

**Settings panel** (slide-out drawer):
- Theme selector with visual thumbnails
- Light/dark toggle
- Volume slider + mute
- Draw mode (1 or 3, only changeable at new game start)

**Responsive:** desktop full spread; tablet compressed; mobile scaled-down cards with tighter overlap.

### Deployment

- Hosted on GitHub Pages from the `ridermw/conductor-playground` repo
- Next.js static export (`output: 'export'` in next.config.ts)
- GitHub Actions workflow builds and deploys on push to main
- Each implementation step is an independent PR merged to main
