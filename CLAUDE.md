# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Deployment

The site is served by nginx directly from `/home/projects/site/`. After editing `index.html`, changes are live immediately — no build step.

To reload nginx (only needed if `/etc/nginx/sites-available/tictactoe.conf` changes):
```bash
nginx -t && systemctl reload nginx
```

Verify the site is live:
```bash
curl -s -o /dev/null -w "%{http_code}" http://304719.fornex.cloud/
```

## Architecture

The entire project is a single self-contained file: `index.html`. It contains inline CSS and JavaScript — no external dependencies except Google Fonts.

**Game state variables** (all global in the `<script>` block):
- `SIZE` / `WIN_LEN` — board dimensions (3→3-in-a-row, 5→4-in-a-row)
- `board[]` — flat array of `SIZE*SIZE` cells, values `'X'`, `'O'`, or `null`
- `current` — whose turn (`'X'` or `'O'`)
- `vsAI`, `hardMode` — mode flags

**Key functions:**
- `init()` — rebuilds the board DOM from scratch based on current `SIZE`; always call after changing `SIZE`/`WIN_LEN`
- `checkWin(b)` — scans all 4 directions for `WIN_LEN` consecutive symbols; works for any board size
- `bestMoveEasy()` — wins if possible, blocks player win, otherwise random
- `bestMoveHard()` — minimax with alpha-beta pruning; depth 9 for 3×3 (perfect), depth 4 for 5×5 (heuristic)
- `heuristic(b)` — scores open sequences of symbols, used at depth limit in 5×5 mode

**Board rendering:** cells are created dynamically in `init()` via `document.createElement`, not hardcoded in HTML. CSS classes `.board.size-3` / `.board.size-5` control grid layout and cell sizing.

## Nginx config

`/etc/nginx/sites-available/tictactoe.conf` — binds to `79.132.141.127:80` with `server_name 304719.fornex.cloud`. Must listen on the specific IP (not `0.0.0.0`) because the hosting panel's `!!default.conf` also listens on that IP and would otherwise intercept all requests.
