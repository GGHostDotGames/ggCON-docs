# NPC System

!!! info "Coming soon as a plugin"
    The NPC system has been moved out of ggCON core and will be available as the **AI NPCs** plugin. See [Plugins](plugins.md#ai-npcs) for details.

ggCON's AI NPC system allows you to create characters that interact with players through in-game chat. NPCs can respond to player messages, execute admin commands, and maintain conversation history per player.

## Requirements

- The **AI NPCs** plugin (coming soon — see [Plugins](plugins.md))
- **LogWatcherEnabled** enabled in the panel's **Settings** tab (NPCs monitor chat via the log watcher)
- An [Anthropic API key](https://console.anthropic.com/) for Claude

## Quick Start

1. Open the web panel and go to the **NPCs** tab
2. Enter your Anthropic API key
3. Enable the master switch
4. Enable one or more NPCs
5. Join the server and type the NPC's trigger word in chat (e.g., `@warden`)

## How It Works

1. A player types a message in chat containing the NPC's trigger word (e.g., `@warden`)
2. ggCON detects the trigger via the log watcher
3. The player's message (with conversation history) is sent to the Claude API
4. Claude responds with a message and optional admin commands
5. ggCON delivers the response as an in-game chat message and executes any commands

## Configuration

NPC configuration is stored in `npcs.json` (created automatically on first startup in the config directory). You can edit it directly or use the web panel's NPCs tab.

### Master Settings

| Field | Type | Description |
|---|---|---|
| `enabled` | bool | Master switch — disables all NPCs when false |
| `api_key` | string | Your Anthropic API key |

### Per-NPC Settings

| Field | Type | Description |
|---|---|---|
| `name` | string | Display name shown in chat (e.g., "Warden") |
| `trigger` | string | Chat trigger word (e.g., "@warden") |
| `system_prompt` | string | Instructions that define the NPC's personality and behavior |
| `model` | string | Claude model to use (e.g., "claude-sonnet-4-6") |
| `max_tokens` | int | Maximum response length (default: 1024) |
| `cooldown_ms` | int | Minimum milliseconds between responses per player (default: 10000) |
| `max_history` | int | Number of conversation turns to remember per player (default: 10) |
| `enabled` | bool | Enable/disable this specific NPC |
| `allowed_commands` | array | List of admin commands the NPC can execute. Empty array = all commands allowed |

## Default NPCs

ggCON ships with two example NPCs:

### The Warden

A ruthless prison warden character that interacts with players in-character. The Warden can:

- Respond to polite or rude players with appropriate reactions
- Punish insolence with weather changes, zombie spawns, and knockouts
- Send private or broadcast messages in various colors

**Trigger:** `@warden`

### Debug Assistant

A compliant helper that translates plain-language requests into admin commands. Useful for testing.

**Trigger:** `@debug`

**Example:** Typing `@debug spawn 3 zombies near me` will generate and execute the appropriate `#SpawnZombie` commands.

## Response Format

NPCs must respond with valid JSON in this format:

```json
{
  "message": "Your chat response text",
  "message_color": "Yellow",
  "commands": ["#SetWeather 1", "#SpawnZombie BP_Zombie_Military_Normal_Male"],
  "use_broadcast": false
}
```

| Field | Description |
|---|---|
| `message` | The text shown in chat (required, keep under 200 characters) |
| `message_color` | Chat color: Yellow, White, Cyan, Green, or Red |
| `commands` | Array of admin commands to execute (can be empty) |
| `use_broadcast` | `true` to send to all players, `false` for just the speaker |

## Command Allowlist

The `allowed_commands` array restricts which admin commands an NPC can execute. This is a safety measure to prevent NPCs from running destructive commands.

```json
"allowed_commands": ["#SpawnZombie", "#SetWeather", "#SetTime", "#Knockout"]
```

- Only commands matching the allowlist prefix will be executed
- An **empty array** (`[]`) means the NPC can execute **any** command
- Commands not in the allowlist are silently ignored

## Available Admin Commands

ggCON automatically provides NPCs with a reference of all available SCUM admin commands, including correct syntax and valid parameter values. This reference is appended to the NPC's system prompt, so the AI knows exactly which commands exist and how to use them.

## Creating Custom NPCs

To create a custom NPC, add a new entry to the `npcs` array in `npcs.json`:

```json
{
  "name": "Trader",
  "trigger": "@trader",
  "system_prompt": "You are a friendly trader on the island...",
  "model": "claude-sonnet-4-6",
  "max_tokens": 512,
  "cooldown_ms": 5000,
  "max_history": 6,
  "enabled": true,
  "allowed_commands": ["#MessagePlayer", "#Broadcast"]
}
```

Tips for system prompts:

- Define the NPC's personality clearly
- Specify the JSON response format
- List example interactions
- Set behavioral rules (e.g., "never reveal you are an AI")
- Keep messages short — game chat has limited display space
- Reference "the commands listed at the end of this prompt" for command usage — ggCON appends the full command reference automatically

## Web Panel Management

The **NPCs** tab in the web panel provides:

- Master enable/disable switch
- API key configuration
- Per-NPC enable/disable toggles
- Activity log showing recent NPC interactions (trigger, response, commands executed)

Changes made in the panel are saved to `npcs.json` and take effect immediately via `#ReloadConfig`.
