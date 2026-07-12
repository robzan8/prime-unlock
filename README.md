# prime-unlock

A browser-based patcher for Metroid Prime (GameCube) `.gci` save files. It
unlocks the memory-card-level "extras" (image galleries, ability to start in
Hard Mode, **Fusion suit**, NES Metroid) on an existing save file, without
requiring GBA link cables or beating the game.
It should not mess with your game progress.

Live tool: https://robzan8.github.io/prime-unlock/

## How it works

Known game codes: `GM8E` (NTSC-U/USA), `GM8P` (PAL/Europe), `GM8J`
(NTSC-J/Japan). The save format was only verified against an NTSC-U (`GM8E`)
file; the PAL and Japanese versions are expected to share the same layout,
but this has not been directly tested.

- Offset `0x152D`-`0x1538` (12 bytes, in the data block): "unlocked extras".
  Values applied: `FE 4F 9F FF FF FF FF FF FF FF FF E0`.
- Offset `0x40`-`0x43`: data block checksum. Algorithm: CRC-32 (poly
  `0xEDB88320`, init `0xFFFFFFFF`, reflected in/out) WITHOUT the final xor
  (a variant known as "JAMCRC"), computed over the bytes from `0x44` to the
  end of the file, written big-endian.

These offsets and the checksum algorithm were reverse-engineered by diffing
save files with and without the extras unlocked (no public documentation of
this specific region was found). A few things worth noting from that
process:

- The 12 bytes contain flags but also some counters or structured data.
  Because of this, the patch overwrites the full 12 bytes with the
  known-good values rather than OR-ing them into the existing bytes.
- The data block seems to contain 3 repeating ~0x3AC-byte regions
  (roughly at `0x15B9`-`0x1879`, `0x1965`-`0x1C25`, `0x1D11`-`0x1FD1`),
  likely corresponding to the 3 save slots, plus 3 single-byte "slot
  occupied" flags around `0x1488`-`0x148A`. These were not needed for the
  patch itself but are noted here in case they're useful for future
  investigation.
