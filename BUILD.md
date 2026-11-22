# Building Reaper Client

This package contains the complete source and the Gradle build, but **no compiled `.jar`**. The workspace it was written
in could not download Minecraft, Fabric Loom or JDK 25, so the real build has never been run. You run it once, on your
own machine; it takes a few minutes the first time. Please also read "What was and was not checked" at the end.

## 1. Get the jar (the short version)

You need **JDK 25** (for example Eclipse Temurin 25) and an internet connection. Nothing else: the Gradle wrapper in
this folder downloads Gradle 9.4.0, Fabric Loom 1.15.5, Minecraft 26.1.2 with its libraries, Fabric Loader 0.18.4 and
Fabric API 0.145.4+26.1.2 by itself.

Open a terminal in the project folder and check that `java -version` says 25 (if not, install JDK 25 and set
`JAVA_HOME` to it). Then:

```
Windows (Command Prompt or PowerShell):   gradlew.bat build
macOS / Linux:                            ./gradlew build
```

The mod is `build/libs/reaper-client-2.0.0.jar`. Copy that file into the `mods` folder of a Fabric profile for Minecraft
26.1.x (Fabric Loader 0.18.4 or newer, with Fabric API 0.145.4 or newer next to it). The other file, `-sources.jar`, is
for developers and does not belong in `mods`. Minecraft 26.1 is not obfuscated, so there is no remapping step: the jar the
build writes is the finished mod.

## 2. Other commands

| Command | What it does |
|---|---|
| `./gradlew runClient` | Starts a development game with the mod loaded (a standard Loom task; not run here). |
| `./gradlew coreTests` | Runs the 3,947 automated checks of everything that is not Minecraft. No game needed, takes seconds. It is deliberately not part of `build`, so a test can never stop you from getting the jar. |
| `./gradlew coreTests --args="Gui Wiring"` | Only some suites (names are in `src/test/java/.../AllTests.java`). |
| `tools/run-core-tests.sh` | The same tests without Gradle, for a plain JDK. Set `GSON=/path/to/gson.jar` first (the default path is the one of the workspace this was written in). |
| `tools/check-adapters.sh` | Type-checks the game layer against hand-written stand-ins for the game's classes (`tools/api-stubs`). Needs a JDK, Gson, SLF4J and Brigadier jars (it finds them in Gradle's cache after a first build, or clones Brigadier from GitHub). Only for developers; it does not replace the real build. |

In IntelliJ IDEA: *Open* the folder as a Gradle project, choose JDK 25 as the Gradle JVM, and run the `runClient`
configuration that Loom creates.

## 3. Versions

Everything is in `gradle.properties`:

```
minecraft_version=26.1.2
loader_version=0.18.4
fabric_api_version=0.145.4+26.1.2
loom_version=1.15.5
```

These match the minimums in `fabric.mod.json` (`fabricloader >=0.18.4`, `fabric-api >=0.145.4`, `minecraft ~26.1`,
`java >=25`). For plain Minecraft 26.1, use `minecraft_version=26.1` and `fabric_api_version=0.145.1+26.1`; for another
26.1.x patch pick the matching Fabric API build on <https://modrinth.com/mod/fabric-api/versions>. Only 26.1.2 was used as
the reference. Minecraft versions after 26.1 are not supported: the game's internals change between them (one mixin is
known to differ on 26.3, see `docs/API-AUDIT.md`).

The licence text is packed into the jar by `build.gradle`; the version number comes from `mod_version`.

## 4. If the build fails

The most likely place is the game layer, `src/main/java/com/reaper/client/mc` and `.../mixin`, which could only be
type-checked against stand-ins here. The compiler names the file and line.

1. Run it again with the output saved:
   `./gradlew build > build-log.txt 2>&1` (Windows: `gradlew.bat build > build-log.txt 2>&1`).
2. Open `docs/API-AUDIT.md`, section 4. It lists the one call that could stop the build (`Options.getCameraType()`, with
   the one-line fix) and the exact spots that were reached by reflection or could behave differently.
3. If it is something else, paste the lines from the first `error:` onwards into the chat where you got this package.
   The core (`com.reaper.client.core`, about 80 % of the code) does not touch the game, so such an error is a small,
   local fix.

If Gradle itself fails before compiling (download or plugin errors), check that the JDK is 25 and that nothing blocks
`maven.fabricmc.net`, `libraries.minecraft.net` and `piston-meta.mojang.com`. A first run needs the network.

## 5. Play-test checklist

Do this once with the jar you built. Everything in the README is the intended behaviour until these pass. In the game:

**Start**
- [ ] The game starts. `logs/latest.log` contains `Reaper 2.0.0 is ready (profile "Default", 21 modules)`.
- [ ] Type `/reaper status` in a world. It should say `Reaper 2.0.0 on Minecraft 26.1.x`, and once you have been in the
      world a few seconds, clicked a mouse button and seen a chat message, `Hooks seen working: zoom camera yes, mouse
      yes, chat yes`. A `NOT SEEN` names a mixin that did not apply (the log says which); only the feature behind it
      is lost.
- [ ] Options > Controls has a heading "Reaper Client" with four keys.

**Menu and HUD**
- [ ] Right Shift opens the menu. Drag a window, resize it, collapse it, use the search box, switch a module on and off.
- [ ] Interface settings: change the theme, the accent, the scale (also try a scale other than 1.0 and check that the
      lists inside the windows are cut at the right edge: this is the scissor point in the audit), animations off/on.
- [ ] FPS, Coordinates, Direction and Armor Status show by default. Switch the other HUD elements on one by one.
- [ ] HUD editor (`/reaper hud`): drag elements, close, restart the game, and check that the positions were kept.
- [ ] Give a module a key; test Toggle and Hold.
- [ ] Profiles: create, copy, rename, delete, export, import (paste text) and load. Close the game and start it again:
      the active profile and window positions are as you left them (`config/reaper/` has the files).

**Visual**
- [ ] Custom Crosshair: switch on, try the four styles, check it sits in the middle (it may be 1 px off: see the audit),
      switch off and see the vanilla crosshair again.
- [ ] Zoom: hold Z. The picture zooms, scrolling changes the factor and does not move the hotbar, mouse sensitivity is
      lower while zoomed, and it returns to normal on release.
- [ ] Fullbright (singleplayer): switch on, dark places are bright. Switch off: your previous brightness is back, also
      as shown in Options > Video Settings. `/reaper status` says whether it can go beyond the slider.

**Utility**
- [ ] Waypoints: add one (window, `/reaper waypoint add home`, or the key). Marker, name and distance appear; the
      Nearby Waypoints element lists it; a different world has its own list; dying leaves a death waypoint.
- [ ] Screenshots: take a few with F2, open the manager (button in the menu or `/reaper screenshots`): thumbnails
      appear, preview, rename, copy path, delete, open folder.
- [ ] Inventory Sorter: the Sort button appears in the inventory and in a chest; the result is tidy; Esc stops it.
- [ ] Chat: time stamps in front of messages; write your own name in the chat of a server or LAN game to see the mention
      notification; "Clear chat history" works.

**Server rules**
- [ ] On a server or a LAN game joined from a second client: `/reaper server strict`. Fullbright, Zoom, waypoint markers
      and the Sort button are off. `/reaper server standard` brings them back. Automation stays off in both.
- [ ] Join a server you have set to Strict: a notice appears.

**Baritone (only if you installed the Baritone mod for 26.1)**
- [ ] Switch on Baritone Bridge in a singleplayer world. "Start mining" / `/reaper baritone mine diamond_ore` and
      `/reaper baritone stop` do what Baritone's own `#mine` / `#stop` do. Without Baritone installed, the buttons say
      so and nothing else happens.

**Afterwards**
- [ ] `logs/latest.log` has no `[Reaper]` warnings other than the ones listed in section 4.2 of the audit (names that
      could not be confirmed: clipboard, ping, reduced-debug flag).

If a step fails, send back: the step, the lines of `logs/latest.log` that contain `Reaper` or `Mixin`, the output of
`/reaper status`, and your Minecraft, Fabric Loader and Fabric API versions.

## 6. Mojo Launcher and other launchers

Reaper contains nothing that depends on a launcher. Any launcher that starts a Fabric profile for 26.1.x with Fabric
Loader 0.18.4 or newer and Java 25 is expected to load it. That is an expectation: **it has not been tested with Mojo
Launcher.** If it does not load there, the log from the launcher's own game folder (`logs/latest.log`) will say why.

## What was and was not checked

| Checked here | How |
|---|---|
| Logic: modules, settings, config, profiles, menu and HUD models drawn on a headless canvas, waypoints, sorter, screenshots, chat, commands, server rules, Baritone bridge, and the wiring of all of it | `./gradlew coreTests` equivalent in a plain JDK: 14 suites, 3,947 checks, all pass (`tools/run-core-tests.sh`) |
| Game layer compiles against the *intended* API | `tools/check-adapters.sh`: 107 source files against 115 stand-ins, no errors or warnings. Proves consistency with the stand-ins only |
| Each game call exists in 26.1.x | Looked up in the source of Wurst, Sodium, Mod Menu, malilib, Fabric API, Baritone and Zoomify: 189 of 191 members, the other 2 from a 26.3 project (`docs/API-AUDIT.md`) |
| Gradle glue: resource expansion, jar name, licence file, sources jar, `coreTests` task | Under Gradle 8.14.3 in a probe project without Loom |

| **Not** checked here | Why |
|---|---|
| `./gradlew build` with Loom 1.15.5, Gradle 9.4.0 and Minecraft 26.1.2 | No access to the Fabric and Mojang servers, no JDK 25 |
| Anything that runs inside the game | The game could not be started |
| Mojo Launcher, other mods, servers, real Baritone | Same |
