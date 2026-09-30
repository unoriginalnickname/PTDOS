# PTDOS

DOS games inside Cisco Packet Tracer 9.0.1, as an app on a simulated PC's Desktop: v86 emulates an x86 PC and boots FreeDOS. Split off from [PTDoom](https://github.com/unoriginalnickname/PTDoom) on 2026-09-30 (the history up to then is shared).

It works because Packet Tracer's Desktop apps and extension windows are web pages in a full Chromium (QtWebEngine 6.8.7, Chrome 130) with WebAssembly.

## Play

A DOS PC inside the Packet Tracer PC: v86 emulates an x86 PC with a Sound Blaster 16 and boots FreeDOS from a floppy. The games you add go on drive C:. Tested with shareware DOOM and Wolfenstein 3D 1.4, both with sound and FM music.

**No games are included.** Only use games you may copy: shareware, freeware, or your own copy of a commercial game. Each one is a `.zip` of the game's files, as it would sit on a hard disk (not an installer).

1. Download [PTDOS-installer.pkt](https://github.com/unoriginalnickname/PTDOS/raw/master/PTDOS-installer.pkt) and open it in Packet Tracer.
2. Allow the file's script. The **DOS games** window opens.
3. Under **Add a game**, choose a `.zip`. Check the **Name** (up to 8 letters: what you type to start it) and the **Program** it guessed, then **Install**. Repeat for more games.
4. Save the file under a new name (**File → Save As**).
5. Click **DOS-PC → Desktop → DOS**. Press a key once to turn the sound on. FreeDOS lists the games; type a game's name at `C:\>`.

Typing `dos` in DOS-PC's Command Prompt reopens the games window. A DOS window that is already open only sees newly added games after you close and reopen it.

A game's CD music can come in its zip as `CDAUDIO\TRACK02.OGG`, `TRACK03.OGG` and on (MP3, WAV and FLAC too), one file per audio track of the CD: the DOS app puts them on an emulated CD as audio tracks, and games that play CD music (through MSCDEX) play them.

The game fills the largest 4:3 area of the PC's window, so resize the window to make it bigger. A mouse driver (CuteMouse) is loaded for every game. Over the game only the game's own cursor shows; if it drifts away from where your hand is, push the mouse against a window edge to line them up again (Packet Tracer's window doesn't let the page capture the mouse).

### Dungeon Keeper

Runs from your own copy of the DOS version (tested with EA's Dungeon Keeper Gold, which runs it in DOSBox), without its CD:

1. `python tools/make_keeper_zip.py "<folder with KEEPER.EXE>"`, for example `"D:/EAGames/Dungeon Keeper/DATA"`. It writes `dosgames/keeper/keeper.zip` (about 140 MB): the game, with the CD's music (`keeper02.ogg` to `keeper07.ogg`) as `CDAUDIO` tracks, without DOSBox or the CD image, and with the game's `KEEPER.CFG` pointed at `C:\KEEPER`.
2. Add that zip in the games window with Name **KEEPER** and Program **KEEPER** (both are guessed from the zip's name).
3. At `C:\>` type `keeper`. Deeper Dungeons is `deeper` in the same folder (`cd keeper`).

The music plays from the emulated CD. The game's music volume slider doesn't change it.

## How it works

A Desktop app is a project folder on the PC with an `app_manifest.xml`, a `main.py`, and page files prefixed `[gui]`. The DOS app works the same way: v86 in `[gui]libv86.js`, the BIOS, FreeDOS floppy and WebAssembly in `[gui]v86data.js`, and each game's zip in a `[gui]g<n>.js`. The page (`web/v86-dos.js`) unzips the games and builds drive C: as a FAT16 disk in memory before booting. At boot `C:\PTDOS.BAT` loads CuteMouse, and for a game with CD music also a CD-ROM driver (UDVD2) and CD extensions (SHSUCDX): the CD is an empty data track plus the music as audio tracks. v86's CD drive only reads data, so the page answers the drive's audio commands itself (table of contents, play, pause, stop, position) and plays each track through an `<audio>` element; UDVD2 got the Audio Channel Info call that the Miles Sound System's CD player asks for (`src/udvd2/`). v86 has no FM synthesis, so the page catches the AdLib ports and plays them through Nuked OPL3 compiled to its own small WebAssembly module (`src/v86opl/`). It also times the VGA retrace like a real card, without which Wolfenstein 3D hangs on a black screen. The installer's module is `pt/dos-installer/`.

## Build

Needs Python and the [Emscripten SDK](https://emscripten.org/docs/getting_started/downloads.html) (for `build_opl.py`).

The DOS app needs FreeDOS 1.3's `ctmouse`, `shsucdx` and `devload` packages unpacked into `dosgames/cdrom/` (from https://www.ibiblio.org/pub/micro/pc-stuff/freedos/files/repositories/1.3/, `base/` and `drivers/`), v86 in `vendor/v86/` (`libv86.js`, `v86.wasm` and `LICENSE` from the npm package `v86`, `seabios.bin` and `vgabios.bin` from its repo's `bios/`) and FreeDOS's boot floppy at `dosgames/freedos/freedos722.img` (from `https://i.copy.sh/freedos722.img`). Then:

```sh
python tools/build_opl.py         # Nuked OPL3 as vendor/opl/opl.wasm, with emcc
python tools/build_udvd2.py       # the CD-ROM driver as vendor/udvd2/UDVD2.SYS, with JWasm 2.20 in vendor/jwasm/
python tools/make_v86_app.py      # dist/v86app/ with shareware DOOM from dosgames/doom/
python tools/make_v86_app.py --no-game   # no games, for the installer
```

`dist/install-v86-app.js` installs it on every PC, like `install-app.js`. The DOS installer is remade like DOOM's (below), with the app installed from a `--no-game` build and `pt/dos-installer/ptdos.js` and `installer.html` in the module.

To remake the installer: install the app without `[gui]wad.js` and with `dist/app/icon-generic.png` as the icon, then in **Extensions → Scripting → Edit File Script Module** import `pt/installer/ptdoom.js` (Script Engine) and `pt/installer/installer.html` and `play.html` (Custom Interfaces), and under General → Security tick Get Network Info, Change Network Info, Change User Interface, Miscellaneous UI, and Allow opening file even if security privileges are not granted. Save with Packet Tracer's own **File → Save**.

## Known issues

- Audio can glitch while loading.
- Opening an app window can disconnect packet-tracer-mcp's bridge; reopen **Extensions → MCP Bridge**.

## Licences

- `PTDOS-installer.pkt` also ships:
  - [v86](https://github.com/copy/v86) 0.5.462: BSD-2-Clause, Copyright (c) 2012, The v86 contributors. Full notice in [`licenses/v86.txt`](licenses/v86.txt).
  - [SeaBIOS](https://www.seabios.org/) and its VGA BIOS, as distributed with v86: LGPL-3.0. Source: https://github.com/coreboot/seabios.
  - FreeDOS, from the boot floppy image `freedos722.img` (v86's copy, trimmed): the FreeDOS kernel, the FreeCOM shell (`COMMAND.COM`), ATTRIB, EDIT, HIMEM and XCOPY, all GPL-2.0. Source: [kernel](https://github.com/FDOS/kernel), [FreeCOM](https://github.com/FDOS/freecom), and the other programs' packages at [ibiblio's FreeDOS archive](https://www.ibiblio.org/pub/micro/pc-stuff/freedos/files/). FreeDOS is a trademark of Jim Hall.
  - Nuked OPL3 (`src/music/opl3.c`, built by `src/v86opl/`): GPL-2.0 or later.
  - From FreeDOS 1.3's packages, loaded from drive C: at boot: CuteMouse 2.1b4 (`CTMOUSE.EXE`, GPL-2.0), DEVLOAD 3.25a (GPL-2.0), UDVD2 (Jack R. Ellis, "free with sources"; changed here, source in `src/udvd2/`) and SHSUCDX 3.09 (John H. McCoy and Jason Hood, freeware). Sources: each package's `SOURCE` folder, and [ctmouse](https://gitlab.com/FreeDOS/base/ctmouse), [devload](https://gitlab.com/FreeDOS/base/devload), [udvd2](https://gitlab.com/FreeDOS/drivers/udvd2), [shsucdx](https://gitlab.com/FreeDOS/base/shsucdx).
  - v86's `libv86.js` is patched at build time (`ATAPI_PATCHES` in `tools/make_v86_app.py`) so the page can answer CD commands first.
- DOS games: none are included. Each belongs to its publisher.
