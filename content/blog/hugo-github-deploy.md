---
title: "使用 Hugo + GitHub 部署博客"
date: 2026-09-27T10:11:00+08:00
slug: "hugo-github-deploy"
description: "从零把博客搭起来：Hugo 是什么、和 Jekyll/Hexo 的差别、GitHub Actions 自动化部署流程，以及我封装的一个 AI 部署 skill。"
tags: [
    "Hugo",
    "GitHub",
    "推荐",
]
categories: [
    "ai-tech",
]
---

## 一、Hugo 是什么？

Hugo 是一个用 Go 语言编写的静态网站生成器（SSG），自 2013 年发布以来，以其**极快的构建速度**和**简单的部署方式**在开发者社区中广受欢迎。

### ⚡ Hugo 的核心优势

- **极致的构建速度**：这是 Hugo 最突出的标签。由于采用 Go 语言编译，它的构建速度在同类工具中处于领先地位。根据基准测试，构建一个包含 5,000 页的网站，Hugo 仅需约 **2.1 秒**，而 Jekyll 则需要约 **4 分钟**。对于内容量大的站点，这个优势会非常明显。
- **简单的部署与零依赖**：Hugo 以单个二进制文件的形式分发，你下载后即可使用，无需安装 Ruby、Node.js 等运行时环境或处理复杂的依赖关系。这使得它在任何操作系统上的安装和部署都极其简单。
- **内置丰富功能**：Hugo 自带了许多开箱即用的功能，例如多语言支持、强大的分类系统（Tags/Categories）、图片处理管道以及对 SASS/SCSS 的原生支持，无需依赖外部插件。

### ⚠️ Hugo 的局限性

- **模板系统学习曲线较陡**：Hugo 使用 Go 语言的 `html/template` 语法，对于不熟悉 Go 的开发者来说，其语法和逻辑可能不太直观，相比 Jekyll 的 Liquid 或 Hexo 的 EJS 等模板语言，需要更多的学习时间。
- **缺乏官方插件系统**：Hugo 没有像 Jekyll 或 Hexo 那样成熟的插件生态系统。虽然可以通过 Shortcodes、主题和 Hugo Modules 进行扩展，但实现高度自定义的功能时，可能不如有插件系统的工具灵活。
- **主题生态相对有限**：虽然 Hugo 拥有超过 900 个主题，但与 Hexo 等拥有庞大社区和丰富主题数量的工具相比，其主题库在数量和一些特定需求的覆盖上可能稍显不足。

### 🔍 与其他主流工具的横向对比

为了更直观地展示差异，下表汇总了 Hugo 与几款主流 SSG 的核心特性：

| 工具 | 编程语言 | 构建速度 | 学习曲线 | 核心优势 | 最适合场景 |
| --- | --- | --- | --- | --- | --- |
| **Hugo** | Go | **极快** (毫秒/秒级) | 中等 | 速度最快、单二进制、零依赖 | 大型站点、内容密集型网站、追求极致性能 |
| **Jekyll** | Ruby | 较慢 (分钟级) | 简单 | GitHub Pages 原生支持、成熟的插件生态 | 简单的博客、GitHub Pages 托管 |
| **Hexo** | Node.js | 快 | 简单 | 丰富的插件和主题生态、对 JS 开发者友好 | 个人博客、Node.js 技术栈用户 |
| **Gatsby** | JavaScript (React) | 较慢 | 陡峭 | 强大的 React 组件和 GraphQL 数据层 | 复杂的 React 应用、需要大量动态交互的站点 |
| **Astro** | JavaScript (多框架) | 快 | 简单 | 默认零 JS、支持多框架组件、内容驱动 | 现代内容网站、电商落地页、追求最小化 JS |
| **Eleventy** | JavaScript | 快 | 简单 | 配置灵活、不依赖特定前端框架、对 JS 生态友好 | 偏好 JS 工具链的开发者、注重灵活性的项目 |

综合来看，Hugo 的核心竞争力在于 **“速度”** 和 **“简洁”**。它非常适合以下场景：

- **内容量大、更新频繁的博客或文档站点**：构建速度的优势会极大提升开发体验和部署效率。
- **不想深入折腾前端工具链的开发者**：单二进制文件、零依赖的特性让部署和维护变得非常简单。
- **对网站性能有极致要求的场景**：Hugo 生成的静态站点本身就非常轻量，配合 CDN 可以轻松获得极快的加载速度。

## 二、整体部署流程概述（简单了解）

使用 Hugo 和 GitHub 部署博客，整个流程可以概括为：

**在本地用 Hugo 生成静态网站 → 通过 Git 推送到 GitHub 仓库 → 再由 GitHub Actions 自动构建并发布到 GitHub Pages。**

### 🧭 整体流程概览

整个部署管线大致分为四个阶段：

1. **本地环境搭建**：安装 Hugo 和 Git。
2. **创建与配置站点**：初始化 Hugo 站点，选择并配置主题。
3. **编写与预览内容**：用 Markdown 写文章，本地启动服务预览。
4. **连接 GitHub 并自动化部署**：将代码推送到 GitHub，配置 Actions 自动构建和发布。

### 🛠️ 步骤一：准备工作与本地搭建

1. **安装必要工具**  
   你需要在本地安装 **Hugo**（建议安装 **extended** 扩展版，因为许多主题依赖 SCSS 支持）和 **Git**。
   - **macOS**: `brew install hugo`
   - **Windows**: `scoop install hugo-extended`
   - **Linux**: `sudo snap install hugo` 或 `sudo apt install hugo`

2. **创建 Hugo 站点并初始化 Git**  
   在终端中运行以下命令，创建一个名为 `my-blog` 的站点，并初始化 Git 仓库：
   ```bash
   hugo new site my-blog
   cd my-blog
   git init
   ```

3. **选择并安装主题**  
   Hugo 有丰富的主题生态，中文友好的热门主题包括 **PaperMod**、**LoveIt**、**hugo-theme-stack** 等。以 PaperMod 为例，通过 Git 子模块安装：
   ```bash
   git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
   ```
   然后，在站点配置文件 `hugo.toml`（或 `config.toml`）中启用主题：
   ```toml
   theme = "PaperMod"
   ```

4. **基础站点配置**  
   编辑 `hugo.toml`，至少设置以下关键项：
   ```toml
   baseURL = "https://你的用户名.github.io/"
   languageCode = "zh-cn"
   title = "我的博客"
   ```

### ✍️ 步骤二：编写内容与本地预览

1. **创建新文章**  
   使用 `hugo new` 命令创建文章，它会根据模板自动生成 Front Matter（元数据）：
   ```bash
   hugo new posts/my-first-post.md
   ```
   打开生成的文件，将 `draft: true` 改为 `false`，文章才会被正式发布。

2. **本地预览**  
   启动 Hugo 内置的开发服务器，实时预览效果：
   ```bash
   hugo server -D
   ```
   访问 `http://localhost:1313/` 即可查看。`-D` 参数表示包含草稿文章。

### 🚀 步骤三：推送到 GitHub 并启用 Pages

1. **创建 GitHub 仓库**  
   在 GitHub 上新建一个仓库。**关键点**：如果你希望博客地址是 `https://你的用户名.github.io`，仓库名应设置为 `你的用户名.github.io`；如果是项目站点，仓库名可以任意，但地址会多一级子路径。

2. **推送本地代码**  
   在本地仓库中添加远程地址并推送：
   ```bash
   git remote add origin https://github.com/你的用户名/仓库名.git
   git add .
   git commit -m "Initial commit"
   git push -u origin main
   ```

3. **启用 GitHub Pages**  
   进入仓库的 **Settings > Pages**，将 **Source** 从 “Deploy from a branch” 改为 **“GitHub Actions”**。这个改动是立即生效的，无需保存按钮。

### ⚙️ 步骤四：配置 GitHub Actions 自动部署

这是实现“推送即部署”的核心。你需要创建一个 GitHub Actions 工作流文件。

1. **创建工作流文件**  
   在站点根目录下创建 `.github/workflows/hugo.yaml` 文件。

2. **配置工作流内容**  
   将以下官方示例内容粘贴进去（注意根据你的 Hugo 版本和分支名微调）。这个工作流会在你向 `main` 分支推送代码时，自动安装 Hugo、构建站点并部署到 GitHub Pages：
   ```yaml
   name: Deploy Hugo site to Pages
   on:
     push:
       branches:
         - main
     workflow_dispatch:
   permissions:
     contents: read
     pages: write
     id-token: write
   concurrency:
     group: "pages"
     cancel-in-progress: false
   defaults:
     run:
       shell: bash
   jobs:
     build:
       runs-on: ubuntu-latest
       env:
         HUGO_VERSION: 0.147.9
       steps:
         - name: Install Hugo CLI
           run: |
             wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
             && sudo dpkg -i ${{ runner.temp }}/hugo.deb
         - name: Checkout
           uses: actions/checkout@v4
           with:
             submodules: recursive
             fetch-depth: 0
         - name: Setup Pages
           id: pages
           uses: actions/configure-pages@v5
         - name: Build with Hugo
           env:
             HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
             HUGO_ENVIRONMENT: production
           run: |
             hugo \
               --minify \
               --baseURL "${{ steps.pages.outputs.base_url }}/"
         - name: Upload artifact
           uses: actions/upload-pages-artifact@v3
           with:
             path: ./public
     deploy:
       environment:
         name: github-pages
         url: ${{ steps.deployment.outputs.page_url }}
       runs-on: ubuntu-latest
       needs: build
       steps:
         - name: Deploy to GitHub Pages
           id: deployment
           uses: actions/deploy-pages@v4
   ```

3. **推送并触发部署**  
   提交这个工作流文件并推送到 GitHub。推送后，GitHub Actions 会自动开始运行。你可以在仓库的 **Actions** 标签页下查看构建和部署的实时日志。

### 🔄 后续写作与发布流程

完成上述配置后，你日常的写作和发布流程会非常简洁：

1. **本地写作**：用 `hugo new posts/文章名.md` 创建文章，用 Markdown 编辑。
2. **本地预览**：`hugo server -D` 确认效果。
3. **一键发布**：`git add . && git commit -m "更新文章" && git push`。

推送后，GitHub Actions 会在云端自动完成构建和部署，几分钟后你的博客就会更新。

### 💡 关键提醒

- **`baseURL` 配置**：务必在 `hugo.toml` 中正确设置 `baseURL`，否则部署后可能出现样式或链接错误。
- **主题更新**：如果通过 Git 子模块安装主题，在克隆仓库或 Actions 中构建时，需要使用 `--recursive` 参数来拉取主题代码，上述工作流示例中已包含 `submodules: recursive`。
- **自定义域名**：如果你绑定了自定义域名，需要在仓库的 Pages 设置中配置，并相应调整 `baseURL`。

## 三、Hugo 其他安装方式

Hugo 还支持免本地安装的构建和部署方式。得益于其“单二进制文件”的设计，你可以将构建过程完全放在云端完成，本地只需要一个浏览器和 Git 即可。

### ☁️ 方式一：利用 GitHub Actions 自动构建

只需要在项目仓库中配置好 GitHub Actions 工作流（如之前封装好的 `hugo.yaml`），每次将 Markdown 源文件推送到 GitHub 后，Actions 的虚拟环境会自动安装 Hugo、构建站点并部署到 GitHub Pages。

**核心配置**：  
在 `hugo.yaml` 中，你只需要通过 `peaceiris/actions-hugo@v3` 这个 Action 来按需安装指定版本的 Hugo（支持 extended 版本和 Hugo Modules），它会在几秒内完成安装。

```yaml
- name: Setup Hugo
  uses: peaceiris/actions-hugo@v3
  with:
    hugo-version: '0.147.9'
    extended: true
```

这样，你的本地环境就完全不需要 Hugo，只需负责编写 Markdown 和推送代码。

### 🐳 方式二：使用 Docker 容器

如果你需要在本地进行**实时预览**，但不想直接安装 Hugo 到系统里，Docker 是最佳选择。

你可以直接运行官方的 Hugo Docker 镜像，将本地项目目录挂载到容器中，即可在容器内执行 `hugo server` 命令。

```bash
# 示例：使用官方镜像启动本地预览服务
docker run --rm -it \
  -v $(pwd):/src \
  -p 1313:1313 \
  ghcr.io/gohugoio/hugo:latest server --bind 0.0.0.0
```

这种方式将 Hugo 的运行环境隔离在容器内，不会污染你的主机系统，同时还能获得完整的本地开发体验。

### 🚀 方式三：使用云构建服务平台

一些静态网站托管平台（如 **Netlify**、**Vercel**、**EdgeOne Pages** 等）内置了对 Hugo 的支持。

你只需要将 GitHub 仓库连接到这些平台，它们会自动检测到这是一个 Hugo 项目，并在云端完成构建和部署，实现真正的“零配置”持续集成。

对于“Hugo + GitHub”的博客方案，**使用 GitHub Actions 进行云端构建是最简洁、最一致的路径**。你的本地角色可以简化为“内容创作者”，只需专注于用 Markdown 写作和 `git push` 即可。

## 四、使用 AI 工具自动部署

下面是一个完整的 Skill 技能，整合了 **Hugo 博客部署** 和 **自定义编辑** 两大流程。你只需将整个 `hugo-blog-manager` 文件夹放入任何 agent 的 `skills/` 目录，即可通过自然语言触发。

我用的是 nanobot，其他 AI 工具一样，**保险起见可以让 AI 对 skill 做一次校验**。

---

### 📁 目录结构

```plain
skills/
└── hugo-blog-manager/
    ├── SKILL.md
    └── scripts/
        ├── _common.ps1          # 公共函数：DryRun 预演、hugo 定位、UTF-8 写文件
        ├── check_env.ps1        # 只读环境体检
        ├── install_hugo.ps1     # 安装 hugo（本地 winget / Docker）
        ├── init_site.ps1        # 新建站点 + git init
        ├── install_theme.ps1    # 装主题并写入 theme:
        ├── preview.ps1          # hugo server 本地预览
        ├── push_github.ps1      # 提交并推送
        └── setup_actions.ps1    # 生成 GitHub Actions 部署 workflow
```

> 脚本全部是 PowerShell（Windows 原生，不依赖 WSL / Git Bash），并且**每个脚本都带 `-DryRun`**：只打印将要执行的命令，不做任何改动。这就是整个 Skill「确认式执行」的地基——先预演给用户看，用户点头了才实跑。

---

### 📄 SKILL.md

````markdown
---
name: hugo-blog-manager
description: 管理 Hugo 博客：从零创建站点、选装主题、推送 GitHub 并配置 GitHub Actions 自动部署到 GitHub Pages；以及安全地修改现有站点的内容、Front Matter、布局、样式和配置（先给 diff 再逐步确认，hugo server 本地预览，构建或发布失败排错）。当用户说“部署 Hugo 博客”“新建 Hugo 站点”“修改 Hugo 页面”“改页脚”“加关于页”“博客发不出去/Pages 404”或提到 Hugo、GitHub Pages、hugo.yaml 时使用。安装 Hugo 本体必须先让用户在“本地安装 / Docker 安装”之间选择；主题必须在安装那一刻现场询问用户。
metadata:
  nanobot:
    emoji: 📝
    requires:
      bins: ["git"]
---

# Hugo 博客管理器

两个工作流：**流程 A 部署新博客**、**流程 B 安全修改现有站点**。

脚本目录（下文记作 `$S`，调用时替换成这个技能的真实路径）：

```
$S = <skills>\hugo-blog-manager\scripts
```

运行时的当前目录不是技能目录，**调用脚本必须用绝对路径**。

## 两条硬性规则

1. **Hugo 本体安装必须先问用户**。只能在用户明确选择「本地安装」（winget `Hugo.Hugo.Extended`）或「Docker 安装」（拉 hugo 镜像，用 `docker run` 跑）之后，才通过 `install_hugo.ps1` 执行。不要自动安装，不要替用户选。
2. **主题必须现场问用户**。本技能不预设任何主题。走到装主题那一步时，先给 3~4 个候选（各自特点 + 仓库地址）让用户挑，选定后才跑 `install_theme.ps1`。

## 确认式执行

每个关键节点三步走：先跑 `-DryRun` → 把命令和影响讲给用户 → 用户明确同意后去掉 `-DryRun` 实跑。

## 全局前置检查

```powershell
powershell -NoProfile -File "$S\check_env.ps1"
```

只读体检（hugo / git / docker / winget），退出码 `0`=就绪、`1`=hugo 缺失、`2`=git 缺失、`3`=两者都缺。hugo 缺失时**不要自己装**，转去问用户选哪种方式：

```powershell
powershell -NoProfile -File "$S\install_hugo.ps1" -Method Local  -DryRun   # 本地安装
powershell -NoProfile -File "$S\install_hugo.ps1" -Method Docker -DryRun   # Docker 安装
```

装完重跑 `check_env.ps1` 确认；本地安装后若 `hugo` 仍不可见，提醒用户新开终端刷新 PATH。

## 流程 A：部署新博客

按顺序走，每步先 DryRun 再等确认。

1. **环境检查** —— 见上。
2. **确定站点位置与标题** —— 问用户博客放哪个目录、叫什么名字（`baseURL` 后续可改）。
3. **创建站点**：`init_site.ps1 -SitePath "<目录>" -Title "<标题>"` → `hugo new site --format yaml` + `git init -b main` + 写 `hugo.yaml` 与 `.gitignore`。目标目录必须为空，脚本拒绝覆盖已有内容。
4. **选主题（必须先问用户）**：`install_theme.ps1 -SitePath "<目录>" -ThemeName <名> -ThemeRepo <仓库地址>`。默认走 `git submodule`（主题可后续升级）；GitHub 不可达时脚本会报错并提示改用 `-Mode Zip -ZipUrl <镜像 zip 地址>`。
5. **本地预览，交给用户确认**：`preview.ps1 -SitePath "<目录>"`（`hugo server`，默认 http://127.0.0.1:1313/）。这是前台长进程，用后台会话跑；**界面效果必须由用户在浏览器确认**，不要用 HTTP 探测 localhost 代替。
6. **建 GitHub 仓库**（让用户在网页端建，**不要勾 README / .gitignore**），然后推送：
   `push_github.ps1 -SitePath "<目录>" -RepoUrl https://github.com/<user>/<repo>.git -Message "Initial commit: Hugo blog"`
7. **配置自动部署**：`setup_actions.ps1 -SitePath "<目录>"` 写 `.github/workflows/hugo.yaml` 并推送。Hugo 版本自动取自本地 hugo 或 hugo 容器，也可 `-HugoVersion x.y.z` 指定。
8. **收尾**：提醒用户去仓库 `Settings → Pages → Source` 选 **GitHub Actions**（一次性手动步骤），之后每次 push 自动发布。

## 流程 B：安全修改现有站点

1. **先读再动**。定位改动点：`hugo.yaml`（站点配置）、`content/`（文章 / Front Matter）、`layouts/`（模板覆盖）、`assets/` 与 `static/`（样式、静态资源）、`themes/<主题>/`（主题源码）。
2. **主题是外部代码（submodule），默认不改它内部文件**。优先在站点根目录放同名文件覆盖（Hugo 的 lookup order），例如 `layouts/partials/footer.html`。确实必须改主题内部时，先说明代价（升级会被覆盖，或需要 fork）再决定。
3. **每处改动都是三步**：读原文 → 把 diff（改什么、为什么、影响哪些页面）给用户看 → 等确认 → 再改。一次只改一处，别攒一堆一起改。
4. **改完必看**：`preview.ps1` 让用户在浏览器确认效果；改模板时可用 `-Drafts` 连草稿一起看。
5. **用户认可后再提交**：`push_github.ps1 -SitePath "<目录>" -RepoUrl <origin> -Message "<改了什么>"`。
6. **不可逆操作先备份**：删内容、换主题、改 `baseURL`、重写 git 历史之前，先复制目录或开新分支。

### 常见排错

- **Pages 404 / 页面不更新**：仓库 `Settings → Pages → Source` 必须是 `GitHub Actions`；`baseURL` 与实际域名不一致时由 workflow 的 `--baseURL` 兜底。
- **样式没生效**：核对主题的 `assets` / `static` 目录结构，以及 `hugo.yaml` 里的 `theme` 值与 `themes/` 下的目录名是否一致。
- **构建失败**：看 Actions 日志；本地 hugo 版本与主题要求不匹配时，用 `setup_actions.ps1 -HugoVersion` 重新生成 workflow。
- **CI 里主题目录是空的**：workflow 的 checkout 已带 `submodules: recursive`；若仍为空，确认 `.gitmodules` 已提交进仓库。
- **用户站点仓库（`<user>.github.io`）**：`baseURL` 必须是根域名（如 `https://mythstraw.github.io/`）；私有仓库开 Pages 需付费，必须 public。

## Hugo 陷阱速查

- `T` / `i18n` 返回值会被 HTML 转义 → 翻译串含 `<a>` 时必须 `| safeHTML`。
- 模板内链接用 `.RelPermalink`；`relURL` 在带语言前缀时会重复加前缀。
- 首页 `.Title` 为空 → 判断首页必须用 `.IsHome`。
- front matter 不写 `slug` 时 Hugo 用**标题**做 URL（中文标题 → 百分号编码）→ 每篇显式写 `slug`。
- `buildFuture=false`（默认）：`date` 写成未来时间会被整篇跳过。
- 用 Docker 跑 hugo 必须把站点子目录挂成工作目录（挂整盘会报 "Unable to locate config file"）。

## 脚本清单

| 脚本 | 作用 | 关键参数 |
| --- | --- | --- |
| `check_env.ps1` | 只读环境体检 | `-Json` |
| `install_hugo.ps1` | 安装 hugo（**须用户先选方式**） | `-Method Local\|Docker`、`-StartEngine` |
| `init_site.ps1` | 新建站点 + git init + 配置 + .gitignore | `-SitePath`、`-Title`、`-BaseUrl`、`-LanguageCode` |
| `install_theme.ps1` | 装主题并写入 `theme:`（**主题须现场问用户**） | `-ThemeName`、`-ThemeRepo`、`-Mode Submodule\|Zip` |
| `preview.ps1` | `hugo server` 本地预览 | `-Port`、`-Drafts`、`-BindAddress` |
| `push_github.ps1` | 提交并推送到 GitHub | `-RepoUrl`、`-Branch`、`-Message`、`-Rebase` |
| `setup_actions.ps1` | 生成 GitHub Actions 部署 workflow | `-HugoVersion`、`-Branch`、`-NoPush` |
````

---

### 🔧 脚本关键片段

为控制篇幅，下面是各脚本的关键部分（真代码节选，`# ...` 表示省略；完整脚本见文末下载）。

#### `scripts/_common.ps1`

这段是整个技能的核心：**本机装了 hugo 就直接跑，没装但 Docker 引擎在跑就用容器跑**——两种安装方式共用同一套脚本。

```powershell
# 有原生 hugo 就原生跑；没有原生 hugo 但 Docker 引擎在跑，就用容器跑
function Invoke-Hugo {
    param(
        [Parameter(Mandatory = $true)][string]$SitePath,
        [Parameter(Mandatory = $true)][string[]]$HugoArgs,
        [string]$MountPath = '',
        [string]$Image = 'hugomods/hugo:exts',
        [string[]]$ExtraDockerArgs = @(),
        [switch]$DryRun
    )
    $status = Get-HugoStatus
    if ($status.Mode -eq 'native') {
        $line = ('cd "{0}"; hugo {1}' -f $SitePath, ($HugoArgs -join ' '))
        if ($DryRun) { Write-Output "DRY-RUN: $line"; return }
        # ... 实跑，退出码非 0 就抛错
    }
    if ($status.Mode -eq 'docker') {
        $mount = $MountPath; if (-not $mount) { $mount = $SitePath }
        $src = ConvertTo-PosixPath $mount      # E:\myblog -> /e/myblog
        $dkArgs = @('run', '--rm') + $ExtraDockerArgs + @('-v', "${src}:/src", '-w', '/src', $Image, 'hugo') + $HugoArgs
        if ($DryRun) { Write-Output ('DRY-RUN: docker ' + ($dkArgs -join ' ')); return }
        # ... 实跑，退出码非 0 就抛错
    }
    throw "hugo is not available: neither a native 'hugo' executable nor a running Docker engine was found. Run install_hugo.ps1 first (ask the user to choose Local or Docker)."
}
```

#### `scripts/check_env.ps1`

只读体检，靠**退出码**把结论告诉 agent：`0`=就绪、`1`=hugo 缺失、`2`=git 缺失、`3`=两者都缺。缺 hugo 时打印的是「去问用户选哪种方式」，而不是自己去装。

```powershell
$hugo   = Get-HugoStatus      # native / docker / none
$git    = Get-GitExe
$docker = Test-DockerEngine   # client / engine / version

$missing = @()
if ($hugo.Mode -eq 'none') { $missing += 'hugo' }
if (-not $git)             { $missing += 'git' }

# exit code: 0 ready, 1 hugo missing, 2 git missing, 3 both missing
$exitCode = 0
if ($missing -contains 'hugo') { $exitCode += 1 }
if ($missing -contains 'git')  { $exitCode += 2 }

if ($missing -contains 'hugo') {
    Write-Output 'NEXT: hugo is missing. Do NOT install silently - ask the user to pick one:'
    Write-Output '  A) local install  : powershell -NoProfile -File install_hugo.ps1 -Method Local    (winget Hugo.Hugo.Extended)'
    Write-Output '  B) docker install : powershell -NoProfile -File install_hugo.ps1 -Method Docker   (needs Docker Desktop running)'
}
```

#### `scripts/install_hugo.ps1`

两种安装方式（`-Method Local|Docker`），都要求用户先表态。装完自己复检一次，而不是假设成功。

```powershell
if ($Method -eq 'Local') {
    $wgArgs = @('install', '--id', $WingetId, '-e', '--accept-source-agreements', '--accept-package-agreements')
    if ($DryRun) { Write-Output ('DRY-RUN: {0} {1}' -f $winget, ($wgArgs -join ' ')); exit 0 }
    & $winget @wgArgs
    $after = Get-HugoStatus
    if ($after.Mode -eq 'native') { Write-Output ("OK: hugo {0} -> {1}" -f $after.Version, $after.Exe); exit 0 }
    # ... 仍不可见时给出提示（下载被墙、需刷新 PATH、或改用 Docker）
}

# Docker 方式：引擎没起时可加 -StartEngine，最多等 240 秒
& $docker pull $Image
& $docker run --rm $Image hugo version
```

#### `scripts/init_site.ps1`

新建站点前的唯一硬校验：**目标目录必须是空的**，宁可报错也不碰已有内容。

```powershell
if (Test-Path -LiteralPath $sitePath) {
    $entries = @(Get-ChildItem -LiteralPath $sitePath -Force | Where-Object { $_.Name -ne '.' })
    if ($entries.Count -gt 0) {
        Write-Fail "Target directory is not empty: $sitePath. Pick another path, or clean it up first (refusing to touch existing content)." 4
    }
}

Invoke-Hugo -SitePath $parent -MountPath $parent -Image $Image -DryRun:$DryRun `
    -HugoArgs @('new', 'site', $leaf, '--format', 'yaml')

# 新站点没有需要保留的内容，直接写一份干净的 hugo.yaml（theme: 留到装主题那步再写）
$config = @"
baseURL: "$BaseUrl"
languageCode: "$LanguageCode"
title: "$Title"
"@
New-Utf8File -Path $configPath -Content $config      # UTF-8 无 BOM，避免中文标题乱码
```

#### `scripts/install_theme.ps1`

默认 `git submodule`（主题可后续升级）；`github.com` 不可达时不硬顶，改走 `-Mode Zip` 从镜像下载。

```powershell
# 先探活：不通就明确告诉用户「加速器没开」或「改用 -Mode Zip」
$probe = & $git ls-remote $ThemeRepo HEAD 2>&1
if ($LASTEXITCODE -ne 0) {
    Write-Fail "Cannot reach $ThemeRepo" 3
}

& $git -C $sitePath submodule add $ThemeRepo "themes/$ThemeName"
& $git -C $sitePath submodule update --init --recursive

# -Mode Zip 分支：下载 + 解压到 themes/<名>
Invoke-WebRequest -Uri $ZipUrl -OutFile $tmpZip -UseBasicParsing -TimeoutSec 300
Expand-Archive -LiteralPath $tmpZip -DestinationPath $tmpDir -Force

# 把 theme: 写进配置（已有就替换，没有就追加）
if ($content -match '(?m)^\s*theme\s*[:=]') {
    $newContent = [regex]::Replace($content, '(?m)^\s*theme\s*[:=].*$', $line)
}
New-Utf8File -Path $configPath -Content $newContent
```

#### `scripts/preview.ps1`

本地预览。Docker 模式下有个坑：容器内得监听 `0.0.0.0`，端口只发布到宿主机回环地址，否则外面打不开。

```powershell
$status = Get-HugoStatus
if ($status.Mode -eq 'none') {
    Write-Fail "hugo is not available. Run check_env.ps1, then ask the user to choose Local or Docker install." 1
}

# 容器模式：容器内监听 0.0.0.0，端口发布在宿主机回环地址上
$innerBind = $BindAddress
$extra = @()
if ($status.Mode -eq 'docker') {
    $innerBind = '0.0.0.0'
    $extra = @('-p', ("{0}:{1}:{1}" -f $BindAddress, $Port))
}

$hugoArgs = @('server', '--port', "$Port", '--bind', $innerBind)
if ($Drafts) { $hugoArgs += '-D' }
```

#### `scripts/push_github.ps1`

三道保险：身份没配不提交、工作区干净不算失败、远端已有提交时**默认中止**（要显式加 `-Rebase`）。

```powershell
# 1) git 身份没配就中止，绝不以未知作者提交
if (-not $email -or -not $name) {
    Write-Fail 'git user.name / user.email are not configured. Ask the user for them and re-run with -GitUserName / -GitUserEmail.' 4
}

# 2) 工作区干净就跳过 commit（不是报错）
$status = (& $git -C $sitePath status --porcelain 2>$null)
if ($status) {
    Invoke-Git -GitArgs @('-C', $sitePath, 'add', '-A')
    Invoke-Git -GitArgs @('-C', $sitePath, 'commit', '-m', $Message)
}

# 3) 远端已有提交时中止，除非显式 -Rebase
$remoteHeads = (& $git ls-remote --heads origin $Branch 2>$null)
if ($remoteHeads -and $LASTEXITCODE -eq 0) {
    if (-not $Rebase) {
        Write-Fail "Remote is not empty. Re-run with -Rebase to pull remote commits first, or push to another repo/branch." 4
    }
    Invoke-Git -GitArgs @('-C', $sitePath, 'pull', '--rebase', 'origin', $Branch)
}

Invoke-Git -GitArgs @('-C', $sitePath, 'push', '-u', 'origin', $Branch) -AllowFail
```

#### `scripts/setup_actions.ps1`

Hugo 版本不写死：优先用 `-HugoVersion`，否则从本地 hugo 或 hugo 容器里探测，保证 CI 用和你本地一致的版本。

```powershell
if (-not $HugoVersion) {
    $status = Get-HugoStatus
    if ($status.Mode -eq 'native') {
        $HugoVersion = Get-HugoVersionFromString -Text $status.Version
    }
    elseif ($status.Mode -eq 'docker') {
        $raw = @(& $status.Exe run --rm $Image hugo version 2>$null) -join ' '
        $HugoVersion = Get-HugoVersionFromString -Text $raw
    }
}

# 模板里占位，再替换成真实版本号与分支，UTF-8 无 BOM 写出
$workflow = $template.Replace('__HUGO_VERSION__', $HugoVersion).Replace('__BRANCH__', $Branch)
New-Utf8File -Path $workflowPath -Content $workflow

# 交给 push_github.ps1 提交推送（复用同一套身份/远端/分支保护）
& (Join-Path $PSScriptRoot 'push_github.ps1') -SitePath $sitePath -RepoUrl $remote -Message $Message
```

---

### 🚀 使用方式

1. 解压后把整个 `hugo-blog-manager` 文件夹放进 nanobot 工作区的 `skills/` 目录（Windows 下不用 `chmod`，PowerShell 脚本直接就能跑）。
2. 对 nanobot 说：
   - **部署新博客**：“部署 Hugo 博客”
   - **修改现有博客**：“修改 Hugo 页脚”、“添加关于页面”等
3. nanobot 会自动加载 Skill，按对应流程逐步询问并执行——每个节点都是「先 `-DryRun` 给你看命令 → 你确认 → 才真正执行」。

---

### 📦 下载这个 Skill

上面第四节贴的只是节选，完整脚本（7 个脚本 + 公共函数，含全部边界检查、退出码与排错提示）打包在这里：

**[⬇️ 下载 hugo-blog-manager.zip](https://mythstraw.github.io/downloads/hugo-blog-manager.zip)**

下载后解压，把整个 `hugo-blog-manager` 文件夹放进 nanobot 工作区的 `skills/` 目录，重启会话即可自动加载。
