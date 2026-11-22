# Changelog

Versions are the mod version in `gradle.properties`. Dates are the day the change was made.

## 2.0.0 "Reaper Client" (2026-11-21)

A complete rebuild of L7N Client (1.1.0 for Minecraft 26.1; 1.0.0 for 1.21.11) under a new name, as a fair-play client.
The previous code was replaced rather than patched, and all 48 of its modules were reviewed against the fair-play rules
(see "Removed"). The result is 107 main source files (11,431 lines) and 25 test files (7,088 lines).

**Verification status of this release, in one place.** The logic is tested (14 suites, 3,947 checks, all pass). The code
that talks to the game was type-checked against stand-ins and compared with other mods' source, but **was never compiled
against the real game and never ran in it**. No `.jar` is included, because the real build could not be run in the
workspace this was made in. Read "Not verified" at the end before relying on anything.

### Renamed: L7N Client is now Reaper Client

| | Before (L7N 1.1.0) | Now |
|---|---|---|
| Name in the menu, pause-screen button, chat, log | L7N CLIENT, "L7N", `[L7N]` | REAPER CLIENT, "Reaper", `[Reaper]` |
| Mod id and assets namespace | `l7n` | `reaper` |
| Java packages | `com.l7n.client` | `com.reaper.client.{core,mc,mixin}` |
| Entry class | `L7NClient` | `ReaperClient` (plus `ReaperCore`, `ReaperScreen`, `ReaperCommands`, new in 2.0.0) |
| Resources | `assets/l7n/{lang,textures/gui/logo.png}` | `assets/reaper/{icon.png, lang/en_us.json, textures/gui/logo.png}`, `reaper.mixins.json` |
| Key names and category | `key.l7n.gui`, `key.l7n.panic`, category `l7n:main` | `key.reaper.{menu,zoom,hud_editor,waypoint}`, category `reaper:main` ("Reaper Client") |
| Commands | chat messages starting with `.` (`.help`, `.toggle`, ...) | `/reaper ...` (a client command, never sent to the server) |
| Settings location | `config/l7n-client.json` | `config/reaper/` (`client.json`, `profiles/`, `waypoints.json`) |
| Profile share text | none | `REAPER1:` |
| Jar, archive and project names | `l7n-client-*.jar` | `reaper-client-2.0.0.jar`, `Reaper-Client-2.0.0.zip` |
| Fabric metadata | name "L7N CLIENT", id `l7n` | name "Reaper Client", id `reaper`, `GPL-3.0-or-later` |
| Look | the old L7N logo | a new default theme "Reaper" (near-black with a red accent), a new original logo and icon |

Old L7N settings are not read (the format changed with the rebuild); `config/l7n-client.json` can be deleted. The source
code and resources contain no `l7n`; only the documents mention it, as the former name.

### Compatibility

* Minecraft Java **26.1.x** (built against 26.1.2), Fabric Loader **>= 0.18.4**, Fabric API **>= 0.145.4**, Java **25**.
  Declared in `fabric.mod.json` (`minecraft ~26.1`, `fabricloader >=0.18.4`, `fabric-api >=0.145.4`, `java >=25`).
* Client-side only (`"environment": "client"`): servers need nothing installed. Reaper builds no packets of its own; the
  one thing it makes the game send is the ordinary inventory click of the sorter, and Baritone, if you use it, does
  what Baritone does.
* No launcher-specific code, so Mojo Launcher and other launchers that run Fabric 26.1 should load it. **Not tested.**
* Baritone is an optional extra (`suggests`), reached by reflection; there is no build dependency.

### Removed: the cheat-style features (fair play)

The previous version had 48 modules, some of which sent forged packets. These are gone, with their code:

| Group | Removed |
|---|---|
| Combat | Kill Aura, Trigger Bot, Crystal Aura, Anchor Aura, **Criticals** and **Mace** (both sent forged position packets), Auto Totem, Legit Totem (swapped by packet after a random delay "to look human") |
| Movement | Flight, Speed, Step, **No Fall** (sent forged on-ground packets), Spider, Glide, Elytra Target, Elytra Boost, Freecam, High Jump, Fast Fall, Jesus, Parkour, and the always-on Sprint |
| Seeing more than the game shows | ESP, Nametags (name, hearts, armor, totem), Radar (nearby player list), Base Finder, Chest Finder, Spawner ESP, Storage ESP, Shulker Finder, Block ESP |
| Staff detection and server-specific tools | Staff List and Anti Vanish (look for staff, including vanished ones and admins watching you), Inspector (CoreProtect lookups), Anti AFK (small movements so you are not kicked) |
| Automation | Scaffold, Limitless Builder, Auto Jump, Auto Sneak, Auto Respawn |
| Replaced by the Baritone bridge | Auto Walk, Auto Mine |
| Commands and keys | `.tp` and `.vclip` (moved the player by setting its position), `.lookat`, `.staff`, `.blockesp`, `.friend`, `.say` and the rest of the dot commands; the panic key (switched every module off and stopped Baritone), whose job is now done by "Disable all modules" in the menu and the bridge's Stop button |

A search of the current source for these names and for packet-sending code finds nothing.

Features that were kept were rewritten: **Fullbright** (now gated by the server rules; it remembers your own brightness
and restores it), **Coordinates** (was Coords; honours the server's reduced-debug-info setting, and has the old
Speedometer as a "Speed" option), **Keystrokes**, **Module List** (was ArrayList), **Sprint Indicator** (shows the
state; it no longer sprints for you). The Watermark module became the name in the menu's top bar.

### Added: the fair-play framework

* **Server rules** chosen by the player, per server address: Standard (default; automation off), Strict (Fullbright,
  Zoom, waypoint markers, inventory sorting and automation off) and Open (all). Singleplayer and LAN-hosting are always
  Open. A notice appears when you join a Strict server; the menu's top bar shows the mode;
  `/reaper server standard|strict|open`.
* Nothing is detected, hidden or spoofed: Reaper does not try to recognise a server's software or anti-cheat, and says
  so in the README. It is not described anywhere as undetectable.
* Modules that the rules switch off show as blocked, with a tooltip saying why, instead of silently doing nothing.
* The HUD and waypoints honour a server that hides coordinates (reduced debug info); if that cannot be read on
  multiplayer, they are treated as hidden.
* The Baritone bridge (below) is off by default, only works in Open mode, and its Stop button always works.

### Added and changed: menu (ClickGUI)

* Rebuilt on a Minecraft-free model: draggable, resizable, collapsible, scrollable windows per category, plus Profiles
  and Waypoints windows; window positions are saved.
* Search over every module and setting; tooltips; compact rows option.
* Eight colour themes (Reaper, Midnight, Ocean, Rose, Forest, Ember, Mono, Daylight) and a custom accent colour; GUI
  scale; animations that can be switched off.
* Setting controls: switches, sliders (right click to type a value), choices, colour picker, text fields, key capture
  with Toggle or Hold per module, action buttons. Settings can show or hide depending on others.
* Key capture: click, then press a key; Esc cancels, Backspace clears.
* "Disable all modules" and a two-click "Reset all settings".

### Added and changed: HUD

* 13 elements: FPS, CPS, Coordinates, Direction (compass strip or text), Armor Status, Durability (with low-durability
  warnings), Ping, Keystrokes, Sprint Indicator, World Info, Module List, Notifications, Nearby Waypoints.
* Each element: text colour, shadow, background and its colour, size 0.5x to 3x, screen anchor and offsets, reset position.
* HUD editor (`/reaper hud` or its key): drag elements, with snapping to the screen edges and centre; positions are
  stored in the profile.
* Notification toasts (info, success, warning, error), used by profiles, durability, chat mentions and server rules.
* World Info can show the server address, but this is off by default so that streams do not leak it.

### Added: quality-of-life features

* **Custom crosshair**: cross, cross with dot, dot, ring; size, thickness, gap, colour, outline, optional opening while
  moving.
* **Zoom**: hold a key (Z), scroll to change, smooth zoom, lower sensitivity while zoomed. The field of view of the world
  changes; the hand does not.
* **Fullbright**: brightness well beyond the slider, remembering your own value and restoring it when you switch it off,
  leave the world or quit, and at the next start after a crash. Without its mixin it is limited to the slider's maximum.
* **Waypoints**: per world and server, on-screen markers with names and distance, death waypoints (the last 3 by
  default), a Waypoints window, the "Nearby Waypoints" HUD list, commands and a key.
* **Screenshot manager**: thumbnail grid, preview, rename, copy path, delete, open folder. At most 32 thumbnails
  (about 2.6 MB) exist whatever the folder holds; pictures over 48 MB are not decoded.
* **Inventory sorter**: Sort button in the inventory and chest screens; merges stacks and orders them by type, name or
  stack size, with ordinary container clicks (one per tick by default, at most four). It stops on the first
  inconsistency or when the screen closes. Off on Strict servers.
* **Chat tools**: time stamps (24 or 12-hour, optional seconds, colour), mention alerts (your name and extra words, with
  a minimum gap), "Clear chat history". Display only.
* **Profiles**: create, copy, rename, delete, load, save; export to and import from the clipboard (`REAPER1:` text,
  size-limited and validated on import). Every module, key, setting and HUD position is part of a profile.
* **Commands**: `/reaper` with help, gui, hud, screenshots, modules, toggle, profile, server, waypoint, baritone, status;
  tab completion; a pause-screen "Reaper" button.
* **Baritone bridge** (optional, off by default): "Start mining" and "Stop everything" buttons and
  `/reaper baritone goto|mine|stop`. It hands the command text to Baritone's own command handler, through reflection on
  Baritone's public API, and does nothing else; the Baritone mod itself is installed separately. Only in Open mode.

### Changed: performance

* The menu and HUD models are plain Java and cache what they draw (lists, fitted texts, measured widths). The HUD
  allocates **0.0 bytes per frame** in a steady state (about 870 bytes per frame while the player moves), measured by a
  unit test with every element on and a canvas that draws nothing. The game's own drawing is not in that figure.
* Saving is debounced (1.5 s after the last change, 10 s at the latest) and atomic, with a `.bak` of the previous file;
  nothing is written per slider step.
* No background threads, except one thumbnail reader that lives only while the screenshot manager is open.
* The HUD is registered with Fabric's `HudElementRegistry`; elements that are off are skipped.
* A failure in a HUD element or screen is caught and logged once, and after repeated failures that part is switched off
  instead of crashing the game. The mod entry point catches everything: if Reaper cannot start, the game starts without it.

No frame-time measurement was possible, so none is claimed.

### Changed: code quality and architecture

* Split into a Minecraft-free **core** (86 files, 9,259 lines, about 80 % of the main code) and a thin **game layer**
  (`mc`, `mixin`, `ReaperClient`: 21 files, 2,172 lines). Only the game layer touches Minecraft, Fabric API, Mixin or LWJGL.
* Duplicate and unused code of the old version removed; the old `Gfx`, `Data`, `Util`, `Config`, `ModuleManager`,
  `CommandManager` and the rest replaced by `core` classes with one job each (`Module`, `ModuleRegistry`, `Settings`,
  `ClientConfig`, `ProfileStore`, `ServerPolicy`, `CommandCenter`, ...).
* Fabric API where it exists: `HudElementRegistry` / `VanillaHudElements`, `KeyMappingHelper`,
  `ClientCommandRegistrationCallback`, `ClientTickEvents`, `ClientLifecycleEvents`, `ClientPlayConnectionEvents`,
  `ClientReceiveMessageEvents`, `ScreenEvents`, `Screens`.
* Five mixin classes, all optional (`"required": false`, `defaultRequire: 0`): zoom field of view, mouse sensitivity /
  buttons / scroll, chat time stamps, brightness override, container screen accessor. If one fails to apply, only its
  feature is lost, and `/reaper status` says which hooks were seen working.
* Exceptions: config and file access never throw into the game (reads return nothing and log; writes return false);
  reflection failures fall back to a safe behaviour; every game callback is guarded.
* No new dependencies. Gson, SLF4J and Brigadier come with the game.
* GPL-3.0-or-later kept: the licence text is in the repository and inside the jar, and `fabric.mod.json` declares it.
  The brightness override and the sensitivity hook follow techniques known from Wurst and Meteor, and say so in the
  code and in the README.

### Changed: build

* New `build.gradle` for Fabric Loom 1.15.5 (`net.fabricmc.fabric-loom`): no `mappings`, no remapping (26.1 is not
  obfuscated), Java 25, sources jar, licence in the jar, version injected into `fabric.mod.json`.
* Gradle wrapper 9.4.0 (scripts and wrapper jar included), versions pinned in `gradle.properties`.
* `./gradlew coreTests` runs the Minecraft-free tests. Gradle's own `test` and `compileTestJava` are switched off, so
  `build` cannot fail because of a test.
* Output: `build/libs/reaper-client-2.0.0.jar`.

### Added: tests and tools

* `src/test`: 14 suites, 3,947 checks (core, GUI, HUD, HUD editor, features, waypoints, panels, sorter, screenshots,
  chat, adapter logic, Baritone, commands, wiring), run with a plain JDK on a headless raster canvas; the GUI and HUD
  tests also write PNG previews. No test framework is needed.
* `tools/run-core-tests.sh` and `tools/check-adapters.sh` (type-check of the game layer against 115 hand-written
  stand-ins in `tools/api-stubs`: 107 files, no errors, no warnings).
* `docs/API-AUDIT.md`: for each of the 191 game members that are used, the evidence (189 seen in 26.1.x projects, 2 in
  a 26.3 project) and the list of what is not confirmed, with the fix for each.
* `README.md`, `BUILD.md` (build steps and a play-test checklist) and this file.

### Known limitations

* A custom crosshair also replaces the vanilla attack-strength indicator; it is drawn in first person only.
* Zoom does not change the field of view of the held item.
* A renamed singleplayer world starts a new, empty waypoint list (worlds are told apart by name).
* The brightness override is written for the 26.1.x version of the game's option class; on a later game version that
  mixin is skipped and Fullbright is limited to the slider's maximum.
* English only.

### Not verified (please do the play-test in `BUILD.md`)

* **`./gradlew build` has never been run.** Loom, Minecraft 26.1.2 and Fabric API could not be downloaded in the
  workspace. A compile error in the game layer is possible; `docs/API-AUDIT.md` section 4.1 has the one known
  candidate and its fix.
* **Nothing has run inside the game**: no screen has been seen, no key pressed, no mixin applied. This includes menu
  drawing and clipping at other GUI scales, HUD draw order, crosshair centring, zoom, Fullbright, the sorter's clicks on
  a real container, screenshots and the clipboard.
* Three things are reached by name with no confirmation: the clipboard read, the player's latency, and the server's
  reduced-debug-info flag. Each has a safe fallback (own paste buffer; no Ping element; hidden coordinates).
* Mojo Launcher and combinations with other mods are untested.
* The Baritone bridge was tested against a stand-in with Baritone's API shape, not against the real Baritone for 26.1.

## 1.1.0 and earlier: L7N Client

1.1.0 ported the 1.0.0 client from Minecraft 1.21.11 to Fabric 26.1 with the same 48 modules. 1.0.0 targeted 1.21.11.
