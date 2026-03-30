# Getting Started

## Requirements

- A [GG Host SCUM Server](https://www.gghost.games/store/scum-server){target=_blank}

## Installation

### 1. Open Mod Manager

In your GG Host game panel, navigate to **Mod Manager** in the left sidebar.

![Mod Manager in sidebar](assets/images/install-sidebar.png)

### 2. Install GG Mod Loader

ggCON requires the **GG Mod Loader** to be installed first. Click **Install** next to GG Mod Loader and wait for it to complete.

![Mod Manager — both mods not installed](assets/images/install-mod-manager.png)

Once GG Mod Loader shows "Installed", you can proceed to install ggCON.

![GG Mod Loader installed](assets/images/install-loader-done.png)

### 3. Install ggCON

Click **Install** next to **ggCON**. You will be prompted to enter a strong **RCon Password** — this is the password you will use to access the web panel and API.

![ggCON install prompt](assets/images/install-ggcon-prompt.png)

Click the green **Install** checkmark to confirm.

### 4. Set your password

After installation, go to **Configuration Files** in the left sidebar. Under the **ggCON** section at the bottom, click **Config Editor** next to `ggcon_password`.

![Configuration Files with ggCON password](assets/images/install-config-files.png)

Enter a strong password and save. This is the only configuration required — ggCON ships with sensible defaults and all other settings can be managed through the web panel once you're logged in.

### 5. Start your server

Start (or restart) your SCUM server. ggCON will activate automatically.

## Access the web panel

The easiest way to open the panel is from the **ggCON Web Panel** shortcut in the left sidebar of your GG Host game panel — it logs you in automatically.

![ggCON Web Panel shortcut](assets/images/install-panel-shortcut.jpg)

You can also access it directly at:

```
https://ggcon.gghost.games/s/<serviceId>/panel
```

All settings — IP restrictions, command filtering, logging, Discord webhooks, and more — can be configured from the panel's **Settings** tab.

See [Web Panel](web-panel.md) for full documentation.

## Verify the mod is running

You can also test the health endpoint directly. Replace `<server-ip>` with your server's IP address and `<port>` with the HTTP port (default: `8081`):

```bash
curl http://<server-ip>:<port>/health
```

Expected response:

```json
{
  "ok": true,
  "mod": "ggCON",
  "version": "0.11.0",
  "service": "http",
  "running": true
}
```

## Make your first API call

Fetch the current player list:

```bash
curl -H "X-Password: yourpassword" http://<server-ip>:<port>/players.json
```

Run an admin command:

```bash
curl -X POST \
     -H "X-Password: yourpassword" \
     -H "Content-Type: application/json" \
     -d '{"command": "#ListPlayers"}' \
     http://<server-ip>:<port>/command
```

## Enabling RCON

To also accept RCON connections (compatible with mcrcon and similar clients), add to your `ggCON.ini`:

```ini
RconEnabled = true
RconPort    = 27020
```

!!! note "Two separate ports"
    ggCON uses two different ports: `Port` (HTTP, default `8081`) for the web panel and API, and `RconPort` (default `27020`) for RCON clients. Make sure you're using the right one for each use case.

See the [RCON](rcon.md) page for client setup instructions.
