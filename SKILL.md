---
name: ws63-sdk-dev
description: Develop, debug, build, and verify firmware in a Huawei or HiSilicon WS63 SDK workspace. Use when working on WS63 LiteOS app targets, menuconfig or Kconfig options, SLE or BLE or WiFi sample integration, build failures, artifact lookup, burn-package generation, or serial boot log diagnosis. Prefer this skill whenever the task involves `build.py`, `ws63-liteos-app`, `menuconfig`, `fwpkg`, serial logs, or SDK sample code under `application/samples`.
---

# WS63 SDK Development

Use this skill for portable, evidence-first WS63 SDK work.

## 1. How to invoke this skill

If another agent or user wants to use this skill explicitly, use prompts like:

- `Use $ws63-sdk-dev to fix this WS63 build failure.`
- `Use $ws63-sdk-dev to add a new WS63 sample feature and verify the build artifact.`

If the task is already clearly inside a WS63 SDK workspace, this skill can also be invoked implicitly.

## 2. Portable workspace detection

Do not hardcode machine-specific paths.

Detect the WS63 SDK root from the current workspace by locating:

- `build.py`
- `config.in`
- `build/config/target_config/ws63/ws63.json`

If multiple candidates exist, prefer the one that also contains:

- `application/samples/`
- `build/script/`
- `output/ws63/` or a place where that output directory would normally be generated

Usually the SDK root is either:

- the current repository root
- or a `src/` subdirectory inside the repository

## 3. Methodology

1. **Evidence before conclusions** — Read code, build scripts, Kconfig, and bundled SDK docs before proposing structural changes.
2. **Prefer reuse over rewriting** — Prefer a working sibling sample over memory or generic embedded advice.
3. **Solve one root cause per round** — When the build is red, fix the first real compiler or linker error and rebuild before making more edits.
4. **Stop blind retries after three failures** — If the same root cause has already failed three times, go back to research instead of forcing another patch.
5. **Keep changes minimal** — Do not call work done unless there is build, artifact, log, or hardware evidence.

If the task is large, use a simple role split:

- `Scout`: collect evidence and references only
- `Builder`: edit code
- `Verifier`: inspect outputs and decide whether evidence is enough

Only one writer should edit files at a time.

## 4. Reference priority

Prefer references in this order:

1. Current sample or app source code under `application/samples/`
2. Local build scripts and target config
3. Working sibling samples in the same SDK
4. Official SDK docs bundled in the workspace
5. Generated build outputs and logs
6. External references only if local evidence is insufficient

### Always inspect these local files first

- `build.py`
- `build/script/cmake_builder.py`
- `build/script/usr_config.py`
- `build/config/target_config/ws63/ws63.json`
- local sample `Kconfig`
- local sample `CMakeLists.txt`

### Good bundled SDK docs to consult

Prefer docs shipped inside the workspace, especially paths like:

- `application/samples/.../ws63download/`
- `application/samples/.../ws63download/_sources/software/`

## 5. What to inspect by task type

### WS63 API quirks

Read [references/ws63-api-quirks.md](references/ws63-api-quirks.md) when you need:
- RISC-V interrupt control (`LOS_IntLock` — NOT ARM `__disable_irq`)
- UART 5-parameter init (field is `baud_rate` not `baudrate`)
- ADC single-shot (`uapi_adc_manual_sample` — NOT `uapi_adc_read`)
- Socket close (`closesocket` — NOT `close` or `lwip_close`)
- GPIO mux rules (GPIO4/5 need explicit pin mode set)

### Display tasks

Inspect:

- `lv_port_disp.c`
- `lv_port_indev.c`
- `lcd_init.c`
- `lcd_display.c`
- `lv_conf.h`
- sibling demos under `application/samples/`

### Wireless tasks

Inspect:

- `sle_*`
- `ble_*`
- `wifi_*`
- `gateway_frame.*`

### Build or config tasks

Inspect:

- sample `Kconfig`
- sample `CMakeLists.txt`
- `config.in`
- generated `mconfig.h`
- `build/config/target_config/ws63/menuconfig/acore/`

### GPIO and pin assignment tasks

Inspect:

- vendor doc IO mux table (usually under `vendor/` or `doc/`, look for GPIO mux diagrams)
- sample source code for actual pin usage

**Critical WS63 GPIO mux rules:**
- GPIO4 default mode is **SSI_CLK** (Mode0), NOT GPIO. Must call `uapi_pin_set_mode(S_MGPIO4, 2)` for GPIO function.
- GPIO5 default mode is **SSI_DATA** (Mode0), NOT GPIO. Must call `uapi_pin_set_mode(S_MGPIO5, 4)` for GPIO function.
- GPIO0–GPIO3, GPIO6–GPIO14: Mode0 is plain GPIO, no mux config needed.
- GPIO4/5 left unconfigured can cause **peripheral false-triggering at power-on** due to SSI signal activity.
- **Always check the vendor IO mux table** for the definitive mapping before assigning any GPIO.
- **Boot-mode pins**: some GPIOs have pull-up/down requirements at boot. If a peripheral holds the wrong level during reset, the chip may enter download mode instead of running firmware.

### Power-on pull-up prohibition

These GPIOs **must NOT be pulled high by hardware before power-on**. A hardware pull-up on any of these pins can prevent the chip from booting normally:

- GPIO1, GPIO3, GPIO4, GPIO6, GPIO9, GPIO11

If your schematic has external pull-ups or pull-downs on these pins, remove the pull-up before power-on.

### Choose pins by peripheral (Mode selection)

When the user asks for a peripheral pinout, use this table to select the right GPIO and Mode:

**PWM (Mode1):**
- GPIO0 → PWM0
- GPIO1 → PWM1 ⚠️ no hardware pull-up before power-on
- GPIO2 → PWM2
- GPIO3 → PWM3 ⚠️ no hardware pull-up before power-on
- GPIO4 → PWM4 ⚠️ no hardware pull-up before power-on
- GPIO5 → PWM5
- GPIO6 → PWM6 ⚠️ no hardware pull-up before power-on
- GPIO7 → PWM7
- GPIO8 → PWM0
- GPIO9 → PWM1 ⚠️ no hardware pull-up before power-on
- GPIO10 → PWM2
- GPIO11 → PWM3 ⚠️ no hardware pull-up before power-on
- GPIO12 → PWM4

**I2C (Mode3):**
- UART1_TXD → I2C1_SDA
- UART1_RXD → I2C1_SCL
- UART0_TXD → I2C0_SDA
- UART0_RXD → I2C0_SCL

**SPI (default Mode0 = SSI, or Mode3/4/5/6/7 for other roles):**
- GPIO4 (Mode0) → SSI_CLK ⚠️ no hardware pull-up before power-on
- GPIO5 (Mode0) → SSI_DATA
- GPIO0 (Mode4) → SPI1_CSN
- GPIO1 (Mode3) → SPI1_IO0 ⚠️ no hardware pull-up before power-on
- GPIO3 (Mode3) → SPI1_IO1 ⚠️ no hardware pull-up before power-on
- GPIO2 (Mode3) → SPI1_IO3
- GPIO6 (Mode3) → SPI1_SCK, (Mode7) → SPI0_OUT ⚠️ no hardware pull-up before power-on
- GPIO10 (Mode3) → SPI0_CS0_N
- GPIO11 (Mode4) → SPI0_IN ⚠️ no hardware pull-up before power-on
- GPIO9 (Mode4) → SPI0_OUT ⚠️ no hardware pull-up before power-on

**Always check the vendor IO mux table for the definitive mapping.**

## 6. Official WS63 SDK build method

Run builds from the detected SDK root unless the workspace already has a verified wrapper.

**Primary build command:**
```
python build.py ws63-liteos-app -j4
```

**Clean rebuild when configuration drift is suspected:**
```
python build.py -c ws63-liteos-app -j4
```

**Open menuconfig:**
```
python build.py ws63-liteos-app menuconfig
```

**Open clean menuconfig if stale config is suspected:**
```
python build.py -c ws63-liteos-app menuconfig
```

Rules:
- Use the active Python interpreter available in the environment.
- Do not hardcode a machine-specific Python path into reusable instructions.
- Do not invent alternative build entrypoints when `build.py` exists.

### How the agent should operate build and menuconfig

The agent runs these commands via terminal, reading stdout to determine success or failure. The full cycle is:

**Step 1: Understand the target**
```
# Read the build target config
read_file: build/config/target_config/ws63/ws63.json

# Read the sample's Kconfig to know what config symbols exist
read_file: <sample_path>/Kconfig

# Read the sample's CMakeLists.txt to know what source files are compiled
read_file: <sample_path>/CMakeLists.txt
```

**Step 2: Modify config if needed**
```
# Option A: Edit Kconfig/config.in directly (preferred for agent workflow)
#   - Edit the Kconfig file to add/remove config symbols
#   - Edit config.in or .config to set values
#   - Rebuild with -c to pick up config changes

# Option B: Use menuconfig TUI (interactive, needs terminal PTY)
python build.py ws63-liteos-app menuconfig
#   - Requires kconfiglib + windows-curses on Windows
#   - Agent cannot drive the TUI reliably — prefer Option A
#   - If the user asks to "change config", edit Kconfig/.config files directly
#     and rebuild; only open menuconfig if the user explicitly wants the TUI
```

**Step 3: Build**
```
python build.py ws63-liteos-app -j4
```

**Step 4: Verify success**
```
# Check stdout for these markers:
#   "######### Build target:ws63_liteos_app success"
#   "packet success!"

# Then verify artifacts exist:
ls output/ws63/acore/ws63-liteos-app/ws63-liteos-app.elf
ls output/ws63/fwpkg/ws63-liteos-app/ws63-liteos-app_all.fwpkg
```

**Step 5: On failure, diagnose**
```
# Read the FIRST real compiler/linker error, not the final "ninja: build stopped"
# Search build output or build.log:
grep -n "error:\|undefined\|multiple definition\|undeclared" build.log
```

### Multi-target build workflow

When building for multiple configurations in sequence:
1. Build the first target, save artifacts to a staging directory before clean
2. Switch config if needed (edit Kconfig/.config or use menuconfig)
3. Clean rebuild with `-c` if config changed
4. Build the second target
5. Verify all artifacts exist

**Multi-sample workflow**: Only one `app_run()` entry point can be active per build. To switch between sample configurations:
```bash
# Edit .config directly (faster than menuconfig)
sed -i 's/CONFIG_SAMPLE_SUPPORT_OLD=y/# CONFIG_SAMPLE_SUPPORT_OLD is not set/' .config
sed -i 's/# CONFIG_SAMPLE_SUPPORT_NEW is not set/CONFIG_SAMPLE_SUPPORT_NEW=y/' .config
python build.py -c ws63-liteos-app -j4
```

Each sample has its own `xxx_main.c` with `app_run(xxx_entry)`. The build system compiles whichever `SAMPLE_SUPPORT_*` is enabled.
### Menuconfig config chain

The config flows through these files:

```
sample Kconfig          ← defines config symbols (config FOO, bool "description")
    ↓ osource
parent Kconfig          ← uses 'osource' to pull in child Kconfig
    ↓ source
config.in               ← top-level config, sources child configs
    ↓
build.py menuconfig     ← generates .config from all Kconfig files
    ↓
mconfig.h               ← auto-generated header with #define CONFIG_*
    ↓
source code             ← uses #ifdef CONFIG_FOO or IS_ENABLED(CONFIG_FOO)
```

When a config symbol is not taking effect:
1. Check if the symbol is defined in the sample's `Kconfig`
2. Check if the parent `Kconfig` has `osource` pointing to the child
3. Check if `config.in` sources the parent correctly
4. Check if `mconfig.h` was regenerated (clean rebuild with `-c`)
5. Check if the source code uses the correct `CONFIG_*` prefix

## 7. Official artifact expectations

After a successful app build, expect these relative paths from the SDK root:

- `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.elf`
- `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.map`
- `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.bin`
- `output/ws63/fwpkg/ws63-liteos-app/ws63-liteos-app_all.fwpkg`

Additional analysis artifacts may include:

- `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.lst`
- `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.mem`
- `output/ws63/acore/ws63-liteos-app/ws63-liteos-app.du`

Do not assume `.bin` is the final flashing artifact if the SDK or IDE expects `.fwpkg`.

**Why `.fwpkg` instead of `.bin`**: The `.bin` is a raw binary extracted from the ELF. The `.fwpkg` is a packaged firmware image that includes partition tables, security headers, and board-specific metadata expected by the WS63 flash tool. Always use `.fwpkg` for production flashing unless the user's toolchain explicitly requires `.bin`.

## 8. How to decide whether the build really passed

Treat the build as successful only if the output contains project-level evidence such as:

- `######### Build target:ws63_liteos_app success`
- `packet success!`

Helpful additional success signals:

- `ws63-liteos-app.elf` was linked
- `ws63-liteos-app.bin` was generated
- `ws63-liteos-app_all.fwpkg` exists

Do not claim success from partial object-file compilation alone.

## 8b. RISC-V API Pitfalls (WS63 is NOT ARM)

WS63 uses RISC-V (rv32imfc), NOT ARM. CMSIS ARM intrinsics do NOT exist.

### Interrupt control — `los_hwi.h`

| ❌ ARM (DO NOT USE) | ✅ RISC-V WS63 | Header |
|---------------------|----------------|--------|
| `__get_PRIMASK()` / `__disable_irq()` | `LOS_IntLock()` → `uint32_t` | `los_hwi.h` |
| `__set_PRIMASK(x)` | `LOS_IntRestore(x)` | `los_hwi.h` |

### UART API — 5 arguments, `.baud_rate`

```c
#include "uart.h"
uart_pin_config_t pins = { .tx_pin = 7, .rx_pin = 8, .cts_pin = PIN_NONE, .rts_pin = PIN_NONE };
uart_attr_t attr = { .baud_rate = 9600, .data_bits = UART_DATA_BIT_8, .stop_bits = UART_STOP_BIT_1, .parity = UART_PARITY_NONE };
uapi_uart_init(bus, &pins, &attr, NULL, NULL);  // 5 args!
```
**Field is `.baud_rate` (underscore), NOT `.baudrate`.**

### ADC API — `uapi_adc_manual_sample()`

```c
#include "adc.h"
uapi_adc_init(0);  // clock=0 default
uapi_adc_open_channel(ch);
int32_t raw = uapi_adc_manual_sample(ch);  // 0~4095 or <0 on error
```
**There is NO `uapi_adc_read()`. Use `uapi_adc_manual_sample()`.**

### NULL on RISC-V

Some SDK headers don't pull in `<stddef.h>`. Add it explicitly when `NULL` is undeclared.

## 8c. Config Drift Recovery

After editing Kconfig/config.in/.config, a normal rebuild may use cached generated files. If config changes don't take effect:

1. **Clean rebuild**: `python build.py -c ws63-liteos-app -j4`
2. **Verify mconfig.h was regenerated**: Check timestamps or grep for the new symbol
3. **If still stale**: Delete `output/` and `build/` caches manually, then rebuild

**Config chain verification order** (when a symbol is not taking effect):
1. Check if the symbol is defined in the sample's `Kconfig`
2. Check if the parent `Kconfig` has `osource` pointing to the child
3. Check if `config.in` sources the parent correctly
4. Check if `mconfig.h` was regenerated (clean rebuild with `-c`)
5. Check if the source code uses the correct `CONFIG_*` prefix

## 8d. Multi-Target Build Staging Convention

When building multiple firmware variants in one session (e.g. node A, then node B, then gateway):

```bash
artifacts/
├── node_a/
│   ├── ws63-liteos-app.elf
│   ├── ws63-liteos-app.bin
│   └── ws63-liteos-app_all.fwpkg
├── node_b/
│   └── ...
└── gateway/
    └── ...
```

**Workflow**:
1. Build target A
2. Copy artifacts to `artifacts/node_a/`
3. Switch config and clean rebuild
4. Copy artifacts to `artifacts/node_b/`
5. Repeat

This prevents accidental overwrites and gives you a clean rollback point.

## 9. How to inspect build failures

When a build fails:
When a build fails:
1. Read the first real compiler or linker error, not just the final `ninja: build stopped`.
2. Fix root causes in this order:
   - missing macro or Kconfig symbol
   - wrong include path or wrong header
   - duplicated symbol or static vs non-static mismatch
   - **CMakeLists.txt condition macro mismatch**: If a linker error shows many `undefined reference` errors from files that clearly exist in the source tree, check whether the `if(DEFINED CONFIG_*)` condition in CMakeLists.txt matches the current Kconfig symbol name. A renamed config symbol that wasn't updated in CMakeLists.txt will silently skip all source files inside that block — zero compile errors, only link errors.
   - **lwip socket API differences**: In lwip-based builds, standard BSD socket names may be macros or unavailable. Use `lwip/sockets.h` explicitly, and prefer `lwip_*` or `closesocket()` over bare `close()` when the error is `undefined reference to 'close'`.
   - API mismatch against this SDK version (check RISC-V pitfalls above)
   - encoding or generated-file issue
3. Rebuild and re-check the next failure.

Treat this as a single-root-cause loop, not a batch-edit loop.

If the same root cause has already failed three times:
1. Stop repeating the same fix pattern.
2. Return to research.
3. Re-read sibling samples, local docs, and build scripts.
4. Reframe the issue before editing again.

Useful failure search:
```
grep -n "error:\|warning:\|undefined\|multiple definition\|undeclared" build.log
```

If the user pasted compiler output directly, trust that first and correlate it back to source files.

## 10. How to reason about menuconfig

Do not guess whether a config exists. Confirm by reading:

- sample `Kconfig`
- parent `application/samples/.../Kconfig`
- `config.in`
- generated `mconfig.h`

Remember:

- `python build.py ws63-liteos-app menuconfig` is the standard entry
- target config files are derived from `build/config/target_config/ws63/menuconfig/acore/`
- missing modules like `kconfiglib` or `_curses` are environment problems, not firmware code problems

### Adding a new config symbol (step by step)

1. Create or edit the sample's `Kconfig`:
   ```
   menu "My Feature Configuration"

   config MY_FEATURE_ADDR
       hex "Device address"
       default 0x01
       range 0x01 0x0F

   config MY_FEATURE_PIN
       int "GPIO pin number"
       default 1

   endmenu
   ```
2. Ensure the parent `Kconfig` has:
   ```
   osource "application/samples/<sample>/Kconfig"
   ```
3. In source code, use `#ifdef CONFIG_MY_FEATURE_ADDR` or `#if IS_ENABLED(CONFIG_MY_FEATURE_ADDR)`
4. Clean rebuild with `-c` to regenerate `mconfig.h`
5. If using menuconfig TUI, the new symbol appears under the menu name

### Common Kconfig pitfalls

- **Declared but not used**: A Kconfig symbol exists but source code still has hardcoded `#define`. The config has no effect until code is changed to use `CONFIG_*`.
- **Missing osource**: Parent Kconfig does not `osource` the child, so the symbol never enters the build.
- **Stale mconfig.h**: After editing Kconfig, a normal rebuild may use cached `mconfig.h`. Always use `-c` for config changes.
- **🔴 CMakeLists.txt not updated after Kconfig rename**: When renaming `config OLD_NAME` to `config NEW_NAME`, ALL `if(DEFINED CONFIG_OLD_NAME)` checks in CMakeLists.txt must also change to `CONFIG_NEW_NAME`. If forgotten, the `if` block silently evaluates to false and ALL source files inside it are skipped from compilation — no error, just missing symbols at link time. **Verification**: after a rename, grep the CMakeLists.txt for the old name; if any hits remain, the build is broken.
- **🔴 `.config` file must match Kconfig rename**: The build config at `build/config/target_config/ws63/menuconfig/acore/ws63_liteos_app.config` must also be updated: change `CONFIG_OLD_NAME=y` to `CONFIG_NEW_NAME=y`. Without this, the Kconfig symbol defaults to `n` and the feature is disabled.
- **🔴 Renaming Kconfig symbols**: When renaming `config OLD_NAME` to `config NEW_NAME` in Kconfig, you MUST ALSO update the build config file at `build/config/target_config/ws63/menuconfig/acore/ws63_liteos_app.config`. Change `CONFIG_OLD_NAME=y` to `CONFIG_NEW_NAME=y`. If you forget, the build will silently skip the feature — no error, just missing code in the binary. The `.config` file is the source of truth for which symbols are enabled; Kconfig only defines what's *available*.

## 11. No-install policy

Do not install tools or Python packages unless the user explicitly asks.

If a dependency is missing, report:

- what command failed
- which module or tool was missing
- why it is needed
- the minimal likely remedy

Stop short of installing unless the user explicitly requests installation.

## 12. Boot log diagnosis

### Flash Init Fail (0x80001341)

This error appears in the serial boot log. Common causes:
- Flash chip communication failure (SPI timing, pin mux issue)
- Security verification enabled but keys not provisioned

### verify_public_rootkey secure verify disable

This message indicates the chip is running with security verification disabled. This is normal for development boards but means the flash is not in secure boot mode.

### How to read boot logs

**Standard serial parameters**: 115200 baud, 8 data bits, 1 stop bit, no parity (8N1), no flow control.

1. Connect serial terminal with the above parameters
2. Reset the board
3. Look for error codes in the early boot messages
4. Correlate with the SDK's error code documentation under `vendor/`

## 13. Burn-package expectations

The primary burnable app package is typically:

- `output/ws63/fwpkg/ws63-liteos-app/ws63-liteos-app_all.fwpkg`

If the user mentions an IDE or burn tool, confirm the package path and let their existing workflow burn it.

If the user provides a specific burn tool command, preserve that workflow instead of substituting a different artifact format.

**Staging convention**: Save each build's artifacts to a staging directory before the next clean build. See section 8d for directory layout.

## 14. Output style for agents using this skill

When finishing a WS63 task, report:

1. What changed
2. Whether the build passed
3. Which artifact to burn or inspect
4. Any remaining risk, such as not run on hardware yet

For larger multi-file work, also report:

- primary reference used
- first success marker seen
- whether the result is compile-verified only or also hardware-verified

Keep the summary short and evidence-based.

## 15. Companion Flutter App (BLE Control)

The WS63 project includes a Flutter mobile app at `smart_home_app/` that connects to the gateway via BLE. When updating the app, the protocol payloads in Dart MUST match the firmware's `gateway_frame.h` exactly.

### BLE GATT structure

| UUID | Role | Direction |
|------|------|-----------|
| 0xFF00 | Service | — |
| 0xFF01 | CTRL (write) | APP→Gateway |
| 0xFF02 | STATUS (notify) | Gateway→APP |
| 0xFF03 | SYNC (optional) | — |

Gateway broadcasts as `"SmartEdge_GW"`. App uses `flutter_blue_plus` + `flutter_riverpod`.

### Protocol alignment pitfalls (DO NOT repeat)

When the firmware changes, the app's `frame.dart` must be updated to match. Common mismatches discovered:

- `lightOn` payload is `[0xFF]` (restore last brightness), NOT `[lightType]`
- `lightOff` has NO payload, NOT `[lightType]`
- `colorTemp` is a single byte `[cct%]` (0=cold, 100=warm), NOT `[warm, cold]` two bytes
- `sceneExec` has NO payload, NOT `[sceneId, delayHi, delayLo]`
- `sensorReadReply` (0x46) is 10 bytes (T×2, H×2, L×2, CO2×2, TVOC×2), NOT 6
- `relayStateReport` (0x44) is 4 bytes (mask, key, irLen, irSlotMask), NOT 1
- IR learn command is `paramConfig(0xF0)` + `[0x01, slot]`, NOT `[0x01]` alone

### Verification

After updating the app, always run `flutter analyze --no-pub` from the `smart_home_app/` directory. Target: 0 errors, 0 warnings.

For detailed protocol mapping, read [references/flutter-app-ble-protocol.md](references/flutter-app-ble-protocol.md).

## 16. Project directory restructuring (inc/src pattern)

When reorganizing a flat sample into `inc/` + `src/` subdirectories:

### Critical: PUBLIC_HEADER propagation chain

`include_directories()` in a subdirectory CMakeLists.txt does **NOT** affect compilation of sources at the parent level. In the WS63 SDK, sources are compiled as part of the top-level component target (e.g. `samples`), not in the subdirectory scope.

**The fix**: export headers via `PUBLIC_HEADER` at EVERY level:

```cmake
# In leaf CMakeLists.txt (e.g. gateway_ui/)
set(PUBLIC_HEADER "${PUBLIC_HEADER}" "${CMAKE_CURRENT_SOURCE_DIR}/inc" PARENT_SCOPE)

# In middle CMakeLists.txt (e.g. mydemo/) — MUST also propagate!
set(PUBLIC_HEADER "${PUBLIC_HEADER}" PARENT_SCOPE)
```

**Pitfall**: The `mydemo/CMakeLists.txt` only had `set(SOURCES "..." PARENT_SCOPE)` and was missing the PUBLIC_HEADER line. This caused `fatal error: xxx.h: No such file or directory` for all headers in `inc/`.

### Recommended directory layout for complex samples

```
sample_name/
├── CMakeLists.txt       # Conditional compilation + PUBLIC_HEADER export
├── Kconfig              # Organized with menu/endmenu blocks
├── inc/                 # All public headers (flat includes still work)
├── src/                 # Application source
│   ├── xxx_main.c       # Entry point (app_run)
│   ├── protocol/        # Communication protocol
│   ├── ble/             # BLE-related
│   ├── sle/             # SLE-related
│   ├── wifi/            # WiFi/MQTT
│   └── lvgl/            # UI + display drivers
├── drivers/             # Board-level peripheral drivers
├── fonts/               # Font resources
├── docs/                # Documentation
└── tools/               # Build tools (lv_font_conv etc.)
```

### Verification after restructuring

```bash
# All .c files referenced in CMakeLists.txt must exist
grep 'CMAKE_CURRENT_SOURCE_DIR' CMakeLists.txt | grep -oP '/[^"]+\.c' | while read f; do
    test -f ".$f" && echo "✓ .$f" || echo "✗ MISSING .$f"
done
```

## 17. Memory optimization (WS63 — 544KB SRAM)

### SRAM budget

WS63 has 544KB total SRAM. After WiFi pkt_ram (48KB), system stacks (6KB), and preserve (256B), ~489KB is available for application + heap.

### Top memory consumers

| Consumer | Typical size | Optimization |
|----------|-------------|-------------|
| LCD frame buffer | 240×140×2 = 65.6KB | Reduce to 240×40×2 = 19.2KB (saves 46KB, more flushes but imperceptible) |
| Main task stack (LVGL) | 64KB | Reduce to 32KB (LVGL peak ~16-24KB) |
| LVGL heap (runtime) | 30-60KB | Reduce object create/destroy, use hide/show instead |
| SLE/BLE protocol stack | 20-40KB | SDK internal, not directly controllable |
| MQTT task stack | 8KB | Reduce to 6KB |
| LVGL draw layer + thread | 16KB | lv_conf.h: LV_DRAW_LAYER_SIMPLE_BUF_SIZE |

### Root cause of random resets

Not static allocation — it's **runtime heap fragmentation + peak overlap**:
- LVGL rendering allocates draw buffers + layer buffers
- SLE connection setup has SDK-internal temporary allocations
- MQTT JSON serialization uses stack space (512B × call depth)
- When all three happen simultaneously → heap exhaustion → watchdog reset

### Quick wins (3 changes, ~80KB saved)

```
lv_port_disp.c:  buf1[WIDTH * 140 * 2]  →  buf1[WIDTH * 40 * 2]   (save 46KB)
main.c:          STACK_SIZE = 0x10000    →  STACK_SIZE = 0x8000    (save 32KB)
gateway_mqtt.c:  STACK_SIZE = 0x2000     →  STACK_SIZE = 0x1800    (save 2KB)
```

### Flash optimization

Remove unused fonts: `font_cn_14.c` and `font_cn_20.c` are often not referenced but compiled. Saves ~130KB Flash. Disable LVGL built-in CJK fonts (`LV_FONT_SOURCE_HAN_SANS_SC_*_CJK=0`) if custom font_cn_* fonts are used. Saves ~200KB Flash.

For detailed memory analysis, read [references/ws63-memory-optimization.md](references/ws63-memory-optimization.md).

## 18. Embedded HTTP server

Some SDKs do **NOT** include an HTTP server module. The LWIP source may be stripped of application-layer modules.

**Viable approach**: BSD socket API is typically supported. Write a lightweight HTTP server using `socket()/bind()/listen()/accept()/send()/recv()`.

Reference implementation: Look for TCP server examples in the SDK's middleware or sample directories.

For WiFi SoftAP mode, reference the SDK's SoftAP sample (DHCP server, typical gateway `192.168.43.1`).

Memory budget for HTTP server: ~9KB SRAM (task stack + TCP buffers + parse buffer) + ~8KB Flash (embedded HTML).

## 19. Schematic-driven code adaptation

When new PCB schematics are available (PDF), use this workflow to adapt firmware pin assignments:

### Step 1: Extract pin assignments from schematic PDFs
```bash
# Convert PDF to image using pymupdf
python -c "import fitz; doc=fitz.open('schematic.pdf'); page=doc[0]; pix=page.get_pixmap(dpi=200); pix.save('schematic.png')"
```
Then use `vision_analyze` to read the PNG and extract GPIO assignments.

### Step 2: Build a pin comparison table
Create a table comparing Kconfig defaults vs schematic reality:
| Function | Kconfig Default | Schematic Actual | Status |

### Step 3: Update in order
1. **Header files** — hardcoded `#define XXX_PIN N` values
2. **Kconfig** — default values for `int` config symbols
3. **Source code** — any `CONFIG_*` macro references that changed names
4. **Pin mux logic** — shared pins need special handling

### Pin mux conflicts (GPIO shared between peripherals)
When a GPIO is shared (e.g. I2C SDA + LCD CS on same pin):
- **Option A**: Keep one peripheral permanently active (e.g. LCD CS = GPIO output LOW, always selected)
- **Option B**: Switch pin mode before/after each transaction (higher overhead)
- **Option C**: Use the peripheral that doesn't need toggling (I2C needs toggling, CS can be static → choose static CS)

### 🔴 Pitfall: NEVER trust schematic readings without user verification
PDF schematic analysis (via vision_analyze or OCR) is unreliable for pin assignments. ALWAYS present the extracted pin table to the user and get explicit confirmation before writing code. One wrong GPIO number can cause hardware damage (e.g. driving a relay on the wrong pin).

**Workflow**:
1. Extract pins from schematic PDF using vision_analyze
2. Present pin table to user in a clear format
3. Wait for user to confirm or correct
4. Only THEN update Kconfig defaults and header #defines

### 🔴 Pitfall: Appending vs overwriting files
When the user says "在后面加" (append) or "在下面加" (add below), use `patch` mode to append content. NEVER use `write_file` to "add" content — it overwrites the entire file. Read the file first to find the last line, then patch from that anchor.

### 🔴 Pitfall: Preserving Chinese comments and documentation
The user communicates in Chinese and expects Chinese comments in code and documentation. When editing code:
- Do NOT replace Chinese comments with English
- Do NOT delete Chinese documentation files
- Keep existing `/* 中文注释 */` style
- New comments should be in Chinese when the codebase uses Chinese

### Kconfig naming conventions
Replace hardware-revision-specific prefixes with generic module names:
- `LCDV427_SPI_DATA_PIN` → `LCD_SPI_DATA_PIN`
- `LCDV427_TOUCH_*` → `TOUCH_*`
- `LCDV427_SUPPORT` → `LCD_SUPPORT`

This requires updating ALL references in .c/.h files and Kconfig simultaneously. Use `grep -rl 'OLD_NAME' . | xargs sed -i 's/OLD_NAME/NEW_NAME/g'` and verify with `grep -r 'OLD_NAME' .` returning zero results.

**🔴 Three-way rename (Kconfig + .config + CMakeLists.txt)**:
1. Kconfig: `config OLD_NAME` → `config NEW_NAME`
2. `.config` file: `CONFIG_OLD_NAME=y` → `CONFIG_NEW_NAME=y`
3. **CMakeLists.txt**: `if(DEFINED CONFIG_OLD_NAME)` → `if(DEFINED CONFIG_NEW_NAME)` (ALL occurrences)

Missing step 3 causes all source files inside the `if` block to be silently excluded — multiple .c files can disappear from the build with zero compiler errors, only undefined symbols at link time.

## 20. LVGL font management pitfalls

### Enabling fonts that code references
If build fails with `'lv_font_montserrat_N' undeclared`, check `lv_conf.h`:
- `LV_FONT_MONTSERRAT_N` must be `1` (not `0`)
- Common mistake: code uses `montserrat_16` but lv_conf.h has it disabled

### Disabling unused built-in fonts to save Flash
Each enabled Montserrat font adds ~16-20KB Flash. Each CJK font adds ~100KB.
- Check which fonts the code actually uses: `grep -oh 'lv_font_montserrat_[0-9]*' *.c | sort -u`
- Disable unused ones in lv_conf.h
- If custom font files (font_cn_*.c) exist, disable `LV_FONT_SOURCE_HAN_SANS_SC_*_CJK` to save ~200KB Flash
- Remove font .c files from CMakeLists.txt that aren't referenced

## 21. Read-on-demand references

Read [references/ws63-reference.md](references/ws63-reference.md) when you need:
- quick command reminders
- artifact checklist
- common WS63 SDK failure patterns
- GPIO mux quick reference (PWM/I2C/SPI pin selection)
- power-on pull-up prohibitions

Read [references/ws63-api-quirks.md](references/ws63-api-quirks.md) when you need:
- RISC-V vs ARM API differences
- correct function signatures and field names
- common API pitfalls

Read [references/ws63-peripheral-api.md](references/ws63-peripheral-api.md) when you need:
- GPIO/I2C/UART/ADC/PWM/SPI API code patterns
- RISC-V interrupt control patterns
- Timing functions

Read [references/schematic-to-code-workflow.md](references/schematic-to-code-workflow.md) when you need:
- step-by-step process for adapting firmware to new PCB schematics
- pin assignment table format conventions
- how to extract pins from PDF schematics

Read [references/http-server-and-flutter-transport.md](references/http-server-and-flutter-transport.md) when you need:
- lightweight HTTP server implementation pattern
- mobile app protocol alignment guidelines
- BLE GATT service/characteristic patterns
