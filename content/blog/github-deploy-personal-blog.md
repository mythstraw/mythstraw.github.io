---
title: "GitHub 部署自己的个人博客网站"
date: 2026-09-27T17:50:00+08:00
slug: "github-deploy-personal-blog"
description: "把知识放进自己的仓库：Hugo + Git + GitHub + Cloudflare 的完整搭建思路，含 Hugo 目录结构速查、Cloudflare Workers 部署设置、GitHub Pages 自定义域名绑定顺序。"
tags: [
    "Hugo",
    "GitHub",
    "Cloudflare",
]
---

*在这个 AI 时代，知识库成了最火的项目。AI 需要知识库来提升专业度，人类需要知识库辅助学习和记忆。但有一个问题是：互联网上有什么是可以永久留存的呢？所有的服务都会随着运营商的倒闭而面临关闭，你用来保存知识库的载体也会随着消失。而目前来看，唯一能够长久存在可能也就只有这个全球最大的仓库 GitHub 了，所以把知识放在 GitHub 上成了很多人的首选方案。*

> 想要了解整个详细流程，可以查看 [使用 Hugo + GitHub 部署博客](/p/hugo-github-deploy/) 这个详细的文档。

## 使用的技术

- **Hugo**：静态 blog 网站工具
- **Git** 工具
- **GitHub** 仓库
- AI 工具：**nanobot**
- **Cloudflare** 部署
- **GitHub 自定义域名**

## 简化的流程说明（AI 自动操作）

1. **本地环境搭建**：安装 Hugo 和 Git。
2. **创建与配置站点**：初始化 Hugo 站点，选择并配置主题样式。
3. **编写与预览内容**：用 Markdown 写文章，本地启动服务预览。
4. **连接 GitHub 并自动化部署**：将代码推送到 GitHub，配置 Actions 自动构建和发布。

## 修改美化 Hugo Blog

先了解一下 Hugo 的整体框架，了解了框架才能知道 AI 改了哪些东西。使用不同的主题，架构可能有一些不同，我们只要了解基础架构，细节就交给 AI 自己去学习。

![Hugo 站点的目录结构](/img/blog-github-deploy/hugo-structure.png)

### 📁 核心内容与模板（你需要经常编辑的）

- **`content/`**：存放你的博客文章。所有的 Markdown 文件（如 `posts/my-first-post.md`）都在这里。
- **`layouts/`**：存放自定义的 HTML 模板。当你需要覆盖主题的页面结构（如修改页脚、首页样式）时，把主题里的文件复制到这里修改。
- **`static/`**：存放不需要 Hugo 处理的静态文件。比如自定义的 CSS、JS、图片、favicon 等，Hugo 会原样复制到最终的 `public/` 目录。
- **`assets/`**：存放需要 Hugo 处理（如压缩、编译 SCSS）的文件。通常由主题使用，你也可以在这里放需要打包的 CSS/JS。
- **`archetypes/`**：存放内容模板（原型）。当你执行 `hugo new posts/xxx.md` 时，Hugo 会以此处的模板生成文章的初始 Front Matter。
- **`data/`**：存放数据文件（JSON/YAML/TOML），供模板调用，常用于生成列表或配置信息。
- **`i18n/`**：存放多语言翻译文件。如果你的博客是中文，这里可能主要用于覆盖主题的默认语言字符串。

### ⚙️ Hugo 配置与构建（自动生成或较少改动）

- **`themes/`**：存放你安装的 Hugo 主题（如 PaperMod 或 BearNeo）。**建议永远不要直接修改这个文件夹内的文件**，以免更新主题时被覆盖。
- **`hugo.yaml`**：Hugo 站点的**主配置文件**。你的站点标题、baseURL、菜单、主题参数都在这里设置。
- **`public/`**：Hugo 构建后生成的**静态网站最终输出目录**。部署时，GitHub Actions 或 Cloudflare 实际上就是把这里的文件发布出去。
- **`resources/`**：Hugo 的缓存目录。存放图片处理、SCSS 编译等生成的中间文件。
- **`.hugo_build.lock`**：Hugo 构建时的锁文件，防止多个构建任务同时修改文件。通常不需要管它，会自动生成。

### 🔀 版本控制与自动化（Git 相关）

- **`.git/`**：Git 的本地仓库数据。不要手动修改里面的文件。
- **`.github/`**：存放 GitHub Actions 工作流。里面有 `workflows/hugo.yaml`，是你推送到 GitHub 后自动部署到 GitHub Pages 的配置文件。
- **`.gitignore`**：告诉 Git 哪些文件和文件夹不需要上传（例如 `public/`、`resources/`、`.hugo_build.lock` 通常会被忽略）。
- **`.gitmodules`**：如果你用 `git submodule` 安装了主题，这个文件记录了主题的仓库地址。由于你用了主题，这个文件很重要，需要在部署时用 `submodules: recursive` 拉取。

### ☁️ 部署与托管（Cloudflare 与 GitHub Pages）

- **`.wrangler/`**：Cloudflare Wrangler CLI 的本地缓存目录。通常不需要手动修改。
- **`wrangler.jsonc`**：Cloudflare Workers 的配置文件。告诉 Cloudflare 项目名称、构建命令（`hugo build --gc --minify`）以及静态资源目录（`./public`）。
- **`CNAME`**：用于 GitHub Pages 绑定自定义域名的文件（通常包含你的域名）。如果你同时部署到了 Cloudflare 和 GitHub，这个文件会指定 GitHub Pages 的访问域名。

**你只要知道这些文件是干什么的、修改的东西要到哪里去查看就可以了，至于操作的事情交给 AI。**

![BearNeo 主题里，每一个导航就是一个 Markdown 文档](/img/blog-github-deploy/bearneo-nav.png)

我使用的主题是 BearNeo，它是一个极简的主题，每一个导航其实就是一个 MD 文档，你也可以自己去打开文档去编辑内容。

![Hugo 的 HTML 模板文件](/img/blog-github-deploy/html-template.png)

**如果想要做很大的样式上的修改，那就需要去改 HTML 模板了，当然你也可以让 AI 去修改。**

## Cloudflare 部署（了解一下，国内访问有限制）

Cloudflare 有两种部署方式 pages 和 workers，现在 Cloudflare 开始转向 workers，pages 现在是停止维护但是仍然可以使用的状态。

这里主要说明 workers 的部署过程。

### 选 Workers：具体设置

**① 仓库里新增 `wrangler.jsonc`**（这是唯一的仓库改动，AI 会帮你自动完成）

```json
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "foreveryang-blog",
  "compatibility_date": "2026-09-27",
  "assets": {
    "directory": "./public",
    "not_found_handling": "404-page"
  }
}
```

- `name` 必须和控制台里的 Worker 名**完全一致**，否则构建直接失败（官方明确警告）。
- `compatibility_date` 填你创建那天的日期。
- `not_found_handling` 默认是 `"none"`（不存在路径只给一句 404）→ 必须写成 `"404-page"`，才会用你 `404.html` 生成的那个自定义 404 页。**Pages 会自动探测 `404.html`，Workers 不会**，这是从 Pages 思路切过来最容易漏的一项。
- 不用写 `html_handling`，默认 `auto-trailing-slash` 正好适配 Hugo 的 `/p/slug/` 这种漂亮 URL。
- 纯静态、没有 Worker 脚本 → **不要**写 `"binding": "ASSETS"`（那是配合 `main` 用的）。

**② 控制台（Workers & Pages → Create application → Continue with GitHub）**

接下来的流程：

1. **Continue with GitHub** → 弹 GitHub 授权页 → 授权 Cloudflare 的 GitHub App（仓库权限建议选 `Only select repositories`，只勾 `xxxxxx.github.io`）
2. **回到 Cloudflare 选仓库** → `xxxxxx.github.io`
3. **Build command** 填：

```
git submodule update --init --recursive && hugo --minify --gc ${HUGO_BASEURL:+--baseURL "$HUGO_BASEURL/"}
```

4. **Advanced settings** 填：

| 项 | 值 |
| --- | --- |
| Root directory | 留空 |
| Build variables and secrets | `HUGO_VERSION` = `0.154.5`（**必填**）；`HUGO_BASEURL` = `https://foreveryang.xxxxx.workers.dev/`（不知道就先不建） |
| API token | 保持默认（Create new token） |

5. **两个开关**

- **Enable Preview builds** → 保持默认开启。它只对“非 main 分支”生效，你平时只推 main，不额外耗构建分钟；以后开分支试版式就能拿预览 URL。
- **Protect with Cloudflare Access** → **必须关**。这是给内部/私密站点套 Zero Trust 登录的，开了访客要先登录才能看博客。

**③ 点部署后，日志里核对三行**

1. Hugo 版本 = `0.154.5`（证明 `HUGO_VERSION` 生效，没退到镜像默认的 `0.147.7`）；
2. 没有 `Unable to find theme "hugo-bearneo"`（submodule 拉下来了）；
3. 末尾 `wrangler deploy` 成功，并给出 `https://xxxxxxxx.<子域>.workers.dev`。

之后就是把域名发给 AI，让它确认就可以了。Cloudflare 的 workers 域名国内网络访问不了，可以自行使用自定义域名。

## GitHub 怎么自定义域名

### 关键顺序（别做反）

GitHub 官方明确要求：**先在仓库里填好域名，再去 DNS 加记录**，反了有子域名被人抢注的风险。

**① GitHub（先做）**

`https://github.com/mythstraw/mythstraw.github.io/settings/pages` → Custom domain 填：

```
blog.foreveryang.top
```

→ Save

**② 腾讯云 DNSPod（后做）** 添加记录：

| 主机记录 | 记录类型 | 记录值 | 线路 | TTL |
| --- | --- | --- | --- | --- |
| `blog` | `CNAME` | `xxxxxx.github.io` | 默认 | 600 |

> 记录值不带仓库名、也不要填 Settings 里那个 `*.pages.github.io` 的临时地址。

**③ 生效后**（几分钟 ~ 24h）回 Pages 页勾 **Enforce HTTPS**（证书签发还要几分钟）。

**④ 让 AI 自行检查去触发一次构建**：仓库 Actions → _Deploy Hugo site to Pages_ → Run workflow（或随便 push 一篇文章）。构建时 `configure-pages` 会把 baseURL 自动换成新域名，canonical / `index.xml` / favicon 就都是 `blog.foreveryang.top` 了。
