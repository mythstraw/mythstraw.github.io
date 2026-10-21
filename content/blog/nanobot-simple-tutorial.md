---
title: "nanobot 的简单教程"
date: 2026-10-14T21:58:00+08:00
slug: "nanobot-simple-tutorial"
description: "nanobot 是什么、有哪些优点，以及从 pip 安装、配置、启动到接入微信的极简操作流程。"
tags: [
    "nanobot",
    "随笔",
]
categories: [
    "ai-tech",
]
---

> *看过我的文章的应该会发现，nanobot 是我使用最多的 AI 工具：做 LLM-Wiki 知识库、搭博客网站、处理电脑问题、安装 GitHub 上的项目等等，基本上都是让它来操作。今天就来说说为什么我选择它，并且这么频繁地使用它。*

## nanobot 是什么

[Nanobot](https://nanobot.wiki/) —— nanobot 的官方 wiki 介绍。

简单来说，nanobot 是一个香港大学开发的**超轻量级开源 AI agent**，核心代码仅约 70000 行。它是简化版本的 openclaw（小龙虾），砍掉了 openclaw 复杂的网关和多消息渠道，保留了 Agent 循环、MCP 能力与基础记忆。

## nanobot 的优点

nanobot 的优点主要在以下几个方面：

- 轻量化，代码少，可以自定义（轻量服务器端也可以使用）
- 配置简单
- CLI 命令行、WebUI、多消息渠道（微信、QQ、飞书等都可以）、浏览器插件
- skill 和 MCP
- 多实例功能，可以用不同的实例处理不同的任务

## 我的使用感受

nanobot 作为日常的 AI 助手来说真的非常好用，安装配置操作都非常简单，该有的功能全都有：管理知识库、处理电脑问题、文件处理、远程控制电脑。而且网关可以静默后台开启，这样就可以全程使用手机来控制。

但是 nanobot 也有它的不足，它的多 agent 协作不行，虽然可以创建多个不同的实例，但实例之间是互相独立的，不能协作。如果看重多 agent 协作，建议还是使用 DeepSeek Harness 会更好一些。

## nanobot 极简操作流程

### 安装前你需要准备的东西

- Python 3.11 或更新版本。
- 访问一个支持的 AI 提供商、公司终端或本地模型服务器。

### pip 安装，配置

也可以查看 [nanobot wiki](https://nanobot.wiki/docs/0.3.5/getting-started/quick-start) 使用其他的安装方式。

#### 步骤 1：创建安装目录（可选）

为了便于管理，建议在用户目录下创建专门文件夹：

```powershell
# 在用户主目录创建 nanobot 文件夹
mkdir $HOME\nanobot
cd $HOME\nanobot
```

#### 步骤 2：安装 nanobot

```powershell
pip install nanobot-ai
```

等待安装完成，看到 `Successfully installed nanobot-ai` 表示成功。

#### 步骤 3：验证安装

```powershell
nanobot --version
```

应输出版本号，如 `nanobot 0.1.4.post3`。

#### 步骤 4：初始配置 nanobot

运行以下命令，nanobot 会自动创建配置文件目录：

```powershell
nanobot onboard
```

这个命令会：

- 创建 `~/.nanobot/` 目录
- 生成默认配置文件 `config.json`
- 创建工作空间目录

##### 编辑配置文件

配置文件位于：`C:\Users\你的用户名\.nanobot\config.json`

##### 打开配置文件

**方法 1：使用记事本**

```powershell
notepad $HOME\.nanobot\config.json
```

**方法 2：使用 VS Code（推荐）**

如果安装了 VS Code：

```powershell
code $HOME\.nanobot\config.json
```

**使用 DeepSeek 配置模板**

```json
{
  "providers": {
    "deepseek": {
      "apiKey": "sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    }
  },
  "agents": {
    "defaults": {
      "model": "deepseek-chat",
      "provider": "deepseek"
    }
  }
}
```

**说明：**

- 将 `apiKey` 替换为你的 DeepSeek API Key
- `model` 可选值：`deepseek-chat`（对话）、`deepseek-coder`（代码）

### 启动测试

#### 启动 Agent

```powershell
nanobot agent
```

启动后会进入交互式对话界面，你可以直接输入问题。

#### 测试示例

```text
你好，请介绍一下自己

帮我写一个 Python 函数，计算斐波那契数列

分析一下最近的人工智能发展趋势
```

#### 退出对话

输入以下任一命令退出：

- `exit`
- `quit`
- `/exit`
- `/quit`

### 微信连接

#### 步骤 1：启用微信渠道（第一次连接）

安装可选的渠道依赖：

```
nanobot plugins enable weixin
```

在 `config.json` 中添加：

```json
{
  "channels": {
    "weixin": {
      "enabled": true
    }
  }
}
```

不填写 `allowFrom` 会启用仅配对模式。新发送者的第一条微信私信只会收到配对码，还不能直接访问 Agent。

登录：

如果二维码识别不了，先使用下面命令，然后再登录：

```powershell
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
```

```
nanobot channels login weixin
```

如果需要丢弃已保存的登录状态并重新认证，添加 `--force`。

- 登录失败时，重新运行 `nanobot channels login weixin --force`。
- 第一条私信返回配对码是正常行为；先批准，再测试普通 Agent 回复。

##### 也可以直接在 WebUI 中设置

![在 WebUI 中设置微信渠道](/img/blog-nanobot-simple-tutorial/webui-weixin.png)

#### 步骤 2：启动网关（每次打开网关就可以连接机器人）

```powershell
nanobot channels status   # 查看连接状态
nanobot gateway
```

启动后，nanobot 会连接到微信，你可以通过微信与机器人对话。

#### 发送测试消息

给 Bot 发送一条微信私信。它应当返回配对码；从可信的本地入口批准：

```
nanobot agent -m "/pairing approve ABCD-EFGH"
```

批准后重新发送消息，同时观察网关日志中的发送者 ID 和回复。

> 📄 **延伸阅读**：[《nanobot 0.3.5 个人知识库自动部署指令》](/p/nanobot-deploy-instruction/) —— 把 nanobot 装成个人知识库助手的完整部署指令。

> 📚 **推荐阅读**：[nanobot 深度解析：6 个月 4.5 万 Star 的开源 AI Agent 是如何炼成的 | 官小西](https://guancyxx.cn/zh/blog/nanobot-deep-dive) —— 想进一步了解 nanobot 内部实现的可以看看。
