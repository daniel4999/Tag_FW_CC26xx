# CC2630 block-wait underflow with OpenEPaperLink 3.00

Investigation and proposed fix, 26 October 2026. **The patched tag firmware has
not yet been flashed or hardware-tested.** Radio-side tests below used the
unchanged tag firmware. The existing AP workarounds should remain until a
patched tag has been tested with stock timing.

## Scope and symptoms

The tested tag is a SoluM TG-GM29MT1C, TI CC2630F128, 296x128 monochrome, built
from upstream commit `5e41218a32a0600211ae5a401d15712a4165bb26`. Its preserved
128 KiB BIN CRC32 `681f555a` matches the recorded successful original flash.
The tag advertises hwType 0x27 / software version 0; those fields alone do not
identify its processor. Confirm each board independently before flashing.

The AP is a yellow ESP32-S3 / ESP32-C6 running OpenEPaperLink 3.00, commit
`44963c7118aee1718a24f93d23450c15a1a1fbe4`, with radio version 0024.

On a new 4736-byte raw monochrome image (4096 + 640 bytes), upload/preload
succeeds and the AP logs block 0 served. No subsequent block 1 request or
completion follows; the image remains pending and later normal check-ins stop
within the observation window. Physical power-cycling recovers the tag, but
another transfer can trigger the failure again. AP logs show attempted bursts,
not proof of RF reception by the tag.

## Source and binary evidence

`getDataBlock()` in `source/syncedproto.c` previously called
`delay(ack->pleaseWaitMs - 20)` for every nonzero wait. `pleaseWaitMs` is uint16_t
and `delay()` takes uint32_t. With the target's 32-bit integer promotions, 15 -
20 becomes -5 and then converts to **4294967291 ms, approximately 49.71 days**.
All nonzero waits below 20 expose the same class of error.

`source/periph.c` implements `delay()` as a loop on `millis()` while `yield()`
feeds the watchdog. The tag can remain before `blockRxLoop()` instead of
recovering through the watchdog or returning to sleep. Remaining active may
also affect battery life; current draw was not measured.

The matching original ARM ELF contains a 16-bit wait load, unchecked subtract
of 20, and call to delay. This defect is present in the known-flashed binary.

The [OEPL C6 source at the tested release](https://github.com/OpenEPaperLink/OpenEPaperLink/blob/44963c7118aee1718a24f93d23450c15a1a1fbe4/ARM_Tag_FW/OpenEPaperLink_esp32_C6_AP/main/main.c#L98)
advertises 15 ms for cached blocks, 30 ms for blocks coming, and 140 ms for
uncached downloads. The cached path exposes an existing tag arithmetic bug.

## Controlled radio comparisons

| C6 cached-block wait | Original tag delay argument | Observed result |
| --- | ---: | --- |
| Official 0024, 15 ms | 4294967291 ms | Block-0 stall |
| Rebuilt control, 15 ms | 4294967291 ms | Same stall |
| Diagnostic 0024, 30 ms | 10 ms | Both blocks, then XFER_COMPLETE |
| Diagnostic 0024, 140 ms | 120 ms | Multiple distinct images completed; later check-in |

Local controls used the same source, ESP-IDF 5.5.5, and SDK configuration,
varying only PLEASE_WAIT_CACHED_MS. The tag firmware was unchanged. These are
bounded tests, not proof of long-term reliability, all-board compatibility, or
a minimum required radio delay. A version string of 0024 cannot distinguish
stock and modified radio binaries.

## Proposed fix and checks

Guard the subtraction when the advertised wait exceeds 20 ms. For 0..20 ms,
continue into reception immediately; for larger waits preserve the existing
20 ms compensation. No OTA/config features, display changes, identity changes,
or tag version bump are included.

Run the source-derived host C regression with:

```sh
python3 tests/test_block_wait.py
```

It extracts the actual wait block from getDataBlock(), replaces delay with an
argument-recording stub, reproduces the original 15 ms wrap, and checks the
delay value and call count for all 65536 uint16 inputs. It does not simulate RF.
The existing build workflow runs this check after the ARM build.

Local full-image validation used GNU Arm Embedded 9-2020-q2-update / GCC 9.3.1
with the original flags. The baseline rebuild matched the previously flashed
image byte for byte. The complete candidate links, its disassembly checks the
bound before subtraction, and its CCFG/bootloader configuration is unchanged.

## Hardware validation still required

1. Confirm one board's CC2630 identity; save its full flash/CCFG and IEEE data.
2. Flash the patched application by wire, preserving its bootloader settings.
3. Confirm ordinary image delivery with the existing AP workaround first.
4. In an isolated test, use stock C6 0024 / 15 ms cached timing. Send distinct
   multi-block images and verify completion, later check-ins, and later updates.
5. Observe continued operation before changing the household AP workarounds.

The current tag receive switch has no handlers for generic AP firmware-update
type 3 or tag-config type 0xA8. Those AP buttons cannot install this compiled
fix. This branch does not add an OTA updater.
