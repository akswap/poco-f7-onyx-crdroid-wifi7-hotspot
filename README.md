# POCO F7 (onyx) crDroid Wi-Fi 7 + 6 GHz Hotspot (Magisk Module)

Exact-build Magisk/KernelSU/APatch module enabling **Wi-Fi 7 (802.11be)**, **6 GHz client/SoftAP**, **320 MHz channel width**, and automatic recovery from the 6 GHz hotspot's low-power `8.00 dBm` state on the POCO F7 (`onyx`).

## Verified result

- Device: POCO F7 (`onyx`)
- ROM: crDroid Android 16, v12.12, build 20260913
- Vendor: `OS3.0.7.0.WOLMIXM`
- Hotspot protocol reported by the client: `802.11be` Wi-Fi 7
- Band/channel: 6 GHz, channel 133 (`6615 MHz`)
- Channel width: 320 MHz
- Client negotiated link rate observed: 5188/5188 Mbps
- Security: WPA3-Personal
- Hotspot TX power after automatic recovery: `24.00 dBm` (driver-reported)
- Tested client: Intel Wi-Fi 7 BE200 320 MHz

## Download

Install the ZIP attached to the latest release:

`POCO-F7-crDroid-WiFi7-6GHz-v0.4-combined-test.zip`

SHA-256:

```text
bb895792a200e9b667a6f7128727214758358d7708a8167f64f0402916c163c3
```

## Exact compatibility guard

This module is intended only for the tested target:

```text
Device: onyx
ROM: crDroid Android 16 v12.12 (20260913)
Vendor fingerprint: POCO/onyx_global/onyx:15/AQ3A.250226.002/OS3.0.7.0.WOLMIXM:user/release-keys
Original hostapd SHA-256: 1622dc7e0ffe4205e806a0418eff87779cd4317a864f05865bbc986758087f5e
service-wifi.jar SHA-256: c9e9f08f4be3400f1014a122afedcc4e1460194f527d47d513ffe42f22f2e52e
```

Installation is rejected when the device, vendor build, or required hashes do not match. Do not use it on a different device or ROM without updating and retesting the guards.

## What the module changes

- Installs a signed persistent RRO enabling the Android Wi-Fi 7 and 6 GHz framework/SoftAP gates.
- Systemlessly supplies the tested EHT-capable Xiaomi.eu hostapd stack and compatible AIDL libraries.
- Creates a systemless copy of the installed WCNSS configuration and sets:

```text
BandCapability=7
scan_mode_6ghz=1
oem_6g_support_disable=0
```

- Maintains the tested US country-code override so the Qualcomm self-managed regulatory table exposes 6 GHz channels.
- Detects active AP interfaces dynamically and requests `txpower auto` only when the AP is on 6 GHz and reports 10 dBm or less.
- Does not modify 2.4/5 GHz hotspot TX power and does not force a fixed dBm value.
- Does not write the physical vendor partition.

## Installation

1. Remove or disable older standalone Wi-Fi 7/hostapd/6 GHz test modules.
2. Install the release ZIP using Magisk, KernelSU, or APatch.
3. Reboot.
4. Use WPA3-Personal for 6 GHz operation.

## Verification

Start the 6 GHz hotspot, wait around 10–20 seconds, then run as root:

```sh
cmd wifi get-country-code
iw reg get
iw dev wlan1 info
cat /data/adb/modules/poco_f7_crdroid_wifi7_test/runtime.log
```

The tested hotspot output was:

```text
type AP
txpower 24.00 dBm
channel 133 (6615 MHz), width: 320 MHz, center1: 6585 MHz
```

A compatible client should report `802.11be`. Android may still display `wifiStandard=6` internally because the crDroid framework uses hostapd AIDL v3 while the tested EHT hostapd service exposes AIDL v2; actual client negotiation is the authoritative test.

## Rollback

Disable or remove the module in the root manager and reboot. The module is systemless. If normal boot fails, create this file from recovery/root-manager safe mode and reboot:

```text
/data/adb/modules/poco_f7_crdroid_wifi7_test/disable
```

## Regulatory and safety warning

This test applies a US regulatory-domain override and requests the driver's automatic TX-power setting for an affected 6 GHz AP. Spectrum rules and permitted power levels vary by country. Use only where the selected channels and power levels are legally permitted.

The value shown by `iw` is a driver-reported setting, not proof of actual EIRP. Firmware, regulatory rules, antenna gain, SAR, thermal policy, and hardware power class can impose lower limits.

This release contains proprietary vendor components extracted from a user-owned Xiaomi.eu ROM for interoperability testing. No ownership is claimed; redistribution may be subject to the original vendor's terms.

## Credits

- Device testing and validation: AKHILESH KUMAR SHUKLA
- Android Wi-Fi resources: AOSP
- EHT hostapd/vendor components: Xiaomi.eu vendor image
