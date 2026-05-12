# WhaleClip（鲸剪）CLI Skills 文档（给 OpenClaw 调用）

本文档用于**规范记录** WhaleClip（鲸剪）当前已开放的 CLI “技能（skills）”及其调用方式，方便 OpenClaw 以一致、可复制的命令行形式调用。

> 约定：文档中所有命令都以 Windows PowerShell 为主（你的环境是 win32 + PowerShell）。

---

## 0. 鲸剪桌面端、官网与用户引导（OpenClaw 必读）

- **正确官网（唯一用于下载安装鲸剪桌面端）**：`https://www.whaleclip.com`  
  请始终引导用户从该地址进入官网并下载 Windows 安装包。**不要**使用 `https://whaleclip.cn` 或其他非官方域名作为“鲸剪下载页”；该域名**不是**鲸剪官方站点，容易误导用户。

- **依赖桌面端的任务（例如视频智能字幕等 CLI 能力）**：用户必须先在本机安装并（建议）至少成功运行过一次鲸剪桌面端，CLI 才能稳定找到入口并同步登录/VIP/token。推荐对用户说明流程：
  1. 打开 **https://www.whaleclip.com** 下载并安装鲸剪；
  2. 安装完成后告知助手，由助手按本文档第 1 节中的探测逻辑自动查找主程序 **`*.exe`（常见名含 `whaleclip`，如 `鲸剪whaleclip.exe`；进程名不一定为 `WhaleClip`）**；**打包版实际调用不依赖 `whaleclip-cli.cjs`**，找到 exe 后使用 **`exe --cli …`** 即可。
  3. 若用户已知安装路径，也可直接提供安装目录，可跳过或辅助探测。

---

## 1. CLI 基本信息

- **CLI 启动器**：`bin/whaleclip-cli.cjs`
- **推荐调用方式（开发环境）**：

```bash
node .\bin\whaleclip-cli.cjs <group> <action> [--flags]
```

- **开发与源码目录直接跑 CLI（务必读）**：仓库 `package.json` 中 `main` 指向 **`dist/main`**。若只改了 `src/main` 但未执行 **`npm run build`**（或等价完整构建），`node .\bin\whaleclip-cli.cjs ...` 仍会加载**旧的已编译主进程**，新加的 `commandId` 会表现为「未知命令」。本地验证或发版前请先构建。
- **已安装 / 便携版 Windows 调用方式**：在鲸剪主程序 **`*.exe` 所在目录**执行：  
  `"<安装目录>\鲸剪whaleclip.exe" --cli <group> <action> [--flags]`  
  （主程序文件名以实际发行为准。`resources\whaleclip-cli.cmd` 为包装脚本；若便携包中 exe 不在 `resources` 同目录，请**优先使用根目录 exe + `--cli`**，避免 cmd 内写死路径找不到 exe。）
- **OpenClaw 调用打包版（与 `node …\whaleclip-cli.cjs` 等价）**：`<group> <action> [--flags]` **与开发环境完全相同**，仅把可执行文件从 `node …cjs` 换成 **主程序 exe**，并在其后紧跟 **`--cli`**。示例（PowerShell，推荐用**参数数组**避免 `mcp`、`ai-video` 等被错误解析）：

```powershell
$exe = (Get-ChildItem -LiteralPath "<安装或解压根目录>" -Filter "*.exe" | Where-Object { $_.Name -match "whaleclip" } | Select-Object -First 1).FullName
$p = Start-Process -FilePath $exe -WorkingDirectory (Split-Path $exe -Parent) -ArgumentList @(
  '--cli','mcp','intelligent-subtitles','--input','D:\in.mp4','--outputDir','D:\out'
) -Wait -PassThru -NoNewWindow -RedirectStandardOutput "$env:TEMP\wc_cli_out.txt"
Get-Content "$env:TEMP\wc_cli_out.txt" | Select-String -Pattern '^\{'  # 解析单行 JSON
```

- **重要备注（给 OpenClaw：如何找到“实际可执行的 .cjs”）**：
  - OpenClaw 当前看到的 skill 目录（如 `C:\Program Files\QClaw\resources\openclaw\config\skills\whaleclip skills\`）**只包含文档**，不会包含鲸剪桌面端的二进制/脚本文件。
  - 上面写的 `bin/whaleclip-cli.cjs` 指的是**源码仓库/开发环境**下的入口；如果你是在用户机器上找“已安装的鲸剪桌面版”的 CLI，需要从安装目录里定位。
  - **正确方案（适配不同用户/不同安装路径）**：不要假设鲸剪安装在固定目录。应按“可靠性从高到低”的顺序自动探测：
    - **A. 进程路径反查（最可靠）**：若用户已经打开鲸剪桌面端，可从正在运行的 `WhaleClip.exe` 直接拿到安装路径。
    - **B. 注册表反查（可靠）**：从 Windows 卸载信息（Uninstall）或 App Paths 查到 `WhaleClip.exe` 安装位置。
    - **C. 开始菜单快捷方式反查（可靠）**：从 `.lnk` 的 TargetPath 取到 `WhaleClip.exe`。
    - **D. 磁盘兜底搜索（最慢）**：在常见根目录递归搜 `WhaleClip.exe` 或 `whaleclip-cli.cjs`。
  - **定位到 `WhaleClip.exe` 后怎么找 CLI**：
    - Electron 打包应用通常在同目录或其下方存在 `resources\`（例如：`...\resources\app.asar` / `...\resources\app.asar.unpacked\`）。
    - CLI 入口脚本通常会在 `resources\` 的某个子目录内（命名可能是 `whaleclip-cli.cjs`，也可能是 `cli.cjs`/`main.cjs` 等），所以应在 `resources\` 下按 `*.cjs` + `*cli*` 关键字搜索。
  - **启用鲸剪（给 OpenClaw 的执行策略）**：
    - 若发现鲸剪未运行：可以先启动桌面端一次（仅用于生成/刷新 `%APPDATA%\whaleclip\cli.json`、同步 token/VIP 信息），再执行 CLI。
    - 若 CLI 入口找不到：先输出“探测报告”（找到的 exe 路径、resources 目录、找到的候选 `.cjs` 列表），方便继续排查。
  - **推荐 PowerShell 探测脚本（OpenClaw 可直接执行）**：

```powershell
Set-StrictMode -Version Latest
$ErrorActionPreference = "SilentlyContinue"

function Get-WhaleClipExeFromProcess {
  $p = Get-Process -Name "WhaleClip" -ErrorAction SilentlyContinue | Select-Object -First 1
  if ($p -and $p.Path) { return $p.Path }
  return $null
}

function Get-WhaleClipExeFromRegistry {
  $candidates = @()
  $uninstallRoots = @(
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*",
    "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*",
    "HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*"
  )
  foreach ($root in $uninstallRoots) {
    Get-ItemProperty $root | ForEach-Object {
      $dn = $_.DisplayName
      if ($dn -and ($dn -match "鲸剪|WhaleClip")) {
        if ($_.InstallLocation) { $candidates += (Join-Path $_.InstallLocation "WhaleClip.exe") }
        if ($_.DisplayIcon)     { $candidates += ($_.DisplayIcon -split ",")[0] }
      }
    }
  }
  $appPaths = @(
    "HKLM:\Software\Microsoft\Windows\CurrentVersion\App Paths\WhaleClip.exe",
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\App Paths\WhaleClip.exe"
  )
  foreach ($k in $appPaths) {
    $v = (Get-ItemProperty $k)."(default)"
    if ($v) { $candidates += $v }
  }

  $candidates = $candidates | Where-Object { $_ -and (Test-Path $_) } | Select-Object -Unique
  return ($candidates | Select-Object -First 1)
}

function Get-WhaleClipExeFromStartMenu {
  $links = @(
    Join-Path $env:APPDATA "Microsoft\Windows\Start Menu\Programs",
    Join-Path $env:ProgramData "Microsoft\Windows\Start Menu\Programs"
  ) | Where-Object { Test-Path $_ }

  $wsh = New-Object -ComObject WScript.Shell
  foreach ($dir in $links) {
    Get-ChildItem -Path $dir -Recurse -Filter "*.lnk" | ForEach-Object {
      if ($_.Name -match "鲸剪|WhaleClip") {
        $sc = $wsh.CreateShortcut($_.FullName)
        if ($sc -and $sc.TargetPath -and (Test-Path $sc.TargetPath) -and ($sc.TargetPath -match "WhaleClip\.exe$")) {
          $sc.TargetPath
        }
      }
    }
  }
}

function Get-WhaleClipExeFallbackSearch {
  $roots = @("C:\Program Files", "C:\Program Files (x86)", (Join-Path $env:LOCALAPPDATA "Programs"), $env:LOCALAPPDATA) |
    Where-Object { $_ -and (Test-Path $_) } | Select-Object -Unique
  foreach ($r in $roots) {
    $hit = Get-ChildItem -Path $r -Recurse -Force -ErrorAction SilentlyContinue -Filter "WhaleClip.exe" |
      Select-Object -First 1 -ExpandProperty FullName
    if ($hit) { return $hit }
  }
  return $null
}

function Get-WhaleClipCliCandidates([string]$whaleClipExe) {
  if (-not $whaleClipExe) { return @() }
  $exeDir = Split-Path $whaleClipExe -Parent

  # 常见 Electron 目录：exeDir\resources
  $resources = Join-Path $exeDir "resources"
  $searchRoots = @($exeDir, $resources) | Where-Object { $_ -and (Test-Path $_) } | Select-Object -Unique

  $candidates = @()
  foreach ($sr in $searchRoots) {
    $candidates += Get-ChildItem -Path $sr -Recurse -Force -ErrorAction SilentlyContinue -Filter "*.cjs" |
      Where-Object { $_.FullName -match "cli|whaleclip" } |
      Select-Object -ExpandProperty FullName
  }
  return ($candidates | Select-Object -Unique)
}

# 0) 读取 cli.json（如果存在，仅用于展示/辅助，并不假设它一定含入口路径）
$cliJson = Join-Path $env:APPDATA "whaleclip\cli.json"
$cliJsonText = if (Test-Path $cliJson) { Get-Content $cliJson -Raw } else { $null }

# 1) 找 WhaleClip.exe
$exe =
  (Get-WhaleClipExeFromProcess) ??
  (Get-WhaleClipExeFromRegistry) ??
  (Get-WhaleClipExeFromStartMenu | Select-Object -First 1) ??
  (Get-WhaleClipExeFallbackSearch)

Write-Output "WHCLIP_EXE=$exe"
Write-Output "CLI_JSON=$cliJson"
if ($cliJsonText) { Write-Output "CLI_JSON_PRESENT=1" } else { Write-Output "CLI_JSON_PRESENT=0" }

# 2) 找 CLI .cjs 候选
$cjs = Get-WhaleClipCliCandidates $exe
Write-Output "CJS_CANDIDATES_COUNT=$($cjs.Count)"
$cjs | ForEach-Object { Write-Output "CJS=$_"}

# 3) （可选）若 whaleclip 未运行且需要刷新 token/vip：启动一次桌面端
# if ($exe -and -not (Get-Process -Name "WhaleClip" -ErrorAction SilentlyContinue)) { Start-Process $exe }
```

- **工作原理**：CLI 会拉起 Electron 主进程的 `--cli` headless 模式执行命令；**不需要**先手动打开桌面端。
- **输出格式**：标准输出为**单行 JSON**（OpenClaw 可直接解析）。

示例输出：

```json
{"ok":true,"data":{"outputPath":"G:\\...\\C3_warm.mp4"}}
```

---

## 2. 前置条件与权限控制（非常重要）

### 2.1 命令白名单（开放/禁用）

鲸剪 CLI 采用白名单控制可执行命令（避免 OpenClaw 调用到未允许的内部能力）。

- **查看当前开放命令**：`cli list`
- **开放命令**：`cli enable --id <commandId>`
- **禁用命令**：`cli disable --id <commandId>`

白名单存储在 `electron-store` 的 `cli` store 中（Windows 通常落在 `%APPDATA%\\whaleclip\\cli.json`）。

### 2.2 VIP 鉴权与 token（24 小时有效）

部分命令标记为 `requiresVip: true`，CLI 会执行 VIP 校验：

- **token 来源**：你登录桌面端后会自动同步到 CLI store（有效期 24 小时）。
- **token 过期/缺失**：CLI 会返回 `ok:false` 并提示重新登录桌面端或手动设置 token。
- **VIP 过期**：会校验 `vipExpireAt`，过期将拒绝执行。

### 2.3 积分类命令（与 VIP 并行理解）

- 部分云端能力（例如 **§4.16 AI 视频生成**）在代码中 **`requiresVip: false`**，不走路径上的 VIP 预检，但仍需要 **有效 CLI token**，并在命令内部校验 **积分余额**（不足则 `ok:false` 直接返回，详见 §4.16 / §5.4）。
- 其它多数云端/智能能力仍为 **`requiresVip: true`**：需 **VIP + token** 同时满足。

---

## 3. 通用约定（OpenClaw 调用建议）

### 3.1 参数解析规则

CLI 参数采用简易解析：

- 以 `--` 开头的为 flag
- `--flag value` 形式传值
- 如果 `--flag` 后面紧跟另一个 `--xxx` 或结束，则该 flag 视为 `true`

### 3.2 PowerShell 下 JSON 参数的推荐写法

`--instructions` 支持两种形态：

1) **直接传一条文本指令**（最稳）：

```bash
node .\bin\whaleclip-cli.cjs mcp process-video --input "in.mp4" --outputDir "outDir" --instructions "加速1.5倍"
```

2) 传 JSON 数组字符串（多步）：

```bash
node .\bin\whaleclip-cli.cjs mcp process-video --input "in.mp4" --outputDir "outDir" --instructions "[\"加速1.5倍\",\"暖色调调色\"]"
```

> 若 JSON 转义在某些场景仍被 shell 干扰，优先改用“多次调用”或“单条指令”形态。

### 3.3 面向用户的回复规范（禁止暴露内部流程与技术细节）

**硬性规则**：凡通过本文档调用的鲸剪 CLI 能力（含第 4 章所列**全部** `commandId`，以及管理/鉴权类命令在用户侧的可感知反馈），OpenClaw 对**最终用户**的说明**一律**不得展开实现细节；**不因功能类别不同而放宽**（视频、音频、字幕、数字人、下载、AI 绘画、画质修复、**AI 视频生成（积分）**等均相同）。

鲸剪在本地或 CLI 返回里**可能**包含识别模型、音视频工具链、GPU、中间文件格式等**实现细节**。OpenClaw 在**对最终用户**说明时，必须遵守：

- **只传达结果与可操作信息**：是否成功、输出文件路径（或让用户在鲸剪里查看）、失败时的简明原因与建议（如登录/VIP/路径）。不要展开「内部是怎么做的」。
- **禁止主动展示**（除非用户明确追问技术实现，且你认为合适再简要回答），包括但不限于：语音识别/大模型名称（如 Whisper）、硬件加速与显卡型号、语言检测步骤、是否生成 SRT/中间临时文件、ffmpeg/FFmpeg、Electron/CLI 内部模块名、逐步流水线清单；以及 **TTS/音色引擎名**、**唇形/驱动引擎名**、**去水印/链接解析**相关链路描述、**AI 绘画/画质修复**所用模型或后端接口形态、**下载**直链与清晰度元数据的复述等——对用户只保留「已完成 / 已保存到某路径」或「失败原因 + 怎么办」。
- **禁止把应用日志或调试信息原样贴给用户**：若工具输出类似「处理流程：提取音频 → … → 烧录字幕」的长列表，应**不要复述**；改为一两句产品化说明即可。
- **管理类返回**（如 `cli list`）：除非用户是开发者且主动排查，否则不要把 `apiBaseUrl`、`allowedCommands` 等字段当「给用户看的说明」全文展示；一般用自然语言说明「某功能是否可用」即可。

**反面示例（禁止作为对用户的常规回复）**——即使用户任务等价于完成了这些事，也不要用这种「技术流水账」呈现：

```text
处理流程：

✅ 提取音频 → Whisper 大模型识别语音（NVIDIA RTX 2080 Ti GPU 加速）
✅ 检测语言：中文
✅ 自动修复字幕断句
✅ 生成 SRT 字幕文件
✅ ffmpeg 烧录字幕到视频（竖屏 1080x1920，白字黑边，底部居中）
```

**正面示例（推荐）**——各功能可把首句换成「已处理完成」「文案已导出」「视频已下载」等，但**信息量保持同一级别**（结果 + 路径 + 可选下一步），不增删技术层：

```text
智能字幕已生成，视频已保存，路径如下：
<outputPath>

如需调整字幕样式或重新生成，可以说明你的要求。
```

若失败：只说明**结果**与**可执行建议**（例如检查 VIP、重新登录鲸剪、更换输出目录），不罗列底层步骤。

---

## 4. Skills 清单（已开放/可开放的 CLI 命令）

下面按 `commandId`（白名单维度）列出技能。OpenClaw 建议以 `group action` 方式调用（CLI 内部会映射到 `commandId`）。

> **对用户可见的回复（全文适用）**：自 **§4.1** 起所列**每一项**技能，在向最终用户反馈成功/失败或进度时，**均须遵守 §3.3**，不得因技能不同而向用户暴露更多处理细节或技术名词；本章不再逐条重复，以免文档冗长。

### 4.0 commandId 与典型 `group action` 速查（总览）

> 下表为**源码中已注册**的全部 CLI 技能；是否在某一用户机器上「可执行」，还取决于 `%APPDATA%\whaleclip\cli.json` 白名单是否已 `cli enable`（**默认仅 §4.1 / §4.2 管理鉴权类**，见 §2.1）。`requiresVip` 为「是」时，执行前 CLI 还会校验 VIP + token。  
> **调用前缀对照**：开发环境为 `node .\bin\whaleclip-cli.cjs <下表相同子命令>`；**已安装/便携版**为 `"<主程序.exe>" --cli <下表相同子命令>`（子命令与参数与下表一致）。

| commandId | 典型子命令前缀（`…` 表示后接各 Skill 文档中的 flags） | requiresVip |
|-----------|------------------------------------------------------|-------------|
| `cli.list` | `cli list` | 否 |
| `cli.enable` | `cli enable --id <commandId>` | 否 |
| `cli.disable` | `cli disable --id <commandId>` | 否 |
| `auth.setToken` | `auth set-token --token <TOKEN>` | 否 |
| `auth.clearToken` | `auth clear-token` | 否 |
| `auth.whoami` | `auth whoami` | 否 |
| `dh.listCharacters` | `dh list-characters` | 否 |
| `dh.textToDigitalHuman` | `dh generate --text "…" --characterId <id> …` | 是 |
| `transcript.extractVideoScript` | `transcript extract --input …` | 是 |
| `video.downloadNoWatermark` | `video download --input <url> …` | 是 |
| `voice.listLibraries` | `voice list` | 否 |
| `voice.clone` | `voice clone --text "…" --voice <库名> …` | 是 |
| `onechain.digitalHumanFromLink` | `onechain digital-human --input <url> …` | 是 |
| `ai.drawing.generateImage` | `ai draw --prompt "…" …` | 是 |
| `ai.quality.enhanceFace` | `ai-quality enhance --input …` | 是 |
| `aiVideo.generate` | `ai-video generate --prompt "…" …`（或 `aivideo` / `aiVideo` 组名 + `generate`） | 否（需 token + 积分，见 §2.3） |
| `mcp.analyzeInstruction` | `mcp analyze --instruction "…"` | 是 |
| `mcp.intelligentEdit` | `mcp intelligent-edit …` | 是 |
| `mcp.intelligentSubtitles` | `mcp intelligent-subtitles --input …` | 是 |
| `mcp.breathCutter` | `mcp breath-cutter --input …` | 是 |
| `mcp.smartAudioEffects` | `mcp smart-audio-effects --input …` | 是 |
| `mcp.smartMusic` | `mcp smart-music --input …` | 是 |
| `mcp.videoUnique` | `mcp unique --input …` | 是 |
| `mcp.processVideo` | `mcp process-video --input … --outputDir … --instructions …` | 是 |
| `smartSlice.analyze` | `smart-slice analyze --input …` | 是 |
| `smartSlice.export` | `smart-slice export --input … --highlights … --exportMode …` | 是 |

---

## 4.1 管理类（无需 VIP）

### Skill: 列出已开放命令

- **commandId**：`cli.list`
- **requiresVip**：否
- **调用**：

```bash
node .\bin\whaleclip-cli.cjs cli list
```

- **成功返回**（示例字段）：
  - `data.allowedCommands`: string[]
  - `data.apiBaseUrl`: string

---

### Skill: 开放某个命令

- **commandId**：`cli.enable`
- **requiresVip**：否
- **参数**：
  - `--id`：要开放的 commandId（如 `mcp.processVideo`）
- **调用**：

```bash
node .\bin\whaleclip-cli.cjs cli enable --id mcp.intelligentSubtitles
```

---

### Skill: 禁用某个命令

- **commandId**：`cli.disable`
- **requiresVip**：否
- **参数**：
  - `--id`：要禁用的 commandId
- **调用**：

```bash
node .\bin\whaleclip-cli.cjs cli disable --id mcp.processVideo
```

---

## 4.2 鉴权类（无需 VIP）

### Skill: 手动设置 token（临时救急）

- **commandId**：`auth.setToken`
- **requiresVip**：否
- **参数**：
  - `--token`：Bearer token（字符串）
- **调用**：

```bash
node .\bin\whaleclip-cli.cjs auth set-token --token "<TOKEN>"
```

- **说明**：会写入 CLI store，并带 24 小时 TTL（`tokenExpiresAt`）。

---

### Skill: 清理 token

- **commandId**：`auth.clearToken`
- **requiresVip**：否
- **调用**：

```bash
node .\bin\whaleclip-cli.cjs auth clear-token
```

---

### Skill: 查看当前账号与 VIP 状态

- **commandId**：`auth.whoami`
- **requiresVip**：否
- **调用**：

```bash
node .\bin\whaleclip-cli.cjs auth whoami
```

- **返回要点**：
  - `data.user.roles`
  - `data.user.vipExpireAt`
  - `data.isVip`

---

## 4.3 MCP：视频理解/处理（需要 VIP）

### Skill: 指令解析（把自然语言变为内部结构）

- **commandId**：`mcp.analyzeInstruction`
- **requiresVip**：是
- **参数**：
  - `--instruction` 或 `--text`：指令文本
- **调用**：

```bash
node .\bin\whaleclip-cli.cjs mcp analyze --instruction "暖色调调色"
```

---

### Skill: MCP 视频处理（链式步骤）

- **commandId**：`mcp.processVideo`
- **requiresVip**：是
- **备注**：`mcp.processVideo` 是目前最通用的【MCP视频剪辑】入口，**可以覆盖绝大多数视频处理需求**，例如：变速、调色、缩放/裁剪、字幕相关、静音处理等。OpenClaw 若不确定该调用哪个具体模块，优先尝试用本命令通过 `--instructions` 组合步骤完成。
- **参数**：
  - `--input`：输入视频路径
  - `--outputDir`（或 `--outDir`）：输出目录
  - `--output`（可选）：输出完整路径（优先级高于 `--outputDir` 自动命名）
  - `--instructions`（或 `--steps`）：JSON 数组字符串，或直接一条文本指令
- **调用（单步文本，推荐）**：

```bash
node .\bin\whaleclip-cli.cjs mcp process-video --input "G:\360MoveData\Users\123\Desktop\C3.mp4" --outputDir "G:\360MoveData\Users\123\Desktop" --instructions "加速1.5倍"
```

- **调用（多步 JSON 数组）**：

```bash
node .\bin\whaleclip-cli.cjs mcp process-video --input "G:\360MoveData\Users\123\Desktop\C3.mp4" --outputDir "G:\360MoveData\Users\123\Desktop" --instructions "[\"加速1.5倍\",\"暖色调调色\"]"
```

- **注意**：
  - FFmpeg 不支持输入输出同路径；若 `--output` 指到输入文件，CLI 会自动改成 `*_output` 以避免失败。

---

## 4.4 智能字幕（需要 VIP）

### Skill: 智能字幕生成（对视频生成字幕并导出）

- **commandId**：`mcp.intelligentSubtitles`
- **requiresVip**：是
- **参数**：
  - `--input`：输入视频路径
  - `--output`（可选）：输出完整路径
  - `--outputDir`（或 `--outDir`，可选）：输出目录（不传 `--output` 时会自动命名）
  - `--instruction`（可选）：字幕生成指令（默认：`生成智能字幕`）

- **调用（推荐：输出目录自动命名）**：

```bash
node .\bin\whaleclip-cli.cjs mcp intelligent-subtitles --input "G:\360MoveData\Users\123\Desktop\C3.mp4" --outputDir "G:\360MoveData\Users\123\Desktop"
```

- **调用（指定输出文件名）**：

```bash
node .\bin\whaleclip-cli.cjs mcp intelligent-subtitles --input "G:\360MoveData\Users\123\Desktop\C3.mp4" --output "G:\360MoveData\Users\123\Desktop\C3_subtitles.mp4"
```

- **调用（自定义字幕风格/规则）**：

```bash
node .\bin\whaleclip-cli.cjs mcp intelligent-subtitles --input "G:\360MoveData\Users\123\Desktop\C3.mp4" --outputDir "G:\360MoveData\Users\123\Desktop" --instruction "生成中文智能字幕，自动断句，标点自然，白字黑边"
```

- **注意**：
  - 同样避免输出与输入同路径；若相同会自动改为 `*_subtitles`。

---

## 4.5 剪辑气口（需要 VIP）

### Skill: 剪辑气口（检测并去除静音片段）

- **commandId**：`mcp.breathCutter`
- **requiresVip**：是
- **参数**：
  - `--input`：输入视频路径
  - `--output`（可选）：输出完整路径
  - `--outputDir`（或 `--outDir`，可选）：输出目录（不传 `--output` 时会自动命名 `${原文件名}_breath${后缀}`）
  - `--threshold`（可选）：静音阈值（单位 dB，默认 `-20`，常用范围 `-15 ~ -30`）
  - `--padding`（可选）：静音段前后保留时长（单位 ms，默认 `30`）
  - `--reverse`（可选）：`true/false`，为 `true` 时“保留静音片段”（反向模式），默认 `false`

- **调用（推荐：输出目录自动命名）**：

```bash
node .\bin\whaleclip-cli.cjs mcp breath-cutter --input "G:\360MoveData\Users\123\Desktop\C3.mp4" --outputDir "G:\360MoveData\Users\123\Desktop"
```

- **调用（自定义阈值与 padding）**：

```bash
node .\bin\whaleclip-cli.cjs mcp breath-cutter --input "G:\360MoveData\Users\123\Desktop\C3.mp4" --outputDir "G:\360MoveData\Users\123\Desktop" --threshold -25 --padding 50
```

---

## 4.6 智能音效（需要 VIP）

### Skill: 智能音效（基于字幕时间点自动加音效）

- **commandId**：`mcp.smartAudioEffects`
- **requiresVip**：是
- **参数**：
  - `--input`：输入视频路径
  - `--output`（可选）：输出完整路径
  - `--outputDir`（或 `--outDir`，可选）：输出目录（不传 `--output` 时会自动命名 `${原文件名}_sfx${后缀}`）
  - `--effectType`（可选）：默认 `smart`；也可传 `normal`（走普通音效流程）

- **调用（推荐：输出目录自动命名）**：

```bash
node .\bin\whaleclip-cli.cjs mcp smart-audio-effects --input "G:\360MoveData\Users\123\Desktop\A1.mp4" --outputDir "G:\360MoveData\Users\123\Desktop"
```

---

## 4.7 智能音乐（需要 VIP）

### Skill: 智能音乐（自动随机选择 BGM 并混音）

- **commandId**：`mcp.smartMusic`
- **requiresVip**：是
- **参数**：
  - `--input`：输入视频路径
  - `--output`（可选）：输出完整路径
  - `--outputDir`（或 `--outDir`，可选）：输出目录（不传 `--output` 时会自动命名 `${原文件名}_music${后缀}`）
  - `--musicPath`（或 `--musicDir`，可选）：音乐文件夹路径；默认 `bgm`（会映射到内置资源 `assets/bgm` / `resources/bgm`）
  - `--volume`（可选）：背景音乐音量（建议 0~1，默认 0.5）

- **调用（推荐：输出目录自动命名）**：

```bash
node .\bin\whaleclip-cli.cjs mcp smart-music --input "G:\360MoveData\Users\123\Desktop\A1.mp4" --outputDir "G:\360MoveData\Users\123\Desktop" --volume 0.5
```

- **调用（指定自定义音乐目录）**：

```bash
node .\bin\whaleclip-cli.cjs mcp smart-music --input "G:\360MoveData\Users\123\Desktop\A1.mp4" --outputDir "G:\360MoveData\Users\123\Desktop" --musicDir "E:\my-bgm" --volume 0.4
```

---

## 4.8 一键去重（需要 VIP）

### Skill: 一键去重（随机效果组合实现防重复）

- **commandId**：`mcp.videoUnique`
- **requiresVip**：是
- **参数**：
  - `--input`：输入视频路径
  - `--output`（可选）：输出完整路径
  - `--outputDir`（或 `--outDir`，可选）：输出目录（不传 `--output` 时会自动命名 `${原文件名}_unique${后缀}`）
  - `--speedEnabled/--scaleEnabled/--flipEnabled/--overlayEnabled`（可选）：开关（true/false），默认都为 true
  - `--randomEffects`（或 `--random_effects`，可选）：是否启用随机效果（true/false），默认 true
  - `--speedX`（可选）：固定速度倍率（如 1.02）。不传则随机范围内微调
  - `--scaleX`（可选）：固定缩放倍率（如 1.01）。不传则随机范围内微调

- **调用（推荐：输出目录自动命名）**：

```bash
node .\bin\whaleclip-cli.cjs mcp unique --input "G:\360MoveData\Users\123\Desktop\A1.mp4" --outputDir "G:\360MoveData\Users\123\Desktop"
```

- **调用（关闭叠加/翻转，仅做轻微变速缩放）**：

```bash
node .\bin\whaleclip-cli.cjs mcp unique --input "G:\360MoveData\Users\123\Desktop\A1.mp4" --outputDir "G:\360MoveData\Users\123\Desktop" --overlayEnabled false --flipEnabled false
```

- **调用（指定固定倍速/缩放）**：

```bash
node .\bin\whaleclip-cli.cjs mcp unique --input "G:\360MoveData\Users\123\Desktop\A1.mp4" --outputDir "G:\360MoveData\Users\123\Desktop" --speedX 1.02 --scaleX 1.01
```

---

## 4.9 文生数字人（需要 VIP）

### Skill: 列出数字人角色库（供 OpenClaw 选择角色）

- **commandId**：`dh.listCharacters`
- **requiresVip**：否
- **说明**：返回角色列表（含 `id/name/defaultVoice/previewVideo`）。OpenClaw 先调用本命令拿到 `characterId`，再调用“文生数字人生成”。
- **调用**：

```bash
node .\bin\whaleclip-cli.cjs dh list-characters
```

- **成功返回要点**：
  - `data.characters[]`：
    - `id`：角色ID（用于 `--characterId`）
    - `name`：角色名称
    - `defaultVoice`：该角色匹配的音色库名称（会用于声音克隆/音色合成）
    - `previewVideo`：默认素材视频（用于数字人视频驱动）

---

### Skill: 文生数字人生成（文案 + 角色 → 数字人视频）

- **commandId**：`dh.textToDigitalHuman`
- **requiresVip**：是
- **能力描述**：用户提供文案，选择数字人角色库的角色；系统会使用该角色的 `defaultVoice` 匹配对应声音克隆音色生成配音，并将配音与角色视频进行唇形同步，输出数字人视频。
- **参数**：
  - `--text`（必填）：文案内容
  - `--characterId`（二选一）：角色ID（推荐）
  - `--characterName`（二选一）：角色名称（完全匹配）
  - `--outputDir`（或 `--outDir`，可选）：输出目录（不传 `--output` 时会自动命名）
  - `--output`（可选）：输出完整路径（会自动确保为 `.mp4`）
  - `--videoIndex`（可选）：使用角色库的第几个视频（从 0 开始，默认 0）
  - `--voiceEngine`（可选）：语音生成引擎，默认 `f5-tts`
  - `--emotion`（可选）：情绪，默认 `neutral`
  - `--lipSyncEngine`（或 `--engine`，可选）：唇形引擎，默认 `facefusion`；可传 `heygem`
  - `--enhanceFace`（可选）：是否做面部增强，默认 `true`
- **调用（推荐：先开放命令，再生成）**：

```bash
node .\bin\whaleclip-cli.cjs cli enable --id dh.textToDigitalHuman
node .\bin\whaleclip-cli.cjs dh generate --text "大家好，欢迎来到鲸剪。" --characterId "<从 dh list-characters 获取>" --outputDir "G:\360MoveData\Users\123\Desktop"
```

- **调用（按角色名称）**：

```bash
node .\bin\whaleclip-cli.cjs dh generate --text "今天我们讲解一个重点。" --characterName "小鲸" --outputDir "G:\360MoveData\Users\123\Desktop"
```

- **注意**：
  - 若角色未配置 `defaultVoice`，会直接失败并提示需要先在角色库完善配置。
  - 若角色视频素材缺失（`previewVideo`/`videos` 不存在），会失败并提示检查角色库资源目录。

---

## 4.10 提取视频文案（需要 VIP）

### Skill: 提取视频文案（本地视频或视频链接 → txt）

- **commandId**：`transcript.extractVideoScript`
- **requiresVip**：是
- **能力描述**：输入本地视频路径或视频链接，自动提取音频并使用 Whisper 转写为文本，最终保存为 `.txt` 文档。
- **参数**：
  - `--input`（必填）：本地视频路径或视频链接（`http/https`）
  - `--outputDir`（或 `--outDir`，可选）：输出目录（不传 `--output` 时会自动命名 `${baseName}_script.txt`）
  - `--output`（可选）：输出完整路径（会自动确保为 `.txt`）
  - `--lang`（可选）：语言，默认 `zh`
- **调用（本地视频）**：

```bash
node .\bin\whaleclip-cli.cjs cli enable --id transcript.extractVideoScript
node .\bin\whaleclip-cli.cjs transcript extract --input "G:\360MoveData\Users\123\Desktop\A1.mp4" --outputDir "G:\360MoveData\Users\123\Desktop"
```

- **调用（视频链接）**：

```bash
node .\bin\whaleclip-cli.cjs transcript extract --input "https://v.douyin.com/xxxxx" --outputDir "G:\360MoveData\Users\123\Desktop"
```

- **返回要点**：
  - `data.outputPath`：生成的 txt 路径
  - `data.chars`：文案字符数

---

## 4.11 视频去水印下载（需要 VIP）

### Skill: 视频去水印下载（视频链接 → 下载保存）

- **commandId**：`video.downloadNoWatermark`
- **requiresVip**：是
- **能力描述**：用户输入视频链接（或包含链接的文本），CLI 会自动解析出可下载的“无水印视频直链”，并下载保存到本地。
- **参数**：
  - `--input`（必填）：视频链接或包含链接的文本（支持常见平台链接，内部会自动提取/清洗链接）
  - `--outputDir`（或 `--outDir`，可选）：输出目录（不传 `--output` 时会按标题自动命名）
  - `--output`（可选）：输出完整路径（用于指定文件名）
- **调用（推荐：输出目录自动命名）**：

```bash
node .\bin\whaleclip-cli.cjs cli enable --id video.downloadNoWatermark
node .\bin\whaleclip-cli.cjs video download --input "https://v.douyin.com/xxxxx" --outputDir "G:\360MoveData\Users\123\Desktop"
```

- **调用（指定输出文件名）**：

```bash
node .\bin\whaleclip-cli.cjs video download --input "https://www.bilibili.com/video/BVxxxxxx/" --output "G:\360MoveData\Users\123\Desktop\download.mp4"
```

- **返回要点**：
  - `data.outputPath`：下载到本地的文件路径
  - `data.selected.quality/format/size`：选择的清晰度/格式/大小（若平台有提供）

---

## 4.12 声音克隆（需要 VIP）

### Skill: 列出本机音色库（供 OpenClaw 选择）

- **commandId**：`voice.listLibraries`
- **requiresVip**：否
- **调用**：

```bash
node .\bin\whaleclip-cli.cjs voice list
```

- **返回要点**：
  - `data.libraries[].name`：音色库名称（用于 `--voice`）

---

### Skill: 声音克隆生成音频（文案 + 音色库 → 音频文件）

- **commandId**：`voice.clone`
- **requiresVip**：是
- **参数**：
  - `--text`（必填）：文案
  - `--voice`（必填）：音色库名称（可通过 `voice list` 获取）
  - `--outputDir`（或 `--outDir`，可选）：输出目录（不传 `--output` 时自动命名）
  - `--output`（可选）：输出完整路径
  - `--format`（可选）：`wav` 或 `mp3`，默认 `wav`
  - `--voiceEngine`（可选）：默认 `f5-tts`
  - `--emotion`（可选）：默认 `neutral`
  - `--speed`（可选）：语速倍率，默认 `1.1`（建议 0.8~1.3）
- **调用（生成 wav）**：

```bash
node .\bin\whaleclip-cli.cjs cli enable --id voice.clone
node .\bin\whaleclip-cli.cjs voice clone --text "今天的努力，都会有回响。" --voice "电影解说小帅" --outputDir "G:\360MoveData\Users\123\Desktop" --format wav
```

- **调用（生成 mp3）**：

```bash
node .\bin\whaleclip-cli.cjs voice clone --text "别焦虑，一切都会更好。" --voice "电影解说小帅" --outputDir "G:\360MoveData\Users\123\Desktop" --format mp3
```

---

## 4.13 一链成片（数字人）（需要 VIP）

### Skill: 一链成片生成数字人视频（视频链接 + 数字人角色 → 数字人视频）

- **commandId**：`onechain.digitalHumanFromLink`
- **requiresVip**：是
- **能力描述**：输入视频链接，系统自动解析/下载视频并提取视频文案（Whisper）；然后根据你选择的数字人角色，匹配该角色的 `defaultVoice` 生成配音并做唇形同步，输出数字人视频。
- **参数**：
  - `--input`（必填）：视频链接（`http/https`）
  - `--characterId`（二选一）：数字人角色ID（推荐，通过 `dh list-characters` 获取）
  - `--characterName`（二选一）：数字人角色名称（完全匹配）
  - `--outputDir`（或 `--outDir`，可选）：输出目录（不传 `--output` 时自动命名）
  - `--output`（可选）：输出完整路径（`.mp4`）
  - `--lang`（可选）：转写语言，默认 `zh`
  - `--lipSyncEngine`（或 `--engine`，可选）：`facefusion`（默认）或 `heygem`
  - `--voiceEngine`（可选）：默认 `f5-tts`
  - `--emotion`（可选）：默认 `neutral`
- **调用（推荐）**：

```bash
node .\bin\whaleclip-cli.cjs cli enable --id onechain.digitalHumanFromLink
node .\bin\whaleclip-cli.cjs dh list-characters
node .\bin\whaleclip-cli.cjs onechain digital-human --input "https://v.douyin.com/xxxxx" --characterId "<从 dh list-characters 获取>" --outputDir "G:\360MoveData\Users\123\Desktop"
```

---

## 4.14 AI 绘画（需要 VIP）

### Skill: AI 绘画生成图片（提示词 → 图片文件）

- **commandId**：`ai.drawing.generateImage`
- **requiresVip**：是
- **能力描述**：用户提供绘画提示词，通过后端 AI 绘画接口生成图片，并下载保存到本地。
- **参数**：
  - `--prompt`（必填）：绘画提示词
  - `--outputDir`（或 `--outDir`，可选）：输出目录（不传 `--output` 时自动命名 png）
  - `--output`（可选）：输出完整路径（默认以 `.png` 保存）
  - `--size`（可选）：尺寸，例如 `1024*1024`、`1120*1440`、`1440*1120`
  - `--type`（可选）：图片类型（后端默认策略），可选 `character` / `storyboard`
- **调用**：

```bash
node .\bin\whaleclip-cli.cjs cli enable --id ai.drawing.generateImage
node .\bin\whaleclip-cli.cjs ai draw --prompt "赛博朋克风格的雨夜城市街道，霓虹灯反射在地面，超清细节" --outputDir "G:\360MoveData\Users\123\Desktop" --size "1024*1024"
```

---

## 4.15 AI 画质修复（需要 VIP）

### Skill: AI 画质修复（面部高清修复）（视频 → 视频）

- **commandId**：`ai.quality.enhanceFace`
- **requiresVip**：是
- **能力描述**：对输入视频进行面部高清修复增强，输出修复后视频（默认 `.mp4`）。
- **参数**：
  - `--input`（必填）：待修复视频路径（本地文件）
  - `--output`（可选）：输出完整路径（默认强制 `.mp4`）
  - `--outputDir`（或 `--outDir`，可选）：输出目录（不传 `--output` 时自动命名：`<原名>_enhanced.mp4`）
- **调用**：

```bash
node .\bin\whaleclip-cli.cjs cli enable --id ai.quality.enhanceFace
node .\bin\whaleclip-cli.cjs ai-quality enhance --input "G:\360MoveData\Users\123\Desktop\A1.mp4" --outputDir "G:\360MoveData\Users\123\Desktop"
```

---

## 4.16 AI 视频生成（按积分；不要求 VIP）

### Skill: AI 视频生成（提示词 → 云端生成视频，可选下载到本地）

- **commandId**：`aiVideo.generate`
- **requiresVip**：否（与桌面端「AI 视频生成」一致：以**登录 token + 积分余额**为主；仍须有效 CLI token）
- **能力描述**：调用后端 `client/ai-video` 接口提交生成任务；CLI 会先拉取公开配置中的单次消耗积分，再拉取当前账号积分；**不足则直接返回 `ok:false` 提示，不提交任务**。默认**同步等待**任务完成（约最长 10 分钟轮询）；成功且指定了输出路径时，将成品 **MP4 下载到本地**。
- **参数**：
  - `--prompt`（必填，可写 `--text` / `--p`）：视频内容描述 / 提示词
  - `--orientation`（或 `--ratio`，可选）：`portrait`（竖屏 9:16，默认）| `landscape`（横屏 16:9）| `square`（方形 1:1）
  - `--duration`（可选）：时长（秒），约 **5~60**，默认 `15`
  - `--referenceImageUrl`（或 `--refImage`，可选）：参考图 URL（须为后端可访问的地址；与桌面端「参考图」一致时传入）
  - `--outputDir`（或 `--outDir`，可选）：成功后将视频下载到该目录（自动命名 `*_ai_<时间戳>.mp4`）
  - `--output`（可选）：本地下载完整路径（`.mp4`）；与 `--outputDir` 二选一或只给其一即可
  - `--no-wait`（或 `--noWait`，可选）：为 `true` 时**仅提交任务**并立即返回 `taskId`（不轮询、不下载）
- **调用（推荐：下载到目录）**：

```bash
node .\bin\whaleclip-cli.cjs cli enable --id aiVideo.generate
node .\bin\whaleclip-cli.cjs ai-video generate --prompt "雨夜霓虹城市街道，电影感镜头" --outputDir "G:\360MoveData\Users\123\Desktop" --orientation portrait --duration 15
```

- **调用（指定输出文件）**：

```bash
node .\bin\whaleclip-cli.cjs ai-video generate --prompt "海边日落延时" --output "G:\360MoveData\Users\123\Desktop\my_ai_video.mp4" --orientation landscape --duration 20
```

- **调用（只提交、不等待）**：

```bash
node .\bin\whaleclip-cli.cjs ai-video generate --prompt "产品展示短视频" --no-wait true
```

- **成功返回要点**（`ok:true`）：
  - `data.outputPath`：已下载到本地的路径（若请求时提供了 `--output` / `--outputDir`）
  - `data.videoUrl`：云端成品地址（便于未指定本地输出时自取）
  - `data.taskId`：任务 ID
  - `data.requiredPoints` / `data.balanceBefore`：本次所需积分与提交前余额（便于对账）

- **积分不足**（`ok:false`）：
  - `message` 中会说明「需要 / 当前可用」积分；`details` 中常含 `requiredPoints`、`balance`
  - 若服务端在提交阶段才拒绝（例如并发扣减），也会将服务端 `message` 原样透出

- **注意**：
  - 须先 `cli enable --id aiVideo.generate`（默认白名单不含该命令）。
  - 与 **文生数字人**（`dh generate`）不同：本技能为 **AI 视频生成（云端文生视频）**，命令组为 `ai-video`。
  - 长时间生成可能超时；超时后任务仍可能在后台继续，请到鲸剪客户端「AI 视频生成」查看。

---

## 4.17 智能切片（需要 VIP）

与桌面端 **「AI 智能切片」** 同源：先对长视频做语音转写得到带时间轴的字幕，再智能标出金句与精彩片段；第二步根据你编辑后的片段列表导出视频（可多文件、可拼接成片，可选配乐与烧录字幕）。

### Skill: 智能切片 — 分析（转写 + 标出候选片段）

- **commandId**：`smartSlice.analyze`
- **requiresVip**：是（须有效 CLI token，校验规则同 §2.2）
- **能力描述**：对本地视频执行「字幕提取 + 智能高光标注」，标准输出返回单行 JSON；`data` 中含 `segments`、`highlights`、`usedFallback` 等字段，便于脚本保存后供 `export` 使用。
- **参数**：
  - `--input`（必填）：输入视频本地路径
  - `--outputJson`（或 `--saveJson`，可选）：将完整 `data` 结构另存为 UTF-8 JSON 文件（便于二次编辑 `highlights` 后再导出）
- **调用**：

```bash
node .\bin\whaleclip-cli.cjs cli enable --id smartSlice.analyze
node .\bin\whaleclip-cli.cjs smart-slice analyze --input "D:\media\long.mp4" --outputJson "D:\media\long_slice_analyze.json"
```

- **成功返回要点**（`ok:true`，`data` 内）：
  - `videoPath`：输入视频绝对路径
  - `segments`：字幕分段（含起止时间与文本）
  - `highlights`：候选片段列表（含 `type`：`quote` / `highlight`，`start` / `end` 秒，`title`，`reason`，默认 `keep: true`）
  - `usedFallback`：布尔；为 `true` 时表示本次为「按字幕合并」生成的候选段，仍可在 JSON 中手动删改后再导出

---

### Skill: 智能切片 — 导出（按 JSON 中的保留片段输出文件）

- **commandId**：`smartSlice.export`
- **requiresVip**：是
- **能力描述**：读取上一步保存的 JSON（或等价结构），按 `keep !== false` 的片段裁剪/拼接视频，写入 `outputDir` 或默认「视频同目录下 `原名_切片`」文件夹。
- **参数**：
  - `--input`（必填）：与分析时相同的源视频路径
  - `--highlights`（或 `--highlightsFile`，必填）：JSON 文件路径。文件内容可为 **数组**（仅 `highlights`），或与 `analyze` 输出一致的对象（含 `highlights` 字段）
  - `--exportMode`（或 `--mode`）：`sequential`（按列表顺序多文件）| `combined`（按时间拼成一个成片）| `smart_each`（多文件，文件名区分金句/精彩片段）
  - `--outputDir`（或 `--outDir`，可选）：输出目录；不传则使用桌面端相同规则（源视频旁 `原名_切片`）
  - `--addSubtitles`（可选）：`true` 时在导出片段上再识别并烧录字幕（耗时显著增加）
  - `--addAudioEnhance`（可选）：`true` 时为导出结果叠加智能音效与背景音乐（耗时增加）
  - `--orderHint`（或 `--order`，可选）：仅当 `exportMode=sequential` 时有意义；`selection`（默认，按 JSON 中片段顺序）| `chronological`（按时间排序）
- **调用示例**：

```bash
node .\bin\whaleclip-cli.cjs cli enable --id smartSlice.export
node .\bin\whaleclip-cli.cjs smart-slice export --input "D:\media\long.mp4" --highlights "D:\media\long_slice_analyze.json" --exportMode combined --outputDir "D:\media\out_slices"
```

- **编辑 highlights 的约定**：
  - 将不需要导出的条目的 `keep` 设为 `false`，或从数组中删除该项
  - 保留 `start` / `end`（秒，浮点）与可选 `type` / `title`，勿与源视频时间轴严重偏离

- **成功返回要点**（`ok:true`，`data` 内）：
  - `outputDir`：实际输出目录
  - `files`：生成的视频文件绝对路径列表

---

## 5. 典型失败与排查（OpenClaw 需要识别）

### 5.1 失败：命令未开放（白名单拦截）

- **现象**：返回 `ok:false`，包含 `message: 该命令未开放：<commandId>`
- **处理**：先执行：

```bash
node .\bin\whaleclip-cli.cjs cli enable --id <commandId>
```

---

### 5.2 失败：token 缺失或过期（24h）

- **现象**：返回 `ok:false`，`message` 提示 token 不存在或已过期
- **处理**：
  - 重新登录桌面端触发自动同步；或
  - 临时用 `auth set-token` 手动设置

---

### 5.3 失败：需要 VIP，但账号未开通/已过期

- **现象**：返回 `ok:false`，包含 `details.vipExpireAt`
- **处理**：续费/开通 VIP

---

### 5.4 失败：AI 视频生成 — 积分不足或无法拉取积分

- **现象**：`aiVideo.generate` 返回 `ok:false`，`message` 含「积分不足」或「无法获取当前积分 / 无法解析积分余额」
- **处理**：
  - 在鲸剪客户端内充值或获取积分后重试；
  - 确认已登录且 CLI token 有效（`auth whoami` / 重新登录桌面端）

---

## 6. 约定的退出码（供 OpenClaw 做分类）

CLI 最终会以进程退出码结束（OpenClaw 可用于快速判断失败类型）：

- `0`：成功（`ok:true`）
- `1`：命令执行失败（`ok:false` 或异常）
- `2`：用法错误 / 未知命令
- `3`：命令未开放（白名单拦截）
- `4`：CLI token 缺失或已过期
- `5`：需要 VIP，但账号非 VIP 或已过期
- `6`：VIP 校验失败（token 无效或网络异常）

---

## 7. 维护说明（给研发/后续扩展）

当新增一个 CLI 能力时，建议同时更新：

- `src/main/cli/commands.js`：增加 `commandId` + 参数约定 + `requiresVip`
- `src/main/cli/runner.js`：若 `group action` 不能映射为 `group.camelAction` 形式的 `commandId`，需在 `toCommandId()` 中增加显式分支
- `cli list` 白名单管理：确认是否需要默认开放（通常不默认开放）
- 本文档：新增一条 Skill（命令、参数、示例、返回）

