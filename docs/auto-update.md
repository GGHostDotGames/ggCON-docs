# Auto-Update

ggCON includes a built-in update system that lets you check for and install updates directly from the web panel — no manual file downloads required.

## How It Works

1. **Check** — ggCON compares your running version against the latest available version
2. **Stage** — the new DLL is downloaded and saved alongside the current one
3. **Apply** — on the next server restart, a pre-start script swaps the new DLL in before the server loads

Only a single server restart is needed to apply the update.

## Update Banner

Shortly after logging into the web panel, ggCON checks for updates. If a new version is available, a green banner appears at the top of the panel showing the available version.

Click **Stage Update** to download the new version. The banner confirms when the update is staged and ready.

!!! note
    The update is not applied until the server restarts. You can continue using the current version until you're ready to restart.

## Critical Updates

Some updates are marked as **critical** (force deploy). When a critical update is available, ggCON automatically stages it — you only need to restart the server to apply it.

## Settings Panel

You can also check for updates from the **Settings** panel:

1. Click the **gear icon** in the navigation bar
2. Select **Updates**
3. View the current version and check for updates

## API Endpoints

| Endpoint | Description |
|---|---|
| `GET /core/update-check` | Check for available updates |
| `POST /core/stage-update` | Download and stage the latest version |

See [HTTP API](http-api.md#update-endpoints) for full endpoint documentation.
