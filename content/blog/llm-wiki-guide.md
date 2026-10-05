---
title: "LLM-Wiki 个人知识库搭建说明"
date: 2026-10-05T20:35:00+08:00
slug: "llm-wiki-guide"
description: "从 LLM-Wiki 与 RAG 的基础概念，到目录结构、Agent 自动搭建、日常维护与迁移：一份用 nanobot + Obsidian 搭建个人知识库的完整说明书。"
tags: [
    "知识库",
    "nanobot",
    "Obsidian",
    "RAG",
]
categories: [
    "ai-tech",
]
---

> 版本：1.0
> 适用：希望用 AI Agent 管理个人知识库的用户
> 默认工具链：nanobot + Obsidian + WorkBuddy（可选）

---

## 第一部分 基础概念

### 1.1 什么是 LLM-Wiki

LLM-Wiki 是一种由 AI Agent 驱动的个人知识库构建范式。它的核心思想是：

> **让大语言模型担任“知识编译器”，把你提供的原始资料持续编译成结构化、可积累的 Markdown 维基（Wiki），而不是每次提问时临时检索。**

LLM-Wiki 的工作方式是“摄入时编译”：LLM 通读原始资料，完成语义理解、要点提炼、知识分类，生成结构化 Wiki 页面。每个主题单独成页，页面自带摘要，并通过双链形成知识网络。知识被持久沉淀下来，实现复利增长。

**核心比喻**：

- Obsidian 是 IDE
- LLM 是程序员
- Wiki 是代码库

---

### 1.2 什么是 RAG

RAG = Retrieval-Augmented Generation，检索增强生成。

它的基本流程：

1. 用户提问
2. 系统从知识库中检索相关内容
3. 把检索结果作为上下文，交给大模型
4. 大模型基于这些内容生成回答
5. 最好附带引用来源

RAG 不是单纯搜索，也不是单纯大模型，而是 **检索系统 + 大模型生成 + 知识库**。

**RAG 与 LLM-Wiki 的关系**：

- RAG 是让大模型使用知识库的检索增强机制
- LLM-Wiki 是在 RAG 之上增加了一个“知识编译层”
- 两者互补：RAG 保证事实精度，LLM-Wiki 负责知识结构化与沉淀

---

## 第二部分 知识库的作用

### 2.1 LLM-Wiki 知识管理

LLM-Wiki 知识管理的核心优势：

| 维度 | 传统 RAG | LLM-Wiki |
| :--- | :--- | :--- |
| 存储 | 向量数据库（二进制） | 纯 Markdown 文件 |
| 可读性 | 需要工具查看 | 用 Obsidian 直接打开 |
| 可编辑 | 需改代码 | 改 Markdown 即可 |
| 知识积累 | 查询时临时计算 | 摄入时编译，持久沉淀 |
| 版本控制 | 需自行配置 | Git 自动提交，可回滚 |
| 与 Obsidian 集成 | 需桥接层 | 同一份文件，原生互通 |
| 隐私 | 取决于部署方式 | 完全本地，文件在你硬盘上 |

**关键点**：你的知识库就是一堆 Markdown 文件。即使 Agent 不再维护，你仍然可以用 Obsidian 打开、阅读、编辑它们。

RAG 适合固定知识检索查阅，比如中学生知识库，把教材教辅资料进行向量化处理放进数据库，通过大模型检索数据生成答案，本质上对资料文件做向量化拆分，匹配检索，生成答案。

Wiki 是知识管理系统，同样是中学生知识库，Wiki 是把资料先按 Wiki 的结构做拆分，方便检索和编辑，每一部分都可以自主编辑添加或者删除，对已有的知识进行维护查缺补漏，让知识更完善。

#### LLM-Wiki 结构：三层分离

一个标准的 LLM Wiki 系统通常分为三层：

1. **原始素材层 (Raw Sources)**：存放不可修改的原始文档（PDF、笔记、PR 记录等）。这一层**只读**，确保所有结论都能溯源回原始资料。
2. **Wiki 结构化知识层 (Wiki Layer)**：系统的核心产出，由 LLM 生成和维护。包含**概念页、实体页、对比页、综合页**等，全部为 Markdown 格式，页面间通过 `[[wikilinks]]` 相互链接。
3. **规则定义层 (Schema Layer)**：定义工作规则的“宪法”。包括文件夹结构、引用规则、摄入流程、问答行为规范等，用来指导 LLM 如何维护 Wiki。

#### ⚙️ 关键操作流程

LLM Wiki 的日常运转依赖几个核心操作：

- **Ingest（摄入/编译）**：将新资料放入原始素材层，LLM 会分析内容、提取实体与概念，并生成或更新对应的 Wiki 页面。这个过程通常采用“两步思维链”（先分析，再生成）来保证质量，并通过 **SHA256 哈希增量缓存**来避免重复处理未修改的文件，节省成本。
- **Query（查询）**：用户提问时，系统直接基于已编译的 Wiki 页面进行回答，速度更快，且答案会引用 Wiki 页面，而非原始文档的碎片。
- **Lint（自检与回填）**：LLM 定期或在触发时，检查知识库中的**矛盾、知识空白**，甚至可以通过“深度研究（Deep Research）”功能自动去外部填补这些空白。

### Wiki 结构说明

- LLM 编译的概念页（`concepts/`）：只负责把一个术语定义清楚
- 实体页（`entities/`）：只负责记录一个人、产品或工具的身份与立场
- 来源摘要页（`sources/`）：只负责把一组相关概念串起来，枢纽的作用
- 对比分析页（`comparisons/`）：只负责把两个易混的东西横向摆开
- 查询快照（`queries/`）：有价值的问答记录，是经过整理的“知识快照”
- 待定页 `candidates/`：wiki 草稿先进入这里，状态为 `needs_review`，需要人工审核

---

### 2.2 AI 笔记自动管理

AI Agent 可以自动完成：

- **Inbox 处理**：读取 `00_Inbox(灵感库)/` 根目录，判断类型（笔记/PDF/剪藏/待办）与领域，**先报告判断与建议去向、等你裁决**，裁决后才落位（编译 Wiki → 移动到 `_nanobot/raw/sources/`；参考资料 → `03_Resources(资源)/{领域}/`；待办/项目 → `01_Projects(项目)/`；归档 → `04_Archive(归档)/{年份}/`）
- **Wiki 编译**：读取原始素材，生成概念页、实体页、来源摘要页
- **候选审核辅助**：生成候选页面到 `candidates/`，等待你审核
- **标注索引**：扫描 `01–04`，生成标签、摘要、领域判断到 `annotations/`
- **质量检查**：检测死链、孤立页面、元数据缺失、矛盾信息
- **知识检索**：优先检索标准定义页和领域目录，减少幻觉

AI 自动管理笔记，对笔记做分类、打标签、做链接，通过 AI 自动管理笔记。

---

## 第三部分 AI 知识库用到的工具

### 3.1 Agent：nanobot（默认）

nanobot 是一个轻量级、本地优先的 AI Agent，适合管理个人知识库。

**核心能力**：

- 文件读写、Shell 执行、Skill 扩展
- 长期记忆（`MEMORY.md`）和事件日志（`HISTORY.md`）
- Dream 后台固化：将对话中的知识沉淀到记忆和 Wiki
- Local Triggers：响应文件系统变化，自动触发任务
- 多实例支持：不同实例物理隔离，互不干扰

**其他可选 Agent**：

- **WorkBuddy**：腾讯云 AI 办公助手，适合与办公生态集成，可写入 Inbox
- **opencode**：面向开发者的 AI 编程助手，适合 LLM-Wiki 编译模式

---

### 3.2 Obsidian

Obsidian 是基于 Markdown 的本地知识管理工具。

**核心作用**：

- 阅读和编辑知识库中的 Markdown 文件
- 双向链接、标签、图谱、MOC 导航
- 搜索、Dataview 查询
- 插件生态（Templates、Folder Bridge、BRAT 等）

**Obsidian Vault 与 nanobot workspace 的关系**：

- Obsidian Vault 是知识资产，必须是一个独立、可自由移动的文件夹
- nanobot workspace 指向 Vault 的父目录，Agent 文件放在 Vault 外
- 只有 Wiki 产出（`05_Agent(智能体)/wiki/`）在 Vault 内，参与 Obsidian 索引

---

## 第四部分 开始搭建知识库

### 4.1 Agent 选择

默认使用 **nanobot**。如果你已经使用 WorkBuddy 或 opencode，也可以混合使用。

**多工具协作原则**：

- WorkBuddy 只写入 `00_Inbox(灵感库)/`
- 知识库 nanobot 从 `00_Inbox(灵感库)/` 读取并处理
- 通用 nanobot 处理电脑问题，不访问知识库
- 两个 nanobot 实例物理隔离，端口不同

---

### 4.2 确定知识库目录结构

推荐目录结构如下（以 `D:\知识库\` 为例）：

```text
D:\知识库\                               ← 知识库实例的 workspace 根
│
├── _nanobot\                            # ★ Agent 专用目录（Obsidian 不可见）
│   ├── config.json                      # 知识库实例配置
│   ├── SOUL.md                          # Agent 沟通风格
│   ├── USER.md                          # 用户画像
│   ├── raw\                             # 原始素材层（只读）
│   │   ├── sources\
│   │   ├── articles\
│   │   ├── papers\
│   │   ├── transcripts\
│   │   └── assets\
│   ├── memory\                          # 记忆与状态
│   │   ├── MEMORY.md
│   │   ├── HISTORY.md
│   │   └── wiki_ingest_state.json
│   └── skills\                          # Agent 技能定义
│       └── llm-wiki\
│           └── SKILL.md
│
└── 我的知识库\                           ← ★ Obsidian Vault 根
    ├── 00_Inbox(灵感库)\                        # 快速捕获，多工具入口
    ├── 01_Projects(项目)\                     # 有截止日期的项目
    ├── 02_Areas(领域)\                        # 长期领域
    │   ├── 室内设计\
    │   ├── 3D打印\
    │   ├── 艺术\
    │   ├── 经济\
    │   ├── 科技\
    │   ├── 人文\
    │   └── 其他\
    ├── 03_Resources(资源)\                    # 主题知识、参考资料
    │   ├── 室内设计\
    │   ├── 3D打印\
    │   ├── 艺术\
    │   ├── 经济\
    │   ├── 科技\
    │   ├── 人文\
    │   └── 其他\
    ├── 04_Archive(归档)\                      # 归档
    │   ├── 2025\
    │   └── 2026\
    ├── 05_Agent(智能体)\                        # Agent 自动管理区
    │   ├── wiki\                        # LLM 编译的结构化知识
    │   │   ├── index.md
    │   │   ├── log.md
    │   │   ├── schema.md
    │   │   ├── concepts\
    │   │   ├── entities\
    │   │   ├── sources\
    │   │   ├── comparisons\
    │   │   ├── queries\
    │   │   └── candidates\              # 候选页面，待审核
    │   └── annotations\                 # 标注索引
    ├── 06_Toolbox(工具箱)\                      # 代码、脚本、工具包
    │   ├── README.md
    │   ├── scripts\
    │   ├── snippets\
    │   └── packages\
    ├── MOC(内容地图)\                             # 领域入口页
    ├── Templates(模板)\                       # 模板
    └── Attachments(附件)\                     # 全局附件
```

**各目录作用速查**：

| 目录 | 作用 | 谁写入 | Obsidian 索引 |
| :--- | :--- | :--- | :--- |
| `00_Inbox(灵感库)/` | 临时捕获，多工具入口 | 你 + WorkBuddy | ✅ |
| `01_Projects(项目)/` | 有截止日期的项目 | 你 | ✅ |
| `02_Areas(领域)/` | 长期维护的领域 | 你 | ✅ |
| `03_Resources(资源)/` | 主题知识、参考资料 | 你 | ✅ |
| `04_Archive(归档)/` | 归档 | 你 + Agent 可移入 | ✅ |
| `05_Agent(智能体)/wiki/` | LLM 编译的知识 | Agent + 你审核 | ✅ |
| `05_Agent(智能体)/annotations/` | 标注索引 | Agent | ✅ |
| `06_Toolbox(工具箱)/` | 代码、脚本 | 你 | 可选 |
| `_nanobot/raw/` | 原始素材 | 你放入，Agent 只读 | ❌ |
| `_nanobot/memory/` | Agent 记忆和状态 | Agent | ❌ |
| `_nanobot/skills/` | Skill 定义 | 你编写 | ❌ |

---

### 4.3 Agent 自动搭建文档

将以下部署指令复制给 nanobot，它会自动创建所有目录、写入所有配置文件。

**部署指令要点**：

1. 创建目录结构
2. 写入 `_nanobot/config.json`
3. 写入 `_nanobot/SOUL.md`
4. 写入 `_nanobot/USER.md`
5. 写入 `_nanobot/skills/llm-wiki/SKILL.md`
6. 写入 `我的知识库/知识库使用手册.md`
7. 写入 `我的知识库/05_Agent(智能体)/wiki/schema.md`
8. 创建 Wiki 索引、日志、MOC、工具箱索引、标注索引占位文件
9. 验证并启动

**完整部署指令见 [nanobot 0.3.5 个人知识库自动部署指令](/p/nanobot-deploy-instruction/)。**

---

## 第五部分 知识库测试优化

### 5.1 了解知识库系统，文件的作用，命令

**核心文件**：

| 文件 | 作用 |
| :--- | :--- |
| `SOUL.md` | Agent 沟通风格和工作原则 |
| `USER.md` | 用户画像和偏好 |
| `MEMORY.md` | 长期事实，每次对话加载 |
| `HISTORY.md` | 事件日志，按需检索 |
| `SKILL.md` | Agent 技能定义 |
| `schema.md` | Wiki 结构规则 |
| `知识库使用手册.md` | 目录职责和流转规则 |
| `Agent 权限与操作规则.md` | 权限边界和触发条件 |

**常用命令**：

| 命令 | 作用 |
| :--- | :--- |
| `/inbox-process` | 处理 `00_Inbox(灵感库)/`（先报判断与建议去向，等你裁决后才落位） |
| `/wiki-ingest` | 摄入 `raw/` 素材，生成候选页面 |
| `/wiki-review` | 审核候选页面 |
| `/wiki-lint` | 检查死链、孤立页面、元数据缺失 |
| `/wiki-save-answer` | 保存有价值的问答快照 |

---

### 5.2 知识库使用手册创建（作用）

**文件位置**：`我的知识库/知识库使用手册.md`

**作用**：

- 定义每个文件夹放什么、不放什么、命名规范、元数据要求
- 定义笔记在文件夹之间的流转规则
- 供你和 nanobot 共同参考
- nanobot 在处理 Inbox、生成标注索引时，会读取此文件作为判断依据

**核心内容**：

- 总原则
- 目录总览
- 逐目录详细说明
- 笔记流转规则
- 元数据字段速查
- 判断速查表

---

### 5.3 Agent 权限与操作规则创建（作用）

**文件位置**：`我的知识库/Agent 权限与操作规则.md`

**作用**：

- 定义 Agent 什么时候可以进入知识库
- 定义进入后可以读取哪些内容
- 定义什么情况下可以写入、移动、归档或调用 Skill
- 定义哪些操作必须等用户确认
- 定义定时任务、其他 Agent 对话的权限

**核心内容**：

- 进入知识库的触发条件
- 读取权限表
- 写入权限矩阵
- 必须等用户确认的操作清单
- 定时任务权限表
- 其他 Agent 权限表
- 冲突处理规则

**配置方式**：

- 在 `SOUL.md` 中添加强制引用指令
- 在 `SKILL.md` 中添加参考文档引用
- 重启 nanobot 使配置生效

---

### 5.4 完整工作流

### 工作流 1：摄入新素材（`/wiki-ingest`）

```text

你把 PDF/网页剪藏放入 raw/sources/
        ↓
执行 /wiki-ingest
        ↓
nanobot 读取 raw/ 内容，计算 SHA256 指纹
        ↓
对比 memory/wiki_ingest_state.json，跳过未修改的文件
        ↓
LLM 分析内容，提取实体与概念
        ↓
在 wiki/entities/ 和 wiki/concepts/ 下生成或更新 Markdown 页面
        ↓
自动添加双链、元数据（type, source, created）
        ↓
写入 wiki/log.md 记录本次摄入
        ↓
Git 自动提交变更
```

### 工作流 2：日常问答与知识沉淀

```text

你提问
        ↓
nanobot 从 MEMORY.md + wiki/ 中检索相关内容（grep + 注入）
        ↓
LLM 基于检索结果生成回答
        ↓
如果对话中产生了新知识，Dream 后台任务会将其固化
        ↓
固化结果写入 MEMORY.md 或 wiki/
        ↓
你可以用 /wiki-save-answer 把有价值的问答快照保存到 wiki/queries/
```

### 工作流 3：定期维护（`/wiki-lint`）

```text

执行 /wiki-lint
        ↓
nanobot 扫描 wiki/ 所有页面
        ↓
检测：矛盾信息、死链、孤立页面、元数据缺失、标题重复
        ↓
生成维护报告，写入 wiki/log.md
        ↓
你可以决定是否让 nanobot 自动修复
```

## 第六部分 如何维护，知识库迁移

### 6.1 日常维护

| 频率 | 任务 |
| :--- | :--- |
| 每天 | 检查 `00_Inbox(灵感库)/`，让 nanobot 处理 |
| 每周 | 执行 `/wiki-lint`，审核候选页面，更新 MOC |
| 每月 | 检查标注索引，整理标签，归档不再活跃的内容 |
| 每季度 | 备份知识库，检查 Agent 权限规则是否需要调整 |

**维护原则**：

- 原始素材放入 `_nanobot/raw/` 后不再修改
- Wiki 页面必须经过人工审核才能进入正式区域
- 人工区（`01–04`）由你主导，Agent 默认不改写
- 定期用 Git 提交变更，保留历史版本

---

### 6.2 知识库迁移

**迁移目标**：把知识库转移到新电脑，继续使用 WorkBuddy 或 Obsidian 编辑。

**核心原则**：

> **你的知识资产（Obsidian Vault）必须是一个独立的、可自由移动的文件夹。**

**迁移步骤**：

1. **备份 Vault**：复制 `我的知识库/` 文件夹到新电脑任意位置。
2. **用 Obsidian 打开**：在新电脑上安装 Obsidian，打开该文件夹。所有笔记、双链、图谱完整呈现，不需要安装 nanobot。
3. **使用 WorkBuddy**：启动 WorkBuddy，将 `我的知识库/` 设置为工作目录或授权文件夹，即可继续编辑。
4. **（可选）迁移 nanobot**：如果需要在新电脑上使用 nanobot 管理知识库，复制 `_nanobot/` 目录，修改 `config.json` 中的 workspace 路径，重新启动实例。

**备份策略**：

- **知识资产**：定期复制 `我的知识库/` 到其他硬盘或云盘
- **Agent 配置**：`_nanobot/` 目录可用 Git 管理，重要性低于知识资产
- **版本控制**：整个 Vault 建议用 Git 版本化，`_nanobot/memory/.git` 自动提交记忆变更

---

**说明书完。**

---

## 关联

- 配套文件：[nanobot 0.3.5 个人知识库自动部署指令](/p/nanobot-deploy-instruction/)
- Wiki 概念：LLM Wiki 模式 ｜ 标准定义层 ｜ 审核状态机
- Wiki 实体：llm-wiki Skill
- 来源摘要：个人知识库 Agent 自动搭建说明
- 规则文档：知识库使用手册 ｜ Agent 权限与操作规则
- 领域入口：科技 MOC

> 📄 **延伸阅读**：[nanobot 0.3.5 个人知识库自动部署指令](/p/nanobot-deploy-instruction/) —— 可直接复制给 Agent 的完整部署指令。
