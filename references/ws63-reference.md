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
| BIN (intermediate) | `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.bin` |
| LST | `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.lst` |
| MEM | `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.mem` |
| DU | `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.du` |

**The burn artifact is `.fwpkg`, not `.bin`.** The `.fwpkg` is a packaged firmware image that includes partition tables, security headers, and board-specific metadata expected by the WS63 flash tool. The `.bin` is just a raw binary extracted from the ELF.**

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
| `close()` undefined | lwip environment uses `closesocket()` | Use `closesocket()` or include `lwip/sockets.h` |
| CMakeLists macro mismatch | Kconfig symbol renamed, CMakeLists not updated | Grep CMakeLists.txt for old symbol name, update all |
| Many undefined links from existing files | CMakeLists.txt condition silently false | Check `if(DEFINED CONFIG_*)` matches current Kconfig name |
| Peripheral false-trigger at boot | GPIO4/5 default to SSI mode, not GPIO | Call `uapi_pin_set_mode()` in init code |
| Font/text not displaying | No font compiled for the character set | Generate font file, add to CMakeLists SOURCES |

## Serial Boot Log

Standard parameters for reading boot logs:
- **Baud rate**: 115200
- **Data bits**: 8
- **Stop bits**: 1
- **Parity**: None
- **Flow control**: None

Connect a serial terminal with these parameters, reset the board, and observe the early boot messages for error codes.

## GPIO Mux Quick Reference

| GPIO | Mode0 (default) | GPIO mode | Notes |
|------|-----------------|-----------|-------|
| GPIO0 | GPIO | Mode0 | PWM0 (Mode1), SPI1_CSN (Mode4) |
| **GPIO1** | GPIO | Mode0 | **PWM1 (Mode1), SPI1_IO0 (Mode3) — no hardware pull-up before power-on** |
| GPIO2 | GPIO | Mode0 | PWM2 (Mode1), SPI1_IO3 (Mode3) |
| **GPIO3** | GPIO | Mode0 | **PWM3 (Mode1), SPI1_IO1 (Mode3) — no hardware pull-up before power-on** |
| **GPIO4** | **SSI_CLK** | **Mode2** | **PWM4 (Mode1), JTAG_ENABLE (Mode5) — no hardware pull-up before power-on** |
| GPIO5 | SSI_DATA | Mode4 | PWM5 (Mode1), SPI0_IN (Mode5) |
| **GPIO6** | GPIO | Mode0 | **PWM6 (Mode1), SPI1_SCK (Mode3), SPI0_OUT (Mode7) — no hardware pull-up before power-on** |
| GPIO7 | GPIO | Mode0 | PWM7 (Mode1), SPI0_SCK (Mode3) |
| GPIO8 | GPIO | Mode0 | PWM0 (Mode1), SPI0_CS1_N (Mode3) |
| **GPIO9** | GPIO | Mode0 | **PWM1 (Mode1), SPI0_OUT (Mode4), I2S_DO (Mode5) — no hardware pull-up before power-on** |
| GPIO10 | GPIO | Mode0 | PWM2 (Mode1), SPI0_CS0_N (Mode3) |
| **GPIO11** | GPIO | Mode0 | **PWM3 (Mode1), SPI0_IN (Mode4), I2S_LRCLK (Mode5) — no hardware pull-up before power-on** |
| GPIO12 | GPIO | Mode0 | PWM4 (Mode1), I2S_DI (Mode5) |
| GPIO13 | GPIO | Mode0 | UART1_CTS (Mode1) |
| GPIO14 | GPIO | Mode0 | UART1_RTS (Mode1) |
| UART1_TXD | UART1_TXD | Mode3 | I2C1_SDA |
| UART1_RXD | UART1_RXD | Mode3 | I2C1_SCL |
| UART0_TXD | UART0_TXD | Mode3 | I2C0_SDA |
| UART0_RXD | UART0_RXD | Mode3 | I2C0_SCL |

### Power-on prohibition

**GPIO1, GPIO3, GPIO4, GPIO6, GPIO9, GPIO11 must NOT be pulled high by hardware before power-on.** External pull-ups on these pins can prevent the chip from booting. Check schematics and remove pull-ups before power-on.

### Choose pins by peripheral

| Peripheral | Pin / Mode |
|------------|-----------|
| PWM0 | GPIO0 (Mode1), GPIO8 (Mode1), GPIO12 is PWM4 |
| PWM1 | GPIO1 ⚠️, GPIO9 ⚠️ |
| PWM2 | GPIO2, GPIO10 |
| PWM3 | GPIO3 ⚠️, GPIO11 ⚠️ |
| PWM4 | GPIO4 ⚠️, GPIO12 |
| PWM5 | GPIO5 |
| PWM6 | GPIO6 ⚠️ |
| PWM7 | GPIO7 |
| I2C0_SDA | UART0_TXD (Mode3) |
| I2C0_SCL | UART0_RXD (Mode3) |
| I2C1_SDA | UART1_TXD (Mode3) |
| I2C1_SCL | UART1_RXD (Mode3) |
| SPI0_CS0_N | GPIO10 (Mode3) |
| SPI0_IN | GPIO11 ⚠️ (Mode4), GPIO5 (Mode5) |
| SPI0_OUT | GPIO9 ⚠️ (Mode4), GPIO6 ⚠️ (Mode7) |
| SPI0_SCK | GPIO7 (Mode3) |
| SPI0_CS1_N | GPIO8 (Mode3) |
| SPI1_CSN | GPIO0 (Mode4) |
| SPI1_IO0 | GPIO1 ⚠️ (Mode3) |
| SPI1_IO1 | GPIO3 ⚠️ (Mode3) |
| SPI1_IO3 | GPIO2 (Mode3) |
| SPI1_SCK | GPIO6 ⚠️ (Mode3) |

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

When multiple samples share a protocol header, copy-paste leads to divergent versions. Extract to a common directory and add it to each sample's include path.

### Tool artifacts in source tree

Build tools (e.g., font generators, code generators) leave temporary files behind. Delete after generating source files. Add tool directories to `.gitignore`.

### Placeholder sensor data

When a UI field has no real sensor, do not fill it with unrelated data. Show "N/A" or use a clearly invalid sentinel value.

## Multi-Device Connection State Machine Pattern

When implementing a coordinator/gateway that connects to multiple peripheral devices:

### State machine rules

1. **Scan only when a slot is free**: Do not start scanning if all connection slots are occupied.
2. **Do not re-scan an already-connected device**: Track connected addresses and skip them.
3. **Defer new scan requests**: If a scan is already active, queue the request instead of starting a duplicate.
4. **Auto-rescan on disconnect**: When a device disconnects and a slot frees up, restart scanning if there are pending devices.
5. **Limit connection attempts**: Try each candidate address a bounded number of times before giving up.

### Pseudo-code pattern

```c
// Connection manager state
typedef enum {
    SCAN_IDLE,
    SCAN_ACTIVE,
    CONNECTING,
    CONNECTED,
    DISCONNECTED
} conn_state_t;

// Rules:
// - SCAN_IDLE → start scan if pending_count > 0 AND free_slots > 0
// - SCAN_ACTIVE → on new device found, check if already connected, if not → CONNECTING
// - CONNECTING → on success → CONNECTED, on fail → back to SCAN_ACTIVE
// - DISCONNECTED → increment free_slots, trigger SCAN_IDLE check
```

### Common pitfalls

- **Scanning without checking free slots**: Leads to overflow when more devices than slots are present.
- **Not de-duplicating scan results**: Connecting to the same device twice wastes slots.
- **No rescan on disconnect**: If a device drops and another is waiting, the waiting device never gets connected.
- **Blocking the main loop**: Connection state machines must be non-blocking; use timers and events.

## Agent Workflow Checklist

- [ ] Detect SDK root (find `build.py`)
- [ ] Read relevant `Kconfig` + `CMakeLists.txt` before editing
- [ ] For config changes: edit files directly → clean rebuild (`-c`) → verify `mconfig.h`
- [ ] For menuconfig TUI: only open if user explicitly requests it (needs PTY + curses)
- [ ] Build with `python build.py ws63-liteos-app -j4`
- [ ] Check stdout for both success markers
- [ ] Verify `.elf` and `.fwpkg` artifacts exist
- [ ] Report: what changed, build result, artifact path, remaining risks
