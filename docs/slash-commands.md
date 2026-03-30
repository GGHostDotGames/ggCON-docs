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

Loot Drops:
  /starter — Claim the Starter Kit
  /event-drop — Claim the Event Drop
```

Commands from plugins are grouped under the plugin name so you can quickly find what's available.

---

## How It Works

1. A player types a `/command` in any chat channel (Local, Global, Squad, Admin)
2. ggCON detects the command instantly — no log polling delay
3. The command handler runs and sends a response back to the player as an in-game chat message or notification

### Cooldown

All slash commands share a **5-second per-player cooldown** to prevent accidental double-triggers and spam. If a player sends a command during cooldown, it is silently ignored.

---

## Plugin Slash Commands

Plugins can register their own slash commands. For example, the [Loot Drops](plugins.md#loot-drops) plugin registers a command for each configured drop pack:

- `/starter` — claim the "Starter Kit" pack
- `/event-drop` — claim the "Event Drop" pack

The command name is configurable per pack. See each plugin's documentation for details on its commands.

---

## Requirements

Slash commands work automatically — no additional configuration is needed. They are detected directly from the game engine, so `LogWatcherEnabled` is **not** required for slash commands to function (though it is still needed for the Chat and Logs tabs in the panel).
