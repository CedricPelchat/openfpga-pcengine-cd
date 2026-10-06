# Testing relative CUE paths (0.2.3-relpath)

Only the MPU firmware changes, so you only need [`release/pce_mpu_bios.bin`](release/pce_mpu_bios.bin).

- **`FILE` paths are relative to the CUE's folder.** For a CUE in
  `/Assets/pcecd/common/_Cues/Japan/`, the line
  `FILE "../../_Games/052 De-Ja/De-Ja (Japan) (Track 01).bin" BINARY` opens
  `/Assets/pcecd/common/_Games/052 De-Ja/De-Ja (Japan) (Track 01).bin`.
  `\` works like `/`.
- **CUEs that work today keep working.** If the path in the CUE doesn't exist,
  the firmware looks next to the CUE, as 0.2.3 did.
- **All bins of one game must be in one folder**, and each file name must be
  under 128 characters. Otherwise the core shows `DataFile Error 2`.
- **The firmware no longer limits the CUE size.** 0.2.3 broke on CUEs over
  15,872 bytes.

| `pce_mpu_bios.bin` | SHA-256 |
| --- | --- |
| 0.2.3 (original) | `5b49d891f83915b7fe06b4f860ec2025a78332cc49c4a7d41a6eb7adebd4f36a` |
| 0.2.3-relpath | `c7e3c3c2f299b4358e266d2cb5ae96a5b8bc19283bba7a5234e0feee2e7cd058` |

## Install as a separate core

This leaves the original core untouched.

1. Back up `/Saves/pcecd/`. Both cores use the same save files.
2. Copy (don't move) `/Cores/Mazamars312.PC Engine CD` to
   `/Cores/Mazamars312.PC Engine CD Test`.
3. In the **copy's** `core.json`, set `"shortname": "PC Engine CD Test"` and
   `"version": "0.2.3-relpath"`. The Pocket needs the folder name to be
   `author.shortname`.
4. Optional, only for CUEs over 16 KiB: in the copy's `data.json`, change the
   CUE slot's (`"id": 100`) `"size_maximum": "0x4000"` to `"0x10000"` (64 KiB).
   A 99-track CUE with long paths is about 34 KB.
5. Copy `release/pce_mpu_bios.bin` to
   `/Assets/pcecd/Mazamars312.PC Engine CD Test/pce_mpu_bios.bin`.

On the Pocket, launch **PC Engine CD Test**. To remove it, delete the two
`... Test` folders.

## What to try

| Test | Expected |
| --- | --- |
| A CUE that works on 0.2.3 | Plays as before |
| A CUE using `../../_Games/<game>/...`; play until CD audio (title music, a cutscene) | Boots, audio tracks play |
| The same CUE written with `\` | The same |
| A CUE with a wrong path | `DataFile Error 2` |

## When something fails

- **`DataFile Error 2`** lists the directory and file name of the **last**
  place the firmware looked. For a relative path that is the fallback next to
  the CUE, so check the path written in the CUE.
- **`DataFile Error 65535`** means a `FILE` line could not be read, for example
  a missing closing quote.
- **UART log.** Turn on *CD Debug* in the core settings. The MPU log then
  shows every path it tries in `dataslot_pathname_file_decode` lines. Reading
  it needs a cartridge-slot UART adapter: TX is `cart_tran_bank0[2]`, at
  115200 baud.

## Rebuilding and tests

```sh
sudo apt-get install gcc-riscv64-unknown-elf libnewlib-dev g++-multilib
src/cdfw/build.sh                                  # -> src/cdfw/app/build.riscv.pce_mpu_bios/pce_mpu_bios.bin
cd src/cdfw/tests/host && make test test32         # CUE loader tests on a PC
```
