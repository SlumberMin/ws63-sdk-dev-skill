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

## 8. How to decide whether the build really passed

Treat the build as successful only if the output contains project-level evidence such as:

- `######### Build target:ws63_liteos_app success`
- `packet success!`

Helpful additional success signals:

- `ws63-liteos-app.elf` was linked
- `ws63-liteos-app.bin` was generated
- `ws63-liteos-app_all.fwpkg` exists

Do not claim success from partial object-file compilation alone.

## 9. How to inspect build failures

When a build fails:

1. Read the first real compiler or linker error, not just the final `ninja: build stopped`.
2. Fix root causes in this order:
   - missing macro or Kconfig symbol
   - wrong include path or wrong header
   - duplicated symbol or static vs non-static mismatch
   - API mismatch against this SDK version
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

1. Connect serial terminal (115200 baud, typically)
2. Reset the board
3. Look for error codes in the early boot messages
4. Correlate with the SDK's error code documentation under `vendor/`

## 13. Burn-package expectations

The primary burnable app package is typically:

- `output/ws63/fwpkg/ws63-liteos-app/ws63-liteos-app_all.fwpkg`

If the user mentions an IDE or burn tool, confirm the package path and let their existing workflow burn it.

If the user provides a specific burn tool command, preserve that workflow instead of substituting a different artifact format.

### Multi-artifact staging

When building for multiple roles, save each build's artifacts to a staging directory before the next clean build.

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

## 15. Read-on-demand reference

Read [references/ws63-reference.md](references/ws63-reference.md) when you need:

- quick command reminders
- artifact checklist
- local reference priority
- common WS63 SDK failure patterns
