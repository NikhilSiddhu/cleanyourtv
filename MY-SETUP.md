# My setup — Fire TV Stick 3rd gen, Apple TV-style front end

Notes from the device this fork was set up on. The full write-up, scripts and screenshots live in
[firetv-tvos](https://github.com/NikhilSiddhu/firetv-tvos); this file is only the "what did I
change on my own TV" record.

## The device

| | |
|---|---|
| Model | Fire TV Stick 3rd gen, `AFTSSS` |
| System | Fire OS 7.7.1.7 (`PS7717/5741`) |
| Platform | Android 9 / SDK 28, 1 GB RAM |
| Stock launcher | `com.amazon.tv.launcher` — **protected**, `pm disable-user` refuses it |

That last line is why this setup differs from the Android TV ones, and it agrees with what
`examples/amazon-fire-tv-stick-lite-2020/debloat.txt` already says: on Fire OS the launcher keeps
Home and `launcher/` *as a replacement* does not apply. What I do instead is put the launcher back
in front every time Amazon's home appears, the same route the other Fire OS entries in
[`SHOWCASE.md`](SHOWCASE.md) take.

## What is installed

| Piece | Package | Notes |
|---|---|---|
| Launcher | `com.gathin.tvhome` | Built from `launcher/` in this repo, 477 KB, installed over ADB |
| Home watcher | `com.firetv.tvos` | My own app: an accessibility service that surfaces the launcher when Amazon's home appears |
| Web tiles | `com.firetv.webtiles` | My own app: any URL as a home tile, icon fetched from the site |
| Settings tile | `com.firetv.settings` | Quick-settings entry that stays reachable after the debloat |

## The debloat

71 packages, disable-only, never uninstalled:
[`scripts/debloat_fireos.txt`](https://github.com/NikhilSiddhu/firetv-tvos/blob/main/scripts/debloat_fireos.txt).

`AFTSSS` is not in `examples/` yet, and the closest list is
[`amazon-fire-tv-stick-lite-2020`](examples/amazon-fire-tv-stick-lite-2020/debloat.txt) (`AFTSS`,
a different stick that ships the **same** Fire OS build). I used it as the base and marked my own
additions `OURS` in the file, so the provenance of every line is visible.

Kept on deliberately, following that list: `com.amazon.tv.launcher`, `tv.settings.*`,
`com.amazon.venezia` (Appstore), `com.amazon.tv.ime`, Bluetooth and remote services, OTA updates,
Alexa voice search, Prime Video, casting, profiles, `tv.oobe`, the screensaver, accessibility
(`logan`) and parental controls.

Protected on this build, so `pm disable-user` is refused and the script skips them rather than
failing: `com.amazon.minerva.client.api`, `com.amazon.device.logmanager`,
`com.amazon.device.crashmanager`, `com.amazon.diode`, `com.fireos.usagestats.proxy`,
`com.amazon.tv.oobe`, `com.amazon.tv.parentalcontrols`, `com.amazon.device.sync*`,
`com.amazon.securitysyncclient`. These are the ones I still want gone and have not solved yet —
network-level DNS blocking is the intended next step, not a script.

## Restore

```bash
bash scripts/restore_debloat.sh     # re-enables every package the debloat disabled
adb uninstall com.firetv.tvos       # removes the watcher
adb uninstall com.firetv.webtiles
adb uninstall com.firetv.settings
adb shell settings put secure enabled_accessibility_services ""
```

The stick then boots to Amazon's home again, with the Appstore and the settings app intact.

## Two things worth knowing before you start

- Take the `getprop` and `pm list packages` backup **before** touching anything. Mine is in
  [`backup/`](https://github.com/NikhilSiddhu/firetv-tvos/tree/main/backup), serial redacted.
- The watcher reacts to Amazon's home window appearing, not to the key press: Fire OS holds a
  background app back for a few seconds after Home, so reacting to the key is too early. If it
  stops working after a reboot, check that the accessibility service is still enabled.
