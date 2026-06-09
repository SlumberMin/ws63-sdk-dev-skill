# ws63-sdk-dev

A reusable skill for AI coding agents (Hermes Agent, Codex, Claude Code, etc.) to develop, build, debug, and verify firmware on **Huawei/HiSilicon WS63** (RISC-V + LiteOS) SDK workspaces.

## What it covers

- **Build workflow** — `build.py ws63-liteos-app` commands, clean rebuild, artifact verification
- **Menuconfig / Kconfig** — config chain, adding symbols, common pitfalls, agent-friendly file editing vs TUI
- **GPIO pin mux** — critical WS63 GPIO4/5 SSI default mode rules, boot-mode pin constraints
- **Build failure diagnosis** — single-root-cause loop, first-error-first-fix methodology
- **Boot log diagnosis** — Flash Init Fail, security verification messages
- **Burn package** — `.fwpkg` artifact workflow, multi-target staging
- **Sample hygiene** — dead file detection, copy-paste artifact cleanup, shared protocol file management

## How to use

### With Hermes Agent

```bash
# Copy to your Hermes skills directory
cp -r ws63-sdk-dev ~/.hermes/skills/
# Or on Windows:
# xcopy ws63-sdk-dev %USERPROFILE%\AppData\Local\hermes\skills\ws63-sdk-dev\ /E /I
```

### With Codex / Claude Code / other agents

Place the `ws63-sdk-dev/` folder under your project's `.agents/skills/` directory.

### In a prompt

```
Use $ws63-sdk-dev to fix this WS63 build failure.
Use $ws63-sdk-dev to add a new sample and verify the build artifact.
```

## Structure

```
ws63-sdk-dev/
├── SKILL.md                    # Main skill document (methodology + workflow)
└── references/
    └── ws63-reference.md       # Quick reference: commands, GPIO table, failure patterns
```

## Philosophy

- **Evidence before conclusions** — read code and build scripts before proposing changes
- **One root cause per round** — fix the first error, rebuild, repeat
- **No auto-install** — report missing dependencies, let the user decide
- **Verify with artifacts** — `.fwpkg` existence, not just compilation output
