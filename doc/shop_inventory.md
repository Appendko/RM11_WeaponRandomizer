# Shop Inventory Structure

Base pointer: `[game.exe + 0xC3F710]` = p (changes each session)

Each item occupies 8 bytes in the bought count array.
- **Bought count:** `p + 0x3A40 + item_index * 8` (byte at offset +0 of each slot)
- **Item record:** `[rdi + 0x4CF8] + item_index * 3` (byte 0 = visible in shop)
- **Equip flag:** `p + 0x3B10 + chip_index` (1 = active, 0 = inactive; all equippable items)

CT entry ID=41 when enabled:
- AA script patches all 11 conditional `set` → `mov al, 1` + NOP (forces shop visibility)
- Lua timer (1 s) writes 1 to bought count for 18 non-consumable items if currently 0

---

## Full Item Table

| Idx | Item (EN / JP) | Price | Unlock | Patch addr | Bought count | Equip flag |
|-----|---------------|-------|--------|------------|--------------|------------|
| 0  | 1-UP / 1UP | 50 / 40 | default | — | `p+0x3A40` | consumable |
| 1  | Energy Tank / E缶 | 100 / 80 | default | — | `p+0x3A48` | consumable |
| 2  | Mystery Tank / M缶 | 300 / 240 | default | — | `p+0x3A50` | consumable |
| 3  | Weapon Tank / W缶 | 100 / 80 | default | — | `p+0x3A58` | consumable |
| 4  | Energy Balancer / エネルギーバランサー | 100 | default | — | `p+0x3A60` | `p+0x3B10` (chip 0) |
| 5  | Pierce Protector / ショックガード | 50 | Die on spikes 5× | `235374` | `p+0x3A68` | `p+0x3B12` ? (chip 2) |
| 6  | Super Guard / ガードアップ | 100 | default | — | `p+0x3A70` | consumable |
| 7  | Auto-Charge Chip / オートチャージチップ | 300 | default | — | `p+0x3A78` | `p+0x3B11` (chip 1) |
| 8  | Eddie Call / エディーコール | 20 / 16 | default | — | `p+0x3A80` | consumable |
| 9  | Beat Call / ビートコール | 50 / 40 | default | — | `p+0x3A88` | consumable |
| 10 | Tank Container / 缶袋 | 200 | default | — | `p+0x3A90` | passive |
| 11 | Buddy Call Plus / バディコール プラス | 200 | default | — | `p+0x3A98` | `p+0x3B13` ? (chip 3) |
| 12 | Bolt Catcher / スクリューキャプチャー | 500 | Access lab on Saturday | `235380` | `p+0x3AA0` | `p+0x3B14` (chip 4) |
| 13 | Energy Catcher / リカバリーキャプチャー | 500 | Game Over 10× | `235399` | `p+0x3AA8` | `p+0x3B15` (chip 5) |
| 14 | Capsule Catcher / エネルギーキャプチャー | 500 | Deplete weapon/Rush gauge 30× | `2353B2` | `p+0x3AB0` | `p+0x3B16` (chip 6) |
| 15 | Shock Absorber / ショックアブソーバー | 300 | default | — | `p+0x3AB8` | `p+0x3B17` (chip 7) |
| 16 | Buster Plus Chip / バスタープラスチップ | 300 | Fire 500 buster shots (250 Newcomer) | `2353CB` | `p+0x3AC0` | `p+0x3B18` (chip 8) |
| 17 | Energy Balancer Neo / エネルギーバランサーNEO | 250 / 125 | Own Energy Balancer | `2353FC` | `p+0x3AC8` | `p+0x3B19` (chip 9) |
| 18 | Cooling System / クールダウンシステム | 300 | Activate gear 100× (50 Newcomer) | `235423` | `p+0x3AD0` | `p+0x3B1A` (chip 10) |
| 19 | Awakener Chip / 覚醒チップ | 3000 | Finish the game | `23543C` | `p+0x3AD8` | `p+0x3B1B` (chip 11) |
| 20 | Speed Gear Booster / スピードギアブースター | 300 | Activate Speed Gear 50× (25 Newcomer) | `235455` | `p+0x3AE0` | `p+0x3B1C` (chip 12) |
| 21 | Power Shield / パワーシールド | 300 | Take 200 damage (100 Newcomer) | `23546E` | `p+0x3AE8` | `p+0x3B1D` (chip 13) |
| 22 | Energy Dispenser / エネルギーディスチャージャー | 1000 | default | — | `p+0x3AF0` | `p+0x3B1E` (chip 14) |
| 23 | Cooling System ∞ / クールダウンシステム∞ | 3000 | Finish the game | `235487` | `p+0x3AF8` | `p+0x3B1F` (chip 15) |
| 24 | Spike Boots / スパイクブーツ | 300 | default | — | `p+0x3B00` | `p+0x3B20` (chip 16) |
| 25 | Mystery Chip / ミステリーチップ | 500 | default | — | `p+0x3B08` | passive |

**Chip flag base:** `p + 0x3B10` (1 byte per chip, contiguous). Chip indices 2 and 3 (`p+0x3B12`, `p+0x3B13`) are unconfirmed — likely Pierce Protector and Buddy Call Plus respectively.

**Note on Energy Balancer Neo (idx 17):** A function call at `2353E8` gates the check. If it returns 0, the item is always unlocked regardless. If non-zero, `setne al` at `2353FC` checks EnergyBalancer bought count. Patching `2353FC` covers the gated case.

---

## Memory Layout Notes

- Bought count stride: 8 bytes per item from `p+0x3A40` (byte at offset +0 of each slot)
  - CT Lua timer (1 s, ID=41 active) writes `1` to 18 non-consumable items when count is 0
- Item record stride: 3 bytes per item from `[rdi+0x4CF8]`
  - Byte 0: unlock slot (1 = visible in shop, 0 = hidden) — set by init loop, overwritten by conditions; patched by ID=41 AA script
  - Byte 1: purchase status flag — written by loop body at `game.exe+2354C0`
- Slot flag (separate): `p + 0x3A44 + item_index * 8` — written to `1` by game loop at `23549F`; does **not** control shop visibility; distinct from the bought count at offset +0
