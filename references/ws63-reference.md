# WS63 Reference

## Command Baseline

Run from the SDK root:

```
python build.py ws63-liteos-app -j4            # normal build
python build.py -c ws63-liteos-app -j4         # clean rebuild (config changed)
python build.py ws63-liteos-app menuconfig      # open menuconfig TUI
python build.py -c ws63-liteos-app menuconfig   # clean menuconfig (stale config)
```

## Success Markers

Look for both in build stdout:

- `######### Build target:ws63_liteos_app success`
- `packet success!`

Do NOT claim success from partial object-file compilation alone.

## Main Artifacts

| Artifact | Path |
|----------|------|
| ELF | `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.elf` |
| MAP | `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.map` |
| BIN | `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.bin` |
| **FWPKG (burn)** | `output/ws63/fwpkg/ws63-liteos-app/ws63-liteos-app_all.fwpkg` |
| LST | `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.lst` |
| MEM | `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.mem` |
| DU | `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.du` |

**The burn artifact is `.fwpkg`, not `.bin`.**

## Good Local References

Read these first when relevant:

- `build/config/target_config/ws63/ws63.json`
- `build/script/cmake_builder.py`
- `build/script/usr_config.py`
- local sample `Kconfig` and `CMakeLists.txt`
- sibling demos under `application/samples/`
- vendor doc IO mux table (under `vendor/` or `doc/`)

## Config Chain

```
sample Kconfig          ← defines symbols (config FOO, bool "description")
    ↓ osource
parent Kconfig          ← uses osource to pull in child
    ↓ source
config.in               ← top-level config
    ↓ build.py menuconfig / build.py -c
.config                 ← selected values
    ↓
mconfig.h               ← auto-generated #define CONFIG_*
    ↓
source code             ← #ifdef CONFIG_FOO
```

## Common Failure Patterns

| Pattern | Root cause | Fix |
|---------|-----------|-----|
| Missing `CONFIG_*` macro | Sample `Kconfig` incomplete or parent missing `osource` | Add Kconfig entry + osource, clean rebuild |
| Config not taking effect | Stale `mconfig.h` after editing Kconfig | Always use `python build.py -c ...` for config changes |
| Declared but unused config | Kconfig symbol exists but code has hardcoded `#define` | Change code to use `CONFIG_*` prefix |
| Macros copied from another sample | Copied without corresponding config entries | Check sibling sample's Kconfig for required symbols |
| API mismatch against SDK version | Using APIs from a different SDK/Lib version | Check actual API in bundled headers or sibling samples |
| Duplicate symbol | Helper functions defined in more than one file | Make non-shared functions `static` |
| `.bin` treated as final artifact | SDK workflow expects `.fwpkg` | Use `ws63-liteos-app_all.fwpkg` |
| `kconfiglib` / `windows-curses` missing | Python env issue, not firmware code | Report to user, do not auto-install |
| Flash Init Fail 0x80001341 | Flash comm failure or security key issue | Check SPI timing, pin mux, key provisioning |
| Peripheral false-trigger at boot | GPIO4/5 default to SSI mode, not GPIO | Call `uapi_pin_set_mode()` in init code |
| Font/text not displaying | No font compiled for the character set | Generate font file, add to CMakeLists SOURCES |

## GPIO Mux Quick Reference

| GPIO | Mode0 (default) | GPIO mode needed | Notes |
|------|-----------------|------------------|-------|
| GPIO0 | GPIO | Mode0 | Direct use OK |
| GPIO1 | GPIO | Mode0 | Direct use OK |
| GPIO2 | GPIO | Mode0 | Direct use OK |
| GPIO3 | GPIO | Mode0 | Direct use OK |
| **GPIO4** | **SSI_CLK** | **Mode2** | **Must call uapi_pin_set_mode()** |
| **GPIO5** | **SSI_DATA** | **Mode4** | **Must call uapi_pin_set_mode()** |
| GPIO6 | GPIO | Mode0 | Direct use OK |
| GPIO7 | GPIO | Mode0 | Direct use OK |
| GPIO8 | GPIO | Mode0 | Direct use OK |
| GPIO9 | GPIO | Mode0 | Direct use OK |
| GPIO10 | GPIO | Mode0 | Direct use OK |
| GPIO11 | GPIO | Mode0 | Direct use OK |
| GPIO12 | GPIO | Mode0 | Direct use OK |
| GPIO13 | GPIO | Mode0 | Direct use OK |
| GPIO14 | GPIO | Mode0 | Direct use OK |

**Always check the vendor IO mux table for the definitive mapping.**

## Sample Hygiene Checklist

When reviewing or creating samples, check for these common issues:

### Dead source files

Find `.c` files in a sample directory that are NOT listed in its `CMakeLists.txt`:
```bash
for f in <sample_dir>/*.c; do
  base=$(basename "$f")
  grep -q "$base" <sample_dir>/CMakeLists.txt || echo "DEAD: $base"
done
```
Dead files are typically leftover from a previous approach (e.g., old display driver before switching libraries).

### Copy-paste artifacts

| What to check | Typical symptom | Fix |
|---------------|----------------|-----|
| CMakeLists `@brief` comment | Wrong project name in header | Change to actual sample name |
| Kconfig config name | Config symbol from another sample | Use sample-specific prefix |
| Hardcoded pin numbers | `#define MY_PIN 1` | Move to Kconfig with `#ifdef CONFIG_MY_PIN` fallback |
| Header guard collision | Multiple files use same `#ifndef` guard | Use unique guard per sample |

### Shared protocol files

When multiple samples share a protocol header (e.g., `gateway_frame.h`), copy-paste leads to divergent versions. Extract to a common directory and add it to each sample's include path.

### Font/tool artifacts in source tree

Font conversion tools (e.g., `lv_font_conv`) leave `node_modules/` behind. Delete after generating `.c` files. Add to `.gitignore`.

### Placeholder sensor data

When a UI field has no real sensor, do not fill it with unrelated data. Show "N/A" or use a clearly invalid sentinel value.

## Agent Workflow Checklist

- [ ] Detect SDK root (find `build.py`)
- [ ] Read relevant `Kconfig` + `CMakeLists.txt` before editing
- [ ] For config changes: edit files directly → clean rebuild (`-c`) → verify `mconfig.h`
- [ ] For menuconfig TUI: only open if user explicitly requests it (needs PTY + curses)
- [ ] Build with `python build.py ws63-liteos-app -j4`
- [ ] Check stdout for both success markers
- [ ] Verify `.elf` and `.fwpkg` artifacts exist
- [ ] Report: what changed, build result, artifact path, remaining risks
