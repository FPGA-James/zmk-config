# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ZMK firmware configuration for a **Lily58** split keyboard using **Nice!Nano v2** controllers. ZMK is a Zephyr-based open-source keyboard firmware. There is no local build — firmware is compiled via GitHub Actions CI.

## Building Firmware

Push to `main` (or open a PR) to trigger the GitHub Actions workflow ([.github/workflows/build.yml](.github/workflows/build.yml)), or trigger it manually via `workflow_dispatch` on GitHub. It uses the `zmkfirmware/zmk-build-arm:stable` Docker image and builds both halves in parallel via a matrix:
- `nice_nano/nrf52840` + `lily58_left`
- `nice_nano/nrf52840` + `lily58_right`

Artifacts are uploaded as `lily58_left-nice_nano_nrf52840.uf2` and `lily58_right-nice_nano_nrf52840.uf2`. Flash by putting a Nice!Nano into bootloader mode (double-tap reset) and copying the `.uf2` onto the drive.

Note: the board name uses the new Zephyr hardware model format (`nice_nano/nrf52840`). The shared ZMK workflow (`build-user-config.yml`) does not support this format — the custom workflow uses `tr '/' '_'` to sanitize slashes in artifact filenames.

## Configuration Files

- [config/lily58.keymap](config/lily58.keymap) — keymap in ZMK's Devicetree syntax (`.dtsi`-style). Defines all layers and key bindings.
- [config/lily58.conf](config/lily58.conf) — Kconfig options. Enables OLED display (`CONFIG_ZMK_DISPLAY=y`) and sets right half as master via preprocessor directives (`#undef MASTER_LEFT` / `#define MASTER_RIGHT`). Encoder HW is disabled (commented out) but `sensor-bindings` are defined in every layer.
- [config/west.yml](config/west.yml) — West manifest tracking ZMK `main` branch (not pinned to a revision; upstream changes can break builds).
- [build.yaml](build.yaml) — GitHub Actions matrix config (boards/shields to build).

## Keymap Architecture

The keymap uses a **Miryoku-inspired** 7-layer structure activated via thumb cluster layer-taps (`lt`):

| Thumb key (tap / hold) | Layer | Purpose |
|---|---|---|
| SPACE / hold | Layer 1 | Navigation (arrows, undo/cut/copy/paste/redo) |
| TAB / hold | Layer 2 | Extended nav (arrows + home/end/pgup/pgdn, caps, ins) |
| LGUI / hold | Layer 3 | Media controls (vol, prev/next, play/pause/stop/mute) |
| RET / hold | Layer 4 | Symbols (`{ } & * ( ) : $ % ^ + ~ ! @ £ \|`) |
| BSPC / hold | Layer 5 | Numbers + numpad layout (`[ ] 7 8 9 ; 4 5 6 = \` 1 2 3 \`) |
| DEL / hold | Layer 6 | Function keys (F1–F12, PRTSCN, SCRLK, PAUSE) |

Layer `0` in the file appears to be a legacy/unused symbols layer (not referenced in the thumb cluster).

## ZMK Binding Syntax

- `&kp KEY` — key press
- `&lt LAYER KEY` — layer-tap (tap=key, hold=activate layer)
- `&trans` — transparent (pass through to layer below)
- `&bt BT_SEL N` — Bluetooth profile selection
- `&inc_dec_kp A B` — encoder: clockwise=A, counter-clockwise=B
