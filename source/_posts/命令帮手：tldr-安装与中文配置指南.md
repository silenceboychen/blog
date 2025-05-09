---
title: 命令帮手：tldr 安装与中文配置指南
toc: true
date: 2025-05-09 13:39:38
categories: shell
tags: shell
---


在日常开发与运维过程中，你是否会觉得传统的 man 手册过于繁琐？[tldr](https://tldr.sh/) 项目正是为了解决这个问题而生，它为上百个常用命令提供了简明直观的示例，极大地提升了查阅效率。本文将手把手介绍 tldr 在 Windows、Mac、Ubuntu 下的安装流程，基本使用方法，以及如何让 tldr 输出为简体中文。

## 一、tldr 安装方法

### 1. Windows 系统

#### 方法一：用 Scoop 安装（推荐）

1. 安装 [Scoop](https://scoop.sh/)（如果还没装）：
   ```powershell
   Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
   irm get.scoop.sh | iex
   ```

2. 安装 tldr：
   ```powershell
   scoop install tldr
   ```

#### 方法二：用 npm 安装

需先装好 [Node.js](https://nodejs.org/) 与 npm，然后在命令行执行：
```powershell
npm install -g tldr
```

#### 方法三：用 pip 安装

需先装好python与pip，然后在命令行执行：
```powershell
pip3 install tldr
```

### 2. macOS 系统

推荐使用 [Homebrew](https://brew.sh/)：

```bash
brew install tldr
```

或者用 npm：

```bash
npm install -g tldr
```

或者用 pip：

```bash
pip3 install tldr
```

### 3. Ubuntu / Debian 系统

推荐方式（snap 安装）：

```bash
sudo snap install tldr
```

或者用 apt（部分老版本仓库没有）：

```bash
sudo apt update
sudo apt install tldr
```

还可以用 npm：

```bash
npm install -g tldr
```

或者用 pip：

```bash
pip3 install tldr
```

## 二、tldr 的基本使用

安装好 tldr 后，在命令行中直接输入：

```bash
tldr 命令名
```

例如：
```bash
tldr tar
```

它会显示 tar 常用的简要用法和示例。

常用选项：

- `tldr -u`
  强制更新离线文档缓存。
- `tldr --list`
  查看有哪些命令有 tldr 页面。
- `tldr --help`
  查看 tldr 的详细帮助信息。


## 三、配置 tldr 显示中文页面

tldr 官方已支持多语言，目前大部分主流命令已有简体中文文档。只需调整默认语言环境变量即可。

### 1. 临时使用中文

只对当前命令生效：

```bash
LANG=zh tldr ls
```
或（部分 tldr 客户端支持）
```bash
LANGUAGE=zh tldr ls
```

### 2. 永久设置为中文

#### macOS / Ubuntu / WSL

将下方内容添加到 `~/.bashrc` 或 `~/.zshrc`（具体看你用哪个 shell）：

```bash
export LANG=zh
```
保存后，执行一次：
```bash
source ~/.bashrc    # 或 source ~/.zshrc
```
这样每次开新终端，tldr 默认输出中文页面。

#### Windows（CMD 或 PowerShell）

**CMD 中临时生效：**
```cmd
set LANG=zh
tldr ls
```
**长期生效**：
在系统环境变量或用户环境变量中添加 `LANG`，值填 `zh`。

### 3. 更新缓存（非常重要）

修改语言后，强烈建议刷新缓存让中文页面生效：

```bash
tldr -u
```

### 4. 验证效果

运行：
```bash
tldr cp
```
如果出现中文简明用法，表明配置成功。

## 四、常见问题

- **部分命令还是英文？**
  可能该命令尚未有中文翻译。同样建议及时更新缓存。
- **未知命令提示或无法联网？**
  检查网络或采用 `tldr --update` 手动补全离线缓存。
- **Windows 环境变量生效问题？**
  尝试重启终端或电脑，确认语言变量设置无误。


## 五、总结

tldr 是极简而实用的命令行速查工具，无论软件开发者还是 Linux/Unix/Windows 运维者都能从中获益。通过简单的安装和配置，你就能轻松查阅常见命令的简明用法，还能设为汉语输出，降低理解难度，为高效开发助力。



> **扩展阅读：**
> - 官方网站：[https://tldr.sh/](https://tldr.sh/)
> - GitHub 项目：[https://github.com/tldr-pages/tldr](https://github.com/tldr-pages/tldr)