# WiZ Connected bulb hardware/firmware research

Notes toward running open, auditable custom firmware (ESPHome/Tasmota) on WiZ Connected
smart bulbs (Signify Netherlands B.V.), collected while investigating why these bulbs
can't have their Wi-Fi network changed remotely without the official app.

**Status: hardware identification phase.** No chip has yet been confirmed on a specific
real bulb, no firmware has been flashed, nothing here has been tested on real hardware yet.
This is a working notebook, not a how-to guide.

## Chip/module family taxonomy (confirmed)

The WiZ Connected v2 Android app (`com.wizconnected.wiz2`, built with Flutter) embeds a
`FirmwareUpdateModuleNameFamilyEnum` naming (at least) six distinct hardware platforms
across WiZ's product line:

- `ESP8266`
- `ESP32C3`
- `ESP32S3`
- `ESP32Platform1`
- `ESP32Platform2`
- `BK7236S` (Beken, RISC-V-based)

Confirmed by `strings` on `libapp.so` inside the APK (the app's real logic is compiled Dart
AOT code; jadx's Java decompile is nearly empty for this app). The app also has a real
`DeviceModel` SQLite table (schema only, columns include `moduleNameId`,
`moduleNameFamily`, `hasBluetooth`, etc.) but the actual per-SKU model-to-chip mapping is
fetched from Signify's backend at runtime, not bundled in the APK.

**Not yet done**: intercepting that live API call to get the real per-SKU table, and
identifying which chip family any specific real bulb (by article/model number) actually
uses.

## Prior public teardowns (third-party, linked not re-hosted)

Three independent teardowns have found three different chip generations, confirming the
hardware is genuinely not uniform across WiZ's product line/years:

- [LimitedResults, "Pwn the WIZ connected" (2019)](https://limitedresults.com/results/pwn-the-wiz-connected) -
  ESP-WROOM-02 (ESP8266). No flash encryption, no secure boot, single-checksum-byte
  integrity check only. WiFi credentials stored in plaintext 4x in flash, printed in
  plaintext over UART during setup. `esptool.py` read/write works directly over UART.
- [blog.secco.dev, "LEDNIFY / WiZ Connected Lampen hacken" (2022)](https://blog.secco.dev/lednify-wiz-lampen-hacken/) -
  ESP32-U4WDH rev 3 across 3 real lamps (one flash-encrypted, two not). Got Tasmota
  (`tasmota32solo1.factory.bin`) running via the
  [`lbernstone/arduino-esp32-solo`](https://github.com/lbernstone/arduino-esp32-solo)
  toolchain.
- [Blakadder's Tasmota template for WiZ SKU 9290024502](https://templates.blakadder.com/wiz_9290024502.html) -
  ESP32-SOLO1, `tasmota32solo1` firmware, UART/serial flashing.

## A practical, non-destructive finding

The v2 app has a genuine graceful BLE Wi-Fi-reconfiguration flow for an *already-paired*
device (package `wiz_pairing_ble`, screen `FixWifiCredentialScreen`) - distinct from, and
much less destructive than, forcing a factory reset to re-open the bulb's own pairing-mode
hotspot. Only available on bulbs with a BT radio (the ESP32/BK7236 families have one; plain
ESP8266 generally doesn't).

## Why this exists

Homelab IoT devices each ship their own inconsistent cloud/security posture. The goal is
open, local-only, auditable firmware (ESPHome for ESP-class devices, OpenIPC for IP
cameras) instead of trusting N different vendors' firmware individually. See
[ESPHome](https://esphome.io/) and [OpenIPC](https://openipc.org/) for the target
firmware projects.

## License / contributions

No license decided yet. This is a personal research notebook; issues/PRs with
corroborating teardown data (chip IDs, model numbers, working flash recipes) are welcome.
