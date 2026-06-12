<div align="center">

# 🔗 ws63-sdk-dev-skill

**WS63 SDK AI Agent 开发技能包**

让 AI 助手自动编译、看报错、修 Bug、重新编译，全程不用手动传话。

[![release](https://img.shields.io/badge/release-v1.0.0-blue?style=flat-square)](https://github.com/SlumberMin/ws63-sdk-dev-skill/releases)
[![platform](https://img.shields.io/badge/platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey?style=flat-square)]()
[![skill](https://img.shields.io/badge/type-AI%20Skill-orange?style=flat-square)]()
[![license](https://img.shields.io/badge/license-MIT-green?style=flat-square)]()

**[English](#english)** · **[中文](#中文)**

</div>

---

<a id="中文"></a>

## 🚀 这是什么

一个给 AI 编程助手用的**标准化技能文件**，加载后 AI 就能直接操作 WS63 SDK，不用你每次手把手教。

**支持的 AI 工具：** Hermes Agent · Codex · Claude Code · Gemini CLI · OpenCode

> **核心能力：AI 自动跑编译 → 自己看 stdout 报错 → 定位第一个 error → 改代码 → 重新编译 → 循环到通过。**

### 🎯 解决的痛点

以前用 AI 写 WS63 固件：

- ❌ 编译报错了，要你把错误信息复制粘贴给它
- ❌ 改完又报错，再复制一次，来回传话
- ❌ AI 不知道 `.fwpkg` 才是烧录产物，拿 `.bin` 就说完成了
- ❌ menuconfig 改了但没 clean rebuild，配置没生效
- ❌ GPIO4/5 默认是 SSI 不是 GPIO，踩坑了才知道

现在：

- ✅ AI 自己跑 `python build.py ws63-liteos-app -j4`
- ✅ 自己读 stdout，找第一个真正的编译错误
- ✅ 自己改代码，重新编译，循环到 `.fwpkg` 生成
- ✅ 同一个问题修三次没过，自动停下来翻文档
- ✅ GPIO4/5 的 SSI 坑写进了规则，不会再踩

### 📦 覆盖内容

| 模块 | 说明 |
|------|------|
| 🔨 编译流程 | `build.py` 命令、clean rebuild、多目标构建暂存 |
| ⚙️ Menuconfig | Kconfig 链路、新增配置项步骤、常见坑 |
| 📌 GPIO 复用 | GPIO4/5 SSI 规则、boot 引脚约束 |
| 🐛 失败诊断 | 单根因循环：第一个 error → 改 → 重建 |
| 📋 启动日志 | Flash Init Fail、安全验证错误码 |
| 📦 烧录包 | `.fwpkg` 产物确认、多产物暂存 |
| 🧹 工程卫生 | 死文件检测、复制粘贴残留清理 |

### 💡 方法论

```
先看证据再动手 → 一次只修一个问题 → 三次失败停下来 → 用产物证明完成
```

### ⚡ 快速开始

```bash
# Hermes Agent
cp -r ws63-sdk-dev-skill ~/.hermes/skills/

# Codex / Claude Code（放到项目目录下）
cp -r ws63-sdk-dev-skill .agents/skills/
```

提示词中使用：
```
Use $ws63-sdk-dev-skill to fix this WS63 build failure.
```

### 📁 目录结构

```
ws63-sdk-dev-skill/
├── SKILL.md                     # 主文档：方法论 + 完整工作流
└── references/
    └── ws63-reference.md        # 快速参考：命令、GPIO 表、失败模式
```

---

<a id="english"></a>

## 🚀 What is this

A standardized **AI Agent skill file** for WS63 SDK development. Load it and the AI can operate the build system directly — no manual error copy-pasting needed.

**Supported AI tools:** Hermes Agent · Codex · Claude Code · Gemini CLI · OpenCode

> **Core capability: AI runs build → reads stdout errors → locates first error → fixes code → rebuilds → loops until `.fwpkg` is generated.**

### 🎯 What it solves

Before:
- ❌ Copy-pasting compiler errors to the AI every time
- ❌ AI doesn't know `.fwpkg` is the real burn artifact
- ❌ Config changes without clean rebuild
- ❌ GPIO4/5 defaults to SSI mode, not GPIO

After:
- ✅ AI runs `python build.py ws63-liteos-app -j4` and reads output itself
- ✅ Finds the first real error, fixes it, rebuilds, loops
- ✅ Stops after 3 failures on the same root cause, goes back to research
- ✅ GPIO mux rules built into the skill

### 📦 Coverage

| Module | Description |
|--------|-------------|
| 🔨 Build | `build.py` commands, clean rebuild, multi-target staging |
| ⚙️ Menuconfig | Kconfig chain, adding symbols, common pitfalls |
| 📌 GPIO Mux | GPIO4/5 SSI rules, boot-mode pin constraints |
| 🐛 Failure Diagnosis | Single-root-cause loop: fix first error → rebuild |
| 📋 Boot Log | Flash Init Fail, security verification codes |
| 📦 Burn Package | `.fwpkg` artifact verification |
| 🧹 Hygiene | Dead file detection, copy-paste cleanup |

### 💡 Methodology

```
Evidence before conclusions → One root cause per round → Stop after 3 failures → Verify with artifacts
```

### ⚡ Quick start

```bash
# Hermes Agent
cp -r ws63-sdk-dev-skill ~/.hermes/skills/

# Codex / Claude Code (place under project directory)
cp -r ws63-sdk-dev-skill .agents/skills/
```

Use in prompts:
```
Use $ws63-sdk-dev-skill to fix this WS63 build failure.
```

### 📁 Structure

```
ws63-sdk-dev-skill/
├── SKILL.md                     # Main document: methodology + full workflow
└── references/
    └── ws63-reference.md        # Quick reference: commands, GPIO table, failure patterns
```

---

<div align="center">

**⭐ 觉得有用就点个 Star 吧！ · Star this repo if you find it useful!**

[![GitHub stars](https://img.shields.io/github/stars/SlumberMin/ws63-sdk-dev-skill?style=social)](https://github.com/SlumberMin/ws63-sdk-dev-skill/stargazers)
[![GitHub issues](https://img.shields.io/github/issues/SlumberMin/ws63-sdk-dev-skill?style=social)](https://github.com/SlumberMin/ws63-sdk-dev-skill/issues)

</div>
