# 当前状态

## 最新交接：2026-09-06，V3 审阅补充

- 当前 F0 / T00：IN_PROGRESS。文档审阅和源码静态核对完成；环境、数据、模型、仿真尚未验证，G1—G5 均未通过。以下早期记录为历史快照。
- 负责人已明确：多人共享服务器，两卡只是可用并发预估；无硬截止日；无已有基础数据/项目环境，由 Codex 自行下载准备。无需再询问已有资源路径或截止日；按实际空闲窗口推进，不扩大为八卡使用。
- 已完整读取负责人提供的公开分享回复（八部分建议，6944 字符；仅该条公开回复，不是未分享的完整聊天）。落实到两份主文档的范围见 DECISIONS；四组、三 seed、七维、20k 默认值、损失权重、教师和学生层位未改变，无第五个模型或新功效 gate。
- 已下载官方 openpi、FastVGGT、LIBERO、LIBERO-plus 源码至工作区 `upstream/`，版本见下表。四个 clone 均退出 0，记录时工作树干净；openpi 子模块未初始化，不能称可运行环境。没有下载训练 HDF5/模型权重或安装项目环境。
- 本次文档变更：EXPERIMENT_PLAN、CODEX_EXECUTION_GUIDE、STATUS、DECISIONS、RUNBOOK、README；此前未提交的服务器审阅记录一并保留。拟推送该文档修订供 GPT 复审，推送结果以实际 Git 返回及远端核验为准，不在提交前声称发布成功。
- 本机原始审计/分享页面解析材料保留在公开仓库外。公开仓库只包含文档与来源/版本索引，不包含原始申请书、数据、权重、环境或凭据。
- 活跃科研作业/自动队列：无；无 GPU lease、checkpoint、有效更新或缓存分片；本次未跑模型/仿真/吞吐测试。科学协议未锁定，无既有实验产物失效。
- 恢复点：本组文档与工作区 `artifacts/audits/scientific_source_inventory_20260906.json`，不是训练检查点。下一任务：复审后继续 T00/T01 环境准备与官方数据获取，F1 优先做 T02/T08 初态关系库存；所需人工 QC 仍按阶段准备，未签收。
- 已验证复查命令：RUNBOOK 中 Git 状态与宿主机资源查询。正式训练/下载包装 CLI 尚不存在；不能执行文档中的拟新增命令。
- 发布前验证：6 个 Markdown 文件的代码围栏、相对链接、分享引用清理、两文档补充项及核心 YAML 默认值检查通过；`git diff --check` 退出 0。远端 fetch 成功，提交前 HEAD 与 origin/main 无分歧（0/0）。这是文档验证，不是实验测试。

| 官方源码 | 本地静态审阅提交 |
|---|---|
| openpi | `215abfb217dbac7d5f1273282331b9b1866c0479` |
| FastVGGT | `6526e275a29572653a034762bb3c6c9ce280ff55` |
| LIBERO | `8f1084e3132a39270c3a13ebe37270a43ece2a01` |
| LIBERO-plus | `4976dc30028e805ff8094b55501d532c48fec182` |

## 历史：仓库准备与首次现场审阅

- 2026-09-06：文档已整理至 `Vault-of-Xiyuan/docs/`，路径说明已同步。
- 仓库准备：DONE / TESTED；已创建并发布 [T1doo/Vault-of-Xiyuan](https://github.com/T1doo/Vault-of-Xiyuan)，GitHub 已确认可见性为 PUBLIC，默认分支为 main。
- 科研实施：TODO / PLANNED；尚未执行 T00 资源审计、安装环境或启动实验。
- 活跃实验作业：本次未启动任何作业。
- 下一步：按负责人指令推进；开始科研实施时先执行 T00 只读审计。

验证：`git diff --cached --check`、`git push -u origin main` 和 `gh repo view T1doo/Vault-of-Xiyuan --json url,visibility,owner,defaultBranchRef` 均成功（退出码 0）。共享目录归属检查通过临时 `GIT_CONFIG_GLOBAL=/tmp/xiyuan-gitconfig` 处理，未修改用户全局 Git 配置。

## 2026-09-06：文档通读与服务器适配审阅

- F0 / T00：IN_PROGRESS；本次完成全部现存 docs 文档及工作区 AGENTS.md 阅读和部分只读现场审计，不标 T00 全部完成。上面的“尚未执行 T00”为本次审阅前历史状态。
- 入口分支 `main`，入口提交 `c50040af2f1e4ffeb1c01f03699a206171aff8b6`；开始时工作树干净。无仓库内追加 AGENTS.md；RUNBOOK 本次创建。
- 宿主机查询确认 8 张 RTX A6000，每张总显存 49140 MiB；首次查询各卡空闲约 48661—48664 MiB、利用率 0%，计算进程查询为空。驱动 535.274.02。这里只是瞬时资源快照，不是项目独占额度或吞吐测试。
- CPU 为双路 EPYC 7542，共 64 物理核/128 逻辑 CPU；内存约 503 GiB。工作区为 NFS，文件系统可用约 26 TiB；/tmp 所在本地 ext4 可用约 424 GiB。均不是项目配额保证，I/O 未测速。
- 沙箱内 /dev/nvidia 不可见、进程视图隔离；宿主机只读查询成功。不能把沙箱内 nvidia-smi 失败判成驱动故障。
- 当前工作区未发现 openpi、FastVGGT、LIBERO/Plus checkout、项目环境、数据、权重或实验结果；其他指定存储位置尚待确认，不能据此声称整台服务器没有这些资源。系统 Python 3.10.12 不代表项目环境已就绪。
- 详细本机证据在公开仓库外：`../artifacts/audits/server_review_20260906T061058Z.json`，7 项命令退出码均为 0；实际命令见 RUNBOOK。没有安装依赖、修改驱动或执行模型/仿真测试。
- 本次变更仅为 STATUS、DECISIONS、RUNBOOK 和本机审计证据；科学协议未改变，无数据/缓存/检查点失效。
- 活跃项目作业：本次未启动；无 run_id、GPU lease、checkpoint、有效更新或自动队列。G1—G5 未通过，protocol.lock 尚不存在。
- 待补充：项目可用 GPU 编号/UUID 与使用时段、已有资源路径；是否有硬截止日/卡时限制；后续人工 QC 审核人。两卡计划保持不变，不自动使用八卡。
- 完整交接点：上述文档和只读证据；无训练恢复源。下一任务仍是补齐 T00 资源/环境路径与 GPU 使用边界，再进入 T01/T02、T08 库存。已验证复查命令见 RUNBOOK；临时 Git 配置存在性需先检查。
