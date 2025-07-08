## Overview

- CPU: Motorola 68010 at ~8MHz.
- RAM: Shipped with 512KB of Static RAM. Eight(?) SRAM slots of 512KB, up 4MB max RAM.
- ROM: TODO!
- IO Ports:
  - 4 expansion slots
  - 2 game controllers
  - PS/2 mouse/keyboard slot
  - (3.5")SCSI Hard disk drive
  - (3.5")DD Floppy disk drive
- Video: TODO!
- Audio: TODO!

> NOTES:
> (aCube) - IO: Maybe we can add a SD card slot?
> (aCube) - RAM: Decide how much RAM will be shipped by default, and how many SRAM
> expansion slots will be part of the system.
> (aCube) - RAM: Manage memory areas for each computer device.
> (aCube) - ROM: Decide how much ROM will be shipped for the OS. And build the OS.
> (aCube) - VIDEO: Choose video output connectors, eg: S-Video and VGA. Name the video
> composer and signal adapter. Set it's capabilities and programmable interface.

---

- Video:
  - Generated via FPGA.
    - Resolution: Fixed 320x240 output resolution.
    - VRAM: Dedicated chip with 128KB of memory.
    - Colors: Up to 256 colors can be displayed. 4 palettes each for sprites and backgrounds; Each palette has 16 indexes from the main palette.[^1]

---

- Sprites:
  - Screen can only display 92 sprites per frame.
  - Dimensions can be up to 64x64 and minimum of 8x8; Each dimension size can be changed independently.
  - All sprites can be affine transformed: Scaled, Rotated, Translated and Skewed.

Sprites OAM data:

> TODO!

| Byte | Bits        | Description                   |
| ---- | ----------- | ----------------------------- |
| ?    | `pp00 0000` | `p`: Set which palette to use |

---

- Background modes:

| Mode | Type   | Layers | Palette      | Size      | Capabilities                         |
| ---- | ------ | ------ | ------------ | --------- | ------------------------------------ |
| 0    | Tiled  | 4      | `64; 4 x 16` | 1024x1024 | Cannot move each layer independently |
| 1    | Tiled  | 2      | `64; 4 x 16` | 1024x1024 | Can move each layer independently    |
| 2    | Bitmap | 2      | `256`        | 320x200   | Double buffered                      |

The rendering output is VGA complaint[^3]:

- Tiled: `QVGA -> 320x240`
- Bitmap: `CGA -> 320x200`

On bitmap mode, the output mode is still `QVGA` with black bars. Even though the `Mode 2` is `CGA`, it cannot directly use the `RGB332` color format.

There's 4 palettes for each background and foreground layers; 4 palettes of 16 colors each. Those palettes must be loaded before usage, and can only use the fixed default 256 color palette[^1]. Palettes can be switched by setting the 2-bits `palette` metadata in `OAM/BlockMap` when sending to the `PPU`.

> NOTES:
> (aCube) - Mode 0: Maybe it's better to have 1 layer that can move independently, but the other 3 move together.
> (aCube) - Mode 1: Maybe add one more layer that move independently.

---

- APU: Z80 running at ~4MHz
  - PSG: 4 square and noise channels.
  - PCM: 4 sample channels; Sample Rate 8-bits.

> TODO: APU's capabilities are yet to be defined. Subject to changes: _Chip; PSG Channels; PCM Channels._

- CPU must send commands to the APU FIFO buffer, each command set the attributes and data of each channel. To send any command to a channel, is necessary to send a command data to the correct register.

- PSG registers:

| Address  | Lenght  | Name                         |
| -------- | ------- | ---------------------------- |
| 0x002000 | 2 bytes | Channel A status             |
| 0x002002 | 4 bytes | Channel A command attributes |
| 0x002006 | 4 bytes | Channel A command address    |
| 0x00200a | 2 bytes | Channel B status             |
| 0x00200c | 4 bytes | Channel B command attributes |
| 0x002010 | 4 bytes | Channel B command address    |
| 0x002014 | 2 bytes | Channel C status             |
| 0x002016 | 4 bytes | Channel C command attributes |
| 0x00201a | 4 bytes | Channel C command address    |
| 0x00201e | 2 bytes | Channel D status             |
| 0x002020 | 4 bytes | Channel D command attributes |
| 0x002024 | 4 bytes | Channel D command address    |

- PSG command attributes:

| Byte | Bits        | Attribute                                                         |
| ---- | ----------- | ----------------------------------------------------------------- |
| 0    | `LLLL LLLL` | How many bytes to read                                            |
| 1    | `cccc cc00` | The first 6 bits determine how many times this command will cycle |
| 2    | `t000 0000` | Bit 0 is the channel type: `0: Noise` \| `1: Square`              |
| 3    | `0000 0000` | Reserved                                                          |
| 5..7 | `.... ....` | Source address                                                    |

---

[^1]: Palette references: [[assets/palette.png]] [[vga_palette.png]]

[^2]: Default 256 palette: [[rgb332_palette.png]] [[rgb332_palette.aseprite]]

[^3]: Even though we use the `CGA` naming, the output never changes from `QVGA` resolution.
