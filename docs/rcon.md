# RCON

ggCON implements the [Valve Source RCON protocol](https://developer.valvesoftware.com/wiki/Source_RCON_Protocol), compatible with any standard RCON client.

## Enabling RCON

Add the following to your `ggCON.ini`:

```ini
RconEnabled = true
RconPort    = 27020
```

!!! note "GG Host servers"
    On a GG Host server the RCON port is usually preset to your **HTTP/panel port + 1** (for example panel `5381` → RCON `5382`), so check your panel or `ggCON.ini` for the actual value rather than assuming `27020`.

By default, RCON binds to the same address as `BindAddress`. To bind RCON to a different address:

```ini
RconBindAddress = 0.0.0.0
```

## Authentication

RCON authentication works the same as the HTTP API:

1. The client's IP must be in `AllowedIPs` or `AllowedCIDRs`
2. If `RequirePassword = true`, the RCON password must match `Password`

Rejected connections receive an auth failure response (`id = -1`) and are immediately closed.

!!! warning "Brute-force lockout"
    Five failed authentication attempts within 60 seconds locks the IP out for 5 minutes. All connection attempts from that IP are rejected during the lockout window.

!!! note "Re-authentication after config changes"
    If the RCON password or the IP allowlist is changed while a client is connected, ggCON drops the existing session. The client must reconnect and authenticate again with the new credentials.

## Connecting with mcrcon

[mcrcon](https://github.com/Tiiffi/mcrcon) is a command-line RCON client that works with ggCON.

RCON is a raw TCP protocol, so the client connects **directly to your server's IP and RCON port**. It does not go through the `ggcon.gghost.games` web proxy (that handles only the HTTP API and web panel).

!!! note "GG Host RCON port"
    On GG Host servers, the RCON port is usually your **HTTP/panel port + 1**. For example, if your panel/HTTP port is `5381`, your RCON port is `5382`. Check your panel or `ggCON.ini` for the exact value rather than assuming `27020`.

**Interactive session:**

```bash
mcrcon -H <server-ip> -P <rcon-port> -p <your-password>
```

**Single command:**

```bash
mcrcon -H <server-ip> -P <rcon-port> -p <your-password> "#ListPlayers"
```

## Protocol details

ggCON implements the standard Source RCON packet format:

```
int32  size   — byte count of the remainder
int32  id     — request ID, echoed in responses
int32  type   — packet type
char[] body   — null-terminated UTF-8 string
char   pad    — second null byte
```

**Packet types:**

| Direction | Type value | Meaning |
|---|---|---|
| Client → Server | `3` | `SERVERDATA_AUTH` |
| Client → Server | `2` | `SERVERDATA_EXECCOMMAND` |
| Server → Client | `2` | `SERVERDATA_AUTH_RESPONSE` |
| Server → Client | `0` | `SERVERDATA_RESPONSE_VALUE` |

**Auth flow:**

1. Client connects
2. ggCON checks the client IP against the allowlist — if rejected, sends `AUTH_RESPONSE id=-1` and closes
3. Client sends `SERVERDATA_AUTH` with the password in the body
4. ggCON checks the password — on failure, sends `AUTH_RESPONSE id=-1` and closes
5. On success, sends `AUTH_RESPONSE id=<original request id>`
6. Client sends `SERVERDATA_EXECCOMMAND` packets; ggCON responds with `SERVERDATA_RESPONSE_VALUE` containing the JSON result

## Command responses

RCON commands return the same JSON as the [HTTP POST /command](http-api.md#post-command) endpoint. Parse the response body as JSON to read `ok`, `lines`, `message`, etc.

## Limits

| Setting | Default | Config key |
|---|---|---|
| Maximum concurrent clients | 16 | [`RconMaxClients`](config-reference.md#rconmaxclients) |
| Idle connection timeout | 30 seconds | [`RconIdleTimeoutSecs`](config-reference.md#rconidletimeoutsecs) |
| Maximum packet body size | 4 096 bytes | *(not configurable)* |

Connections beyond `RconMaxClients` are rejected until an existing client disconnects. Idle connections are automatically dropped after `RconIdleTimeoutSecs` to prevent automation tools from exhausting connection slots.
