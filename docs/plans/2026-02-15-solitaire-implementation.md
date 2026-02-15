# Klondike Solitaire Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a fully playable Klondike solitaire game with 4 selectable themes, light/dark mode, sound effects, drag-and-drop, undo, and rotating win celebrations, hosted on GitHub Pages.

**Architecture:** Single-page Next.js app with static export. Pure TypeScript game engine (useReducer), DOM-based card rendering with CSS animations, Web Audio API for sounds, CSS custom properties for theming. No external dependencies beyond React/Next.js/Tailwind.

**Tech Stack:** Next.js 16, React 19, TypeScript 5, Tailwind CSS 4, Vitest + React Testing Library

**Repo:** `ridermw/conductor-playground` on GitHub (public). Each task below is an independent PR to `main`.

**Branch protection:** `main` requires PRs and passing `build-and-test` CI check. No direct pushes.

**GitHub Pages URL:** `https://ridermw.github.io/conductor-playground/`

**Testing:** Every PR must include comprehensive unit tests. Target high coverage for all game logic, hooks, and component behavior. CI runs `pnpm test -- --coverage` and `pnpm build` as the `build-and-test` status check.

---

## PR 1: Deployment Pipeline + Testing Infrastructure

**Goal:** Configure Next.js for static export, GitHub Pages deployment, and Vitest testing framework. Establishes the CI pipeline that gates all future PRs.

**Branch:** `ridermw/pr1-deploy-and-testing`

**Files:**
- Modify: `next.config.ts`
- Modify: `src/app/layout.tsx` (update metadata)
- Modify: `src/app/page.tsx` (replace boilerplate with placeholder)
- Create: `.github/workflows/ci.yml` (build + test on PRs and push to main)
- Create: `.github/workflows/deploy.yml` (deploy to GitHub Pages on push to main)
- Create: `vitest.config.ts`
- Create: `src/setupTests.ts`
- Create: `src/__tests__/page.test.tsx` (smoke test)

### Step 1: Install Vitest and React Testing Library

Run:
```bash
pnpm add -D vitest @vitejs/plugin-react jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event @vitest/coverage-v8
```

### Step 2: Create vitest.config.ts

```typescript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    globals: true,
    setupFiles: ["./src/setupTests.ts"],
    coverage: {
      provider: "v8",
      reporter: ["text", "text-summary", "lcov"],
      include: ["src/**/*.{ts,tsx}"],
      exclude: ["src/**/*.test.{ts,tsx}", "src/setupTests.ts", "src/app/layout.tsx"],
    },
  },
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```

### Step 3: Create src/setupTests.ts

```typescript
import "@testing-library/jest-dom/vitest";
```

### Step 4: Add test scripts to package.json

Add to `scripts`:
```json
"test": "vitest run",
"test:watch": "vitest",
"test:coverage": "vitest run --coverage"
```

### Step 5: Configure Next.js for static export

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

### Step 6: Update layout metadata

Modify `src/app/layout.tsx` — change the metadata:

```typescript
export const metadata: Metadata = {
  title: "Solitaire",
  description: "Klondike solitaire card game",
};
```

### Step 7: Replace page.tsx with placeholder

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

### Step 8: Write smoke test

Create `src/__tests__/page.test.tsx`:

```tsx
import { render, screen } from "@testing-library/react";
import { describe, it, expect } from "vitest";
import Home from "../app/page";

describe("Home page", () => {
  it("renders the coming soon heading", () => {
    render(<Home />);
    expect(screen.getByText(/Solitaire/i)).toBeInTheDocument();
  });
});
```

### Step 9: Create CI workflow (runs on PRs — this is the `build-and-test` check)

Create `.github/workflows/ci.yml`:

```yaml
name: build-and-test

on:
  pull_request:
    branches: [main]

jobs:
  build-and-test:
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
      - run: pnpm test -- --coverage
      - run: pnpm build
```

### Step 10: Create deploy workflow (runs on push to main)

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
      - run: pnpm test -- --coverage
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

### Step 11: Verify locally

Run: `pnpm test` — expect 1 passing test.
Run: `pnpm build` — expect successful static export to `out/`.

### Step 12: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr1-deploy-and-testing
git add next.config.ts src/app/layout.tsx src/app/page.tsx .github/ vitest.config.ts src/setupTests.ts src/__tests__/ package.json pnpm-lock.yaml
git commit -m "Configure GitHub Pages deploy, Vitest testing, and CI pipeline"
```

Create PR to `main`. CI must pass (the `build-and-test` check). Merge. Verify GitHub Pages deploys.

---

## PR 2: Game Engine — Types, State, and Rules

**Goal:** Build the pure TypeScript game engine with all Klondike rules, shuffling, dealing, and undo. Comprehensive test coverage for all game logic.

**Branch:** `ridermw/pr2-game-engine`

**Files:**
- Create: `src/game/types.ts`
- Create: `src/game/deck.ts`
- Create: `src/game/rules.ts`
- Create: `src/game/reducer.ts`
- Create: `src/__tests__/game/deck.test.ts`
- Create: `src/__tests__/game/rules.test.ts`
- Create: `src/__tests__/game/reducer.test.ts`

### Step 1: Define types

Create `src/game/types.ts`:

```typescript
export type Suit = "spades" | "hearts" | "diamonds" | "clubs";
export type Rank = 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13;

export interface Card {
  suit: Suit;
  rank: Rank;
  faceUp: boolean;
  id: string; // e.g. "hearts-7"
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
  history: GameState[]; // previous states for undo (stored without their own history)
  gameWon: boolean;
  winCount: number; // tracks wins for rotating celebrations
}

export type GameAction =
  | { type: "NEW_GAME"; drawMode: DrawMode }
  | { type: "DRAW" }
  | { type: "MOVE"; from: Location; to: Location }
  | { type: "AUTO_MOVE"; cardId: string }
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

export const FOUNDATION_SUITS: Suit[] = ["spades", "hearts", "diamonds", "clubs"];
```

### Step 3: Write deck tests

Create `src/__tests__/game/deck.test.ts`:

Tests to write:
- `createDeck()` returns 52 cards
- `createDeck()` returns 13 cards per suit
- `createDeck()` returns all unique card IDs
- `createDeck()` returns all cards face-down
- `shuffle()` returns same number of cards
- `shuffle()` returns same set of cards (no duplicates/losses)
- `shuffle()` produces a different order (with high probability — run 10 shuffles, at least one differs)
- `shuffle()` does not mutate original array
- `isRed()` returns true for hearts and diamonds
- `isRed()` returns false for spades and clubs
- `oppositeColor()` returns true for red+black pairs
- `oppositeColor()` returns false for same-color pairs
- `RANK_DISPLAY` maps all 13 ranks correctly
- `SUIT_SYMBOL` maps all 4 suits to correct unicode characters

### Step 4: Create rules module

Create `src/game/rules.ts`:

Implements:
- `canMoveToTableau(card: Card, targetColumn: Card[]): boolean` — alternating color, descending rank. Kings on empty columns.
- `canMoveToFoundation(card: Card, targetFoundation: Card[]): boolean` — same suit, ascending rank. Aces on empty.
- `findAutoMoveTarget(card: Card, state: GameState): Location | null` — checks foundations first, then tableau columns. Returns first valid destination.
- `isGameWon(state: GameState): boolean` — all 4 foundations have 13 cards.
- `canAutoComplete(state: GameState): boolean` — stock empty, waste empty, all tableau cards face-up.
- `getMovableCards(from: Location, state: GameState): Card[]` — returns the card(s) that would be moved. For tableau, returns the card at `position` plus all cards below it.

### Step 5: Write rules tests

Create `src/__tests__/game/rules.test.ts`:

Tests to write (use helper functions to create test cards):

**canMoveToTableau:**
- Red 6 on black 7 → true
- Black 6 on red 7 → true
- Red 6 on red 7 → false (same color)
- Red 6 on black 8 → false (not sequential)
- Red 6 on black 6 → false (same rank)
- King on empty column → true
- Non-king on empty column → false
- Any card on face-down card → false

**canMoveToFoundation:**
- Ace on empty foundation → true
- Non-ace on empty foundation → false
- 2 of spades on Ace of spades → true
- 2 of hearts on Ace of spades → false (wrong suit)
- 3 of spades on Ace of spades → false (not sequential)
- King of spades on Queen of spades → true (completing a foundation)

**findAutoMoveTarget:**
- Ace with empty foundation → returns foundation location
- Card that fits on foundation → returns foundation (priority over tableau)
- Card that fits only on tableau → returns tableau location
- Card with no valid move → returns null

**isGameWon:**
- All foundations with 13 cards → true
- One foundation incomplete → false
- All foundations empty → false

**canAutoComplete:**
- Empty stock, empty waste, all tableau face-up → true
- Non-empty stock → false
- Face-down card in tableau → false

**getMovableCards:**
- Single card from waste → returns [card]
- Top card from tableau → returns [card]
- Middle card from face-up stack in tableau → returns [card, ...cards below]
- Face-down card → returns []

### Step 6: Create game reducer

Create `src/game/reducer.ts`:

Implements `gameReducer(state: GameState, action: GameAction): GameState`:

- **NEW_GAME:** Creates shuffled deck, deals 28 cards to tableau (column i gets i+1 cards, top face-up), remaining 24 to stock. Resets history. Preserves winCount.
- **DRAW:** If stock has cards, move drawMode cards to waste (face-up). If stock empty, recycle waste back to stock (face-down, reversed). Pushes previous state to history.
- **MOVE:** Validates using rules. If valid: move cards, auto-flip newly exposed tableau cards, push history, check win. If invalid: return state unchanged.
- **AUTO_MOVE:** Finds card by ID, determines its location, calls findAutoMoveTarget, dispatches internal MOVE if target found.
- **UNDO:** Pops most recent state from history. Returns it with winCount preserved.
- **AUTO_COMPLETE:** Repeatedly moves the lowest available card from tableau/waste to foundations until no more auto-moves possible.

Export helper: `createInitialState(drawMode: DrawMode): GameState`

### Step 7: Write reducer tests

Create `src/__tests__/game/reducer.test.ts`:

Tests to write (use `createInitialState` for setup, or build specific states for targeted tests):

**NEW_GAME:**
- Creates 7 tableau columns with correct card counts (1,2,3,4,5,6,7)
- Top card of each tableau column is face-up
- All other tableau cards are face-down
- Stock has 24 cards
- Waste is empty
- Foundations are empty
- Total cards = 52 (no cards lost)
- History is empty
- gameWon is false
- Preserves winCount from previous state

**DRAW (draw-1 mode):**
- Moves 1 card from stock to waste, face-up
- Stock decreases by 1
- Waste increases by 1
- Pushes state to history

**DRAW (draw-3 mode):**
- Moves 3 cards from stock to waste (or fewer if stock < 3)
- Cards in waste are face-up

**DRAW (empty stock):**
- Recycles waste back to stock (reversed, face-down)
- Waste becomes empty

**MOVE (tableau to tableau, valid):**
- Card moves from source column to target column
- Source column is shorter, target is longer
- Newly exposed card in source is flipped face-up
- History grows by 1

**MOVE (tableau to tableau, invalid):**
- State is unchanged
- History does not grow

**MOVE (tableau to foundation, valid):**
- Card moves from tableau to correct foundation
- Foundation grows by 1

**MOVE (waste to tableau):**
- Top waste card moves to tableau column
- Waste shrinks by 1

**MOVE (waste to foundation):**
- Top waste card moves to foundation

**MOVE (stack of cards from tableau):**
- Multiple face-up cards move together
- Source loses the whole stack
- Target gains the whole stack

**AUTO_MOVE:**
- Moves card to foundation if possible
- Moves card to tableau if no foundation target
- Does nothing if no valid target

**UNDO:**
- Returns to previous state
- History shrinks by 1
- Cannot undo past empty history (returns same state)

**AUTO_COMPLETE:**
- Moves all available cards to foundations when all cards are face-up
- Sets gameWon to true when all cards reach foundations
- Increments winCount on win

**Win detection:**
- gameWon becomes true when final card reaches foundation
- winCount increments

### Step 8: Run tests and verify

Run: `pnpm test -- --coverage`
Expected: All tests pass. High coverage on `src/game/` files (aim for >95% on rules.ts, reducer.ts, deck.ts).

### Step 9: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr2-game-engine
git add src/game/ src/__tests__/game/
git commit -m "Add Klondike game engine with comprehensive tests"
```

Create PR to `main`. CI must pass.

---

## PR 3: Board Layout and Card Rendering

**Goal:** Render the game board with all card piles and a dealt game. Visual rendering with click-to-draw from stock and button actions (new game, undo).

**Branch:** `ridermw/pr3-board-layout`

**Files:**
- Create: `src/components/Card.tsx`
- Create: `src/components/Pile.tsx`
- Create: `src/components/Board.tsx`
- Create: `src/components/GameProvider.tsx`
- Create: `src/__tests__/components/Card.test.tsx`
- Create: `src/__tests__/components/Pile.test.tsx`
- Create: `src/__tests__/components/Board.test.tsx`
- Create: `src/__tests__/components/GameProvider.test.tsx`
- Modify: `src/app/page.tsx`
- Modify: `src/app/globals.css`

### Step 1: Create GameProvider

Create `src/components/GameProvider.tsx`:

- Wraps `useReducer(gameReducer, createInitialState(1))`
- React context exposes `state` and `dispatch`
- Provides `useGame()` hook

### Step 2: Write GameProvider tests

Create `src/__tests__/components/GameProvider.test.tsx`:

- Provides initial game state to children
- dispatch triggers state changes (e.g., DRAW)
- useGame() throws when used outside provider

### Step 3: Create Card component

Create `src/components/Card.tsx`:

- ~70px wide x ~100px tall (uses CSS custom properties)
- Front face: rank top-left + bottom-right, suit symbol center, red/black coloring
- Back face: default green/gold pattern
- CSS 3D transform for face-up/face-down
- `data-card-id` attribute
- `onClick`, `onDoubleClick`, `style`, `className` props

### Step 4: Write Card tests

Create `src/__tests__/components/Card.test.tsx`:

- Renders face-up card showing rank and suit
- Renders face-down card showing back pattern
- Applies correct color class for red suits
- Applies correct color class for black suits
- Calls onClick when clicked
- Calls onDoubleClick when double-clicked
- Sets data-card-id attribute
- Renders all 13 ranks correctly (A, 2-10, J, Q, K)

### Step 5: Create Pile component

Create `src/components/Pile.tsx`:

- **Stock type:** single card face-down, or empty placeholder with recycle icon. Click handler for drawing.
- **Waste type:** shows top card(s) face-up. Draw-3 mode fans top 3 cards.
- **Foundation type:** shows top card or empty placeholder with suit symbol.
- **Tableau type:** vertical fan with overlap. Face-down: 15px offset. Face-up: 25px offset. Absolute positioning within relative container.

### Step 6: Write Pile tests

Create `src/__tests__/components/Pile.test.tsx`:

- Stock pile: renders face-down card when stock is not empty
- Stock pile: renders empty placeholder when stock is empty
- Stock pile: calls onDraw when clicked
- Waste pile: renders top card face-up
- Foundation pile: renders top card when not empty
- Foundation pile: shows suit symbol placeholder when empty
- Tableau pile: renders correct number of cards
- Tableau pile: applies face-up/face-down correctly based on card.faceUp

### Step 7: Create Board component

Create `src/components/Board.tsx`:

- Full viewport layout
- Top row: Stock, Waste, spacer, 4 Foundations
- Main area: 7 Tableau columns
- Bottom bar: New Game button, Undo button
- Stock click dispatches DRAW
- New Game shows draw mode selector, then dispatches NEW_GAME
- Undo dispatches UNDO

### Step 8: Write Board tests

Create `src/__tests__/components/Board.test.tsx`:

- Renders 7 tableau columns
- Renders 4 foundation placeholders
- Renders stock pile
- Renders waste pile
- New Game button triggers new game flow
- Undo button dispatches UNDO
- Stock click dispatches DRAW

### Step 9: Wire up page.tsx

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

### Step 10: Add base CSS

Add to `src/app/globals.css`:
- Card dimension CSS vars
- 3D perspective on board
- Card face/back base styles
- Empty pile placeholder styles
- Default board background: dark green (`#1a472a`)

### Step 11: Run tests and verify

Run: `pnpm test -- --coverage`
Expected: All tests pass. Build succeeds.

Run: `pnpm dev` — visual verification: board renders with dealt game, stock click draws, new game works.

### Step 12: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr3-board-layout
git add src/components/ src/__tests__/components/ src/app/page.tsx src/app/globals.css
git commit -m "Add board layout with card rendering, game context, and tests"
```

---

## PR 4: Card Interaction — Drag-and-Drop + Double-Click

**Goal:** Make the game fully playable with drag-and-drop and double-click auto-move.

**Branch:** `ridermw/pr4-card-interaction`

**Files:**
- Create: `src/hooks/useDragAndDrop.ts`
- Create: `src/__tests__/hooks/useDragAndDrop.test.ts`
- Modify: `src/components/Card.tsx`
- Modify: `src/components/Pile.tsx`
- Modify: `src/components/Board.tsx`
- Modify: `src/__tests__/components/Board.test.tsx`

### Step 1: Create useDragAndDrop hook

Create `src/hooks/useDragAndDrop.ts`:

- Tracks drag state: `{ isDragging, dragCards, dragOrigin, position }` or null
- `onPointerDown`: records origin, starts tracking (with 5px minimum distance threshold)
- `onPointerMove`: updates dragged card position via `requestAnimationFrame`
- `onPointerUp`: resolves drop — checks `document.elementsFromPoint()` for `[data-drop-zone]`, dispatches MOVE if valid
- `setPointerCapture` for reliable cross-element tracking
- Supports grabbing stacks from tableau (clicked card + all below)

### Step 2: Write useDragAndDrop tests

Create `src/__tests__/hooks/useDragAndDrop.test.ts`:

Use `renderHook` + `fireEvent`:
- Returns isDragging=false initially
- Starts drag on pointerdown + pointermove beyond threshold
- Does not start drag for small movements (< 5px)
- Updates position during pointermove
- Resets drag state on pointerup
- Includes dragOrigin location info
- Includes correct card(s) in dragCards

### Step 3: Add double-click to Card

In Card component, `onDoubleClick` dispatches `AUTO_MOVE` with card ID.

### Step 4: Add drop zones to Pile

Add `data-drop-zone="${type}-${index}"` to each pile container element.

### Step 5: Add drag overlay to Board

In Board, render a floating layer when drag is active:
- Fixed position following pointer
- Scale 1.05x, elevated shadow
- z-index 1000
- Valid drop zones get highlighted border/glow

### Step 6: Update Board tests

Add to Board tests:
- Double-clicking a face-up card triggers AUTO_MOVE
- Valid drop zones show visual feedback during drag (via CSS class)

### Step 7: Run tests and verify

Run: `pnpm test -- --coverage`
Expected: All tests pass.

Run: `pnpm dev` — manually verify: drag cards between piles, double-click auto-moves, draw from stock, undo.

### Step 8: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr4-card-interaction
git add src/hooks/ src/__tests__/hooks/ src/components/ src/__tests__/components/
git commit -m "Add drag-and-drop and double-click interaction with tests"
```

---

## PR 5: Card Animations

**Goal:** Smooth animations for dealing, card flipping, movement, and invalid move feedback.

**Branch:** `ridermw/pr5-animations`

**Files:**
- Create: `src/hooks/useAnimationState.ts`
- Create: `src/__tests__/hooks/useAnimationState.test.ts`
- Modify: `src/components/Card.tsx`
- Modify: `src/components/Board.tsx`
- Modify: `src/app/globals.css`

### Step 1: Add CSS keyframes

Add to `src/app/globals.css`:
- `@keyframes card-shake` — left-right shake for invalid moves
- `@keyframes card-deal` — translate from stock position to target
- Card flip: `transition: transform 300ms ease-in-out` on card container
- Card move: `transition: top 200ms ease-out, left 200ms ease-out`

### Step 2: Create useAnimationState hook

Create `src/hooks/useAnimationState.ts`:

Tracks:
- `isDealing: boolean` — true during deal animation, blocks interaction
- `shakingCardId: string | null` — card currently shaking
- `dealCard(index, targetPosition)` — triggers staggered deal animation
- `shakeCard(cardId)` — triggers shake, auto-clears after animation

### Step 3: Write useAnimationState tests

- isDealing starts as false
- dealCard sets isDealing to true
- isDealing resets to false after deal completes
- shakeCard sets shakingCardId
- shakingCardId auto-clears after timeout

### Step 4: Card flip animation

In Card: CSS class toggle when faceUp changes. 3D rotateY transition.

### Step 5: Deal animation in Board

On NEW_GAME: set isDealing=true, stagger card appearances (~50ms each), set isDealing=false when done. Disable interaction during deal.

### Step 6: Invalid move shake

On failed drop: apply shake class to card, remove after animation end.

### Step 7: Auto-complete animation

When AUTO_COMPLETE fires: stagger card movements to foundations (~80ms each).

### Step 8: Run tests and verify

Run: `pnpm test -- --coverage`
Run: `pnpm dev` — visual verification of animations.

### Step 9: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr5-animations
git add src/hooks/ src/__tests__/hooks/ src/components/ src/app/globals.css
git commit -m "Add deal, flip, move, and shake animations with tests"
```

---

## PR 6: Theming — 4 Themes + Light/Dark Mode + Settings Panel

**Goal:** 4 visual themes, light/dark toggle, card back patterns, and a settings drawer.

**Branch:** `ridermw/pr6-theming`

**Files:**
- Create: `src/styles/themes.ts`
- Create: `src/components/ThemeProvider.tsx`
- Create: `src/components/SettingsPanel.tsx`
- Create: `src/__tests__/styles/themes.test.ts`
- Create: `src/__tests__/components/ThemeProvider.test.tsx`
- Create: `src/__tests__/components/SettingsPanel.test.tsx`
- Modify: `src/app/globals.css`
- Modify: `src/app/layout.tsx`
- Modify: `src/components/Board.tsx`
- Modify: `src/components/Card.tsx`

### Step 1: Define themes

Create `src/styles/themes.ts`:

Export a `THEMES` record mapping theme names to objects of CSS custom properties. 4 themes: `casino`, `minimal`, `cozy`, `playful`. Each theme has light and dark variants.

~15 properties per theme variant (see design doc for color specs).

### Step 2: Write themes tests

Create `src/__tests__/styles/themes.test.ts`:

- THEMES has exactly 4 entries
- Each theme has all required CSS property keys
- Each theme has both light and dark variants
- No duplicate property values across unrelated properties (sanity check)

### Step 3: Create ThemeProvider

- Stores `themeName` and `isDark` in state
- Reads initial values from localStorage (with fallback to `casino` + system preference for dark)
- Applies CSS custom properties to `document.documentElement.style`
- Persists changes to localStorage
- 300ms transition class on root element
- Exposes `useTheme()` hook

### Step 4: Write ThemeProvider tests

- Defaults to casino theme
- Applies CSS properties to document root
- setTheme changes active theme and persists
- toggleDark switches dark mode
- Reads persisted values from localStorage on mount

### Step 5: Create themed card backs

In Card component, render CSS-only patterns per theme:
- Casino: red/gold diamond pattern
- Minimal: geometric blue grid
- Cozy: warm radial gradient swirls
- Playful: rainbow diagonal stripes

### Step 6: Create SettingsPanel

Slide-out drawer from right:
- Theme thumbnails (4 miniature previews)
- Light/Dark toggle
- Draw mode selector (1 or 3, note: "applies on next new game")
- Close button + backdrop click to close

### Step 7: Write SettingsPanel tests

- Renders all 4 theme options
- Clicking a theme calls setTheme
- Dark mode toggle works
- Draw mode radio buttons update state
- Close button calls onClose
- Not rendered when closed

### Step 8: Wire into Board and layout

- Add settings gear button to Board bottom bar
- Wrap layout.tsx children with ThemeProvider

### Step 9: Run tests and verify

Run: `pnpm test -- --coverage`
Run: `pnpm dev` — switch all themes in both modes.

### Step 10: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr6-theming
git add src/styles/ src/components/ src/__tests__/ src/app/globals.css src/app/layout.tsx
git commit -m "Add 4-theme system with light/dark mode, settings panel, and tests"
```

---

## PR 7: Sound Effects

**Goal:** Synthesized sound effects for all game interactions via Web Audio API.

**Branch:** `ridermw/pr7-sound-effects`

**Files:**
- Create: `src/audio/SoundEngine.ts`
- Create: `src/components/SoundProvider.tsx`
- Create: `src/__tests__/audio/SoundEngine.test.ts`
- Create: `src/__tests__/components/SoundProvider.test.tsx`
- Modify: `src/components/SettingsPanel.tsx`
- Modify: `src/components/Board.tsx` (or `src/components/GameProvider.tsx`)

### Step 1: Create SoundEngine

Create `src/audio/SoundEngine.ts`:

Class wrapping `AudioContext`:
- `shuffle()`: 10 rapid noise bursts over 500ms
- `deal(index)`: thwip sound, pitch increases with index
- `cardPlace()`: low-freq pulse, 50ms
- `cardFlip()`: high-freq noise burst, 30ms
- `draw()`: noise sweep low→high, 200ms
- `invalidMove()`: sawtooth 80Hz, 200ms decay
- `win()`: C-E-G-C ascending sine tones

All methods respect `volume` and `muted` properties. Lazy-init AudioContext on first call.

### Step 2: Write SoundEngine tests

Create `src/__tests__/audio/SoundEngine.test.ts`:

Mock `AudioContext`, `OscillatorNode`, `GainNode` using vitest mocks:
- Constructor does not create AudioContext (lazy init)
- First sound call creates AudioContext
- shuffle() creates expected oscillator/noise nodes
- Muted mode: methods return without creating nodes
- Volume of 0: gain is set to 0
- Each method creates and starts the expected node types

### Step 3: Create SoundProvider

- Single SoundEngine instance via useRef
- State: volume (0-1), muted boolean
- Persisted to localStorage
- `useSound()` hook: `{ play, volume, setVolume, muted, toggleMute }`

### Step 4: Write SoundProvider tests

- Provides play function to children
- play('shuffle') calls SoundEngine.shuffle()
- Mute toggle updates state and persists
- Volume change updates state and persists

### Step 5: Add volume/mute to SettingsPanel

- Volume slider (0-100 range input)
- Mute toggle (speaker icon)

### Step 6: Trigger sounds from game actions

Wrap dispatch in GameProvider (or Board) to intercept actions:
- NEW_GAME → shuffle + deal sounds
- DRAW → draw sound
- MOVE (valid) → cardPlace
- Card flip → cardFlip
- Invalid drop → invalidMove
- Game won → win

### Step 7: Run tests and verify

Run: `pnpm test -- --coverage`
Run: `pnpm dev` — play with sound on, verify each interaction has a sound.

### Step 8: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr7-sound-effects
git add src/audio/ src/__tests__/audio/ src/components/ src/__tests__/components/
git commit -m "Add synthesized sound effects via Web Audio API with tests"
```

---

## PR 8: Win Celebrations

**Goal:** 3 rotating celebration animations, 10 seconds each, triggered on game win.

**Branch:** `ridermw/pr8-win-celebrations`

**Files:**
- Create: `src/components/celebrations/CascadingCards.tsx`
- Create: `src/components/celebrations/Fireworks.tsx`
- Create: `src/components/celebrations/CardFlyAway.tsx`
- Create: `src/components/celebrations/CelebrationOverlay.tsx`
- Create: `src/__tests__/components/celebrations/CelebrationOverlay.test.tsx`
- Modify: `src/components/Board.tsx`

### Step 1: Create CascadingCards

Canvas-based: 52 card rectangles launched from foundations with gravity physics, bouncing off edges, colorful trails. 10 seconds. Uses `requestAnimationFrame`.

### Step 2: Create Fireworks

Canvas-based: particle bursts from random positions, multi-wave, theme-colored. "You Win!" text with glow. 10 seconds.

### Step 3: Create CardFlyAway

DOM-based CSS animations: 52 cards spiral outward from center in vortex, fade out. "Congratulations" text after cards clear. 10 seconds.

### Step 4: Create CelebrationOverlay

Wrapper:
- Full-screen fixed overlay, z-index 9999
- Selects celebration based on `winCount % 3`
- Shows "Play Again" button after 10 seconds
- "Play Again" dispatches NEW_GAME

### Step 5: Write CelebrationOverlay tests

- Renders CascadingCards when winCount % 3 === 0
- Renders Fireworks when winCount % 3 === 1
- Renders CardFlyAway when winCount % 3 === 2
- Shows "Play Again" button after timeout
- Play Again click dispatches NEW_GAME
- Overlay renders when gameWon is true
- Overlay does not render when gameWon is false

### Step 6: Wire into Board

- When `state.gameWon` → show CelebrationOverlay
- When `canAutoComplete(state)` → show "Auto Complete" button or auto-trigger

### Step 7: Run tests and verify

Run: `pnpm test -- --coverage`
Run: `pnpm dev` — trigger a win (can temporarily rig a winning deal in tests/dev).

### Step 8: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr8-win-celebrations
git add src/components/celebrations/ src/__tests__/components/celebrations/ src/components/Board.tsx
git commit -m "Add 3 rotating win celebrations with tests"
```

---

## PR 9: Responsive Design + Accessibility + Polish

**Goal:** Mobile-friendly layout, keyboard accessibility, final visual polish.

**Branch:** `ridermw/pr9-responsive-polish`

**Files:**
- Modify: `src/app/globals.css` — responsive breakpoints
- Modify: `src/components/Board.tsx` — responsive layout
- Modify: `src/components/Card.tsx` — scalable sizing
- Modify: `src/components/Pile.tsx` — mobile overlap adjustments
- Modify: `src/components/SettingsPanel.tsx` — mobile drawer
- Create: `src/__tests__/components/Board.responsive.test.tsx`

### Step 1: Responsive card sizing

CSS custom properties at breakpoints:
- Desktop (>1024px): `--card-width: 80px`, `--card-height: 112px`
- Tablet (768-1024px): `--card-width: 65px`, `--card-height: 91px`
- Mobile (<768px): `--card-width: 48px`, `--card-height: 67px`

### Step 2: Responsive board layout

Adjusted spacing, gaps, and overlap per breakpoint.

### Step 3: Mobile touch

- `touch-action: none` on board during drag
- 44px minimum touch targets for buttons
- No text selection during drag

### Step 4: Keyboard accessibility

- Tab through piles
- Enter/Space to pick up card
- Arrow keys to choose destination
- Escape to cancel
- `aria-label` on cards (e.g. "Ace of Spades, face up")

### Step 5: Write responsive/accessibility tests

Create `src/__tests__/components/Board.responsive.test.tsx`:

- Cards have correct aria-labels
- Buttons are keyboard-focusable
- Board has appropriate role

### Step 6: Final visual polish

- Card hover lift on desktop
- Page load smoothness (no FOUC)
- Board surface texture

### Step 7: Run tests and verify

Run: `pnpm test -- --coverage`
Run: `pnpm dev` — test at 1440px, 1024px, 768px, 375px. Test keyboard navigation.

### Step 8: Commit and create PR

```bash
git checkout main && git pull
git checkout -b ridermw/pr9-responsive-polish
git add src/
git commit -m "Add responsive design, accessibility, and visual polish with tests"
```

---

## PR Dependency Chain

```
PR 1 (Deploy + Testing) → merged to main
PR 2 (Engine + Tests) → branches from main after PR 1
PR 3 (Board + Tests) → branches from main after PR 2
PR 4 (Interaction + Tests) → branches from main after PR 3
PR 5 (Animations + Tests) → branches from main after PR 4
PR 6 (Theming + Tests) → branches from main after PR 5
PR 7 (Sound + Tests) → branches from main after PR 6
PR 8 (Celebrations + Tests) → branches from main after PR 7
PR 9 (Polish + Tests) → branches from main after PR 8
```

Each PR includes comprehensive tests, passes CI (`build-and-test`), and is independently reviewable.
