# F0：现状盘点——阶段日志

## 当前进展

最后核对：2026-09-07 14:00:21 +08:00。**F0/T00 DONE（盘点完成）**。六步结果与证据见本日志末尾阶段总结，PLAN 已按证据勾选；G1—G5 均未通过，F1 尚未开始。

当前文档分支 main，已发布基线 `eaf03864f3e75d6ff132e28bdc41fc6030cd26bf`；本轮 F0 PLAN/LOG 包含此前认证修复日志，未覆盖历史；最新文档版本以本文件所在Git提交为准，阶段执行时基线与发布版本分开追溯。实际学生工程选 `/nfs_share/lijunhui2/upstream/openpi`，上游 commit `215abfb217dbac7d5f1273282331b9b1866c0479`；本轮仅新增该工程短 AGENTS.md，尚无科研实现 commit。其余上游版本见执行记录。

活跃科研作业/自动队列：无；本轮审计脚本及三次只读新会话均已结束，最终进程核查未发现匹配残留。无 GPU lease、checkpoint、有效更新或分片恢复源。两卡仍只是多人共享服务器的可用并发预估，资源快照不保证下次可用。

F0无剩余阻塞。F1待验证事项包括依赖解析与锁定、驱动/JAX实际兼容、渲染依赖、原始HDF5回放/初态及安全转换适配，不能因F0完成宣称这些已通过。新会话核验的失败及只读越范围偏差已在末尾披露。

**下一步（本轮不执行）：等待负责人决定进入F1后，先在工作区用户目录获取并校验固定版本uv，准备Python3.11与隔离policy-train环境。**前置条件为继续授权、官方发行版/校验值核对及所选路径可写；不需要此刻GPU空闲，安装后再按F1计划验证。README仍指向F0。可复用的已验证审计命令见本日志，重新运行前先检查实际路径/配置与进程；不重复clone。

## 执行记录

以下为迁移前完整记录，按原文件保留内容；其中旧目录/旧入口、当时“未推送”等状态均是历史快照，不是当前维护要求。后来的简化规则取代旧结构规划；科学约束未改变。后续记录按时间追加，不再修改这些历史内容。

<details>
<summary>迁移前 STATUS.md 完整历史记录</summary>

# 当前状态

## 最新交接：2026-09-07，三个执行细节补充

- 追加工作区组织约定：负责人已确认，完整目录/文档/日志规则写入仓库外 AGENTS.md 第 14—16 节；原第 1—13 节保留。仅规划落文，目录创建/迁移尚未执行。此次未启动任何作业或推送；下一步可按该规则建立 F0/F1 文档导航与步骤，现有 T00/T01 前置核验仍有效。
- F0/T00 仍 IN_PROGRESS；入口 docs_commit 为 `ee60b41402c8ee96bafe17cae9f4e2134ffd4d1c`，本次为其上的未提交文档修改；implementation_commit 尚无项目实现值，上游快照见历史表。
- 已修改两份主文档、DECISIONS、RUNBOOK、README，新增文档仓库短 AGENTS.md。A 指标与版本归属是文档合同，尚无实现测试；文档入口已建立，工程/worktree 入口及无旧上下文的新会话核验尚待 T00/T01，不标 TESTED。
- 本次未安装环境或启动科研作业/自动队列；无 checkpoint、有效更新或 GPU lease。没有模型/仿真测试，未复查当前整机 GPU 占用；未修改实验参数或推送本轮内容。
- 恢复点为当前工作树与前次上游审计记录；下一步是 T00/T01 规则加载验证与基础环境准备，再按 T02/T08 做数据/初态库存。检查命令见 RUNBOOK；人工 QC 仍未签收，G1—G5 未通过。
- 文档检查通过（退出 0）：`git diff --check`，以及 7 个 Markdown 的围栏/相对链接、两文档三项补充与核心配置默认值静态检查。不是规则加载或模型验收。

## 历史交接：2026-09-06，V3 审阅补充

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

</details>

<details>
<summary>迁移前 DECISIONS.md 完整历史记录</summary>

# 决策记录

## 2026-09-07：工作区结构写入本机规则

负责人确认 phases/tasks 分工后授权继续，将已讨论的工作区树、Vault 层级、单步任务模板、日志/证据及恢复维护要求追加到工作区 AGENTS.md 第 14—16 节。phases 管阶段推进，tasks 管独立验收步骤，sessions 保存实际过程；权威文档留在 Vault，实现/环境/数据/原始产物留在工作区其他目录。

现有第 1—13 节原样保留。本次只登记组织规划，不创建整套目录、不迁移现有代码/环境/产物，不改变科学协议，也不将私人 AGENTS.md 全文纳入公开仓库。后续按 F0/F1 优先渐进落地，F2—F6 随真实接口细化。

## 2026-09-07：落实复审的三个执行细节

依据负责人授权，采纳 [GPT 第二轮公开回复](https://chatgpt.com/s/t_6a9e2067fdd88191a64f43d3a1f7cf14) 的定点建议；已读取完整公开回复 5035 字符，不代表新实验验收。

1. 手册 2.4/计划 4.3 明确新会话的规则来源检查，文档仓库新增短 AGENTS.md 入口，保留工作区私人原件。依据 [OpenAI 官方规则发现说明](https://learn.chatgpt.com/docs/agent-configuration/agents-md)，不假定 Git 根外父目录规则自动加载；工程/worktree 入口及新会话实测在 T00/T01 执行，目前未完成。
2. 手册 5.2/计划 6.2 明确正式自回归方向准确率、完整词/槽位与整段评分、预测前固定可靠 GT 集合和失败分母；teacher-forced 指标单列。T03 仅规则/标签统计，T05/T07 才与开发模型比较。
3. 手册 2.3/16.3、计划 4.3 分开 docs_commit、implementation_commit 与各上游版本；实际命令验证绑定实际代码。文档交接更新不改变科学协议 hash。修正 RUNBOOK 的只读措辞并同步 README。

仅修改文档与读取入口；主干、教师、层位、损失权重、K、四组与 seed 不变，无数据/缓存/检查点失效。本次没有启动新会话测试、安装环境、运行模型或公开推送。

## 2026-09-06：采纳 GPT 审阅补充，维持 V3 主方案

依据：负责人先要求只审查，随后明确授权完整读取 [GPT 公开分享回复](https://chatgpt.com/s/t_6a9d0c9220788191aef7802ee55bdec8)、修改文档并推送供复审。实际从页面数据中取得一条完整公开 assistant 回复，状态为 finished_successfully/is_complete，正文 6944 字符、八部分建议；没有声称读取未公开的原聊天或附件原件。分享回复是审阅意见，事实依据仍为本项目文档和官方论文/源码。

采用局部补强，不推翻 V3、不扩大实验范围。对应落点如下，均为待实施诊断/审计，非已完成实验：

| 审阅项 | 采用方式 | 文档位置（计划 / 手册） |
|---|---|---|
| 模块效果与算力边界 | 比较模块整体；同更新/曝光不是同 GPU·小时；组合最好不等于正交互 | 2.2、10.2 / 0.2、15 |
| A 多数类参照 | 每槽位 train 固定规则，独立 dev 同集合评估；混淆矩阵/逐类召回；GT 阶段只作离线诊断，不因偏斜改权重 | 6.2 / 5.2 |
| A 条件诊断 | 沿用既有诊断；动作敏感性不自动证明有益性，无 GT 最优或差异幅度门槛 | 6.7 / 5.7 |
| B 分视图质量 | 真实合并路径与第三人称/腕部各自质量；检查归一化后变化；稠密恢复不等于信息无损，不凭源码判断腕部失效 | 7.2、7.6 / 6.1、6.6 |
| 梯度夹角 | 成本允许选做，同模型/批次/共享 LoRA 集合；近零标不稳定，不作同向 gate，不加梯度手术 | 8.5 / 8.5 |
| 初态关系 | F1 联合训练—开发—最终关系审计，在 norm/全量标注/缓存前决定；重合分类披露，重划训练集须另据证据确认 | 5.1 / 4.1、9.2 |
| clean/Plus 与单元数 | 沿用库存/匹配；一次指官方扩展配置，不限每基础任务每维一个单元；匹配不足限制下降指标归因 | 9.1、9.2 / 9.2 |
| 统计精度 | 真实关联清单分析评测层精度，未知训练波动只作假设情景；不承诺最小增益，不新增功效 gate | 10.3 / 15.3 |
| K、Spatial 与新颖性 | 保留 20k/三 seed/Spatial；单终点仅支持预算性能差异，改 K 联动学习率；限制任务族语言结论，不以广义组合自称首次 | 5.1、5.5 / 4.1、8.5 |

InSpire 的 π₀-FAST 先例与当前 LoRA/Spatial 设置有差异；Spatial Forcing 还提供 π₀ LoRA/RoboTwin 相邻证据，但不直接验证当前 π₀-FAST。相邻的 [3D-CAVLA](https://3d-cavla.github.io/) 结合推理/深度/区域处理，既不证明本方案已被完整做过，也不保证具体差异就足够新颖。当前保留基础四组实证定位，不新增研究支路。

实际训练集/初态划分、教师合并设置和精度情景数值尚未选择；本次不自动作这些语义决策。A/B 通过门槛仍是正确实现、可信监督与稳定训练，不要求正式实验前先证明胜过 S。未建立或修改 protocol.lock，无重建缓存/重训/重评影响。

资源约定依据负责人新说明更新：八卡共享，两卡是预估；无硬截止日；基础环境/数据由 Codex 自行获取。此前关于资源路径/截止日待询问的记录是历史，不再作为当前阻塞。正式长作业仍须 gates、资源核查与固定授权队列。

## 2026-09-06：个人公开文档仓库

按负责人明确要求，在 `/nfs_share/lijunhui2/Vault-of-Xiyuan` 创建个人公开 Git 仓库，并将原工作区 `docs/` 移入仓库。`AGENTS.md` 留在工作区根目录，其阅读入口更新为 `Vault-of-Xiyuan/docs/...`。

负责人已授权将本仓库发布到 GitHub。此次变更只涉及文档组织和路径说明，不改变实验协议，也未启动科研实验。

## 2026-09-06：服务器适配审阅，保持科学协议与两卡预算

宿主机只读审计确认实际八张 RTX A6000；当前可见卡数与本项目可分配卡数分开记录。负责人尚未指定本项目 GPU 编号/UUID、时段与独占条件，继续保留两卡规划，不因瞬时空闲扩大作业额度。无需重建缓存、重训或重评；目前没有实验产物。

本次沙箱不暴露 GPU 设备且限制进程视图，宿主机只读查询成功。以后遇到同类现象先核实执行环境，不自行修驱动。共享仓库沿用已有临时 GIT_CONFIG_GLOBAL 方式读取，不改变用户全局 Git 配置、不改所有者；/tmp 文件可能丢失，RUNBOOK 明确其前置条件。

工作区实际为 NFS。后续实现应将容量与吞吐分开审计，并保证分片临时文件与最终文件位于同一文件系统；本地 staging 后需复制到目标侧临时文件、校验后再在目标侧原子提交。具体环境/缓存/本地工作目录尚未创建，依据实际 I/O 和路径可用性再确定。

执行手册 2.2 是未来工程布局示意，其中 repo/AGENTS.md 以及 STATUS/DECISIONS“待创建”标记不代表当前文档仓库现场。当前权威位置仍为工作区 AGENTS.md 与 Vault-of-Xiyuan/docs；后续工程 checkout 应明确单一文档入口，避免产生两份状态账本。未修改 AGENTS.md、两份实验主文档或原项目书；本机完整硬件/挂载信息保存在公开仓库外，本次未提交或推送。

</details>

<details>
<summary>迁移前 RUNBOOK.md 完整历史记录</summary>

# 已验证命令

2026-09-07 文档复查：沿用下列 `git diff --check` 命令，退出 0；用系统 Python 标准库检查 7 个 Markdown 的代码围栏、相对链接、两文档的三项补充和原核心 YAML 默认值，退出 0。新会话规则验证尚未执行，待执行步骤仅在手册 2.4，不列作已验证命令。当前工作树含未提交的本轮修改。

F0/T00 文档与服务器审阅。以下为实际执行过的命令记录，其中硬件与状态查询为只读操作；clone/fetch 会写入本地仓库。没有可运行的项目训练 CLI，没有模型测试或 benchmark。

## Git 状态

在 `/nfs_share/lijunhui2` 执行。前置条件：已有 `/tmp/xiyuan-gitconfig` 且含本仓库的精确 safe.directory 配置。该临时文件来自前一次仓库创建操作，不是仓库内依赖，也不保证重启后存在。

```sh
GIT_CONFIG_GLOBAL=/tmp/xiyuan-gitconfig git -C Vault-of-Xiyuan status --short
GIT_CONFIG_GLOBAL=/tmp/xiyuan-gitconfig git -C Vault-of-Xiyuan rev-parse HEAD
GIT_CONFIG_GLOBAL=/tmp/xiyuan-gitconfig git -C Vault-of-Xiyuan branch --show-current
```

本次入口工作树干净，分支 main，提交 c50040af2f1e4ffeb1c01f03699a206171aff8b6。普通 git 命令因共享目录归属检查失败；临时配置方式成功，未修改用户全局配置。

## 宿主机资源

以下须在能访问宿主机 GPU 的执行环境运行。本次默认沙箱内 GPU 查询失败、进程视图隔离；经工具权限审核后的宿主机只读查询成功。不能用沙箱 ps 判断整机空闲。

```sh
nvidia-smi --query-gpu=index,uuid,name,memory.total,memory.free,utilization.gpu,driver_version --format=csv
nvidia-smi --query-compute-apps=gpu_uuid,pid,process_name,used_memory --format=csv
nvidia-smi topo -m
lscpu
free -h
df -hT /nfs_share/lijunhui2 /tmp /dev/shm
/usr/bin/python3 --version
```

7 项命令分别退出 0，带时间戳的原始输出和退出码在公开仓库外 `../artifacts/audits/server_review_20260906T061058Z.json`。该证据只证明查询时硬件与容量，不证明持续使用权限、JAX/CUDA 兼容、渲染可用或训练吞吐。下一次资源分配前重新查询占用。

## 官方源码获取与审阅

在工作区根执行过以下命令，四项 clone 退出码均为 0。目标目录现已存在，以下为历史记录，不应重复执行 clone 或覆盖目录；具体版本见 STATUS 和仓库外 `artifacts/audits/scientific_source_inventory_20260906.json`。未初始化 openpi 子模块，未安装依赖、下载训练数据或权重。

```sh
git clone --depth 1 https://github.com/Physical-Intelligence/openpi.git /nfs_share/lijunhui2/upstream/openpi
git clone --depth 1 https://github.com/mystorm16/FastVGGT.git /nfs_share/lijunhui2/upstream/FastVGGT
git clone --depth 1 https://github.com/Lifelong-Robot-Learning/LIBERO.git /nfs_share/lijunhui2/upstream/LIBERO
git clone --depth 1 https://github.com/sylvestf/LIBERO-plus.git /nfs_share/lijunhui2/upstream/LIBERO-plus
```

默认沙箱首次网络访问失败，宿主机网络执行成功。共享目录 Git 状态检查使用另建的临时精确 safe.directory 配置，未改变用户全局设置；源码审阅四个版本的 rev-parse/status 均退出 0。静态 AST 检查 Spatial 十任务均为黑碗到盘子，不等于真实环境回放通过。

## 分享回复读取

实际执行 curl 获取负责人指定的分享页：HTTP 200，611218 字节。普通 HTML 可见文本没有回复正文；用标准库解析页面脚本中的结构化数据，提取公开 message_slice 的唯一完整回复，6944 字符，完成状态及结尾均检查。原始 HTML/提取正文保存在本机 /tmp，不上传到文档仓库；来源链接见 DECISIONS。本次未运行浏览器（CLI/浏览器不可用），不声称进行了浏览器验证。

## 文档检查

以下命令已在本次编辑前执行通过（退出 0），发布前对最终内容复查；只检查文档，不替代项目测试：

```sh
GIT_CONFIG_GLOBAL=/tmp/xiyuan-gitconfig git -C Vault-of-Xiyuan diff --check
```

最终修订复查通过（退出 0）：6 个 Markdown 的围栏/相对链接/无聊天内部引用标记；两份主文档的新增审计内容；手册中 K=20000、seeds 0/1/2、λ_A=0.3、λ_B=0.1、第 12 层默认值保留。一次初始检查因把“不得引入”语义检查写成过严的完全相同措辞而失败，修正检查以接受两份文档的实际等义表达后通过，未为此改实验内容。

发布前已执行 `git fetch origin main`（宿主机网络，退出 0），以及 `git rev-list --left-right --count HEAD...origin/main`（退出 0，结果 0/0），均使用上述临时 GIT_CONFIG_GLOBAL 和仓库路径。commit/push 的发布标识以 Git 提交和远端实际记录为准。

</details>

### 2026-09-07｜按负责人要求简化文档结构

完整读取 [GPT简化建议](https://chatgpt.com/s/t_6a9e269e23dc8191a45b3facc60a9661) 的公开回复（3469字符，完成状态）。采用每阶段 PLAN/LOG，取消任务、会话和独立全局账本的平行维护。两份主文档、工作区规则和仓库短入口同步；实验设计、参数、测试、人工gate、失败与恢复要求保留。

迁移前所有相关文件（含未提交/未跟踪记录）已备份至工作区仓库外 `artifacts/docs-backup-20260907-105230/`，附原始 SHA256 清单。旧 STATUS/DECISIONS/RUNBOOK 全文迁入上方历史区后停止维护；已提交历史保留，未改写Git历史。代码、环境、数据、原始产物未搬动、删除或重建。本轮未提交/推送，未启动实验。

实际操作：运行 `/usr/bin/python3 /tmp/xiyuan-simplify-docs.py` 完成备份后的迁移（退出0），随后清理主文档旧入口与阅读规则。验证（退出0）：旧三份账本全文与备份一致且完整包含于本日志；七阶段各仅PLAN/LOG；Markdown相对链接/围栏通过；两份主文档原YAML、bash块及公式保留；核心审阅约束保留。`GIT_CONFIG_GLOBAL=/tmp/xiyuan-gitconfig git -C Vault-of-Xiyuan diff --check` 退出0。以上为文档检查，不是模型或新会话加载验收。

### 2026-09-07｜细化 F0 计划并准备发布

将 PLAN.md 细化为六步：保护现场、核验规则入口、更新资源快照、明确环境与路径、核对来源与依赖、收口交接。每步写清操作、证据和边界，均未冒充实际执行；F0以只读盘点为主，安装/大资源下载/模型验证留F1及后续。负责人随后授权推送，发布范围包括本轮文档简化迁移、此前未提交的三项复审细节和F0计划。当前阶段仍F0，不启动科研作业，不迁移代码或环境；私人工作区AGENTS不进入公开仓库。

发布前检查：18个Markdown的相对链接、围栏及七阶段PLAN/LOG成对结构通过；`git diff --check`退出0，远端fetch退出0。此前历史全文保留和配置/公式不变检查见上一条迁移记录。本次为文档检查，无模型/仿真测试。发布成功与提交标识以远端实际Git记录为准。

### 2026-09-07｜修复 Vault 命令行 Git 认证

用户询问并要求处理命令行 push 无凭据问题。实际发现现有 gh 2.94.0 与 T1doo 登录可复用：沙箱内 auth status 曾报无效，宿主机网络复查退出0确认登录有效，未重新登录或读取令牌。仅在 Vault 的本地 Git 配置为 github.com 接入已有 gh auth git-credential，未改变用户全局配置或凭据，配置不进入公开提交。

实际验证：在宿主机网络执行 `GIT_TERMINAL_PROMPT=0 GIT_CONFIG_GLOBAL=/tmp/xiyuan-gitconfig git -C /nfs_share/lijunhui2/Vault-of-Xiyuan push --dry-run origin main`，随后同环境执行 `push origin main`，均退出0并返回 Everything up-to-date。共享目录仍使用已有临时 safe.directory 配置，文件存在性须检查；该认证修复不代表沙箱网络可访问 GitHub，也不自动配置其他仓库。未启动科研作业。此条为修复后的本地日志追加，尚未发布。

### 2026-09-07T13:50:29.114630+08:00｜开始执行 F0

负责人授权本轮仅执行F0并记录日志。先复用现有材料，保护入口工作树中尚未提交的Git认证日志。当前文档基线为已发布 eaf03864f3e75d6ff132e28bdc41fc6030cd26bf；本次原始证据集中在仓库外 `artifacts/audits/f0-20260907/`。不进入F1、不修改其文档、不安装环境或运行模型。

### 2026-09-07｜F0资源、来源与静态依赖检查

现场保护：复用三份历史审计/备份清单；备份SHA256全部匹配。文档库main为 eaf03864f3e75d6ff132e28bdc41fc6030cd26bf，保留既有认证日志未提交修改。四上游提交与原记录一致，LIBERO为master，其余main。F1工程根选现有 `upstream/openpi`，只新增短AGENTS入口，源码未改；其aloha/libero子模块尚未初始化，所记libero子模块提交与独立LIBERO checkout不同，F1必须显式选择实际来源，不能混用。FastVGGT干净；LIBERO/Plus最初status各25秒超时，保留记录；用GIT_OPTIONAL_LOCKS=0作有界复核均退出0、无修改。

宿主机快照：8张RTX A6000，总显存各49140 MiB，空闲48661—48664 MiB，查询时计算进程列表为空，驱动535.274.02；并非独占或持续空闲承诺。双路EPYC7542/128逻辑CPU、内存503GiB/可用448GiB，NFS可用约26TiB，本地盘约421GiB；未测速、不改资源预算为八卡。原始命令、时间与退出码见 `artifacts/audits/f0-20260907/host-and-repos.json`。仅需启动GPU作业前重新检查，当前无科研作业。

工具：系统 `/usr/bin/python3` 为3.10.12，没有python、uv、conda、mamba可执行入口；常用本用户位置也未发现uv/conda。codex-cli 0.153.4、现有gh、tmux和sbatch二进制可见；sbatch存在不代表已获资源预约或调度可用。动态库缓存可见EGL/GL、expat、fontconfig；包查询未找到libfontconfig1-dev、libmagickwand-dev、libosmesa6，退出1是缺包记录，不是安装失败。未更改系统。MagickWand/渲染后端在F1验证，必要时先评估用户目录依赖方案，再提出具体管理员需求。

候选环境（均未创建/未安装，最终锁定须F1验证）：

| 环境 | 候选及依据 | 待核验 |
|---|---|---|
| policy-train | Python3.11；固定openpi锁中的JAX0.5.3 CUDA12、Flax0.10.2、Torch2.7.1 | 535驱动下完整wheel/PTX/JIT兼容；不盲升驱动，uv.lock包含nvcc12.9.41等依赖 |
| teacher | Python3.10；官方Torch2.3.1、torchvision0.18.1、NumPy1.26.1 | 其余未钉死依赖解析后固定；权重/特征接口与双视图另验 |
| sim-clean | 官方Python3.8.13、Torch1.11.0+cu113、NumPy1.22.4、robosuite1.4.0 | openpi示例列robosuite1.4.1，与官方要求不同；先统一依据并做回放验证，不混装 |
| sim-plus | 独立Python3.8.13，沿相同LIBERO核心候选；usd-core25.5作为候选 | PyPI元数据有cp38 Linux wheel，不存在据此认定必须升级Python的依据；Wand/ImageMagick及渲染需验证 |

本机路径选择（以下除源码/已有审计外均拟建；没有创建目录树）：

| 用途 | 工作区相对路径 | 状态 |
|---|---|---|
| 实际学生工程 | upstream/openpi | 已有，新增短入口待纳入其后续实现版本 |
| 教师/clean/plus来源 | upstream/FastVGGT、upstream/LIBERO、upstream/LIBERO-plus | 已有，不复制到另一个code树 |
| 四环境 | envs/policy-train、envs/teacher、envs/sim-clean、envs/sim-plus | 拟建，分别隔离 |
| 环境管理工具 | tools/uv | 拟建，仅本用户工作区 |
| LIBERO配置 | local/libero-clean、local/libero-plus | 拟建；源码支持LIBERO_CONFIG_PATH，不改HOME |
| 原始/转换数据 | data/raw/libero_spatial、data/processed | 拟建 |
| 基础与教师权重/tokenizer | weights/pi0_fast_base、weights/fastvggt、weights/tokenizers | 拟建 |
| 方向/教师监督 | supervision/directions、supervision/teacher-features | 拟建，正式推理不挂载 |
| 协议与运行结果 | protocols、runs（按run_id收集日志/checkpoint/结果） | 拟建 |
| 包/下载缓存 | cache | 拟建，不能作唯一恢复源 |
| 本轮审计 | artifacts/audits/f0-20260907 | 已有，含原始命令/退出码 |

来源检查仅GET小型元数据或HEAD，未下载HDF5、模型或资产压缩包。7项HTTP200，详细清单/时间在 `resource-access.json`：

| 资源 | 官方来源 | 当前元数据revision/后续校验 |
|---|---|---|
| π₀-FAST base | gs://openpi-assets/checkpoints/pi0_fast_base | 公共对象列表可读，本次仅3项部分列表；F1全清单记录generation/校验和并验完整下载 |
| PaliGemma词表 | gs://big_vision/paligemma_tokenizer.model | HEAD200；F1保存对象版本/内容hash |
| FAST processor | physical-intelligence/fast（HF） | ec4d7aa71691cac0b8bed6942be45684db2110f4；含远程代码，执行前审阅并固定revision |
| Spatial原始演示 | yifengzhu-hf/LIBERO-datasets（HF dataset） | f13aa24a3da8c43c7225569f28c562979fa0e35a；列出10个Spatial HDF5，不代表已核验轨迹/状态 |
| Plus资产 | Sylvest/LIBERO-plus（HF dataset） | dd2bd61b7d9a6fef1abc52d606e983b41886a149；有assets.zip，不采用Plus混合训练集 |
| 教师权重 | facebook/VGGT_tracker_fixed（HF） | 4d9d9494b2211c90ab9cd3c8c6aa188fe363103d；按官方model_tracker_fixed_e20.pt核验 |

实际入口仅静态检查（详见 dependency-and-entrypoints.json），未运行项目CLI --help：openpi train/serve存在，但没有xiyuan四组注册；官方转换示例读取多套件RLDS并可能清理同名输出，不能直接用于我们的Spatial原始HDF5，F1需安全适配并保留sample_id/回放关联；LIBERO下载器可选Spatial/HF，但有覆盖/强制下载逻辑，正式下载前需固定revision和避免覆盖；FastVGGT eval_custom存在，默认合并参数不是“关闭”。不把示例可见写成链路可用。

### 2026-09-07｜新会话规则核验、失败与修复

实际入口为已有 `upstream/openpi`，原先没有AGENTS/override，本轮只新增短读取入口，没有改变源码。使用已核验 help 的 codex-cli 0.153.4，以 `codex exec --cd /nfs_share/lijunhui2/upstream/openpi --sandbox read-only --ephemeral --json` 启动无旧聊天上下文核验，未使用resume/fork。完整参数/工具调用与结果分别保存在本轮审计目录的events JSONL；非科研runner。

- 第一次：已读主文档，但180秒到限，退出124，无最终结论，不能算完整核验成功。
- 第二次：会话退出0，但其唯一工具误用不存在的python，工具退出127；文件读取未完成，该次不通过。
- 第三次：显式使用本机已验证 `/usr/bin/python3`，工具退出0、会话退出0。自动指令以本目录AGENTS指令块提供；工具显式读取工程AGENTS、工作区AGENTS、README、两份主文档及F0 PLAN/LOG，输出字节数/SHA256。独立检查确认七个必需文件均实际读取，稳定来源hash相符，当前阶段F0被正确识别。

第三次的路径提取范围偏宽，还只读了README中F1—F6的链接，并探测了不存在的根PLAN/LOG；这是已记录的核验偏差，不是完美限定范围测试。没有修改其他阶段或执行科研。原始 `rules-python3-events.jsonl`、`rules-python3-final.txt` 与 `rules-verification.json` 支持本次仅关于必需文件发现/可读的通过结论，不证明模型或任意未来会话行为。

中断恢复时先检查退出码和产物，未重复启动已完成检查。最终只读查询本用户相关F0脚本/新会话进程，无匹配残留，见 `final-active-check.json`。

### 2026-09-07｜F0 阶段总结

**结论：F0/T00盘点完成，停在F0；没有进入F1，G1/G2未通过。**

| 计划步骤 | 实际结论 | 证据（工作区 artifacts/audits/f0-20260907/） |
|---|---|---|
| 1 保护现场 | 文档/四上游根、分支与commit明确；原未提交日志保留；备份校验通过；两仓库status超时后复核干净 | host-and-repos.json、LIBERO*-status-retry.txt/.exit |
| 2 规则入口 | 沿用openpi，新增短入口；最终新会话实际读取必需文件通过，保留前两次失败及第三次额外只读偏差 | rules-*-events.jsonl、rules-verification.json |
| 3 资源快照 | 宿主机GPU/CPU/RAM/文件系统有实测快照；共享使用边界清楚；没有做压测 | host-and-repos.json |
| 4 路径与环境 | 实际工具/目录访问检查完成；四环境及数据/监督/输出路径明确为拟建 | host-and-repos.json及本日志路径表 |
| 5 来源与依赖 | 固定源码、候选组合、官方来源与revision、入口风险已查明；7项轻量HTTP检查成功 | resource-access.json、各metadata.json、dependency-and-entrypoints.json |
| 6 交接 | 日志已汇总、PLAN按证据勾选；F1首个动作和待验项明确，未自动切阶段 | 本阶段PLAN/LOG |

实际运行的审计脚本为 `python3 /tmp/xiyuan-f0-audit.py`、`python3 /tmp/xiyuan-f0-sources.py`（执行时解释器使用/usr/bin/python3），脚本自身均退出0；脚本内部各命令的失败/超时另存JSON，不用外层退出0掩盖。脚本副本已保留于本轮审计目录，便于复核真实操作；它们是本次审计证据，不冒充已注册的项目CLI。新会话和Git复核的退出码见各独立文件。后续复查可以使用这些已有系统查询，不能照抄未验证的训练命令。

剩余项属于F1及后续实施验证：获取固定uv/Python并解析隔离依赖；检查JAX/CUDA实际兼容；确认clean/plus统一核心版本与渲染依赖；安全下载及HDF5转换、回放和初态关系审计。若用户目录依赖方案不能解决系统库问题，再提出具体管理员需求，当前不修改驱动或使用sudo。

本轮修改范围仅F0 PLAN/LOG和必要工程短入口，原始证据在仓库外；未修改F1—F6文档、两份主文档或README，未创建环境/大数据目录、下载数据或权重、运行模型/仿真/教师缓存，也未推送。无科研作业或训练恢复点。等待负责人审阅F0结果后决定是否进入F1。

收口检查通过（退出0）：Vault仅F0 PLAN/LOG有修改；README、两份主文档及F1—F6与HEAD一致；17项勾选、链接/围栏及必需审计证据检查通过，`git diff --check`退出0。结果保存于本轮审计目录final-document-check.txt。本轮仍为未提交、未推送状态。

### 2026-09-07｜负责人要求每次完成后发布并提供审阅链接

负责人持续授权：以后每次完成工作，提交并推送Vault内本次相关修改，最终回复附对应文件的GitHub链接与提交标识。本轮补发F0计划勾选、完整日志和仓库短入口中的发布约定；完整本机规则仅留工作区。历史记录中的“未推送”描述保留为当时状态，发布结果由实际远端核验确定。

该约定已写入工作区AGENTS和仓库短入口。此为F0核验后新增的发布规则，原规则读取证据仍对应其当时hash，不冒充对新增文字再次做了新会话验证。无科学参数或实验运行变化，当前仍停在F0。
