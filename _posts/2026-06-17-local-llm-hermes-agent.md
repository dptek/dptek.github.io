---
layout: post
title: "本地部署 llama.cpp/Gemma 4 并接入 Hermes Agent 打造 Telegram 随身 AI 助理"
author: DPTEK
date: 2026-06-17 21:25:47 -0400
categories: Linux AI
image: assets/images/2026-06-17/hermes_ai.png
---

## 安装 llama.cpp-cuda（nVIDIA） 或者 llama.cpp-vulkan（AMD）

```bash  
paru llama.cpp-cuda
```

## 下载合适的大语言模型

在你的终端中直接复制并运行以下对应的命令即可：

---

### 1. `Qwen3-VL-8B-Thinking-abliterated.Q8_0.gguf`

```bash
python3 -c "from huggingface_hub import hf_hub_download; hf_hub_download(repo_id='mradermacher/Qwen3-VL-8B-Thinking-abliterated-GGUF', filename='Qwen3-VL-8B-Thinking-abliterated.Q8_0.gguf', local_dir='.')"
```

### 2. `gemma-4-12B-it-heretic.Q4_K_M.gguf`

*(注：请注意此仓库文件中的 `12B` 为大写)*

```bash
python3 -c "from huggingface_hub import hf_hub_download; hf_hub_download(repo_id='mradermacher/gemma-4-12B-it-heretic-i1-GGUF', filename='gemma-4-12B-it-heretic.Q4_K_M.gguf', local_dir='.')"
```

### 3. `gemma-4-12b-heretic-abliterated.i1-Q4_K_M.gguf`

```bash
python3 -c "from huggingface_hub import hf_hub_download; hf_hub_download(repo_id='mradermacher/gemma-4-12b-heretic-abliterated-i1-GGUF', filename='gemma-4-12b-heretic-abliterated.i1-Q4_K_M.gguf', local_dir='.')"
```

### 4. `gemma-4-E4B-it-uncensored-Q4_K_M.gguf`

*(注：原 TrevorJS 仓库中的连字符已为你修正校对)*

```bash
python3 -c "from huggingface_hub import hf_hub_download; hf_hub_download(repo_id='TrevorJS/gemma-4-E4B-it-uncensored-GGUF', filename='gemma-4-E4B-it-uncensored-Q4_K_M.gguf', local_dir='.')"
```

## 载入运行LLM
以 gemma-4-E4B-it-uncensored-Q4_K_M.gguf 为例：

```bash
llama-server -m ./gemma-4-E4B-it-uncensored-Q4_K_M.gguf -c 65536 --host 0.0.0.0 --port 8080 -ngl 99
```

## 打开llama.cpp的WebUI

如果可以访问并聊天就说明成功了  
[http://localhost:8080](http://localhost:8080)

Nous Research 最硬核的 **Hermes Agent** 自动化代理，配合本地无审查模型是一套极其强大的神器组合（能自动执行复杂的 Shell、编写 Skill、定时执行 Cron 任务）。这里为你补齐最地道的 Linux/CachyOS 本地及 VPS 环境下的安装与 Telegram 接入指南：

---

## 安装Hermes Agent

Hermes Agent 提供了一键安装脚本，它会全自动在你的系统里配置好 `uv`、Python 环境以及所有必要的底层依赖。

### 1. 一键执行安装

打开终端，运行官方提供的安装单行命令：

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

### 2. 初始化全量配置

安装完成后，你需要配置大模型提供商（由于你有本地跑的 `llama-server` 或者 `ollama`，可以直接在这一步关联你的本地地址 `http://127.0.0.1:8080/v1` 或 `http://127.0.0.1:11434/v1`）：

```bash
hermes setup
```

> 💡 **提示**：在弹出的配置 TUI 界面中，选择 **"Full Setup"**，在 Model Provider 里选择 `Ollama` 或 `OpenAI Compatible`（指向你的 `llama-server` 本地端口）。

### 3. 环境健康检查

配置完成后，运行以下命令，确保所有环境依赖（如 `ffmpeg`、`ripgrep` 等）全部亮起绿灯：

```bash
hermes doctor
```

---

## 接入Telegram

为了不需要一直守在 Linux 终端前，通过 Telegram 把它做成随身携带的 24 小时在线 AI 助理，我们需要通过网关（Gateway）将其无缝接入。

### 1. 申领 Telegram Bot 凭证

1. 在 Telegram 中搜索官方机器人 **`@BotFather`**，点击 Start。
2. 发送指令 `/newbot`，按照提示为你的 AI 助理起一个**昵称**（如：`My Hermes Admin`）和**用户名**（必须以 `_bot` 结尾，如：`my_hermes_3060_bot`）。
3. 保存 BotFather 随后发给你的 **`API Token`**（格式类似：`7123456789:AAH1bGciOi...`）。
4. **关键安全步骤**：在 Telegram 中搜索 **`@userinfobot`**，点击 Start，它会返回一串纯数字的 `Id`（例如 `987654321`）。**记下这个 ID**，我们要用它来锁死机器人，防止别人白嫖你的本地 3060 算力。

### 2. 配置 Hermes 网关

在终端中启动网关配置向导：

```bash
hermes gateway setup
```

在交互界面中：

* 选择 **Telegram**。
* 粘贴刚刚获得的 **Bot Token**。
* 在 **Allowed Users（允许的用户）** 选项中，填入刚才获取的你的 **Telegram User ID**。这样除了你以外，任何人都无法唤醒这个 Bot。

### 3. 守护进程持久化运行

安装好后检测 Hermes Gateway 服务的运行状态：

```bash
systemctl --user status hermes-gateway
```

下面显示用户服务正在运行：

```
● hermes-gateway.service - Hermes Agent Gateway - Messaging Platform Integration
     Loaded: loaded (/home/dptek/.config/systemd/user/hermes-gateway.service; enabled; preset: enabled)
     Active: active (running) since Wed 2026-06-17 18:43:13 EDT; 2h 18min ago
 Invocation: 4dd4b01fb197470a8024ef118258dd06
   Main PID: 1072 (hermes)
      Tasks: 6 (limit: 38123)
     Memory: 233.6M (peak: 306.3M)
        CPU: 13.835s
     CGroup: /user.slice/user-1000.slice/user@1000.service/app.slice/hermes-gateway.service
             └─1072 /home/dptek/.hermes/hermes-agent/venv/bin/python -m hermes_cli.main gateway run
```

现在，掏出你的手机，打开 Telegram 找到你刚才自己创建的 Bot，发送一句 `/start` 或 `Hi`。你的本地无审查 Gemma-4 / Qwen 战队就已经通过 Telegram 正式向你报到了！
