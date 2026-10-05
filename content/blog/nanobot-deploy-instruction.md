---
title: "nanobot 0.3.5 个人知识库自动部署指令"
date: 2026-10-05T21:17:00+08:00
slug: "nanobot-deploy-instruction"
description: "可直接复制给 nanobot 的完整部署指令：目录结构、config.json、SOUL/USER、llm-wiki Skill 与 Wiki schema 全文，含 0.3.5 会话存储位置的关键约束、验证清单与升级注意事项。"
tags: [
    "知识库",
    "nanobot",
    "提示词",
]
categories: [
    "ai-tech",
]
---

以下指令已适配 **nanobot 0.3.5** 版本，工作区路径为 `D:\知识库\`，配置目录为 `D:\nanobot-config\`。**关键修正：会话存储（sessions）已移至 workspace 之外**，避免触发 0.3.5 的路径校验错误。

> 配套阅读：[LLM-Wiki 个人知识库搭建说明](/p/llm-wiki-guide/) —— 先看它了解整体设计，再按本文逐步执行。

---

## 0.3.5版本适配说明（关键约束）

nanobot 0.3.5 对会话存储位置引入了硬性校验，**session storage 必须位于 agent workspace 之外**，否则启动时会抛出 `RuntimeError`。

校验逻辑（`nanobot/session/manager.py:551-563`）：

```python
canonical_workspace = Path(workspace).expanduser().resolve(strict=False)
root = Path(sessions_root).expanduser().resolve(strict=False)
if root == canonical_workspace or root.is_relative_to(canonical_workspace):
    raise RuntimeError("session storage must be outside the agent workspace; ...")
```

**正确的路径关系**：

| 数据项 | 存储位置 | 是否在 workspace 内 |
| :--- | :--- | :--- |
| 配置文件 | `D:\nanobot-config\config.json` | ❌ 否 |
| Workspace | `D:\知识库\` | — |
| 会话历史 | `D:\nanobot-config\sessions\<workspace-id>\` | ❌ 否 |
| 记忆 | `D:\知识库\memory\` | ✅ 是 |
| 技能 | `D:\知识库\skills\` | ✅ 是 |

**核心原则**：配置目录（含 sessions）与 workspace **平级**，sessions 不落在 workspace 内。nanobot 0.3.5 首次启动时会自动迁移 workspace 内的旧会话文件，迁移是幂等的。

---

## 给 nanobot 的完整部署指令

你是一个自动化部署代理。请在 Windows 环境下完成以下部署任务。按顺序执行，遇到错误时尝试修复并记录。所有文件使用 UTF-8 编码保存。JSON 文件中的路径使用双反斜杠 `\\`，Markdown 文件中的路径使用正斜杠 `/`。

### 任务概述

部署一个适配 nanobot 0.3.5 的 LLM Wiki 个人知识库系统，包含：

- 完整目录结构（配置目录与 workspace 分离）
- Agent 配置文件（`config.json`）
- Skill 定义文件（`llm-wiki/SKILL.md`）
- Wiki Schema（`schema.md`）
- 知识库使用手册
- Agent 权限与操作规则
- Obsidian Vault 结构

### 步骤 1：创建目录结构

```powershell
# ============================================
# 路径定义
# ============================================
$base = "D:\知识库"                    # workspace 根
$configDir = "D:\nanobot-config"           # 配置目录（在 workspace 外）
$vault = "$base\我的知识库"                 # Obsidian Vault

# ============================================
# 1. 创建配置目录（含 sessions，在 workspace 外）
# ============================================
New-Item -ItemType Directory -Force -Path `
  "$configDir\sessions", "$configDir\media" | Out-Null

Write-Host "✓ 配置目录创建完成：$configDir"
Write-Host "  - sessions/ 存储在 workspace 外，满足 nanobot 0.3.5 校验要求"

# ============================================
# 2. 创建 workspace 目录（Agent 工作区）
# ============================================
New-Item -ItemType Directory -Force -Path `
  "$base\raw\sources", "$base\raw\articles", "$base\raw\papers", `
  "$base\raw\transcripts", "$base\raw\assets", `
  "$base\memory", "$base\skills\llm-wiki" | Out-Null

Write-Host "✓ workspace 目录创建完成：$base"

# ============================================
# 3. 创建 Obsidian Vault 目录
# ============================================
$domains = @("室内设计","3D打印","艺术","经济","科技","人文","其他")
foreach ($d in $domains) {
    New-Item -ItemType Directory -Force -Path "$vault\02_Areas(领域)\$d", "$vault\03_Resources(资源)\$d" | Out-Null
}

New-Item -ItemType Directory -Force -Path `
  "$vault\00_Inbox(灵感库)", "$vault\01_Projects", `
  "$vault\04_Archive(归档)\2025", "$vault\04_Archive\2026", `
  "$vault\05_Agent(智能体)\wiki\concepts", "$vault\05_Agent\wiki\entities", `
  "$vault\05_Agent(智能体)\wiki\sources", "$vault\05_Agent\wiki\comparisons", `
  "$vault\05_Agent(智能体)\wiki\queries", `
  "$vault\05_Agent(智能体)\wiki\queries", `
  "$vault\05_Agent(智能体)\wiki\candidates\concepts", 
"$vault\05_Agent(智能体)\wiki\candidates\entities", `
  "$vault\05_Agent(智能体)\wiki\candidates\sources",
  "$vault\05_Agent(智能体)\wiki\candidates\rejected", `
  "$vault\05_Agent(智能体)\annotations", `
  "$vault\06_Toolbox(工具箱)\scripts", "$vault\06_Toolbox(工具箱)\snippets", "$vault\06_Toolbox(工具箱)\packages", `
  "$vault\MOC(内容地图)", "$vault\Templates(模板)", "$vault\Attachments(附件)" | Out-Null

Write-Host "✓ Obsidian Vault 目录创建完成：$vault"
Write-Host ""
Write-Host "最终目录结构："
Write-Host "  D:\nanobot-config\          ← 配置目录（workspace 外）"
Write-Host "  ├── config.json"
Write-Host "  ├── sessions\               ← 会话存储（workspace 外）"
Write-Host "  └── media\"
Write-Host "  D:\知识库\              ← workspace（Agent 工作区）"
Write-Host "  ├── SOUL.md"
Write-Host "  ├── USER.md"
Write-Host "  ├── raw\"
Write-Host "  ├── memory\"
Write-Host "  ├── skills\"
Write-Host "  └── 我的知识库\             ← Obsidian Vault"
```

### 步骤 2：写入 `D:\nanobot-config\config.json`

```json
{
  "agents": {
    "defaults": {
      "workspace": "D:\\知识库",
      "model": "anthropic/claude-sonnet-4-6"
    }
  },
  "gateway": {
    "port": 18791
  },
  "tools": {
    "restrictToWorkspace": true
  }
}
```

**说明**：`workspace` 字段指向 `D:\知识库`。会话历史将自动存储在配置目录 `D:\nanobot-config\sessions\<workspace-id>\` 下，该路径不在 workspace 内，满足 0.3.5 校验要求。

### 步骤 3：写入 `D:\知识库\SOUL.md`

```markdown
# SOUL

你是一个知识库管理助手，负责维护“我的知识库”。

## 工作原则
- 所有结论必须能溯源到 `raw/` 中的原始素材。
- 如果资料中没有答案，明确说“资料未覆盖”，不要编造。
- Wiki 输出到 `我的知识库/05_Agent/wiki/`。
- 候选页面输出到 `我的知识库/05_Agent/wiki/candidates/`。
- 从 `我的知识库/00_Inbox/` 读取待处理内容。
- 处理 Inbox 和生成标注索引时，**必须参考** `我的知识库/知识库使用手册.md`。
- 处理权限相关任务时，**必须参考** `我的知识库/Agent 权限与操作规则.md`。
- 用中文回答，风格简洁准确。

## 会话存储说明（nanobot 0.3.5）
- 会话历史存储在 `D:\nanobot-config\sessions\<workspace-id>\` 下，**不在 workspace 内**。
- 你无法直接读取或修改会话文件，它们由 nanobot 运行时自动管理。
- workspace 内的 `.nanobot/workspace-id` 文件由 nanobot 自动维护，不要修改它。
- 如需回顾历史对话，使用 nanobot 提供的会话查询命令，不要尝试直接读写 sessions 目录。

## 领域分类
知识库按 7 个领域组织：室内设计、3D打印、艺术、经济、科技、人文、其他。
处理任何内容时，先判断它属于哪个领域，再放入对应目录。
跨领域内容同时标注多个领域标签。

## 审核原则
- LLM 生成的 Wiki 页面一律先进入 `candidates/`，状态为 `needs_review`。
- 只有 `approved` 和 `verified` 状态的页面参与检索。
- 审核由人工完成，不要自动批准。
- 人工区（01–04）默认不修改原文件，只生成标注索引到 `05_Agent(智能体)/annotations/`。
```

### 步骤 4：写入 `D:\知识库\USER.md`

```markdown
# USER

- 知识库所有者：洋
- 主要领域：室内设计、3D打印、艺术、经济、科技、人文
- 工具链：Obsidian + nanobot + WorkBuddy

## 知识库规则
- 知识按 PARA + 领域细分组织。
- `00_Inbox(灵感库)` 是多工具协作入口，由 nanobot 自动读取和处理，
  处理后原文件移入 `raw/` 或 `04_Archive(归档)/`，保持 Inbox 为空。
- `01–04` 是人工主导区：
  - LLM 默认不修改原文件；
  - LLM 自动生成标注索引到 `05_Agent(智能体)/annotations/`，供检索使用；
  - 如需写入原文件 YAML，LLM 先建议，经我确认后再写入。
- Wiki 由 Agent 自动编译，但必须先进入 `candidates/`，经我审核后才能进入正式 Wiki。
- 如需 LLM 处理某篇手动笔记，我会主动将其放入 `00_Inbox(灵感库)/` 或
  `raw/sources/` 并明确指示。
- 所有目录职责和流转规则以 `我的知识库/知识库使用手册.md` 为准。

## nanobot 版本说明
- 当前使用 nanobot 0.3.5。
- 会话历史存储在 `D:\nanobot-config\sessions\` 下，与 workspace 分离。
- 升级时请先备份 `D:\nanobot-config\` 和 `D:\知识库\`，再停止旧进程，最后启动新版本。
```

### 步骤 5：写入 `D:\知识库\skills\llm-wiki\SKILL.md`

```markdown
---
name: llm-wiki
description: |
  管理知识库 Wiki。当用户要求摄入素材、整理知识、查询 Wiki、
  检查质量、审核候选页面、处理 Inbox 时使用。
  触发词：wiki-ingest、wiki-lint、wiki-review、inbox-process、
  摄入资料、整理知识库、审核笔记、处理收件箱。
---

# LLM Wiki 技能

## 参考文档
处理任何任务前，先读取 `我的知识库/知识库使用手册.md`，按其中的目录职责和流转规则执行。
同时读取 `我的知识库/Agent 权限与操作规则.md`，遵守权限边界。

## 路径配置

- 原始素材目录：`raw/`
- Wiki 输出目录：`我的知识库/05_Agent(智能体)/wiki/`
- 候选页面目录：`我的知识库/05_Agent(智能体)/wiki/candidates/`
- Inbox 监听目录：`我的知识库/00_Inbox(灵感库)/`
- 标注索引目录：`我的知识库/05_Agent(智能体)/annotations/`
- 记忆目录：`memory/`
- 领域目录：`我的知识库/03_Resources(资源)/{领域}/`
- 标准定义目录：`我的知识库/05_Agent(智能体)/wiki/concepts/`

## 领域检索规则

按以下优先级检索：

1. **标准定义优先**：先在 `concepts/` 下查找 `type: definition` 且 `confidence: verified` 的页面。
2. **领域限定**：判断问题领域（室内设计/3D打印/艺术/经济/科技/人文/其他），优先检索：
   - `我的知识库/03_Resources(资源)/{领域}/`
   - `我的知识库/02_Areas(领域)/{领域}/`
   - `我的知识库/MOC(内容地图)/{领域} MOC.md`
1. **扩展检索**：结果不足时扩展到 `05_Agent(智能体)/wiki/` 全库。
2. **跨领域问题**：同时检索多个领域。

候选页面（`candidates/`）不参与检索，只有 `approved` 和 `verified` 状态参与。

## /inbox-process（处理收件箱）

1. 扫描 `我的知识库/00_Inbox(灵感库)/` 下的所有文件。
2. 对每个文件：
   - 读取内容，判断类型（笔记/PDF/剪藏/待办）和领域。
   - 参照 `知识库使用手册.md` 决定去向：
     - 需要编译成 Wiki → 复制到 `raw/sources/`，触发 `/wiki-ingest`
     - 参考资料 → 移动到 `raw/` 对应子目录
     - 无法判断 → 保留，标记 `status: needs_human`
   - 原文件从 Inbox 移走。
1. 记录处理日志到 `05_Agent(智能体)/wiki/log.md`。

## /wiki-ingest（摄入与编译）

1. 扫描 `raw/sources/` 下的新文件。
2. 计算 SHA256，对比 `memory/wiki_ingest_state.json`，跳过未修改文件。
3. 对每个新文件，调用 LLM 提取实体和概念，判断领域。
4. **生成候选页面**（不直接写入正式 Wiki）：
   - 概念页 → `candidates/concepts/`
   - 实体页 → `candidates/entities/`
   - 来源摘要 → `candidates/sources/`
5. 候选页面 YAML 必须包含：`status: needs_review`、`type`、`domain`、`source`、`created`。
6. 自动添加双链（指向已有 `approved` 页面）和领域标签。
7. 更新 `05_Agent(智能体)/wiki/log.md` 和摄入状态文件。

## /wiki-review（审核辅助）

1. 列出 `candidates/` 下所有 `status: needs_review` 的页面。
2. 逐个展示页面内容、来源、LLM 置信度。
3. 你决定：批准 / 修改后批准 / 拒绝。
4. 批准后，页面移入 `05_Agent(智能体)/wiki/{type}/`，`status` 改为 `approved`，更新相关双链和 MOC。

## /wiki-lint（质量检查）

检查并报告：
- 死链、孤立页面、元数据缺失
- 矛盾信息
- 候选积压（`candidates/` 超过 20 个待审核）
- 标准定义缺口
- 标注索引与实际文件不一致

## 人工区（01–04）标签处理规则

- 默认不修改 01–04 下的原文件。
- 定期扫描 01–04，生成标注索引到 `05_Agent(智能体)/annotations/{目录名}.md`。
- 标注索引包含：文件名、领域、标签、摘要、最后分析时间、状态。
- 当用户明确要求“给某篇笔记打标签”时，先建议标签，等用户确认后再写入原文件 YAML。
- 标注索引本身自动更新，不需要用户确认。
```

### 步骤 6：写入 `D:\知识库\我的知识库\知识库使用手册.md`

创建该文件，内容为完整的《知识库目录职责说明》，包含以下章节：

- **一、总原则**：文件夹管状态，标签管主题，双链管关系
- **二、目录总览**
- **三、逐目录详细说明**：每个目录的放什么、不放什么、命名规范、元数据、流转规则、示例
- **四、笔记流转规则**：从 Inbox 到最终归档的完整流程图
- **五、元数据字段速查**：`type`、`domain`、`status`、`created`、`source`、`confidence` 等
- **六、判断速查表**：面对不同情况时该放哪个目录


### 步骤 7：写入 `D:\知识库\我的知识库\Agent 权限与操作规则.md`

创建该文件，内容包含以下章节：

- **一、进入知识库的触发条件**：可以主动进入的情况、不可以主动进入的情况
- **二、读取权限表**：各目录能否读取及条件
- **三、写入权限矩阵**：操作 / 目标目录 / 能否自动执行 / 是否需要确认
- **四、必须等用户确认的操作清单**
- **五、定时任务权限表**
- **六、其他 Agent 的权限表**：WorkBuddy、通用 nanobot、外部工具
- **七、冲突处理规则

### 步骤 8：写入 `D:\知识库\我的知识库\05_Agent\wiki\schema.md`

```markdown
# Wiki Schema

## 页面类型
- concepts/：概念、理论、方法
- entities/：人物、组织、产品、工具
- sources/：来源摘要
- comparisons/：对比分析
- queries/：查询快照

## 标准定义层（Authoritative Definitions）

### 识别规则
标准定义页面的 YAML 必须包含：
- `type: definition`
- `confidence: verified`
- `source: <权威来源>`

### 检索优先级
1. `concepts/` 下 `type: definition` 且 `confidence: verified`（最高优先级）
2. `concepts/` 下其他页面
3. `entities/`、`sources/`、`comparisons/`
4. 手动区 `03_Resources(资源)/`

### 标准定义页面模板
---
type: definition
confidence: verified
source: <教材名/官方文档/论文>
domain: <领域>
created: {{date}}
tags: []
---

# <概念名>

## 权威定义
## 我的理解
## 常见误解
## 关联

## 元数据规范

### 所有页面必须包含
- `type`
- `domain`（室内设计/3D打印/艺术/经济/科技/人文/其他）
- `created`
- `status`（needs_review/approved/rejected/verified）

### 审核状态机

| 状态 | 含义 | 谁设置 |
| :--- | :--- | :--- |
| `needs_review` | LLM 生成的草稿，待审核 | LLM 自动 |
| `approved` | 人工已批准 | 你手动 |
| `rejected` | 审核未通过 | 你手动 |
| `verified` | 已审核且标注权威来源（仅限 definition） | 你手动 |

### 审核规则
- `needs_review` 页面存放于 `candidates/`
- 批准后移动到 `concepts/` 或 `entities/`，`status` 改为 `approved`
- 只有 `approved` 和 `verified` 参与检索
- `rejected` 页面移回 `candidates/rejected/`

## 标注索引（Annotations）

### 位置
`05_Agent/annotations/{01_Projects|02_Areas|03_Resources|04_Archive}.md`

### 每条记录格式
- 文件名
- 领域
- 标签
- 摘要（一句话）
- 最后分析时间
- 状态

### 更新规则
- 每次 `/wiki-ingest` 或 `/wiki-lint` 时同步更新
- 原文件未修改时不重复分析（SHA256）
- 原文件修改后重新分析并更新条目

## 链接规则
- 每个页面至少链接 2 个其他页面
- 同领域页面互相链接
- 标准定义页面必须链接到至少 1 个 `entities/` 页面

## 矛盾处理
不同来源冲突时，创建 `comparisons/` 页面并列展示，并标注来源。
```

### 步骤 9：创建 Wiki 索引和日志

**`D:\知识库\我的知识库\05_Agent\wiki\index.md`**：
```markdown
# Wiki 索引

## 概念
（由 /wiki-ingest 自动填充）

## 实体
（由 /wiki-ingest 自动填充）

## 标准定义
（由人工审核后添加）

## 对比
（由 /wiki-ingest 自动填充）
```

**`D:\知识库\我的知识库\05_Agent\wiki\log.md`**：
```markdown
# Wiki 日志

## 2026-09-24
- 知识库初始化完成（nanobot 0.3.5）
- 创建配置目录（D:\nanobot-config\）与 workspace（D:\知识库\）
- 会话存储已配置在 workspace 外
```

### 步骤 10：创建领域 MOC 文件

为每个领域创建 MOC 文件。模板：

```markdown
# {领域} MOC

## 02_Areas(领域)
（待填充）

## 03_Resources(资源)
### 书籍
（待填充）
### 课程
（待填充）
### 文章
（待填充）

## 05_Agent(智能体)/wiki
（待填充）

## 06_Toolbox(工具箱)
（待填充）
```

需创建：室内设计、3D打印、艺术、经济、科技、人文、其他，共 7 个文件，放在 `D:\知识库\我的知识库\MOC\` 下。

### 步骤 11：创建工具箱索引

**`D:\知识库\我的知识库\06_Toolbox(工具箱)\README.md`**：
```markdown
# 工具箱索引

## scripts/
（待添加）

## snippets/
（待添加）

## packages/
（待添加）
```

### 步骤 12：创建标注索引占位文件

在 `D:\知识库\我的知识库\05_Agent(智能体)\annotations\` 下创建：

- `01_Projects.md`
- `02_Areas.md`
- `03_Resources.md`
- `04_Archive.md`

每个文件初始内容：
```markdown
# {目录名} 标注索引

（由 /wiki-lint 或 /wiki-ingest 自动填充）
```

### 步骤 13：验证

```powershell
# ============================================
# 关键文件验证
# ============================================
Write-Host "=== 配置文件验证 ==="
Test-Path "D:\nanobot-config\config.json"
Test-Path "D:\nanobot-config\sessions"

Write-Host "`n=== Agent 文件验证 ==="
Test-Path "D:\知识库\SOUL.md"
Test-Path "D:\知识库\USER.md"
Test-Path "D:\知识库\skills\llm-wiki\SKILL.md"
Test-Path "D:\知识库\memory"

Write-Host "`n=== Vault 文件验证 ==="
Test-Path "D:\知识库\我的知识库\知识库使用手册.md"
Test-Path "D:\知识库\我的知识库\Agent 权限与操作规则.md"
Test-Path "D:\知识库\我的知识库\05_Agent(智能体)\wiki\schema.md"
Test-Path "D:\知识库\我的知识库\05_Agent(智能体)\wiki\candidates"
Test-Path "D:\知识库\我的知识库\05_Agent(智能体)\annotations"

Write-Host "`n=== 路径校验（关键） ==="
$ws = [System.IO.Path]::GetFullPath("D:\知识库")
$ss = [System.IO.Path]::GetFullPath("D:\nanobot-config\sessions")
Write-Host "workspace: $ws"
Write-Host "sessions:  $ss"
if ($ss.StartsWith($ws)) {
    Write-Host "❌ 错误：sessions 在 workspace 内，启动将失败！" -ForegroundColor Red
} else {
    Write-Host "✅ 正确：sessions 在 workspace 外，校验通过" -ForegroundColor Green
}

Write-Host "`n=== 目录树 ==="
Get-ChildItem "D:\nanobot-config" -Recurse | Select-Object FullName
Get-ChildItem "D:\知识库" -Recurse -Directory | Select-Object FullName
```

### 步骤 14：启动知识库实例

```powershell
nanobot gateway --config "D:\nanobot-config\config.json"
```

**首次启动时会发生什么**：

1. nanobot 检测到配置目录下的 `sessions/` 目录，创建或确认 `<workspace-id>` 子目录。
2. 如果 workspace 内存在旧版会话文件（`D:\知识库\sessions\*.jsonl`），自动迁移到 `D:\nanobot-config\sessions\<workspace-id>\` 下，迁移前会验证副本完整性。
3. workspace 内生成 `.nanobot/workspace-id` 文件，仅包含一个不透明 ID。
4. 迁移过程是**幂等的**，重复启动不会重复迁移。

预期输出：
```
✓ Loaded config from D:\nanobot-config\config.json
✓ Workspace: D:\知识库
✓ Sessions stored at D:\nanobot-config\sessions\<workspace-id>\
✓ Gateway listening on port 18791
```

### 步骤 15：启动后验证清单

| 检查项 | 命令 | 预期结果 |
| :--- | :--- | :--- |
| 配置文件 | `Test-Path D:\nanobot-config\config.json` | `True` |
| 会话根目录 | `Test-Path D:\nanobot-config\sessions` | `True` |
| 会话子目录 | `Get-ChildItem D:\nanobot-config\sessions` | 至少一个 `<workspace-id>` 目录 |
| workspace 内无 sessions | `Test-Path D:\知识库\sessions` | `False`（旧版路径不应存在） |
| workspace-id 文件 | `Test-Path D:\知识库\.nanobot\workspace-id` | `True` |
| Gateway 启动 | 观察启动输出 | 端口 18791 监听成功，无 RuntimeError |

### 执行要求

- 每完成一个步骤，输出确认信息。
- 如果某个文件已存在，先备份为 `.bak`，再覆盖。
- **首次启动前，确保没有旧版 nanobot 进程正在写入同一个 workspace**。
- **不要让旧版本和新版本同时写入同一个 workspace**，否则会导致会话数据冲突。
- 全部完成后，输出完整的目录树和文件清单。

现在开始执行。


## 新旧版本关键差异对照

| 变更项 | 旧版（≤0.3.4） | nanobot 0.3.5 |
| :--- | :--- | :--- |
| 会话存储位置 | `workspace/sessions/` | `<config-dir>/sessions/<workspace-id>/` |
| 配置目录 | 可与 workspace 相同 | 必须与 workspace 分离 |
| 校验逻辑 | 无 | `sessions_root` 不能在 workspace 内 |
| 会话迁移 | 无 | 首次启动自动迁移，幂等 |
| SOUL.md | 无会话说明 | 新增会话存储说明 |
| 降级方式 | 无 | `nanobot sessions restore-workspace` |

## 升级注意事项

如果你已经在旧版 nanobot 上部署了知识库，升级到 0.3.5 后需要遵循官方升级说明：

1. **备份**：先备份配置、workspace 和会话存储。
2. **停止旧进程**：停止所有旧版 nanobot 进程。
3. **启动新版本**：首次启动时 nanobot 会自动迁移旧会话文件。
4. **不要并行写入**：不要让旧版本和新版本同时写入同一个 workspace。
5. **如需降级**：停止 nanobot 后，使用 `nanobot sessions restore-workspace --config <config.json> --workspace <workspace>` 恢复会话到 workspace 内。

## 启动脚本（可选）

在桌面创建 `启动知识库.bat`：

```batch
@echo off
echo Starting Knowledge Base Nanobot...
nanobot gateway --config "D:\nanobot-config\config.json"
pause
```

以后双击即可启动。
也可以让nanobot把脚本修改成 wiki 命令来调用

---

这份部署指令已自包含，直接复制给 nanobot 即可执行。如果执行过程中遇到报错，把错误信息发给AI查看。
---

## 关联

- 配套文件：[LLM-Wiki 个人知识库搭建说明](/p/llm-wiki-guide/)
- Wiki 概念：LLM Wiki 模式 ｜ 标准定义层 ｜ 审核状态机
- Wiki 实体：llm-wiki Skill
- 来源摘要：个人知识库Agent自动搭建说明
- 规则文档：知识库使用手册 ｜ Agent 权限与操作规则
- 领域入口：科技 MOC

> 📄 **延伸阅读**：[LLM-Wiki 个人知识库搭建说明](/p/llm-wiki-guide/) —— 概念、目录结构、日常维护与迁移。
