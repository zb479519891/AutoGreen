# auto_green

[![Daily Commit](https://github.com/zb479519891/auto_green/actions/workflows/Daily%20Commit.yml/badge.svg)](https://github.com/zb479519891/auto_green/actions/workflows/Daily%20Commit.yml)
[![Cleanup old Actions runs](https://github.com/zb479519891/auto_green/actions/workflows/Cleanup%20old%20Actions%20runs.yml/badge.svg)](https://github.com/zb479519891/auto_green/actions/workflows/Cleanup%20old%20Actions%20runs.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

使用 GitHub Actions 自动创建每日空提交，让仓库保持持续的 Git 提交记录。

本项目无需个人 Access Token，通过 GitHub Actions 自带的 `GITHUB_TOKEN` 完成提交和推送。

> **说明：** 本项目主要用于学习 GitHub Actions、Git 自动化以及个人实验。自动提交不代表真实、有意义的代码贡献。

## ✨ 功能

- 📅 每天自动创建一次空提交
- ▶️ 支持在 GitHub Actions 页面手动触发
- 🔐 使用 GitHub Actions 内置的 `GITHUB_TOKEN`
- 📦 使用 `contents: write` 权限推送提交
- 📧 使用 GitHub `noreply` 邮箱，不暴露个人邮箱
- 🧹 定期清理旧的 GitHub Actions 运行记录
- ⚡ 使用 Concurrency 避免同一任务重复并发运行

## 🔄 工作原理

项目包含两个 GitHub Actions 工作流：

```text
┌──────────────────────┐
│     Daily Commit     │
│   每天 UTC 06:30     │
└──────────┬───────────┘
           │
           ▼
     Checkout 仓库
           │
           ▼
      创建空提交
           │
           ▼
        git push
           │
           ▼
      Git 提交记录

┌──────────────────────────────┐
│   Cleanup old Actions runs   │
│      每 5 天 UTC 08:00       │
└──────────────┬───────────────┘
               │
               ▼
       查询 Actions Runs
               │
               ▼
      保留最近 5 次已完成运行
               │
               ▼
          删除旧记录
```

### 1. Daily Commit

工作流文件：

```text
.github/workflows/Daily Commit.yml
```

默认每天 **UTC 06:30** 自动运行一次，同时支持手动触发。

```yaml
on:
  schedule:
    - cron: "30 6 * * *"
  workflow_dispatch:
```

执行流程：

1. 检出仓库
2. 配置 Git 提交者
3. 创建空提交
4. 使用 `GITHUB_TOKEN` 推送到当前分支

核心命令：

```bash
git commit --allow-empty -m "chore: daily commit"
git push
```

由于使用 `--allow-empty`，即使仓库没有文件变化，也可以正常创建提交。

### 2. Cleanup old Actions runs

工作流文件：

```text
.github/workflows/Cleanup old Actions runs.yml
```

默认每 **5 天 UTC 08:00** 运行一次，同时支持手动触发。

该任务通过 GitHub CLI 查询当前仓库的 Actions 运行记录，并删除较早的**已完成运行记录**。

当前配置：

```yaml
env:
  KEEP_RECENT: 5
```

也就是说，任务会按照创建时间从新到旧排序，并保留最近 **5 次已完成的 Workflow Run**。

正在运行、排队或等待中的任务不会被删除。

> **注意：** Cleanup 工作流针对的是当前仓库的 Actions 运行记录整体，并不只针对 `Daily Commit`。

## 🔐 权限

两个工作流分别使用不同的最小权限。

### Daily Commit

```yaml
permissions:
  contents: write
```

用于创建 Git commit 并推送到仓库。

### Cleanup old Actions runs

```yaml
permissions:
  actions: write
```

用于查询并删除旧的 GitHub Actions 运行记录。

项目不需要配置个人 GitHub Token。

## 🚀 快速开始

### 1. Fork 项目

Fork 本仓库到你的 GitHub 账号。

### 2. 检查 Actions 权限

进入：

```text
Settings → Actions → General
```

确认仓库允许 GitHub Actions 运行，并允许 Workflow 使用必要的写入权限。

### 3. 修改提交身份

如果你是 Fork 使用，请检查：

```text
.github/workflows/Daily Commit.yml
```

当前工作流使用：

```bash
git config user.name "zb479519891"
git config user.email "25305863+zb479519891@users.noreply.github.com"
```

建议修改为你自己的 GitHub 用户名和对应的 `noreply` 邮箱。

### 4. 手动测试

进入：

```text
Actions → Daily Commit → Run workflow
```

确认 Workflow 成功后，再等待定时任务自动运行。

清理任务也可以通过：

```text
Actions → Cleanup old Actions runs → Run workflow
```

手动执行。

## 🖱️ 手动运行

两个工作流都包含：

```yaml
workflow_dispatch:
```

因此可以随时从 GitHub Actions 页面手动触发，无需等待 Cron。

## ⏰ 定时任务

GitHub Actions 的 `schedule` 使用 **UTC 时间**。

| Workflow | Cron | 频率 |
|---|---|---|
| Daily Commit | `30 6 * * *` | 每天 UTC 06:30 |
| Cleanup old Actions runs | `0 8 */5 * *` | 每 5 天 UTC 08:00 |

GitHub Actions 的定时任务可能受到系统负载等因素影响，因此 Cron 时间不是严格的实际启动时间保证。

## 🧹 自定义清理数量

编辑：

```text
.github/workflows/Cleanup old Actions runs.yml
```

修改：

```yaml
env:
  KEEP_RECENT: 5
```

例如保留最近 20 次已完成运行：

```yaml
env:
  KEEP_RECENT: 20
```

## 🕐 自定义每日提交时间

编辑：

```text
.github/workflows/Daily Commit.yml
```

修改 Cron，例如每天 UTC 00:00：

```yaml
schedule:
  - cron: "0 0 * * *"
```

Cron 使用 UTC 时间。

## 📌 关于空提交

本项目使用：

```bash
git commit --allow-empty
```

因此不会修改项目文件，只会产生一个新的 Git commit。

默认提交信息：

```text
chore: daily commit
```

这种方式适合：

- 学习 GitHub Actions
- 学习 Git 自动化
- 测试自动提交流程
- 个人 GitHub Actions 实验

请合理使用自动化提交。真实、有意义的代码贡献仍然比单纯增加提交记录更有价值。

## ⚠️ 注意事项

### Actions 必须保持启用

如果仓库的 GitHub Actions 被禁用，定时任务将无法运行。

### 确认 Workflow 具有写权限

`Daily Commit` 需要：

```yaml
permissions:
  contents: write
```

如果仓库或组织级别的 Actions 策略禁止 Workflow 写入仓库，`git push` 可能失败。

### Fork 后检查提交身份

当前工作流中的 Git 用户信息是项目作者的账号信息。如果 Fork 后不修改，提交可能不会按照你的预期归属到自己的 GitHub 账号。

请修改为自己的 GitHub 用户名及对应的 `noreply` 邮箱。

### Cleanup 会删除旧的 Actions 运行记录

Cleanup 工作流具有：

```yaml
permissions:
  actions: write
```

并会删除较早的已完成 Workflow Run。如果你需要完整保留 Actions 历史记录，可以关闭或调整该工作流。

## 📁 项目结构

```text
auto_green/
├── .github/
│   └── workflows/
│       ├── Daily Commit.yml
│       └── Cleanup old Actions runs.yml
├── .gitattributes
├── .gitignore
├── LICENSE
└── README.md
```

## 📄 License

本项目采用 [MIT License](LICENSE) 开源。

---

如果这个项目对你有帮助，欢迎 ⭐ Star。
