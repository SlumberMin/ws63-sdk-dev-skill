[English](#english) | [中文](#中文)

---

<a id="english"></a>

# ws63-sdk-dev

A reusable skill for AI coding agents (Hermes Agent, Codex, Claude Code, etc.) to develop, build, debug, and verify firmware on **Huawei/HiSilicon WS63** (RISC-V + LiteOS) SDK workspaces.

## What it covers

### 1. Build workflow

- Primary build: `python build.py ws63-liteos-app -j4`
- Clean rebuild (config changed): `python build.py -c ws63-liteos-app -j4`
- Auto-detect SDK root by locating `build.py`, `config.in`, `ws63.json`
- Multi-target workflow: build → save artifacts → reconfig → clean rebuild → build next
- Success must be verified from both stdout markers AND artifact existence

### 2. Menuconfig / Kconfig

- Config chain: `sample Kconfig → osource → parent Kconfig → config.in → .config → mconfig.h → source code`
- Agent-friendly approach: edit Kconfig files directly, clean rebuild with `-c`; only open TUI when user explicitly requests
- Step-by-step guide for adding new config symbols (define → osource → code → rebuild)
- Common pitfalls: declared-but-unused symbols, missing osource, stale mconfig.h

### 3. GPIO pin mux

- GPIO4 default mode is **SSI_CLK** (Mode0), not GPIO. Must call `uapi_pin_set_mode(S_MGPIO4, 2)`.
- GPIO5 default mode is **SSI_DATA** (Mode0), not GPIO. Must call `uapi_pin_set_mode(S_MGPIO5, 4)`.
- GPIO0–3, GPIO6–14: Mode0 is plain GPIO, direct use OK.
- Unconfigured GPIO4/5 can cause **peripheral false-triggering at power-on**.
- Always check vendor IO mux table before assigning any GPIO.
- Some GPIOs have boot-mode pull-up/down requirements.

### 4. Build failure diagnosis

- Read the **first real** compiler/linker error, not the final `ninja: build stopped`
- Single-root-cause loop: fix one error → rebuild → next error
- Fix priority: missing macro → wrong include → duplicate symbol → API mismatch → encoding issue
- After 3 failures on the same root cause: stop retrying, go back to research

### 5. Boot log diagnosis

- Flash Init Fail (0x80001341): flash communication or security key issue
- `verify_public_rootkey secure verify disable`: normal for dev boards, not in secure boot mode
- Serial terminal at 115200 baud, correlate error codes with vendor doc

### 6. Burn package

- Primary artifact: `output/ws63/fwpkg/ws63-liteos-app/ws63-liteos-app_all.fwpkg`
- `.fwpkg` is the burn artifact, NOT `.bin`
- Multi-artifact staging: save each build's fwpkg before the next clean build

### 7. Sample hygiene

- Dead file detection: find `.c` files not in CMakeLists SOURCES_LIST
- Copy-paste artifacts: wrong `@brief` comments, reused config names, hardcoded pins
- Shared protocol files: extract to common directory, use unique include guards
- Font/tool artifacts: delete `node_modules/` after generating font `.c` files
- Placeholder data: never fill missing sensor fields with unrelated values

## Methodology

### Five rules

1. **Evidence before conclusions** — Read code, build scripts, Kconfig, and bundled SDK docs before proposing structural changes. Prefer a working sibling sample over memory or generic advice.
2. **Prefer reuse over rewriting** — When something works in another sample, copy and adapt it. Do not reinvent.
3. **Solve one root cause per round** — When the build is red, fix the first real error and rebuild before making more edits. Do not batch-edit.
4. **Stop blind retries after three failures** — If the same root cause has already failed three times, stop repeating the same fix pattern. Return to research. Re-read sibling samples, local docs, and build scripts. Reframe the issue.
5. **Keep changes minimal** — Do not call work done unless there is build, artifact, log, or hardware evidence.

### Role split for large tasks

- `Scout`: collect evidence and references only
- `Builder`: edit code
- `Verifier`: inspect outputs and decide whether evidence is enough
- Only one writer should edit files at a time

### Reference priority

1. Current sample source code under `application/samples/`
2. Local build scripts and target config
3. Working sibling samples in the same SDK
4. Official SDK docs bundled in the workspace
5. Generated build outputs and logs
6. External references only if local evidence is insufficient

### No-install policy

Do not install tools or Python packages unless the user explicitly asks. Report what's missing and let the user decide.

### Output style

When finishing a task, report:
1. What changed
2. Whether the build passed
3. Which artifact to burn or inspect
4. Any remaining risk (e.g., not tested on hardware)

## Quick start

### With Hermes Agent

```bash
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

---

<a id="中文"></a>

# ws63-sdk-dev

适用于 AI 编程助手（Hermes Agent、Codex、Claude Code 等）的通用技能，用于在**华为海思 WS63**（RISC-V + LiteOS）SDK 工作区中进行固件开发、构建、调试和验证。

## 覆盖内容

### 1. 编译流程

- 主编译命令：`python build.py ws63-liteos-app -j4`
- 配置变更后 clean 重建：`python build.py -c ws63-liteos-app -j4`
- 自动检测 SDK 根目录：定位 `build.py`、`config.in`、`ws63.json`
- 多目标构建：编译 → 保存产物 → 重新配置 → clean 重建 → 编译下一个
- 成功判定：必须同时看到 stdout 标记和产物文件存在

### 2. Menuconfig / Kconfig

- 配置链路：`sample Kconfig → osource → 父级 Kconfig → config.in → .config → mconfig.h → 源码`
- Agent 优先方案：直接编辑 Kconfig 文件，`-c` clean 重建；仅在用户明确要求时才开 TUI
- 新增配置项的完整步骤（定义 → osource → 源码引用 → 重建）
- 常见坑：声明了但没用、缺 osource、mconfig.h 过期未重新生成

### 3. GPIO 引脚复用

- GPIO4 默认模式是 **SSI_CLK**（Mode0），不是 GPIO，必须调用 `uapi_pin_set_mode(S_MGPIO4, 2)`
- GPIO5 默认模式是 **SSI_DATA**（Mode0），不是 GPIO，必须调用 `uapi_pin_set_mode(S_MGPIO5, 4)`
- GPIO0–3、GPIO6–14：Mode0 就是 GPIO，可直接使用
- GPIO4/5 未配置可能导致**上电时外设误触发**
- 分配任何 GPIO 前必须查阅厂商 IO 复用表
- 部分 GPIO 有 boot 模式的上下拉要求

### 4. 编译失败诊断

- 读**第一个真正的**编译器/链接器错误，不是最后的 `ninja: build stopped`
- 单根因循环：修一个错 → 重建 → 下一个
- 修复优先级：缺宏 → include 路径错 → 重复符号 → API 不匹配 → 编码问题
- 同一根因失败 3 次后：停止盲改，回到研究阶段

### 5. 启动日志诊断

- Flash Init Fail (0x80001341)：Flash 通信故障或安全密钥问题
- `verify_public_rootkey secure verify disable`：开发板正常现象，表示未启用安全启动
- 串口波特率 115200，将错误码与厂商文档对照

### 6. 烧录包

- 主要产物：`output/ws63/fwpkg/ws63-liteos-app/ws63-liteos-app_all.fwpkg`
- 烧录用 `.fwpkg`，不是 `.bin`
- 多产物暂存：每次构建后保存 fwpkg 再做下一次 clean 重建

### 7. 工程卫生

- 死文件检测：找出不在 CMakeLists SOURCES_LIST 中的 `.c` 文件
- 复制粘贴残留：错误的 `@brief` 注释、复用的配置名、硬编码的引脚号
- 共享协议文件：抽到公共目录，使用唯一的 include guard
- 字体/工具产物：生成字体 `.c` 文件后删除 `node_modules/`
- 占位数据：不要用无关数据填充缺失的传感器字段

## 方法论

### 五条铁律

1. **先看证据再下结论** — 读代码、构建脚本、Kconfig 和 SDK 自带文档后再提方案。优先参考能跑的兄弟 sample，不要凭记忆或泛泛建议。
2. **优先复用不要重写** — 其他 sample 里有能用的，复制适配即可，不要从头造。
3. **一次只修一个根因** — 编译红了只修第一个真正的 error，rebuild 后再看下一个，不要批量编辑。
4. **同一根因失败 3 次后停止盲改** — 停止重复同样的修复模式，回到研究阶段，重读兄弟 sample、本地文档和构建脚本，重新定义问题。
5. **改动尽量小** — 没有构建成功、产物存在、日志通过或硬件验证，就不算完成。

### 大任务角色分工

- `Scout`：只收集证据和参考资料
- `Builder`：编辑代码
- `Verifier`：检查输出，判断证据是否充分
- 同一时间只有一个写手在改文件

### 参考资料优先级

1. 当前 sample 源码 `application/samples/`
2. 本地构建脚本和目标配置
3. 同 SDK 下能跑的兄弟 sample
4. SDK 自带文档
5. 构建输出和日志
6. 外部资料（本地不够才查）

### 不自动装包

发现缺依赖只报告，不自动安装，等用户决定。

### 输出规范

完成任务后报告：
1. 改了什么
2. 构建过没过
3. 哪个产物可以烧录或检查
4. 剩余风险（如未在硬件上测试）

## 快速开始

### Hermes Agent 使用

```bash
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
