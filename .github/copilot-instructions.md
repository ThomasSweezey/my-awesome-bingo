# GitHub Copilot Instructions — Bingo Mixer

Mobile-first social bingo game. React 19 + Vite 8 + TypeScript + Tailwind CSS v4. No backend — browser only with `localStorage` persistence.

---

## How the app is wired together

**No router.** Navigation is a single `gameState` value in `useBingoGame.ts`:

```
'start'   → <StartScreen />
'playing' → <GameScreen /> [+ <BingoModal /> when showBingoModal=true]
'bingo'   → <GameScreen hasBingo=true> [+ <BingoModal />]
```

**`src/hooks/useBingoGame.ts` owns all state.** Every component is pure/presentational — props in, callbacks out. Don't lift state into components.

**The board is a flat 25-element array** (`BingoSquareData[]`). `square.id === arrayIndex` always. Index 12 is the FREE SPACE (center). `checkBingo()` and `toggleSquare()` in `src/utils/bingoLogic.ts` operate on this array.

**Persistence:** a `useEffect` in `useBingoGame` saves `{ version, gameState, board, winningLine }` to `localStorage` key `'bingo-game-state'` on every change. `validateStoredData()` guards the load path and clears stale/corrupt data silently. Don't access `localStorage` anywhere else.

**`queueMicrotask` in `handleSquareClick`:** bingo detection (`setWinningLine`, `setGameState`, `setShowBingoModal`) is scheduled via `queueMicrotask` inside the `setBoard` updater to avoid synchronous `setState` inside another `setState`.

---

## Styling — Tailwind CSS v4

No `tailwind.config.js`. Tokens live in `src/index.css`:

```css
@theme {
  --color-accent: #2563eb;
  --color-accent-light: #3b82f6;
  --color-marked: #dcfce7;
  --color-marked-border: #22c55e;
  --color-bingo: #fbbf24;
}
```

Use as classes: `bg-accent`, `border-marked-border`, etc. Never hardcode hex in JSX. See `.github/instructions/tailwind-4.instructions.md` for full v4 reference.

---

## Commands

```bash
npm run dev    # dev server → http://localhost:5173
npm run build  # tsc + vite build → dist/
npm test       # vitest run (once, no watch)
npm run lint   # eslint
```

New tests → `src/**/*.test.ts` (Vitest auto-discovers). Push to `main` → CI builds with `VITE_REPO_NAME` env var and copies `dist/` to `docs/game/` on GitHub Pages. Locally, `base` is `/`.

---

## Conventions & gotchas

- **Named exports everywhere** — except `App.tsx` (default export, Vite convention).
- **Pure logic → `bingoLogic.ts`** — if it doesn't need React/DOM, it goes there with a unit test.
- **`questions.ts` must have ≥ 24 strings** — `generateBoard()` picks exactly 24; extras add shuffle variety.
- **Free spaces are immutable** — `toggleSquare()` guards `isFreeSpace`; `BingoSquare` sets `disabled={square.isFreeSpace}`. Don't bypass this.
- **Pass `winningSquareIds` (a `Set<number>`) to components**, not `winningLine`. The Set is derived via `useMemo` for O(1) render lookup.
- **`BingoLine.type = 'corners'`** is defined in `src/types/index.ts` but NOT implemented in `getWinningLines()`. It's a reserved future feature.
- **`docs/` ≠ the React app.** `docs/` is the static workshop landing page. The built React app lands at `docs/game/` after CI.
