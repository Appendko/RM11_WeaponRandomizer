# Shop Inventory Structure

Base pointer: `[game.exe + 0xC3F710]` = p (changes each session)

Each item occupies 8 bytes. The bought count field is at `p + 0x3880 + item_index * 8 + 0x1C0` = `p + 0x3A40 + item_index * 8`.

Item records table: `[rdi + 0x4CF8] + item_index * 3` (byte 0 = unlock slot, set by condition checks or init).

---

## Always Available (default unlock)

Initialized to `1` at function entry; no condition check overwrites them.

| Idx | Item (EN / JP) | Price | Offset |
|-----|---------------|-------|--------|
| 0  | 1-UP / 1UP | 50 / 40 | `p+0x3A40` |
| 1  | Energy Tank / E缶 | 100 / 80 | `p+0x3A48` |
| 2  | Mystery Tank / M缶 | 300 / 240 | `p+0x3A50` |
| 3  | Weapon Tank / W缶 | 100 / 80 | `p+0x3A58` |
| 4  | Energy Balancer / エネルギーバランサー | 100 | `p+0x3A60` |
| 6  | Super Guard / ガードアップ | 100 | `p+0x3A70` |
| 7  | Auto-Charge Chip / オートチャージチップ | 300 | `p+0x3A78` |
| 8  | Eddie Call / エディーコール | 20 / 16 | `p+0x3A80` |
| 9  | Beat Call / ビートコール | 50 / 40 | `p+0x3A88` |
| 10 | Tank Container / 缶袋 | 200 | `p+0x3A90` |
| 11 | Buddy Call Plus / バディコール プラス | 200 | `p+0x3A98` |
| 15 | Shock Absorber / ショックアブソーバー | 300 | `p+0x3AB8` |
| 22 | Energy Dispenser / エネルギーディスチャージャー | 1000 | `p+0x3AF0` |
| 24 | Spike Boots / スパイクブーツ | 300 | `p+0x3B00` |
| 25 | Mystery Chip / ミステリーチップ | 500 | `p+0x3B08` |

---

## Conditionally Unlocked

Condition checks overwrite the item record byte 0. Patch the `set` instruction to `mov al, 1` + NOP (`B0 01 90`) to force unlock.

| Idx | Item (EN / JP) | Price | Unlock Condition | Patch address |
|-----|---------------|-------|-----------------|---------------|
| 5  | Pierce Protector / ショックガード | 50 | Die on spikes 5 times | `235374` |
| 12 | Bolt Catcher / スクリューキャプチャー | 500 | Access the lab on a Saturday | `235380` |
| 13 | Energy Catcher / リカバリーキャプチャー | 500 | Get a Game Over 10 times | `235399` |
| 14 | Capsule Catcher / エネルギーキャプチャー | 500 | Deplete any Special Weapon / Rush gauge 30 times | `2353B2` |
| 16 | Buster Plus Chip / バスタープラスチップ | 300 | Fire 500 Mega Buster shots (250 Newcomer) | `2353CB` |
| 17 | Energy Balancer Neo / エネルギーバランサーNEO | 250 / 125 | Obtain the Energy Balancer | `2353FC` |
| 18 | Cooling System / クールダウンシステム | 300 | Activate Power/Speed Gear 100 times (50 Newcomer) | `235423` |
| 19 | Awakener Chip / 覚醒チップ | 3000 | Finish the game | `23543C` |
| 20 | Speed Gear Booster / スピードギアブースター | 300 | Activate Speed Gear 50 times (25 Newcomer) | `235455` |
| 21 | Power Shield / パワーシールド | 300 | Take 200 total damage (100 Newcomer) | `23546E` |
| 23 | Cooling System ∞ / クールダウンシステム∞ | 3000 | Finish the game | `235487` |

All `set` instructions are 3 bytes (`0F 9x C0`); replace with `B0 01 90` (`mov al, 1` + NOP).

**Note on Energy Balancer Neo (idx 17):** A function call at `2353E8` gates the check. If it returns 0, the item is always unlocked. If non-zero, `setne al` at `2353FC` checks EnergyBalancer bought count. Patching `2353FC` covers the gated case.

---

## Memory Layout Notes

- Bought count stride: 8 bytes per item from `p+0x3A40`
- Item record stride: 3 bytes per item from `[rdi+0x4CF8]`
  - Byte 0: unlock slot (1 = available, 0 = hidden) — set by init loop, overwritten by conditions
  - Byte 1: purchase status flag — written by loop body at `game.exe+2354C0`
  - Byte 2: `p + item_index*8 + 0x3A46` value — written by init loop
- Unlock flag (separate): `p + 0x3A44 + item_index * 8` — written to `1` by game loop at `23549F`; does **not** directly control shop visibility
