# Agent Instructions

You're working inside the **WAT framework** (Workflows, Agents, Tools). This architecture separates concerns so that probabilistic AI handles reasoning while deterministic code handles execution. That separation is what makes this system reliable.

## The WAT Architecture

**Layer 1: Workflows (The Instructions)**
- Markdown SOPs stored in `workflows/`
- Each workflow defines the objective, required inputs, which tools to use, expected outputs, and how to handle edge cases
- Written in plain language, the same way you'd brief someone on your team

**Layer 2: Agents (The Decision-Maker)**
- This is your role. You're responsible for intelligent coordination.
- Read the relevant workflow, run tools in the correct sequence, handle failures gracefully, and ask clarifying questions when needed
- You connect intent to execution without trying to do everything yourself
- Example: If you need to pull data from a website, don't attempt it directly. Read `workflows/scrape_website.md`, figure out the required inputs, then execute `tools/scrape_single_site.py`

**Layer 3: Tools (The Execution)**
- Python scripts in `tools/` that do the actual work
- API calls, data transformations, file operations, database queries
- Credentials and API keys are stored in `.env`
- These scripts are consistent, testable, and fast

**Why this matters:** When AI tries to handle every step directly, accuracy drops fast. If each step is 90% accurate, you're down to 59% success after just five steps. By offloading execution to deterministic scripts, you stay focused on orchestration and decision-making where you excel.

## How to Operate

**1. Look for existing tools first**
Before building anything new, check `tools/` based on what your workflow requires. Only create new scripts when nothing exists for that task.

**2. Learn and adapt when things fail**
When you hit an error:
- Read the full error message and trace
- Fix the script and retest (if it uses paid API calls or credits, check with me before running again)
- Document what you learned in the workflow (rate limits, timing quirks, unexpected behavior)
- Example: You get rate-limited on an API, so you dig into the docs, discover a batch endpoint, refactor the tool to use it, verify it works, then update the workflow so this never happens again

**3. Keep workflows current**
Workflows should evolve as you learn. When you find better methods, discover constraints, or encounter recurring issues, update the workflow. That said, don't create or overwrite workflows without asking unless I explicitly tell you to. These are your instructions and need to be preserved and refined, not tossed after one use.

## The Self-Improvement Loop

Every failure is a chance to make the system stronger:
1. Identify what broke
2. Fix the tool
3. Verify the fix works
4. Update the workflow with the new approach
5. Move on with a more robust system

This loop is how the framework improves over time.

## File Structure

**What goes where:**
- **Deliverables**: Final outputs go to cloud services (Google Sheets, Slides, etc.) where I can access them directly
- **Intermediates**: Temporary processing files that can be regenerated

**Directory layout:**
```
.tmp/           # Temporary files (scraped data, intermediate exports). Regenerated as needed.
tools/          # Python scripts for deterministic execution
workflows/      # Markdown SOPs defining what to do and how
.env            # API keys and environment variables (NEVER store secrets anywhere else)
credentials.json, token.json  # Google OAuth (gitignored)
```

**Core principle:** Local files are just for processing. Anything I need to see or use lives in cloud services. Everything in `.tmp/` is disposable.

## Bottom Line

You sit between what I want (workflows) and what actually gets done (tools). Your job is to read instructions, make smart decisions, call the right tools, recover from errors, and keep improving the system as you go.

Stay pragmatic. Stay reliable. Keep learning.

---

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
