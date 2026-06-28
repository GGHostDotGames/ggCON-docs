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
  "version": "0.13.3",
  "build": "2026-05-28 10:22:24",
  "service": "http",
  "running": true
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
      "profileId": 8,
      "realName": "John Smith",
      "fakeName": "Shadow",
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
      "yaw": 87.4,
      "health": 1.0,
      "bodyEffects": [
        { "name": "Hypothermia", "severity": 1.5, "maxSeverity": 3, "kind": "condition", "stage": "untreated" }
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
| `profileId` | number | SCUM internal user profile ID (`0` if not matched in the database) |
| `realName` | string | Character's real in-game name from the database |
| `fakeName` | string | Character's alias / fake name if set, otherwise empty |
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
| `yaw` | number \| null | Body facing direction in degrees (0–360) |
| `health` | number \| null | Health as a fraction (0.0 = dead, 1.0 = full health) |
| `bodyEffects` | array | Active body conditions and symptoms. Each entry has `name` (string), `severity` (number), `maxSeverity` (number), `kind` (`"condition"` or `"symptom"`), and `stage` (`"none"`, `"untreated"`, `"stabilization"`, or `"recovery"` — treatment progress for conditions; `"none"` for symptoms) |
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
  "error": "player not found"
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
  "modVersion": "0.13.3",
  "modBuild": "2026-05-28 10:22:24",
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
  "nimbostratusCoverage": 0,
  "timeOfDaySpeed": 3.84,
  "sunIntensity": 10,
  "fogEnabled": false,
  "dayPeriod": "Day"
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
| `timeOfDaySpeed` | number \| null | In-game day-length multiplier vs real time (e.g. `3.84`×) |
| `sunIntensity` | number \| null | Sun brightness intensity |
| `fogEnabled` | bool \| null | Whether fog is enabled on the server |
| `dayPeriod` | string \| null | Derived part of day: `"Dawn"`, `"Day"`, `"Dusk"`, or `"Night"` |

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

## GET /flags.json

Returns all base building flags with owner info, location, and element counts, plus the server's flag rules (element caps, influence radius, overtake/decay periods). Auth required. The server flag-rule fields are omitted if that configuration could not be read from the game.

Flag data auto-refreshes every 30 seconds. Read-only database access — does not block the game server.

**Response**

```json
{
  "ok": true,
  "count": 2,
  "maxElementsPerFlag": 555,
  "maxExpandedPerFlag": 580,
  "extraElementsPerSquadMember": 25,
  "flagInfluenceRadius": 5000,
  "flagOvertakeDuration": 28800,
  "flagOvertakePeriod": 1,
  "decayProcessingPeriod": 120,
  "allowMultipleFlagsPerPlayer": false,
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
| `maxElementsPerFlag` | number | Server cap on building elements per flag |
| `maxExpandedPerFlag` | number | Server cap on elements when a flag is fully expanded |
| `extraElementsPerSquadMember` | number | Extra elements granted per additional squad member |
| `flagInfluenceRadius` | number | Flag influence radius in Unreal units |
| `flagOvertakeDuration` | number | Flag overtake duration (seconds) |
| `flagOvertakePeriod` | number | Flag overtake processing period |
| `decayProcessingPeriod` | number | Flag decay processing period |
| `allowMultipleFlagsPerPlayer` | bool | Whether a player may own multiple flags |
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
      "i": "Backpack_02_01",
      "ico": "ICO_Backpack_02_01",
      "c": "Equipment",
      "dn": "Hiking Backpack"
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
| `items[].ico` | string \| absent | Icon asset name (present when an icon mapping exists). Turn it into an image URL — see [Icon images](#icon-images) below |
| `items[].c` | string \| absent | Item category (e.g., `"Weapons"`, `"Ammunition"`, `"Food"`, `"Equipment"`) |
| `items[].dn` | string \| absent | Friendly display name (present when a name mapping exists) |

### Icon images

Every `ico` value maps to a publicly served icon image — no authentication and no API key required:

```
https://icons.gghost.games/icons/{ico}.webp
```

For example, an item with `"ico": "ICO_Backpack_02_01"` has its icon at
[`https://icons.gghost.games/icons/ICO_Backpack_02_01.webp`](https://icons.gghost.games/icons/ICO_Backpack_02_01.webp).

- **Format:** WebP (`Content-Type: image/webp`). These are the same icons the web panel and storefront display.
- **Public + cached:** the icon CDN is open and edge-cached, so you can hotlink these URLs directly from your own tool or website — you do not need to proxy them through your game server.
- **No icon:** items without an icon mapping omit the `ico` field entirely — there is simply no image to fetch, so render your own placeholder. A small number of mapped icons may also return `404` if the item was added in a newer game build than the current icon set; fall back to a placeholder in that case too.
- The same `{ico}` → URL scheme applies to the `ico` field on [`/vehicle-types.json`](#get-vehicle-typesjson), [`/zombies.json`](#get-zombiesjson), [`/animals.json`](#get-animalsjson), and [`/armed-npcs.json`](#get-armed-npcsjson).

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
  "type": "ServerMessage",
  "steamId": "76561198000000001"
}
```

| Field | Required | Description |
|---|---|---|
| `text` | Yes | Message text |
| `type` | No | Chat color (see [Message types](#message-types)). Defaults to `ServerMessage` (orange) |
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
| `ping` | No | Play notification sound (default: `false`) |
| `steamId` | No | Target player's Steam ID. Omit to send to all players |

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

## GET /icons/{name}.webp

Serves an item/vehicle icon (WebP, cached 24h) directly from your game server. **No authentication required** (image tags can't send auth headers). The icon name comes from the `ico` field on catalog endpoints such as `/items.json`. Returns `404` for an unknown icon, `400` for a malformed name. Most integrations should prefer the edge-cached CDN at `https://icons.gghost.games/icons/{ico}.webp`.

---

## Message types

The `type` field controls the color of the message in-game. Values are case-insensitive.

| Value | Color | Notes |
|---|---|---|
| `ServerMessage` | Orange | Default. SCUM's MOTD highlight color |
| `Orange` | Orange | Same as `ServerMessage` |
| `Yellow` | Yellow | Admin-yellow color |
| `White` | White | |
| `Cyan` | Cyan | |
| `Green` | Green | |
| `Red` | Red | Same as `Error` |
| `Error` | Red | Alias for `Red` |

Unknown values fall back to `ServerMessage` (orange).

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

---

## Plugin endpoints

These endpoints are provided by optional ggCON plugins. They are only available when the matching plugin is installed on the server — if a plugin is not installed, its endpoints return `404`. All require the same authentication (IP allowlist + `X-Password`) as the core endpoints.

### Kill Feed

The Kill Feed plugin records every kill, suicide, and trap death on the server and exposes them over the HTTP API. These endpoints are only available when the **Kill Feed** plugin is installed.

#### GET /kill-feed/events.json

Returns kill events from the current session, newest data held in memory. Designed for polling — pass the `since` cursor from the previous response to fetch only new events. Auth required.

Requires the **Kill Feed** plugin installed.

**Query parameters**

| Parameter | Required | Description |
|---|---|---|
| `since` | No | Only return events with a timestamp greater than this value. Pass the `next` value from the previous response to poll for new events. Defaults to `0` (all in-memory events) |
| `type` | No | Filter by event type: `pvp`, `npc`, `suicide`, `trap`, or `all` (default) |
| `player` | No | Filter by player. Case-insensitive substring match against killer/victim name, or an exact match against a Steam ID |
| `weapon` | No | Filter by weapon. Case-insensitive substring match against the cleaned weapon name |

**Response**

```json
{
  "ok": true,
  "events": [
    {
      "t": 1712160045123,
      "type": "pvp",
      "killer": {
        "name": "John Smith",
        "sid": "76561198000000001",
        "x": -253574.0,
        "y": 443844.0,
        "z": 91506.7
      },
      "victim": {
        "name": "Jane Doe",
        "sid": "76561198000000002",
        "x": -253210.0,
        "y": 444102.0,
        "z": 91500.2
      },
      "weapon": "MP5",
      "weaponRaw": "Weapon_MP5_C [Projectile]",
      "icon": "Weapon_MP5",
      "dmgType": "Projectile",
      "dist": 142.6,
      "tod": "14:32"
    },
    {
      "t": 1712160098456,
      "type": "npc",
      "killer": {
        "name": "John Smith",
        "sid": "76561198000000001",
        "x": -250100.0,
        "y": 441002.0,
        "z": 90120.0
      },
      "victim": {
        "name": "BP_Bear",
        "sid": "",
        "x": -250080.0,
        "y": 441050.0,
        "z": 90118.0
      },
      "weapon": "Compound Bow",
      "weaponRaw": "Weapon_CompoundBow_C [Projectile]",
      "icon": "Weapon_CompoundBow",
      "dmgType": "Projectile",
      "dist": 31.4,
      "cat": "Animal",
      "tod": "14:36"
    }
  ],
  "next": 1712160098456,
  "total": 1284
}
```

| Field | Type | Description |
|---|---|---|
| `events[].t` | number | Event timestamp in milliseconds |
| `events[].type` | string | Event type: `pvp`, `npc`, `suicide`, or `trap` |
| `events[].killer` | object | Killer info `{name, sid, x, y, z}`. `sid` is empty for kills made by an NPC |
| `events[].victim` | object | Victim info `{name, sid, x, y, z}`. For NPC kills the victim is the NPC and `sid` is empty |
| `events[].weapon` | string | Cleaned weapon name for display (e.g. `"MP5"`) |
| `events[].weaponRaw` | string | Raw weapon string as written by the game |
| `events[].icon` | string | Weapon icon name. Build an image URL with `https://icons.gghost.games/icons/{icon}.webp` |
| `events[].dmgType` | string | Damage type (e.g. `"Projectile"`, `"Melee"`) |
| `events[].dist` | number | Distance between killer and victim in metres |
| `events[].cat` | string \| absent | NPC category (e.g. `"Animal"`, `"Zombie"`, `"Guard"`, `"Drifter"`). Present only for NPC kills |
| `events[].tod` | string \| absent | In-game time of day when the event occurred. Present when available |
| `next` | number | Cursor for the next poll — pass this back as the `since` parameter |
| `total` | number | Total number of events held in memory for the current session |

---

#### GET /kill-feed/history.json

Returns kill events across stored log history, not just the current session. Use this to query past kills over a fixed time window. Auth required.

Requires the **Kill Feed** plugin installed.

**Query parameters**

| Parameter | Required | Description |
|---|---|---|
| `range` | No | Time window: `session` (in-memory events, default), `24h`, `48h`, or `all` |
| `type` | No | Filter by event type: `pvp`, `npc`, `suicide`, `trap`, or `all` |
| `player` | No | Filter by player. Case-insensitive substring match against killer/victim name, or a Steam ID match |
| `weapon` | No | Filter by weapon (case-insensitive substring). Applies to the `session` range only |

**Response**

```json
{
  "ok": true,
  "events": [
    {
      "t": 1712070045123,
      "type": "pvp",
      "killer": {
        "name": "John Smith",
        "sid": "76561198000000001",
        "x": -253574.0,
        "y": 443844.0,
        "z": 91506.7
      },
      "victim": {
        "name": "Jane Doe",
        "sid": "76561198000000002",
        "x": -253210.0,
        "y": 444102.0,
        "z": 91500.2
      },
      "weapon": "M9",
      "weaponRaw": "Weapon_M9_C [Projectile]",
      "icon": "Weapon_M9",
      "dmgType": "Projectile",
      "dist": 18.3,
      "tod": "21:07"
    }
  ],
  "next": 1712070045123,
  "total": 1,
  "capped": false
}
```

| Field | Type | Description |
|---|---|---|
| `events[]` | array | Kill events for the selected window, sorted oldest-first. Each entry uses the same fields as `/kill-feed/events.json` |
| `next` | number | Timestamp of the newest event in the response |
| `total` | number | Number of events returned |
| `capped` | boolean \| absent | Present and `true` only when the result hit the maximum size limit and was truncated |

---

#### GET /kill-feed/stats.json

Returns aggregate kill statistics for the current session, including totals by type and the top weapons and killers. Auth required.

Requires the **Kill Feed** plugin installed.

**Response**

```json
{
  "ok": true,
  "total": 1284,
  "pvp": 842,
  "npc": 401,
  "suicide": 28,
  "trap": 13,
  "topWeapons": [
    { "name": "MP5", "count": 211 },
    { "name": "M9", "count": 168 }
  ],
  "topKillers": [
    { "name": "John Smith", "count": 94 },
    { "name": "Jane Doe", "count": 71 }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `total` | number | Total number of events in the current session |
| `pvp` | number | Number of player-vs-player kills |
| `npc` | number | Number of NPC kills |
| `suicide` | number | Number of suicides |
| `trap` | number | Number of trap deaths |
| `topWeapons` | array | Up to the 10 most-used weapons, each `{name, count}`, sorted by count descending |
| `topKillers` | array | Up to the 10 players with the most PvP kills, each `{name, count}`, sorted by count descending |

---

#### GET /kill-feed/leaderboard.json

Returns a per-player kill leaderboard aggregated over a time window. Auth required.

Requires the **Kill Feed** plugin installed.

**Query parameters**

| Parameter | Required | Description |
|---|---|---|
| `window` | No | Time window to aggregate: `session`, `24h`, `7d` (default), `30d`, or `all` |
| `limit` | No | Maximum number of rows to return (default: 50, minimum: 1, maximum: 500) |

**Response**

```json
{
  "ok": true,
  "count": 2,
  "rows": [
    {
      "name": "John Smith",
      "sid": "76561198000000001",
      "pvpKills": 94,
      "pvpDeaths": 37,
      "kd": 2.54,
      "armedNpcKills": 22,
      "animalKills": 15,
      "puppetKills": 88,
      "trapKills": 3,
      "suicides": 1,
      "longestShot": 312.7,
      "favoriteWeapon": "MP5"
    },
    {
      "name": "Jane Doe",
      "sid": "76561198000000002",
      "pvpKills": 71,
      "pvpDeaths": 0,
      "kd": null,
      "armedNpcKills": 9,
      "animalKills": 4,
      "puppetKills": 40,
      "trapKills": 0,
      "suicides": 2,
      "longestShot": 198.4,
      "favoriteWeapon": "M9"
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `count` | number | Number of rows returned |
| `rows[].name` | string | Player character name |
| `rows[].sid` | string | Player's Steam ID |
| `rows[].pvpKills` | number | Player-vs-player kills |
| `rows[].pvpDeaths` | number | Times killed by another player |
| `rows[].kd` | number \| null | Kill/death ratio (PvP kills ÷ PvP deaths). `null` when the player has zero PvP deaths |
| `rows[].armedNpcKills` | number | Kills of armed human NPCs |
| `rows[].animalKills` | number | Animal kills |
| `rows[].puppetKills` | number | Puppet (zombie) kills |
| `rows[].trapKills` | number | Kills credited to the player's traps |
| `rows[].suicides` | number | Number of suicides |
| `rows[].longestShot` | number | Longest kill distance in metres |
| `rows[].favoriteWeapon` | string | The player's most-used weapon |

---

### NPC Tracker

Endpoints provided by the **NPC Tracker** plugin. Real-time tracking of non-player characters — guards, drifters, zombies, animals, sentries, and drones — including current health, alive/dead status, world location, and a session kill log.

---

#### GET /npc-tracker/npcs.json

Returns every tracked NPC currently in loaded areas, plus a log of NPC kills recorded this session. Auth required.

Requires the **NPC Tracker** plugin installed.

**Response**

```json
{
  "ok": true,
  "npcs": [
    {
      "type": "Guard",
      "class": "BP_Guard_Lvl_1_C",
      "name": "Guard Lvl 1",
      "health": 320.0,
      "maxHealth": 400.0,
      "alive": true,
      "x": -185420.5,
      "y": 372110.0,
      "z": 21805.3,
      "foe": "PlayerName",
      "target": "PlayerName",
      "killerSteamId": "",
      "killerName": ""
    },
    {
      "type": "Animal",
      "class": "BP_Deer_C",
      "name": "Deer",
      "health": 0.0,
      "maxHealth": 150.0,
      "alive": false,
      "x": -142003.0,
      "y": 410882.7,
      "z": 18950.0,
      "killerSteamId": "76561198000000001",
      "killerName": "PlayerName"
    }
  ],
  "kills": [
    {
      "type": "Zombie",
      "class": "BP_Infected_C",
      "name": "Infected",
      "x": -150220.0,
      "y": 398540.5,
      "z": 19002.0,
      "killerSteamId": "76561198000000001",
      "killerName": "PlayerName"
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `npcs` | array | NPCs currently in loaded areas (both alive and recently dead) |
| `npcs[].type` | string | Category — one of `Guard`, `Drifter`, `Zombie`, `Animal`, `Sentry`, `Drone`, `Unknown` |
| `npcs[].class` | string | NPC class name |
| `npcs[].name` | string | Human-readable display name derived from the class |
| `npcs[].health` | number | Current health |
| `npcs[].maxHealth` | number | Maximum health |
| `npcs[].alive` | boolean | Whether the NPC is currently alive |
| `npcs[].x` | number | World X position in Unreal units |
| `npcs[].y` | number | World Y position in Unreal units |
| `npcs[].z` | number | World Z position in Unreal units |
| `npcs[].foe` | string \| null | Display name of the NPC's current target, or `null` when not targeting or unresolved |
| `npcs[].target` | string \| null | Alias of `foe` |
| `npcs[].killerSteamId` | string | Steam ID of the killer (empty while the NPC is alive or if the killer is unknown) |
| `npcs[].killerName` | string | Character name of the killer (empty while the NPC is alive or if the killer is unknown) |
| `kills` | array | Kill log for this session, most recent first |
| `kills[].type` | string | Category of the killed NPC — same set of values as `npcs[].type` |
| `kills[].class` | string | Killed NPC class name |
| `kills[].name` | string | Human-readable display name of the killed NPC |
| `kills[].x` | number | World X position where the kill occurred, in Unreal units |
| `kills[].y` | number | World Y position where the kill occurred, in Unreal units |
| `kills[].z` | number | World Z position where the kill occurred, in Unreal units |
| `kills[].killerSteamId` | string | Steam ID of the killer (empty if unknown) |
| `kills[].killerName` | string | Character name of the killer (empty if unknown) |

---

---

### Trap Feed

The Trap Feed plugin tracks trap crafting, arming, disarming, and trigger events in real time and exposes them over the HTTP API. Both endpoints below return the same event shape; `events.json` serves the live current-session feed (ideal for incremental polling), while `history.json` reaches back across past log files over a chosen time range.

#### GET /trap-alerts/events.json

Live trap events from the current session, held in memory. Supports an incremental cursor for efficient polling.

Auth required.

Requires the Trap Feed plugin installed.

**Query parameters**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `since` | No | Only return events newer than this cursor (the `next` value from a previous response). Defaults to `0` (all events). |
| `action` | No | Filter by event type: `crafted`, `armed`, `disarmed`, `triggered`, or `all`. |
| `player` | No | Case-insensitive substring match against the player or owner name, or a Steam ID. |

**Response**

```json
{
  "ok": true,
  "events": [
    {
      "t": 1719240512000,
      "action": "crafted",
      "trap": "Improvised Mine",
      "player": {
        "name": "Alex Rivers",
        "sid": "76561198000000001",
        "id": 482
      },
      "x": -41250.5,
      "y": 18730.0,
      "z": 1024.0
    },
    {
      "t": 1719240731000,
      "action": "triggered",
      "trap": "Bear Trap",
      "player": {
        "name": "Mara Voss",
        "sid": "76561198000000002",
        "id": 615
      },
      "owner": {
        "name": "Alex Rivers",
        "sid": "76561198000000001",
        "id": 482
      },
      "x": -39880.0,
      "y": 20115.5,
      "z": 1031.0
    }
  ],
  "next": 1719240731000,
  "total": 2
}
```

| Field | Type | Description |
|-------|------|-------------|
| `ok` | boolean | Always `true` on success. |
| `events` | array | List of trap events matching the filters, oldest to newest. |
| `events[].t` | number | Event timestamp in epoch milliseconds. |
| `events[].action` | string | One of `crafted`, `armed`, `disarmed`, `triggered`. |
| `events[].trap` | string | Display name of the trap (e.g. `Improvised Mine`, `Bear Trap`). |
| `events[].player` | object | The player who performed the action (crafted / armed / disarmed), or who triggered the trap. |
| `events[].player.name` | string | Player display name. |
| `events[].player.sid` | string | Player Steam ID. |
| `events[].player.id` | number | Player profile ID. |
| `events[].owner` | object | Only present on `triggered` events: the player who originally placed the trap. Omitted otherwise. |
| `events[].owner.name` | string | Trap owner display name. |
| `events[].owner.sid` | string | Trap owner Steam ID. |
| `events[].owner.id` | number | Trap owner profile ID. |
| `events[].x` | number | World X coordinate of the event. |
| `events[].y` | number | World Y coordinate of the event. |
| `events[].z` | number | World Z coordinate of the event. |
| `next` | number | Cursor to pass as `since` on the next request to fetch only newer events. |
| `total` | number | Total number of events held for the current session (before filtering). |

---

#### GET /trap-alerts/history.json

Historical trap events parsed from server log files over a chosen time range. Returns the same event shape as `events.json`. Results are capped at 10,000 events.

Auth required.

Requires the Trap Feed plugin installed.

**Query parameters**

| Parameter | Required | Description |
|-----------|----------|-------------|
| `range` | No | Time window to cover: `session` (current session only, the default), `24h`, or `48h`. |
| `action` | No | Filter by event type: `crafted`, `armed`, `disarmed`, `triggered`, or `all`. |
| `player` | No | Case-insensitive substring match against the player or owner name, or a Steam ID. |

**Response**

```json
{
  "ok": true,
  "events": [
    {
      "t": 1719154100000,
      "action": "armed",
      "trap": "Improvised Mine",
      "player": {
        "name": "Devon Park",
        "sid": "76561198000000003",
        "id": 207
      },
      "x": -38120.0,
      "y": 16540.5,
      "z": 998.0
    },
    {
      "t": 1719161842000,
      "action": "triggered",
      "trap": "Improvised Mine",
      "player": {
        "name": "Mara Voss",
        "sid": "76561198000000002",
        "id": 615
      },
      "owner": {
        "name": "Devon Park",
        "sid": "76561198000000003",
        "id": 207
      },
      "x": -38120.0,
      "y": 16540.5,
      "z": 998.0
    }
  ],
  "next": 1719161842000,
  "total": 2
}
```

| Field | Type | Description |
|-------|------|-------------|
| `ok` | boolean | Always `true` on success. |
| `events` | array | Matching trap events over the requested range, sorted oldest to newest. |
| `events[].t` | number | Event timestamp in epoch milliseconds. |
| `events[].action` | string | One of `crafted`, `armed`, `disarmed`, `triggered`. |
| `events[].trap` | string | Display name of the trap. |
| `events[].player` | object | The player who performed the action or triggered the trap (with `name`, `sid`, `id`). |
| `events[].owner` | object | Only present on `triggered` events: the player who placed the trap (with `name`, `sid`, `id`). |
| `events[].x` | number | World X coordinate of the event. |
| `events[].y` | number | World Y coordinate of the event. |
| `events[].z` | number | World Z coordinate of the event. |
| `next` | number | Timestamp of the most recent event returned. |
| `total` | number | Number of events returned. |
| `capped` | boolean | Present and `true` only when the 10,000-event limit was reached and results were truncated. |

---

### Log Analyzer

These endpoints serve server performance and entity-count metrics parsed from the SCUM server's Global Stats log lines (FPS, character/zombie/animal/vehicle/item counts, and network object counts). They power the **Analytics** tab in the panel.

Data covers the **current server session only** (since the last restart). It is read on demand — call the plugin's load action from the panel to ingest new log lines before reading these endpoints.

#### GET /analyzer/stats.json

Returns the most recent metrics sample. Auth required.

Requires the Log Analyzer plugin installed.

**Response — data available**

```json
{
  "ok": true,
  "hasData": true,
  "timestamp": 1712160300000,
  "minFps": 27.4,
  "avgFps": 30.1,
  "maxFps": 32.8,
  "characters": 142,
  "charactersAlive": 138,
  "prisoners": 12,
  "prisonersAlive": 12,
  "zombies": 96,
  "zombiesAlive": 94,
  "razors": 3,
  "razorsAlive": 3,
  "sentries": 8,
  "sentrysAlive": 8,
  "animals": 24,
  "animalsAlive": 22,
  "vehicles": 268,
  "itemsTotal": 184302,
  "itemsActive": 9120,
  "netObjectsTotal": 41250,
  "netObjectsActive": 6840
}
```

**Response — no data loaded yet**

```json
{
  "ok": true,
  "hasData": false
}
```

| Field | Type | Description |
|---|---|---|
| `hasData` | boolean | `false` when no samples have been loaded; the metrics fields are then omitted |
| `timestamp` | number | Unix timestamp in milliseconds of this sample |
| `minFps` | number | Minimum server FPS during the sample window |
| `avgFps` | number | Average server FPS during the sample window |
| `maxFps` | number | Maximum server FPS during the sample window |
| `characters` | number | Total characters tracked by the server |
| `charactersAlive` | number | Characters currently alive |
| `prisoners` | number | Total prisoners (players) |
| `prisonersAlive` | number | Prisoners currently alive |
| `zombies` | number | Total zombies |
| `zombiesAlive` | number | Zombies currently alive |
| `razors` | number | Total Razors |
| `razorsAlive` | number | Razors currently alive |
| `sentries` | number | Total sentries |
| `sentrysAlive` | number | Sentries currently alive |
| `animals` | number | Total animals |
| `animalsAlive` | number | Animals currently alive |
| `vehicles` | number | Total vehicles |
| `itemsTotal` | number | Total items in virtualization |
| `itemsActive` | number | Items currently active |
| `netObjectsTotal` | number | Total network objects |
| `netObjectsActive` | number | Network objects currently active |

---

#### GET /analyzer/history.json

Returns a time series of metrics over a selectable time window, for charting. Auth required.

Requires the Log Analyzer plugin installed.

**Query parameters**

| Parameter | Required | Description |
|---|---|---|
| `range` | No | Time window: `1h` (default), `6h`, `24h`, or `7d`. The `1h` range returns per-sample resolution; longer ranges return one point per minute. An unrecognized value falls back to `1h` |

**Response**

```json
{
  "ok": true,
  "range": "1h",
  "data": [
    {
      "t": 1712160000000,
      "minFps": 27.4,
      "avgFps": 30.1,
      "maxFps": 32.8,
      "C": 142,
      "Ca": 138,
      "P": 12,
      "Pa": 12,
      "Z": 96,
      "Za": 94,
      "R": 3,
      "Ra": 3,
      "S": 8,
      "Sa": 8,
      "A": 24,
      "Aa": 22,
      "V": 268,
      "IV": 184302,
      "IVa": 9120,
      "NO": 41250,
      "NOa": 6840
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `range` | string | The applied time window (`1h`, `6h`, `24h`, or `7d`) |
| `data` | array | Time-ordered data points within the window |
| `data[].t` | number | Unix timestamp in milliseconds |
| `data[].minFps` | number | Minimum FPS |
| `data[].avgFps` | number | Average FPS |
| `data[].maxFps` | number | Maximum FPS |
| `data[].C` / `data[].Ca` | number | Characters total / alive |
| `data[].P` / `data[].Pa` | number | Prisoners total / alive |
| `data[].Z` / `data[].Za` | number | Zombies total / alive |
| `data[].R` / `data[].Ra` | number | Razors total / alive |
| `data[].S` / `data[].Sa` | number | Sentries total / alive |
| `data[].A` / `data[].Aa` | number | Animals total / alive |
| `data[].V` | number | Vehicles |
| `data[].IV` / `data[].IVa` | number | Items in virtualization, total / active |
| `data[].NO` / `data[].NOa` | number | Network objects, total / active |

---

#### GET /analyzer/summary.json

Returns aggregate statistics computed across all loaded samples — useful for a high-level overview card. Auth required.

Requires the Log Analyzer plugin installed.

**Response — data available**

```json
{
  "ok": true,
  "totalSamples": 4320,
  "fpsAvg": 29.6,
  "fpsMin": 18.2,
  "fpsMax": 34.1,
  "peakZombies": 118,
  "peakAnimals": 31,
  "peakVehicles": 271,
  "peakItems": 192044,
  "peakNetObjects": 44210,
  "timeSpanMs": 86400000
}
```

**Response — no data loaded yet**

```json
{
  "ok": true,
  "totalSamples": 0
}
```

| Field | Type | Description |
|---|---|---|
| `totalSamples` | number | Number of samples loaded for the current session. When `0`, the aggregate fields are omitted |
| `fpsAvg` | number | Average FPS across all samples |
| `fpsMin` | number | Lowest FPS observed |
| `fpsMax` | number | Highest FPS observed |
| `peakZombies` | number | Highest zombie count observed |
| `peakAnimals` | number | Highest animal count observed |
| `peakVehicles` | number | Highest vehicle count observed |
| `peakItems` | number | Highest item-in-virtualization count observed |
| `peakNetObjects` | number | Highest network-object count observed |
| `timeSpanMs` | number | Milliseconds between the first and last loaded sample (effective coverage window) |

---

### Loot Drops

These endpoints are added by the **Loot Drops** plugin. They expose the plugin's reward packs and the per-player claim history shown in the panel.

#### GET /drops/packs.json

Returns every reward pack configured on the server, including disabled ones, with each pack's items, allowlist, and total claim count.

Auth required.

Requires the **Loot Drops** plugin installed.

**Response**

```json
{
  "ok": true,
  "packs": [
    {
      "id": 1,
      "displayName": "Starter Kit",
      "description": "A small care package for new arrivals.",
      "command": "starter",
      "claimLimit": 1,
      "enabled": true,
      "expiresAt": "",
      "createdAt": "2026-06-01 14:22:08",
      "updatedAt": "2026-06-18 09:41:55",
      "items": [
        {
          "id": 12,
          "itemClass": "BP_Weapon_M9",
          "vehicleClass": "",
          "quantity": 1,
          "stackCount": 0
        },
        {
          "id": 13,
          "itemClass": "Cal_9mm_FMJ_Ammo",
          "vehicleClass": "",
          "quantity": 1,
          "stackCount": 60
        }
      ],
      "allowlist": [
        {
          "steamId": "76561198000000001",
          "name": "SurvivorJoe"
        }
      ],
      "totalClaims": 14
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `id` | number | Unique pack identifier. |
| `displayName` | string | Player-facing pack name. |
| `description` | string | Optional description shown in the panel. |
| `command` | string | The slash command players type in chat to claim the pack (without the leading slash). |
| `claimLimit` | number | How many times a player may claim the pack. `0` means unlimited, `1` means one-time, `N` means up to N claims. |
| `enabled` | boolean | Whether the pack is currently claimable. |
| `expiresAt` | string | ISO datetime after which the pack can no longer be claimed; empty means no expiry. |
| `createdAt` | string | When the pack was created. |
| `updatedAt` | string | When the pack was last modified. |
| `items` | array | The contents granted on claim. |
| `items[].id` | number | Unique row identifier. |
| `items[].itemClass` | string | Item to grant; empty for vehicle rows. |
| `items[].vehicleClass` | string | Vehicle to grant; empty for item rows. |
| `items[].quantity` | number | Number of separate items or vehicles spawned per claim. |
| `items[].stackCount` | number | Stack size for stackable items such as ammo or cash. `0` for single items; always `0` for vehicles. |
| `allowlist` | array | If non-empty, only these players may claim the pack. An empty array means everyone may claim. |
| `allowlist[].steamId` | string | Steam ID permitted to claim. |
| `allowlist[].name` | string | Optional display name stored alongside the Steam ID. |
| `totalClaims` | number | Total number of times this pack has been claimed. |

---

#### GET /drops/claims.json

Returns the claim history — which players have claimed which packs, and when. Supports filtering by pack or player and pagination.

Auth required.

Requires the **Loot Drops** plugin installed.

**Query parameters**

| Parameter | Required | Description |
|---|---|---|
| `limit` | No | Maximum number of claims to return. Defaults to `1000`. Use `0` to return all claims. |
| `offset` | No | Number of claims to skip, for pagination. Defaults to `0`. |
| `pack_id` | No | Return only claims for the pack with this ID. |
| `steam_id` | No | Return only claims made by this Steam ID. |

**Response**

```json
{
  "ok": true,
  "claims": [
    {
      "id": 482,
      "packId": 1,
      "steamId": "76561198000000001",
      "charName": "SurvivorJoe",
      "claimedAt": "2026-06-24 11:05:37"
    },
    {
      "id": 481,
      "packId": 3,
      "steamId": "76561198000000002",
      "charName": "RaiderKim",
      "claimedAt": "2026-06-24 09:48:12"
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `id` | number | Unique claim identifier. |
| `packId` | number | ID of the pack that was claimed. |
| `steamId` | string | Steam ID of the player who claimed the pack. |
| `charName` | string | Character name of the player at the time of the claim. |
| `claimedAt` | string | When the claim was made. |

---

### Quarter Master

The following endpoints are provided by the Quarter Master shop plugin. They return read-only shop data — packages, categories, player balances, and sales figures — for building dashboards or external integrations. Each requires the Quarter Master plugin to be installed and enabled on the server.

#### GET /shop/stats.json

Returns headline shop statistics: lifetime sales, currency spent and granted, and currency currently held by players. Auth required.

Requires the Quarter Master plugin installed.

**Response**

```json
{
  "ok": true,
  "totalPackagesSold": 1284,
  "totalCoinsSpent": 96750,
  "totalCoinsGranted": 142300,
  "coinsInCirculation": 45550,
  "activePlayers": 87
}
```

| Field | Type | Description |
|---|---|---|
| `totalPackagesSold` | number | Total number of packages purchased over the shop's lifetime |
| `totalCoinsSpent` | number | Total rations spent on purchases |
| `totalCoinsGranted` | number | Total rations granted to players (rewards and admin grants) |
| `coinsInCirculation` | number | Total rations currently held across all player balances |
| `activePlayers` | number | Number of players with a balance greater than zero |

---

#### GET /shop/sales-summary.json

Returns aggregated sales figures, broken down by category and by best-selling package. Auth required.

Requires the Quarter Master plugin installed.

**Response**

```json
{
  "ok": true,
  "totalSales": 1284,
  "totalRations": 96750,
  "totalMoney": 41200,
  "byCategory": [
    {
      "categoryId": 1,
      "name": "Weapons",
      "salesCount": 412,
      "totalRations": 38600,
      "totalMoney": 12400
    },
    {
      "categoryId": 2,
      "name": "Vehicles",
      "salesCount": 96,
      "totalRations": 28800,
      "totalMoney": 9600
    }
  ],
  "topPackages": [
    {
      "packageId": 14,
      "name": "Starter Rifle Kit",
      "category": "Weapons",
      "salesCount": 203,
      "totalRations": 10150,
      "totalMoney": 0
    },
    {
      "packageId": 31,
      "name": "Off-Road Jeep",
      "category": "Vehicles",
      "salesCount": 64,
      "totalRations": 19200,
      "totalMoney": 6400
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `totalSales` | number | Total number of completed purchases |
| `totalRations` | number | Total rations spent across all purchases |
| `totalMoney` | number | Total in-game cash spent across all purchases |
| `byCategory[].categoryId` | number | Category ID |
| `byCategory[].name` | string | Category name |
| `byCategory[].salesCount` | number | Number of purchases in this category |
| `byCategory[].totalRations` | number | Rations spent in this category |
| `byCategory[].totalMoney` | number | In-game cash spent in this category |
| `topPackages[].packageId` | number | Package ID |
| `topPackages[].name` | string | Package name |
| `topPackages[].category` | string | Name of the category the package belongs to |
| `topPackages[].salesCount` | number | Number of times this package was purchased |
| `topPackages[].totalRations` | number | Rations spent on this package |
| `topPackages[].totalMoney` | number | In-game cash spent on this package |

---

#### GET /shop/balances.json

Returns the rations balance and lifetime totals for every player who has a shop balance. Auth required.

Requires the Quarter Master plugin installed.

**Response**

```json
{
  "ok": true,
  "balances": [
    {
      "steamId": "76561198000000001",
      "coins": 525,
      "totalEarned": 1340,
      "totalSpent": 815,
      "ggCoinsCached": 0
    },
    {
      "steamId": "76561198000000002",
      "coins": 90,
      "totalEarned": 600,
      "totalSpent": 510,
      "ggCoinsCached": 250
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `balances[].steamId` | string | Player's Steam ID |
| `balances[].coins` | number | Current rations balance |
| `balances[].totalEarned` | number | Lifetime rations earned by the player |
| `balances[].totalSpent` | number | Lifetime rations spent by the player |
| `balances[].ggCoinsCached` | number | Last-known cross-server coin balance for the player |

---

#### GET /shop/packages.json

Returns every shop package, including pricing, any active discount, stock, and the items each package delivers. Auth required.

Requires the Quarter Master plugin installed.

**Response**

```json
{
  "ok": true,
  "packages": [
    {
      "id": 14,
      "categoryId": 1,
      "name": "Starter Rifle Kit",
      "description": "A reliable rifle with two spare magazines.",
      "imageUrl": "",
      "priceCoins": 50,
      "priceMoney": 0,
      "effectiveCoins": 40,
      "effectiveMoney": 0,
      "discountPercent": 20,
      "discountExpiresAt": "2026-07-01T00:00:00",
      "stockLimit": -1,
      "stockSold": 203,
      "enabled": true,
      "vehicleClass": "",
      "leadItemClass": "Weapon_M16A4",
      "dropzoneName": "",
      "items": [
        {
          "itemClass": "Weapon_M16A4",
          "quantity": 1,
          "stackCount": 0,
          "ico": "Weapon_M16A4"
        },
        {
          "itemClass": "Cal_556x45mm_Ammobox",
          "quantity": 2,
          "stackCount": 30,
          "ico": "Cal_556x45mm_Ammobox"
        }
      ]
    },
    {
      "id": 31,
      "categoryId": 2,
      "name": "Off-Road Jeep",
      "description": "A rugged 4x4 delivered to your drop zone.",
      "imageUrl": "",
      "priceCoins": 300,
      "priceMoney": 100,
      "effectiveCoins": 300,
      "effectiveMoney": 100,
      "discountPercent": 0,
      "discountExpiresAt": "",
      "stockLimit": 10,
      "stockSold": 6,
      "enabled": true,
      "vehicleClass": "BPC_LaikaOffroad",
      "leadItemClass": "",
      "dropzoneName": "North Outpost",
      "vehicleIco": "BPC_LaikaOffroad",
      "items": []
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `packages[].id` | number | Package ID |
| `packages[].categoryId` | number | ID of the category the package belongs to |
| `packages[].name` | string | Package name |
| `packages[].description` | string | Package description |
| `packages[].imageUrl` | string | Optional custom image URL for the storefront card |
| `packages[].priceCoins` | number | List price in rations |
| `packages[].priceMoney` | number | List price in in-game cash |
| `packages[].effectiveCoins` | number | Rations price after any active discount |
| `packages[].effectiveMoney` | number | Cash price after any active discount |
| `packages[].discountPercent` | number | Discount percentage (0–100); 0 means no discount |
| `packages[].discountExpiresAt` | string | ISO 8601 UTC expiry for the discount, or empty for no expiry |
| `packages[].stockLimit` | number | Maximum units available; `-1` means unlimited |
| `packages[].stockSold` | number | Units sold so far |
| `packages[].enabled` | boolean | Whether the package is available in the storefront |
| `packages[].vehicleClass` | string | Vehicle identifier for vehicle packages, or empty for item packages |
| `packages[].leadItemClass` | string | Item shown as the package's storefront image; empty falls back to the first item |
| `packages[].dropzoneName` | string | Drop zone the package is restricted to, or empty for any zone |
| `packages[].vehicleIco` | string | Icon name for the vehicle (vehicle packages only) |
| `packages[].items` | array | Items delivered by the package (empty for vehicle packages) |
| `packages[].items[].itemClass` | string | Item identifier |
| `packages[].items[].quantity` | number | Number of this item delivered |
| `packages[].items[].stackCount` | number | Stack size per delivered entity; 0 means single items |
| `packages[].items[].ico` | string | Icon name for the item |

---

#### GET /shop/categories.json

Returns the shop's categories, in display order. Auth required.

Requires the Quarter Master plugin installed.

**Response**

```json
{
  "ok": true,
  "categories": [
    {
      "id": 1,
      "name": "Weapons",
      "sortOrder": 0,
      "icon": "weapon",
      "isProtected": false,
      "enabled": true
    },
    {
      "id": 2,
      "name": "Vehicles",
      "sortOrder": 1,
      "icon": "vehicle",
      "isProtected": true,
      "enabled": true
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `categories[].id` | number | Category ID |
| `categories[].name` | string | Category name |
| `categories[].sortOrder` | number | Display order; lower values appear first |
| `categories[].icon` | string | Icon name for the category |
| `categories[].isProtected` | boolean | Whether the category is auto-managed and cannot be deleted |
| `categories[].enabled` | boolean | Whether the category and its packages are shown in the storefront |

---

### Taxi Service

Endpoints provided by the **Taxi Service** plugin for reading the configured taxi destinations and destination clusters. These are available only when the plugin is installed.

#### GET /taxi/destinations.json

Returns all configured taxi destinations with their fare, location, and availability flags. Auth required.

Requires the **Taxi Service** plugin installed.

**Response**

```json
{
  "destinations": [
    {
      "id": 1,
      "name": "Airfield",
      "x": -123456.0,
      "y": 654321.0,
      "z": 12500.0,
      "fare": 250,
      "enabled": true,
      "listed": true,
      "description": "Central airfield drop-off",
      "tripLimit": 0
    },
    {
      "id": 2,
      "name": "Trader B4",
      "x": 305120.0,
      "y": -88240.0,
      "z": 9800.0,
      "fare": 100,
      "enabled": true,
      "listed": false,
      "description": "",
      "tripLimit": 3
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `destinations[].id` | number | Destination ID |
| `destinations[].name` | string | Destination name (used when booking a ride) |
| `destinations[].x` | number | World X position in Unreal units |
| `destinations[].y` | number | World Y position in Unreal units |
| `destinations[].z` | number | World Z position in Unreal units |
| `destinations[].fare` | number | Cost of a ride to this destination |
| `destinations[].enabled` | boolean | Whether the destination is bookable |
| `destinations[].listed` | boolean | Whether the destination appears in the public destination list (when `false`, it is hidden from the list but still bookable by exact name) |
| `destinations[].description` | string | Optional note shown alongside the destination |
| `destinations[].tripLimit` | number | Per-player lifetime ride limit for this destination (`0` means unlimited) |

---

#### GET /taxi/clusters.json

Returns all configured destination clusters, which group multiple destinations under a single name with a shared ride limit. Auth required.

Requires the **Taxi Service** plugin installed.

**Response**

```json
{
  "clusters": [
    {
      "id": 1,
      "name": "Traders",
      "members": "trader b4,trader z3,trader c2",
      "enabled": true,
      "listed": true,
      "tripLimit": 5
    },
    {
      "id": 2,
      "name": "Airfields",
      "members": "airfield,airfield north",
      "enabled": true,
      "listed": false,
      "tripLimit": 0
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `clusters[].id` | number | Cluster ID |
| `clusters[].name` | string | Cluster name (used when booking a ride) |
| `clusters[].members` | string | Comma-separated list of the destination names belonging to this cluster |
| `clusters[].enabled` | boolean | Whether the cluster is bookable |
| `clusters[].listed` | boolean | Whether the cluster appears in the public destination list (when `false`, it is hidden from the list but still bookable by exact name) |
| `clusters[].tripLimit` | number | Shared per-player lifetime ride limit across the whole cluster (`0` means unlimited) |

---

---

### ggHaul

Endpoints provided by the ggHaul plugin, which lets admins place persistent appliances (such as fridges) into the world and keeps a registry of every appliance that has been placed.

#### GET /gghaul/appliances

Returns every appliance in the ggHaul registry, newest first. Auth required.

Requires the ggHaul plugin installed.

**Response**

```json
{
  "ok": true,
  "appliances": [
    {
      "id": 12,
      "type": "fridge",
      "owner": "John Smith",
      "steamId": "76561198000000001",
      "x": -253574.0,
      "y": 443844.0,
      "z": 91506.7,
      "yaw": 135.0,
      "entityId": 71805135,
      "status": "completed"
    },
    {
      "id": 11,
      "type": "fridge",
      "owner": "",
      "steamId": "76561198000000002",
      "x": 0.0,
      "y": 0.0,
      "z": 0.0,
      "yaw": 0.0,
      "entityId": 0,
      "status": "in-progress"
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `appliances[].id` | number | Registry ID of the appliance |
| `appliances[].type` | string | Appliance type (e.g. `"fridge"`) |
| `appliances[].owner` | string | Name of the player the appliance belongs to, or empty if unset |
| `appliances[].steamId` | string | Steam ID of the player the appliance is anchored to |
| `appliances[].x` | number | World X position in Unreal units (`0` until placed) |
| `appliances[].y` | number | World Y position in Unreal units (`0` until placed) |
| `appliances[].z` | number | World Z position in Unreal units (`0` until placed) |
| `appliances[].yaw` | number | Facing rotation in degrees |
| `appliances[].entityId` | number | In-game entity ID once the appliance is placed (`0` while still being placed) |
| `appliances[].status` | string | Placement state: `"in-progress"`, `"completed"`, or `"failed"` |

### Stash 'n Dash

Endpoints provided by the Stash 'n Dash plugin — a contraband dead-drop race. You define **drops** (the item lists players must collect), the **sites** where drop cabinets spawn, and the game **settings**, then start and stop the game. Every endpoint requires auth and the Stash 'n Dash plugin installed.

Together these let you automate the whole game lifecycle from your own tooling (a bot, a Discord command, a scheduler) without opening the ggCON panel.

#### GET /stash-n-dash/status.json

Live game status: whether the game is active, how many cabinets are live, and the current drop's progress.

**Response**

```json
{
  "ok": true,
  "active": true,
  "cabinets": 3,
  "drop": {
    "active": true,
    "name": "Up 'n smoke",
    "won": false,
    "winner": "",
    "required": "10x Spliff",
    "secondsLeft": 3240,
    "players": 0
  }
}
```

| Field | Type | Description |
|---|---|---|
| `active` | bool | Whether the game machinery is running |
| `cabinets` | number | Count of live drop cabinets currently spawned |
| `drop.active` | bool | Whether a drop round is currently running |
| `drop.name` | string | Name of the active drop |
| `drop.won` | bool | Whether the active drop has been completed by someone |
| `drop.winner` | string | Name of the winning player (empty until won) |
| `drop.required` | string | Human-readable summary of the required items (e.g. `"10x Spliff"`) |
| `drop.secondsLeft` | number | Seconds remaining in the current round |
| `drop.players` | number | Players with progress toward the current drop |

#### GET /stash-n-dash/drops.json

Lists every drop definition (the "quests"), including disabled ones.

**Response**

```json
{
  "ok": true,
  "drops": [
    {
      "id": 1,
      "name": "Up 'n smoke",
      "enabled": true,
      "durationSec": 3600,
      "weight": 1,
      "required": [
        { "itemClass": "Spliff", "count": 10, "singleRow": true }
      ],
      "rewards": [
        { "kind": "item", "itemClass": "1H_KitchenKnife", "quantity": 1, "stackCount": 1, "amount": 0 }
      ]
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `drops[].id` | number | Drop ID (use it in update / delete / toggle) |
| `drops[].name` | string | Display name of the drop |
| `drops[].enabled` | bool | Whether the drop is eligible for selection |
| `drops[].durationSec` | number | Round length in seconds |
| `drops[].weight` | number | Relative weight for weighted-random selection |
| `drops[].required[].itemClass` | string | Item class players must deposit |
| `drops[].required[].count` | number | Quantity required of that item |
| `drops[].required[].singleRow` | bool | Whether the count must fit in a single inventory stack |
| `drops[].rewards[].kind` | string | Reward kind (`"item"` in v1) |
| `drops[].rewards[].itemClass` | string | Item class granted as a reward |
| `drops[].rewards[].quantity` | number | Number of reward items |
| `drops[].rewards[].stackCount` | number | Stack size per reward item |

#### POST /stash-n-dash/drops

Create a new drop (omit `id` or send `0`) or update an existing one (send its `id`).

**Request body**

```json
{
  "name": "Up 'n smoke",
  "enabled": true,
  "weight": 1,
  "durationMin": 60,
  "required": [
    { "itemClass": "Spliff", "count": 10 }
  ],
  "rewards": [
    { "kind": "item", "itemClass": "1H_KitchenKnife", "quantity": 1, "stackCount": 1 }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `id` | number | Omit or `0` to create; supply it to update |
| `name` | string | Required. Display name |
| `enabled` | bool | Whether the drop is eligible for selection |
| `weight` | number | Weighted-random weight (min 1) |
| `durationMin` | number | Round length in minutes (1–1440). Or send `durationSec` instead |
| `required[]` | array | At least one item, each with `itemClass` + `count`. Required item classes may not be substrings of one another |
| `rewards[]` | array | Item rewards: `kind` (`"item"`), `itemClass`, `quantity`, `stackCount` |

**Response**

```json
{ "ok": true, "id": 1 }
```

#### POST /stash-n-dash/drops/delete

Delete a drop by ID.

**Request body** — `{ "id": 1 }`

**Response** — `{ "ok": true }`

#### POST /stash-n-dash/drops/toggle

Enable or disable a drop by ID.

**Request body** — `{ "id": 1, "enabled": false }`

**Response** — `{ "ok": true }`

#### GET /stash-n-dash/sites.json

Lists the cabinet spawn locations.

**Response**

```json
{
  "ok": true,
  "sites": [
    { "id": 1, "name": "The Lockup", "x": -1834.2, "y": 9921.7, "z": 312.0, "yaw": 90.0, "enabled": true }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `sites[].id` | number | Site ID (preserve it when updating to keep cabinet keying stable) |
| `sites[].name` | string | Site label |
| `sites[].x` / `.y` / `.z` | number | World position in Unreal units |
| `sites[].yaw` | number | Facing rotation in degrees |
| `sites[].enabled` | bool | Whether cabinets may spawn at this site |

#### POST /stash-n-dash/sites

Replace the **full** set of cabinet spawn locations — send every site you want to keep (omitted sites are removed). Two enabled sites cannot sit within the cabinet recovery radius of each other.

**Request body**

```json
{
  "sites": [
    { "id": 1, "name": "The Lockup", "x": -1834.2, "y": 9921.7, "z": 312.0, "yaw": 90.0, "enabled": true }
  ]
}
```

**Response**

```json
{ "ok": true, "count": 1 }
```

#### GET /stash-n-dash/settings.json

Returns the game tunables.

**Response**

```json
{
  "ok": true,
  "settings": {
    "masterEnabled": false,
    "liveCabinets": 3,
    "rotationSec": 1800,
    "dropGapSec": 0,
    "depositRangeCm": 300,
    "selectionMode": "random"
  }
}
```

| Field | Type | Description |
|---|---|---|
| `masterEnabled` | bool | Whether the game is currently running |
| `liveCabinets` | number | How many cabinets are live at once (1–50) |
| `rotationSec` | number | Seconds between cabinet rotations (min 60) |
| `dropGapSec` | number | Idle gap between drop rounds, in seconds |
| `depositRangeCm` | number | How close a player must be to deposit, in centimetres (min 100) |
| `selectionMode` | string | `"random"` (weighted) or `"sequential"` |

#### POST /stash-n-dash/settings

Update any subset of the tunables. Sending `masterEnabled` starts or stops the game.

**Request body**

```json
{
  "liveCabinets": 3,
  "rotationSec": 1800,
  "selectionMode": "random",
  "masterEnabled": true
}
```

**Response** — `{ "ok": true }`

#### POST /stash-n-dash/start

Activate the game: spawn cabinets and begin a drop.

**Response** — `{ "ok": true, "queued": true }`

#### POST /stash-n-dash/stop

Deactivate the game: despawn cabinets and end the round.

**Response** — `{ "ok": true, "queued": true }`
