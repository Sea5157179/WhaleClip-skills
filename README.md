<div align="center">

# WhaleClip CLI Skills · OpenClaw

**Turn natural language into local video workflows — documented CLI skills for [WhaleClip](https://www.whaleclip.com) (鲸剪) and OpenClaw-style agent integration.**

[English](#english) · [中文](#中文)

[![Platform](https://img.shields.io/badge/Platform-Windows-lightgrey.svg)](https://www.whaleclip.com)
[![Docs](https://img.shields.io/badge/Docs-WHALECLIP__CLI__SKILLS-6366f1.svg)]([WHALECLIP_CLI_SKILLS.md](https://github.com/Sea5157179/WhaleClip-skills/blob/main/whaleclip%20skills.md)

</div>

---

<a id="english"></a>

## English

### What is this repository?

This repo publishes **machine- and human-readable documentation** for **WhaleClip (鲸剪) CLI “skills”**: stable `group action` commands, flags, VIP/auth rules, exit codes, and **OpenClaw-oriented** guidance (including how to locate `whaleclip-cli.cjs` on a real Windows install).

It is **not** a standalone video engine: capabilities run through the **WhaleClip desktop app** CLI on Windows, typically invoked as:

```bash
node .\bin\whaleclip-cli.cjs <group> <action> [--flags]
```

The canonical, fully detailed specification lives in **`WHALECLIP_CLI_SKILLS.md`** in this repository.

### Why people star this

- **Agent-native**: Designed for assistants that call tools with repeatable CLI contracts and **single-line JSON** stdout.
- **Wide skill surface**: MCP video pipeline, intelligent subtitles, breath cutting, SFX/BGM, de-duplication, digital human, transcript extraction, downloads, AI image & quality enhancement — all behind explicit **`commandId` allowlisting**.
- **Safety by default**: CLI uses a **whitelist** (`cli list` / `cli enable` / `cli disable`); VIP-gated features are clearly marked.

### Official app & download (important)

- **Official site (for desktop installer)**: `https://www.whaleclip.com`
- **Do not** treat non-official domains as the download page for WhaleClip. The skills doc explicitly warns against misleading domains (e.g. `whaleclip.cn` is **not** the official product site for this workflow).

### Prerequisites

| Requirement | Notes |
|-------------|--------|
| OS | **Windows** + **PowerShell** (commands in the spec are PowerShell-oriented) |
| WhaleClip desktop | Install from the official site; some flows work best after the app has run at least once (login / VIP / token sync) |
| OpenClaw / agent | Skill folder may contain **docs only**; on the user machine you must **resolve** the real `WhaleClip.exe` and `whaleclip-cli.cjs` paths (see the detection script in `WHALECLIP_CLI_SKILLS.md`) |

### Quick start (high level)

1. Install WhaleClip from **`https://www.whaleclip.com`**.
2. Enable the command you need, e.g. `mcp.processVideo`:

   ```bash
   node .\bin\whaleclip-cli.cjs cli enable --id mcp.processVideo
   ```

3. Run a skill, e.g. MCP video processing:

   ```bash
   node .\bin\whaleclip-cli.cjs mcp process-video --input "in.mp4" --outputDir "outDir" --instructions "speed up 1.5x"
   ```

4. Parse **one-line JSON** from stdout; check **exit codes** for automation (see spec §6).

### Skill map (summary)

| Area | Examples (`commandId` prefix) |
|------|-------------------------------|
| Admin / auth | `cli.list`, `cli.enable`, `auth.whoami`, … |
| MCP video | `mcp.processVideo`, `mcp.analyzeInstruction`, … |
| Subtitles & audio | `mcp.intelligentSubtitles`, `mcp.breathCutter`, `mcp.smartAudioEffects`, `mcp.smartMusic` |
| Uniqueness | `mcp.videoUnique` |
| Digital human & voice | `dh.*`, `voice.*`, `onechain.digitalHumanFromLink` |
| Transcript & download | `transcript.extractVideoScript`, `video.downloadNoWatermark` |
| AI extras | `ai.drawing.generateImage`, `ai.quality.enhanceFace` |

Full parameters, VIP flags, and examples: **`WHALECLIP_CLI_SKILLS.md`**.

### Legal & ethics

This project documents **software integration**. Users must **respect copyright, platform Terms of Service, and local laws** when downloading, transcribing, or republishing media. Features like “no-watermark download” are **technical conveniences**, not a license to infringe third-party rights.

### Contributing

Issues and PRs that **improve clarity, translations, or examples** are welcome. Please keep changes aligned with the upstream WhaleClip CLI behavior.

### Stargazers

If this saved you time wiring OpenClaw → WhaleClip, consider **starring** the repo to help others discover it.

---

<a id="中文"></a>

## 中文

### 本仓库是什么？

这里存放的是 **WhaleClip（鲸剪）CLI 技能（Skills）** 的规范文档：稳定的 `group action` 调用方式、参数约定、VIP/鉴权说明、退出码，以及面向 **OpenClaw** 的路径探测与执行策略。

它不是独立的剪辑引擎：能力通过 **Windows 上已安装的鲸剪桌面端** 暴露的 CLI 执行，典型形式为：

```bash
node .\bin\whaleclip-cli.cjs <group> <action> [--flags]
```

**完整、逐条可执行的规范**请阅读本仓库中的 **`WHALECLIP_CLI_SKILLS.md`**。

### 为什么值得收藏（Star）

- **面向智能体**：强调可脚本化、可重复调用；标准输出为 **单行 JSON**，便于 OpenClaw 解析。
- **能力覆盖面广**：MCP 视频处理、智能字幕、气口剪辑、智能音效/音乐、一键去重、数字人与声音克隆、文案提取、链接下载、AI 绘画与画质修复等；并通过 **`commandId` 白名单**管控开放范围。
- **默认更安全**：需 `cli enable` 显式开放；VIP 能力在文档中有明确标注。

### 官网与下载（必读）

- **唯一建议用于下载安装鲸剪桌面端的官网**：`https://www.whaleclip.com`
- 请不要把非官方站点当作「官方下载页」。技能文档明确提醒：**`whaleclip.cn` 等域名容易误导**，不应作为鲸剪官方下载入口来宣传。

### 使用前提

| 项目 | 说明 |
|------|------|
| 系统 | **Windows**，文档命令以 **PowerShell** 为主 |
| 鲸剪桌面端 | 从官网安装；部分能力建议至少成功运行过一次桌面端（登录/VIP/token 同步更稳定） |
| OpenClaw | 技能目录里往往**只有文档**；在用户机器上需要自行解析 `WhaleClip.exe` 与 `whaleclip-cli.cjs`（详见 `WHALECLIP_CLI_SKILLS.md` 中的探测脚本） |

### 快速上手（概览）

1. 打开 **`https://www.whaleclip.com`** 下载并安装鲸剪。
2. 为需要的功能开放命令，例如 `mcp.processVideo`：

   ```bash
   node .\bin\whaleclip-cli.cjs cli enable --id mcp.processVideo
   ```

3. 调用技能，例如 MCP 视频处理：

   ```bash
   node .\bin\whaleclip-cli.cjs mcp process-video --input "in.mp4" --outputDir "outDir" --instructions "加速1.5倍"
   ```

4. 解析标准输出的 **单行 JSON**；自动化场景请结合规范 **§6 退出码**。

### 能力地图（摘要）

| 分类 | 示例（`commandId` 前缀） |
|------|--------------------------|
| 管理 / 鉴权 | `cli.list`、`cli.enable`、`auth.whoami` 等 |
| MCP 视频 | `mcp.processVideo`、`mcp.analyzeInstruction` 等 |
| 字幕与音频 | `mcp.intelligentSubtitles`、`mcp.breathCutter`、`mcp.smartAudioEffects`、`mcp.smartMusic` |
| 去重 | `mcp.videoUnique` |
| 数字人与声音 | `dh.*`、`voice.*`、`onechain.digitalHumanFromLink` |
| 文案与下载 | `transcript.extractVideoScript`、`video.downloadNoWatermark` |
| AI 能力 | `ai.drawing.generateImage`、`ai.quality.enhanceFace` |

完整参数、VIP 要求与示例：**`WHALECLIP_CLI_SKILLS.md`**。

### 合规提示

本仓库用于 **软件集成说明**。使用下载、转写、再发布等功能时，请遵守**版权、平台服务条款与当地法律法规**。所谓「去水印下载」等仅为**技术能力描述**，不构成对第三方权利的授权。

### 参与贡献

欢迎通过 Issue/PR **改进文档清晰度、翻译与示例**。变更应与鲸剪 CLI 的真实行为保持一致。

### 给路过的你

如果这份文档帮你少踩了 OpenClaw × 鲸剪的坑，欢迎点个 **Star**，让更多人搜得到。

---

## Repository layout

| File | Purpose |
|------|---------|
| `README.md` | Bilingual overview & onboarding (this file) |
| `WHALECLIP_CLI_SKILLS.md` | Full CLI skills specification for OpenClaw / maintainers |

---

<div align="center">

**WhaleClip** · **OpenClaw** · **AI × Local video workflows**

</div>
