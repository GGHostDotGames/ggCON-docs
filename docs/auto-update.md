# Auto-Update

ggCON includes a built-in update system that lets you check for and install updates directly from the web panel — no manual file downloads required.

## How It Works

1. **Check** — ggCON compares your running version against the latest available version
2. **Stage** — the new DLL is downloaded and saved alongside the current one
3. **Apply** — on the next server restart, a pre-start script swaps the new DLL in before the server loads

Only a single server restart is needed to apply the update.

## Update Banner

Shortly after you log in, ggCON checks for updates and re-checks every few minutes while the panel stays open. A small dot appears on the **Settings** navigation item when a new version is available (it turns blue once staged), so you'll notice even on another tab. Open Settings to see the full banner showing the available version.

Click **Stage Update** to download the new version. The banner confirms when the update is staged and ready.

The server also checks for updates in the background on startup and periodically, even with no panel open.

!!! note
    The update is not applied until the server restarts. You can continue using the current version until you're ready to restart.

## Critical Updates

Some updates are marked as **critical** (force deploy). When a critical update is released, ggCON downloads and stages it automatically in the background — no admin needs to open the panel. You only need to restart the server to apply it.

## Major Updates

When a larger version jump is available, the banner turns orange and warns that your current version may lose license support soon — update promptly.

## What's New

The first time you open the panel after an update, a one-time **What's New** popup appears with a link to the changelog. It shows once per version.

## Settings Panel

You can also check for updates from the **Settings** panel:

1. Click the **gear icon** in the navigation bar
2. Select **Updates**
3. View the current version and check for updates

