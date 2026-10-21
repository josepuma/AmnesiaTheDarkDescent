Amnesia: The Dark Descent — macOS (Apple Silicon) port
=======================

This fork builds the official [Amnesia: The Dark Descent source code](https://github.com/FrictionalGames/AmnesiaTheDarkDescent) on current macOS, so you can play the game on an Apple Silicon Mac with the assets from your own copy of the game.

The Mac executables that ship on Steam are 32-bit (i386) and cannot run on macOS 10.15 or later. This build produces a 64-bit `Amnesia.app` (x86_64) that runs through Rosetta 2.

> **You need to own the game.** This repository contains only the GPLv3 engine and game code. Maps, models, textures, sounds and scripts are not included and are not free software: they come from your own copy (Steam, GOG, etc.).

> Community project, not affiliated with or endorsed by Frictional Games.

## Quick path

Tested on a MacBook with an Apple M3, macOS 27, Xcode with the macOS 27 SDK and the Steam version of the game.

1. Install the tools:

   ```bash
   xcode-select --install          # if you don't have Xcode or its Command Line Tools
   softwareupdate --install-rosetta --agree-to-license
   brew install cmake ninja
   ```

2. Extract the bundled dependencies (they must keep their symlinks, so use `tar`, not `unzip`):

   ```bash
   mkdir -p HPL2/dependencies
   tar -xf HPL2/dependencies.zip -C HPL2/dependencies
   # the zip stores this symlink as a plain text file, recreate it
   ln -sf libIL.1.dylib HPL2/dependencies/lib/macosx64/libIL.dylib
   ```

3. Build:

   ```bash
   cmake -S amnesia/src -B build -G Ninja \
     -DCMAKE_BUILD_TYPE=RelWithDebInfo \
     -DCMAKE_OSX_ARCHITECTURES=x86_64 \
     -DCMAKE_POLICY_VERSION_MINIMUM=3.5
   cmake --build build --target Amnesia
   ```

   The result is `build/Amnesia.app`.

4. Make a working copy of the game data. An APFS clone (`cp -c`) is instant and takes no extra disk space, and your Steam install stays untouched:

   ```bash
   cp -cR ~/Library/Application\ Support/Steam/steamapps/common/Amnesia\ The\ Dark\ Descent ~/Games/AmnesiaTDD
   ```

5. Put the new app next to `resources.cfg`, replacing the 32-bit one, and run it:

   ```bash
   cd ~/Games/AmnesiaTDD
   mv Amnesia.app Amnesia_Steam_i386.app
   ditto /path/to/this/repo/build/Amnesia.app Amnesia.app
   open Amnesia.app
   ```

   The game looks for `resources.cfg` in the folder that contains `Amnesia.app` (`StartFolder=PARENT` in `Info.plist`).

After every rebuild, copy the app again with `ditto build/Amnesia.app ~/Games/AmnesiaTDD/Amnesia.app`.

## Troubleshooting

| Symptom | What to check |
|---------|---------------|
| The game closes on start | Read `~/Library/Application Support/Frictional Games/Amnesia/Main/hpl.log`. |
| Messages or crash output | Start it from a terminal: `~/Games/AmnesiaTDD/Amnesia.app/Contents/MacOS/Amnesia` |
| Odd player state in an old save | Saves made with a broken build may keep bad values (move speed, camera roll). Start a new game. |
| CMake errors mentioning `dependencies` or `OALWrapper` | `HPL2/dependencies` was not extracted (step 2). |

## Build options

| CMake option | Default | Purpose |
|--------------|---------|---------|
| `HPL2_WITH_FBX` | `OFF` | FBX mesh loader. Needs the FBX SDK 2012, which is no longer available. The game does not use FBX. |
| `HPL2_BUILD_TOOLS` | `OFF` | Level, model, particle and material editors, and viewers. Not tested on macOS. |
| `AMNESIA_BUILD_LAUNCHER` | `OFF` | Graphics quality launcher. Upstream lacks `Launcher_Prefix.pch`. Use the in-game options instead. |
| `IS_APPLE_64` | `ON` | 64-bit build. `OFF` means i386, which current macOS cannot build or run. |

## Known limitations

- **Not native arm64.** The prebuilt libraries in `HPL2/dependencies.zip` are x86_64 only. A native build needs Newton Dynamics 2.x and a newer AngelScript built for arm64, plus adapting the engine to the newer AngelScript API.
- **Depends on Rosetta 2.** Apple has announced that macOS 27 is the last release with full Rosetta 2 support, so a native build will eventually be required.
- **OpenGL 2.1.** The renderer uses Apple's legacy OpenGL 2.1 context, which runs on top of Metal. It works on M3, but Apple has deprecated it.

## Changes from upstream

Required by GPLv3 section 5(a). See the commit history of the `macos-port` branch for full detail.

| Area | Change |
|------|--------|
| C++ / libc++ | Removed `std::binary_function`. Made comparison operators `const`. Compare a pointer with `NULL` instead of `false`. Stopped taking the address of temporaries in Newton calls. |
| CMake | x86_64 by default, deployment target 11.0, FBX/tools/launcher are optional, modern CMake fixes. |
| Graphics | The fullscreen desktop window takes the desktop size instead of a fixed 800x600. |
| Scripting | Script functions that return `float` are registered with AngelScript's generic calling convention. The prebuilt AngelScript 2.19.2 for x86_64 returned 0 for them, which broke walking, sanity, lamp oil and more. |

## License

All code is under the GNU General Public License version 3. Read the [LICENSE](LICENSE) file for terms of use. Copyright notices from Frictional Games are kept in every source file.

`HPL2/dependencies.zip` is redistributed as published upstream. It contains third-party libraries under their own licenses (SDL, OpenAL, DevIL, AngelScript, Newton Dynamics, Ogg/Vorbis/Theora, GLEW) and headers from NVIDIA Cg and FMOD.

---

Original upstream README
-----------------

Currently the engine uses fbx sdk 2012 which isn't avalable anymore which means the engine wont compile. If you want to give a shot anyway you can find the sdk here:
https://www.autodesk.com/fbx

Other than that, here is almost everything you need to build Amnesia: The Dark Descent. Included are project files for Visual Studio 2010 and CMake for Linux & macOS.

### Contributing Code

We encourage everyone to contribute code to this project, so just sign up for a github account, create a fork and hack away at the codebase.

### License Information

All code is under the GPL Version 3 license. Read the LICENSE file for terms of use of the license.
