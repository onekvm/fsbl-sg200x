# fsbl-sg200x

First-stage boot loader (FSBL, acting as TF-A BL2) and FIP packaging for
the SG2002 on NanoKVM. The BootROM loads `fip.bin`, which carries this FSBL,
DDR initialisation, OpenSBI, U-Boot and the C906L firmware.

| Path | Content |
|---|---|
| repository root | FSBL sources (`fsbl/` in the LicheeRV Nano SDK) |
| `boards/sg2002_licheervnano_sd/memmap.py` | NanoKVM memory map |
| `scripts/mmap_conv.py` | Converts the memory map to `cvi_board_memmap.h` |

The commit history keeps only these paths from the LicheeRV Nano SDK, with
the original authors and dates; each commit names the SDK commit it came
from.

## Build

Generate the memory map header, then build the FIP. The FSBL includes the
header from `build/` in the source tree.

```sh
python3 scripts/mmap_conv.py --type h \
    boards/sg2002_licheervnano_sd/memmap.py build/cvi_board_memmap.h

make O="$OUT" ARCH=riscv BOOT_CPU=riscv CHIP_ARCH=sg200x \
    CROSS_COMPILE=riscv64-linux- DDR_CFG=ddr3_1866_x16 \
    MONITOR_PATH="$OPENSBI/fw_dynamic.bin" \
    LOADER_2ND_PATH="$UBOOT/u-boot.bin" \
    BLCP_2ND_PATH="$CVIRTOS/cvirtos.bin" \
    BLCP_PATH=test/empty.bin \
    LDLIBS="$LIBGCC" LDFLAGS=--no-warn-rwx-segments \
    FSBL_SECURE_BOOT_SUPPORT= TPU_PERF_MODE=y OD_CLK_SEL=y FIP_COMPRESS=lzma
```

The result is `$OUT/fip.bin`. OneKVM builds it with the `sophgo-fsbl`
recipe in `onekvm-distro`.

## License

There is no top-level license. Files carry their own headers: the TF-A
derived code is BSD-3-Clause, the bundled libraries under `lib/` ship their
own license files, and some files are GPL-2.0+ or Unlicense. Most vendor
platform and DDR files have no license header.
