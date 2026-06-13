---
title: Mac 本地工具链基线（Homebrew）
created: 2026-06-13
updated: 2026-06-13
type: system
domain: hermes-ops
tags: [hermes, tooling, macos, homebrew, shell, environment]
sources: []
confidence: high
---

# Mac 本地工具链基线（Homebrew）

> 适用：Mac mini（macOS 26.5.1, Apple Silicon arm64, 用户 yifan）。
> 范围：Homebrew 6.0.1 + 7 个常用 shell/脚本 formula。
> 状态：2026-06-13 安装并通过 `brew doctor`；brew 升级或新增 formula 时更新本页面。

## 1. Homebrew 基线

| 项 | 值 |
|---|---|
| 版本 | Homebrew 6.0.1 |
| 前缀 | `/opt/homebrew` |
| Cellar | `/opt/homebrew/Cellar` |
| Sudo 要求 | 是（Apple Silicon 上 brew installer 需要 admin 提权创建目录） |
| 安装方式 | 官方 `install.sh`，需 `yifan` 是 Administrator |
| 已知坑 | 不要在 `sudo` 后再调用 brew，否则触发 "Don't run this as root!"；让脚本自管 sudo |

## 2. PATH 配置

`~/.zprofile` 末尾追加：

```bash
eval "$(/opt/homebrew/bin/brew shellenv zsh)"
```

完整 `.zprofile`：

```bash
# Hermes Agent — login shell PATH.
export PATH="$HOME/.hermes/bin:$HOME/.hermes/node/bin:$HOME/.local/bin:$PATH"

eval "$(/opt/homebrew/bin/brew shellenv zsh)"
```

PATH 解析顺序（实测 2026-06-13）：

1. `/opt/homebrew/bin` ← brew 优先
2. `/opt/homebrew/sbin`
3. `/usr/local/bin`
4. `/usr/bin` ← macOS 系统
5. ...

`brew doctor` 输出：`Your system is ready to brew.`（零警告）。

## 3. 已安装 formula

| Formula | 命令 | 路径 | 版本 | 备注 |
|---|---|---|---|---|
| `jq` | `jq` | `/opt/homebrew/bin/jq` | jq-1.8.1 | 高于系统 `/usr/bin/jq` (1.7.1) |
| `yq` | `yq` | `/opt/homebrew/bin/yq` | v4.53.3 | mikefarah/yq（Go 版）|
| `ripgrep` | `rg` | `/opt/homebrew/bin/rg` | ripgrep 15.1.0 | ⚠ 包名 ≠ 命令名（见 4.1）|
| `fd` | `fd` | `/opt/homebrew/bin/fd` | fd 10.4.2 | |
| `bat` | `bat` | `/opt/homebrew/bin/bat` | bat 0.26.1 | 依赖 openssl@3、libssh2、llhttp、libgit2 |
| `eza` | `eza` | `/opt/homebrew/bin/eza` | eza 0.23.4 | `ls` 现代替代 |
| `htop` | `htop` | `/opt/homebrew/bin/htop` | htop 3.5.1 | ⚠ 看全部进程需 sudo（见 4.2）|

## 4. 已知坑

### 4.1 `ripgrep` 包 → `rg` 命令
- 包名：`ripgrep`
- 命令名：`rg`
- 验证：`which rg` → `/opt/homebrew/bin/rg`
- 后续脚本统一用 `rg`，不要写 `ripgrep`

### 4.2 `htop` 看全部进程需 `sudo`
- 普通模式只能看当前用户进程（macOS 限制）
- 看全部：`sudo htop`
- brew 层面无修复

### 4.3 `jq` brew 版 vs 系统版
- 系统：`/usr/bin/jq` 1.7.1（root 拥有，2025-05-21）
- Brew：`/opt/homebrew/bin/jq` 1.8.1
- 当前 shell `which jq` → brew 版本（PATH 中 brew 在前），无问题
- sudo 环境或脚本中显式 PATH 时可能临时调用系统版；用绝对路径或 `command -v jq` 校验

## 5. 维护动作

### 升级 brew 自身
```bash
brew update
brew upgrade
```

### 升级所有 formula
```bash
brew upgrade
```

### 新增 formula（更新第 3 节表格）
```bash
brew install <formula-name>
which <command-name>
<command-name> --version
```

### 验证整个基线
```bash
brew doctor
for cmd in jq yq rg fd bat eza htop; do
  printf "%-6s -> %s | %s\n" "$cmd" "$(which $cmd)" "$($cmd --version | head -1)"
done
```

## 6. 验证记录

- 2026-06-13：`brew install jq yq ripgrep fd bat eza htop` 全部成功
- 2026-06-13：`brew doctor` → `Your system is ready to brew.`
- 2026-06-13：7 个命令全部 `--version` 验证通过（见第 3 节表格）
- 2026-06-13：zsh completions 自动装到 `/opt/homebrew/share/zsh/site-functions`

## 7. 后续候选（按需）

```bash
# Python 多版本
brew install pyenv

# Node 多版本
brew install fnm

# 音视频
brew install ffmpeg imagemagick

# 其余 shell 增强
brew install shellcheck tree watch tmux
```