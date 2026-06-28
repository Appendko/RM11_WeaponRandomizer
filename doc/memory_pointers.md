# Memory Pointer Reference

All runtime addresses change each session. Always dereference from the static game.exe offsets below.
#### NOT SURE IF THIS IS CORRECT, just a memo and need to check later
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

### Shop Item Unlock Flags
#### NOT SURE IF THIS IS CORRECT
`p + 0x3A44 + item_index * 8` (byte, stride 8) — written to `1` by the game during shop init for each populated slot. Does **not** directly control purchase availability.

---

## Code Patches

### Always Saturday (shop discount)
#### From autosplit, NOT SURE IF THIS IS CORRECT
- **Address:** `game.exe + 0xFB806`
- **Original:** `48 C1 FA 17 48 8B C2 48 C1 E8 3F 48 03 D0`
- **Patch:** `48 33 D2 BA A9 1F 1E 6B 90 90 90 90 90 90`
- Replaces day-of-week computation with a constant that always evaluates to Saturday.



### Never Use Ammo
- **AOB:** `85 C9 0F 4F C1 89 84 B7 78 04 00 00 48 8B 74 24 38`
- **Patch offset:** `+5`, 7 bytes → `90 90 90 90 90 90 90`

---

## Shop Unlock Analysis
#### NOT SURE IF THIS IS CORRECT
| Address | Purpose |
|---------|---------|
| `game.exe + 0x235490` | Loop top: `cmp [rdx], 00` / skip empty slots |
| `game.exe + 0x23549F` | `mov [rax+rcx*8+3A44], 01` — sets item unlock flag |
| `game.exe + 0x2353F5` | Pre-loop: computes icon display flags (not purchase gates) |
| `[rdi + 0x4CF8 + i*3]` | Item record byte 0 — 1=available, 0=hidden; set by init then overwritten by conditions |
| `game.exe + 0x2354CE` | Loop increment: `inc ebx / add rdx,3 / cmp ebx,1A / jb 235490` |

The preloop check at `2353F5` reads `[r14 + 0x3A60]` (EnergyBalancer bought count) as the prerequisite gate for NeoEnergyBalancer. The other conditions compare progress counters at `p + 0x3B34 / 0x3B38 / 0x3B3C / 0x3B40` against thresholds from item config data.
