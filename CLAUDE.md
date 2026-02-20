# Klondike Solitaire — Project Guidelines

## Project Overview

A browser-based Klondike solitaire card game built with Next.js 16, React 19, and Tailwind CSS 4. The game features four selectable visual themes, light/dark mode, synthesized sound effects, drag-and-drop interaction, unlimited undo, and rotating win celebrations. Deployed to GitHub Pages.

**Repository:** `ridermw/conductor-playground` (public)
**GitHub Pages URL:** `https://ridermw.github.io/conductor-playground/`
**Main Branch:** `main` (protected, requires PRs and passing CI)

## Tech Stack

- **Framework:** Next.js 16.1.6 (static export mode)
- **UI Library:** React 19.2.3
- **Language:** TypeScript 5
- **Styling:** Tailwind CSS 4 with CSS custom properties
- **Testing:** Vitest + React Testing Library + @vitest/coverage-v8
- **Package Manager:** pnpm (version 9)
- **Node Version:** 20

## Commands

### Development
```bash
pnpm dev              # Start development server
pnpm build            # Build for production
pnpm start            # Start production server (not used for GitHub Pages)
pnpm lint             # Run ESLint
```

### Testing (available after PR1)
```bash
pnpm test             # Run all tests once
pnpm test:watch       # Run tests in watch mode
pnpm test:coverage    # Run tests with coverage report
```

### Installation
```bash
pnpm install          # Install dependencies
pnpm install --frozen-lockfile  # Install with locked versions (used in CI)
```

## Project Structure

```
conductor-playground/
├── .github/
│   └── workflows/
│       ├── ci.yml           # Build & test CI (runs on PRs)
│       └── deploy.yml       # GitHub Pages deployment (runs on push to main)
├── docs/
│   └── plans/
│       ├── 2026-02-15-solitaire-design.md
│       └── 2026-02-15-solitaire-implementation.md
├── src/
│   ├── __tests__/           # Test files mirror src/ structure
│   │   ├── game/            # Game engine tests
│   │   ├── hooks/           # React hooks tests
│   │   └── components/      # Component tests
│   ├── app/
│   │   ├── layout.tsx       # Root layout with metadata
│   │   ├── page.tsx         # Main page component
│   │   ├── globals.css      # Global styles
│   │   └── favicon.ico
│   ├── audio/               # Web Audio API sound engine
│   ├── components/          # React components
│   │   └── celebrations/    # Win celebration animations
│   ├── game/                # Pure TypeScript game logic
│   │   ├── types.ts         # Game state types
│   │   ├── deck.ts          # Card utilities
│   │   ├── rules.ts         # Game rules validation
│   │   └── reducer.ts       # Game state reducer
│   ├── hooks/               # Custom React hooks
│   ├── styles/              # Theme definitions
│   └── setupTests.ts        # Vitest setup
├── next.config.ts
├── tsconfig.json
├── vitest.config.ts
├── package.json
├── pnpm-lock.yaml
└── CLAUDE.md
```

## Code Style & Conventions

### TypeScript
- **Strict mode enabled** — all compiler strict flags are on
- **No implicit any** — always provide explicit types
- **Explicit return types** on exported functions
- **Path aliases:** Use `@/` for `./src/` imports
- **Target:** ES2017

### React
- **Functional components** with TypeScript types
- **React 19** — use modern APIs (useTransition, useOptimistic, etc.)
- **Client components** — mark with `"use client"` directive when needed
- **Server components** — default for app/ directory
- **Hooks:** Custom hooks in `src/hooks/`, prefixed with `use`

### Naming Conventions
- **Files:** PascalCase for components (`Card.tsx`), camelCase for utilities (`deck.ts`)
- **Components:** PascalCase (`GameProvider`, `Board`)
- **Functions/Variables:** camelCase (`createDeck`, `isGameWon`)
- **Types/Interfaces:** PascalCase (`GameState`, `Card`, `Suit`)
- **Constants:** UPPER_SNAKE_CASE for enums/constants (`RANK_DISPLAY`, `SUIT_SYMBOL`)
- **CSS Custom Properties:** kebab-case (`--card-width`, `--theme-background`)

### File Organization
- **One component per file** — export as default or named export
- **Co-locate tests** — `__tests__/` directory mirrors `src/` structure
- **Group related code** — e.g., `game/` for game engine, `components/` for UI
- **Index files:** Avoid barrel exports; prefer explicit imports

### CSS & Styling
- **Tailwind utility classes** in JSX
- **CSS custom properties** for theming (stored in `document.documentElement.style`)
- **Global styles** in `globals.css` for CSS variables and keyframes
- **No inline styles** except for dynamic positioning (drag-and-drop)
- **Responsive breakpoints:**
  - Mobile: `< 768px`
  - Tablet: `768px - 1024px`
  - Desktop: `> 1024px`

### Testing
- **Test file naming:** `*.test.ts` or `*.test.tsx`
- **Location:** `src/__tests__/` directory mirroring source structure
- **Coverage targets:** >95% for game logic (rules, reducer, deck)
- **Test structure:** Use `describe` blocks for grouping, `it` for individual tests
- **Assertions:** Use `@testing-library/jest-dom` matchers
- **Mocking:** Use Vitest's `vi.mock()` for external dependencies
- **Testing principles:**
  - Test behavior, not implementation
  - Test user interactions (click, drag, etc.)
  - Test edge cases and error conditions
  - Mock Web APIs (AudioContext, localStorage) when needed

### Git & GitHub Workflow
- **Protected main branch** — requires PR approval and passing CI
- **Branch naming:** `ridermw/<pr-number>-<short-description>` (e.g., `ridermw/pr1-deploy-and-testing`)
- **Commit messages:** Clear, descriptive, imperative mood
  - Good: "Add drag-and-drop interaction with tests"
  - Bad: "fixed stuff"
- **PR structure:** Each PR is independently reviewable with comprehensive tests
- **CI requirements:** Must pass `build-and-test` check before merge
- **No direct pushes to main** — all changes via PR

## Architecture Patterns

### Game Engine (Pure TypeScript)
- **Reducer pattern:** All game state changes via `gameReducer(state, action)`
- **Immutable state:** Never mutate state directly; return new objects
- **No UI dependencies** — game logic is framework-agnostic
- **Validation first** — check move validity before applying state changes
- **History tracking** — previous states stored for undo (without nested history)

### Component Architecture
- **Provider pattern:** `GameProvider`, `ThemeProvider`, `SoundProvider` for shared state
- **Custom hooks:** Extract reusable logic (`useGame`, `useTheme`, `useSound`, `useDragAndDrop`)
- **Separation of concerns:**
  - Game logic in `src/game/`
  - UI components in `src/components/`
  - Interaction logic in `src/hooks/`
  - Audio in `src/audio/`
- **Composition over inheritance** — small, focused components

### State Management
- **useReducer** for complex state (game state)
- **useState** for simple local state (UI toggles)
- **Context** for sharing state across components
- **localStorage** for persistence (theme, sound settings)

### Animations
- **CSS animations** for card movements, flips, and transitions
- **CSS keyframes** for reusable animations (shake, deal)
- **requestAnimationFrame** for complex physics (cascading cards)
- **Canvas** for particle effects (fireworks, confetti)

### Theming
- **CSS custom properties** for all theme values
- **Four theme variants:** Casino, Minimal, Cozy, Playful
- **Light/dark mode** as an additional layer on themes
- **300ms transitions** on theme changes
- **Persistence** to localStorage

### Sound
- **Web Audio API** — all sounds synthesized, no audio files
- **Lazy initialization** — AudioContext created on first sound
- **Volume control** and mute toggle
- **Persistence** to localStorage

## Dependencies

### Production
- `next` (16.1.6)
- `react` (19.2.3)
- `react-dom` (19.2.3)

### Development
- `@tailwindcss/postcss` (^4)
- `@types/node` (^20)
- `@types/react` (^19)
- `@types/react-dom` (^19)
- `@vitejs/plugin-react` (for Vitest)
- `@vitest/coverage-v8` (for coverage reporting)
- `@testing-library/react`
- `@testing-library/jest-dom`
- `@testing-library/user-event`
- `eslint` (^9)
- `eslint-config-next` (16.1.6)
- `jsdom` (for Vitest)
- `tailwindcss` (^4)
- `typescript` (^5)
- `vitest`

## Next.js Configuration

**Static Export for GitHub Pages:**
```typescript
// next.config.ts
const nextConfig: NextConfig = {
  output: "export",
  basePath: "/conductor-playground",
  images: {
    unoptimized: true,
  },
};
```

- **output: "export"** — generates static HTML/CSS/JS files
- **basePath** — matches GitHub Pages repo path
- **images.unoptimized** — required for static export (no image optimization)

## CI/CD Pipeline

### Build & Test CI (`.github/workflows/ci.yml`)
- **Trigger:** Pull requests to `main`
- **Job name:** `build-and-test` (this is the status check name)
- **Steps:**
  1. Checkout code
  2. Setup pnpm (v9)
  3. Setup Node.js (v20)
  4. Install dependencies (`pnpm install --frozen-lockfile`)
  5. Run tests with coverage (`pnpm test -- --coverage`)
  6. Build for production (`pnpm build`)
- **Requirement:** Must pass for PR to be mergeable

### GitHub Pages Deploy (`.github/workflows/deploy.yml`)
- **Trigger:** Push to `main`
- **Permissions:** Read contents, write to pages, id-token for deployment
- **Jobs:**
  1. **Build:** Install, test, build, upload artifact
  2. **Deploy:** Deploy artifact to GitHub Pages
- **Result:** Site live at `https://ridermw.github.io/conductor-playground/`

## Implementation Plan

The game is implemented across 9 independent PRs, each fully tested and reviewable:

1. **PR1: Deployment Pipeline + Testing Infrastructure** — Next.js static export, GitHub Actions CI/CD, Vitest setup
2. **PR2: Game Engine** — Pure TypeScript game logic with comprehensive tests
3. **PR3: Board Layout** — Card rendering, game board, basic interaction
4. **PR4: Card Interaction** — Drag-and-drop, double-click auto-move
5. **PR5: Card Animations** — Deal, flip, move, shake animations
6. **PR6: Theming** — 4 themes, light/dark mode, settings panel
7. **PR7: Sound Effects** — Web Audio API synthesized sounds
8. **PR8: Win Celebrations** — 3 rotating celebration animations
9. **PR9: Responsive Design + Accessibility** — Mobile layout, keyboard nav, polish

See `docs/plans/2026-02-15-solitaire-implementation.md` for detailed steps.

## Game Rules (Klondike Solitaire)

- **Tableau:** 7 columns, descending rank, alternating colors (red/black)
- **Foundations:** 4 piles (one per suit), ascending rank from Ace to King
- **Stock/Waste:** Draw-1 or draw-3 mode (player chooses at game start)
- **Empty tableau columns:** Only Kings
- **Empty foundations:** Only Aces
- **Win condition:** All 52 cards in foundations
- **No scoring** — play for fun

## Important Notes for AI Assistants

### When Working on This Project:
1. **Always write tests** — every PR must include comprehensive tests
2. **Run tests before committing** — `pnpm test -- --coverage`
3. **Build locally** — `pnpm build` must succeed
4. **Follow the PR sequence** — each PR builds on the previous
5. **No direct main pushes** — all changes via PR
6. **Preserve test coverage** — maintain >95% for game logic
7. **Use the plan** — `docs/plans/2026-02-15-solitaire-implementation.md` is authoritative
8. **Pure game logic** — game engine has no React/DOM dependencies
9. **Type safety** — no `any` types, explicit typing everywhere
10. **Accessibility** — keyboard navigation, ARIA labels, responsive design

### Common Pitfalls to Avoid:
- ❌ Mutating state directly in reducer
- ❌ UI logic in game engine files
- ❌ Skipping tests or test coverage
- ❌ Using relative paths instead of `@/` alias
- ❌ Committing without running build + test
- ❌ Creating PRs with missing dependencies on previous PRs
- ❌ Using external dependencies beyond the approved stack

### When Adding Features:
1. Check if it fits within the existing PR plan
2. Add tests first (TDD approach preferred)
3. Update types if needed
4. Run full test suite
5. Verify build succeeds
6. Consider responsive/mobile implications
7. Update this file if architecture changes

## Resources

- **Design Doc:** `docs/plans/2026-02-15-solitaire-design.md`
- **Implementation Plan:** `docs/plans/2026-02-15-solitaire-implementation.md`
- **Next.js Docs:** https://nextjs.org/docs
- **React 19 Docs:** https://react.dev
- **Tailwind CSS 4:** https://tailwindcss.com/docs
- **Vitest Docs:** https://vitest.dev
- **Testing Library:** https://testing-library.com/react

---

**Last Updated:** 2026-02-16
**Current Branch:** `ridermw/solitaire-game`
**Status:** Project initialization complete, ready for PR1
