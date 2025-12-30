# 🏈 NFL Picks League - Repository Documentation

A fully automated NFL picks prediction league with live scoring, ESPN integration, cloud storage, and comprehensive player statistics.

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
├── Assets/
│   ├── [player avatars]
│   ├── [team logos]
│   └── Freshman.ttf
└── README.md               # This file
```

---

## 🔄 How Data Flows

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           COMPLETE DATA FLOW                                 │
└─────────────────────────────────────────────────────────────────────────────┘

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
    │  PICKS BIN  │◄────────────────────────────┤
    │  (JSONBin)  │         Reads picks         │
    │             │                             │
    │ Resets each │                             │
    │    week     │                             │
    └─────────────┘                             │
                                                │
                           ┌────────────────────┼────────────────────┐
                           │                    │                    │
                           ▼                    ▼                    ▼
                  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
                  │  ESPN API       │  │  SCORES BIN     │  │ PICKS HISTORY   │
                  │  (Live Scores)  │  │  (JSONBin)      │  │     BIN         │
                  │                 │  │                 │  │  (JSONBin)      │
                  │ Auto-detects    │  │  NEVER resets   │  │                 │
                  │ game winners    │  │  All-time data  │  │  NEVER resets   │
                  └────────┬────────┘  └────────┬────────┘  │  Detailed picks │
                           │                    │           └────────┬────────┘
                           │                    │                    │
                           └──────────┬─────────┴────────────────────┘
                                      │
                                      ▼
                             ┌─────────────────┐
                             │   index.html    │
                             │  (League Table) │◄──── ESPN Schedule API
                             │                 │      (Next Game info)
                             │  - Leaderboard  │
                             │  - Player Stats │
                             │  - Prize Pot    │
                             └─────────────────┘
```

---

## 🗄️ JSONBin Storage Architecture

The system uses **three separate JSONBin bins** for different purposes:

| Bin | Purpose | Bin ID | Reset? |
|-----|---------|--------|--------|
| **Picks Bin** | Current week's player picks | `69487a81ae596e708fa937cb` | ✅ Weekly |
| **Scores Bin** | All-time weekly scores + config | `6951c2b0ae596e708fb6c625` | ❌ NEVER |
| **Picks History Bin** | Detailed pick-by-pick history | `695317c4ae596e708fb8f9ec` | ❌ NEVER |

### Picks Bin Structure
```json
{
  "week": 18,
  "picks": {
    "matt": { "0": "Team A", "1": "Team B", ... },
    "jarv": { "0": "Team B", "1": "Team A", ... }
  },
  "submissions": [
    { "player": "matt", "timestamp": "2024-01-04T17:30:00Z", "inGracePeriod": false }
  ]
}
```

### Scores Bin Structure
```json
{
  "weeks": [
    { "week": 1, "scores": { "matt": 6, "jarv": 7, ... } },
    { "week": 2, "scores": { "matt": 7, "jarv": 8, ... } }
  ],
  "config": {
    "perfectWeekPoints": {
      "1": 10,
      "2": 11,
      "12.5": 6
    }
  }
}
```

### Picks History Bin Structure
```json
{
  "weeks": [
    {
      "week": 1,
      "games": [
        {
          "home": "Indianapolis Colts",
          "away": "Miami Dolphins",
          "winner": "Indianapolis Colts",
          "doublePoints": false,
          "picks": {
            "matt": "Indianapolis Colts",
            "jarv": "Miami Dolphins"
          }
        }
      ]
    }
  ]
}
```

---

## 📄 File Reference

### 🏠 index.html (League Table)

**Purpose:** Public-facing league standings and comprehensive player stats

**Data Sources:**
| Data | Primary Source | Fallback |
|------|----------------|----------|
| Scores | JSONBin (Scores Bin) | weeks.json |
| Players | players.json | — |
| Config | JSONBin + config.json merged | config.json |
| Picks History | JSONBin (Picks History Bin) | — |
| Schedule | ESPN API (cached in localStorage) | — |

**Key Features:**
- Prize pot display with animated money rain 💰
- Weekly/Total score views with special week labels (🦃 Thanksgiving, 🎄 Christmas)
- Comprehensive player stats modal
- Hall of Fame section
- Automatic ESPN schedule fetching (no manual updates needed!)

**Player Stats Modal Features:**
| Stat | Description | Data Source |
|------|-------------|-------------|
| Win Rate 🎯 | % of correct picks | Picks History Bin (or estimated from scores) |
| Best Team Accuracy | Team picked most accurately (min 5 picks) | Picks History Bin |
| Correct Picks | Total correct / total picks | Picks History Bin |
| Most Picked Team | Team selected most often | Picks History Bin |
| Current Form | Last 5 weeks: 🟢 above avg, 🔴 below avg | Scores Bin |
| Perfect Weeks 👌🏼 | Weeks with max possible score | Scores Bin + config |
| NFL Regular Season Wins 🏆 | League championships won | players.json |
| Super Bowl Wins 🌟 | Playoff championships won | players.json |

**What Gets Updated Automatically:**
- ✅ Scores (from JSONBin after you save from admin dashboard)
- ✅ Picks history & detailed stats (from JSONBin Picks History Bin)
- ✅ Next game schedules (from ESPN API, cached 1 hour)
- ✅ Perfect week points (from JSONBin config)
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

**Purpose:** Process scores, set winners, save to league table and picks history

**Configuration (edit weekly):**
```javascript
const CONFIG = {
  // Picks storage (resets weekly)
  JSONBIN_BIN_ID: '69487a81ae596e708fa937cb',
  
  // Permanent storage (NEVER reset)
  SCORES_BIN_ID: '6951c2b0ae596e708fb6c625',
  PICKS_HISTORY_BIN_ID: '695317c4ae596e708fb8f9ec',
  
  JSONBIN_API_KEY: '$2a$10$...',
  ADMIN_PIN: '292292',
  
  WEEK: 18,                                     // ← Update each week
  WEEK_TYPE: 'week18',                          // ← Update each week
  
  GAMES: [                                      // ← Must match player-picks!
    { home: 'Team A', away: 'Team B', doublePoints: false },
    // ...
  ]
};
```

**Status Indicators:**
The admin dashboard header shows two connection status indicators:
- **JSONBin** - Shows connection to picks data (green = connected)
- **ESPN Scores API** - Shows ESPN connection when fetching scores (green = connected)

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
| Button | What It Does |
|--------|--------------|
| 🔄 **Fetch Live Scores** | Gets results from ESPN API, auto-selects winners for completed games |
| 💾 **Save to League Table** | Calculates points, saves to BOTH Scores Bin AND Picks History Bin |
| ⬇ **Download JSON** | Downloads current week data as backup |
| 🔄 **Reset for New Week** | Clears Picks Bin only (do AFTER saving!) |

**What "Save to League Table" Does:**
1. Calculates each player's score based on picks vs winners
2. Saves scores to the **Scores Bin** (`weeks` array)
3. Saves `perfectWeekPoints` for the week to config
4. Saves detailed pick-by-pick data to the **Picks History Bin**
5. All three bins update simultaneously

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
- Picks objects in Picks History Bin

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
    "1": 10,                                 // Week 1: 8 games × 1pt + 2 bonus
    "12.5": 6,                               // Thanksgiving: 3 games × 2pts
    "16.5": 6,                               // Christmas: 3 games × 2pts
    "17": 7                                  // Normal: 5 games × 1pt + 2 bonus
  }
}
```

**When to Edit:**
- End of season (update winner, spoon, honorary)
- New season (update season year)
- Perfect week points are auto-calculated by admin dashboard

**How perfectWeekPoints Works:**
- Used to calculate "Win Rate" and "Perfect Weeks" stats
- Auto-populated when you save from admin dashboard
- Formula: `(games × pointsPerGame) + fullHouseBonus`
- JSONBin values merge with and override local config values

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
- Keep this file in sync with JSONBin as a disaster recovery option

---

## 🌐 API Integrations

### JSONBin.io (Cloud Storage)

**Three bins for different purposes:**

| Bin | ID | Contents | Reset? | Used By |
|-----|----|---------| -------|---------|
| Picks Bin | `69487a81ae596e708fa937cb` | Current week's picks | ✅ Weekly | player-picks, admin |
| Scores Bin | `6951c2b0ae596e708fb6c625` | All-time scores + config | ❌ NEVER | admin, index.html |
| Picks History Bin | `695317c4ae596e708fb8f9ec` | Detailed pick history | ❌ NEVER | admin, index.html |

**How Config Merging Works:**
1. League table loads `config.json` (local file)
2. League table loads Scores Bin from JSONBin
3. `perfectWeekPoints` from JSONBin is merged into local config
4. JSONBin values take precedence for any conflicts

---

### ESPN API (Live Scores & Schedules)

**Scores Endpoint (Admin Dashboard):**
```
https://site.api.espn.com/apis/site/v2/sports/football/nfl/scoreboard?week={WEEK}
```
- Used by "Fetch Live Scores" button
- Returns current/recent game results
- Auto-detects winners for completed games
- Shows "Live" badge for in-progress games

**Schedule Endpoint (League Table):**
```
https://site.api.espn.com/apis/site/v2/sports/football/nfl/teams/{teamId}/schedule
```
- Used for "Next Game" display in player stats
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

## 📈 Player Stats Explained

The league table's player stats modal shows comprehensive statistics:

### Win Rate 🎯
- **With Picks History:** Exact % of correct picks (e.g., "89/134 = 66%")
- **Without Picks History:** Estimated from weekly scores vs perfect week points

### Best Team Accuracy
- The team you predict most accurately
- Requires minimum 5 picks of that team (`bestTeamMinPicks` in config)
- Shows team name + accuracy % (e.g., "49ers (78%)")

### Correct Picks
- Total correct picks / total picks made
- Only available when Picks History Bin has data

### Most Picked Team
- The team you select to win most often
- Shows team name + count (e.g., "Chiefs (23)")

### Current Form
- Your performance over the last 5 weeks
- 🟢 = scored at or above league average that week
- 🔴 = scored below league average that week
- Displayed oldest to newest (left to right)

### Perfect Weeks 👌🏼
- Weeks where you scored the maximum possible points
- Compared against `perfectWeekPoints` in config/JSONBin

### NFL Regular Season Wins / Super Bowl Wins
- From `nflWins` and `superbowlWins` in players.json
- Updated manually at end of season

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
  - Set doublePoints: true for special games

□ Set WEEK_TYPE in admin-dashboard-v2.html:
  - 'normal' | 'thanksgiving' | 'christmas' | 'week18'

□ Commit and push changes

□ Notify players (WhatsApp message)
```

### After Games (Sunday Night/Monday)

```
□ Open admin dashboard and log in (PIN: 292292)

□ Click "🔄 Fetch Live Scores"
  - Check both status indicators turn green
  - Winners auto-populate for completed games
  - Verify all winners are correct

□ Click "💾 Save to League Table"
  - Review the confirmation modal
  - Check player scores look correct
  - Click "SAVE TO LEAGUE"
  - Console shows: "✅ Picks history saved"

□ Verify on league table (index.html)
  - Scores updated?
  - Console shows: "✅ Loaded scores from JSONBin"
  - Console shows: "✅ Loaded picks history from JSONBin"

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
| Week 18 (6 games) | 6 | 6×2 + 2 | **14** |
| With 1 London game | 7+1 | 6×1 + 1×2 + 2 | **10** |

---

## 🔧 Troubleshooting

### Scores not updating on league table

1. Check browser console for errors (F12)
2. Verify JSONBin has the data: https://jsonbin.io
3. Hard refresh the page (Cmd+Shift+R)
4. Check the Scores Bin ID matches in both files
5. Look for: "✅ Loaded scores from JSONBin" in console

### Player stats showing "—" for some fields

1. Check if Picks History Bin has data
2. Look for: "✅ Loaded picks history from JSONBin" in console
3. Stats require picks history to be saved from admin dashboard
4. Some stats (Win Rate) fall back to estimates if no picks history

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

### ESPN API status shows error

1. Check your internet connection
2. ESPN API may be temporarily unavailable
3. You can still set winners manually using the dropdown menus
4. Try clicking "Fetch Live Scores" again after a few minutes

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
- **DO NOT** reset the Scores Bin or Picks History Bin
- System starts fresh automatically at Week 1
- ESPN schedule will show new season games automatically

---

## 🔑 Important IDs & PINs

| Item | Value |
|------|-------|
| Admin PIN | `292292` |
| Picks Bin ID | `69487a81ae596e708fa937cb` |
| Scores Bin ID | `6951c2b0ae596e708fb6c625` |
| Picks History Bin ID | `695317c4ae596e708fb8f9ec` |
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

- [ ] Email/push notifications for deadline reminders
- [ ] Head-to-head comparison feature
- [ ] Playoff bracket system
- [ ] Historical season archives
- [ ] Mobile app version
- [ ] Automatic weekly config updates

---

*Last updated: December 2024*
*System Version: 3.0 (Picks History + Enhanced Stats)*
