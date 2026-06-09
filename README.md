[English](#english) | [中文](#中文)

---

<a id="english"></a>

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

## Quick start

### With Hermes Agent

```bash
# Copy to your Hermes skills directory
cp -r ws63-sdk-dev ~/.hermes/skills/
# Windows:
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

---

<a id="中文"></a>

# ws63-sdk-dev

适用于 AI 编程助手（Hermes Agent、Codex、Claude Code 等）的通用技能，用于在**华为海思 WS63**（RISC-V + LiteOS）SDK 工作区中进行固件开发、构建、调试和验证。

## 覆盖内容

- **编译流程** — `build.py ws63-liteos-app` 命令、clean 重建、产物验证
- **Menuconfig / Kconfig** — 配置链路、添加配置项、常见坑、agent 直接编辑 vs TUI 交互
- **GPIO 引脚复用** — WS63 GPIO4/5 默认 SSI 模式的关键规则、boot 引脚约束
- **编译失败诊断** — 单根因循环修复法，先修第一个错误再重建
- **启动日志诊断** — Flash Init Fail、安全验证信息
- **烧录包** — `.fwpkg` 产物流程、多目标构建暂存
- **工程卫生** — 死文件检测、复制粘贴残留清理、共享协议文件管理

## 快速开始

### Hermes Agent 使用

```bash
# 复制到 Hermes skills 目录
cp -r ws63-sdk-dev ~/.hermes/skills/
# Windows:
# xcopy ws63-sdk-dev %USERPROFILE%\AppData\Local\hermes\skills\ws63-sdk-dev\ /E /I
```

### Codex / Claude Code / 其他 agent 使用

将 `ws63-sdk-dev/` 文件夹放到项目的 `.agents/skills/` 目录下。

### 在提示词中调用

```
Use $ws63-sdk-dev to fix this WS63 build failure.
Use $ws63-sdk-dev to add a new sample and verify the build artifact.
```

## 目录结构

```
ws63-sdk-dev/
├── SKILL.md                    # 主文档（方法论 + 完整工作流）
└── references/
    └── ws63-reference.md       # 快速参考：命令、GPIO 表、常见失败模式
```

## 方法论

- **先看证据再下结论** — 读代码和构建脚本后再提方案
- **一次只修一个根因** — 修第一个错误，rebuild，再看下一个
- **不自动装依赖** — 发现缺失只报告，让用户决定
- **用产物验证完成** — `.fwpkg` 存在才算成功，光编译不算
