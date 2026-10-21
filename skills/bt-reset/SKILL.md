---
name: bt-reset
description: Escalating reset of the Linux Bluetooth stack — power-cycle the adapter, restart bluetoothd, reload the kernel module. Use only after bt-status shows a real fault (adapter missing, daemon dead, everything timing out), not as a first step for a device that just isn't showing in a GUI.
---

# Bluetooth Reset

Work down the ladder; stop at the first rung that fixes it and re-run `bt-status`
after each. Every rung drops all current Bluetooth connections — warn the user if
they are on a Bluetooth headset or keyboard.

## 1. Power-cycle the adapter (no sudo)

```bash
bluetoothctl power off; sleep 2; bluetoothctl power on
```

## 2. Unblock

```bash
rfkill unblock bluetooth
```

## 3. Restart the daemon

```bash
sudo systemctl restart bluetooth
```

Paired devices survive this; bonds live in `/var/lib/bluetooth/<adapter-MAC>/`.

## 4. Reload the kernel driver

Most laptop adapters are USB-attached and use `btusb`:

```bash
sudo modprobe -r btusb && sudo modprobe btusb
sudo systemctl restart bluetooth
```

Check the driver first — `lsmod | grep -E '^bt|bluetooth'` — some Intel and Qualcomm
combo chips also load `btintel` / `btqca`.

## 5. Check the kernel log

```bash
sudo dmesg | grep -iE 'bluetooth|btusb|hci0' | tail -30
```

`firmware … failed to load` means a missing firmware package (`linux-firmware`), not
something a restart fixes.

## Not a reset

Deleting `/var/lib/bluetooth` wipes every pairing on the machine. Never do it as a
troubleshooting step; use `bt-manage` → remove for the one device instead.

Rungs 3–5 are standard BlueZ/kernel practice, not exercised against a live fault yet
(as of 2026-10-21).
