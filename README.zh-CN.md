# fsbl-sg200x

[English](README.md) | 简体中文

NanoKVM 上 SG2002 的第一阶段引导程序（FSBL，相当于 TF-A BL2）和 FIP 打包。BootROM 加载 `fip.bin`，其中包含这个 FSBL、DDR 初始化、OpenSBI、U-Boot 和 C906L 小核固件。

| 路径 | 内容 |
|---|---|
| 仓库根目录 | FSBL 源码（LicheeRV Nano SDK 中的 `fsbl/`） |
| `boards/sg2002_licheervnano_sd/memmap.py` | NanoKVM 内存布局 |
| `scripts/mmap_conv.py` | 把内存布局转换为 `cvi_board_memmap.h` |

提交历史只保留 LicheeRV Nano SDK 中的这些路径，原作者和日期不变；每个提交都注明了对应的 SDK 提交。

## 构建

先生成内存布局头文件，再构建 FIP。FSBL 从源码树的 `build/` 目录包含该头文件。

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

生成的文件是 `$OUT/fip.bin`。OneKVM 用 `onekvm-distro` 中的 `sophgo-fsbl` 配方构建它。

## 许可证

仓库没有统一的许可证，以各文件头为准：源自 TF-A 的代码是 BSD-3-Clause，`lib/` 下的第三方库各自带有许可证文件，另有部分文件是 GPL-2.0+ 或 Unlicense。大多数厂商平台和 DDR 源文件没有许可证头。
