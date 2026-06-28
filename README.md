# Mega Man 11 Weapon Randomizer

A Cheat Engine table that randomly cycles your equipped weapon and gear state every few seconds — designed for challenge runs and streaming.

## Requirements

- [Cheat Engine 6.4+](https://www.cheatengine.org/)
  - sorry i didn't check the dependency - actually i only check with the newest version CE 7.7. Please tell me if you meet any problem in other version.
- Mega Man 11 (PC, Steam)

## Setup

1. Launch Mega Man 11.
2. Open Cheat Engine and load `RM11_WeaponRandomizer.CT`.
3. Cheat Engine will auto-attach to `game.exe`. If it doesn't, attach manually via the process list.

## Usage

| Action | Hotkey |
|--------|--------|
| Toggle randomizer on/off | `Ctrl + A` |

- **On:** The randomizer immediately picks a weapon and gear state, then keeps switching every 5–10 seconds.
- **Off:** All locks are released and the randomizer stops.

The "Never Use Ammo" cheat is bundled into the same toggle — weapons won't deplete while the randomizer is active.

### What gets randomized

| Roll | Chance | Effect |
|------|--------|--------|
| Normal | 90% | Random weapon, gear gauge locked to overheat (can't activate gear) |
| Power Gear | 10% | Random weapon with Power Gear active and charge locked full |

All 9 weapons (including Mega Buster) are eligible. The AutoCharge chip is automatically enabled when Power Gear is active.

## Streaming Overlay

The `overlay/` folder contains a browser-based bar you can add to OBS or any streaming software.

### OBS Setup

1. Add a **Browser Source** to your scene.
2. Check **Local file** and browse to `overlay/overlay.html`.
3. Set **Width** to however wide you want the bar (e.g. 600).
4. Set **Height** to **72** (the bar has no padding — position it in OBS as needed).
5. Check **Transparent background**.

The overlay reads the randomizer state automatically while the CT is active. When the randomizer is off it shows a dim placeholder.

### Language

Weapon names are auto-detected from your browser language. You can override with a URL parameter:

| Language | URL |
|----------|-----|
| English (default) | `overlay.html` |
| Japanese | `overlay.html?lang=ja` |
| Traditional Chinese | `overlay.html?lang=zh` |

To add a new language, copy `overlay/weapons.js`, rename it `overlay/weapons.XX.js` (where `XX` is the BCP 47 language tag), and translate the names in order.

## File Overview

```
RM11_WeaponRandomizer.CT  — Cheat Engine table (load this)
overlay/
  overlay.html            — OBS browser source overlay
  weapons.js              — English weapon names
  weapons.ja.js           — Japanese weapon names
  weapons.zh.js           — Traditional Chinese weapon names
  rm11_stream.js          — Written by the CT at runtime; do not edit
  0.webp – 8.webp         — Weapon icons (index matches weapon slot)
```
