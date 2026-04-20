# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project structure

This is a multi-game arcade platform served by nginx on `304719.fornex.cloud`. Each game is a **single self-contained `index.html`** file with inline CSS and JS — no build step, no dependencies except Google Fonts.

```
/home/projects/
├── games/       → 304719.fornex.cloud/          (catalog landing page)
├── site/        → 304719.fornex.cloud/tictactoe/ (tic-tac-toe)
├── tetris/      → 304719.fornex.cloud/tetris/
├── snake/       → 304719.fornex.cloud/snake/
```

Each project has its own git repo pushed to `github.com/ikrutikof/<name>`.

## Deployment

Editing `index.html` in any game folder makes changes **live immediately** — nginx serves files directly.

Nginx reload is only needed when changing `/etc/nginx/sites-available/tictactoe.conf`:
```bash
nginx -t && systemctl reload nginx
```

Verify a URL is live:
```bash
curl -s -o /dev/null -w "%{http_code}" http://304719.fornex.cloud/tictactoe/
```

Adding a new game requires two steps:
1. Add a `location /gamename/` block in `tictactoe.conf` pointing to the game folder
2. Add a card to `/home/projects/games/index.html`

## Nginx config

`/etc/nginx/sites-available/tictactoe.conf` — must listen on `79.132.141.127:80` (specific IP, not `0.0.0.0`) because the hosting panel's `!!default.conf` intercepts requests on the same IP.

## Git & GitHub

GitHub CLI (`gh`) is installed and authenticated as `ikrutikof` using a fine-grained personal access token (stored via `gh auth setup-git`). Token expires 2026-07-19.

Push workflow for any game:
```bash
cd /home/projects/<game>
git add index.html
git commit -m "description"
git push
```

Create a new repo for a new game:
```bash
gh repo create <name> --public
```

## Visual design system

All games share the same cyberpunk aesthetic — copy the `<style>` block conventions:
- **Colors:** `--cyan: #00f5ff`, `--magenta: #ff00a0`, `--yellow: #f5e642`, `--dark: #050510`
- **Fonts:** `Orbitron` (headings/UI), `Share Tech Mono` (body)
- **Background:** dark base + scanline `::before` + grid `::after` (both `position: fixed`, `pointer-events: none`)
- **Viewport:** always include `viewport-fit=cover` and `overflow-x: hidden; overflow-y: auto` on body for Telegram WebView compatibility
- **Safe area:** `padding-bottom: max(40px, env(safe-area-inset-bottom, 20px))` on the main container

## Tic-tac-toe (`/home/projects/site/index.html`)

**State:** `SIZE`, `WIN_LEN`, `board[]` (flat `SIZE*SIZE`), `current`, `vsAI`, `hardMode`, `scores`

**Key functions:**
- `init()` — rebuilds board DOM; call after any `SIZE`/`WIN_LEN` change
- `checkWin(b)` — scans 4 directions for `WIN_LEN` consecutive symbols
- `bestMoveHard()` — minimax + alpha-beta; depth 9 for 3×3 (perfect play), depth 4 for 5×5

## Tetris (`/home/projects/tetris/index.html`)

Canvas-based (300×600). Pieces defined as matrices in `SHAPES`, colors in `COLORS`.

**Key functions:**
- `rotate(matrix)` — pure rotation, used for all piece rotations
- `collides(mat, ox, oy)` — collision check against board and walls
- `lockPiece()` → calls `clearLines()` → calls `spawnPiece()`
- `ghostY()` — calculates hard-drop landing row for ghost preview
- Drop interval: `max(100, 800 - (level-1) * 70)` ms

## Snake Run (`/home/projects/snake/index.html`)

Canvas-based (300×540). Endless runner: snake always advances upward in world coordinates; camera follows head.

**Core model:**
- `snake[]` — array of `{x, wy}` (world Y), head-first
- `objects[]` — `{x, wy, type}` where type is `'food'` | `'soft'` | `'hard'`
- `camWY` — world Y of the top screen row; updated each tick to keep head at `ROWS-3` from top
- `inputDX` — `-1 | 0 | 1`, set by held arrow keys

**Key functions:**
- `generateAhead()` / `spawnRow(wy)` — procedurally fills objects 2 screens ahead
- `tick()` — advances snake, resolves collisions, cleans off-screen objects
- Speed: `max(80, 220 - (speedLevel-1) * 18)` ms per tick; `speedLevel` = `floor(score/150) + 1`

Soft blocks destroy on contact (score +5, brief grow). Hard blocks = death. Food = grow + score +10.
