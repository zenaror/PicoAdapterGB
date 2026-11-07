# PicoAdapterGB — agent guidance

PicoAdapterGB is firmware for Raspberry Pi Pico boards that emulates Nintendo's Mobile Adapter GB. The protocol belongs to the `dependences/libmobile` submodule; this repository handles the board: link cable (PIO and interrupts), networking, flash, LED and web configuration page.

This file summarizes contribution rules. Start with `README.md` and the technical documentation in `doc/`.

## Before starting

1. Check `git status`, the branch, commit and exact `dependences/libmobile` submodule commit. Preserve unrelated local changes.
2. Read `doc/ARCHITECTURE.md`, `doc/BUILDING.md` and `doc/CONFIGURATION.md` as relevant to the task.
3. Identify the board, implementation and pinout involved. Check the affected callers before changing behavior.

## Main rules

- The Mobile Adapter protocol belongs to libmobile; this repository handles the platform. Do not copy libmobile state machines into the firmware.
- `src/core/adapter_bridge.c` is the libmobile boundary. Before changing a callback, read its documentation in `dependences/libmobile/mobile.h`.
- `src/pio/` and the link-cable interrupt path are timing-sensitive: no waiting, networking, DNS, flash, allocation or heavy logging there. Check both 8-bit and 32-bit modes.
- Networking: respect ownership of each lwIP object, never call lwIP from an interrupt and do not block the main loop. Core code (`src/` outside `src/implementations/`) must not depend on cyw43, lwIP or ESP.
- Flash: configuration writes are deferred through the pending-write mechanism. Do not write flash from interrupts, timing-sensitive callbacks or web routes. The authorized exception is `/api/format`, triggered by the web interface's **Format EEPROM** button: this explicit action writes the snapshot directly in both implementations.
- `dependences/libmobile` is a separate repository. Do not modify it for convenience or update it incidentally. Before moving its pointer, confirm that the commit exists in its public origin.
- Investigate before changing anything, make the smallest justified change and do not mix in unrelated cleanup.
- Distinguish code inspection, documentation, build results and hardware observations. A successful build does not prove board behavior.

## Project map

- `README.md` introduces the project; `doc/README.md` indexes architecture, build, configuration, tools and validation. Code stays in `src/`, hardware in `PicoAdapter_PCB/`, and CMake/Makefile/scripts in their existing locations; do not create directories merely to match a template.
- The project repository is `https://github.com/zenaror/PicoAdapterGB`. Libmobile is an independently maintained submodule; keep changes to it separate from firmware maintenance.
- Keep technical contracts needed to build or review the firmware in `doc/`; keep private work records out of the public repository.

## Build

- There are three independent choices: `PICO_BOARD`, `PICOADAPTER_IMPLEMENTATION` and `ADAPTER`. The two implementations are `picow` (boards `pico_w` and `pico2_w`) and `esp` (boards `pico` and `pico2`, with an external ESP8266). Each supports REON and STACKSMASHING pinouts, giving eight targets. See `doc/BUILDING.md` and `doc/ARCHITECTURE.md`.
- Rerun CMake after adding or moving `.c` files: sources are discovered through `GLOB_RECURSE`.
- With `CMAKE_BUILD_TYPE=Debug`, debug output uses UART. Every other build type, including an unspecified type, uses USB.

## Language, Git and privacy

- This repository is public on GitHub. Keep README, documentation, agent guidance, code comments, web pages, serial messages and CMake messages in English. Preserve functional translations, original data and third-party attribution.
- Do not commit or push without the Operator's explicit instruction. When authorized, make one commit per completed work round. History rewriting or force-pushing requires direct confirmation in the current session.
- Do not automatically add Claude/Anthropic `Co-authored-by` trailers. Preserve human authorship, human coauthor credits and legitimate upstream contributions; AI assistance is disclosed in the README.
- Do not commit private work records or local configuration. Never copy passwords, tokens or keys into code, logs or commits.

## Cleanup and retention

- CMake/Ninja and the Makefile generate caches, objects, logs and firmware in build directories. At task completion, checkpoints or pauses, classify only artifacts owned by your task: reproducible outputs may be removed after preserving commands, versions, board, implementation, pinout, hashes and results needed for review. Retain unexplained failures until resolved.
- Preserve originals, saves, `mobile_config.bin`, fixtures, console/hardware evidence, accepted releases and Git recovery bundles/maps in durable storage. `/tmp` is not permanent reference storage. Ignored files are not disposable merely because they are outside Git; establish ownership and provenance before cleanup.
- The current `make clean` removes all of `build/`, not just one variant or the selected `BUILD_DIR`; `make -n clean` shows that scope. Do not run it before classifying and preserving the contents of every affected variant. For an isolated task, prefer explicit task-owned paths and check destinations and symlinks; do not use `git clean -xfd`, broad globs or age-based deletion.
- Do not remove outputs of active processes or other sessions. Retain releases and evidence until an explicit decision; retain Git recovery bundles/maps while needed for old clones. Keep required evidence before cleanup; do not run new tests solely for cleanup.

## Before submitting changes

- Describe the affected board, implementation and pinout, what changed and why, and the checks actually performed.
- Distinguish static inspection, build results and real hardware evidence. State remaining validation needs and preserve unrelated work.
