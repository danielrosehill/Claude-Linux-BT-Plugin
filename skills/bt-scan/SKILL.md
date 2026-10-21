---
name: bt-scan
description: Scan for nearby Bluetooth devices from the command line and list what was found with MAC addresses. Use when a device is in pairing mode but the desktop Bluetooth dialog doesn't show it, or to get a MAC before pairing.
---

# Bluetooth Scan

Ask the user to put the device in pairing mode **first**, then scan.

```bash
timeout 25 bluetoothctl --timeout 20 scan on 2>&1 | grep -E 'NEW|CHG.*Name' | sed 's/\x1b\[[0-9;]*m//g'
```

`--timeout` is what makes `bluetoothctl` exit on its own in non-interactive mode; the
outer `timeout` is a safety net. The `sed` strips the colour codes.

Output lines look like:

```
[NEW] Device FB:72:FA:E9:FB:28 MX Vertical
[NEW] Device 02:43:72:EC:DC:ED 02-43-72-EC-DC-ED
```

A name that is just the MAC with dashes means the device advertised no name — usually
a phone, beacon or someone else's gadget, not the thing you are pairing.

## Picking the right device

- Match on name. If two candidates exist, ask the user to switch the device off,
  rescan, and see which disappears.
- `bluetoothctl devices` afterwards lists everything cached, including devices paired
  long ago that aren't nearby. A device appearing there is **not** proof it is in
  range; a `[NEW]` line during this scan is.
- BLE devices (most modern mice and keyboards) often use a *random* address
  (`bluetoothctl info <MAC>` shows `(random)`). It can change after a factory reset or
  when switching Easy-Switch channels, so always take the MAC from a fresh scan.

## Why the desktop dialog misses things

The KDE/GNOME dialogs only show devices found while *their* scan is running, and they
stop scanning after a short window. A device put into pairing mode after that window
never appears. Not established which of these caused the 2026-10-21 miss; the CLI
scan simply has no such window. Hand off to `bt-pair` with the MAC.
