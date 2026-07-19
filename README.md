# atari_xl_fpga_common

Shared core for the Atari XL/XE FPGA projects. This repository holds the
reusable, board-independent parts of the Atari 800XL/5200 FPGA implementation:
the main core RTL, the ZPU soft-core with its supporting firmware services, and
the common building blocks used across every board port.

It is designed to be consumed as a **git submodule** by the board-level
repositories (for example
[atari_xl_fpga](https://github.com/scrameta/atari_xl_fpga) and
[tonnere](https://github.com/scrameta/tonnere)), so the core lives in exactly
one place and each board pins the precise revision it builds against.

> This code was migrated from a long-lived SVN history; some of the layout
> reflects that heritage.

## Contents

- [What this is](#what-this-is)
- [Layout](#layout)
- [The core](#the-core)
- [The ZPU and firmware](#the-zpu-and-firmware)
- [Simulation](#simulation)
- [Using this repository](#using-this-repository)
- [Licensing](#licensing)
- [Credits and references](#credits-and-references)

## What this is

The core implements the Atari 800XL (and, with a swapped memory map, the Atari
5200). It is written mostly in VHDL. A ZPU soft-core provides the "glue" a real
setup needs — SIO disk-drive emulation, on-screen menus/UI, timers and similar.

The same core is reused unchanged across every board target; each board
repository supplies only a thin top level (clocking, pin mapping, memory
controller, board-specific I/O) wrapped around what lives here.

## Layout

| Path | Description |
| --- | --- |
| `rtl/a8core` | The main Atari 800XL/5200 core RTL. |
| `rtl/components` | Shared, largely Atari-agnostic building blocks used by the core. |
| `rtl/zpu` | ZPU soft-core plus the extra functionality (drive emulator, timers, menus, etc.). |
| `rtl/romgen` | ROM generation helpers. |
| `testbench/tb`, `testbench/tb_*` | Testbenches for the core and sub-blocks (ANTIC, full 800XL). |
| `testbench/simulate_*.sh` | Simulation run scripts, with their `.cmd` / `.wcfg` waveform configs. |
| `COPYRIGHT_NOTICE` | Aggregated copyright notices for the various RTL/firmware pieces. |

## The core

The main core is in `rtl/a8core`, with reusable pieces factored out into
`rtl/components`. The design targets cycle-accurate 800XL behaviour and is shared
across all board ports. The 5200 variants reuse the same core with the 5200
memory map and I/O differences — the differences are supplied by the consuming
board repository, not here.

## The ZPU and firmware

`rtl/zpu` contains the ZPU soft-core and the supporting services that run on it: the
SIO drive emulator, the menu/UI system, timers and housekeeping. The board
repositories compile this to a ZPU ROM that is embedded into the built core. A
ZPU toolchain is required to (re)build the firmware.

## Simulation

Testbenches live under `testbench/` (`tb`, `tb_antic`, `tb_atari800xl`), driven
by the `testbench/simulate_*.sh` scripts and the associated `.cmd` / `.wcfg`
waveform configs. These let you exercise the core (and sub-blocks such as ANTIC)
without a board.

## Using this repository

This repo is meant to be added as a submodule of a board project, typically at
`common/`. In a consuming repository:

```sh
# add it once (board-repo maintainer)
git submodule add https://github.com/scrameta/atari_xl_fpga_common common
git commit -m "Add common core as submodule"
```

To check out a board project that already uses it:

```sh
# clone a consumer with the submodule populated
git clone --recursive https://github.com/scrameta/atari_xl_fpga

# …or, if you already cloned without --recursive:
git submodule update --init --recursive
```

To move a board project onto a newer core, bump the submodule pointer:

```sh
cd common
git checkout <desired-commit-or-branch>
cd ..
git add common
git commit -m "Bump common to <rev>"
```

Because each board repo records a specific commit of this repository, an old
checkout of a board still builds against the exact core it was developed with.

## Licensing

**Please read this carefully — the licensing here is genuinely mixed and, in
places, unsettled.**

This repository combines RTL and firmware from many sources. The author does
**not** own copyright in all of it, so a single blanket open-source licence
(GPL or otherwise) **cannot** be applied to the whole tree. See the
`COPYRIGHT_NOTICE` file, which aggregates the copyright notices for the various
components — note that it is acknowledged to be **incomplete**.

For the parts authored by the project author, the intent is:

> **Free to use.** If you want to use it commercially, or build significantly on
> it in a commercial product, please get in touch first.

Third-party components (e.g. the ZPU and other imported IP) remain under their
own respective licences, which take precedence for those files.

Because some included IP may carry redistribution or build restrictions, no
warranty is made that a complete build is free of such constraints. If you are a
rights-holder for any included component and something here is mislabelled or
shouldn't be present, please contact the author and it will be corrected.

*(This section is a description of intent, not formal legal advice, and is
expected to be tightened up over time.)*

## Credits and references

- **Author:** Mark ("nick foft" on the AtariAge forums) — <https://www.64kib.com/>
- **Phaeron (Avery Lee)** — for **Altirra** and the superb *Altirra Hardware
  Reference Manual*, an invaluable reference throughout this project:
  <https://virtualdub.org/downloads/Altirra%20Hardware%20Reference%20Manual.pdf>

### Consuming projects

- [atari_xl_fpga](https://github.com/scrameta/atari_xl_fpga) — the multi-board
  Atari 800XL/5200 FPGA project and chip replacements.
- [tonnere](https://github.com/scrameta/tonnere) — Project Thunder / MegaXE, the
  successor board to the EclaireXL.

### Threads and background

- Main announcement / discussion thread — *Potential new hardware*:
  <https://forums.atariage.com/topic/213827-potential-new-hardware/>
- Decap and original schematics (chip reverse-engineering reference):
  <https://forums.atariage.com/topic/223747-decap-and-original-schematics/>
- Atari 8-bit family background (Wikipedia):
  <https://en.wikipedia.org/wiki/Atari_8-bit_computers>
