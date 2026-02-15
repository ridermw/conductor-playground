# Klondike Solitaire Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a fully playable Klondike solitaire game with 4 selectable themes, light/dark mode, sound effects, drag-and-drop, undo, and rotating win celebrations, hosted on GitHub Pages.

**Architecture:** Single-page Next.js app with static export. Pure TypeScript game engine (useReducer), DOM-based card rendering with CSS animations, Web Audio API for sounds, CSS custom properties for theming. No external dependencies beyond React/Next.js/Tailwind.

**Tech Stack:** Next.js 16, React 19, TypeScript 5, Tailwind CSS 4

**Repo:** `ridermw/conductor-playground` on GitHub. Each task below is an independent PR to `main`.

**GitHub Pages URL:** `https://ridermw.github.io/conductor-playground/`

---

## PR 1: GitHub Pages Deployment Pipeline

**Goal:** Configure Next.js for static export and deploy to GitHub Pages via GitHub Actions.

**Branch:** `ridermw/pr1-gh-pages-deploy`

**Files:**
- Modify: `next.config.ts`
- Modify: `src/app/layout.tsx` (update metadata)
- Modify: `src/app/page.tsx` (replace boilerplate with placeholder)
- Create: `.github/workflows/deploy.yml`

### Step 1: Configure Next.js for static export

Modify `next.config.ts`:

```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  output: "export",
  basePath: "/conductor-playground",
  images: {
    unoptimized: true,
  },
};

export default nextConfig;
```

### Step 2: Update layout metadata

Modify `src/app/layout.tsx` — change the metadata:

```typescript
export const metadata: Metadata = {
  title: "Solitaire",
  description: "Klondike solitaire card game",
};
```

### Step 3: Replace page.tsx with placeholder

Replace `src/app/page.tsx` with:

```tsx
export default function Home() {
  return (
    <div className="flex min-h-screen items-center justify-center bg-green-900">
      <h1 className="text-4xl font-bold text-white">Solitaire — Coming Soon</h1>
    </div>
  );
}
```

### Step 4: Create GitHub Actions deploy workflow

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with:
          version: 9
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: out

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

### Step 5: Verify build works locally

Run: `pnpm build`
Expected: Successful static export to `out/` directory.

### Step 6: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr1-gh-pages-deploy
git add next.config.ts src/app/layout.tsx src/app/page.tsx .github/workflows/deploy.yml
git commit -m "Configure GitHub Pages deployment with static export"
```

Create PR to `main`, merge it. Verify GitHub Pages deploys at `https://ridermw.github.io/conductor-playground/`.

---

## PR 2: Game Engine — Types, State, and Rules

**Goal:** Build the pure TypeScript game engine with all Klondike rules, shuffling, dealing, and undo. No UI in this PR.

**Branch:** `ridermw/pr2-game-engine`

**Files:**
- Create: `src/game/types.ts`
- Create: `src/game/deck.ts`
- Create: `src/game/rules.ts`
- Create: `src/game/reducer.ts`

### Step 1: Define types

Create `src/game/types.ts`:

```typescript
export type Suit = "spades" | "hearts" | "diamonds" | "clubs";
export type Rank = 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13;

export interface Card {
  suit: Suit;
  rank: Rank;
  faceUp: boolean;
  id: string; // e.g. "hearts-7" — unique identifier for React keys and drag tracking
}

export type PileType = "stock" | "waste" | "tableau" | "foundation";

export interface Location {
  pile: PileType;
  index: number; // which pile (0-6 for tableau, 0-3 for foundation, 0 for stock/waste)
  position?: number; // card position within pile (for grabbing stacks from tableau)
}

export type DrawMode = 1 | 3;

export interface GameState {
  stock: Card[];
  waste: Card[];
  tableau: Card[][]; // 7 columns
  foundations: Card[][]; // 4 piles (spades, hearts, diamonds, clubs)
  drawMode: DrawMode;
  history: GameState[]; // previous states for undo (stored without their own history to avoid deep nesting)
  gameWon: boolean;
  winCount: number; // tracks wins for rotating celebrations
}

export type GameAction =
  | { type: "NEW_GAME"; drawMode: DrawMode }
  | { type: "DRAW" } // flip card(s) from stock to waste
  | { type: "MOVE"; from: Location; to: Location }
  | { type: "AUTO_MOVE"; cardId: string } // double-click auto-move
  | { type: "UNDO" }
  | { type: "AUTO_COMPLETE" };
```

### Step 2: Create deck utilities

Create `src/game/deck.ts`:

```typescript
import { Card, Suit, Rank } from "./types";

const SUITS: Suit[] = ["spades", "hearts", "diamonds", "clubs"];
const RANKS: Rank[] = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13];

export function createDeck(): Card[] {
  const cards: Card[] = [];
  for (const suit of SUITS) {
    for (const rank of RANKS) {
      cards.push({ suit, rank, faceUp: false, id: `${suit}-${rank}` });
    }
  }
  return cards;
}

/** Fisher-Yates shuffle */
export function shuffle(cards: Card[]): Card[] {
  const shuffled = [...cards];
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
  }
  return shuffled;
}

export function isRed(suit: Suit): boolean {
  return suit === "hearts" || suit === "diamonds";
}

export function isBlack(suit: Suit): boolean {
  return suit === "spades" || suit === "clubs";
}

export function oppositeColor(a: Suit, b: Suit): boolean {
  return isRed(a) !== isRed(b);
}

export const RANK_DISPLAY: Record<Rank, string> = {
  1: "A", 2: "2", 3: "3", 4: "4", 5: "5", 6: "6", 7: "7",
  8: "8", 9: "9", 10: "10", 11: "J", 12: "Q", 13: "K",
};

export const SUIT_SYMBOL: Record<Suit, string> = {
  spades: "\u2660", hearts: "\u2665", diamonds: "\u2666", clubs: "\u2663",
};

/** Foundation suit order — index matches foundations array */
export const FOUNDATION_SUITS: Suit[] = ["spades", "hearts", "diamonds", "clubs"];
```

### Step 3: Create rules module

Create `src/game/rules.ts`:

Implements:
- `canMoveToTableau(card: Card, targetColumn: Card[]): boolean` — checks alternating color, descending rank, Kings on empty
- `canMoveToFoundation(card: Card, targetFoundation: Card[]): boolean` — checks same suit, ascending rank, Ace on empty
- `findAutoMoveTarget(card: Card, state: GameState): Location | null` — finds best legal destination (foundations first)
- `isGameWon(state: GameState): boolean` — all 4 foundations have 13 cards
- `canAutoComplete(state: GameState): boolean` — all cards are face-up (stock and waste empty, all tableau cards face-up)
- `getMovableCards(from: Location, state: GameState): Card[]` — returns the card(s) that would be moved (single card or stack from tableau)

### Step 4: Create game reducer

Create `src/game/reducer.ts`:

Implements `gameReducer(state: GameState, action: GameAction): GameState`:

- **NEW_GAME:** Creates shuffled deck, deals 28 cards to tableau (1,2,3,4,5,6,7 with top face-up), remaining 24 to stock. Resets history. Preserves winCount.
- **DRAW:** If stock has cards, move drawMode cards to waste (face-up). If stock empty, recycle waste back to stock (face-down). Pushes previous state to history.
- **MOVE:** Validates move using rules, executes if valid, auto-flips newly exposed tableau cards. Pushes previous state to history. Checks for win.
- **AUTO_MOVE:** Finds card by ID, determines its location, calls findAutoMoveTarget, executes MOVE if found.
- **UNDO:** Pops most recent state from history. Returns it (with current history intact).
- **AUTO_COMPLETE:** Repeatedly moves the lowest available card from tableau/waste to foundations until no more moves possible.

Helper: `createInitialState(drawMode: DrawMode): GameState` — called by NEW_GAME.

### Step 5: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr2-game-engine
git add src/game/
git commit -m "Add Klondike game engine with types, rules, and reducer"
```

Create PR to `main` with description covering the engine architecture.

---

## PR 3: Board Layout and Card Rendering

**Goal:** Render the game board with all card piles and a playable dealt game (static — no drag/drop yet, just visual).

**Branch:** `ridermw/pr3-board-layout`

**Files:**
- Create: `src/components/Card.tsx` — single card component with front/back faces
- Create: `src/components/Pile.tsx` — renders a stack of cards (handles overlap for tableau)
- Create: `src/components/Board.tsx` — full game board layout
- Create: `src/components/GameProvider.tsx` — React context wrapping useReducer
- Modify: `src/app/page.tsx` — render Board
- Modify: `src/app/globals.css` — add card styles and CSS custom properties

### Step 1: Create GameProvider

Create `src/components/GameProvider.tsx`:

React context that:
- Wraps `useReducer(gameReducer, initialState)` where initial state is a new game with drawMode 1
- Exposes `state` and `dispatch` via context
- Provides `useGame()` hook

### Step 2: Create Card component

Create `src/components/Card.tsx`:

A card element ~70px wide by ~100px tall (scaled via CSS). Two faces:
- **Front:** rank in top-left and bottom-right corners, suit symbol, colored red or black
- **Back:** themed pattern (use a default green/gold pattern for now, theming comes in PR 6)
- CSS 3D transform: `rotateY(180deg)` toggles face. `transform-style: preserve-3d`, `backface-visibility: hidden` on each face
- Card has `data-card-id` attribute for drag tracking
- Accepts `onClick` and `onDoubleClick` props
- Accepts `style` prop for positioning (absolute positioning within piles)

### Step 3: Create Pile component

Create `src/components/Pile.tsx`:

Renders a stack of Card components:
- **Stock:** shows top card face-down, or empty placeholder. Click draws.
- **Waste:** shows top card(s) face-up (draw-3 shows top 3 fanned right).
- **Foundation:** shows top card or empty placeholder with suit symbol.
- **Tableau:** cards overlap vertically. Face-down cards offset ~15px, face-up offset ~25px. Uses absolute positioning.

### Step 4: Create Board component

Create `src/components/Board.tsx`:

Full-viewport layout:
- CSS Grid or Flexbox layout
- **Top row:** Stock pile, Waste pile, spacer, 4 Foundation piles
- **Main area:** 7 Tableau columns evenly spaced
- **Bottom bar:** New Game button, Undo button (no Settings yet — that's PR 6)
- Buttons dispatch NEW_GAME and UNDO actions
- Initial game mode selector: before first game, show draw-1 vs draw-3 choice

### Step 5: Wire up page.tsx

Replace `src/app/page.tsx`:

```tsx
import { GameProvider } from "@/components/GameProvider";
import { Board } from "@/components/Board";

export default function Home() {
  return (
    <GameProvider>
      <Board />
    </GameProvider>
  );
}
```

### Step 6: Add base card CSS

Add to `src/app/globals.css`:
- Card dimensions as CSS vars (`--card-width: 70px`, `--card-height: 100px`)
- 3D perspective on board container (`perspective: 1000px`)
- Card face/back base styles
- Empty pile placeholder styles (dashed border, subtle)
- Default background: dark green (`#1a472a`)

### Step 7: Verify visual rendering

Run: `pnpm dev`
Verify: Board shows dealt game with 7 tableau columns, stock, waste, foundations. Cards render with correct ranks/suits. Click stock to draw. Click "New Game" to redeal.

### Step 8: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr3-board-layout
git add src/components/ src/app/page.tsx src/app/globals.css
git commit -m "Add board layout with card rendering and game context"
```

---

## PR 4: Card Interaction — Drag-and-Drop + Double-Click

**Goal:** Make the game fully playable with drag-and-drop card movement and double-click auto-move.

**Branch:** `ridermw/pr4-card-interaction`

**Files:**
- Create: `src/hooks/useDragAndDrop.ts` — pointer event handling for drag-and-drop
- Modify: `src/components/Card.tsx` — attach drag handlers
- Modify: `src/components/Pile.tsx` — drop zone logic
- Modify: `src/components/Board.tsx` — drag overlay layer

### Step 1: Create useDragAndDrop hook

Create `src/hooks/useDragAndDrop.ts`:

Custom hook that manages:
- **Drag state:** `{ isDragging, dragCards, dragOrigin, position }` or null
- **pointerdown** on a card: records origin location, starts tracking pointer position
- **pointermove:** updates position of dragged card overlay (uses `requestAnimationFrame` for smooth tracking)
- **pointerup:** checks if pointer is over a valid drop zone. If valid, dispatches MOVE action. If invalid, cancels (card snaps back).
- Minimum drag distance threshold (5px) to distinguish click from drag
- Captures pointer via `setPointerCapture` for reliable tracking
- For tableau, allows grabbing a stack (the clicked card plus all cards below it)

### Step 2: Add drop zone detection

In Pile components, add `data-drop-zone` attributes with pile type and index. On pointerup, use `document.elementsFromPoint()` to find the drop target under the cursor.

### Step 3: Add double-click auto-move

On Card's `onDoubleClick`, dispatch `AUTO_MOVE` action with the card's ID. The reducer finds the best destination automatically.

### Step 4: Add drag overlay to Board

In Board, render a floating layer for the dragged card(s):
- Position: fixed, follows pointer
- Scale: 1.05x with elevated shadow
- z-index above everything
- Contains the Card component(s) being dragged

### Step 5: Add visual feedback for valid drop zones

When dragging, highlight valid drop zones:
- Calculate which piles the dragged card(s) can legally go to
- Add a subtle glow/border highlight to those piles
- Remove highlight when drag ends

### Step 6: Handle stock pile click

Click on stock pile dispatches DRAW action. Click on empty stock recycles waste.

### Step 7: Verify full playability

Run: `pnpm dev`
Test:
- Drag cards between tableau columns (alternating colors, descending rank)
- Drag Aces to foundations, build up by suit
- Double-click to auto-move cards
- Draw from stock (draw-1 and draw-3)
- Undo moves
- Play to completion

### Step 8: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr4-card-interaction
git add src/hooks/ src/components/
git commit -m "Add drag-and-drop and double-click card interaction"
```

---

## PR 5: Card Animations

**Goal:** Add smooth animations for dealing, flipping, moving, and invalid moves.

**Branch:** `ridermw/pr5-animations`

**Files:**
- Create: `src/hooks/useAnimations.ts` — animation state management
- Modify: `src/components/Card.tsx` — CSS transitions for flip
- Modify: `src/components/Board.tsx` — deal animation orchestration
- Modify: `src/app/globals.css` — keyframes and transition classes

### Step 1: Add CSS keyframes and transitions

Add to `src/app/globals.css`:
- `@keyframes shake` — left-right shake for invalid moves (100ms, 3 cycles)
- `@keyframes deal-fly` — card flies from stock position to target (300ms ease-out)
- Card flip transition: `transition: transform 300ms ease-in-out` on the card container
- Card move transition: `transition: top 200ms ease-out, left 200ms ease-out`

### Step 2: Card flip animation

In Card component:
- When `faceUp` changes, trigger 3D Y-axis rotation via CSS class toggle
- Card container has `transform-style: preserve-3d`
- Front face: `rotateY(0deg)`, Back face: `rotateY(180deg)`
- When face-down: container has `rotateY(180deg)`
- The CSS transition handles the smooth rotation

### Step 3: Deal animation

When a new game starts:
- Cards start stacked at the stock position
- Animate each card flying to its tableau position with staggered timing (~50ms between cards)
- Use `useAnimations` hook to track deal-in-progress state (block interaction during deal)
- Total deal time: ~28 cards * 50ms = ~1.4 seconds

### Step 4: Move animation

When a card moves between piles:
- Briefly animate from old position to new position using CSS transitions
- Card smoothly slides to new location (~200ms)

### Step 5: Invalid move shake

When a drag-and-drop lands on an invalid target:
- Card snaps back to original position
- Apply shake animation class, remove after animation ends

### Step 6: Auto-complete animation

When auto-complete triggers:
- Cards fly to foundations one at a time with ~80ms stagger
- Each card arcs upward slightly during flight

### Step 7: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr5-animations
git add src/hooks/ src/components/ src/app/globals.css
git commit -m "Add deal, flip, move, and shake animations"
```

---

## PR 6: Theming System — 4 Themes + Light/Dark Mode

**Goal:** Implement the 4 visual themes with light/dark mode toggle, theme selector UI, and settings panel.

**Branch:** `ridermw/pr6-theming`

**Files:**
- Create: `src/components/ThemeProvider.tsx` — context for theme + dark mode
- Create: `src/components/SettingsPanel.tsx` — slide-out settings drawer
- Create: `src/styles/themes.ts` — theme definitions
- Modify: `src/app/globals.css` — CSS custom properties for all themes
- Modify: `src/app/layout.tsx` — wrap with ThemeProvider
- Modify: `src/components/Board.tsx` — add settings button, apply theme classes
- Modify: `src/components/Card.tsx` — theme-aware card back designs

### Step 1: Define theme CSS custom properties

Create theme definitions in `src/styles/themes.ts` and corresponding CSS in `globals.css`.

Each theme defines:
- `--bg-primary` — page background
- `--bg-surface` — board surface
- `--bg-card-front` — card face background
- `--bg-card-back` — card back base color
- `--bg-card-back-pattern` — CSS gradient for card back pattern
- `--color-card-red` — red suit color
- `--color-card-black` — black suit color
- `--color-accent` — buttons, highlights
- `--color-text` — general text
- `--shadow-card` — card box shadow
- `--border-card` — card border
- `--border-empty-pile` — empty pile placeholder border
- `--radius-card` — card border radius

Light/dark mode modifies: `--bg-primary`, `--bg-surface`, `--shadow-card`, `--color-text`, brightness adjustments.

### Step 2: Create ThemeProvider

Create `src/components/ThemeProvider.tsx`:

- Stores active theme name + dark mode boolean in state
- Reads from / writes to localStorage on change
- Applies CSS custom properties to document root
- 300ms CSS transition on `--bg-*` and `--color-*` properties for smooth theme switching
- Exposes `useTheme()` hook: `{ theme, setTheme, isDark, toggleDark }`

### Step 3: Create card back patterns

In Card component, render themed card backs using CSS-only patterns:

- **Classic Casino:** Repeating diamond pattern in red/gold using `linear-gradient` + `repeating-linear-gradient`
- **Modern Minimal:** Geometric grid pattern, muted blue on darker blue
- **Cozy Warm:** Subtle floral-inspired swirl using `radial-gradient` layers
- **Playful:** Rainbow diagonal stripes using `repeating-linear-gradient`

Each pattern is a CSS class that reads from theme custom properties.

### Step 4: Create SettingsPanel

Create `src/components/SettingsPanel.tsx`:

Slide-out drawer from right side:
- **Theme selector:** 4 small thumbnail previews (miniature card + background), click to select. Active theme highlighted.
- **Light/Dark toggle:** Switch component
- **Draw mode:** Radio buttons for 1 or 3. Disabled during active game — shows note "starts on next new game"
- Close button (X)
- Backdrop click to close
- Slide-in animation from right

### Step 5: Wire up Board

Add settings gear icon button to bottom bar. Opens SettingsPanel. Apply theme class to board container.

### Step 6: Wrap layout with ThemeProvider

Modify `src/app/layout.tsx` to wrap children with `<ThemeProvider>`.

### Step 7: Verify all 4 themes + dark mode

Run: `pnpm dev`
Test: Switch between all 4 themes in light and dark mode. Verify card backs change, backgrounds change, transitions are smooth.

### Step 8: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr6-theming
git add src/components/ src/styles/ src/app/globals.css src/app/layout.tsx
git commit -m "Add 4-theme system with light/dark mode and settings panel"
```

---

## PR 7: Sound Effects

**Goal:** Add synthesized sound effects for all game interactions using Web Audio API.

**Branch:** `ridermw/pr7-sound-effects`

**Files:**
- Create: `src/audio/SoundEngine.ts` — Web Audio API sound synthesis
- Create: `src/components/SoundProvider.tsx` — context for sound settings
- Modify: `src/components/SettingsPanel.tsx` — add volume slider and mute toggle
- Modify: `src/components/Board.tsx` — trigger sounds on game actions
- Modify: `src/components/GameProvider.tsx` — expose action callbacks for sound triggers

### Step 1: Create SoundEngine

Create `src/audio/SoundEngine.ts`:

Class that manages an `AudioContext` and provides methods for each sound:

- **`shuffle()`:** 10 rapid noise bursts (white noise filtered through bandpass) over 500ms, simulating card riffle
- **`deal(index: number)`:** Short "thwip" — quick noise burst through highpass filter. Pitch increases slightly with index.
- **`cardPlace()`:** Low-frequency oscillator pulse (~100Hz, 50ms) — soft thud
- **`cardFlip()`:** Short white noise burst through highpass filter (~3000Hz, 30ms) — paper snap
- **`draw()`:** Filtered noise sweep from low to high (~200ms) — swish
- **`invalidMove()`:** Low sawtooth oscillator (~80Hz) with quick decay (~200ms) — buzz
- **`win()`:** Ascending sequence of sine tones (C-E-G-C, each ~200ms) — chime melody

Each method:
- Creates oscillators/noise nodes on the fly (cheap, no pre-allocation needed)
- Respects current volume setting
- No-ops if muted
- Lazy-initializes AudioContext on first user interaction (browser autoplay policy)

### Step 2: Create SoundProvider

Create `src/components/SoundProvider.tsx`:

- Creates single SoundEngine instance
- Stores volume (0-1) and muted boolean in state, persisted to localStorage
- Exposes `useSound()` hook: `{ play, volume, setVolume, muted, toggleMute }`
- `play(soundName)` delegates to SoundEngine methods

### Step 3: Add volume/mute controls to SettingsPanel

Add to SettingsPanel:
- Volume slider (range input, 0-100)
- Mute toggle button (speaker icon / muted speaker icon)

### Step 4: Trigger sounds from game actions

In GameProvider or Board, intercept dispatched actions and play corresponding sounds:
- NEW_GAME → `shuffle()` then `deal()` in sequence
- DRAW → `draw()`
- MOVE (valid) → `cardPlace()`
- MOVE (to foundation) → `cardPlace()` (slightly different?)
- AUTO_MOVE → `cardPlace()`
- Card flip → `cardFlip()`
- Invalid drop → `invalidMove()`
- Game won → `win()`

Use a wrapper around dispatch that plays sounds before/after the action.

### Step 5: Verify all sounds

Run: `pnpm dev`
Test: Each interaction produces appropriate sound. Volume slider works. Mute silences all sounds.

### Step 6: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr7-sound-effects
git add src/audio/ src/components/
git commit -m "Add synthesized sound effects via Web Audio API"
```

---

## PR 8: Win Detection and Celebrations

**Goal:** Detect wins, trigger auto-complete when possible, and display rotating celebration animations.

**Branch:** `ridermw/pr8-win-celebrations`

**Files:**
- Create: `src/components/celebrations/CascadingCards.tsx`
- Create: `src/components/celebrations/Fireworks.tsx`
- Create: `src/components/celebrations/CardFlyAway.tsx`
- Create: `src/components/celebrations/CelebrationOverlay.tsx`
- Modify: `src/components/Board.tsx` — win detection, auto-complete prompt, celebration overlay
- Modify: `src/game/reducer.ts` — increment winCount on win

### Step 1: Create CascadingCards celebration

Create `src/components/celebrations/CascadingCards.tsx`:

Canvas-based animation (this is the one place canvas makes sense — particle effects):
- 52 card-shaped rectangles launch from foundation positions
- Simple physics: gravity pulls down, bounce off screen edges with energy loss
- Each card leaves a fading colorful trail
- Cards have random initial velocity (upward and outward)
- Runs for 10 seconds
- Uses `requestAnimationFrame` loop

### Step 2: Create Fireworks celebration

Create `src/components/celebrations/Fireworks.tsx`:

Canvas-based:
- Multiple bursts of particles from random positions
- Particles spread outward, fade, drift down with gravity
- Colors match current theme accent
- "You Win!" text in center with pulsing glow effect (CSS)
- 3-4 waves of fireworks over 10 seconds

### Step 3: Create CardFlyAway celebration

Create `src/components/celebrations/CardFlyAway.tsx`:

DOM-based (CSS animations):
- All 52 cards are positioned in center, then spiral outward
- Each card rotates and scales down while moving outward
- Staggered start times for vortex effect
- Cards fade out as they reach screen edge
- "Congratulations" text appears in center after cards clear
- CSS keyframes with `animation-delay` for stagger

### Step 4: Create CelebrationOverlay

Create `src/components/celebrations/CelebrationOverlay.tsx`:

Wrapper component:
- Full-screen overlay (fixed, z-index above everything)
- Receives `celebrationType: 0 | 1 | 2` (cycles based on `winCount % 3`)
- Renders the appropriate celebration component
- Shows "Play Again" button after celebration ends (10 seconds)
- "Play Again" dispatches NEW_GAME

### Step 5: Wire into Board

In Board:
- When `state.gameWon` becomes true, show CelebrationOverlay
- Before win, when `canAutoComplete(state)` is true, show "Auto Complete" button or automatically trigger auto-complete
- Increment `winCount` in reducer on win

### Step 6: Verify all 3 celebrations

Run: `pnpm dev`
Test: Win a game (or temporarily make winning easy). Verify each celebration type renders correctly for 10 seconds. Verify rotation across wins.

### Step 7: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr8-win-celebrations
git add src/components/celebrations/ src/components/Board.tsx src/game/reducer.ts
git commit -m "Add win detection with 3 rotating celebration animations"
```

---

## PR 9: Responsive Polish and Final Touches

**Goal:** Responsive design, mobile support, keyboard accessibility, and final polish.

**Branch:** `ridermw/pr9-responsive-polish`

**Files:**
- Modify: `src/app/globals.css` — responsive breakpoints, card sizing
- Modify: `src/components/Board.tsx` — responsive layout adjustments
- Modify: `src/components/Card.tsx` — scalable card sizes
- Modify: `src/components/Pile.tsx` — adjusted overlap for mobile
- Modify: `src/components/SettingsPanel.tsx` — mobile-friendly drawer

### Step 1: Responsive card sizing

Add responsive CSS custom properties:
- Desktop (>1024px): `--card-width: 80px`, `--card-height: 112px`
- Tablet (768-1024px): `--card-width: 65px`, `--card-height: 91px`
- Mobile (<768px): `--card-width: 48px`, `--card-height: 67px`

All card dimensions reference these variables.

### Step 2: Responsive board layout

Adjust Board layout at breakpoints:
- Desktop: comfortable spacing between columns
- Tablet: reduced gaps
- Mobile: minimal gaps, tableau overlap tighter (face-down: 8px, face-up: 18px)

### Step 3: Mobile touch improvements

- Increase touch targets on mobile (buttons at least 44px)
- Prevent scroll while dragging cards (`touch-action: none` on board)
- Prevent text selection during drag

### Step 4: Keyboard accessibility

- Tab navigation between piles
- Enter/Space to select a card, then arrow keys to choose destination, Enter to place
- Escape to cancel selection
- `aria-label` on cards ("Ace of Spades, face up")
- `role="application"` on board

### Step 5: Final visual polish

- Smooth page load (prevent flash of unstyled content)
- Loading state while game initializes
- Subtle background texture/pattern on the board surface
- Card hover effect (slight lift on desktop)

### Step 6: Verify on multiple screen sizes

Run: `pnpm dev`
Test at: 1440px, 1024px, 768px, 375px widths. Verify game is playable at all sizes.

### Step 7: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr9-responsive-polish
git add src/
git commit -m "Add responsive design, mobile support, and accessibility"
```

---

## PR Dependency Chain

```
PR 1 (Deploy) → merged to main
PR 2 (Engine) → branches from main after PR 1
PR 3 (Board) → branches from main after PR 2
PR 4 (Interaction) → branches from main after PR 3
PR 5 (Animations) → branches from main after PR 4
PR 6 (Theming) → branches from main after PR 5
PR 7 (Sound) → branches from main after PR 6
PR 8 (Celebrations) → branches from main after PR 7
PR 9 (Polish) → branches from main after PR 8
```

Each PR builds on the previous but is independently reviewable and deployable.
