### ✅ Version 1.1 — What changed

**Version bump:** `1.0 → 1.1` (versionCode `1 → 2`)

**New cleanup step** runs before the lock-down, and is carefully scoped to avoid any damage:

| Safety measure | Why |
|---|---|
| Only deletes **files inside** the directory, never the directory itself | Deleting the dir would break the `chattr +i` target |
| `chattr -i` is lifted first, then re-applied after | In case the module was already installed once |
| `find -maxdepth 1` — no recursion | Won't descend into subdirectories |
| Skips anything that isn't a plain file (`-type f`) | Won't touch symlinks, dirs, or devices |
| Everything logged to `/data/adb/disable-ramdump.log` | You can `cat` that file after first boot to see exactly what was removed |

**File patterns cleaned:** `.elf`, `.dump`, `.bin`, `.gz`, `.lzo`, `DDRCS*`, `load.cmm`, `rpmh_data*`, plus any remaining plain files via a final `find` pass.

After first boot, check the log with:
```sh
adb shell su -c "cat /data/adb/disable-ramdump.log"
```
