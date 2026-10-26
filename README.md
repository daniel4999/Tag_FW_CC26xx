# Tag_FW_CC26xx

[OpenEPaperLink](https://github.com/OpenEPaperLink/OpenEPaperLink) firmware for CC26xx based tags.

## Community wait-fix release

This fork provides a CC2630 block-wait fix for image-transfer hangs exposed by
OpenEPaperLink 3.00 / ESP32-C6 radio 0024's 15 ms cached-block timing.

[Download v0.1.0](https://github.com/daniel4999/Tag_FW_CC26xx/releases/tag/v0.1.0)
for the tested firmware, flashing instructions, source patch and checksums.
Read the [investigation and validation report](docs/cc2630-block-wait-underflow.md)
before flashing. The patched spare completed multi-block image delivery and
later check-ins using the published, unmodified official C6 application.
Testing is limited to one patched tag; long-term and all-board validation remain open.

This is a **community fork and release**, not an upstream-endorsed release.
Confirm each board is compatible before programming. Current firmware requires
wired flashing; the AP's generic firmware-update button is not supported here.

### Credits

- Original firmware: [tsl0922 and upstream contributors](https://github.com/tsl0922/Tag_FW_CC26xx).
- Bug discovery, reproduction and physical hardware validation:
  **[Daniel (@daniel4999)](https://github.com/daniel4999)**.
- Source investigation, patch and regression tooling were prepared with Codex assistance.

See [CREDITS.md](CREDITS.md) for attribution and [CHANGELOG.md](CHANGELOG.md) for scope.

## Supported Chip / Tags

- CC2630: `TG‐GM29MT1C` / `ST‐GM29MTF` (sleep current: `~2uA`)

### Specs

* TI [CC2630](https://www.ti.com/product/CC2630) ARM Cortex-M3 core, 128KB flash, 8KB SRAM, 20KB Ultra Low-Leakage SRAM, 24MHZ clock*,  802.15.4 2.4Ghz radio
* 2.9” ePaper display (`UC8151` or `SSD1680`)
* SPI EEPROM (1M, W25Q80)
* [Dedicated wake-IC](https://twitter.com/atc1441/status/1388252497316392963)
* NXP NFC controller (I2C)
* 2x CR2450 Coin cell

ST-GM29MTF uses the same PCB as TG-GM29MT1C, but without the LED and NFC chip (not soldered).

### Photos
![screenshot](images/screenshot.jpg)
![photo](images/photo.jpg)

### SoC and Debug pinouts

![pcb](images/pcb.jpg)

To program or read-out this tag you have two options:

- Connect BL pin to GND and power on (or reset if already powered on) to enter bootloader mode, then use [cc2538-bsl](https://github.com/JelmerT/cc2538-bsl) to flash firmware via UART.
- cJTAG or JTAG programmer (J-Link or TI's XDS110), you need to solder the debug pins from the SOC, which is pretty hard for most people.

## Develop

- Install [toolchain](https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-rm/9-2020q2/gcc-arm-none-eabi-9-2020-q2-update-win32.exe)
- Build: `make`
- Flash: `make flash PORT=COM3`

## License

[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
