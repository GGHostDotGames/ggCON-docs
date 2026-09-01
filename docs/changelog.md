# Changelog

Release notes for ggCON, newest first.

!!! tip "Staying up to date"
    ggCON checks for updates automatically and can stage them for the next server restart — see [Auto-Update](auto-update.md). Critical updates stage themselves.

---

## 0.15.0 — September 1, 2026

### New Features
- **ggCON now tells you when SCUM updates your server.** Game updates often arrive unannounced, and a new SCUM build can change things ggCON relies on. From now on, the moment ggCON sees the game version change it posts a one-off notice in your Discord log and audit channels naming the old and new version, and warns that compatibility with the new build has not been re-verified yet — so an unexplained oddity right after an update no longer takes days to connect to its cause. It announces each change once, not on repeat. Server status also reports the game version it was last verified against, for anyone building dashboards or monitoring on top of it.

### Improvements
- **Stash 'n Dash: drop messages now name the reward, and cabinet locations can be pasted straight into a teleport.** The new-drop announcement, the win announcement and `/stash` all show what the drop pays out (when it pays out items), and `/stash` now prints each cabinet as its site name followed by three plain numbers — X, Y and height — instead of a bracketed pair with no height.
- **Copy a Steam ID from anywhere it's shown.** The copy button next to Steam IDs now also appears in the player popup opened from the All Players list and on squad member rows — the two places it was missing — and its touch target is larger, so it's easier to hit on a phone. Clicking a squad member's Steam ID link now only opens their Steam profile; it no longer also pops up the member details behind it.

### Fixes
- **The panel's mod list no longer goes missing.** On some browsers the list of installed mods could come up empty — and sometimes a mod's own page would stay blank when you clicked it — with no error shown and no way back except reloading the page, sometimes repeatedly. Fixed on our side: nothing to install, and no update needed.
- **New in Settings: Live Vehicle Tracking (optional).** Switching it on makes the vehicle list and map track the world directly — vehicles that are destroyed or deflated disappear within half a minute instead of lingering as phantom markers, and the vehicle API's "last used" time is filled in. It is **off** by default in this release while we finish testing it on the busiest servers; your save file is never touched either way.
- **Quarter Master: delivery zones can no longer be silently lost when saving.** Zones with brackets or quotes in their names no longer delete the zones listed after them, saving immediately after opening Settings can no longer wipe the zone list (you now get a clear "still loading" warning instead), and zones with a zero radius now show a warning naming how many were skipped rather than silently disappearing. Zone names with quotes also display fully in the editor again.
- **Quarter Master: kill rewards recover on their own.** If the server's log folder wasn't ready the moment the shop started up, kill rewards stayed off for the rest of the session with no way back except a restart. They now keep looking every few minutes and switch themselves on as soon as the folder appears — kills from the gap are still paid.
- **Quarter Master: saving Settings can no longer clear your per-class kill rewards.** In a rare timing case — where the override list failed to reload while a save was already in progress — the save could wipe every override and still report "Saved!". The save now sends exactly the list you were looking at when you clicked Save.
- **Quarter Master: the delivery-zone and kill-reward editors now say when a list could not be loaded.** Previously a failed load looked identical to "you have none configured", which was alarming; the editors now show a clear amber note explaining the list was not loaded and will not be included when you save.
- **Quarter Master: package Import now imports every package.** Previously a multi-package import silently imported only the first package and could create a stray category named after it. If you used Import in the past, check your category list for leftovers.
- **Kill Feed: the Discord map snippet now shows where the victim died.** Trap and mine kills previously pointed at a map area far from the kill site — often several map squares off. The image now always matches the sector and coordinates shown next to it, for every kill type, and trap kills no longer show a made-up distance.
- **Kill Feed: kills made by a player whose profile name is "Unknown" were incorrectly shown and counted as trap kills.** They now display and count as regular player kills, including on the leaderboard, with their real distance.
- **Stash 'n Dash: hand-ins that merge into another stack are now counted, and never silent.** If you dropped a second stack of the same item into a cabinet while the first was still being collected, the two merged in-game and part of your delivery could disappear without being counted and without a word. The dealer now counts what he actually takes at the moment he takes it, so a merged delivery is credited to the stack that survives. And the two cases that used to end in silence now get an answer: one when your delivery blended into another stack, one when he could not collect the item at all and it is still sitting in the cabinet. Tip that still applies: hand stacks over one at a time.
- **Stash 'n Dash: the drops list now shows which drop is actually running.** The green dot only ever meant "this drop is allowed to be picked", which read as "this drop is live" — so several drops could look active at once. The running drop is now marked ACTIVE, the column is labelled Enabled, and both the drops list and the Control tab spell out that cabinets stay out between drops and anything deposited then is taken as an uncredited freebie.
- **Sending a message no longer freezes the server for a few seconds.** Every in-game announcement sent while nobody was online — automatic messages, scheduled messages, kill announcements and messages sent from the panel — locked up the whole server for two to five seconds while it looked for someone to send to, and a handful of kills at once could stack those pauses into a freeze long enough for players to notice. That search is now effectively instant, so announcements cost no noticeable time at all. Servers with in-game kill announcements switched on benefit the most.
- **A message sent while nobody is online is no longer reported as a failure.** It now comes back as sent successfully with a clear "no players online — 0 recipients" note, instead of an error that made automatic and scheduled messages look broken on a quiet server. A message that genuinely cannot be delivered while players ARE online still reports an error, as before.
- **Stash 'n Dash: saving cabinet sites now keeps every site, whatever characters are used in a site name.** A site name containing a square bracket used to delete that site and every site listed after it, while still reporting a successful save — so the loss was easy to miss. Names can now contain any characters, including brackets, braces and quotes, and they are stored and shown exactly as you typed them. Deliberately clearing the list still works as before. If a save can't be accepted for any reason, nothing is changed and you get a clear message explaining why. **Please check your Cabinet Sites list** — any sites lost this way cannot be recovered and will need re-adding.
- **Stash 'n Dash: editing a drop no longer wipes its rewards.** Saving a drop while sending only some of its details used to reset its rewards, duration, weight and on/off state back to defaults. Saved drops now keep everything you didn't change. (Editing from the panel was never affected.)
- **Stash 'n Dash: saving Settings is now all-or-nothing.** If one value can't be accepted, nothing is saved and you're told which one — instead of some settings being stored while the save reports a failure.
- **Stash 'n Dash: clicking [+ At player] twice no longer adds the site twice.** The button now disables itself while it is fetching the position.
- **New squads and squad joins now show up on their own.** A squad created — or joined — while the server was running stayed missing from the panel's Squads tab until an admin pressed Refresh or the server restarted, which also meant anything checking whether two players are squadmates was working from the old list. The squad list now refreshes itself roughly every two minutes, so a new squad or a new member appears within that time with nothing to click. Removing a member or deleting a squad still takes effect immediately, and the Refresh button behaves exactly as before. As part of the same fix, a squad list that can't be read for a moment no longer comes back empty — the last known list is kept until the next successful read.
- **Kill Feed: a quotation mark in an announcement template no longer cuts the rest of it off.** A template like `{killer} said "GG" to {victim}` saved as `{killer} said \` — everything from the quote onwards was thrown away, and the save still reported success. Quotes, backslashes and every other character now save and display exactly as typed. **Please check your Kill Feed announcement templates** — any that were cut short this way need re-typing.
- **Kill Feed and Trap Feed: settings saved from a script or integration no longer wipe the settings you didn't send.** Two ways this went wrong, both silently and both answering "saved successfully". Sending settings in the ordinary style with a space after each colon — which is what most Python tooling produces by default — blanked every announcement template and every Discord webhook URL. And sending a change to a single setting reset all the others back to defaults, which included switching PvP and trap announcements off. Settings now save exactly what was sent and leave everything else untouched, whatever formatting the request uses, and a request that can't be read is refused with a reason instead of being half-applied. Sending an empty value still clears a field, as before. **Please check your Kill Feed and Trap Feed settings** — templates, colours and both Discord webhooks — and re-enter anything that has gone blank. Saving from the panel was never affected.
- **Kill Feed: delayed PvP kill posts are no longer lost when the server restarts.** With the PvP announcement delay switched on, kills waiting their turn are held on disk until they're due — and a restart could quietly drop some of them, so those announcements never arrived. The whole queue is now restored, with each post's text intact.
- **Stash 'n Dash: the [+ At player] button now uses the player's current position.** Previously it used the position from when the Cabinet Sites tab was opened, so a player who had moved produced a site at their old spot. It now reads their live position at the moment you click, and tells you clearly if the player has gone offline or has no position yet — instead of silently using stale coordinates.
- **Loot Drops: a quotation mark in a pack's name or description no longer cuts the rest of it off.** A pack named `The "Big" Drop` saved as `The \` — everything from the quote onwards was thrown away, and the save still reported success. Quotes, backslashes and every other character now save and display exactly as typed. **Please check your Loot Drops pack names and descriptions** — any that were cut short this way need re-typing.
- **Loot Drops: saving a pack from a script or integration no longer wipes the parts you didn't send.** A save that left out the pack's allow list silently emptied it — turning a restricted pack into one anybody could claim — and the same save also cleared the pack's item list, description and expiry date and switched the pack off, all while answering "saved successfully". A save now changes only what it actually sends and leaves everything else alone. Deliberately sending an empty allow list still makes a pack public, exactly as before, and saving from the panel was never affected. **Please check your Loot Drops packs** — allow lists, items and expiry dates — and re-enter anything that has gone missing.
- **Stash 'n Dash: cabinets no longer stop appearing because of a single player.** After a restart, cabinet placement locked onto the first player who logged in and never tried anyone else — so if that one player couldn't be used (an admin flying in drone mode, for example), no cabinets appeared at any site until they logged out, while the log repeated a placement failure every few seconds. Placement now tries each online player in turn and uses the first that works, remembering them for next time. If nobody online can be used, it says so **once** — in the log and as a note on the Stash 'n Dash page — instead of repeating the same error, and it resumes on its own within seconds of a usable player being available.
- **Stash 'n Dash: handing in more of the right item than the drop needs now gets its own reply.** Bringing a bigger stack than the dealer asked for used to get the wrong-item answer — "not quite what I asked for" — even though you had brought exactly the right thing. He now says plainly that the extra stays with him. What happens to the extra is unchanged: anything over what the drop still needs is kept and not credited, and the part that counts is credited as before.
- **Loot Drops: clearing claims can no longer delete every claim on the server by accident.** A clear request that named a pack in a slightly different way — an id sent as text instead of a number, a zero, or a request that couldn't be read at all — was quietly treated as "clear everything": it deleted every claim for every pack and reported success. Those requests are now refused with a reason and delete nothing. The panel's Clear All button, the per-pack and per-player clears and "clear older than N days" all behave exactly as before.

---

## 0.14.3 — August 25, 2026

### Fixes
- **Updated for the SCUM 1.3.3.0 game update.** The game update changed internals ggCON relies on, which left the panel's command list empty on every server. This release restores full compatibility, including the two commands the update added.
- **Icons for the new 1.3.3.0 items.** The items added by the game update — including the Research Facility keycard, sportbike parts and the new food and trophy items — now show their proper thumbnails in the panel and shop.

---

## 0.14.2 — July 30, 2026

### Improvements
- **`/who` can now be switched off.** A new Settings option — *Allow /who (online player list)* — controls whether players can list who is currently online. It matters most on PvP servers, where being able to check who is on (or that the server is empty) takes away an advantage you may want players to earn. It stays **on** by default, so nothing changes unless you turn it off. When it is off, a player who tries `/who` is told the list is disabled on this server rather than getting no reply at all, and the command no longer appears in `/help`. The setting applies immediately — no restart needed.

---

## 0.14.1 — July 30, 2026

### New Features
- **Server Messages** — a new Settings card for automatic chat announcements, a gentler alternative to Server Notifications (which take over the player's screen). **Rotating** cycles through your list of tips on a timer; **Scheduled** sends a message once a day at a set time (e.g. a nightly restart warning at 00:00, an event reminder at 20:30). Both can run at the same time, and both are off until you switch them on.
- Players can now type `/who` in chat to see who's currently online — handy on PvE servers for spotting players from other squads. Works for everyone, no admin rights needed.
- The vehicle API now reports when each vehicle was last used, so you can find abandoned vehicles and build clean-up or wipe tooling around real activity instead of guesswork.

### Fixes
- **Stash 'n Dash: drop cabinets are placed again.** After SCUM's 1.3.2.1 hotfix the event could not place any cabinet, so rounds could not run. It now waits for the game to finish loading the cabinet asset and starts placing on its own. The Stash 'n Dash tab also tells you when placement is paused and why — either it is still waiting for the asset, or it is waiting for a player to be online (a connected player is required to place cabinets)
- **Player skills are working again.** SCUM's 1.3.2.1 hotfix changed how the game stores character skill data, which left the skills list empty everywhere — on the web panel, in `players.json` and in `players/{steamId}.json` — for every player on every server. Skills now read correctly again, with no configuration needed. Attributes were unaffected throughout
- Web panel: a player's attributes (STR, CON, DEX, INT) now show in their own section on the player card. Previously they were drawn as part of the skills panel, so if that player's skills were unavailable the attributes disappeared too — even though they were being reported correctly by the API
- Spawn Plus now tells you whether a new player's first-deploy teleport actually happened. Previously the log only said it was about to try, so a teleport that silently didn't happen looked identical to one that worked. You now get a clear result — including when no destination is set
- Stash 'n Dash: the Clear Cabinets button now also removes leftover drop cabinets the event lost track of, and the event retries removing leftovers on every rotation — previously a leftover cabinet could stay in the world with no way to delete it from the panel. Saving drop sites now also warns you when a site sits within a few meters of an existing container, which could interfere with the drop cabinet during a round
- Giving or spawning an item for a player now tells you the truth. Previously the panel and API reported "Spawned" for every attempt — even when the game refused the item (some items, like appliances, can't be spawned), the player was offline, or the spawn failed — so you'd see success while the player got nothing. You now get a clear reason instead, e.g. "'Refrigerator' is blocked by the game and cannot be spawned". Spawning entities reports honestly too.
- Shop purchases and loot claims of an item your server has blocked from spawning are no longer silently marked as delivered. The item now stays claimable and reports an honest failure, instead of being consumed while nothing actually spawned.

---

## 0.14.0 — July 13, 2026

### Improvements
- New API actions for automated events: clear all items or all zombies within a radius of a chosen map coordinate, and spawn a razor at a chosen coordinate. These run through the same load-safe path as coordinate item spawning and are naturally paced so an event bot can fire them in bursts. (Requires a player to be online.)
- Stash 'n Dash: the Settings tab now explains that auto-start brings the event back after a server restart, and the Stop button asks for confirmation and tells you how to turn it off permanently

### Fixes
- Player moderation (Silence, Unsilence, Mute, Unmute) now takes effect in-game and reports honestly: the panel/API previously said the action succeeded even when the game rejected it. These actions now run as an online admin (never as the target themselves), and if the game reports a problem — or no other admin is online to perform it — you get a clear error instead of a false success
- ggHaul: appliances now spawn facing the correct direction — the Kitchen Stove, Lathe Machine, and both Drill Presses previously faced the wrong way when placed (the Fridge was already correct). Already-placed appliances keep their old facing; delete and re-place to fix them
- Web panel: the Server Notifications editor in Settings now accepts SCUM's standard Notifications.json format — previously, saving a valid notifications file was rejected with a "Must be a JSON array" error
- Fixed a server freeze that could hit busy servers when a plugin removed an item or appliance (for example Stash 'n Dash collecting a deposit, or deleting a ggHaul appliance) — the server could lock up for many seconds or stay frozen until restarted. Removals now also complete reliably instead of timing out
- Smoother under load: the panel and plugins no longer briefly stall each other when players join the server, and a teleport issued while the server is very busy now fails cleanly and quickly instead of holding up other panel actions for up to a minute
- Web panel: a player's Teleport window now shows your saved destinations right away on a freshly opened panel — previously they only appeared after you had visited the Settings tab once
- ggHaul: removing a placed appliance now reliably takes it out of the world, instead of sometimes leaving it in-game while it disappears from the panel. If no players are online, removal now tells you to try again once someone is connected (rather than losing track of the appliance) — appliances removed this way also stay gone after a server restart

---

## 0.13.17 — July 2, 2026

### Improvements
- Startup stability: after a server restart, ggCON now waits for the game to finish loading before it starts reading live server data — so player, vehicle and squad info may take a few extra seconds to appear on boot, and the panel briefly shows a "starting" state during that window
- Appliances: you can now place a Kitchen Stove, Lathe Machine or Drill Press as well as a Fridge — pick the appliance type when adding a placement

### Fixes
- Web panel: the Squads tab now loads and refreshes on its own — no more needing to press Refresh to see your squads after opening the panel
- Kill Feed: switching from the Leaderboard back to the Live Feed no longer leaves the view broken, and the mini-map no longer tries to load map tiles that don't exist
- Appliances: a placement that wasn't Completed before a server restart now shows as "Gone · restarted before Complete" instead of looking active, clearing it no longer flashes an error in the game chat, and the Add window now warns that placements only survive a restart once Completed

---

## 0.13.16 — June 30, 2026

### Improvements
- Stash 'n Dash: the `/stash` command now shows each cabinet's site name (e.g. "A2 Church Ruins") next to its coordinates, so players can find drops without reading grid coordinates

### Fixes
- Restored the admin command list and API Explorer autocomplete in the panel after the latest SCUM game update (the update had left the command list empty on updated servers — command execution itself was unaffected)
- Stash 'n Dash: depositing items into drop cabinets no longer freezes or stutters the server, and the items you deposit are now reliably counted toward the drop — fixes a stall (and missed rewards) that could hit busy or long-running servers when several items were deposited in quick succession
- Fixed a server freeze (and the resulting automatic restart) that could affect servers with a large number of vehicles — the vehicle tracker no longer stalls the server while refreshing live vehicle data
- `/unstuck` now drops you at ground level instead of keeping your current height — no more landing in mid-air and taking fall damage when the ground is lower where you're teleported
- Stash 'n Dash: drop cabinets now spread randomly across your enabled sites and change location each rotation, instead of always appearing at the first site in your list

---

## 0.13.15 — June 28, 2026

### Plugin Published!
- **Stash 'n Dash** — a contraband dead-drop race for your players. From the new **Stash 'n Dash** panel tab, define "drops" (the items players must round up) and the map locations where drop cabinets appear, then flip it on. The server hides rotating drop cabinets around the map; players race to deposit the full required item list into a cabinet, and the first to complete it wins your configured reward. Includes weighted-random or sequential drop selection, configurable cabinet count, rotation and time limits, in-character status whispers, an **Active Cabinets** map view with one-click teleport, and a player `/stash` command to find the current drop. Ships off until you switch it on, and comes pre-loaded with example drops and locations you can edit or replace

### Improvements
- Web panel: faster, steadier live updates — the panel no longer gets stuck in a slower refresh mode after a brief connection hiccup (it now recovers on its own), and more admins can keep the panel open at the same time without it bogging down
- Player conditions now show real severities: the player-card Conditions panel reads true values for hunger, thirst, radiation sickness, hypothermia and other afflictions (many previously read 0%), and is redesigned to mirror the in-game medical monitor — symptoms listed on their own, conditions grouped by treatment stage (Untreated / Stabilization / Recovery) with severity classes and bars. Players carrying many conditions at once (more than 20) previously showed an empty Conditions panel — now fixed
- Friendly item names: the item API now includes a readable display name for each item (e.g. "Painkillers - Blister"), used across the Stash 'n Dash pickers and item lists
- The player data API now reports each player's facing direction (yaw)
- Loot Drops: the Claim list now pages through your **full** claim history (older claims are no longer hidden behind the most-recent cap), and a new "Clear older than N days" button lets you bulk-remove old one-time claim records
- NPC Tracker: puppets now show their combat state, the kill log records where the fatal hit landed, and sentry health bars read correctly
- Web panel: the Status tab now surfaces more world and performance detail (time-of-day period, day-length speed, sun intensity, fog, and a worst-frame timing card), the Flags tab shows your server's flag rules at a glance (element capacity, influence radius, overtake/decay periods, multiple-flags), and a player's card shows their squad capacity and a leader badge

### Fixes
- Flags: element counts now show each flag's real capacity instead of an inflated server-wide maximum
- Vehicles: cars owned by players with very large account IDs now show the correct owner instead of "Unknown"
- Loot Drops: you no longer lose a claim if you disconnect before the items are delivered
- Player Spawn Plus: the "spawn on ground" setting is now read correctly even when written with a space (e.g. `= True`)

---

## 0.13.14 — June 24, 2026

### New Features
- New **Integrations** tab: a directory of community-built tools and integrations that work with ggCON — browse them by category, search, and submit your own
- New **API Explorer** tab: a built-in, searchable reference for the ggCON HTTP API with a live "Try it" console — run read-only calls right from the panel, copy ready-made code snippets (curl / JavaScript / Python), and a confirmation step guards any action that changes your server

### Improvements
- Stability and performance: fewer crashes, and reduced rubberbanding/lag on busy servers — most noticeable around players joining

### Fixes
- Admin command output (for example the confirmation from kicking a player through the API) is no longer shown to a random online player when "Suppress Command Output" is enabled — it's now properly hidden
- Kill Feed: the scheduled leaderboard webhook no longer shows a literal "\n" between rows — the board now formats with proper line breaks
- Loot Drops: the Claim list now loads far more history (previously only the most recent 100 claims), so older claims can be searched and filtered from the panel
- Player Spawn Plus: players who die and respawn during an in-game event (Tab 8) are no longer teleported out to the respawn point and re-kitted — they stay in the event where they respawned

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
