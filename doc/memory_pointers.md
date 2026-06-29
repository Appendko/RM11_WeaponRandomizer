# Memory Pointer Reference

All runtime addresses change each session. Always dereference from the static game.exe offsets below.

---

## Player Base

**Pointer:** `[[game.exe + 0xC3EFA8] + 0x1D0]` = `q`

| Field | Address | Type | Notes |
|-------|---------|------|-------|
| Weapon slot | `q + 0x341C` | 4 Bytes | 0–8 |
| Charge gauge | `q + 0x35A8` | Float | continuous charge value |
| Charge level | `q + 0x35AC` | Byte | 0=bean, 1=small, 2=full, 3=over |

---

## Gear Base

**Pointer:** `[game.exe + 0xB87A58]` = `g`

| Field | Address | Type | Notes |
|-------|---------|------|-------|
| Gear gauge | `g + 0x4A4` | Float | 0.0=empty, 300.0=overheat |
| Gear type | `g + 0x4B8` | Byte | 0=Speed, 1=Power, 2=Double, 3=Normal |
| Cooldown gauge | `g + 0x4C0` | Float | |

---

## Save/Inventory Base

**Pointer:** `[game.exe + 0xC3F710]` = `p`

### Weapon Unlock

| Field | Address | Type | Notes |
|-------|---------|------|-------|
| Weapon unlock flags | `p + 0x3C70` | 2 Bytes | bitmask; 2047 = all 9 weapons |

### Equipment Chip Flags

Array base: `p + 0x3B10` (1 byte per chip, contiguous)

| Chip | Address | Notes |
|------|---------|-------|
| AutoCharge (index 1) | `p + 0x3B11` | 1=enabled, 0=disabled |
| EnergyDispenser (index 14) | `p + 0x3B1E` | 1=enabled, 0=disabled |

### Shop Item Bought Counts

See [shop_inventory.md](shop_inventory.md) for the full table. Key entries:

| Item | Address | Notes |
|------|---------|-------|
| EnergyBalancer | `p + 0x3A60` | gate condition for NeoEnergyBalancer unlock |
| AutoCharge | `p + 0x3A78` | also used to mark chip as purchased |
| NeoEnergyBalancer | `p + 0x3AC8` | |

Formula: `p + 0x3A40 + item_index * 8` (stride 8, byte at base of each slot).

### Shop Item Slot Flags

`p + 0x3A44 + item_index * 8` — written to `1` by the game at `game.exe+23549F` during shop init for each populated slot. This is a separate field from the bought count (which sits at offset +0 of each slot, not +4). It does **not** control shop visibility or purchase availability.

---

## Code Patches

### Always Saturday (shop discount)

- **AOB:** `48 C1 FA 17 48 8B C2 48 C1 E8 3F 48 03 D0` (CT uses AOB scan; static location is `game.exe + 0xFB806`)
- **Patch:** `48 33 D2 BA A9 1F 1E 6B 90 90 90 90 90 90`
- Replaces day-of-week computation with a constant that always evaluates to Saturday.

### Never Use Ammo

- **AOB:** `85 C9 0F 4F C1 89 84 B7 78 04 00 00 48 8B 74 24 38`
- **Patch offset:** `+5`, 7 bytes → `90 90 90 90 90 90 90`

### Shop Unlock — Force All Items Available

Patches all 11 conditional `set` instructions to `mov al, 1` + NOP (`B0 01 90`), making every locked shop item always appear as available.

| Address | Item | Original |
|---------|------|---------|
| `game.exe + 0x235374` | Pierce Protector | `0F 93 C0` (setae) |
| `game.exe + 0x235380` | Bolt Catcher | `0F 95 C0` (setne) |
| `game.exe + 0x235399` | Energy Catcher | `0F 93 C0` (setae) |
| `game.exe + 0x2353B2` | Capsule Catcher | `0F 93 C0` (setae) |
| `game.exe + 0x2353CB` | Buster Plus Chip | `0F 93 C0` (setae) |
| `game.exe + 0x2353FC` | Energy Balancer Neo | `0F 95 C0` (setne) |
| `game.exe + 0x235423` | Cooling System | `0F 93 C0` (setae) |
| `game.exe + 0x23543C` | Awakener Chip | `0F 93 C0` (setae) |
| `game.exe + 0x235455` | Speed Gear Booster | `0F 93 C0` (setae) |
| `game.exe + 0x23546E` | Power Shield | `0F 93 C0` (setae) |
| `game.exe + 0x235487` | Cooling System ∞ | `0F 93 C0` (setae) |

---

## Shop Unlock Analysis

The shop init function spans roughly `game.exe + 0x2352C0` – `0x2354F3`.

| Address | Purpose |
|---------|---------|
| `game.exe + 0x235320` | Init loop start: sets all 26 item records byte 0 = 1 |
| `game.exe + 0x235374` | Condition block start: 11 `set` instructions overwrite specific item records |
| `game.exe + 0x2353E8` | Function call gating the NeoEnergyBalancer check (returns 0 → always unlock) |
| `game.exe + 0x2353F5` | NeoEnergyBalancer condition block: reads EnergyBalancer bought count at `[r14+0x3A60]` |
| `game.exe + 0x235490` | Loop top: `cmp [rdx], 00` — skips empty item slots |
| `game.exe + 0x23549F` | `mov [rax+rcx*8+3A44], 01` — writes slot flag (separate from bought count) |
| `game.exe + 0x2354C0` | Writes purchase status into item record byte 1 |
| `game.exe + 0x2354CE` | Loop increment: `inc ebx / add rdx,3 / cmp ebx,1A / jb 235490` |

Item records table: `[rdi + 0x4CF8] + item_index * 3`
- Byte 0: unlock slot (1 = visible in shop, 0 = hidden) — set to 1 by init loop, overwritten by condition checks
- Byte 1: purchase status flag — written by loop body at `game.exe+2354C0`

The condition checks at `235374`–`235487` read progress counters at `p + 0x3B34 / 0x3B38 / 0x3B3C / 0x3B40` and compare them against thresholds from item config data. Patching each `set` to `mov al, 1` bypasses the check unconditionally.
