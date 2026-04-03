# Config Reference

ggCON uses two layers of configuration:

1. **`ggCON.ini`** — static settings managed by GG Host (network binding, ports, RCON). These are pre-configured for your server and not directly visible to customers.
2. **`ggcon_settings.json`** — runtime settings managed through the web panel's Settings tab, applied immediately without restart
3. **`ggcon_password`** — the authentication password, set during installation and changeable from the panel

All three files live in:

```
<GameRoot>/Saved/Config/WindowsServer/
```

!!! info "You don't need to edit the INI file"
    The `ggCON.ini` file is managed by GG Host and pre-configured when ggCON is installed. All customer-facing settings are available in the web panel's **Settings** tab. The INI-only settings documented below are for reference only.

| File | What goes here | How to change |
|---|---|---|
| `ggCON.ini` | Network binding, ports, RCON | Managed by GG Host |
| `ggcon_settings.json` | Everything else (admin IDs, logging, webhooks, etc.) | Panel Settings tab |
| `ggcon_password` | Authentication password | Panel Settings tab |

---

## INI-only settings (managed by GG Host)

These settings are pre-configured by GG Host during installation. They require a server restart to take effect and are not exposed in the web panel. All keys are under the `[ggCON]` section header.

### BindAddress
**Type:** string &nbsp;|&nbsp; **Default:** `127.0.0.1`

The address the HTTP server binds to.

- `127.0.0.1` — localhost only (recommended when using a reverse proxy)
- `0.0.0.0` — all interfaces

---

### Port
**Type:** integer &nbsp;|&nbsp; **Default:** `8081`

Port for the HTTP API.

---

### PanelId
**Type:** string &nbsp;|&nbsp; **Default:** *(empty — proxy disabled)*

Identifier for the SSL panel proxy. When set, ggCON registers with the GG Host panel proxy, making the web panel available over HTTPS at:

```
https://ggcon.gghost.games/s/<PanelId>/panel
```

For GG Host servers using TCAdmin, set this to your TCAdmin Service ID (available as `![ServiceId]` in TCAdmin templates).

Leave empty to disable SSL proxy access entirely.

```ini
PanelId = 1234567
```

---

### RconEnabled
**Type:** bool &nbsp;|&nbsp; **Default:** `false`

Set to `true` to start the RCON server on mod load.

---

### RconPort
**Type:** integer &nbsp;|&nbsp; **Default:** `27020`

Port the RCON server listens on.

---

### RconBindAddress
**Type:** string &nbsp;|&nbsp; **Default:** *(inherits `BindAddress`)*

Address the RCON server binds to. Leave empty to use the same value as `BindAddress`.

---

### RconMaxClients
**Type:** integer &nbsp;|&nbsp; **Default:** `16`

Maximum number of simultaneous RCON connections. Additional connections are rejected until an existing client disconnects.

---

### RconIdleTimeoutSecs
**Type:** integer (seconds) &nbsp;|&nbsp; **Default:** `30`

Seconds of inactivity before an idle RCON connection is dropped. Prevents automation tools from exhausting connection slots by leaving sessions open.

---

## Password

The password is stored in a dedicated file — `ggcon_password` — in the same config directory. It contains a single line of plain text.

The password is required via the `X-Password` header for HTTP API and RCON authentication.

!!! warning
    Passwords are transmitted in plaintext HTTP headers. Use the SSL panel proxy or a reverse proxy with TLS if the API is exposed beyond localhost.

You can change the password from the panel's **Settings** tab or by editing the `ggcon_password` file directly and running `#ReloadConfig`.

---

## Panel-managed settings

These settings are managed through the web panel's **Settings** tab. Changes apply immediately without a restart and are persisted to `ggcon_settings.json`.

---

### AllowAllIPs
**Type:** bool &nbsp;|&nbsp; **Default:** `true`

When `true`, connections from any IP address are allowed (IP allowlist is skipped). Authentication still requires a valid password.

This is the default — ggCON ships open and relies on password authentication. Set to `false` and configure `AllowedIPs` or `AllowedCIDRs` if you want IP-level restrictions.

!!! warning
    With the default `AllowAllIPs = true`, any IP can attempt to authenticate. Make sure you set a strong password.

---

### AllowedIPs
**Type:** semicolon-separated list &nbsp;|&nbsp; **Default:** `127.0.0.1;::1`

IP addresses allowed to connect. Both IPv4 and IPv6 are supported. Only used when `AllowAllIPs = false`.

If both `AllowedIPs` and `AllowedCIDRs` are empty, the mod falls back to allowing only `127.0.0.1` and `::1`.

---

### AllowedCIDRs
**Type:** semicolon-separated list &nbsp;|&nbsp; **Default:** *(empty)*

CIDR ranges allowed to connect. Only used when `AllowAllIPs = false`.

---

### RestrictToAdmins
**Type:** bool &nbsp;|&nbsp; **Default:** `false`

If `true`, only players whose Steam ID is in `AdminSteamIDs` may be used as the command executor. If no qualifying admin is online when a command is issued, the command fails with `"no admin player is online"`.

---

### AdminSteamIDs
**Type:** semicolon-separated list &nbsp;|&nbsp; **Default:** *(empty)*

The Steam IDs of players who are permitted to act as command executors when `RestrictToAdmins = true`.

---

### PreferredSteamIDs
**Type:** semicolon-separated list &nbsp;|&nbsp; **Default:** *(empty)*

Ordered list of Steam IDs to prefer as the command executor. The mod tries each ID in order and uses the first one that is currently online.

When `RestrictToAdmins = true`, candidates must also appear in `AdminSteamIDs`.
When `RestrictToAdmins = false`, falls back to any online player if none of the preferred IDs are online.

!!! note "Per-command override with #ExecAs"
    These settings control which player is used for commands that go through the normal executor selection pipeline. The [`#ExecAs`](commands.md#execas) command bypasses this entirely — it always targets the player you specify, regardless of `RestrictToAdmins` or `PreferredSteamIDs`. The target player does not need to be an admin.

---

### LimitAdminCommands
**Type:** bool &nbsp;|&nbsp; **Default:** `false`

If `true`, only commands listed in the allowed commands file will be executed. Any command not in the list is rejected before reaching the game. See `AllowedCommandsFile` below to configure a custom path.

If the allowed commands file is missing or empty, **all commands are blocked** (fail-safe behaviour).

---

### SuppressCommandOutput
**Type:** bool &nbsp;|&nbsp; **Default:** `false` &nbsp;|&nbsp; **Recommended:** `true`

When ggCON runs a command, the player being used receives the command output in their in-game chat. Set to `true` to suppress that in-game delivery. The output is still captured and returned in the API response as normal.

This is recommended for most setups to avoid unexpected output appearing for whichever player happens to be online.

---

### SlashResponseColor
**Type:** string &nbsp;|&nbsp; **Default:** `yellow`

Color used for slash command responses sent back to the player in chat. Valid values: `yellow`, `white`, `cyan`, `green`, `red`.

---

### SlashResponsePrefix
**Type:** string &nbsp;|&nbsp; **Default:** *(empty)*

Optional prefix prepended to slash command responses in chat, e.g. `[SERVER]`.

---

### LogMinSeverity
**Type:** string &nbsp;|&nbsp; **Default:** `INFO`

Minimum severity level for the log file. Valid values (from least to most severe):

| Value | Description |
|---|---|
| `TRACE` | Verbose internal events |
| `INFO` | Normal operational messages |
| `WARN` | Warnings and recoverable issues |
| `ERROR` | Errors only |

---

### DiscordLogWebhook
**Type:** string &nbsp;|&nbsp; **Default:** *(empty)*

Discord webhook URL to receive operational log messages. Leave empty to disable Discord logging.

Log entries at or above `LogMinSeverity` are sent as Discord messages.

!!! warning
    The URL must start with `https://`. An invalid URL triggers a one-time warning in the log file and disables Discord logging.

---

### DiscordAuditWebhook
**Type:** string &nbsp;|&nbsp; **Default:** *(empty)*

Discord webhook URL to receive the audit log. Each command dispatched through ggCON generates one audit entry, regardless of `LogMinSeverity`.

Leave empty to disable Discord audit logging.

---

### LogWatcherSources
**Type:** semicolon-separated list &nbsp;|&nbsp; **Default:** *(empty — auto-discover)*

Explicit list of log sources to watch. Leave empty to automatically discover all log file prefixes in `SaveFiles/Logs/`.

`SCUM` refers to the main `SCUM.log` file. All other names correspond to prefixes of files in `SaveFiles/Logs/` (e.g. `chat` watches `chat_20260316183008.log`).

---

### LogWatcherBacklog
**Type:** integer &nbsp;|&nbsp; **Default:** `100`

Number of historical lines to load from each log file when the watcher starts. Set to `0` to only capture lines written after startup.

!!! note
    Changes to this setting require a server restart to take effect.

---

### LogWatcherStartupDelay
**Type:** integer (seconds) &nbsp;|&nbsp; **Default:** `30`

Seconds to wait after mod load before scanning for log files. This gives the SCUM server time to create its log files during startup.

!!! note
    Changes to this setting require a server restart to take effect.

---

### AllowedCommandsFile
**Type:** string (file path) &nbsp;|&nbsp; **Default:** *(empty)*

Path to the allowed commands file used when `LimitAdminCommands` is enabled. One command per line (case-insensitive). Leave empty to use `ggCON_allowed_commands.txt` in the config directory.

---

## Plugin configuration

Plugins can have their own configuration sections in `ggCON.ini` using the `[Plugin:<id>]` section header, where `<id>` is the plugin's unique identifier. Plugin configuration is INI-only and requires a restart.

```ini
[Plugin:loot-drops]
DropCooldownSecs = 300
MaxActiveDrops = 3
```

See [Plugins](plugins.md) for available plugins and their configuration options.
