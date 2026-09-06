# mlbb_data

Static data repo + a single-page viewer (`index.html`) for Mobile Legends: Bang Bang
tournament rosters and per-player stat lookups. No build step, no backend — everything
is fetched client-side as plain JSON.

## Repo layout

- `data/<game_id>.json` — one cached stats profile per MLBB Game ID. This is the format
  documented below.
- `data.json` — legacy/default tournament roster (same shape as `tournaments/*.json`).
- `tournaments/<slug>.json` — one file per tournament (`event_name`, `lastupdate`, `teams[]`).
  Every file listed must also be added to `tournaments/index.json`.
- `index.html` — DataTables-based viewer. For each roster member it takes `game_id`,
  extracts the numeric ID, and fetches `data/<id>.json` to show rank/win-rate/heroes.

## `data/<game_id>.json` format

File name **must** be `<id>.json` where `<id>` matches the `id` field exactly (this is
how `index.html` looks the file up — it does `fetch('data/' + gameId + '.json')`). Don't
create extension-less files or files whose name doesn't match `id`.

### Fields actually rendered by `index.html`

These are the only fields the viewer reads (`formatChild()` in `index.html`) — treat
them as required for a profile to display correctly:

| Field | Type | Notes |
|---|---|---|
| `id` | string | MLBB Game ID, same as the filename (without `.json`) |
| `nick` | string | In-game nickname, shown instead of the roster's `game_nick` when the file exists |
| `role` | string | Free text, e.g. `"jungler"`, `"roamer"`, `"midlaner"`, `"goldlaner"`, `"explainer"`. Not a strict enum — existing data has inconsistent spelling (e.g. `"explaner"`), don't propagate typos in new files |
| `rank_current` | string | Current rank points/tier as shown by the stats source |
| `rank_highest` | string | Peak rank points/tier |
| `statistic.current_season.matches_count` | number | |
| `statistic.current_season.win_rate` | number | Percentage, e.g. `62.25` |
| `statistic.all_season.matches_count` | number | |
| `statistic.all_season.win_rate` | number | |
| `hero_favorite.current_season[]` | array | Top heroes this season, see below |
| `hero_favorite.all_season[]` | array | Top heroes all-time, see below |

Each entry in `hero_favorite.current_season` / `hero_favorite.all_season`:

| Field | Type | Notes |
|---|---|---|
| `hero_name` | string | |
| `matches` | number | |
| `win_rate` | number | Percentage |

The viewer shows at most 6 heroes per list (`renderHeroBadges(..., 6)`) — keep lists
sorted by `matches` descending, most-played first, and cap at 6 for new entries.

### Optional / extended fields (kept for future use, not yet rendered)

Present in some files, safe to include but not currently displayed anywhere:

| Field | Type | Notes |
|---|---|---|
| `server` or `zone_id` | string | Player's zone/server number. **Inconsistent in existing data** — use `zone_id` for new files |
| `region` | string | e.g. `"Indonesia"`, `"Indonesia/Jawa Barat"` |
| `guild` | string | In-game guild name |
| `organization` | string | Sponsoring org / team tag, e.g. `"Ve Voltage™"` |
| `squad_status` | string | e.g. `"Not in a squad"` |
| `achievements` | object | See below |
| `statistic.current_season` / `.all_season` extra keys | number/bool | `kda`, `team_participation`, `gold_per_min`, `dmg_per_min`, `deaths_per_match`, `turret_dmg_per_match`, and when available: `mvp_count`, `mvp_loss`, `legendary`, `maniac`, `savage`, `double_kill`, `triple_kill`, `most_kills`, `most_assists`, `first_blood`, `longest_win_streak`, `highest_dmg_per_min`, `highest_gold_per_min`, `highest_dmg_taken_per_min` |

`achievements` object shape when present:

| Field | Type | Notes |
|---|---|---|
| `total_matches` | number | |
| `total_likes` | number | |
| `current_rank` | string | e.g. `"Master"` |
| `highest_rank` | number | |
| `weekly_champion` | number | |
| `leader_of_men` | boolean | |
| `best_teammate` | boolean | |
| `guardian_level` | number | |
| `renowned_collector` | string | e.g. `"IV"` |

### Full template

Use this as the starting point for a new `data/<game_id>.json`. Required fields are
uncommented; optional ones are marked — remove any you don't have data for (don't emit
`null`/empty placeholders for fields you don't know).

```json
{
  "id": "123456789",
  "nick": "PlayerNickname",
  "zone_id": "2001",
  "role": "jungler",
  "rank_current": "50",
  "rank_highest": "80",
  "region": "Indonesia",
  "guild": "Guild Name",
  "organization": "Team Tag",
  "squad_status": "Not in a squad",
  "statistic": {
    "current_season": {
      "matches_count": 100,
      "win_rate": 55.0,
      "kda": 3.5,
      "team_participation": 55.0,
      "gold_per_min": 600,
      "dmg_per_min": 3200,
      "deaths_per_match": 4.0,
      "turret_dmg_per_match": 3000
    },
    "all_season": {
      "matches_count": 5000,
      "win_rate": 54.0,
      "kda": 3.4,
      "team_participation": 54.0,
      "gold_per_min": 590,
      "dmg_per_min": 3000,
      "deaths_per_match": 4.1,
      "turret_dmg_per_match": 2900
    }
  },
  "hero_favorite": {
    "current_season": [
      { "hero_name": "HeroA", "matches": 50, "win_rate": 60.0 },
      { "hero_name": "HeroB", "matches": 30, "win_rate": 55.0 }
    ],
    "all_season": [
      { "hero_name": "HeroC", "matches": 800, "win_rate": 58.0 },
      { "hero_name": "HeroD", "matches": 600, "win_rate": 52.0 }
    ]
  },
  "achievements": {
    "total_matches": 5000,
    "total_likes": 1000,
    "current_rank": "Mythic",
    "highest_rank": 80,
    "weekly_champion": 1,
    "leader_of_men": true,
    "best_teammate": false,
    "guardian_level": 3,
    "renowned_collector": "II"
  }
}
```

## `tournaments/<slug>.json` roster format (for reference)

Each team member entry links to a `data/<game_id>.json` file via `game_id` (the viewer
extracts the first run of digits from this field, so `"43936957 (2066)"` resolves to
`data/43936957.json`):

```json
{
  "event_name": "Tournament Name",
  "lastupdate": "YYYY-MM-DD HH:MM:SS",
  "teams": [
    {
      "team_name": "Team Name",
      "captain_name": "Captain Name",
      "captain_whatsapp": "08xxxxxxxxxx",
      "captain_email": "captain@example.com",
      "logo": false,
      "idcard": false,
      "members": [
        {
          "full_name": "Player Full Name",
          "nip": "employee/vendor ID",
          "join_from": null,
          "game_id": "43936957 (2066)",
          "game_nick": "InGameNick"
        }
      ]
    }
  ]
}
```

Adding a new tournament file requires also appending its filename to
`tournaments/index.json`.
