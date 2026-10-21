---
name: bt-manage
description: Manage already-known Bluetooth devices on Linux — list paired/connected devices, connect, disconnect, forget (remove), block, and read battery level. Use for "connect my headphones", "what's paired", "forget that speaker", "mouse battery", or clearing a stale pairing before re-pairing.
---

# Manage Known Devices

`bluetoothctl` accepts one command per invocation for these; no session needed
because the device is already known to BlueZ.

## List

```bash
bluetoothctl devices Paired
bluetoothctl devices Connected
```

`bluetoothctl devices` with no filter also includes devices merely *seen* in past
scans (neighbours' phones, TVs), so don't present that list as "your devices".

Resolve a name to a MAC:

```bash
bluetoothctl devices Paired | grep -i 'bose' | awk '{print $2}'
```

## Connect / disconnect

```bash
bluetoothctl connect <MAC>
bluetoothctl disconnect <MAC>
```

`connect` fails with `Device not available` or `…Error.Failed` when the device is off,
out of range, or connected to another host. Say that rather than retrying.

## Battery

```bash
bluetoothctl info <MAC> | grep 'Battery Percentage'
```

`Battery Percentage: 0x64 (100)` — the decimal is in brackets. Only devices that
expose the Battery Service report this; many headsets don't.

## Forget (remove)

```bash
bluetoothctl remove <MAC>
```

Deletes the bond on the laptop side only. The device still remembers the laptop; for
a clean re-pair, also reset/clear pairing on the device, then use `bt-pair`. Confirm
with the user before removing a device they rely on.

## Trust / block

```bash
bluetoothctl trust <MAC>      # auto-reconnect allowed
bluetoothctl untrust <MAC>
bluetoothctl block <MAC>      # refuse all connections from it
bluetoothctl unblock <MAC>
```

A paired device that never reconnects by itself after reboot is usually `Trusted: no`.
