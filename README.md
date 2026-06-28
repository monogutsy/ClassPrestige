# ClassPrestige

<p align="center">
  <img src="https://img.shields.io/badge/TShock-6.1.0-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Terraria-1.4.5.6-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/.NET-9-purple?style=for-the-badge" />
  <img src="https://img.shields.io/badge/C%23-13-orange?style=for-the-badge" />
  <img src="https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge" />
</p>

## ClassPrestige v1.0.1

A class-based RPG progression plugin for TShock servers.

### Features

- Per-class EXP and leveling
- Prestige ranks and rebirth system
- Boss participation tracking
- AFK and farming protection
- Leaderboards
- SQLite and MySQL support
- Milestone rewards
- Custom progression titles

### Quick Install

1. Download `ClassPrestige-v1.0.1.zip`
2. Extract `ClassPrestige.dll`
3. Place it inside your `ServerPlugins` folder
4. Restart your TShock server

---

## What is this?

ClassPrestige adds an RPG-style class leveling system to your Terraria server. Kill mobs, level up four combat classes, earn prestige ranks, and eventually rebirth for permanent bonuses.

None of this touches combat stats. No damage buffs, no health boosts, no broken balance. It's purely a progression and cosmetic system meant to give players long-term goals on your server.

**The four classes:**

| Class | Detected by |
|-------|------------|
| Melee | Swords, spears, flails, yoyos |
| Ranged | Bows, guns, launchers |
| Magic | Staves, spell tomes |
| Summoner | Minion/sentry kills, summon weapons |

The weapon you hold when you kill something determines which class gets the EXP. Summoner minions always count as Summoner regardless of what you're holding.

---

## How it works

### EXP from kills

| Enemy | EXP |
|-------|-----|
| Common mob | 100 |
| Rare mob | 2,500 - 5,000 |
| Boss | 10,000 - 25,000 |

The pipeline for every kill:
```
Base EXP → bonus multiplier → diminishing returns → event reduction → award
```

All EXP also counts toward your cumulative prestige total.

### Leveling

Each class levels from 0 to 100 independently. The formula:

```
EXP needed = max(level^2 * 100, 100)
```

So level 1 needs 100 EXP, level 10 needs 10,000, and level 100 needs 1,000,000. Multi-level-ups work — if you get a fat boss reward at level 3, you might jump to level 5 in one shot.

### Prestige

Your prestige rank is based on total EXP earned across all classes combined:

| Rank | Total EXP | Bonus |
|------|-----------|-------|
| Prestige I | 500K | +2% |
| Prestige II | 1.5M | +4% |
| Prestige III | 4M | +6% |
| Prestige IV | 8M | +8% |

After Prestige IV, every additional 2.5M EXP counts as a "prestige cycle." You need 3 cycles to rebirth.

### Rebirth

Once you have 3 prestige cycles, you can rebirth. This resets everything (levels, EXP, prestige) but gives you a permanent EXP bonus that stacks across rebirths:

| Rebirth | Title | Permanent Bonus |
|---------|-------|----------------|
| 1 | (Reborn I) | +5% |
| 2 | (Reborn II) | +10% |
| 3 | (Reborn III) | +15% |
| 4 | (Ascended) | +20% |

Max combined bonus is 25% (configurable). Unlocked titles and rewards persist through rebirths.

---

## Anti-abuse

The plugin handles the obvious exploits:

- **AFK farming** — No movement for 10 minutes = no EXP. Resets when you move 2+ tiles.
- **Statue farms** — Statue-spawned NPCs give 0 EXP.
- **Spawn camping** — Same mob type in 60 seconds: 100% → 75% → 50% EXP.
- **Event grinding** — During invasions/moons, EXP is halved.
- **Boss leeching** — Need 5% of total damage to get boss EXP.

All of these are configurable or can be disabled.

---

## Commands

### Player commands (`classprestige.player`)

| Command | What it does |
|---------|-------------|
| `/level` | Show all class levels |
| `/classstats` | Detailed EXP per class |
| `/prestige` | Prestige rank and progress |
| `/rebirth` | Attempt rebirth or see requirements |
| `/expbonus` | View your bonus breakdown |
| `/rebirthinfo` | Detailed rebirth status |
| `/progression` | Explains how systems work |
| `/exptoggle` | Turn EXP notifications on/off |
| `/toplevels` | Level leaderboard |
| `/topprestige` | Prestige leaderboard |
| `/toprebirth` | Rebirth leaderboard |

### Admin commands (`classprestige.admin`)

| Command | What it does |
|---------|-------------|
| `/addexp <player> <class> <amount>` | Give EXP |
| `/setlevel <player> <class> <level>` | Set level directly |
| `/setprestige <player> <rank>` | Set prestige rank |
| `/resetplayer <player>` | Wipe all progression |
| `/reloadlevels` | Reload config without restart |

Admin commands work on offline players too — they'll load from the database.

---

## Chat titles

Players with prestige or rebirth get a title in chat:

```
(Prestige IV) Mono: hello everyone
(Ascended) Steve: gg
(Reborn II) Alex: nice boss fight
```

Only the highest title shows. The plugin overrides the TShock group prefix for players with progression titles.

---

## Permissions

| Permission | For |
|-----------|-----|
| `classprestige.player` | All player commands |
| `classprestige.admin` | Admin commands |

On first startup, `classprestige.player` is automatically added to the default group. You can disable this with `autoGrantPlayerPermission: false` in config.

---

## Setup

1. Drop `ClassPrestige.dll` in `ServerPlugins/`
2. Start TShock
3. Check console for `[ClassPrestige] Plugin initialized successfully.`
4. The config and database are created automatically
5. Players need to `/login` before commands work (that's TShock auth, not us)

To tweak settings, edit `{TShock.SavePath}/ClassPrestige/config.json` and run `/reloadlevels`.

---

## Configuration

Located at `{TShock.SavePath}/ClassPrestige/config.json`. Generated with defaults on first run.

```json
{
  "maxLevel": 100,
  "commonMobExp": 100,
  "rareMobMinExp": 2500,
  "rareMobMaxExp": 5000,
  "bossMinExp": 10000,
  "bossMaxExp": 25000,
  "bossParticipationPercent": 5,
  "prestigeThresholds": [500000, 1500000, 4000000, 8000000],
  "rebirthCyclesRequired": 3,
  "prestigeCycleExp": 2500000,
  "maxEXPBonusPercent": 25,
  "maxRebirthCount": 4,
  "afkTimeoutMinutes": 10,
  "enableStatueProtection": true,
  "enableSpawnFarmProtection": true,
  "enableEventFarmingReduction": true,
  "eventExpMultiplier": 0.5,
  "rareMobIds": [195, 471, 473, 474, 475, 85],
  "enableLeaderboards": true,
  "leaderboardTopCount": 10,
  "leaderboardRefreshMinutes": 5,
  "databaseType": "sqlite",
  "mySQLHost": "localhost",
  "mySQLDatabase": "classprestige",
  "mySQLUser": "root",
  "mySQLPassword": "",
  "autoSaveIntervalMinutes": 5,
  "milestones": {},
  "autoGrantPlayerPermission": true,
  "enableExpNotifications": true,
  "expNotificationDefaultState": true,
  "enableFancyUI": true,
  "enableItemIcons": true
}
```

Most of this is self-explanatory. A few notes:

- `prestigeThresholds` — cumulative EXP for ranks I through IV
- `prestigeCycleExp` — how much extra EXP per cycle after Prestige IV
- `databaseType` — "sqlite" (default, zero config) or "mysql"
- `milestones` — keyed by level, each maps to an array of reward objects
- `enableFancyUI` — set to false for plain text output (mobile-friendly either way)

---

## Database

**SQLite** works out of the box. File lives at `{TShock.SavePath}/ClassPrestige/classprestige.sqlite`.

**MySQL** — set `databaseType` to "mysql", fill in the connection fields, restart. Tables are created automatically. Database settings require a full restart to change (unlike gameplay config which hot-reloads).

---

## Performance

Built for 100+ players:

- Player data lives in a `ConcurrentDictionary` — lock-free reads on the kill path
- Kill processing is synchronous on cached data, no async overhead
- DB writes batch every 5 minutes (configurable), only dirty records
- Leaderboards are cached and refreshed on a timer
- AFK checks throttled to 500ms intervals
- Clean shutdown saves all records with a 30s timeout

---

## Troubleshooting

**"Your progression data could not be found"** — Player isn't logged in. They need to `/login` first.

**No EXP from kills** — Check: are they logged in? Are they AFK? Is the weapon doing positive damage? Is the NPC from a statue?

**Prestige not updating** — Should auto-fix on next EXP gain. If loading from old data, just run `/prestige` and it'll recalculate.

**Plugin won't load** — Need TShock 6.1.0 and .NET 9 runtime. Check console for error messages.

---

## Progression pacing

For reference, here's roughly how the default settings feel in practice:

| Phase | Hours | Where you'll be |
|-------|-------|----------------|
| Starting out | 2-3 | Classes around 10-15, nearing Prestige I |
| Regular player | 10-20 | Classes 25-35, Prestige II-III |
| Dedicated | 40-60 | Classes 50-60, hit Prestige IV |
| Endgame | 100+ | Classes 75+, first Rebirth |
| No-life | 300+ | Level 100s, Ascended |

---

## Changelog

### v1.0.1

- Updated plugin metadata and author information to MonoGutsy
- Fixed `/help <page>` command bug for players with prestige rank titles

### v1.0.0

- Per-class EXP and leveling (Melee, Ranged, Magic, Summoner)
- Prestige ranks I-IV with configurable thresholds (500K/1.5M/4M/8M)
- Rebirth system (Reborn I-III + Ascended) with permanent bonuses
- Anti-abuse: AFK, statues, spawn farming, event reduction
- Boss participation tracking (5% minimum contribution)
- Milestone rewards at level thresholds
- Leaderboards (levels, prestige, rebirth)
- SQLite and MySQL with auto-migration
- Auto-save with batch persistence
- Colored UI with Terraria item icons
- Chat titles: (Prestige IV), (Ascended), (Reborn II)
- EXP notifications (toggleable per-player)
- First-login tutorial
- /expbonus, /rebirthinfo, /progression commands
- Auto-permission grant on startup
- Hot-reload config via /reloadlevels
- Configurable prestige cycle EXP (2.5M default)

---

## License

MIT. See [LICENSE](LICENSE).
