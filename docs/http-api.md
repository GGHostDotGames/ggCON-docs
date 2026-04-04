# HTTP API

All endpoints return `Content-Type: application/json; charset=utf-8`.

## Authentication

Authenticated endpoints require **both** of the following:

- The client's IP address must be in `AllowedIPs` or `AllowedCIDRs` (unless `AllowAllIPs = true`)
- If `RequirePassword = true`, the request must include the header `X-Password: <password>`

A failed auth check returns HTTP `401`. The `reason` field describes which check failed:

```json
{
  "ok": false,
  "error": "unauthorized",
  "reason": "IP address not in allowlist"
}
```

```json
{
  "ok": false,
  "error": "unauthorized",
  "reason": "Authentication failed"
}
```

!!! warning "Brute-force lockout"
    Five failed authentication attempts within 60 seconds locks the IP out for 5 minutes. All requests from that IP return `401` during the lockout window.

---

## GET /health

Returns the mod's running status. **No authentication required.**

**Response**

```json
{
  "ok": true,
  "mod": "ggCON",
  "version": "0.12.2",
  "build": "2026-04-04 10:15:00",
  "service": "http",
  "running": true
}
```

---

## GET /help

Returns all available HTTP routes and RCON commands. Auth required.

**Response**

```json
{
  "ok": true,
  "rcon": [
    { "command": "#Help", "description": "Show this help" },
    { "command": "#ReloadConfig", "description": "Reload all configuration without restarting the server" }
  ],
  "http": [
    { "method": "GET", "path": "/health", "auth": false, "description": "Mod health check" },
    { "method": "GET", "path": "/help",   "auth": true,  "description": "This help document" },
    { "method": "GET", "path": "/players.json", "auth": true, "description": "List all online players" }
  ]
}
```

---

## GET /players.json

Returns the current player list. Auth required.

**Response**

```json
{
  "ok": true,
  "count": 1,
  "players": [
    {
      "characterName": "John Smith",
      "steamName": "jsmith99",
      "userId": "76561198000000001",
      "fame": 1500,
      "accountBalance": 2500,
      "goldBalance": 10,
      "location": { "x": 12345.6, "y": 67890.1, "z": 200.0 },
      "isAdmin": false,
      "fameLevel": 3,
      "ping": 42,
      "isGodMode": false,
      "isImmortal": false,
      "hasInfiniteAmmo": false,
      "isSuperJumpEnabled": false,
      "gearWeightKg": 14.2,
      "ipAddress": "198.51.100.25",
      "gender": "male",
      "itemInHands": "1H_KitchenKnife_02_Deluxe",
      "velocity": { "x": 293.6, "y": -196.1, "z": 0 },
      "health": 1.0,
      "bodyEffects": [
        { "name": "Exhaustion", "severity": 0, "maxSeverity": 2 }
      ],
      "squad": { "name": "Alpha Squad", "members": 4 },
      "attributes": {
        "strength": 5.0,
        "constitution": 5.0,
        "dexterity": 5.0,
        "intelligence": 5.0
      },
      "skills": [
        { "id": 1, "name": "Rifles", "xp": 450, "level": 2, "levelName": "Medium" },
        { "id": 2, "name": "Sniper Rifles", "xp": 120, "level": 1, "levelName": "Basic" }
      ]
    }
  ]
}
```

**Player fields**

| Field | Type | Description |
|---|---|---|
| `characterName` | string | SCUM in-game character name |
| `steamName` | string | Steam display name |
| `userId` | string | Steam ID (64-bit) |
| `fame` | number \| null | Fame points |
| `accountBalance` | number \| null | In-game currency balance |
| `goldBalance` | number \| null | Gold balance |
| `location` | object \| null | World position `{x, y, z}` in Unreal units |
| `isAdmin` | boolean \| null | Whether the player has admin status |
| `fameLevel` | number \| null | Fame tier |
| `ping` | number \| null | Network ping in ms |
| `isGodMode` | boolean \| null | God mode active |
| `isImmortal` | boolean \| null | Immortal flag active |
| `hasInfiniteAmmo` | boolean \| null | Infinite ammo active |
| `isSuperJumpEnabled` | boolean \| null | Super jump active |
| `gearWeightKg` | number \| null | Current gear weight in kg |
| `ipAddress` | string \| null | Player's IP address |
| `gender` | string \| null | `"male"`, `"female"`, or `"unknown"` |
| `itemInHands` | string \| null | Class name of the item currently held, or `null` if empty hands |
| `velocity` | object \| null | Movement velocity `{x, y, z}` in cm/s. Divide by 100 for m/s |
| `health` | number \| null | Health as a fraction (0.0 = dead, 1.0 = full health) |
| `bodyEffects` | array | Active body conditions. Each entry has `name` (string), `severity` (number), and `maxSeverity` (number) |
| `squad` | object \| null | Squad info `{name, members}`, or `null` if not in a squad |
| `attributes` | object \| null | Physical attributes `{strength, constitution, dexterity, intelligence}` as decimal values |
| `skills` | array | Character skills. Each entry has `id`, `name`, `xp`, `level` (0–4), and `levelName` |

**Skill levels**

| Level | Name |
|---|---|
| 0 | None |
| 1 | Basic |
| 2 | Medium |
| 3 | Advanced |
| 4 | Above Advanced |

Fields that cannot be read from the game return `null`.

---

## GET /players/{steamId}.json

Returns data for a single player by Steam ID. Auth required.

**Response — player found**

Same fields as the player objects in `/players.json`, wrapped in a top-level object with `"ok": true`.

**Response — player not found**

HTTP `404`:

```json
{
  "ok": false,
  "error": "not found"
}
```

---

## GET /players/all.json

Returns all players who have ever connected, including offline players. Auth required.

Player data is loaded from the SCUM database at startup and enriched with live data for players who are currently online.

**Query parameters**

| Parameter | Required | Description |
|---|---|---|
| `search` | No | Search string — matches against character name or Steam ID (case-insensitive, min 2 characters) |
| `page` | No | Page number (default: 1). Each page returns up to 50 results |

**Response**

```json
{
  "ok": true,
  "count": 2,
  "total": 847,
  "page": 1,
  "players": [
    {
      "characterName": "John Smith",
      "userId": "76561198000000001",
      "online": true,
      "fame": 1500,
      "accountBalance": 2500,
      "goldBalance": 10,
      "lastLogin": "2026-04-01T14:22:08Z",
      "lastLogout": null,
      "location": { "x": 12345.6, "y": 67890.1, "z": 200.0 }
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `count` | number | Number of players in this response |
| `total` | number | Total matching players across all pages |
| `page` | number | Current page number |
| `players[].online` | boolean | Whether the player is currently connected |
| `players[].lastLogin` | string \| null | ISO 8601 timestamp of last login |
| `players[].lastLogout` | string \| null | ISO 8601 timestamp of last logout |

Online players include the full set of fields from `/players.json`. Offline players include database fields only (name, Steam ID, fame, balance, gold, last login/logout, last known location).

---

## GET /server.json

Returns server state. Auth required.

**Response**

```json
{
  "ok": true,
  "online": true,
  "modVersion": "0.12.2",
  "modBuild": "2026-04-03 14:30:00",
  "scumVersion": "1.2.2.1.108938+0",
  "onlinePlayers": 12,
  "timeOfDay": 14.5,
  "timezoneOffsetMin": -300,
  "fps": 30.4,
  "avgFps": 30.3,
  "minFps": 28.1
}
```

| Field | Type | Description |
|---|---|---|
| `online` | boolean | Whether the server world is currently active |
| `modVersion` | string | ggCON version |
| `modBuild` | string \| null | Build timestamp (`YYYY-MM-DD HH:MM:SS`) |
| `scumVersion` | string \| null | SCUM server version |
| `onlinePlayers` | number | Number of players currently online |
| `timeOfDay` | number \| null | Time of day in hours (0–24) |
| `timezoneOffsetMin` | number \| null | Server timezone offset from UTC in minutes (e.g., `-300` for UTC-5) |
| `fps` | number \| null | Current server FPS |
| `avgFps` | number \| null | Average FPS over the sampling window |
| `minFps` | number \| null | Minimum FPS over the sampling window |

---

## GET /weather.json

Returns live weather and environment data. Auth required.

**Response**

```json
{
  "ok": true,
  "timeOfDay": 15.32,
  "sunriseTime": 6,
  "sunsetTime": 21,
  "rainIntensity": 0,
  "snowIntensity": 0,
  "windAzimuth": 307.169,
  "windIntensity": 0.025170,
  "windSpeedKph": 5.03,
  "airTemperature": 34.94,
  "waterTemperature": 25,
  "humidity": 0.025,
  "fogDensity": 0.38,
  "lightningRate": 0,
  "nighttimeDarkness": 0,
  "cirrostratusCoverage": 0,
  "cumulonimbusCoverage": 0.12,
  "nimbostratusCoverage": 0
}
```

Fields that cannot be read from the game return `null`.

| Field | Type | Description |
|---|---|---|
| `timeOfDay` | number \| null | Time of day in hours (0–24) |
| `sunriseTime` | number \| null | Sunrise hour |
| `sunsetTime` | number \| null | Sunset hour |
| `rainIntensity` | number \| null | Rain intensity (0–1) |
| `snowIntensity` | number \| null | Snow intensity (0–1) |
| `windAzimuth` | number \| null | Wind direction in degrees (0–360) |
| `windIntensity` | number \| null | Wind intensity (0–1) |
| `windSpeedKph` | number \| null | Wind speed in km/h |
| `airTemperature` | number \| null | Air temperature in °C |
| `waterTemperature` | number \| null | Water temperature in °C |
| `humidity` | number \| null | Relative humidity (0–1) |
| `fogDensity` | number \| null | Fog density (0–1) |
| `lightningRate` | number \| null | Lightning rate |
| `nighttimeDarkness` | number \| null | Nighttime darkness modifier |
| `cirrostratusCoverage` | number \| null | Cirrostratus cloud coverage (0–1) |
| `cumulonimbusCoverage` | number \| null | Cumulonimbus cloud coverage (0–1) |
| `nimbostratusCoverage` | number \| null | Nimbostratus cloud coverage (0–1) |

---

## GET /vehicles.json

Returns all spawned vehicles on the server. Auth required.

Vehicle data comes from the server database and is enriched with live data for vehicles near players. Ownership information is resolved when at least one player is online.

**Response**

```json
{
  "ok": true,
  "count": 268,
  "ownershipResolved": true,
  "vehicles": [
    {
      "id": 71805135,
      "class": "Barba",
      "name": "Barba",
      "location": { "x": -507248.0, "y": 669942.0, "z": 0.0 },
      "rendered": false,
      "owner": "PlayerName",
      "ownerSteamId": "76561198000000001",
      "spawnDate": "2026-03-15T14:22:08Z"
    },
    {
      "id": 71807441,
      "class": "Barba",
      "name": "Barba",
      "location": { "x": 477566.0, "y": -441884.0, "z": 0.0 },
      "rendered": true,
      "owner": null,
      "spawnDate": "2026-03-10T09:45:31Z"
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `count` | number | Total number of vehicles |
| `ownershipResolved` | boolean | Whether ownership data has been resolved (requires a player online) |
| `vehicles[].id` | number | Vehicle entity ID |
| `vehicles[].class` | string | Vehicle class (e.g., "Barba", "Laika", "Rager") |
| `vehicles[].name` | string | Display name |
| `vehicles[].location` | object \| null | World position `{x, y, z}` |
| `vehicles[].rendered` | boolean | `true` if the vehicle is currently loaded in the game world (live position data), `false` if position is from the database |
| `vehicles[].owner` | string \| null | Owner's character name, or `null` if unowned |
| `vehicles[].ownerSteamId` | string | Owner's Steam ID (present only when owner is known) |
| `vehicles[].spawnDate` | string \| null | ISO 8601 timestamp of when the vehicle was spawned |

---

## POST /vehicles/refresh

Forces a full vehicle cache rebuild. Auth required.

**Response — success**

```json
{
  "ok": true,
  "message": "Vehicle data refreshed."
}
```

**Response — partial (no player online)**

```json
{
  "ok": false,
  "message": "Partial refresh — no player online. Database data updated."
}
```

---

## GET /squads.json

Returns all squads on the server. Auth required.

Squad data is loaded from the server database and enriched with live online status for members who are currently connected.

**Response**

```json
{
  "ok": true,
  "count": 3,
  "squads": [
    {
      "id": 42,
      "name": "Alpha Squad",
      "message": "Recruiting!",
      "information": "PvP squad based in B3",
      "score": 15400,
      "memberLimit": 10,
      "pendingCount": 2,
      "hasOnlineMembers": true,
      "members": [
        {
          "profileId": 8,
          "characterName": "John Smith",
          "steamId": "76561198000000001",
          "rank": 4,
          "rankName": "Boss",
          "online": true,
          "isAlive": true,
          "inDanger": false
        }
      ]
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `squads[].id` | number | Squad ID |
| `squads[].name` | string | Squad name |
| `squads[].message` | string | Squad message / recruitment text |
| `squads[].information` | string | Squad description |
| `squads[].score` | number | Squad score (live value when members are online) |
| `squads[].memberLimit` | number | Maximum member count |
| `squads[].pendingCount` | number | Pending join requests |
| `squads[].hasOnlineMembers` | boolean | Whether any member is currently online |
| `members[].profileId` | number | Player profile ID |
| `members[].characterName` | string | Character name |
| `members[].steamId` | string | Steam ID |
| `members[].rank` | number | Rank value (0–4) |
| `members[].rankName` | string | `"None"`, `"Member"`, `"Enforcer"`, `"Underboss"`, or `"Boss"` |
| `members[].online` | boolean | Currently connected to the server |
| `members[].isAlive` | boolean | Whether the member is alive (always `false` when offline) |
| `members[].inDanger` | boolean | Whether the member is in danger (always `false` when offline) |

---

## POST /squads/refresh

Forces a squad data reload from the database. Auth required.

**Response**

```json
{ "ok": true }
```

---

## POST /squads/remove-member

Removes a player from their squad. Auth required.

**Request body**

```json
{ "profileId": 8 }
```

**Response**

```json
{ "ok": true }
```

---

## POST /squads/delete

Deletes a squad by removing all its members. Auth required.

**Request body**

```json
{ "squadId": 42 }
```

**Response**

```json
{ "ok": true }
```

---

## GET /flags.json

Returns all base building flags with owner info, location, and element counts. Auth required.

Flag data auto-refreshes every 30 seconds. Read-only database access — does not block the game server.

**Response**

```json
{
  "ok": true,
  "count": 2,
  "flags": [
    {
      "flagId": 1,
      "baseId": 1,
      "baseName": "Base #1",
      "owner": "PlayerName",
      "ownerSteamId": "76561198000000004",
      "ownerProfileId": 2,
      "location": { "x": -253574, "y": 443844, "z": 91506.7 },
      "elementCount": 17,
      "expandedElements": 0,
      "maxElements": 100
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `count` | number | Total number of flags |
| `flags[].flagId` | number | Flag ID |
| `flags[].baseId` | number | Base ID |
| `flags[].baseName` | string | Base name |
| `flags[].owner` | string | Owner's character name |
| `flags[].ownerSteamId` | string | Owner's Steam ID |
| `flags[].ownerProfileId` | number | Owner's profile ID |
| `flags[].location` | object | World position `{x, y, z}` in Unreal units |
| `flags[].elementCount` | number | Current number of building elements |
| `flags[].expandedElements` | number | Number of expanded elements |
| `flags[].maxElements` | number | Maximum allowed elements |

---

## POST /flags/refresh

Forces an immediate refresh of flag data. Auth required.

**Response**

```json
{ "ok": true }
```

---

## GET /items.json

Returns the full item catalog (6,000+ items) with categories and subcategories. Auth required.

This is the same catalog used by the panel's Give Item feature.

**Response**

```json
{
  "v": 2,
  "source": "engine",
  "n": 2585,
  "items": [
    {
      "i": "Backpack_Tactical_01",
      "ico": "ICO_Backpack_Tactical_01",
      "c": "Equipment"
    },
    {
      "i": "Knife_Hunting",
      "c": "Weapons"
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `v` | number | Catalog format version |
| `source` | string | Data source (`"engine"` = queried from the running game) |
| `n` | number | Total number of items |
| `items[].i` | string | Item class name (use with `POST /spawn`) |
| `items[].ico` | string \| absent | Icon asset name (present when an icon mapping exists) |
| `items[].c` | string \| absent | Item category (e.g., `"Weapons"`, `"Ammunition"`, `"Food"`, `"Equipment"`) |

---

## GET /vehicle-types.json

Returns the catalog of all vehicle types available for spawning. Auth required.

**Response**

```json
{
  "v": 1,
  "source": "engine",
  "n": 19,
  "type": "Vehicle",
  "items": [
    { "i": "BPC_Barba", "ico": "ICO_BPC_Barba" },
    { "i": "BPC_Rager", "ico": "ICO_BPC_Rager" },
    { "i": "BPC_Laika", "ico": "ICO_BPC_Laika" }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `v` | number | Catalog format version |
| `source` | string | Data source (`"engine"` = queried from the running game) |
| `n` | number | Total number of vehicle types |
| `items[].i` | string | Vehicle class name (use with `POST /spawn-vehicle`) |
| `items[].ico` | string \| absent | Icon asset name |

---

## GET /zombies.json

Returns the catalog of all zombie variants available for spawning. Auth required.

**Response**

```json
{
  "v": 1,
  "source": "engine",
  "n": 30,
  "type": "Zombie",
  "items": [
    { "i": "BP_Zombie_Military_Normal_Male" },
    { "i": "BP_Zombie_Civilian_Normal_Female" }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `items[].i` | string | Zombie variant name (use with `POST /spawn-entity`, verb `SpawnZombie`) |

---

## GET /animals.json

Returns the catalog of all animal types available for spawning. Auth required.

**Response**

```json
{
  "v": 1,
  "source": "engine",
  "n": 14,
  "type": "Animal",
  "items": [
    { "i": "BP_Bear" },
    { "i": "BP_Boar" },
    { "i": "BP_Deer" }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `items[].i` | string | Animal type name (use with `POST /spawn-entity`, verb `SpawnAnimal`) |

---

## GET /armed-npcs.json

Returns the catalog of all armed NPC types available for spawning. Auth required.

**Response**

```json
{
  "v": 1,
  "source": "engine",
  "n": 20,
  "type": "ArmedNPC",
  "items": [
    { "i": "BP_ArmedNPC_Soldier_01" },
    { "i": "BP_ArmedNPC_Hunter_01" }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `items[].i` | string | Armed NPC type name (use with `POST /spawn-entity`, verb `SpawnArmedNPC`) |

---

## GET /commands.json

Returns the admin command reference, built from the game engine at startup. Auth required.

**Response**

```json
{
  "ok": true,
  "source": "reflected",
  "total": 214,
  "commands": [
    {
      "verb": "Teleport",
      "description": "Teleport player to coordinates",
      "args": [
        { "name": "SteamId", "type": "string", "required": true },
        { "name": "X", "type": "float", "required": true },
        { "name": "Y", "type": "float", "required": true },
        { "name": "Z", "type": "float", "required": true }
      ]
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `source` | string | `"reflected"` = built from the running game engine |
| `total` | number | Total number of admin commands |
| `commands[].verb` | string | Command name (used after `#` in commands) |
| `commands[].description` | string | Human-readable description |
| `commands[].args` | array | Argument definitions with name, type, and required flag |

---

## POST /spawn

Spawns item(s) for a player. Auth required.

**Request body**

```json
{
  "steamId": "76561198000000001",
  "item": "Backpack_Tactical_01",
  "qty": 1
}
```

| Field | Required | Description |
|---|---|---|
| `steamId` | Yes | Target player's Steam ID |
| `item` | Yes | Item class name (from `GET /items.json`) |
| `qty` | No | Quantity to spawn (default: 1, minimum: 1) |

**Response**

```json
{
  "ok": true,
  "accepted": true,
  "dispatched": true,
  "command": "#GiveItem 76561198000000001 Backpack_Tactical_01 1"
}
```

---

## POST /spawn-vehicle

Spawns a vehicle near a specific player. Auth required.

**Request body**

```json
{
  "steamId": "76561198000000001",
  "vehicle": "BPC_Rager"
}
```

| Field | Required | Description |
|---|---|---|
| `steamId` | Yes | Target player's Steam ID (player must be online) |
| `vehicle` | Yes | Vehicle class name (from `GET /vehicle-types.json`) |

**Response**

```json
{
  "ok": true,
  "accepted": true,
  "dispatched": true,
  "command": "#GiveVehicle 76561198000000001 BPC_Rager"
}
```

---

## POST /spawn-entity

Spawns a zombie, animal, armed NPC, Brenner, or Razor near a specific player. Auth required.

**Request body**

```json
{
  "steamId": "76561198000000001",
  "verb": "SpawnZombie",
  "entity": "BP_Zombie_Military_Normal_Male"
}
```

| Field | Required | Description |
|---|---|---|
| `steamId` | Yes | Target player's Steam ID (player must be online) |
| `verb` | Yes | Spawn type: `SpawnZombie`, `SpawnAnimal`, `SpawnArmedNPC`, `SpawnBrenner`, or `SpawnRazor` |
| `entity` | No | Entity variant name (from the corresponding catalog endpoint). Not needed for `SpawnBrenner` and `SpawnRazor` |

**Response**

```json
{
  "ok": true,
  "accepted": true,
  "dispatched": true,
  "command": "#SpawnEntity 76561198000000001 SpawnZombie BP_Zombie_Military_Normal_Male"
}
```

---

## GET /logs

Returns real-time log lines from SCUM server log files. Auth required.

See [Log Watcher](log-watcher.md) for full documentation, configuration, and usage examples.

**Query parameters:** `since` (Unix ms timestamp), `sources` (comma-separated filter).

**Response**

```json
{
  "ok": true,
  "lines": [
    { "t": 1710612345123, "src": "chat", "line": "2026.03.16-18.22.07: ..." }
  ],
  "next": 1710612345124
}
```

---

## POST /command

Dispatches a SCUM admin command and returns its output. Auth required.

**Request body**

```json
{ "command": "#ListPlayers" }
```

The leading `#` is optional for ggCON-specific commands but required for native SCUM commands.

**Response — success**

```json
{
  "ok": true,
  "accepted": true,
  "dispatched": true,
  "command": "#ListPlayers",
  "lines": [
    "Player list (2):",
    "76561198000000001 John Smith",
    "76561198000000002 Jane Doe"
  ]
}
```

**Response — error**

```json
{
  "ok": false,
  "accepted": true,
  "dispatched": false,
  "command": "#ListPlayers",
  "message": "no admin player is online"
}
```

**Response fields**

| Field | Type | Description |
|---|---|---|
| `ok` | boolean | `true` if the command executed successfully |
| `accepted` | boolean | `true` if the command passed policy checks |
| `dispatched` | boolean | `true` if the command reached the game |
| `command` | string | The command as received |
| `lines` | string[] | Output lines captured from the game (omitted if empty) |
| `message` | string | Error description (omitted on success) |

**HTTP 400** is returned if the request body is missing or the `command` field is absent.

---

## POST /message

Sends an in-game message or notification to players. Auth required.

The `method` field selects the delivery type. When omitted, defaults to a standard chat message.

### Chat message (default)

Standard in-game chat message with color selection.

```json
{
  "text": "Server restart in 10 minutes.",
  "type": "Yellow",
  "steamId": "76561198000000001"
}
```

| Field | Required | Description |
|---|---|---|
| `text` | Yes | Message text |
| `type` | No | Chat color (see [Message types](#message-types)). Defaults to `Yellow` |
| `steamId` | No | Target player's Steam ID. Omit to broadcast to all players |

### Warning notification

Center-screen notification with custom color and duration.

```json
{
  "method": "warning",
  "text": "Server restarting in 5 minutes!",
  "color": "#ff0000",
  "duration": 5
}
```

| Field | Required | Description |
|---|---|---|
| `method` | Yes | `"warning"` |
| `text` | Yes | Notification text |
| `color` | No | Hex color string (default: yellow) |
| `duration` | No | Seconds to display (default: 5) |
| `steamId` | No | Target player's Steam ID. Omit to send to all players |

### KillFeed notification

Bottom-center notification in the kill feed area.

```json
{
  "method": "killfeed",
  "prefix": "ALERT",
  "characterName": "Server",
  "suffix": "Restart imminent",
  "ping": true
}
```

| Field | Required | Description |
|---|---|---|
| `method` | Yes | `"killfeed"` |
| `prefix` | No | Text before the character name |
| `characterName` | No | Center text (displayed as a name) |
| `suffix` | No | Text after the character name |
| `ping` | No | Play notification sound (default: `true`) |
| `steamId` | No | Target player's Steam ID. Omit to send to all players |

### HUD notification

Bottom-left overlay notification. Does not appear in chat — ideal for non-intrusive alerts.

```json
{
  "method": "hud",
  "text": "Quest complete!"
}
```

| Field | Required | Description |
|---|---|---|
| `method` | Yes | `"hud"` |
| `text` | Yes | Notification text |
| `steamId` | No | Target player's Steam ID. Omit to send to all players |

---

### Per-player targeting

Any method supports targeting a specific player by adding the `steamId` field:

```json
{
  "method": "warning",
  "steamId": "76561198000000003",
  "text": "You have been warned!",
  "color": "#ff0000",
  "duration": 5
}
```

**Response**

```json
{
  "ok": true,
  "sent": true,
  "message": ""
}
```

---

## Message types

The `type` field controls the color of the message in-game. Values are case-insensitive.

| Value | Color | Notes |
|---|---|---|
| `Yellow` | Yellow | Default. Same as `ServerMessage` |
| `ServerMessage` | Yellow | Alias for `Yellow` |
| `White` | White | |
| `Cyan` | Cyan | |
| `Green` | Green | |
| `Red` | Red | Same as `Error` |
| `Error` | Red | Alias for `Red` |

Unknown values fall back to `Yellow`.

---

## Player action endpoints

These endpoints perform actions on individual players using direct execution — they bypass the chat pipeline for faster, more reliable results. All require authentication.

### POST /players/{steamId}/teleport

Teleports a player to specified coordinates.

**Request body**

```json
{
  "x": -253574.0,
  "y": 443844.0,
  "z": 91506.7
}
```

| Field | Required | Description |
|---|---|---|
| `x` | Yes | Target X coordinate (Unreal units) |
| `y` | Yes | Target Y coordinate (Unreal units) |
| `z` | Yes | Target Z coordinate (Unreal units) |

---

### POST /players/{steamId}/currency

Modifies a player's cash balance.

**Request body**

```json
{
  "action": "set",
  "amount": 5000
}
```

| Field | Required | Description |
|---|---|---|
| `action` | Yes | `"set"`, `"add"`, or `"remove"` |
| `amount` | Yes | Currency amount |

---

### POST /players/{steamId}/fame

Modifies a player's fame points.

**Request body**

```json
{
  "action": "set",
  "amount": 10000
}
```

| Field | Required | Description |
|---|---|---|
| `action` | Yes | `"set"`, `"add"`, or `"remove"` |
| `amount` | Yes | Fame point amount |

---

### POST /players/{steamId}/kick

Kicks a player from the server.

**Request body**

```json
{}
```

No additional parameters required. The player is identified by the Steam ID in the URL.

---

### POST /players/{steamId}/ban

Bans a player from the server.

**Request body**

```json
{}
```

No additional parameters required. The player is identified by the Steam ID in the URL.

---

## Vehicle action endpoints

### POST /vehicles/{vehicleId}/destroy

Destroys a vehicle by its entity ID.

**Request body**

```json
{}
```

No additional parameters required. The vehicle is identified by the entity ID in the URL.

---

### POST /vehicles/spawn

Spawns a vehicle at specified coordinates.

**Request body**

```json
{
  "class": "BP_Barba",
  "x": -253574.0,
  "y": 443844.0,
  "z": 91506.7
}
```

| Field | Required | Description |
|---|---|---|
| `class` | Yes | Vehicle blueprint class name |
| `x` | Yes | Spawn X coordinate |
| `y` | Yes | Spawn Y coordinate |
| `z` | Yes | Spawn Z coordinate |

---

## Server control endpoints

### POST /server/time

Sets the server time of day.

**Request body**

```json
{
  "hour": 14
}
```

| Field | Required | Description |
|---|---|---|
| `hour` | Yes | Hour (0–23) |

---

### POST /server/weather

Sets the server weather.

**Request body**

```json
{
  "value": 0.5
}
```

| Field | Required | Description |
|---|---|---|
| `value` | Yes | Weather intensity (0.0 = clear, 1.0 = maximum storm) |

---

## GET /fps.json

Returns live server FPS statistics. Auth required.

**Response**

```json
{
  "ok": true,
  "currentFps": 30.4,
  "avgFps": 30.3,
  "minFps": 28.1
}
```

| Field | Type | Description |
|---|---|---|
| `currentFps` | number | Current server FPS |
| `avgFps` | number | Average FPS over the sampling window |
| `minFps` | number | Minimum FPS over the sampling window |

---

## GET /fps/history.json

Returns historical FPS data at minute resolution, up to 7 days. Auth required.

Used by the panel's FPS chart on the Status tab.

**Response**

```json
{
  "ok": true,
  "intervalMinutes": 1,
  "count": 1440,
  "data": [
    { "t": 1712160000000, "avg": 30.2, "min": 28.1, "max": 31.5 },
    { "t": 1712160060000, "avg": 29.8, "min": 27.3, "max": 30.9 }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `intervalMinutes` | number | Sampling interval (always `1`) |
| `count` | number | Number of data points in the response |
| `data[].t` | number | Unix timestamp in milliseconds |
| `data[].avg` | number | Average FPS during this interval |
| `data[].min` | number | Minimum FPS during this interval |
| `data[].max` | number | Maximum FPS during this interval |

