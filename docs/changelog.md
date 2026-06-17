# Changelog

Release notes for ggCON, newest first.

!!! tip "Staying up to date"
    ggCON checks for updates automatically and can stage them for the next server restart — see [Auto-Update](auto-update.md). Critical updates stage themselves.

---

## 0.13.13 — June 17, 2026

### Plugin Published!
- **ggHaul** — a new admin tool for placing appliances in the world. From the new **Appliances** panel tab, spawn a fridge right in front of any online player (it lands on the ground, facing them), then **Complete** it to keep it permanently — it survives server restarts — or **Delete** it. To move one, delete it and add it again from a better spot. Fridges to start, with more appliance types to come

### New Features
- Taxi: you can now hide a destination from the `/taxi` list while keeping it bookable by its exact name — handy for secret or reward destinations
- Taxi: each destination can show a short description on the `/taxi` list (for example `Novice Bay ($0): Safe spawn for newcomers`)
- Taxi: optional per-player lifetime trip limit per destination (for example one free trip to a reward spot); admins can reset trip counts from the panel
- Quarter Master: players can cancel a purchased package from their stash as long as it hasn't been claimed yet, and get refunded automatically — the refund goes back in whatever currency they paid. A new admin "Cancellation refund %" setting controls how much is returned (default is a full refund)
- Loot Drops: the Claim list now has a search box, sortable columns, and filtering by pack, Steam ID, or name
- Kill Feed: the scheduled leaderboard webhook can now post the Armed NPC, Animal, and Puppet boards as well — pick any combination in the schedule settings, each posted as its own message
- Taxi: group destinations into clusters — `/taxi <group>` sends the player to a random spot in that group, you can enable or disable a whole group in one click (great for event drop-offs), and a group can share a single trip limit
- Kill Feed: Animal kills now have their own filter button in the live feed, separate from the NPC filter
- Player Spawn Plus: the respawn loadout no longer replaces a player's gear when they are revived on the spot (for example with Phoenix Tears) — it now only fires on a real respawn. This is on by default; you can turn the distinction off, and set how far a player must move for it to count as a respawn

### Improvements
- Panel: Save buttons now show a clear "Saved" confirmation in the corner, with a red message if a save fails

### Fixes
- Kill Feed: kills of NPCs, animals, and puppets no longer show a misleading weapon (you could see things like "killed with a banana" — the held item at the time, not the actual weapon). These kills now simply show who made the kill, with no weapon. Player-vs-player weapons are unchanged.
- NPC Tracker: animal and bird kills now register correctly, and live animals (crows in particular) no longer show as "dead" while they're still alive. Animal health bars are also more accurate.
- Fixed players sometimes vanishing from the live player list and map — and shop deliveries, loot packs, teleports, and Give Item failing with "player not online" even though the player was actually online
- Fixed certain weapons (the MP5 family and other load-on-demand items) failing to spawn at a map location via the bot/API — they now spawn at the requested coordinates
- Loot Drops: allow-list player names now stay visible when you reopen a pack's editor (they were showing up blank)
- Fixed some panel inputs and helper text that could appear blank or invisible
- Fixed the panel, storefront, or API sometimes staying locked out ("too many attempts") even once you started entering the correct password — the lockout now counts down and clears on its own
- Reduced crashes on servers that have a corrupted vehicle: ggCON now detects the bad vehicle, stops re-reading it, and flags it in the dashboard so it can be removed

---

## 0.13.12 — June 15, 2026

### Plugin Published!
- Player Spawn Plus — give players a configurable gear loadout automatically: on a new player's first deploy, after a character re-roll, on every respawn, or on demand from a player's detail card. First-deploy and respawn kits can teleport the player to a location you choose first. Build multiple named loadouts with worn clothing and inventory items (weapons included — they're delivered to the player's inventory, so make sure the kit includes enough storage). Separate configurable cooldowns keep respawn and re-roll kits from being farmed. Automatic loadouts require `scum.EnableSpawnOnGround=True` in your server settings (the Rules page checks this for you and explains if it isn't set). When you first turn it on, players who have already played on your server are treated as existing players — only brand-new arrivals get the new-player kit. Ships disabled by default

### New Features
- Quarter Master: new "Clear All" button in Economy → Player Balances sets every player's ration balance to zero — for resetting the shop economy after a server wipe. Requires typing CLEAR to confirm; GG Coins are not affected

### Fixes
- Fixed a rare timing issue where an item delivery could freeze the server for about 15 seconds and disconnect players
- Fixed a rare issue where recording a crash report could freeze the server for several minutes before it restarted — crash details are now saved instantly
- Quarter Master: renaming a claim/delivery zone no longer clears that zone from your packages — package zone assignments now follow the renamed zone automatically
- Item thumbnails: Military Masks / Balaclavas now show their real mask icons instead of a beanie picture. The same fix corrects about 30 other items whose thumbnail showed a lookalike item — including some ammo piles (7.62x39 vs 7.62x54R), fish fillets, harvested animal parts, and the red RK beanie

---

## 0.13.11 — June 9, 2026

### New Features
- Warning messages can now use a custom prefix / sender template (for example `Server Botcast: {message}`) so they read like an official announcement. Set it in Settings → General → Warning Prefix; leave it blank to keep sending warnings with no prefix
- Save named colour presets for warning messages in Settings → General → Saved Warning Colours, then pick one from a dropdown when sending a Warning — no more re-choosing the colour each time

### Fixes
- Further reduced server crashes that can occur when items, vehicles, or NPCs are spawned for players — for example shop deliveries, loot packs, or several players claiming at once on a busy server
- Kill Feed leaderboard: the **All** time window now loads quickly on servers with a long kill history, instead of being slow or timing out

---

## 0.13.10 — June 5, 2026

### Improvements
- Admin chat/whisper templates now support a `{playerDisplayName}` placeholder that inserts the recipient's fake IGN (their `#setfakename` alias) when they have one, and otherwise their normal character name — so one template can address VIPs by their alias and everyone else by their real name. (The existing `{playerName}` still always shows the real character name.)

### Fixes
- Fixed a server crash that could happen when a large batch of items was delivered at once — for example a big shop package or loot pack, or several players claiming at the same time on a busy server. Large item deliveries are now paced out smoothly instead of spawning everything in a single instant
- The panel's Send Message and Warning fields no longer trigger your browser's saved-login autofill popup
- Fixed repeated panel Settings saves slowly corrupting text fields that contain backslashes (e.g. file paths or message templates) — backslashes were duplicated on each save, which over time could bloat the settings file and mangle the log watcher source list
- `/unstuck` now posts to its Discord webhook every time a player uses it (previously only when admin-approval mode was enabled). Also, the saved Discord webhook fields in Settings now clearly show "✓ Saved" instead of appearing blank, so it's obvious the URL is still stored

---

## 0.13.8 — June 4, 2026

### Improvements
- Kill Feed leaderboard: the **NPC** column (and `/leaderboard npc`) is now **Armed NPC** — it counts only armed human enemies (Drifters, Guards), so animals and puppets are no longer double-counted there (they keep their own columns). Drones and sentries aren't counted. `/leaderboard armed` also works

### Fixes
- Stability: fixed a rare server crash that could happen when items, vehicles, or NPCs were spawned for a player on a busy server — for example claiming a loot pack or completing a shop purchase right as that player's session was changing. The server now delivers against the player's current session instead of crashing (and if the player has just left, the items stay claimable instead of being lost)
- Kill Feed leaderboard: the in-game `/leaderboard` (alias `/top`) and the scheduled Discord leaderboard now match the panel — previously they could come back empty or wrong for time-windowed views while the panel showed the right standings. Usage: `/leaderboard [type] [window]` — **type** = `pvp` (default), `armed`, `animal`, or `puppet`; **window** = `24h`, `7d` (default), `30d`, or `all` (e.g. `/leaderboard armed 30d`)

---

## 0.13.7 — June 3, 2026

### New Features
- Quarter Master: each shop package can now be assigned its own delivery zone, so players claim that package only inside the chosen zone (e.g. boats at the docks, planes at the airfield). Packages left on "Any zone" behave as before. Applies when "Require delivery zone" is enabled

### Improvements
- The Copy SteamID button now also appears on offline players' detail cards (previously it was only on online players)

### Fixes
- Taxi rides, shop purchases, and panel currency adjustments no longer occasionally fail with "Fare deduction failed" (or a similar error) on busy servers

---

## 0.13.6 — June 2, 2026

### New Features
- Kill Feed leaderboard now ranks NPC, Animal, and Puppet kills too — new Animal and Puppet columns on the Leaderboard tab, and `/leaderboard` (and `/top`) accept a type, e.g. `/leaderboard npc`, `/leaderboard animal 30d`, or `/leaderboard puppet`
- The events feed now shows NPC and animal kills when the NPC Tracker plugin is installed

### Improvements
- Player detail window now has a one-click Copy SteamID button next to the Steam profile link

### Fixes
- Taxi: free rides (fare set to 0) now work instead of failing with "Fare deduction failed. No charge, no ride."
- Kill Feed leaderboard no longer shows NPCs (like Drifters) as top players — only player kills are ranked

---

## 0.13.5 — June 2, 2026

### New Features
- After a ggCON update, the panel now shows a quick "What's New" popup with a link to the changelog, plus a reminder of where to send feedback and feature requests. Appears once per version

### Fixes
- Squads panel now shows each member's correct online and alive status — a living, online squad member could previously show as dead

---

## 0.13.4 — June 1, 2026

### Fixes
- Stability: fixed a rare server crash that could happen when a player disconnected at the exact moment the server was processing an action for them (admin commands, chat or kill notifications, shop currency changes, squad edits) — the server now safely skips the stale action instead of crashing
- Item deliveries no longer crawl on busy servers — loot-drop packs, shop claims, and bulk grants that could dribble out one item every several seconds now deliver at full speed again; in-game chat and live panel updates are no longer held up either
- Panel update checks and Plugin Manager are now much faster — fixed a Windows-side delay that could add up to 10 seconds per check on certain server configurations
- Kill Feed: turning off "Include map image" in the public webhook settings now stays off after a reload

---

## 0.13.3 — May 26, 2026

### New Features
- **Suppress "Spawned X" confirmations** — a new toggle in Settings → General hides spawn confirmations from the executor's in-game chat when items are handed out from the panel, RCON, or a plugin (off by default).

### Improvements
- Map sector grid lines and labels are now easy to read against any tile background.

### Fixes
- Panel login now shows a specific message (wrong password, lockout countdown, IP blocked, or network error) instead of silently bouncing back to a blank screen.
- The panel password now displays correctly in TCAdmin's Configuration Files editor.
- `#SetCurrencyBalance`, `#ChangeCurrencyBalance`, `#SetFamePoints`, and `#ChangeFamePoints` typed in the panel console now respond instantly and accept unquoted Steam IDs.
- "Loading…" item-load messages no longer spam the executor's chat when items are dispatched from the panel, RCON, or a plugin.
- "Yellow" chat is yellow again after the SCUM update, and a new "Orange" color option was added for the bright MOTD style.
- Edits to the Admin Chat Sender Prefix now save correctly.

---

## 0.13.2 — May 26, 2026

### Fixes
- **In-game chat restored after the SCUM update** — after the game's update, some servers stopped seeing chat in-game: panel broadcasts and `#MessagePlayer` reported success but nothing appeared, `/shop` no longer showed the login and password, and slash commands like `/taxi` teleported the player but skipped the chat confirmation. Once a player hit this, even SCUM's own admin replies stopped displaying until they reconnected. ggCON now matches the new chat format, so chat reaches players normally again — no admin action required.

---

## 0.13.1 — May 25, 2026

### Fixes
- **Updated for the latest SCUM build** — the admin-command system stopped responding after the SCUM update (panel admin commands, `#ExecAs`, and slash commands that run admin commands would silently do nothing). Fixed, with all other systems verified working.
- **Plugin Manager** — the "Check for Plugins" button no longer errors on certain manifest text.

---

## 0.13.0 — May 24, 2026

### Plugin Published!
- **Taxi Service** — a paid teleport service. Players use `/taxi <name>` to travel to destinations you configure, paying in in-game cash. A configurable countdown (default 120s) cancels and refunds the ride if the player takes damage, dies, or logs out, so it can't be used to escape a fight. Includes per-player cooldowns. Off by default — enable it from the plugin marketplace when you're ready.

### New Features
- **Kill Feed: leaderboard** — a new Leaderboard sub-tab ranks players by PvP kills, deaths, K/D, NPC kills, trap kills, suicides, longest shot, and favorite weapon, with selectable time windows and top-3 medals. Players can run `/leaderboard` (alias `/top`) in chat, and you can schedule a leaderboard to post to Discord daily or weekly.
- **Kill Feed: per-type Discord webhooks + PvP delay** — route PvP, NPC, Trap, and Suicide kills to separate Discord channels, and optionally delay player-kill posts by a set number of minutes. The delay queue survives a restart.
- **Kill Feed: in-game chat broadcasts** — announce kills in chat, per type, with editable templates (`{killer}`, `{victim}`, `{weapon}`, `{distance}`, `{category}`) and a color. All off by default.
- **Quarter Master: export / import packages** — back up your shop or share package templates as a JSON file. Categories are matched by name, so templates move cleanly between servers.
- **Quarter Master: delivery zones** — optionally require players to stand in a configured zone (shop counter, safe room, event point) to claim their stash.
- **Quarter Master: squad-kill penalty** — when killer and victim are in the same squad, replace the PvP reward with a configurable deduction to discourage friendly fire.
- **Quarter Master: per-class kill rewards** — set a custom reward for a specific NPC type (e.g. Razor) that overrides the broad category default.
- **Quarter Master: bundle thumbnails** — multi-item packages show a representative item icon (you choose the "lead" item) and a Bundle badge.
- **Quarter Master: open / close controls** — open or close the shop and deliveries independently, plus a per-category enable toggle.
- **Quarter Master: Grant Rations to All** — bulk-grant rations to every online player, optionally including offline players who have used the shop before.
- **Loot Drops: vehicles** — packs can now include assembled vehicles alongside items.
- **Admin chat sender prefix** — chat sent from the panel shows a configurable prefix (default `[ADMIN]` / `[WHISPER from ADMIN]`) so players know an admin is talking. Supports placeholders and per-message overrides.
- **Identity columns** — the Players and All Players lists now show Account ID, Real IGN, and Fake IGN, and the search box matches all of them.
- **`/unstuck` command** — players type `/unstuck` to free themselves from terrain glitches. Admin-configurable distance, cooldown, optional approval queue, and Discord alert.
- **Teleport: paste coordinates + saved destinations** — teleport by pasting `X Y Z`, choosing a saved destination, or clicking the map, with an optional facing direction.
- **Stack count** — Loot Drops and Quarter Master package items get a stack-size field (great for ammo and cash).
- **`#GiveItem` keyword syntax** — `#GiveItem` now accepts SCUM's native `StackCount` keyword form.

### Improvements
- **Quantity safety cap** — pack and package items are capped at 999 per slot to prevent server-crashing configurations; use the Stack field for high amounts.

### Fixes
- **Quarter Master: in-game cash now deducts immediately on purchase** — previously it waited until the player's next login.
- **No more false "player left" events on single-player servers.**
- **`#Teleport` from the panel, RCON, or API is now under a second** and handles unquoted Steam IDs.
- **Panel FPS card no longer stuck at 0.0** if SCUM was accidentally removed from the watched log sources.
