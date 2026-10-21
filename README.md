# linux-bt

![License: MIT](https://img.shields.io/badge/license-MIT-green)
![BlueZ 5.x](https://img.shields.io/badge/BlueZ-5.x-0082fc)

Claude Code plugin for **Bluetooth on Linux from the command line**.

The desktop pairing dialogs (KDE, GNOME) are the weak link: they only list devices
found during their own short scan window, and they fail silently. `bluetoothctl` has
no such window and says exactly what went wrong. This plugin packages the sequences
that work so an agent can diagnose and pair without the GUI.

## Skills

| Skill | What it does | State |
| --- | --- | --- |
| `bt-status` | Adapter, rfkill, bluetoothd, recent daemon log — is the laptop side healthy? | Verified 2026-10-21 |
| `bt-scan` | Non-interactive scan; list discovered devices + MACs | Verified 2026-10-21 |
| `bt-pair` | Scan + pair + trust + connect in one piped `bluetoothctl` session, then verify | Verified 2026-10-21 (BLE mouse) |
| `bt-manage` | List paired/connected, connect, disconnect, remove, trust, block, battery | List/battery verified; rest standard BlueZ |
| `bt-reset` | Escalating reset: power-cycle → unblock → restart daemon → reload `btusb` | Rungs 1–2 trivial; 3–5 not yet exercised on a live fault |

Typical flow: `bt-status` → `bt-scan` → `bt-pair`.

## The case that prompted it

2026-10-21, Ubuntu 26.04, BlueZ 5.85, Plasma 6: a Logitech MX Vertical in pairing
mode never appeared in the KDE Bluetooth dialog. Adapter, rfkill and daemon were all
healthy; a CLI scan found the mouse within seconds and the `bt-pair` sequence paired
it first time. Nothing was wrong except the GUI.

## Key facts the skills encode

- `Discoverable: no` on the adapter is irrelevant to finding a mouse — it only
  controls whether other devices can find the laptop.
- Pair inside the same `bluetoothctl` session that is scanning. BLE devices drop out
  of BlueZ's cache and a cold `pair <MAC>` fails with `not available`.
- `trust` is what makes a device reconnect by itself after reboot/sleep.
- `bluetoothctl devices` with no filter includes every device ever *seen*; use
  `devices Paired` / `devices Connected`.
- Easy-Switch style multi-host devices bond per channel.

## Requirements

`bluez` (provides `bluetoothctl`), `rfkill`, systemd. Desktop-agnostic. `sudo` only
for `bt-reset` rungs 3–5.

## Install

Listed in the [`danielrosehill`](https://github.com/danielrosehill/Claude-Code-Plugins)
marketplace:

```
/plugin marketplace add danielrosehill/Claude-Code-Plugins
/plugin install linux-bt@danielrosehill
```

Skills are self-contained (commands inline, no scripts), so they work when read
outside an installed plugin.
