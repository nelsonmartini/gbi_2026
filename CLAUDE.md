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
Basham, Bode, Bryce, Braden, Kyle, Evan, Jordan, Jake B, Moose, Nail, NateDawg, Neebs, Ponzi, Rusty, Shawn, Tanner, Tom, Trevor, Ty, Zach M.
- Trevor → 🇪🇬, everyone else → 🇺🇸 (set in `flag()` function)

## Teams
- Default names/colors: Team O'Hara (blue `#1a4a8a`) vs Team Bode (red `#7a1a1a`) — fully renamable/recolorable live in the UI, never hardcode a team name in new code
- Not pre-assigned — set via **Draft and Rosters** tab in the UI (snake draft or manual assignment)

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
Scoreboard → Day 1 → Day 2 → Hole Payouts → Monies → Draft and Rosters → Analytics → Format & Rules

## Key Technical Notes
- **Points math:** `rankedPts()` correctly averages tied positions. Bug was previously giving 7pts to solo 1st (should be 8).
- **Player dedup:** Selecting a player removes them from all *other slots in that same format* for the day (all 5 four-man groups dedupe against each other; all 5 2v2 matches dedupe against each other, including both slots of the same match). A player WILL legitimately appear once in a 4-Man group AND once in a 2v2 Match the same day — that's correct, not a duplicate, since every player competes in both formats daily.
- **Team badges depend on roster assignment:** `teamOf(name)` returns `null` until that player has a team set in Draft and Rosters — if badges look blank everywhere, the roster needs assigning, that's not a rendering bug.
- **2v2 teams locked:** Each side only shows players from that team per roster assignment.
- **Roster sync:** `applyRoster()` rebuilds roster from `PLAYERS` array on every load — renames in code take effect immediately.
- **Real-time sync:** Firebase `onValue` listener updates all connected devices. Own-write echoes suppressed via `CLIENT_ID` stamp.
- **Mobile:** Scoreboard hides Team column on small screens; Score to Par always visible.

## Core Integrity — Non-Negotiable, Verify After ANY Change Touching Scoring/Roster Code
The player↔team relationship and the points/money math are the entire reason this site exists — a bug here corrupts real money and real standings, silently. After touching `defaultState()`, `playerStats()`, `teamTotals()`, `rank()`/`rankedPts()`, `fourManPoints()`, `matchResults()`, `renderDay()`'s dropdown logic, or anything in `S.roster`/`S.day1`/`S.day2`:
1. **No duplicate selection within a format.** Simulate (don't just read the code) that picking a player in one 4-Man slot removes them from every other 4-Man slot that day, and same for 2v2 Match slots — see the Node `vm`-sandbox technique used in this session's transcript for how to test this without a browser.
2. **Team badges populate once rosters are assigned.** Confirm `teamOf(name)` resolves correctly for an assigned player.
3. **Math matches hand-calculation.** Run at least one scenario with a tie in each format (4-Man and 2v2) and verify the tie-averaging and win/rank point totals by hand against the code's output before trusting it.
4. **Never use a raw player name (or any other free-text value) as an object key inside `S`.** Firebase Realtime Database forbids `.` `#` `$` `/` `[` `]` in keys, and rejects the *entire* write if even one key anywhere is invalid — not just that one field. "Zach M." (the period) silently broke every save for hours on 2026-06-26 this exact way. If you need to key something by player name, sanitize through a function like `ratingKey()` (`name.replace(/[.#$/\[\]]/g, '_')`) first, or use an array instead.
5. **Any code path that calls `_ref.set()` must fail loudly, not silently.** `.catch()` alone is not enough — Firebase's `.set()` can throw *synchronously* (e.g. on an invalid key or an `undefined` value), before it ever returns a promise, so a bare `.catch()` on its return value never sees that error. Wrap the call itself in `try/catch` too. `showErrorBanner()` (near `boot()`) exists for this — use it for any new Firebase call site rather than `console.warn`, since nobody is watching the console on a phone.
- Incident history: a 2026-06-25 boot-sequence bug wiped team assignments by auto-writing blank state to Firebase on a transient empty read — fixed, see [[project-data-loss-incident]] in memory. A 2026-06-26 user report of "duplicate names" turned out to be the correct cross-format design (point 1 above) plus blank team badges from an unassigned roster, not a logic bug — verified via direct simulation, not assumption. Later that same day, a *real* incident: the Bode Matrix's `S.ratings` object used "Zach M." as a literal key, silently breaking every single save for hours (point 4 above) — see [[project-firebase-key-incident]].
