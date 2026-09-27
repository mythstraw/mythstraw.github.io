---
title: "使用 Hugo + GitHub 部署博客"
date: 2026-09-27T10:11:00+08:00
slug: "hugo-github-deploy"
description: "从零把博客搭起来：Hugo 是什么、和 Jekyll/Hexo 的差别、GitHub Actions 自动化部署流程，以及我封装的一个 AI 部署 skill。"
tags: [
    "Hugo",
    "GitHub",
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
    ├── scripts/
    │   ├── check_env.sh
    │   ├── init_site.sh
    │   ├── install_theme.sh
    │   ├── push_github.sh
    │   └── setup_actions.sh
    └── templates/
        └── hugo-workflow.yaml
```

---

### 📄 SKILL.md

````markdown
---
name: hugo-blog-manager
description: 管理 Hugo 博客，支持从零部署到 GitHub Pages，以及安全地自定义修改页面。每个关键节点等待用户确认。
version: 1.0.0
author: foreveryang
---

# Hugo 博客管理器

本 Skill 提供两大工作流：
- **流程 A：部署新博客** — 从零创建 Hugo 站点，推送到 GitHub，并配置 GitHub Actions 自动部署。
- **流程 B：自定义修改** — 安全地修改现有 Hugo 项目的内容、布局、样式或配置，每步展示 diff 并等待确认。

所有操作均遵循“确认式执行”：nanobot 执行具体命令，但在每个关键节点暂停，等待用户明确确认。

---

## 触发条件

- 当用户说“部署 Hugo 博客”、“新建 Hugo 站点”、“创建博客”时，进入 **流程 A**。
- 当用户说“修改 Hugo 页面”、“自定义主题”、“改页脚”、“加关于页”、“编辑博客”时，进入 **流程 B**。
- 若意图不明确，询问用户：“您是要部署新博客，还是修改现有博客？”

---

## 全局前置检查

无论进入哪个流程，首先执行：

```bash
bash scripts/check_env.sh
```

该脚本检查 `hugo` 和 `git` 是否已安装。若未安装，提示用户安装后重新开始。

---

### 流程 A：部署新博客

#### 节点 A1：项目文件夹位置

1. 询问用户：
   > “请输入博客项目文件夹的完整路径（例如 ~/my-blog）：”
2. 等待用户回复，记录为 `$BLOG_PATH`。
3. 执行：
   ```bash
   bash scripts/init_site.sh "$BLOG_PATH"
   ```
4. 输出：
   > “节点 A1 完成：Hugo 站点已创建于 `$BLOG_PATH`。是否继续到节点 A2？(y/n)”
5. 等待用户输入 `y` 或 `n`。若为 `n`，终止流程。

---

#### 节点 A2：主题样式

1. 询问用户：
   > “请选择主题：
   > 1. PaperMod（推荐）
   > 2. LoveIt
   > 3. Stack
   > 4. 自定义（请输入 Git 仓库地址）”
2. 等待用户选择，确定 `$THEME_NAME` 和 `$THEME_REPO`。
3. 继续询问：
   > “请输入站点标题：”
   > “请输入 baseURL（格式：https://你的用户名.github.io/）：”
4. 记录 `$SITE_TITLE` 和 `$BASE_URL`。
5. 执行：
   ```bash
   bash scripts/install_theme.sh "$BLOG_PATH" "$THEME_NAME" "$THEME_REPO" "$SITE_TITLE" "$BASE_URL"
   ```
6. 输出：
   > “节点 A2 完成：主题已安装并写入配置。是否继续到节点 A3？(y/n)”
7. 等待确认。

---

#### 节点 A3：推送到 GitHub

1. 询问用户：
   > “请输入 GitHub 仓库地址（例如 https://github.com/user/repo.git）：”
2. 等待回复，记录 `$REPO_URL`。
3. 执行：
   ```bash
   bash scripts/push_github.sh "$BLOG_PATH" "$REPO_URL"
   ```
4. 输出：
   > “节点 A3 完成：代码已推送到 GitHub。是否继续到节点 A4？(y/n)”
5. 等待确认。

---

#### 节点 A4：启用 GitHub Pages

1. 输出指引：
   > “请打开以下链接，将 Source 改为 **GitHub Actions**：
   > https://github.com/<你的用户名>/<仓库名>/settings/pages
   > 完成后回复「已启用」。”
2. 等待用户回复“已启用”。
3. 收到后继续节点 A5。

---

#### 节点 A5：配置 GitHub Actions 自动部署

1. 询问用户：
   > “请输入 Hugo 版本号（默认 0.147.9，可运行 `hugo version` 查看）：”
2. 等待回复，记录 `$HUGO_VERSION`。
3. 执行：
   ```bash
   bash scripts/setup_actions.sh "$BLOG_PATH" "$HUGO_VERSION"
   ```
4. 输出：
   > “节点 A5 完成：GitHub Actions 自动部署已配置。
   > 推送后 Actions 将自动构建并发布。流程结束。”

---

### 流程 B：自定义修改

#### 节点 B1：需求确认

1. 询问用户：
   > “请描述您要修改的内容（例如：修改页脚版权、添加关于页面、调整菜单等）：”
2. 等待用户回复，记录需求 `$REQUIREMENT`。
3. 确认：
   > “您要修改的是：`$REQUIREMENT`。是否继续？(y/n)”
4. 等待确认。

---

#### 节点 B2：定位与 Diff

1. 根据需求，确定需要修改的文件。原则：
   - **内容**：`content/` 下的 Markdown 文件。
   - **布局/模板**：`layouts/` 下的 HTML 文件。若主题中有同名文件，先复制到项目 `layouts/` 对应路径，再修改副本。
   - **样式**：`static/css/` 下的 CSS 文件。
   - **配置**：根目录的 `hugo.yaml`。
2. 读取相关文件当前内容。
3. 生成修改方案和 diff，展示给用户：
   > “即将修改 `路径/文件`，变更如下：
   >
   > ```diff
   > - 旧内容
   > + 新内容
   > ```
   >
   > 是否确认修改？(y/n)”
4. 等待用户确认。若为 `n`，终止或重新调整。

---

#### 节点 B3：写入与验证

1. 用户确认后，执行修改（写入文件）。
2. 运行构建验证：
   ```bash
   cd "$BLOG_PATH" && hugo --minify
   ```
3. 若构建失败：
   - 报告错误信息。
   - 询问用户是否回滚（`git checkout -- .`）或手动修复。
4. 若构建成功：
   > “本地构建通过。是否启动本地预览？(y/n)”
5. 若用户选择预览，运行：
   ```bash
   cd "$BLOG_PATH" && hugo server -D
   ```
   并提示用户访问 `http://localhost:1313/` 查看效果。

---

#### 节点 B4：提交推送

1. 展示 `git diff` 摘要。
2. 询问：
   > “以上修改是否满意？确认后我将提交并推送到 GitHub。请回复提交信息（或直接回复「确认」使用默认信息）：”
3. 等待用户输入。
4. 执行：
   ```bash
   cd "$BLOG_PATH"
   git add .
   git commit -m "$COMMIT_MSG"
   git push
   ```
5. 输出：
   > “已推送。GitHub Actions 将自动构建并部署，约 1–2 分钟后博客更新。”

---

### 安全与确认机制

| 节点 | 确认方式 |
| :--- | :--- |
| A1 | 输入文件夹路径 → 执行后询问 `y/n` |
| A2 | 选择主题 → 输入标题和 baseURL → 执行后询问 `y/n` |
| A3 | 输入仓库地址 → 执行后询问 `y/n` |
| A4 | 用户手动在 GitHub 设置 → 回复“已启用” |
| A5 | 输入 Hugo 版本 → 执行完成，流程结束 |
| B1 | 描述需求 → 确认需求 |
| B2 | 展示 diff → 确认修改 |
| B3 | 构建验证 → 可选本地预览 |
| B4 | 展示 diff → 确认提交信息 → 推送 |

**关键原则**：

- 永远不要直接修改 `themes/` 目录，先复制到项目 `layouts/` 再改。
- 任何写入前必须展示 diff 并获得用户确认。
- 构建失败时不得推送，需先修复。
- 大改前建议创建分支：`git checkout -b customize`。
````

---

### 🔧 脚本文件

#### `scripts/check_env.sh`

```bash
#!/bin/bash
command -v hugo >/dev/null 2>&1 || { echo "❌ Hugo 未安装"; exit 1; }
command -v git  >/dev/null 2>&1 || { echo "❌ Git 未安装"; exit 1; }
echo "✅ 环境检查通过"
```

#### `scripts/init_site.sh`

```bash
#!/bin/bash
BLOG_PATH="$1"
hugo new site "$BLOG_PATH" --format yaml
cd "$BLOG_PATH" || exit 1
git init
echo "✅ 站点初始化完成"
```

#### `scripts/install_theme.sh`

```bash
#!/bin/bash
BLOG_PATH="$1"
THEME_NAME="$2"
THEME_REPO="$3"
SITE_TITLE="$4"
BASE_URL="$5"

cd "$BLOG_PATH" || exit 1
git submodule add "$THEME_REPO" "themes/$THEME_NAME"

cat > hugo.yaml <<EOF
baseURL: "$BASE_URL"
languageCode: "zh-cn"
title: "$SITE_TITLE"
theme: "$THEME_NAME"
EOF

echo "✅ 主题 $THEME_NAME 安装完成"
```

#### `scripts/push_github.sh`

```bash
#!/bin/bash
BLOG_PATH="$1"
REPO_URL="$2"

cd "$BLOG_PATH" || exit 1
git add .
git commit -m "Initial commit: Hugo blog"
git remote add origin "$REPO_URL" 2>/dev/null || git remote set-url origin "$REPO_URL"
git branch -M main
git push -u origin main

echo "✅ 推送完成"
```

#### `scripts/setup_actions.sh`

```bash
#!/bin/bash
BLOG_PATH="$1"
HUGO_VERSION="$2"

cd "$BLOG_PATH" || exit 1
mkdir -p .github/workflows

cat > .github/workflows/hugo.yaml <<EOF
name: Deploy Hugo site to Pages
on:
  push:
    branches: [main]
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
      HUGO_VERSION: $HUGO_VERSION
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O \${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v\${HUGO_VERSION}/hugo_extended_\${HUGO_VERSION}_linux-amd64.deb \\
          && sudo dpkg -i \${{ runner.temp }}/hugo.deb
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
          HUGO_CACHEDIR: \${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
        run: |
          hugo --minify --baseURL "\${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
  deploy:
    environment:
      name: github-pages
      url: \${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
EOF

git add .github/workflows/hugo.yaml
git commit -m "Add GitHub Actions workflow for auto deploy"
git push

echo "✅ GitHub Actions 配置完成"
```

---

### 🚀 使用方式

1. 将 `hugo-blog-manager` 文件夹放入 nanobot 的 `skills/` 目录。
2. 赋予脚本执行权限：
   ```bash
   chmod +x skills/hugo-blog-manager/scripts/*.sh
   ```
3. 对 nanobot 说：
   - **部署新博客**：“部署 Hugo 博客”
   - **修改现有博客**：“修改 Hugo 页脚”、“添加关于页面”等
4. nanobot 会自动加载 Skill，按对应流程逐步询问并执行，每个节点等待你的确认。