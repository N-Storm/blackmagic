# Common code for the Black Magic Probe Firmware for WeAct Studio F401CC/F401CE/F411CE boards

This code allows using a [Black Pill F4](https://github.com/WeActStudio/WeActStudio.MiniSTM32F4x1) as a Black Magic Probe.

This directory contains the common code for the following platforms:
- [blackpill-f401cc](./../../blackpill-f401cc/README.md)
- [blackpill-f401ce](./../../blackpill-f401ce/README.md)
- [blackpill-f411ce](./../../blackpill-f411ce/README.md)

## Pinout

| Function        | Pinout | Alternative pinout 1 | Cluster   |
| --------------- | ------ | -------------------- | --------- |
| TDI             | PB6    | PB5                  | JTAG/SWD  |
| TDO/TRACESWO    | PB7    | PB6                  | JTAG/SWD  |
| TCK/SWCLK       | PB8    | PB7                  | JTAG/SWD  |
| TMS/SWDIO       | PB9    | PB8                  | JTAG/SWD  |
| nRST            | PA5    | PB4                  | JTAG/SWD  |
| TRST (optional) | PA6    | PB3                  | JTAG/SWD  |
| UART TX         | PA2    | PA2                  | USB USART |
| UART RX         | PA3    | PA3                  | USB USART |
| Power pin       | PA1    | PB9                  | Power     |
| LED idle run    | PC13   | PC13                 | LED       |
| LED error       | PC14   | PC14                 | LED       |
| LED bootloader  | PC15   | PC15                 | LED       |
| LED UART        | PA4    | PA1                  | LED       |
| User button KEY | PA0    | PA0                  |           |

## Instructions for Makefile buildsystem and ST MaskROM
### How to Build

In the following commands replace `blackpill-f4x1cx` with the platform you are using, e.g. `blackpill-f401cc`.

To build the code using the default pinout, run:

```sh
cd blackmagic
make clean
make PROBE_HOST=blackpill-f4x1cx
```

or, to use alternative pinout 1, run:

```sh
cd blackmagic
make clean
make PROBE_HOST=blackpill-f4x1cx ALTERNATIVE_PINOUT=1
```

or, if you are using a PCB (printed circuit board) as a shield for your Black Pill F4, run:

```sh
cd blackmagic
make clean
make PROBE_HOST=blackpill-f4x1cx SHIELD=1
```

### Firmware upgrade with dfu-util

- Install [dfu-util](https://dfu-util.sourceforge.net).
- Connect the Black Pill to the computer via USB. Make sure it is a data cable, not a charging cable.
- Force the F4 into system bootloader mode:
  - Push NRST and BOOT0
  - Wait a moment
  - Release NRST
  - Wait a moment
  - Release BOOT0
- Upload the firmware:
```
./dfu-util -a 0 --dfuse-address 0x08000000:leave -R -D blackmagic.bin
```
- Exit dfu mode: press and release nRST. The newly flashed Black Magic Probe should boot and enumerate.

For other firmware upgrade instructions see the [Firmware Upgrade](https://black-magic.org/upgrade.html) section.

## SWD/JTAG frequency setting

In special cases the SWD/JTAG frequency needs to be adapted. The frequency can be set to `900k`, as this value was found to be helpful in practice, using:

```
mon freq 900k
```

For details see the [pull request](https://github.com/blackmagic-debug/blackmagic/pull/783#issue-529197718) that implemented the `mon freq` command.


## Performance

System clock is set to either 84 MHz or 96 MHz depending on chosen platform, expecting a 25 MHz HSE crystal.

Because of low expectations to signal integrity or quality wiring, default SWD frequency is reduced to 3 MHz, but in practice 8 MHz is possible, JTAG is slower.

Aux serial runs via USART2 DMA on APB1 (Pclk = 48 MHz) at 732..3000000 baud (6Mbaud with OVER8).

TraceSWO Async capture runs via USART1 DMA on APB2 (Pclk = 96 MHz) at 1465..6000000 baud (12Mbaud with OVER8).

SPI ports are set to Pclk/8 each (use with `bmpflash`). As SPI1 pins may conflict with certain functions, and platform code does not bother restoring them, please soft-reset the probe when done with SPI operations.

## Instructions for Meson buildsystem

### Variant with meson and BMD bootloader enabled (default)

0. Clone the repo and libopencm3 submodule, install toolchains, meson, etc.

```sh
git clone https://github.com/blackmagic-debug/blackmagic.git
cd blackmagic
git submodule update --init
```

1. Create a build configuration for specific platform (WeActStudio MiniSTM32F401CC) with specific options (enable BMD bootloader flags and offsets -- this is the default)

```sh
meson setup build --cross-file=cross-file/blackpill-f401cc.ini -Dbmd_bootloader=true
```

2. Compile the firmware and bootloader

```sh
ninja -C build blackmagic_blackpill_f401cc_firmware.bin
ninja -C build boot-bin
```

3. Flash the "BMD bootloader" to blank board in ST MaskROM (BOOT0) mode (then it switches to this bootloader):

```sh
dfu-util -d 0483:df11 -S 375F334C3032 --alt 0 -s 0x08000000:leave -D build/blackmagic_blackpill_f401cc_bootloader.bin
```

4. Flash the [Black Magic Debug Probe compatible platform] firmware at offset to board which is running BMD bootloader (then it reboots and jumps to firmware):

```sh
dfu-util -d 1d50:6018 -S 375F334C3032 --alt 0 -s 0x08004000:leave -D build/blackmagic_blackpill_f401cc_firmware.bin
```

5. Verify the board enumerates, jumps between bootloader and firmware (hold PA0 onboard pushbutton active-low on reset to stay in bootloader) by `dfu-util --detach` and `dfu-util -s :leave`, verify the firmware works with GDB and BMDA HL-remote and bmpflash SPI.

### Variant with meson and ST MaskROM for bootloader
1. Create a build configuration for specific platform (WeActStudio MiniSTM32F401CC) with specific options (disable BMD bootloader flags and offsets, use ST MaskROM)

```sh
meson setup build --cross-file=cross-file/blackpill-f401cc.ini -Dbmd_bootloader=false
```

2. Compile the firmware

```sh
ninja -C build blackmagic_blackpill_f401cc_firmware.bin
```

3. Flash the [Black Magic Debug Probe compatible platform] firmware to blank board in ST MaskROM (BOOT0) mode (then it reboots and jumps to firmware), this will take a while:

```sh
dfu-util -d 0483:df11 -S 375F334C3032 --alt 0 -s 0x08000000:leave -D build/blackmagic_blackpill_f401cc_firmware.bin
```

4. Verify the board enumerates, jumps between MaskROM and firmware (hold PA0 onboard pushbutton active-low on reset to make it jump to MaskROM, or hold BOOT0 to enter MaskROM directly) by `dfu-util --detach` and `dfu-util -s :leave`, verify the firmware works with GDB and BMDA HL-remote and bmpflash SPI.

## More details

* The Makefiles are the historical build system, and it also supports both variants: pass `BMD_BOOTLOADER=1` makeflag to switch into two-binaries configuration (was `BMP_BOOTLOADER=1` for a year between PR1508 and PR1843).
* ST MaskROM is the read-only System Memory bootloader which starts up per BOOT0-triggered SYSCFG mem-remap. It talks USART AN2606 so you can use stm32flash etc. When USBOTG_FS is plugged, it spins up HSE 25 MHz and measures its frequency relative to HSI (RC 8 MHz with worse tolerance) and sometimes can misdetect it as 24 or 26 MHz then configure a wrong input PLL divider, resulting in broken USB DFU. New 8 MHz HSE boards should not be affected by this, but also they are not yet supported (needs a different pll config).
* BMD bootloader is the port of BMP bootloader with dfu_f1.c internal flash driver swapped to dfu_f4.c (and different serialno algorithm etc.) which
    a) has a fixed per-board PLL config, no autodetection;
    b) understands buttons, drives LED, does not touch other GPIOs, talks USB DfuSe, ~~has MS OS descriptors for automatic driver installation on Windows~~, uses same libopencm3 code so you can verify hardware config via a smaller binary;
    c) erases and writes to internal Flash ~2.4x faster than MaskROM;
    d) all of that in first 8-9 KiB of first page of 16 KiB (of F2/F4/F7 flash), just like on `native`/`stlink`/`swlink` etc.