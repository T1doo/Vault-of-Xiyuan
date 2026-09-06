# 当前状态

- 2026-09-06：文档已整理至 `Vault-of-Xiyuan/docs/`，路径说明已同步。
- 仓库准备：DONE / TESTED；已创建并发布 [T1doo/Vault-of-Xiyuan](https://github.com/T1doo/Vault-of-Xiyuan)，GitHub 已确认可见性为 PUBLIC，默认分支为 main。
- 科研实施：TODO / PLANNED；尚未执行 T00 资源审计、安装环境或启动实验。
- 活跃实验作业：本次未启动任何作业。
- 下一步：按负责人指令推进；开始科研实施时先执行 T00 只读审计。

验证：`git diff --cached --check`、`git push -u origin main` 和 `gh repo view T1doo/Vault-of-Xiyuan --json url,visibility,owner,defaultBranchRef` 均成功（退出码 0）。共享目录归属检查通过临时 `GIT_CONFIG_GLOBAL=/tmp/xiyuan-gitconfig` 处理，未修改用户全局 Git 配置。
