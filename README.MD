# SquadJS Switch Plugin

**Team-change management with database persistence and TeamBalancer integration.**

## Overview

Manages player team-change requests with cooldown enforcement and admin restrictions. Survives server restarts via database. Responds to team scrambles by locking all players from switching.

## Features

* **Persistent cooldowns** — stored in database to survive restarts
* **Scramble lockout** — blocks all players from switching for configurable duration after scramble
* **Admin controls** — in-game chat and Discord

## Commands

### User
* `!switch` — request team change (checks balance, cooldowns, locks)
* `!switch help` — in-game warning popup explaining eligibility

### Admin Commands
* `!switch now {username}` — Changes team of a player by its username
* `!switch now {steamID}` — Changes team of a player by its SteamID64
* `!switch squad {squad_number} {team_short_name}` — Changes team of all the players of a squad by squad number and team short name
* `!switch squad {squad_number} {teamID}` — Changes team of all the players of a squad by squad number and team ID
* `!switch double {username}` — Double switches a player by its username
* `!switch double {steamID}` — Double switches a player by its SteamID64
* `!switch matchend {username}` — Switches a player by its username, at the end of the match
* `!switch matchend {steamID}` — Switches a player by its SteamID64, at the end of the match
* `!switch doublesquad {squad_number} {team_short_name}` — Double switches all the players of a squad by squad number and team short name
* `!switch matchendsquad {squad_number} {team_short_name}` — Switches all the players of a squad at the end of the match
* `!switch disable {minutes}` — Temporarily disable command for all non admin players
* `!switch enable` — Re-enable command
* `!switch list` — view current switch match end queue
* `!switch wipequeue` — clear match end queue

### Admin (Discord & In-Game)
* `!switch diag` — database health + top 10 players by remaining lock time
* `!switch check <Name/SteamID>` — real-time eligibility lookup
* `!switch clear <Name/SteamID>` — remove cooldowns/locks for player
* `!switch clearall` — wipe entire cooldown database
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
    "maxDisableDurationMinutes": 30
}
```

**Configuration Options:**

**Required:**
- `plugin` — must be `"Switch"`
- `enabled` — `true` to enable plugin
- `database` — Sequelize connector (typically `"sqlite" or "mysql"`)

**Optional:**
- `commandPrefix` — command(s) to trigger switch (string or array). Default: `["!switch", "!change"]`
- `discordClient` — Discord connector name for Discord commands. Default: none
- `discordChannelID` — Discord channel ID for admin commands and logs. Default: none

**Switch Behavior:**
- `switchCooldownHours` — hours before player can switch again. Default: `3`
- `switchCooldownMinutes` — overrides `switchCooldownHours` if set. Default: `0` (which causes `switchCooldownHours` to be used)
- `switchEnabledMinutes` — time window after match start or player join to allow switches. Default: `5`
- `maxUnbalancedSlots` — max player difference between teams to allow switch. Default: `3`

**Double Switch:**
- `doubleSwitchCommands` — commands that swap player to opposite team and back (fixes bugs without changing teams). Default: `['!bug', '!stuck', '!doubleswitch']`
- `doubleSwitchCooldownHours` — cooldown for double switch commands. Default: `0.5`
- `doubleSwitchDelaySeconds` — delay between first and second swap. Default: `1`
- `doubleSwitchEnabledMinutes` — time window for double switch. Default: `5`

**Advanced:**
- `switchToOldTeamAfterRejoin` — auto-switch rejoining players to their pre-disconnect team. Default: `false`
- `scrambleLockdownDurationMinutes` — blocks all players from switching after scramble. Default: `20`
- `showDisabledTimeRemaining` — show remaining time when switch is globally disabled. Default: `true`
- `maxDisableDurationMinutes` — maximum duration (minutes) for global disable. Default: `30`

## TeamBalancer Integration

Works with **[Slacker's Team Balancer Plugin](https://github.com/mikebjoyce/squadjs-team-balancer)**:
- Listens for `TEAM_BALANCER_SCRAMBLE_EXECUTED` event
- Adds scrambled players to database with `scrambleLockdownExpiry` timestamp
- Prevents players from changing teams immediately following a team balancer scramble

---

## Credits

**Original Author:** [fantinodavide](https://github.com/fantinodavide)  
**Modified by:** [Slacker](https://github.com/mikebjoyce) (Discord: `real_slacker`)

This plugin is a modified fork with added persistence, TeamBalancer integration, and expanded admin controls.
