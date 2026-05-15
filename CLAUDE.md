# GBI 2026 — Garland Bend Invitational Tournament Site

## What it is
A Masters-themed live golf tournament scoring website for the Garland Bend Invitational 2026. 20 players, 2 teams, 2 days of competition. Single self-contained HTML file with Firebase Realtime Database for live sync across all devices.

## Files
| File | Purpose |
|---|---|
| `gbi_2026.html` | The entire app — edit this for all changes |
| `index.html` | Always a copy of gbi_2026.html — GitHub Pages serves this as root |
| `gbi-logo.jpg` | GBI golf flag + anchor logo used in header, PIN screen, favicon |

**Always copy gbi_2026.html → index.html before every commit.**

## Deploy
- **Live URL:** https://nelsonmartini.github.io/gbi_2026/
- **Repo:** https://github.com/nelsonmartini/gbi_2026 (user: nelsonmartini)
```bash
cp gbi_2026.html index.html
git add index.html gbi_2026.html
git commit -m "description"
git push https://nelsonmartini:<TOKEN>@github.com/nelsonmartini/gbi_2026.git main
```
Token needs `repo` scope — generate from GitHub → Settings → Developer settings → Personal access tokens (classic).

## Firebase Backend
- **Project:** gbi-2026
- **Database URL:** https://gbi-2026-default-rtdb.firebaseio.com
- **DB path:** gbi2026/state
- **API key** domain-restricted to `nelsonmartini.github.io/*` in GCP Console
- **Auth:** Anonymous auth enabled — DB rules require `auth != null`
- **After player name changes**, clear stale Firebase roster:
```bash
curl -X DELETE "https://gbi-2026-default-rtdb.firebaseio.com/gbi2026/state/roster.json"
```
- **Post-tournament:** Set Firebase rules `.read` and `.write` to `false`

## Site PIN
`PETERputter26` — case-insensitive. Change via `const SITE_PIN` near top of script.

## Players (20)
Basham, Bode, Bryce, Braden, Chris O, Evan, Grant, Jake B, Moose, Nail, NateDawg, Neebs, Ponzi, Rusty, Shawn, Tanner, Tom, Trevor, Ty, Zach M.
- Trevor → 🇪🇬, everyone else → 🇺🇸 (set in `flag()` function)

## Teams
- **Team O'Hara** (blue `#1a4a8a`) vs **Team Bode** (red `#7a1a1a`)
- Not pre-assigned — set via Roster & Teams tab in the UI

## Scoring
| Format | Day 1 | Day 2 |
|---|---|---|
| 4-Man 1st place | 8 pts | 10.5 pts |
| 4-Man 2nd | 6 | 8.5 |
| 4-Man 3rd | 4 | 6.5 |
| 4-Man 4th | 2 | 4.5 |
| 4-Man 5th | 1 | 2 |
| 2v2 Match Win | +3 | +5 |
| 2v2 Match Tie | +1 | +2.5 |
| 2v2 Rank 1st (of 10) | 10 | 15 |
| 2v2 Rank 10th | 1 | 3 |
| Hole payout | 2 pts / $20 | 2 pts / $20 |

## Prize Money ($2,000 total, $100/player buy-in)
1st: $400 · 2nd: $200 · 3rd: $100 · Team champs: $500 · Hole payouts: $800 (40 × $20)

## Tab Order
Scoreboard → Day 1 → Day 2 → Hole Payouts → Roster & Teams → Format & Rules

## Key Technical Notes
- **Points math:** `rankedPts()` correctly averages tied positions. Bug was previously giving 7pts to solo 1st (should be 8).
- **Player dedup:** Selecting a player in one group/match removes them from all other dropdowns for that day.
- **2v2 teams locked:** Each side only shows players from that team per roster assignment.
- **Roster sync:** `applyRoster()` rebuilds roster from `PLAYERS` array on every load — renames in code take effect immediately.
- **Real-time sync:** Firebase `onValue` listener updates all connected devices. Own-write echoes suppressed via `CLIENT_ID` stamp.
- **Mobile:** Scoreboard hides Team column on small screens; Score to Par always visible.
