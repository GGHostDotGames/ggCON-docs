# ggCON

**ggCON** is a remote administration mod for SCUM dedicated servers, built by [GG Host](https://www.gghost.games) — the official hosting provider of SCUM.

- **Web Panel** — browser-based admin dashboard with player list, live map, console, chat, vehicles, squads, flags, and server status
- **Live Map** — real-time player and vehicle positions on a 14K tiled map with teleport-to-location, distance measuring, and grid overlay
- **Rich Player Data** — health, skills, attributes, body effects, economy, gear, IP, gender, squad, and more
- **Player Actions** — teleport, kick, ban, give items, and edit fame/cash/gold directly from the panel
- **Vehicle Management** — full vehicle list with ownership, sortable columns, type filtering, map overlay, and one-click destroy
- **Squad Management** — view squads with live online status, remove members, delete squads
- **Flag & Base Tracking** — all base building flags with owner info, element counts, and map markers
- **Item Spawning** — searchable catalog of 6,000+ items with category filter and one-click spawning to any player
- **Vehicle Spawning** — give any of the 19 vehicle types directly to a player from the panel
- **Entity Spawning** — spawn zombies, animals, armed NPCs, Brenner, and Razor near any player with a tabbed picker
- **Multi-language** — web panel available in English, Chinese, Thai, Russian, German, and Ukrainian
- **In-game Economy** — the Quarter Master plugin adds a web storefront, a package shop priced in Rations or Cash, cross-server GG Coins, stock and discounts, and PvP/NPC kill rewards
- **Kill Feed & Leaderboards** — real-time PvP, NPC, trap, and suicide kill feed with a player leaderboard, in-game chat broadcasts, and Discord webhooks
- **NPC Tracking** — live status, health, and map positions for guards, drifters, zombies, animals, sentries, drones, and Razor
- **Paid Teleport (Taxi)** — players pay in-game cash to travel to admin-set destinations, with a PvP-safe countdown
- **AI NPCs** *(coming soon)* — AI-powered characters that interact with players in chat and execute admin commands
- **Slash Commands** — players type `/help`, `/unstuck`, `/shop`, `/taxi`, and more in chat for instant responses; admins and plugins can register custom commands
- **Plugin System** — extend ggCON with plugins for the in-game shop, kill feed, loot drops, NPC tracking, paid taxi, analytics, and more — with a built-in marketplace
- **Auto-Update** — check for and stage updates directly from the panel, applied on next server restart
- **SSL Access** — access the panel securely over HTTPS via GG Host's built-in SSL proxy
- **Server Controls** — set time of day and weather directly from the panel
- Run admin commands and capture their output via HTTP or RCON
- Execute developer commands without any database modifications
- Send in-game chat messages, center-screen warnings, kill feed notifications, and HUD overlays to all players or a specific player
- Monitor server logs in real-time (chat, kills, economy, admin, and more)
- Secure access with IP allowlists, password authentication, and rate limiting

## Quick links

| I want to… | Go to |
|---|---|
| Install and configure the mod | [Getting Started](getting-started.md) |
| Use the web panel | [Web Panel](web-panel.md) |
| Set up AI NPCs | [NPC System](npc-system.md) |
| Install or manage plugins | [Plugins](plugins.md) |
| Set up slash commands | [Slash Commands](slash-commands.md) |
| Call the HTTP API | [HTTP API](http-api.md) |
| Connect an RCON client | [RCON](rcon.md) |
| See all available commands | [Commands](commands.md) |
| Look up a config key | [Config Reference](config-reference.md) |
| Set up auto-updates | [Auto-Update](auto-update.md) |

## Built by GG Host

[![GG Host](assets/images/logo.png)](https://www.gghost.games)

[GG Host](https://www.gghost.games) is the official host of the SCUM dedicated servers, trusted by the SCUM development team and the community. With purpose-built hardware, in-house DDoS protection, and support that responds within minutes, GG Host is designed from the ground up for SCUM server owners.

ggCON is built and maintained by the same team — bringing the same level of depth and reliability to server automation and remote administration.

[Get a SCUM server from GG Host →](https://www.gghost.games/store/scum-server)

---

## How it works

ggCON runs alongside the SCUM server process. It starts an HTTP server and, optionally, an RCON server on configurable ports. Both interfaces route through the same authentication layer and command pipeline.

Commands are dispatched through the native SCUM admin command system, so they behave identically to commands typed in-game by an admin. Output produced by the game (e.g. player lists, error messages) is captured and returned in the API response.

In addition, ggCON provides direct-action REST endpoints for common operations like teleporting players, modifying economy values, destroying vehicles, and spawning items — these bypass the chat pipeline entirely for faster, more reliable execution.
