# Shop Inventory Structure

Base pointer: `[game.exe + 0xC3F710]` = p (changes each session)

Each item occupies 8 bytes. The bought count (_num) field is at `p + 0x3880 + item_index * 8 + 0x1C0`.

## Item Table

| Index | Item | Offset (flat) | CE pointer offset |
|------:|------|--------------|-------------------|
| 0  | 1up                  | `p + 0x3A40` | `3880+1C0` |
| 1  | E_num                | `p + 0x3A48` | `3880+1C8` |
| 2  | M_num                | `p + 0x3A50` | `3880+1D0` |
| 3  | W_num                | `p + 0x3A58` | `3880+1D8` |
| 4  | EnergyBalancer       | `p + 0x3A60` | `3880+1E0` |
| 5  | SpikeProtector       | `p + 0x3A68` | `3880+1E8` |
| 6  | 1/2                  | `p + 0x3A70` | `3880+1F0` |
| 7  | AutoCharge           | `p + 0x3A78` | `3880+1F8` |
| 8  | Eddie                | `p + 0x3A80` | `3880+200` |
| 9  | Beat                 | `p + 0x3A88` | `3880+208` |
| 10 | SubTankBag           | `p + 0x3A90` | `3880+210` |
| 11 | FriendCall_upgrade   | `p + 0x3A98` | `3880+218` |
| 12 | BoltCatcher          | `p + 0x3AA0` | `3880+220` |
| 13 | EnergyCatcher        | `p + 0x3AA8` | `3880+228` |
| 14 | CapsuleCatcher       | `p + 0x3AB0` | `3880+230` |
| 15 | ShockAbsorber        | `p + 0x3AB8` | `3880+238` |
| 16 | AdvancedBuster       | `p + 0x3AC0` | `3880+240` |
| 17 | NeoEnergyBalancer    | `p + 0x3AC8` | `3880+248` |
| 18 | CoolingSystem        | `p + 0x3AD0` | `3880+250` |
| 19 | FreeWeapon           | `p + 0x3AD8` | `3880+258` |
| 20 | SpeedgearAccelerator | `p + 0x3AE0` | `3880+260` |
| 21 | EnergyBarrier        | `p + 0x3AE8` | `3880+268` |
| 22 | EnergyDispenser      | `p + 0x3AF0` | `3880+270` |
| 23 | FreeGearGauge        | `p + 0x3AF8` | `3880+278` |
| 24 | SpikeBoots           | `p + 0x3B00` | `3880+280` |
| 25 | MysteriousChip       | `p + 0x3B08` | `3880+288` |

## Notes

- The loop at `game.exe+235490` iterates all 26 items (`cmp ebx, 1A`).
- Unlock flag per item: `p + item_index * 8 + 0x3A44` — written to 1 by the game when the item is registered, but does **not** directly control shop availability.
- The preloop check at `game.exe+2353F5` (`cmp [r14+0x3A60], ebx`) reads the **EnergyBalancer bought count** (index 4) — this is the prerequisite gate for NeoEnergyBalancer.
- Icon visibility flags (not purchase gates): `[rdi+0x4D2B/2E/31/34/37/3D]` — set by the preloop condition checks.

## Related Offsets

| Field | Offset | Notes |
|-------|--------|-------|
| Weapon unlock flags | `p + 0x3C70` | `writeSmallInteger(..., 2047)` unlocks all 9 weapons |
| AutoCharge chip flag | `p + 0x3B11` | 1 = enabled, 0 = disabled |
| AutoCharge bought count | `p + 0x3A78` | same as item index 7 above |
