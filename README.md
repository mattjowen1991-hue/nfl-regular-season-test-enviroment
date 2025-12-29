# 🏈 NFL Picks League - Repository Documentation

A fully automated NFL picks prediction league with live scoring, ESPN integration, and cloud storage.

---

## 📁 File Structure Overview

```
NFL-League/
├── index.html              # League table (public view)
├── player-picks.html       # Player submission form
├── admin-dashboard-v2.html # Admin control panel
├── players.json            # Player profiles (LOCAL ONLY)
├── config.json             # Season settings (LOCAL + JSONBin merge)
├── weeks.json              # Score history (LOCAL BACKUP)
├── picks.json              # Historical picks data (LOCAL ONLY)
├── Assets/
│   ├── [player avatars]
│   ├── [team logos]
│   └── Freshman.ttf
└── README.md               # This file
```

---

## 🔄 How Data Flows

```
┌─────────────────────────────────────────────────────────────────┐
│                        DATA FLOW DIAGRAM                         │
└─────────────────────────────────────────────────────────────────┘

    PLAYER SUBMITS PICKS              ADMIN PROCESSES SCORES
    ─────────────────────             ─────────────────────────
           │                                    │
           ▼                                    ▼
    ┌─────────────┐                    ┌─────────────────┐
    │ player-picks │                    │ admin-dashboard │
    │    .html     │                    │    -v2.html     │
    └──────┬──────┘                    └────────┬────────┘
           │                                    │
           │ Saves picks                        │ 1. Loads picks
           ▼                                    │ 2. Fetches ESPN scores
    ┌─────────────┐                             │ 3. Calculates points
    │  PICKS BIN  │◄────────────────────────────┘
    │  (JSONBin)  │         Reads picks
    │             │
    │ Resets each │                    ┌─────────────────┐
    │    week     │                    │  ESPN API       │
    └─────────────┘                    │  (Live Scores)  │
                                       └────────┬────────┘
                                                │
                                                ▼
                                       ┌─────────────────┐
                                       │  SCORES BIN     │
                                       │  (JSONBin)      │
                                       │                 │
                                       │  NEVER resets   │
                                       │  All-time data  │
                                       └────────┬────────┘
                                                │
                                                ▼
                                       ┌─────────────────┐
                                       │   index.html    │
                                       │  (League Table) │
                                       └─────────────────┘
```

---

## 📄 File Reference

### 🏠 index.html (League Table)

**Purpose:** Public-facing league standings and player stats

**Data Sources:**
| Data | Primary Source | Fallback |
|------|----------------|----------|
| Scores | JSONBin (Scores Bin) | weeks.json |
| Players | players.json | — |
| Config | JSONBin + config.json merged | config.json |
| Schedule | ESPN API (cached in localStorage) | — |

**Key Features:**
- Prize pot display with animated money rain 💰
- Weekly/Total score views
- Player stats modal with next game info
- Hall of Fame section
- Automatic ESPN schedule fetching (no manual updates needed!)

**What Gets Updated Automatically:**
- ✅ Scores (from JSONBin after you save from admin dashboard)
- ✅ Next game schedules (from ESPN API)
- ❌ Player profiles (manual edit of players.json)
- ❌ Hall of Fame (manual edit of config.json)

---

### 📝 player-picks.html (Player Submission)

**Purpose:** Players submit their weekly predictions

**Configuration (edit weekly):**
```javascript
const CONFIG = {
  JSONBIN_BIN_ID: '69487a81ae596e708fa937cb',  // Picks Bin
  JSONBIN_API_KEY: '$2a$10$...',
  
  WEEK: 18,                                     // ← Update each week
  DEADLINE: '2026-01-04T18:00:00',             // ← Update each week
  GRACE_PERIOD_MINUTES: 10,
  
  PLAYERS: {
    'matt': '2922',    // Player PINs
    'jarv': '8351',
    // ...
  },
  
  GAMES: [                                      // ← Update each week
    { home: 'Team A', away: 'Team B', doublePoints: false },
    // ...
  ]
};
```

**What It Does:**
1. Player selects name and enters PIN
2. Player picks winners for each game
3. Picks are saved to the **Picks Bin** (JSONBin)
4. Deadline enforcement with 10-minute grace period

---

### ⚙️ admin-dashboard-v2.html (Admin Panel)

**Purpose:** Process scores, set winners, save to league table

**Configuration (edit weekly):**
```javascript
const CONFIG = {
  PICKS_BIN_ID: '69487a81ae596e708fa937cb',    // Same as player-picks
  SCORES_BIN_ID: '6951c2b0ae596e708fb6c625',   // Permanent scores storage
  API_KEY: '$2a$10$...',
  ADMIN_PIN: '292292',
  
  WEEK: 18,                                     // ← Update each week
  WEEK_TYPE: 'week18',                          // ← Update each week
  
  GAMES: [                                      // ← Must match player-picks!
    { home: 'Team A', away: 'Team B', doublePoints: false },
    // ...
  ]
};
```

**Week Types:**
| Type | When to Use | Points/Game | Full House Bonus |
|------|-------------|-------------|------------------|
| `'normal'` | Regular weeks 1-17 | 1 | ✅ +2 |
| `'thanksgiving'` | Week 12.5 | 2 | ❌ No |
| `'christmas'` | Week 16.5 | 2 | ❌ No |
| `'week18'` | Final week | 2 | ✅ +2 |
| `'international'` | London/Mexico games | 1* | ✅ +2 |

*Use `doublePoints: true` on specific international games

**Buttons:**
- 🔄 **Fetch Live Scores** - Gets results from ESPN API, auto-selects winners
- 💾 **Save to League Table** - Calculates points, saves to Scores Bin
- 🔄 **Reset for New Week** - Clears Picks Bin (do AFTER saving!)

---

## 📊 JSON File Reference

### players.json

**Purpose:** Player profiles displayed on league table

**Location:** Local file only (not in JSONBin)

**Structure:**
```json
{
  "id": "matt",                              // Unique identifier (lowercase)
  "name": "Matt",                            // Display name
  "avatar": "Assets/matt.png",               // Profile picture path
  "team": "San Francisco 49ers",             // Favorite team (for Next Game)
  "teamLogo": "Assets/san-francisco-49ers.png",
  "superbowlWins": 0,                        // Your league's playoff wins
  "nflWins": 1                               // Regular season wins
}
```

**When to Edit:**
- Adding/removing players
- Updating avatars
- Changing favorite teams
- After someone wins the league (increment nflWins/superbowlWins)

**⚠️ Important:** The `id` must match the keys used in:
- `CONFIG.PLAYERS` in player-picks.html
- Score objects in weeks.json
- Picks objects in picks.json

---

### config.json

**Purpose:** Season configuration and display settings

**Location:** Local file + merged with JSONBin data

**Structure:**
```json
{
  "season": "2025",                          // Displayed on league table
  "repoUrl": "https://github.com/your/repo", // GitHub link
  "bestTeamMinPicks": 5,                     // Min picks for "Best Team" stat
  
  "highlight": {
    "winner": "matt",                        // Current/last season winner ID
    "spoon": "ste"                           // Wooden spoon holder ID
  },
  
  "honorary": [                              // Hall of Fame
    { "year": 2024, "player": "matt" },
    { "year": 2023, "player": "gaz" }
  ],
  
  "perfectWeekPoints": {                     // Max possible points per week
    "1": 10,                                 // Week 1: 8 games × 1pt + 2 bonus = 10
    "12.5": 6,                               // Thanksgiving: 3 games × 2pts = 6
    "16.5": 6,                               // Christmas: 3 games × 2pts = 6
    "17": 9                                  // Normal: 7 games × 1pt + 2 bonus = 9
  }
}
```

**When to Edit:**
- End of season (update winner, spoon, honorary)
- New season (update season year)
- Perfect week points are auto-calculated by admin dashboard, but you can manually adjust

**How perfectWeekPoints Works:**
- Used to calculate "Win Rate" and "Perfect Weeks" stats
- Auto-populated when you save from admin dashboard
- Formula: `(games × pointsPerGame) + fullHouseBonus`

---

### weeks.json

**Purpose:** Score history - LOCAL BACKUP ONLY

**Location:** Local file (primary data is in JSONBin)

**Structure:**
```json
[
  { 
    "week": 1, 
    "scores": { 
      "matt": 6, 
      "jarv": 7,
      // ... all 12 players
    } 
  },
  { 
    "week": 12.5,    // Special weeks use decimals
    "scores": { ... } 
  }
]
```

**⚠️ Important Notes:**
- This file is a **BACKUP** - the live data is in JSONBin
- The league table reads from JSONBin first, falls back to this file
- You can manually edit this to fix mistakes, but changes won't appear until JSONBin is updated

---

### picks.json

**Purpose:** Historical picks data for detailed stats

**Location:** Local file only (not automatically updated)

**Structure:**
```json
[
  {
    "week": 1,
    "games": [
      {
        "home": "Indianapolis Colts",
        "away": "Miami Dolphins",
        "winner": "Indianapolis Colts",        // Actual winner
        "picks": {
          "matt": "Indianapolis Colts",        // Each player's pick
          "jarv": "Miami Dolphins",
          // ...
        }
      }
    ]
  }
]
```

**What It's Used For:**
- "Win Rate" stat (accurate per-game %)
- "Best Team Accuracy" stat
- Historical record of who picked what

**⚠️ Current Limitation:**
This file is NOT automatically updated by the system. You would need to:
1. Export picks from JSONBin before resetting each week
2. Manually add them to this file
3. Or build an export feature (future enhancement)

**Without picks.json:**
- Win Rate uses an estimate based on weekly scores
- Best Team Accuracy shows "—"

---

## 🌐 API Integrations

### JSONBin.io (Cloud Storage)

**What It Stores:**

| Bin | ID | Contents | Reset? |
|-----|----|---------| -------|
| Picks Bin | `69487a81ae596e708fa937cb` | Current week's picks | ✅ Weekly |
| Scores Bin | `6951c2b0ae596e708fb6c625` | All-time scores + config | ❌ NEVER |

**Scores Bin Structure:**
```json
{
  "weeks": [
    { "week": 1, "scores": {...} },
    { "week": 2, "scores": {...} }
  ],
  "config": {
    "perfectWeekPoints": {
      "1": 10,
      "2": 11
    }
  }
}
```

**How Config Merging Works:**
1. League table loads `config.json` (local file)
2. League table loads Scores Bin from JSONBin
3. `perfectWeekPoints` from JSONBin is merged into local config
4. JSONBin values take precedence for any conflicts

---

### ESPN API (Live Scores & Schedules)

**Scores Endpoint:**
```
https://site.api.espn.com/apis/site/v2/sports/football/nfl/scoreboard
```
- Used by admin dashboard "Fetch Live Scores" button
- Returns current/recent game results
- Auto-detects winners for completed games

**Schedule Endpoint:**
```
https://site.api.espn.com/apis/site/v2/sports/football/nfl/teams/{teamId}/schedule
```
- Used by league table for "Next Game" display
- Fetched for each unique team in players.json
- Cached in localStorage for 1 hour
- **No manual schedule.json needed!**

**Team ID Mapping (in index.html):**
```javascript
const ESPN_TEAM_IDS = {
  "San Francisco 49ers": "25",
  "Seattle Seahawks": "26",
  "Dallas Cowboys": "6",
  // ... all 32 teams
};
```

---

## ✅ Weekly Workflow Checklist

### Before Sunday (Setup)

```
□ Update WEEK number in both files:
  - player-picks.html
  - admin-dashboard-v2.html

□ Update DEADLINE in player-picks.html:
  - Format: 'YYYY-MM-DDTHH:MM:SS'
  - Example: '2026-01-11T18:00:00'

□ Update GAMES array in both files:
  - Must be identical in both files!
  - Set doublePoints: true for international games

□ Set WEEK_TYPE in admin-dashboard-v2.html:
  - 'normal' | 'thanksgiving' | 'christmas' | 'week18'

□ Commit and push changes

□ Notify players (WhatsApp message)
```

### After Games (Sunday Night/Monday)

```
□ Open admin dashboard and log in (PIN: 292292)

□ Click "🔄 Fetch Live Scores"
  - Winners auto-populate for completed games
  - Verify all winners are correct

□ Click "💾 Save to League Table"
  - Review the confirmation modal
  - Check player scores look correct
  - Click "SAVE TO LEAGUE"

□ Verify on league table (index.html)
  - Scores updated?
  - Console shows: "✅ Loaded scores from JSONBin"

□ Click "🔄 Reset for New Week"
  - Only do this AFTER saving!
  - Clears picks bin for next week
```

---

## 📈 Scoring Reference

### Points Calculation

| Week Type | Points per Correct Pick | Full House Bonus |
|-----------|------------------------|------------------|
| Normal | 1 | +2 (all correct) |
| Thanksgiving | 2 | None |
| Christmas | 2 | None |
| Week 18 | 2 | +2 (all correct) |
| International* | 1 (or 2 for marked games) | +2 |

*Set `doublePoints: true` on specific international games

### Perfect Week Examples

| Week | Games | Calculation | Max Score |
|------|-------|-------------|-----------|
| Normal (8 games) | 8 | 8×1 + 2 | **10** |
| Normal (7 games) | 7 | 7×1 + 2 | **9** |
| Thanksgiving (3) | 3 | 3×2 + 0 | **6** |
| Christmas (3) | 3 | 3×2 + 0 | **6** |
| Week 18 (16) | 16 | 16×2 + 2 | **34** |
| With 1 London game | 7+1 | 6×1 + 1×2 + 2 | **10** |

---

## 🔧 Troubleshooting

### Scores not updating on league table

1. Check browser console for errors (F12)
2. Verify JSONBin has the data: https://jsonbin.io
3. Hard refresh the page (Cmd+Shift+R)
4. Check the Scores Bin ID matches in both files

### "Next Game" not showing

1. Check player's `team` in players.json matches exactly
2. Clear schedule cache: `localStorage.removeItem('nfl_schedule_cache')`
3. Check console for ESPN API errors
4. Verify team is in `ESPN_TEAM_IDS` mapping

### Player can't submit picks

1. Check deadline hasn't passed
2. Verify PIN matches in CONFIG.PLAYERS
3. Check DEADLINE format: `'YYYY-MM-DDTHH:MM:SS'`
4. Check Picks Bin ID is correct

### Wrong scores calculated

1. Verify WEEK_TYPE is correct
2. Check doublePoints flags on games
3. Re-save from admin dashboard with correct settings

---

## 🏆 End of Season Tasks

### After Week 18

1. **Update config.json:**
```json
{
  "highlight": {
    "winner": "winning_player_id",
    "spoon": "last_place_player_id"
  }
}
```

2. **Add to Hall of Fame:**
```json
{
  "honorary": [
    { "year": 2025, "player": "winner_id" },
    // ... previous years
  ]
}
```

3. **Update players.json:**
```json
{
  "id": "winner",
  "nflWins": 2  // Increment by 1
}
```

### Before Next Season

- Update `season` in config.json
- **DO NOT** reset the Scores Bin
- System starts fresh automatically at Week 1
- ESPN schedule will show new season games automatically

---

## 🔑 Important IDs & PINs

| Item | Value |
|------|-------|
| Admin PIN | `292292` |
| Picks Bin ID | `69487a81ae596e708fa937cb` |
| Scores Bin ID | `6951c2b0ae596e708fb6c625` |
| JSONBin API Key | `$2a$10$C7BY0Kl0u74gNn/kXO7xNuayWv493/f1jAxlHUmx3ENADQKDii61C` |

### Player PINs

| Player | PIN |
|--------|-----|
| Matt | 2922 |
| Jarv | 8351 |
| Gaz | 6194 |
| Joe | 2847 |
| Ben | 9563 |
| Coley | 3718 |
| Mark | 5926 |
| Ste | 1482 |
| Tony | 7035 |
| Jay | 4263 |
| Lee | 8914 |
| Liam | 6357 |

---

## 🚀 Future Enhancements (Ideas)

- [ ] Automatic picks.json export before weekly reset
- [ ] Email/push notifications for deadline reminders
- [ ] Head-to-head comparison feature
- [ ] Playoff bracket system
- [ ] Historical season archives
- [ ] Mobile app version

---

*Last updated: December 2024*
*System Version: 2.0 (ESPN Live Schedule Integration)*
