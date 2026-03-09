# Battlefield 6 — Capture the Flag (Portal)

A fully custom Capture the Flag game mode built for BF6 Portal using TypeScript. Two or more teams compete to steal the enemy flag and return it to their base. First team to the target score wins.

> **Derived from open-source work by [Mystfit](https://github.com/Mystfit/BF6-CTF-Portal)**
> **Maintained by xAP3XRONINx**

---

## How It Works

### Flag Lifecycle

1. **At Home** — The flag sits at its spawn point. Opposing players can interact with it to pick it up.
2. **Being Carried** — The carrier is forced to melee weapon (sledgehammer), globally visible via an overhead icon and smoke trail, forcefully spotted on the minimap, and blocked from driving vehicles.
3. **Dropped** — When the carrier dies or switches weapons, the flag drops. A configurable pickup delay (default 5s) prevents immediate re-grab. The flag auto-returns to base after a timeout (default 30s). The owning team can return it early by interacting with it.
4. **Captured** — The carrier enters their own team's capture zone while holding an enemy flag. Their team's flag **must be at home** to score. First team to the target score wins.

### Multi-Team Support

The default classic mode is 2 teams, but a 4-team configuration is included out of the box and the system supports 3–7 teams. Each flag can optionally restrict which teams are allowed to capture it.

---

## Features

### Core Gameplay
- Classic 2-team and multi-team (3–7) CTF configurations
- Configurable target score, pickup delays, and auto-return timers
- Flag carrier restrictions: forced melee weapon, vehicle driver blocking, global spotting
- Flag-must-be-home-to-score rule
- Team auto-balancing with configurable delay and check intervals
- Manual team-switch stations near each base (interact points)

### Flag Physics & Spawning
- Projectile arc throw on flag drop with configurable speed and gravity
- Async raycast queue system for accurate ground detection
- Radial collision validation to prevent flags spawning inside geometry
- Terrain raycast bug protection (clamping Y distance)
- Smooth flag-follow mode (optional) with exponential position/rotation smoothing

### Visual & Audio Feedback
- Per-team colored smoke VFX trails on carried flags
- Spark VFX when dropped flag becomes available for pickup
- Impact VFX on flag drop
- Flag alarm SFX on steal from home
- VO voice lines for flag taken, dropped, returned, and captured events
- Pickup delay charging/ready SFX feedback loop
- Capture explosion VFX and stinger audio

### HUD & Scoreboard
- **Global HUD** (all players): team scores with bracket-style leading indicator, round timer (mm:ss), and animated flag progress bar showing flag position between bases
- **Team HUD** (per-team): real-time team orders bar driven by flag events (taken, dropped, returned, captured)
- **Player HUD** (per-player): reserved slot for future player-specific widgets
- Custom flag icon UI widget (pole + flag shape) with fill/outline modes and pulsing animation for dropped state
- Scoreboard integration with captures, assists, and carrier kills columns
- Supports both `CustomTwoTeams` and `CustomFFA` scoreboard types

### Architecture
- **WorldIcon Manager** — centralized icon lifecycle with automatic refresh on player first-deploy to fix Portal visibility bugs
- **VFX Manager** — centralized VFX tracking with toggle-based refresh for new players
- **Raycast Manager** — Promise-based async raycast queue (FIFO) wrapping `mod.RayCast` / `OnRayCastHit` / `OnRayCastMissed`, supporting concurrent raycasts
- **Animation Manager** — path-based object animation with concurrent `AsyncGenerator` support for animating along paths that are still being calculated (flag throw arcs)
- **Event Dispatcher** — generic type-safe pub/sub system used for flag lifecycle events, decoupling game logic from UI updates
- **JSPlayer** — wrapper class tracking per-player score, held flags, velocity, UI state, and join order (used for team balance priority)
- **Game Mode Config** — data-driven setup via `GameModeConfig` interface; add new team/flag layouts without touching game logic

---

## Configuration

All tuning constants are at the top of `CTF.ts`:

| Constant | Default | Description |
|---|---|---|
| `GAMEMODE_TARGET_SCORE` | `10` | Points needed to win |
| `FLAG_PICKUP_DELAY` | `5` | Seconds before a dropped flag can be picked up |
| `FLAG_AUTO_RETURN_TIME` | `30` | Seconds before a dropped flag auto-returns |
| `CARRIER_FORCED_WEAPON` | Sledgehammer | Weapon given to flag carriers |
| `TEAM_AUTO_BALANCE` | `true` | Enable automatic team balancing |
| `TEAM_BALANCE_DELAY` | `5.0` | Seconds before balance executes |
| `TEAM_BALANCE_CHECK_INTERVAL` | `10` | Seconds between balance checks |
| `VEHICLE_BLOCK_CARRIER_DRIVING` | `true` | Prevent flag carriers from driving |
| `FLAG_ENABLE_ARC_THROW` | `true` | Enable projectile arc on flag drop |
| `FLAG_THROW_SPEED` | `5` | Throw velocity (units/s) |
| `FLAG_FOLLOW_MODE` | `false` | Flag trails behind carrier instead of attaching |
| `DEBUG_MODE` | `false` | Enable verbose console logging and raycast visualization |

---

## Game Mode Configs

### Classic 2-Team CTF (`ClassicCTFConfig`)
- Teams: Purple (Team 1) vs Orange (Team 2)
- One flag and one capture zone per team
- Uses `ClassicCTFScoreHUD` with flag progress bar between scores

### 4-Team CTF (`FourTeamCTFConfig`)
- Teams: Purple, Orange, Green, Blue
- One flag and one capture zone per team
- Any opposing team can capture any other team's flag
- Uses `MultiTeamScoreHUD` with column layout

Configs are registered in the `DEFAULT_GAMEMODES` map keyed by spatial object ID. The game auto-detects which config to load based on which spatial objects exist in the Portal Builder map.

---

## Project Structure

The entire game mode lives in a single `CTF.ts` file (Portal SDK limitation). Major sections:

```
CTF.ts
├── Configuration Constants
├── Color & Math Utilities (rgba, Vec3, lerp, euler conversion)
├── Raycast Manager (async queue, projectile arcs, spawn validation)
├── Animation Manager (path animation, generator-based concurrent animation)
├── Event Dispatcher (generic pub/sub)
├── Global State & Team References
├── Main Game Loop (OnGameModeStarted, TickUpdate, SecondUpdate)
├── Event Handlers (join, leave, deploy, die, interact, vehicle, area trigger)
├── JSPlayer Class (per-player state, score, UI)
├── Scoring & Rules
├── Flag Class (full flag lifecycle, VFX, SFX, VO, events)
├── Capture Zone Class
├── WorldIcon Manager (centralized icon management with refresh)
├── VFX Manager (centralized VFX management with refresh)
├── UI System
│   ├── TickerWidget (base class with brackets, progress bars)
│   ├── ScoreTicker (team score display)
│   ├── RoundTimer (mm:ss countdown)
│   ├── FlagBar (animated flag position indicator)
│   ├── FlagIcon (custom flag shape widget)
│   ├── TeamOrdersBar (event-driven team status)
│   ├── ClassicCTFScoreHUD (2-team layout)
│   ├── MultiTeamScoreHUD (3+ team layout)
│   ├── GlobalScoreboardHUD (singleton manager)
│   ├── TeamScoreboardHUD (team-scoped)
│   └── PlayerScoreboardHUD (player-scoped)
└── Game Mode Configs (ClassicCTF, FourTeamCTF)
```

---

## Setup

### Prerequisites
- Battlefield 6 with Portal access
- BF6 Portal SDK Tool (Windows only — macOS users need a VM)
- The Portal Builder map must include spatial objects and area triggers matching the expected IDs (see `TEAM_ID_START_OFFSET` and `FlagIdOffsets` in the code)

### Portal Builder Map Requirements

Each team needs the following spatial objects placed in the Portal Builder (Godot):

| Object | ID Formula | Purpose |
|---|---|---|
| Flag Spawn | `100 + (teamId × 10) + 4` | Where the flag spawns at home |
| Capture Zone Area Trigger | `100 + (teamId × 10) + 2` | Area trigger for scoring |
| Capture Zone Spatial | `100 + (teamId × 10) + 3` | Position reference for capture zone icon |

For example, Team 1 uses IDs `114`, `112`, `113`. Team 2 uses `124`, `122`, `123`.

Game mode detection objects:
- ID `40000` → Classic 2-team CTF
- ID `40001` → 4-team CTF

### Deployment
1. Place required spatial objects and area triggers in the Portal Builder map
2. Copy `CTF.ts` into your Portal project
3. Build and deploy via the Portal SDK Tool
4. Test on PS5 or PC (note: `console.log` output is not visible on PS5)

---

## Known Limitations

- **Single file constraint**: Portal SDK requires all code in one `.ts` file
- **Portal SDK Tool is Windows-only**: macOS users need a VM or remote build setup
- **Terrain raycasts**: There is a known Portal bug where raycasts pass through terrain; `FLAG_TERRAIN_FIX_PROTECTION` provides a workaround by clamping the Y position
- **PS5 debugging**: No `console.log` output on PS5; use `mod.DisplayHighlightedWorldLogMessage` or `mod.SendErrorReport` for in-game debug output
- **WorldIcon visibility bug**: Icons created before a player deploys may be invisible; the `WorldIconManager` refresh system works around this
- **Team-scoped UI visibility**: Team-scoped widgets may not appear for newly joined players; the code tears down and rebuilds team UI on first deploy as a workaround

---

## Credits

- **Original CTF framework**: [Mystfit](https://github.com/Mystfit/BF6-CTF-Portal)
- **Game mode development & maintenance**: xAP3XRONINx

---

## License

See the original repository for license terms.
