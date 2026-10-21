---
name: bt-status
description: Diagnose whether Linux Bluetooth itself is healthy — adapter present and powered, not rfkill-blocked, bluetoothd running, recent daemon errors. Run first whenever a device "can't be found" or won't connect, before blaming the device or the GUI.
---

# Bluetooth Status

One read-only pass that answers "is the laptop's side of Bluetooth working?".

```bash
bluetoothctl show | grep -E 'Controller|Powered|PowerState|Pairable|Discovering'
rfkill list bluetooth
systemctl is-active bluetooth
bluetoothctl devices Connected
journalctl -u bluetooth -n 30 --no-pager | grep -viE 'Endpoint registered'
```

The `grep -v` drops the ~20 `Endpoint registered` lines PipeWire produces on every
login; they are normal and bury anything useful.

## Reading it

| Signal | Healthy | If not |
| --- | --- | --- |
| `show` prints a `Controller` | yes | No adapter — `lsusb`, `dmesg \| grep -i bluetooth`, firmware missing |
| `Powered: yes` | yes | `bluetoothctl power on` |
| rfkill `Soft blocked: no` | no | `rfkill unblock bluetooth` |
| rfkill `Hard blocked: no` | no | Physical switch / BIOS / Fn key — software can't fix it |
| `systemctl is-active` → `active` | active | `sudo systemctl restart bluetooth` (see `bt-reset`) |

`Discoverable: no` is **not** a fault. It controls whether *other* devices can find
the laptop; it has no bearing on the laptop finding a mouse, keyboard or headset.

A healthy adapter here plus a device the desktop dialog "can't find" means the dialog
is the problem — go straight to `bt-scan`. Verified 2026-10-21 (Ubuntu 26.04, BlueZ
5.85, Plasma 6): every row above was healthy and a CLI scan found the mouse at once.
