# ZAMN Recomp — Zombies Ate My Neighbors (SNES) → Android

Static recompilation of the 65816 code of the USA ROM into C, running on a hardware
runtime (PPU/APU/DMA from LakeSnes, MIT) with an SDL2 front end (touch controls + gamepads).

**The ROM is NOT included.** The app asks for your own dump once and keeps it in its private storage.
`gen/recomp_blocks.c` is a translation of the game's code: keep this repository **private**, don't publish it.

## How it works
- `tools/recomp.py` traces the ROM from the interrupt vectors, tracks the M/X/E flags and emits one C
  function per basic block + a dispatch switch (`gen/recomp_blocks.c`, ~8000 blocks / ~11.7k instructions).
- Each instruction becomes `cpu_fetchOpcode(); cpu_op_XX();` — the exact per-opcode micro-ops of the
  runtime CPU (same bus accesses and cycle counts), with opcode decode removed.
- Anything not translated (RAM code, unseen jump-table targets) falls back to the exact interpreter and is
  logged; `tools/iterate.sh` folds those misses back into the next generation (coverage-guided).
- Verified: recompiled build vs. interpreter is identical (per-instruction state hash chain, WRAM, framebuffer)
  over 7000 frames on 2 input scripts, with 0 interpreter fallbacks.

## Build the APK
**A) GitHub Actions (no setup):** push this folder to a *private* GitHub repo → Actions → "Build Android APK" → download `zamn-recomp-apk`.
**B) Android Studio:** `./android/setup.sh`, then open `android/` and Run (needs NDK + CMake 3.22 from the SDK Manager).
Install the APK, open it, pick your `.sfc` (1 MB, LoROM; a 512-byte copier header is stripped automatically).

## Desktop test build
`tools/build.sh` + SDL2: `gcc -O2 -w -DRECOMP -Iruntime -I<SDL>/include host/main_sdl.c host/recomp_support.c gen/recomp_blocks.c runtime/{spc,dsp,apu,cpu,dma,ppu,cart,input,statehandler,snes,snes_other}.c -lSDL2 -lm`
then `./a.out rom.sfc`  (keys: arrows, Z/X/A/S, Enter, RShift, D/C).

## Regenerate / extend coverage
`tools/iterate.sh ROM.sfc 7000 0 1 2 3` (frames, then input seeds). More seeds / longer runs = more blocks recompiled.

## Credits
LakeSnes by elzo-d (MIT) — `runtime/LICENSE-LakeSnes.txt`. SDL2 (zlib).
