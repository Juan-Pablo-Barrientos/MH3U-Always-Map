# Monster Hunter 3 Ultimate (MH3U) Always Have Map for Cemu

A Cemu graphic pack for **Monster Hunter 3 Ultimate** (MH3U, also called Monster Hunter Tri Ultimate) on the Wii U emulator. It shows the full map in quests without carrying the Map item, the same way Moga Woods always shows it. You no longer have to grab the Map from the supply box, and it doesn't take a pouch slot.

## Requirements

- Monster Hunter 3 Ultimate, **US**, title `00050000-10118300`, **update v32**
- Cemu 2.x, tested on Cemu 2.6

The patch only applies to that exact game build: the game code checksum must be `0x9d2f15e5`. On any other version, Cemu skips it and the game runs normally. EUR and MH3G aren't supported yet.

## Install

1. Copy the `MH3U_AlwaysMap` folder into Cemu's `graphicPacks` folder:
   - Normal install: `%APPDATA%\Cemu\graphicPacks\`. Create the `graphicPacks` folder if it doesn't exist.
   - Portable install: `graphicPacks\` next to `Cemu.exe`
2. **Restart Cemu.** It only reads graphic packs when it starts.
3. Open **Options → Graphic packs → Monster Hunter 3 Ultimate → Mods → Always Have Map** and tick it.
4. Start the game. To turn it off later, untick it and restart the game.

To confirm it's active, look in `%APPDATA%\Cemu\log.txt` for:

```
Applying patch group 'MH3U_AlwaysMap_US_v32'
```

## What changes

Only the "carrying a Map" requirement is removed. Every other rule the game uses to decide whether you see the map still applies, so a place or situation that never shows a map still won't.

The Map item itself still exists and works as before. You can still pick it up; it just isn't needed.

Tested on Cemu 2.6: in quests, the map shows without the Map item in the pouch.

## How it works

Addresses are Wii U addresses for US v32.

- The Map is item ID `1010` (`0x3F2`).
- The pouch is reached through the hunter object (class `0x1007F1A0`): the pointer at `+0xE30` leads to the 24 pouch slots of (16-bit item ID, 16-bit count), starting at `+0x4D8`.
- `getItemCount(player, itemId)` at `0x0285e7c8` finds the item's slot and returns its count.
- The function at `0x0260eea8` decides whether the map UI gets filled in. It returns:
  - **no map** for one type of location, looked up from the current location ID in a table at `0x10206ab0`;
  - **map** if a global flag (byte `+0x1b9` of the object pointed to by `0x101fdb9c`) is set, or if either of two player conditions (`0xdb`, `0x87`) is active;
  - **no map** if a third condition (`0x88`) is active;
  - otherwise, **map** only if `getItemCount(player, 1010)` isn't 0.

The patch replaces the branch after that item count (`0x0260ef34`, `beq`) with a `nop`, so the last case always returns "map". The checks before it run unchanged.

The conditions are tested with the function at `0x02891004`. Which skills or states they are hasn't been identified.

### How it was found

1. Use one Potion and compare memory before and after. A 16-bit value that went from 10 to 9 led to the pouch list.
2. The game's item names are a null-separated text list in memory, ordered by item ID. Counting from "Potion" (ID 9) gives Map = 1010.
3. Search the game code for instructions with `0x3F2` as an immediate. One of them, `li r4, 0x3f2` before `bl getItemCount`, sits in the map check above.

### Porting to other versions

On another build, the addresses and the `moduleMatches` checksum will be different. Look for `li r4, 0x3f2` followed by a call to the item-count function and a `beq` to a `li r3, 0` return, and replace that `beq` with a `nop`.

## License

[MIT](LICENSE)
