# DisableRamdump — Magisk Module

> ⚠️ **BETA** — Tested logic, but not yet verified across all OxygenOS builds.
> Flash at your own risk and check the log after first boot.

---

## What it does

OnePlus devices running Qualcomm SoCs silently accumulate crash diagnostic data in `/data/vendor/ramdump`. This folder is hidden from the storage UI and can grow to **10–30+ GB**, causing phantom storage loss.

This module does three things on every boot:

1. **Disables ramdump collection** via `persist.vendor.ramdump.enable 0`
2. **Cleans existing dump files** — removes known Qualcomm SSR artifacts (`.elf`, `.dump`, `.gz`, `.lzo`, `DDRCS*`, `load.cmm`, `rpmh_data*`)
3. **Locks the directory** — applies `chmod 000` + `chattr +i` so it can never be written to again

---

## Requirements

- Magisk v20.4 or newer
- Rooted OnePlus device (OxygenOS, tested on Qualcomm SoCs)
- Android 10+

---

## Installation

1. Open **Magisk** → Modules → Install from storage
2. Select `DisableRamdumpMagisk.zip`
3. Reboot

---

## Verifying it worked

After first boot, check the log:

```sh
adb shell su -c "cat /data/adb/disable-ramdump.log"
```

You should see something like:

```
2026-05-08 09:31:04 === disable-ramdump started ===
2026-05-08 09:31:04 setprop persist.vendor.ramdump.enable 0
2026-05-08 09:31:05   removed: /data/vendor/ramdump/mdm_modem_mem.elf
2026-05-08 09:31:05   removed: /data/vendor/ramdump/DDRCS0.BIN
2026-05-08 09:31:05 Cleanup done. Auto-removed: 2 file(s).
2026-05-08 09:31:05 chmod 000 + chattr +i applied to /data/vendor/ramdump
2026-05-08 09:31:05 === disable-ramdump done ===
```

If you see any `UNKNOWN` lines — those files were **not** auto-deleted. Review them manually before deciding to remove them.

---

## File structure

```
DisableRamdumpMagisk.zip
├── module.prop               # Module metadata and version
├── post-fs-data.sh           # Copies script into service.d at early boot
├── system/
│   └── bin/
│       └── disable-ramdump.sh   # Main script (Magisk installs to /system/bin/)
└── service.d/
    └── disable-ramdump.sh       # Redundant copy for persistence
```

---

## Safety notes

| Concern | Status |
|---|---|
| Bootloop risk | ✅ None — script runs after Android is fully up (`service.d` stage) |
| System partition touched | ✅ No — only `/data/` is ever modified |
| Directory itself deleted | ✅ No — only files *inside* it are removed |
| Unknown files auto-deleted | ✅ No — logged only, you review manually |
| Already-immutable directory | ✅ Handled — `chattr -i` is lifted before cleanup, re-applied after |

### Known limitation
`chattr +i` prevents any process from writing to `/data/vendor/ramdump` — including Qualcomm's SSR subsystem. If your OxygenOS build has a vendor daemon that requires write access to that path at boot, it will fail silently. The phone will still boot normally; that specific vendor service may not function. This is the same behaviour as the original v1.0.

---

## Changelog

### v1.2 (versionCode 3) — 2026-05-08
- **FIXED** Removed `*.bin` from auto-delete — too generic, could match non-dump vendor files on some OxygenOS builds
- **FIXED** Replaced catch-all `find+rm` (deleted all unrecognised files) with log-only pass — unknown files are now listed in the log but never auto-deleted

### v1.1 (versionCode 2) — 2026-05-08
- **ADDED** Cleanup of existing dump files on first boot
- **ADDED** Logging to `/data/adb/disable-ramdump.log`

### v1.0 (versionCode 1) — original
- Initial release by Akiva Freund
- Disables ramdump property, locks directory with `chmod 000` + `chattr +i`

---

## Credits

Original module by **Akiva Freund**.
v1.1–v1.2 cleanup and safety improvements added May 2026.

---

*This module is in **beta**. If something unexpected appears in your log, open an issue or review the files manually before re-flashing.*
