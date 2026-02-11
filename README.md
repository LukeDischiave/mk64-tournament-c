# Mario Kart 64 Tournament rom

## Current features
- title screen menu for quickly jumping into the game
- improved debug functions and input display
- better crash handler

## Potential Planned features
- implement existing features from abitalive's mk64 tournament rom repo
- tracking metrics per round (number of MTs, average speed, items etc)
- add race finish timer to 4pVS games (pop up in the middle of the player quadrant; timer exists in 2p vs already)
- fix tie logic (change from port priority -> calculated based off timer)
- fix DMA decrypting rom 60 times per second (dump cartridge to expansion pak)
- make game run at 30,60fps (Look at optimizations for mario64; Turn off smoke effects)
- have extra colors/hats for each character (hue shift)
- store player inputs for each race
- store stats for each race (i.e. rng/bananas thrown.) put it as qr code so commentators can use the info
- extra controller port for referee
- lap timer flashes(Use DKR 4p for inspiration)
- Multipause (assembly code already exists)
- Patch shortcuts
- record input data + output via QR Code


## Planned features by Rena Kunisaki
- save settings to memory card/flash cart
- fix timer precision bugs
- skip logo screen
- output time/game state info over USB for tracking (auto splitter, ghost sharing, screenshots, etc)
- load resources over USB for quick testing of custom tracks
- optional tweaks for tournament play (stat rebalancing, etc)
- optional tweaks for performance on emulators (improve FPS/resolution)
- linked multiplayer (USB/local wifi)
- save states and other nice features stolen from OoT GZ ROM
- built-in Gameshark code support
- possibility to patch the game in RAM via Gameshark serial port to use with a real cartridge


## Build instructions
This project has only been developed on Linux. If you can build it on Windows, let me know how and I'll update the instructions.

You will need:
- Python 3 **(Python 3.14.2 tested working 2/10/26)**
- mips64-elf-gcc and related tools **(mips64-elf-gcc 14.2.0 tested working 2/10/26)**
- the Ultra64 SDK header files (refer to ultra64.txt)
- A Mario Kart 64 USA ROM image named `mk64.rom` in Big Endian (ABCD) byte order (the only legal way to get this is to dump it from your own cartridge)

You should be able to run `./buildall.sh` to build the ROM with all patches. It will produce the files:
- patched.rom (the ROM image)
- patched.cod (symbol file that debuggers can read)

The resulting ROM can be played on emulators or flash carts. Development is done with 64drive HW2, but it should work on any cartridge.

Run `clean.sh` to remove all generated files.
Run `run.sh` to upload the ROM to a 64drive cartridge. This file can also be modified to upload to Everdrive64 or launch Mupen64plus.


## Project structure
The project is divided into modules:

### bootstrap
This is the patch that loads other patches from ROM and executes them at boot. Since it sets up the framework for the others, it can't use it itself, so it's just one assembly file.


### hooks
This patch provides functions used by other patches to inject themselves into the game code in a way that allows them to coexist.

### lib
This patch provides utility functions for other patches, such as displaying text on the screen.

### include
This directory isn't a patch, but contains header files defining the game's variables/functions that patches (and hackers) can refer to. It's also where you need to place the ultra64 SDK files.


### crash-handler
This patch replaces the game's crash handler with a more useful one, similar to the one found in Ocarina of Time. It displays without requiring any "cheat code" and shows several pages:

1. GPRs, game state, exception info, hardware info
    - OP: the opcode, if PC is within RAM
    - HE, HS: heap end/start
    - MT, PT, ST: main thread task, prev task, subtask
    - SM: screen mode
    - RT: race type
    - TN: track number
    - NP: number of players
2. FPRs (currently disabled)
3. Stack dump (can scroll with controller 1)
4. RAM dump
5. Frame buffer 1 (last displayed frames before crash)
6. Frame buffer 2


### debug
This patch improves some of the built-in debug functions and adds some more.
- display list moved to corner of screen
- debug mode can be toggled on/off with L+R+Z or with the button on the 64drive cartridge
- input display
- player coords, speed, race progress display
- live RAM viewer (currently disabled)


### new-menu
This patch adds a new title screen menu for quickly jumping to any game state.
- Race Mode: Mario GP, Time Trial, VS, Battle
- GP Cup: Which cup to start on
- GP Round: Which round of the cup to start on (not yet implemented)
- Course: Course to play for non-GP modes (buggy)
- Players: Number of players and screen setup
- Player 1/2/3/4: Player character selection
- Class: 50cc/100cc/150cc/Extra (separate from Mirror Mode)
- Mirror Mode: on/off
- Items: on/off (not yet implemented)
- Music: on/off (not yet implemented)
- Debug Mode: on/off

This menu allows choosing some settings that aren't possible normally, such as vertical splitscreen, 50cc mirror mode, multiple players with the same character, etc.
