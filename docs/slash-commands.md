# Slash Commands

Slash commands let players interact with ggCON and its plugins directly from in-game chat. Type a command starting with `/` in any chat channel and ggCON responds instantly.

## Built-in Commands

### /help

Displays all available slash commands, grouped by source.

```
/help
```

**Output example:**

```
ggCON Commands:
  /help — Show available commands
  /unstuck — Free yourself when stuck

Loot Drops:
  /starter — Claim the Starter Kit
  /event-drop — Claim the Event Drop
```

Commands from plugins are grouped under the plugin name so you can quickly find what's available.

---

### /unstuck

Frees a player who is stuck in terrain, fallen through the world, or otherwise unable to move, by teleporting them a short distance in a random horizontal direction.

```
/unstuck
```

Admin-configurable in **Settings → General**:

- **Distance** — how far the player is moved (default 100 m)
- **Cooldown** — how long a player must wait between uses (default 5 hours; set to 0 to disable the cooldown)
- **Require admin approval** — when enabled, `/unstuck` queues the request for an admin to review instead of teleporting immediately. Admins see pending requests via a bell icon in the panel header and can approve or deny each one
- **Discord webhook** — optionally post an alert to Discord whenever a new `/unstuck` request is queued for approval

Dead players can't use `/unstuck` — they're asked to respawn first.

---

## How It Works

1. A player types a `/command` in any chat channel (Local, Global, Squad, Admin)
2. ggCON detects the command instantly — no log polling delay
3. The command handler runs and sends a response back to the player as an in-game chat message or notification

### Cooldown

Most slash commands share a **5-second per-player cooldown** to prevent accidental double-triggers and spam. If a player sends a command during cooldown, it is silently ignored. (Some commands set their own longer cooldown — for example `/unstuck`.)

---

## Plugin Slash Commands

Plugins can register their own slash commands. Examples from the built-in plugins:

[Loot Drops](plugins.md#loot-drops) registers a command for each configured drop pack:

- `/starter` — claim the "Starter Kit" pack
- `/event-drop` — claim the "Event Drop" pack

The command name is configurable per pack.

[Quarter Master](plugins.md#quarter-master) registers:

- `/shop` — sends the player their personal storefront URL with a one-time login PIN

[Kill Feed](plugins.md#kill-feed) registers:

- `/leaderboard` (alias `/top`) — shows a top-10 leaderboard in chat; optional window argument (`24h` / `7d` / `30d` / `all`)

[Taxi Service](plugins.md#taxi-service) registers:

- `/taxi` — lists destinations and fares; `/taxi <name>` books a ride; `/taxi cancel` cancels a pending ride and refunds the fare

See each plugin's documentation for details on its commands.

---

## Custom Slash Commands

Admins can create their own slash commands with predefined responses — no plugin required. Go to **Settings → Slash Commands** in the web panel to add commands like `/discord`, `/rules`, or `/website`.

For each custom command, you configure:

- **Command name** — the word after `/` (e.g., `discord`)
- **Response text** — the message sent back to the player
- **Color** — the chat color of the response (Yellow, White, Cyan, Green, Red, Orange)
- **Prefix** — optional text prepended to the response (e.g., `[INFO]`)

The response text supports two placeholders that are filled in when the command runs:

- `{playerName}` — the name of the player who ran the command
- `{steamId}` — that player's Steam ID

Custom commands are registered at startup and when settings are saved. They appear in `/help` output alongside built-in and plugin commands.

!!! note "Overriding built-in commands"
    A custom command may reuse the name of a built-in or plugin command (for example `help`) to replace its response — useful for translating or rewording built-in replies. Removing the custom command restores the original. Two custom commands cannot share the same name.

---

## Listing registered commands

The [`GET /slash-commands.json`](http-api.md#get-slash-commandsjson) endpoint returns every registered slash command grouped by source (core, custom, and each plugin) — the same data shown in `/help`.

---

## Requirements

Slash commands work automatically — no additional configuration is needed. They are detected directly from the game engine, so the Log Watcher is **not** required for slash commands to function (though it is still needed for the Chat and Logs tabs in the panel).
