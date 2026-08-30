# auto_green

使用 GitHub Actions 每天创建一次空提交，用于保持仓库的每日提交记录。

## 特点

- 每天自动运行一次，也支持手动触发
- 使用 GitHub Actions 自带的 `GITHUB_TOKEN`，无需个人 Token
- 仅授予工作流所需的最小权限：每日提交使用 `contents: write`，清理任务使用 `actions: write`
- 提交邮箱使用 GitHub `noreply` 地址，不暴露个人邮箱
- 清理任务只处理 `Daily Commit` 自身的运行记录，不影响其他 Actions
- 默认保留最近 10 次已完成的 `Daily Commit` 运行记录

## 工作流

### Daily Commit

`.github/workflows/Daily Commit.yml` 默认每天 UTC 06:30 运行，也可以通过 GitHub Actions 页面手动触发。

执行流程：

1. 检出仓库
2. 配置 Git 提交者
3. 创建一个空提交
4. 使用 `GITHUB_TOKEN` 推送到当前分支

提交使用 GitHub 的 `noreply` 邮箱格式，避免在公开仓库中暴露个人邮箱。

### Cleanup Daily Commit runs

`.github/workflows/Cleanup old Actions runs.yml` 每 5 天运行一次，也支持手动触发。

它只查找名称为 `Daily Commit` 的 workflow，并保留最近 10 次已完成运行；正在运行的任务不会被删除。

## 注意

GitHub Actions 的定时任务可能存在延迟，因此 cron 时间不是严格的执行时间保证。

本项目创建的是空提交，适合学习 GitHub Actions 或个人实验。请合理使用自动化提交，真实、有意义的代码贡献仍然是最有价值的贡献。

## License

MIT
