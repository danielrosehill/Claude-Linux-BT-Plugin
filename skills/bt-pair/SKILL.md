---
name: bt-pair
description: Pair, trust and connect a Bluetooth device (mouse, keyboard, headset, speaker) on Linux in one non-interactive bluetoothctl session, then verify it. Use when the user wants to pair a device, or after bt-scan found a device the desktop GUI couldn't.
---

# Bluetooth Pair

Needs the device's MAC (from `bt-scan`) and the device **still in pairing mode** —
most drop out after 1–3 minutes, so don't dawdle between scan and pair.

```bash
M=FB:72:FA:E9:FB:28   # from bt-scan
{
  echo "scan on";   sleep 5
  echo "pair $M";   sleep 12
  echo "trust $M";  sleep 1
  echo "connect $M"; sleep 6
  echo "info $M";   sleep 1
  echo "scan off"
} | bluetoothctl 2>&1 | sed 's/\x1b\[[0-9;]*m//g' \
  | grep -E 'Pairing|trust|Connection|Failed|Error|Paired:|Bonded:|Trusted:|Connected:|Battery|Name:'
```

## Why it is one piped session, not four `bluetoothctl pair …` calls

- **The scan must be running in the same session.** A BLE device seen in an earlier
  scan can be evicted from BlueZ's cache before you pair, and `pair` then fails with
  `Device <MAC> not available`. Starting `scan on` inside the session re-discovers it.
- **The session provides the pairing agent.** Interactive `bluetoothctl` registers a
  default agent; a one-shot `bluetoothctl pair <MAC>` may exit before the exchange
  finishes.
- **`trust` is what makes it reconnect by itself** after reboot or sleep. Pairing
  without trust is the usual cause of "it worked yesterday, now I have to re-pair".

The `sleep`s are generous on purpose. If `pair` is still mid-exchange when `trust`
arrives, the trust usually still applies, but re-run if `Paired: yes` is missing.

## Success looks like

```
Pairing successful
Changing <MAC> trust succeeded
Connection successful
Paired: yes  Bonded: yes  Trusted: yes  Connected: yes
Battery Percentage: 0x64 (100)
```

Then ask the user to move the mouse / type / play audio. `Connected: yes` is the
daemon's view; the device actually working is the real test.

## Failures

| Output | Meaning | Do |
| --- | --- | --- |
| `Device … not available` | Not discovered this session | Device left pairing mode — re-enter it, re-run |
| `Failed to pair: org.bluez.Error.AuthenticationFailed` | Stale bond on one side | `bt-manage` → remove, put device in pairing mode, re-run |
| `Failed to pair: …AlreadyExists` | Already paired | Skip to `trust` + `connect` |
| `Failed to connect: …br-connection-profile-unavailable` | Audio device, no audio stack profile | Check PipeWire / `wireplumber` is running |
| Passkey printed (`Passkey: 123456`) | Keyboard wants it typed | User types it on the keyboard + Enter. Inferred from BlueZ behaviour, not yet exercised with this skill |

## Multi-host devices (Logitech Easy-Switch etc.)

The device pairs to whichever channel (1/2/3) was selected when pairing mode was
entered, and only reconnects to this machine on that channel. Tell the user which
channel they used.

Verified 2026-10-21: Logitech MX Vertical (BLE, random address), Ubuntu 26.04, BlueZ
5.85 — the KDE dialog never listed it; this sequence paired it first time.
