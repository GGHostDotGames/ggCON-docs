# Plugins

ggCON supports a plugin system that lets you extend the mod with additional features. Plugins are standalone modules that live in a `plugins/` folder alongside the main ggCON DLL and are loaded automatically on startup.

## How Plugins Work

Each plugin can:

- **Add a panel tab** — custom HTML UI that appears as a new tab in the web panel
- **Register HTTP routes** — add new API endpoints accessible via the ggCON HTTP API
- **Register slash commands** — add in-game chat commands (e.g. `/starter`) that players can use
- **React to events** — respond when players join, leave, or send chat messages
- **Execute commands** — run admin commands and spawn items through the host API
- **Send notifications** — deliver chat messages, warnings, and HUD overlays to players

## Version Compatibility

Plugins can declare a minimum ggCON version in their metadata. If your installed ggCON version is older than what a plugin requires, the Plugin Manager will show a **"Requires vX.Y.Z+"** notice and block installation until you update ggCON.

---

## Available Plugins

### Loot Drops

Configurable item drop packs with slash commands, per-player allowlists, and claim limits.

- **Drop packs** — create bundles of items that players can claim with a slash command (e.g. `/starter`, `/event-drop`)
- **Per-player limits** — set how many times each player can claim a pack (e.g. once per wipe)
- **Allowlists** — optionally restrict packs to specific Steam IDs
- **Expiry dates** — set an expiry time after which the pack is no longer claimable
- **Panel management** — create, edit, enable/disable, and delete packs from the Loot Drops tab in the web panel
- **Claim tracking** — view all player claims with timestamps; clear claims in bulk (e.g. after a server wipe)
- **Category filter** — item picker includes a category dropdown to quickly find items by type (Weapons, Ammunition, Food, etc.)
- **Smart delivery** — items are queued and spawned one per tick to prevent server overload

![Loot Drops Panel Tab](assets/images/panel/panel-drops.jpg)

**Slash commands:** Each pack registers its own command (configurable). Players type the command in chat to claim.

**Requires:** ggCON v0.11.0+

---

### Log Analyzer

Server performance analytics — replaces the standalone ServerLogAnalyzer desktop tool.

- **Analytics tab** — 10 interactive time-series charts: FPS, frame time, character count, prisoner count, zombie count, animal count, vehicle count, and more
- **Summary cards** — at-a-glance stats including average FPS, minimum FPS, peak zombies, peak animals, and sample count
- **Time range selector** — view data over the last 1 hour, 6 hours, 24 hours, or 7 days
- **Load Log button** — triggers incremental log read on demand
- **Persistent history** — up to 7 days of minute-resolution data retained across restarts

---

### Kill Feed

Real-time kill event feed for PvP, NPC, trap, and suicide kills.

- **Kill list** — scrollable feed showing killer, victim, weapon, distance, and kill type (PvP, NPC, Trap, Suicide)
- **Weapon icons** — dynamic icon lookup from the item catalog with a manual fallback table for common weapons
- **Detail strip** — expanded view of the selected kill event with full player info
- **Interactive map** — shows killer and victim positions with markers
- **LIVE / PAUSED mode** — auto-follow new events in LIVE mode, or freeze the list to inspect past events
- **Session clear** — kill history resets automatically when the server restarts

**Requires:** ggCON v0.11.6+

---

### Trap Feed

Real-time trap event feed showing crafting, arming, disarming, and triggering of traps.

- **Event list** — scrollable feed of trap events with player name, trap type, and action (Crafted, Armed, Disarmed, Triggered)
- **Trap icons** — trap-specific icons for each trap type
- **Detail strip** — expanded view of the selected event
- **Interactive map** — shows trap locations with markers
- **LIVE / PAUSED mode** — auto-follow new events or freeze the list

**Requires:** ggCON v0.11.6+

---

### Quarter Master

A complete in-game economy and player reward system with a web storefront — give your players something to work toward.

Admins build item packages and set prices in Rations, in-game Cash, or both. Players browse and purchase from a web storefront, then claim their items in-game. Combined with GG Coins — a cross-server currency earned by playtime — Quarter Master turns your server's economy into a reason to keep playing.

#### Admin Panel

The Quarter Master tab in the web panel is where you create and manage everything:

- **Packages** — bundle any combination of items into purchasable packages; set a name, description, and price in Rations, Cash, or both
- **Categories** — organize packages into custom categories (e.g., Weapons, Supplies, Gear); categories appear as filter pills in the storefront
- **Discounts** — set a percentage discount on any package with an optional expiry date; storefront shows the original price crossed out with a "X% OFF" badge
- **Stock limits** — cap how many times a package can be purchased; storefront shows "3 left" or "Sold Out" automatically
- **Vehicle packages** — sell vehicles through the shop; a permanent "Vehicles" category is auto-populated from the game engine's 19 vehicle types; enable individual vehicles and set prices
- **Example presets** — new installs come with 3 example categories and 5 sample packages to get started quickly
- **Economy overview** — Player Lookup tool in the Economy tab for reviewing transaction history, stash contents, and resolving disputes

#### Player Storefront

Players access their server's storefront at `shop.gghost.games`, log in with their Steam ID and a one-time PIN (generated by the `/shop` slash command in-game), and browse available packages.

- **Browse and filter** — category pills, package cards with item icons, quantity badges, and descriptions
- **Purchase flow** — select a package, confirm the price, items are added to the player's stash
- **Offline purchasing** — players can buy packages even when not connected to the server; Ration deductions happen immediately, Cash deductions are queued for next login
- **Stash** — purchased items are grouped by package in a persistent stash; players click "Claim All" in the storefront when in-game to receive their items
- **Bank Statement** — full purchase history with timestamps
- **Multi-language** — storefront available in English, Chinese, Thai, Russian, German, and Ukrainian; language picker on login and in the header

#### GG Coins

A global currency earned through playtime across all GG Host servers. The earn rate is the same globally, with special events offering multipliers (e.g. 2x weekend).

- **Earn by playing** — players automatically earn GG Coins based on time spent online; the rate is the same across all servers
- **Cross-server wallet** — GG Coins are stored in a global wallet, not tied to a single server; players who play on multiple GG Host servers accumulate coins across all of them
- **Withdraw to Rations** — players can convert GG Coins into Rations at an admin-configured exchange rate via the storefront
- **Balance display** — current GG Coin balance shown in the storefront header with a withdraw button

#### Delivery Safety

Items never get lost:

- Purchased items go to a persistent stash, not spawned immediately
- If spawning fails (e.g., player logs out mid-delivery), items return to the stash for re-claiming
- Concurrent purchase protection prevents double-charges
- Admin dispute tools: full transaction audit trail and package re-delivery endpoint

#### Server Directory

The `shop.gghost.games` landing page lists all servers running Quarter Master. Each card shows the server name, custom shop name, owner welcome message, and GG Coin exchange rate. Offline servers are shown with a status badge. Players can browse servers and jump directly to any storefront.

**Slash command:** Players type `/shop` in-game to receive their personal storefront URL with login PIN.

**Requires:** ggCON v0.12.0+

---

## Coming Soon

### AI NPCs

AI-powered NPC characters that interact with players in chat using Claude. Replaces the built-in NPC system with a standalone plugin.

- Configure trigger words (e.g., `@warden`) and NPC personalities
- NPCs respond to player chat messages with context-aware replies
- NPCs can execute admin commands (with optional per-NPC command allowlists)
- Per-player conversation history
- Configurable model, token limits, and response cooldowns

See [NPC System](npc-system.md) for details on the current built-in version.

---

## Marketplace

The plugin marketplace lets you install, update, and uninstall plugins directly from the web panel — no manual file management needed.

### Accessing the Marketplace

1. Open the web panel
2. Click the **gear icon** in the navigation bar
3. Select **Plugins**

The marketplace shows all available plugins with their current status:

- **Install** — download and install a plugin
- **Update** — update an installed plugin to the latest version (shown when an update is available)
- **Uninstall** — remove an installed plugin
- **Requires vX.Y.Z+** — shown when a plugin needs a newer ggCON version; update ggCON first

Plugin changes take effect on the next server restart.


---

## Plugin Configuration

Plugins can read configuration from `ggCON.ini` using section headers in the format `[Plugin:<id>]` (requires a server restart):

```ini
[Plugin:log-analyzer]
SomeKey = SomeValue
```

Refer to each plugin's documentation for its specific configuration options.
