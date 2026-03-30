# RCON

ggCON implements the [Valve Source RCON protocol](https://developer.valvesoftware.com/wiki/Source_RCON_Protocol), compatible with any standard RCON client.

## Enabling RCON

Add the following to your `ggCON.ini`:

```ini
RconEnabled = true
RconPort    = 27020
```

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

## Connecting with mcrcon

[mcrcon](https://github.com/Tiiffi/mcrcon) is a command-line RCON client that works with ggCON.

**Interactive session:**

```bash
mcrcon -H 127.0.0.1 -P 27020 -p change_me_now
```

**Single command:**

```bash
mcrcon -H 127.0.0.1 -P 27020 -p change_me_now "#ListPlayers"
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
