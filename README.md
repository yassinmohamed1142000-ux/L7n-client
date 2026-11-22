# Reaper Client

*Formerly "L7N Client".* A client-side quality-of-life mod for **Minecraft Java Edition 26.1.x**, made for **Fabric**.
Version 2.0.0, licence **GPL-3.0-or-later**.

Reaper gives you a clean menu, a configurable HUD and a few everyday tools. It is built to be used openly:
it does not hide from servers, it does not try to get past an anti-cheat, and the cheat-style features of the old
L7N client are gone (see [Fair play](#fair-play)).

## Read this first: what is tested and what is not

This source tree was written in a workspace that could not download Minecraft, Fabric Loom or JDK 25. So:

| | Status |
|---|---|
| The logic (modules, settings, saving, profiles, menu and HUD models, waypoints, sorter, screenshots, chat tools, commands, server rules, Baritone bridge) | **Tested.** 14 suites, 3,947 checks, plain JDK, no game needed. All pass. Run them with `./gradlew coreTests` or `tools/run-core-tests.sh`. |
| The code that talks to the game (`mc/`, `mixin/`, `ReaperClient`) | **Type-checked only**, against hand-written stand-ins for the game's classes (`tools/check-adapters.sh`). Every game call was also compared with the source of other mods for 26.1.x: [`docs/API-AUDIT.md`](docs/API-AUDIT.md) lists each one with its evidence and what is *not* confirmed. |
| `./gradlew build` with the real Minecraft 26.1.2 | **Not run.** There is no compiled `.jar` in this package; you build it once on your machine ([BUILD.md](BUILD.md)). A compile error in the game layer is possible, and BUILD.md says what to do then. |
| Running in the game (menu, HUD, keys, zoom, fullbright, crosshair, waypoints, screenshots, sorter, mixins) | **Not done.** [BUILD.md](BUILD.md) has a play-test checklist. |
| Mojo Launcher, other launchers, other mods | **Not tested.** Reaper is an ordinary Fabric mod with no launcher-specific code, so any launcher that runs Fabric for 26.1 should load it, but that is an expectation, not a result. |
| The Baritone bridge with the real Baritone mod | **Not tested.** Its logic is tested against a stand-in with the same API shape. |

Nothing below should be read as "verified in the game" until you have done the play-test.

`docs/previews/` holds four pictures of the menu and the HUD that the test suite draws on a headless canvas. They are
**not screenshots from the game**: the canvas uses a stand-in font and a plain background, so the real look will differ
in details.

## Requirements

* Minecraft Java Edition **26.1.x** (`fabric.mod.json` asks for `~26.1`; built against 26.1.2)
* Java **25** (the game's own runtime for 26.1)
* Fabric Loader **0.18.4** or newer
* Fabric API **0.145.4** or newer
* Optional: the separate **Baritone** mod for the same game version (only used by the automation bridge)

## Install

1. Build the jar (see [BUILD.md](BUILD.md)): `build/libs/reaper-client-2.0.0.jar`. Ignore the `-sources.jar`.
2. Put it in the `mods` folder of a Fabric 26.1 profile, next to Fabric API.
3. Start the game and press **Right Shift** in a world (or type `/reaper`).

Reaper writes its files to `config/reaper/` (see [Files](#files)). Nothing else on your disk is touched except the
screenshots folder you point it at (it reads and, on request, renames or deletes pictures in the game's own
`screenshots` folder).

## What is in it

21 modules in five windows. Defaults are chosen to be quiet: only FPS, Coordinates, Direction, Armor Status,
Notifications, Zoom, Waypoints, Inventory Sorter and Chat Tools start switched on.

**Menu (ClickGUI).** Draggable, resizable, collapsible windows, one per category, plus Profiles and Waypoints
windows. Search over every module and setting. Eight themes (Reaper, Midnight, Ocean, Rose, Forest, Ember, Mono,
Daylight), your own accent colour, GUI scale, optional animations and tooltips. Every module can have its own key
(toggle or hold). Settings save by themselves a moment after you change them.

**HUD** (category HUD; each element has text colour, shadow, background, size, anchor and offset, and can be dragged
in the HUD editor):

| Element | Shows |
|---|---|
| FPS Counter (on) | frame rate |
| Coordinates (on) | position |
| CPS Counter | clicks per second |
| Ping | latency to the server (`--` while unknown; only on multiplayer) |
| Sprint Indicator | sprinting, sneaking, flying or swimming (display only; it never changes your movement) |
| World Info | dimension, biome, light level, time of day |
| Direction (on) | which way you face |
| Armor Status (on) | worn armour with durability |
| Durability | wear of the item in your hand, plus a low-durability warning |
| Keystrokes | movement keys and mouse buttons |
| Module List | the modules that are switched on |
| Notifications (on) | short messages such as "Profile loaded" |
| Nearby Waypoints | the closest saved waypoints with distance and direction |

**Visual**

* *Custom Crosshair*: cross, cross with dot, dot or ring; size, thickness, gap, colour, outline. First person only.
  Replacing the crosshair also replaces the vanilla attack-strength indicator.
* *Zoom* (on): hold **Z** (rebindable in Controls). Scroll to change the factor, optional smooth zoom, optional lower
  mouse sensitivity while zoomed. Your hand and held item are not zoomed.
* *Fullbright*: brightens dark areas. Your own brightness setting is remembered and restored when you switch it off,
  leave the world or close the game, and also after a crash.

**Utility**

* *Waypoints* (on): per world and server, shown as on-screen markers with name and distance, with a "remember where I
  died" option. Manage them in the Waypoints window, with `/reaper waypoint` or with the "Add waypoint here" key
  (unbound by default).
* *Screenshot manager*: thumbnails of the game's screenshots with preview, rename, copy path, delete and open folder
  (button in the menu, or `/reaper screenshots`). At most 32 small thumbnails are ever held in memory.
* *Inventory Sorter* (on): a Sort button in the inventory and chest screens. It merges stacks and orders items using
  ordinary container clicks, one per tick by default.
* *Chat Tools* (on): time stamps in front of chat messages (24 or 12-hour, optional seconds), a notification when
  somebody writes your name or one of your extra words, and a "Clear chat history" button. Display only; nothing is sent.

**Client**

* *Interface*: theme, accent, scale, animations, compact rows, tooltips.
* *Profiles*: create, copy, rename, delete, export to and import from the clipboard as text (`REAPER1:...`).
* *Server rules*: see below.

**Automation (optional, off by default)**

* *Baritone Bridge*: "Start mining" and "Stop everything" buttons, and `/reaper baritone goto|mine|stop`. See below.

## Fair play

The rules this client is built to:

* **No anti-cheat bypasses.** Reaper does not detect, probe, evade or manipulate any anti-cheat, and it is not
  written to be hard to detect. It is **not undetectable and must never be described as such.**
* **Nothing is hidden from server staff.** No spoofed brand or client name, no hidden modules, no packet editing.
  Reaper contains no code that builds or sends its own packets.
* **No unfair combat or movement cheats.** The previous version's kill aura, criticals, trigger bot, crystal and anchor
  aura, totem automation, flight, speed, step, no-fall, spider, jesus, scaffold, freecam, ESP and finder modules, vanish
  and staff detection (and the rest of the 48 modules) were removed. [CHANGELOG.md](CHANGELOG.md) lists them.
* **You decide what a server allows.** Reaper cannot know a server's rules, so you tell it, per server, in
  *Client > Server rules* (or `/reaper server standard|strict|open`):

| Mode | Meant for | Available |
|---|---|---|
| **Standard** (default on multiplayer) | servers with no special rules for these things | everything except automation |
| **Strict** | servers or tournaments that ask for a vanilla-like client | menu, HUD, crosshair, chat tools, screenshots, profiles. **Off:** Fullbright, Zoom, waypoint markers, inventory sorting, automation |
| **Open** | servers whose rules explicitly allow automation | everything, including the Baritone bridge |

  Singleplayer (and a LAN world you host) is always Open. Joining a Strict server shows a notice. If a server hides
  coordinates (the `reducedDebugInfo` setting), Reaper hides Coordinates, World Info, waypoints and waypoint markers,
  and treats an unknown state on multiplayer the same way.
* **Please read the server's rules yourself.** Even "Standard" allows things some servers do not, for example
  Fullbright, and the Inventory Sorter clicks faster than a person normally would. When in doubt, choose Strict.
* **Baritone is a separate mod.** Reaper only hands the text of a command to Baritone's own command handler when you
  press a button or type a `/reaper baritone` command. It is off by default, needs Open mode, and the Stop button
  always works. Whether a server allows Baritone is for you to find out.

## Keys and commands

Keys (Controls screen, heading "Reaper Client"):

| Key | Default |
|---|---|
| Open Reaper menu | Right Shift |
| Zoom (hold) | Z |
| Open HUD editor | unbound |
| Add waypoint here | unbound |

Each module can also be given its own key in the menu (Keybind row; Esc cancels, Backspace clears).
There is also a small "Reaper" button on the pause screen.

Commands (`/reaper` is a client command; it is not sent to the server):

```
/reaper                      open the menu          /reaper hud           open the HUD editor
/reaper screenshots          open the screenshot manager
/reaper toggle <module>      switch a module on or off     /reaper modules   list the ones that are on
/reaper profile [list | save | load <name> | new <name> | delete <name>]
/reaper server [standard|strict|open]        show or set the rules for the server you are on
/reaper waypoint [add <name>|list|remove <name>]
/reaper baritone goto <x> <y> <z> | mine <block> | stop      (needs Baritone and Open rules)
/reaper status               /reaper help
```

## Files

```
config/reaper/client.json        active profile, window positions, server rules, brightness safety value
config/reaper/profiles/*.json    one file per settings profile ("Default" at first)
config/reaper/waypoints.json     your waypoints, grouped by world or server
```

Each file is written under a temporary name and moved into place, and the previous good version is kept next to it as
`*.bak`. If a file is damaged, Reaper reads the `.bak` instead; if that is unreadable too, it uses the defaults and
writes a warning to the log. Files over 1 MB are refused. Delete `config/reaper` to start fresh.

## Performance

* The HUD reuses its text and buffers. A unit test runs it with every element switched on and measures what the HUD
  code itself allocates: **0.0 bytes per frame** while nothing changes, and about 870 bytes per frame while the player
  moves (numbers being reformatted). That measurement uses a canvas that draws nothing, so the game's own drawing calls
  are not in it.
* Settings are saved once, 1.5 seconds after the last change (10 seconds at the latest), not on every slider step.
* There are no background threads, except one thumbnail reader that is started when the screenshot manager opens and
  stopped when it closes.
* Every mixin is optional (`"required": false`). If one cannot be applied, for example on a game version it was not
  written for, the game still starts and only the feature behind it stops working.

No frame-time measurement was made, because the game could not be run here, so there is no measured claim about
frames per second.

## Build from source

See [BUILD.md](BUILD.md). In short: install JDK 25, run `./gradlew build` (`gradlew.bat build` on Windows), take
`build/libs/reaper-client-2.0.0.jar`.

## Credits and licence

Reaper Client is free software under the **GNU General Public License, version 3 or (at your option) any later
version**; the full text is in [LICENSE](LICENSE) and travels inside the jar.

The code was written for this project. Where it uses a technique known from another GPL-3.0 client, it says so:
the Fullbright brightness override (`mixin/OptionInstanceMixin`) and the sensitivity hook for Zoom
(`mixin/MouseHandlerMixin`) follow the approach of [Wurst](https://github.com/Wurst-Imperium/Wurst7) and, for the
first, [Meteor Client](https://github.com/MeteorDevelopment/meteor-client); [Zoomify](https://github.com/isXander/Zoomify)
and others served as references for how 26.1 code looks. Their source was used to check names and signatures
(`docs/API-AUDIT.md`), not copied wholesale. Thanks to the authors of Wurst, Meteor, Sodium, Mod Menu, malilib, Zoomify,
Fabric and Baritone, whose public source made the checking possible.

Reaper Client is not affiliated with Mojang Studios or Microsoft. "Minecraft" is their trademark.
