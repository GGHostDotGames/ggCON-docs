# Log Watcher

The LogWatcher service monitors SCUM server log files in real time and exposes them through the `GET /logs` HTTP endpoint. It watches both the main `SCUM.log` and all per-category log files in the `SaveFiles/Logs/` directory (chat, login, admin, economy, kill, and more).

---

## How it works

1. **Startup delay** — After the mod loads, LogWatcher waits a configurable number of seconds (default 30) before scanning for log files. This gives the SCUM server time to create its log files.

2. **Auto-discovery** — If `LogWatcherSources` is not configured, the watcher automatically discovers all log file prefixes in `SaveFiles/Logs/` (e.g. `chat`, `login`, `admin`, `economy`, `kill`, `famepoints`, `loot`, `gameplay`, etc.) and always includes the main `SCUM.log`.

3. **Polling** — Every 1 second, the watcher checks each tracked file for new content. New lines are pushed into a shared ring buffer (max 2000 lines).

4. **Backlog** — On startup, the watcher seeds the ring buffer with the last N lines from each file (configurable via `LogWatcherBacklog`, default 100).

5. **Rotation detection** — If a log file shrinks (e.g. `SCUM.log` is rotated on server restart), the watcher resets and begins reading from the start of the new file.

---

## GET /logs

Returns log lines that have been captured since a given timestamp. Auth required.

**Query parameters**

| Parameter | Required | Description |
|---|---|---|
| `since` | No | Unix timestamp in milliseconds. Only lines with `t >= since` are returned. Defaults to `0` (all buffered lines). |
| `sources` | No | Comma-separated list of source names to include (e.g. `SCUM,chat,login`). Omit to return all sources. |

**Response**

```json
{
  "ok": true,
  "lines": [
    {
      "t": 1710612345123,
      "src": "chat",
      "line": "2026.03.16-18.22.07: '76561198000000001:PlayerName(1)' 'Local: hello'"
    },
    {
      "t": 1710612346124,
      "src": "SCUM",
      "line": "[2026.03.16-18.22.07:537][ 42]LogEntitySystem: Auto-save took 0.203ms"
    }
  ],
  "next": 1710612346125
}
```

| Field | Type | Description |
|---|---|---|
| `lines` | array | Array of log line objects |
| `lines[].t` | number | Capture timestamp (Unix ms) — use this for the next `since` value |
| `lines[].src` | string | Source name (e.g. `SCUM`, `chat`, `login`, `admin`, `economy`) |
| `lines[].line` | string | The raw log line content |
| `next` | number | Suggested value for `since` on your next poll (equals `max(t) + 1`) |

**Response — LogWatcher disabled**

HTTP `503`:

```json
{
  "ok": false,
  "error": "LogWatcher is not enabled"
}
```

---

## Available sources

When auto-discovery is enabled (default), the watcher finds all log file types present in `SaveFiles/Logs/`. Common sources include:

| Source | Description |
|---|---|
| `SCUM` | Main server log (`SCUM.log`) — frame stats, net events, game state |
| `chat` | Player chat messages (local, global) |
| `login` | Player login/logout events |
| `admin` | Admin command executions |
| `economy` | Currency transactions |
| `kill` | Player kills |
| `event_kill` | Event-related kills |
| `famepoints` | Fame point changes |
| `loot` | Loot spawning events |
| `gameplay` | General gameplay events |
| `violations` | Anti-cheat / rule violations |
| `base_building_destruction` | Base destruction events |
| `vehicle_destruction` | Vehicle destruction events |
| `chest_ownership` | Chest ownership changes |
| `armor_absorption` | Armor damage absorption |
| `quests` | Quest-related events |
| `raid_protection` | Raid protection events |
| `sentry` | Sentry/turret events |
| `server_notifications` | Server notification events |

!!! note
    The exact set of sources depends on your server's game version and configuration. New source types added by SCUM updates are discovered automatically.

---

## Configuration

The LogWatcher starts automatically — no configuration is needed for basic use. You can configure **LogWatcherSources** in the panel's **Settings** tab to filter which log sources are monitored (leave empty to auto-discover all available sources).

The following optional settings are INI-only and must be set in `ggCON.ini` under the `[ggCON]` section:

```ini
; (Optional) Number of historical lines to load per file on startup.
; Default: 100
; LogWatcherBacklog = 100

; (Optional) Seconds to wait before scanning for log files.
; Default: 30
; LogWatcherStartupDelay = 30
```

See [Config Reference](config-reference.md) for full details on each key.

---

## Polling example (PowerShell)

The following script polls `GET /logs` every second and prints new lines with color-coded sources:

```powershell
# Poll-Logs.ps1
# Usage:
#   .\Poll-Logs.ps1
#   .\Poll-Logs.ps1 -ServerHost 10.0.0.5 -Port 8081 -Password yourpassword
#   .\Poll-Logs.ps1 -Sources "chat,login,kill"

param(
    [string]$ServerHost = "127.0.0.1",
    [int]   $Port       = 8081,
    [string]$Password   = "",
    [string]$Sources    = ""          # comma-separated, empty = all
)

$BaseUrl = "http://${ServerHost}:${Port}/logs"
$Headers = @{}
if ($Password -ne "") { $Headers["X-Password"] = $Password }

# Start from now
$Since = [DateTimeOffset]::UtcNow.ToUnixTimeMilliseconds()

Write-Host "Polling $BaseUrl every 1s  (Ctrl-C to stop)" -ForegroundColor Cyan
if ($Sources -ne "") { Write-Host "Sources filter: $Sources" -ForegroundColor DarkCyan }
Write-Host ""

while ($true) {
    try {
        $Url = "${BaseUrl}?since=${Since}"
        if ($Sources -ne "") {
            $Url += "&sources=$([Uri]::EscapeDataString($Sources))"
        }

        $Response = Invoke-RestMethod -Uri $Url -Headers $Headers `
                        -Method Get -ErrorAction Stop

        if ($Response.lines.Count -gt 0) {
            foreach ($entry in $Response.lines) {
                $ts  = [DateTimeOffset]::FromUnixTimeMilliseconds(
                           $entry.t).LocalDateTime.ToString("HH:mm:ss.fff")
                $src = $entry.src.PadRight(26)
                $color = switch ($entry.src) {
                    "SCUM"                       { "Gray"        }
                    "login"                      { "Cyan"        }
                    "admin"                      { "Yellow"      }
                    "economy"                    { "Green"       }
                    "kill"                       { "Red"         }
                    "event_kill"                 { "DarkRed"     }
                    "chat"                       { "Magenta"     }
                    "famepoints"                 { "Blue"        }
                    "loot"                       { "DarkYellow"  }
                    "gameplay"                   { "DarkCyan"    }
                    default                      { "White"       }
                }
                Write-Host "$ts  [$src]  $($entry.line)" `
                    -ForegroundColor $color
            }
        }

        $Since = $Response.next
    }
    catch {
        $msg = $_.Exception.Message
        Write-Host "$(Get-Date -Format 'HH:mm:ss')  ERROR: $msg" `
            -ForegroundColor Red
    }

    Start-Sleep -Seconds 1
}
```

**Example output:**

```
21:34:38.650  [login                     ]  2026.03.16-19.34.08: '10.0.0.1 76561198000000001:PlayerName(1)' logged in at: X=150991.000 Y=-40018.000 Z=33531.000
21:34:38.651  [chat                      ]  2026.03.16-19.34.12: '76561198000000001:PlayerName(1)' 'Global: hello everyone'
21:34:38.652  [economy                   ]  2026.03.16-19.34.15: '76561198000000001:PlayerName(1)' bought item for 500
21:34:39.660  [SCUM                      ]  [2026.03.16-19.34.39:199][ 42]LogSCUM: Global Stats:  32.9ms ( 30.4FPS)
```

---

## Known limitations

- **SaveFiles/Logs latency** — SCUM's per-category log files (`SaveFiles/Logs/`) do not include a trailing newline on the last written line until the next log entry is appended. ggCON handles this by processing unterminated lines immediately, so there is no additional delay beyond the 1-second poll interval.

- **SCUM.log volume** — The main `SCUM.log` can be very verbose (frame stats every 5 seconds, net events, etc.). If you only need gameplay events, use the `sources` filter to exclude `SCUM` — e.g. `?sources=chat,login,kill,admin`.

- **Ring buffer size** — The internal buffer holds up to 2000 lines across all sources. On a busy server with many sources, older lines may be evicted before they are polled. Increase your poll frequency or filter to specific sources if this is a concern.
