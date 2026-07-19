# SquadJS Switch Plugin

**Team-change management with database persistence, a fair switch queue, and TeamBalancer integration.**

## Overview

Manages player team-change requests with cooldown enforcement and admin restrictions. Survives server restarts via any Sequelize-compatible database (SQLite, MySQL, PostgreSQL, etc.). When teams are full, players are placed in a switch queue and served as slots open or swap partners appear, instead of being flatly denied. Responds to team scrambles by locking all players from switching, and coordinates with SmartAssign so plugin-initiated switches are correctly attributed.

## Features

* **Persistent cooldowns** — stored in the configured database, survive restarts (keyed by `eosID`)
* **Multi-database support** — works with SQLite, MySQL, PostgreSQL, and any Sequelize-compatible backend
* **Switch queue** — when the target team is full, players are queued (with pairing/solo processing) and switched automatically once eligible, rather than denied
* **Liberal / Seed mode** — relaxed switching rules (no time/cooldown limits) on Seed/Jensen-style layers
* **Dynamic balance tolerance** — optionally allow extra imbalance while the server is below full capacity, interpolated by player count
* **Scramble lockout** — blocks all players from switching for a configurable duration after a scramble
* **SmartAssign-aware attribution** — emits `PLAYER_MOVED_BY_PLUGIN` before every switch so the balancer never mistakes a plugin switch for a manual one
* **Admin global disable** — temporarily disable `!switch` for all non-admins for a bounded duration
* **Admin controls** — in-game chat and Discord

## Commands

### User
* `!switch` — request team change (checks balance, cooldowns, locks; queues if teams are full)
* `!switch check` — show your own eligibility (balance / time window / cooldown / scramble lock)
* `!switch explain` — in-game explanation of how switching works
* `!switch cancel` — leave the switch queue
* `!switch help` — in-game popup explaining eligibility

### Admin Commands (in-game)
* `!switch now {username|steamID}` — change a player's team immediately
* `!switch squad {squad_number} {team_short_name|teamID}` — change the team of all players in a squad
* `!switch double {username|steamID}` — double-switch a player (swap out and back)
* `!switch doublesquad {squad_number} {team_short_name}` — double-switch all players in a squad
* `!switch matchend {username|steamID}` — queue a player to be switched at the end of the match
* `!switch matchendsquad {squad_number} {team_short_name}` — queue a whole squad to be switched at match end
* `!switch disable {minutes}` — temporarily disable `!switch` for all non-admin players (up to `maxDisableDurationMinutes`)
* `!switch enable` — re-enable the command
* `!switch list` — view the current match-end switch queue
* `!switch wipequeue` — clear the match-end switch queue

### Admin (Discord & In-Game)
* `!switch diag` — database health + top 10 players by remaining lock time
* `!switch check <Name/SteamID>` — real-time eligibility lookup for another player
* `!switch clear <Name/SteamID>` — remove cooldowns/locks for a player
* `!switch clearall` — wipe the entire cooldown database
* `!switch help` — list all admin commands

## Configuration

```json
{
    "plugin": "Switch",
    "enabled": true,
    "commandPrefix": ["!switch", "!change"],
    "discordClient": "discord",
    "discordChannelID": "YOUR_ADMIN_CHANNEL_ID",
    "database": "sqlite",
    "switchCooldownHours": 3,
    "switchCooldownMinutes": 0,
    "switchEnabledMinutes": 5,
    "doubleSwitchCommands": ["!bug", "!stuck", "!doubleswitch"],
    "doubleSwitchCooldownHours": 0.5,
    "doubleSwitchDelaySeconds": 1,
    "doubleSwitchEnabledMinutes": 5,
    "maxUnbalancedSlots": 5,
    "switchToOldTeamAfterRejoin": false,
    "scrambleLockdownDurationMinutes": 20,
    "showDisabledTimeRemaining": true,
    "maxDisableDurationMinutes": 30,
    "liberalSwitchGameModes": ["Seed", "Jensen"],
    "liberalSwitchMaxUnbalancedSlots": 6,
    "dynamicBalanceTolerance": false,
    "dynamicBalancePlayerFloor": 90,
    "dynamicBalanceExtraSlots": 2
}
```

**Configuration Options:**

**Required:**
- `plugin` — must be `"Switch"`
- `enabled` — `true` to enable plugin
- `database` — Sequelize connector (e.g. `"sqlite"`, `"mysql"`, `"postgres"`)

**Optional:**
- `commandPrefix` — command(s) to trigger switch (string or array). Default: `["!switch", "!change"]`
- `discordClient` — Discord connector name for Discord commands. Default: none
- `discordChannelID` — Discord channel ID for admin commands and logs. Default: none

**Switch Behavior:**
- `switchCooldownHours` — hours before player can switch again. Default: `3`
- `switchCooldownMinutes` — overrides `switchCooldownHours` if set. Default: `0` (which causes `switchCooldownHours` to be used)
- `switchEnabledMinutes` — time window after match start or player join to allow switches. Default: `5`
- `maxUnbalancedSlots` — max player difference between teams to allow a switch. Default: `3`

**Double Switch:**
- `doubleSwitchCommands` — commands that swap a player to the opposite team and back (fixes bugs without changing teams). Default: `['!bug', '!stuck', '!doubleswitch']`
- `doubleSwitchCooldownHours` — cooldown for double switch commands. Default: `0.5`
- `doubleSwitchDelaySeconds` — delay between first and second swap. Default: `1`
- `doubleSwitchEnabledMinutes` — time window for double switch. Default: `5`

**Liberal / Seed Mode:**
- `liberalSwitchGameModes` — substrings for layer/gamemode names where switching rules are relaxed (no time/cooldown limits). Default: `['Seed', 'Jensen']`
- `liberalSwitchMaxUnbalancedSlots` — balance cap while in a liberal mode. Default: `6`

**Dynamic Balance Tolerance:**
- `dynamicBalanceTolerance` — enable interpolated extra imbalance tolerance when the server is below full capacity. Default: `false`
- `dynamicBalancePlayerFloor` — total player count at which maximum extra tolerance kicks in. Default: `90`
- `dynamicBalanceExtraSlots` — additional allowed imbalance slots at the floor player count (interpolated up to 98 players). Default: `2`

**Advanced:**
- `switchToOldTeamAfterRejoin` — auto-switch rejoining players to their pre-disconnect team. Default: `false`
- `scrambleLockdownDurationMinutes` — blocks all players from switching after a scramble. Default: `20`
- `showDisabledTimeRemaining` — show remaining time when switching is globally disabled by an admin. Default: `true`
- `maxDisableDurationMinutes` — maximum duration (minutes) for an admin global disable. Default: `30`

## TeamBalancer Integration

Works with **[Slacker's Team Balancer Plugin](https://github.com/mikebjoyce/squadjs-team-balancer)**:
- Listens for `TEAM_BALANCER_SCRAMBLE_EXECUTED` event
- Adds scrambled players to the database with a `scrambleLockdownExpiry` timestamp
- Prevents players from changing teams immediately following a team balancer scramble
- Emits `PLAYER_MOVED_BY_PLUGIN` before every plugin-initiated switch and listens for `SMART_ASSIGN_EVAL_START` / `SMART_ASSIGN_EVAL_END` so switches are correctly attributed and don't fight the balancer

---

## Credits

**Original Author:** [fantinodavide](https://github.com/fantinodavide)  
**Modified by:** [Slacker](https://github.com/mikebjoyce) (Discord: `real_slacker`)

This plugin is a modified fork with added persistence, a switch queue, TeamBalancer/SmartAssign integration, and expanded admin controls.
