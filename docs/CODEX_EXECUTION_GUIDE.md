# 曦源项目：Codex 基础阶段完整项目执行手册 V3

当前文档仓库为 `Vault-of-Xiyuan`，本文中的 `docs/...` 路径均相对于该仓库根目录。后文的工程布局是实施阶段的推荐方案，尚未创建。
## 分阶段推进 S、S+A、S+B、S+A+B 与仿真验证

版本日期：2026-09-07，V3 审阅补充版，采纳范围见 当前阶段 `LOG.md` 的决策记录。按 F0—F6 阶段和验收证据推进，可跨多次会话暂停/续接，不要求负责人每天在线。无硬截止日，约一个月仅为资源估算参考。本文是工程任务合同与实验背景，不是已完成代码或模型实测记录。

本文放在 `docs/CODEX_EXECUTION_GUIDE.md`；配套人类阅读计划为 `docs/EXPERIMENT_PLAN.md`，当前服务器的工作规则位于仓库上一级 `/nfs_share/lijunhui2/AGENTS.md`（不纳入本公开仓库）。本文包含实施所需的项目背景、数据与模型定义、实现接口、任务顺序、验收测试、资源预算和分阶段安排，不依赖原聊天才能理解范围。

**当前交付只有文档。下文 `src/xiyuan/`、`scripts/xiyuan/`、`configs/xiyuan/` 等均是拟新增路径；不得声称它们已经存在。上游 openpi 文件定位也要以服务器实际 checkout 核对。**

---

## 0. 先锁定任务，不重新扩大范围

### 0.1 用户已经确定的约束

本基础阶段做完四组：S=π₀-FAST；A=项目书 2.2 的显式空间提示；B=项目书 2.3+2.4 对应的隐式三维对齐与三维教师；完成仿真验证。已只读查见宿主机八张 RTX A6000 48GB，多人共享；两卡仅为可用并发预估，不是独占额度或持续可用承诺。每次启动前检查占用并登记资源，不自动使用八卡。负责人确认无硬截止日、无已有项目环境/数据，由 Codex 自行下载与准备；正式长作业仍遵守 gates/批准队列，空闲不足先等待或单卡推进。

**本基础阶段不做 B_focus、双源融合、方向标签反向投影、DINOv2 显著性、前背景差异加权和相关消融。**B 就是均匀三维对齐。不要为被排除的研究支路创建配置、数据字段、缓存、依赖或额外训练作业。FastVGGT 自身运行所必需的内部组件不等于另加一个显著性研究支路，不要误删其正常依赖。

本基础阶段也不做真机、从零预训练、换成 π₀/π₀.₅、PyTorch 移植、额外任务套件、全量 LIBERO-Plus 和组合扰动矩阵。

原项目书不修改，也不要求先返修申请书。执行中的纠错、具体化和范围差异写入本文、实现及 当前阶段 `LOG.md` 的决策记录。本版文档版本不自动改写已有正式协议/结果；检查已有工作后决定继续原协议或显式开启新版本。

### 0.2 研究目标

视觉-语言-动作模型以当前图像、本体状态和指令生成动作。本项目要区分两种训练机制：A 在动作生成前要求模型显式输出方向；B 在中间视觉表征施加冻结教师监督。四组形成 2×2 因子实验，判断独立增益与交互，而不是预设组合模型必须获胜。[P1]

代码正确、数据监督可信、对照公平和仿真真实是交付条件。没有提升可以报告；不能通过推理使用 GT、删除失败回合、替换主干、改变动作接口或增加增强组训练步数得到“提升”。

四组识别具体模块整体的效果，不识别 A 内问题/监督/条件化各自的唯一因果作用，也不证明 B 收益唯一来自三维知识。固定预算指样本曝光与有效更新，不指相同 GPU·小时。诊断不以增强组提前胜出作为 G2 门槛；组合最好、正交互和统计显著是三个不同判断。

### 0.3 来源与假设分开

项目书已有：π₀-FAST、先方向再动作、冻结三维教师的逐视觉 token 余弦对齐、具体使用 FastVGGT、LIBERO 干净与 LIBERO-Plus 七维评测。[P1]

本计划补充且尚需验证：Spatial 10 任务、45/5 轨迹划分、LoRA 具体冻结范围/rank/alpha、共享 LM 方向输出头、对象相对夹爪的坐标语义、第 12 层、投影头宽度、20k、三个种子、各项超参和抽样量。项目书第 2 页已有 LoRA 微调安排，本版只纠正具体参数来源的表述。[P1] 不得把这些描述为项目书已规定、用户已逐项确认或论文已验证最优。

本基础阶段采用共享 LM 输出头实现方向回答，区别于新增独立七分类网络；项目书 3.1 的“方向词分类头”没有指定结构。保留这条差异记录，不自行在两个含 A 的模型中实现两种不同头。InSpire 与 Spatial Forcing 仅为机制参考，LIBERO-Plus 是本项目统一选择的扰动评测平台；不沿用所引论文版本尚不支持的共同平台说法，不做其全部设定的逐项复现。[R1][R2][B2]

## 1. 实验合同

### 1.1 四组唯一命名

| ID | variant | A | B | 训练目标 |
|---|---|---:|---:|---|
| E00 | `s` | 0 | 0 | action |
| E10 | `sa` | 1 | 0 | action + λ_A direction |
| E01 | `sb` | 0 | 1 | action + λ_B alignment |
| E11 | `sab` | 1 | 1 | action + λ_A direction + λ_B alignment |

默认每组 seeds `[0,1,2]`，12 个正式作业。全部从同一个 `pi0_fast_base` 初始化，不从已微调 S 接着训练增强组。开发权重放 pilot 目录，正式权重放 formal 目录，禁止混入主表。

B 必须不依赖 A 标签、问题解析或在线输出。SB 的配置在没有方向标注文件时仍应能运行。SA 不加载任何教师缓存。S 不加载两类额外监督。

### 1.2 公平性合同

共享：数据划分、动作样本、sample_id 采样序列、图像预处理、本体状态定义、归一化、动作正逆变换、FAST tokenizer、动作块长与执行策略、冻结参数集合、LoRA 配置、共享参数优化器、有效 batch、更新数、最终检查点规则、评测 manifest、终止规则。

允许差异只有 A 的方向序列/损失、B 的投影头/损失及其直接计算开销。B 增加可训练投影头必须报告参数量，但不允许悄悄解冻其余基线参数。四组的计算时间无需相等，样本曝光和更新数必须相等。

共享参数初始化使用固定名字的独立 RNG 流；新头初始化不能改变共享 LoRA。禁止使用进程随机化的 Python `hash()` 生成跨进程实验种子；使用固定整数或稳定摘要。

### 1.3 评测合同

主任务暂定 LIBERO-Spatial 10 个。干净与七维：布局、视角、机器人初始状态、语言、光照、背景纹理、传感器噪声。[B1][B2]

**默认 P1-Lite**：三训练 seed，干净和每维一个核心变体包；P0 为同清单的 seed 0 四组先行成果；P1 为三 seed 的预登记第二变体包扩展，不增加训练模型。每任务每包目标 20 个真实不同单元，但数量必须由资源库存决定，不可重复确定性 reset 凑数。

一个 `variant_pack_id` 可以包含一个官方配置及多个初态；状态不足时在冻结前纳入多个同维度官方配置，或按实际数量评测并披露。必须保留 `official_config_id` 和包内聚合权重，不能冒称多个配置都是同一配置的不同初态。

若每任务每包确有 20 个单元，P0/P1-Lite/P1 分别为 6,400/19,200/36,000 模型评测回合；实际预算与分母从唯一 manifest 单元计算。三档嵌套、同模型同 eval_id 复用，不重复加数。核心和扩展后 R 的覆盖口径分别标记；复用的是原始结果，不是把不同覆盖的汇总值当成同一指标。

开发快测起点为 3 任务×5 开发单元×干净/视角/布局=45 回合/模型；实际按库存确认，优先与最终资源及已知底层初态簇隔离。F1 盘点资源，G2 冻结真实核心/可行扩展清单、配置权重、独立单元与关联簇。资源不足先调整清单，不把最终唯一单元拿来调参。

P1 是否启用只依照预登记资源条件，不依照已看到的效果；默认先确保三 seed 核心完整。全部属于官方资源的项目子协议，不称完整 LIBERO-Plus。

## 2. 开始前的仓库工作流程

### 2.1 首次会话必须做的事

1. 读取所在路径的 `AGENTS.md` 与更高目录规则；检查当前仓库/分支/未提交修改。已有规则不得直接覆盖。
2. 从 README 的当前阶段入口读 PLAN、LOG，再查本手册与 `EXPERIMENT_PLAN.md`；首次进入项目须理解两份主文档。路径不符先定位，不创建空文件冒充。
3. 只读审计服务器 GPU、CPU、内存、磁盘路径、现有 Python 环境、数据、权重和正在运行的作业。
4. 记录真实现状：哪些已有、哪些缺失、哪些未测试。不要把文档中的目标目录当成已存在。
5. 建立任务状态账本，先执行最早未完成且依赖已满足的任务。默认从 T00 开始，不直接提交 12 个长训练。

用户已明确四组、排除双源和磁盘足够，无需反复询问这三件事。实际硬件、路径、缺失资源与语义冲突先通过只读检查解决；不能自行解决的必要问题才询问负责人。

### 2.2 简化文档布局与实际工程路径

权威文档仅保留两份主文档和 F0—F6 阶段目录，每阶段 PLAN.md/LOG.md 两个文件。具体目录由 README 链接；工作区完整规则与文档仓库短入口保留。源码、环境、数据、缓存、权重和产物沿用实际路径，不为文档迁移重建或搬动；已下载源码位于工作区 upstream/，路径与版本证据记 F0 日志。后文 src/xiyuan/、scripts/xiyuan/、configs/xiyuan/ 等仍是工程接口规划，并非已实现目录。

阶段 PLAN.md 写目标及前置条件、具体实施步骤（复选框）、完成条件和实际需负责人确认的事项。步骤下面按需要写输入、方法、产物和检查方法，共用技术要求引用两份主文档；不按单步任务另拆文件。T00—T12 仅保留为任务引用，G1—G5（含 G3a/G3b）验收条件保留在阶段完成条件，结论及证据写同一阶段日志，不另建 gate 文档。

阶段 LOG.md 顶部维护最后核对时间、当前进展、活跃作业/完整恢复位置、阻塞及下一步；下面按时间追加实际执行记录，包括必要的命令、退出码、产物、版本、失败及决策理由。字段按工作复杂度选用，不机械填写空表，不记录每条普通 shell 命令。关键作业启动、失败和完整恢复点及时记录。顶部可更新，历史不改写；修复追加新记录。阶段完成在同一 LOG.md 末尾写阶段总结、验收证据、遗留项与下一阶段。

开始工作：检查规则和 Git 状态，通过 README 进入当前阶段，读取 PLAN、LOG 顶部及最近记录，再按需查阅两份主文档。续跑先核对真实进程和产物，不能仅凭旧日志启动。结束工作：只更新当前阶段 LOG；步骤有真实证据才勾选 PLAN。切换阶段时写旧阶段总结，更新 README 的当前阶段入口；跨阶段事项引用旧日志及产物，不复制历史或重复启动。

决定与理由记发生阶段的 LOG，影响后续的已批准规则同步主文档；已验证命令与证据也记发生阶段 LOG，之后引用该记录。草案命令仍标 PLANNED，未实现接口不能当作可运行命令。原始测试输出、配置、标注、模型、缓存、检查点与评测结果保留在工程/运行目录，LOG 仅写摘要和真实路径/hash；不受每阶段两个 Markdown 限制。版本分别记录 docs_commit、implementation_commit 和各上游实际执行 commit，dirty 修改另存差异证据；交接日志更新不改变科学协议 hash。

不再建立或平行维护 phases/、tasks/、sessions/、gates/、commands/ 等文档管理目录，以及独立 STATUS、DECISIONS、RUNBOOK、REVIEW 或同类替代管理文件。先细化当前与下一阶段，后续阶段保留有实质内容的简要计划。完整本机规则不公开；必要的仓库短读取入口保留，工程/worktree 新会话的实际规则加载仍须核验。

### 2.3 状态与任务账本

任务进度状态采用 TODO/IN_PROGRESS/PAUSED/BLOCKED/BLOCKED_HUMAN/DONE；只有已有产物/测试证明完成才写 DONE。实现证据另分 PLANNED/IMPLEMENTED/TESTED/FORMAL_DONE，不能把写完代码等同于实验完成。每个任务记录阶段 F、负责人/会话、依赖 gate、分支、允许修改范围、实际命令、证据/产物、当前阻塞和下一步。

暂停/恢复字段至少包括：当前协议/代码哈希、运行 run_id/PID/GPU/心跳、最近完整检查点/缓存分片/评测 shard、已完成有效更新、待人工问题、已批准队列及资源上限。不要因用户隔几天回来就重新分数据、重训已完成模型或重置 seed。

每次结束在当前阶段 LOG.md 记录实际操作、结果、证据和决定理由，更新顶部恢复信息；计划步骤实际完成才勾选 PLAN.md。阶段总结及 gate 结论也写同一日志。历史命令的版本和证据必须可定位，不把草案接口称为已验证命令。

版本归属在 T00/T01 明确：`docs_repo_root/docs_commit` 指权威文档，`implementation_repo_root/implementation_commit` 指实际执行的项目代码；`upstream_commits` 按 openpi、FastVGGT、LIBERO、LIBERO-plus 分别记录，修改过的教师/仿真副本另记实际执行 commit。仅下载上游不能标为已有项目实现，当前 implementation_commit 可为空。作业、checkpoint、恢复与命令登记使用同一含义；开发期未提交修改另存 dirty 状态与可追溯 diff，不能用 HEAD 冒充完整执行版本。docs_commit 是启动时读取的文档快照，不纳入科学协议 hash；阶段日志顶部交接更新不自动使模型失效，真正改变科学语义时仍按协议变更处理。

### 2.4 AGENTS.md 与完整文档的分工

根目录 AGENTS.md 只放长期约束、验收门槛和阅读入口；完整实现保留在两份 docs 中并按任务主动读取。Codex 官方文档说明项目指令有合并大小限制（默认 32 KiB），不能把整本执行手册塞进 AGENTS.md 后默认全部会生效。[C1] 服务器存在全局或更近目录规则时，先核查实际生效指导和冲突，不盲目覆盖。

官方发现机制在启动时构建规则链，项目范围通常从 Git 根向当前目录扫描，不保证读取 Git 根以外的父目录规则。[官方说明](https://learn.chatgpt.com/docs/agent-configuration/agents-md) 因此本仓库新增的短 AGENTS.md 仅指向权威规则与文档，不复制私人规则全文。未来工程 checkout/worktree 须保留上游规则，补充同类入口或采用已验证的显式启动方式；不假定相对路径在换目录后仍成立。

T00/T01 在正式代码开发前，从实际长期使用的工程根及使用中的 worktree 各启动一次无旧聊天上下文的新会话，只做读取检查：记录启动目录、客户端版本、自动加载来源与按入口显式读取的来源、规则/两份主文档/当前阶段 LOG.md 的路径及最新交接摘要。检查 override、大小限制、路径缺失与冲突；保存会话证据后才标 TESTED。当前会话手动读到文件或仅新增入口不等于新会话验证通过。核验步骤是待执行检查，不是已验证 CLI；不启动训练或改全局设置。

## 3. 软件与资源架构

### 3.1 主路线

采用 openpi JAX/Flax。当前核对的官方 README 未支持 π₀-FAST PyTorch，而配置文件提供 JAX 的低显存 LIBERO 微调起点。[O1][O2] 不将 `train_pytorch.py` 能运行其他 π 模型当成本项目可迁移的证据。

四环境：policy-train（JAX 学生）、teacher（PyTorch FastVGGT）、sim-clean、sim-plus。仿真通过独立学生策略服务取动作。LIBERO/Plus 的共享用户级配置也要隔离，不能只看 pip 环境不同就认定无冲突。[B2]

依赖以实际固定 commit 的官方说明与锁文件为准。不强写一套可能冲突的 CUDA/Python 版本，不在主系统环境反复升级。缺系统依赖可记录并请求管理员处理，不默认有 sudo 授权。

### 3.2 硬件审计命令

以下为只读审计命令示例；本机已执行的具体查询、结果和退出码见阶段日志中的已验证命令记录。默认沙箱可能不暴露 GPU/宿主机进程，失败时先核实执行环境，不判定驱动损坏；不能凭沙箱 ps 判断整机空闲。

```bash
pwd
git status --short
git rev-parse HEAD
nvidia-smi --query-gpu=name,memory.total,driver_version --format=csv
nvidia-smi topo -m
nvidia-smi
lscpu
free -h
df -h
```

非 Git 目录先定位仓库，不能让 `git` 失败后便重建覆盖。输出保存到审计目录时不要包含 API 密钥或完整环境变量。

两卡默认一个大作业一张卡。worker 渲染显存也计算在内。以 `CUDA_VISIBLE_DEVICES` 限定可见卡，注意进程内部逻辑编号可能变为 0，不能把它与物理编号混淆。不得终止不属于本项目的进程。

### 3.3 上游定位表

| 上游位置（核对后使用） | 需要理解的内容 | 本项目修改原则 |
|---|---|---|
| `src/openpi/training/config.py` | π₀-FAST LIBERO、norm、delta、freeze、训练配置 | 添加/注册四组；不改其他配置默认含义 |
| `src/openpi/models/pi0_fast.py` | 前缀/后缀 attention、loss、decode、freeze filter | 定义四组共用项目 S，再增加 A/B；不保留错误冻结假设 |
| `src/openpi/models/gemma_fast.py` | 扫描层、LoRA、隐状态 | 只取一个选定层；保持参数树兼容 |
| `src/openpi/models/tokenizer.py` | FAST 段、token mask、动作抽取/截断 | 四组严格错误状态；A 打包/剥离单独实现 |
| `src/openpi/models/model.py` | 训练内部 preprocess/augmentation | 四组确定性变换；真实入口截取验证 |
| `src/openpi/policies/libero_policy.py` | RGB/状态/零占位视图映射 | 四组相同；对齐有效 mask 独立 |
| 数据变换、Observation、data_loader、训练 step | 新监督是否穿过管线、RNG、梯度累计 | 不让标签/缓存被丢弃；不污染推理接口 |
| `scripts/train.py`、`scripts/serve_policy.py` | 训练和策略服务入口 | 核验参数名后再写阶段日志中的已验证命令记录 |
| `examples/libero/` | 数据转换、仿真客户端 | 不照抄示例的 pi05 检查点替代 π₀-FAST |

这些位置来自公开实现核对。[O2]—[O7] 服务器固定版本不同需更新位置表，不能编造不存在的类名/函数名。

### 3.4 四组共用的项目 S 工程修正

**冻结不是一个配置布尔值。**核查到的 `Pi0FASTConfig.get_freeze_filter()` 只覆盖 LLM 内非 LoRA 参数，训练器又以冻结集合的补集确定可训练参数；直接沿用低显存配置不能保证 SigLIP 冻结。[O2][O3] 实现显式参数白名单：共享 Gemma LoRA；仅 SB/SAB 额外允许 projector。参数实际名称按 checkout 查明，不虚构正则。

记录 name/shape/dtype/group/trainable/count，并核对优化器状态/weight decay 的实际作用集合。保存几次有效更新前后的参数摘要和差值，断言原 SigLIP/Gemma 每个冻结叶子不变，至少部分共享 LoRA 真实更新，B 仅增加指定新头。前向 `train=False` 不等于优化器冻结。

**严格 FAST 封装覆盖 S/SA/SB/SAB。**固定上游存在缺 Action 标记返回零数组、超长训练序列警告后截断的路径。[O5] 项目在进入这些回退前检查 Action/结束边界、合法动作 token、长度、shape 和有限值，返回独立状态或明确异常。合法零值动作有效，不能用 `all(actions==0)` 检测失败。正式评测的解码失败计策略失败，不以零动作继续伪装正常。

**确定性预处理检查到真实模型入口。**上游 `preprocess_observation(train=True)` 内含随机变换。[O7] 在实际训练路径截取同一 sample_id 进入视觉编码器的图像，验证不同 RNG/重复调用符合确定性约定，再核对教师缓存对应源图与坐标变换。禁止只在外部 loader 或 YAML 里关闭增强。

以上三项是项目 S 的共用定义，不属于 A/B 因素。合法 token/编解码与未修改上游回归；错误输入按严格项目协议处理。`test_*_disabled_matches_baseline` 的 baseline 始终是这个项目 S，而不是要求复制上游异常行为。

必须具备的测试：`test_trainable_parameter_allowlist`、`test_frozen_parameters_unchanged_after_updates`、`test_shared_lora_updates`、`test_fast_decode_explicit_errors`、`test_valid_zero_actions_are_not_errors`、`test_sequence_overflow_rejected`、`test_training_preprocess_deterministic_and_cache_aligned`。T01 先做，T07 四组复验。

## 4. 数据合同与隐私边界

### 4.1 数据范围与划分

LIBERO-Spatial 10 任务；拟每任务约 45 条训练、5 条验证。先核实有效轨迹数与原始资源，再生成固定 episode-level split。禁止 frame-level 随机切分造成同轨迹泄漏。

归一化仅训练数据；四组使用同一文件及其哈希。原始动作、状态、相机和 instruction 完整保留。A 缺标签不删动作样本，B 缓存缺失不自动换帧。

T02/T08 在 norm、完整标注与全量缓存前联合输出训练轨迹—开发—最终评测底层初态关系表：来源、映射证据、重合数、未知项。相同任务不同初态符合范围；已见底层初态上的新扰动可研究匹配初态鲁棒性，但不称严格初态留出。开发调参使用的实际单元不得伪装独立最终单元。若按初态组重划训练集，先记录证据和影响并确认；不因猜测重合自动删演示或造状态。Spatial 黑碗到盘子任务族的语言改写稳健性不证明广泛语言理解或依赖语言消歧。[B1]

### 4.2 稳定键

`sample_id = suite/task/episode/frame`，每个转换样本保留原始 episode/frame 和原文件摘要。manifest 顺序与 dataloader 顺序无关。

建议预生成每个训练 seed 的 sample index schedule，或实现由 `(seed, effective_step, position_in_batch)` 确定的采样器。保存 schedule 哈希，四组共用。恢复以已完成的有效更新为准，不使用数据预取到哪里就从哪里开始。

训练清单、验证清单、教师缓存清单、最终评测清单分离。缓存文件存在不代表允许训练读取验证样本；loader 必须检查 split。

### 4.3 运行时公共输入

```text
PublicObservation
  RGB: actual images, uint8 或已验证的统一格式
  proprioception: 基线允许的本体状态，D_state 由数据审计确定
  instruction: 当前收到的原始语言指令
```

推理可用的夹爪/机械臂自身状态不等于目标对象 GT。禁止在公共请求里加入目标对象坐标、参考对象坐标、GT 方向、仿真成功谓词、教师特征、未来图像或标签缓存索引。

评测器可以持有 task_id 以便建环境，但策略只能获得原始指令，不能按 task_id 查回标准指令。eval_id 用于调度日志，不让模型分支根据它选择答案。

### 4.4 训练批次建议接口

以下为待实现 schema 合同，不是已有 openpi API：

```text
XiyuanTrainBatch
  observation                       # 图像、本体状态、训练 token 序列
  actions: float[B, H, D_action]
  action_loss_mask: bool[B, L]
  direction_loss_mask: bool[B, L]    # A 关闭时全 0/不启用分支
  alignment_targets: float[B, V_real, N_patch, D_teacher] | None
  alignment_valid_mask: bool[B, V_real, N_patch] | None
```

host-only metadata 单独保留：`sample_id`、raw paths、split、hash、方向质检原因。字符串不要作为每步变化的 JAX 静态参数，避免反复 JIT 或不支持的 pytree 类型。

训练 token 序列的真实后缀属于 teacher forcing 标签，允许在训练分支出现；公共推理构造器只生成无答案前缀，且必须有负面测试证明两条路径不会混用。

模型可提供项目适配的 `compute_loss_with_aux` 一类接口（名称实施时确定），返回每样本主损失及数值 metrics。不要直接把 `(loss, dict)` 塞进原本期待数组的上游训练器；需要相应适配并做回归。A/B 关闭时保持原接口可调用。


**F1恢复实测补充（2026-09-07）：** 本机固定JAX环境下，即使输入与StableHLO一致，默认GPU编译设置仍出现跨进程更新差异。统一步数int32非weak类型、显式分片，并在学生进程固定 `XLA_FLAGS=--xla_gpu_deterministic_ops=true --xla_gpu_autotune_level=0` 后，真实权重/数据的独立保存恢复及下一次更新逐参数精确一致。此设置作为当前项目S运行要求，四组共享并纳入配置/检查点指纹；不是新方法或泛化结论。实际累计入口和其他硬件仍须各自验证，吞吐按该设置重新实测；详情见[F1日志](F1_基线与数据/LOG.md)。

### 4.5 时间和动作正确性

对当前观测 t 使用当前位姿生成方向，并监督从 t 起的动作块。明确原数据记录的是动作前还是动作后状态，逐样本用回放验证。末尾动作块不能跨下一 episode。

**2026-09-07 F1实测与负责人批准的时序具体化：** 固定Spatial原始HDF5的obs在step后采集；500条演示全部61,750个可比较位置的obs[i]关节状态与states[i+1]精确对应。四组统一使用obs[i]、states[i+1]及actions[i+1:]（0≤i<L−1）；每条演示排除缺少动作前原始RGB的首动作起点，共500个，不重渲染或修改原数据。完整episode划分450/50，得到55,682/6,068训练/验证起点；norm仅训练集。当前实施清单为工作区protocols/data-v2，尚非正式协议锁；初态关联未知不宣称严格留出。审计、负责人确认及限制见[F1日志](F1_基线与数据/LOG.md)。


审计单位、旋转表示、夹爪符号和 state/action 维数。官方 π₀-FAST 配置中的 action_dim=7 不自动意味着原始 state 也恰好 7 维。[O2][O6] 不靠随手切片消除 shape 报错。

## 5. A 模块工程规格

### 5.1 方向语义 v1

对象槽位按语义角色固定排序：`manipulated_object` 是被操作对象的完整指代表达，`placement_target` 是明确放置目标，最多两个。不是句中前两个名词。颜色、左右、between 等消歧关系和内部参照对象均须保留；消歧参照不自动占第二槽位。解析仅用当前文本和训练/开发阶段冻结的规则，不在 canonicalization 中抹去扰动。

协议示例：“拿起盘子与小烤碗之间的黑碗，再放到盘子上”，槽位一保留完整黑碗限定，槽位二为盘子。在线构造问题与离线 GT 实例关联是两个函数/数据边界；后者只给已确定问题生成监督，不利用隐藏实例 ID 回写在线问题。

参考原点：当前末端/夹爪位置。方向坐标轴：固定机器人基座坐标系，不随视角或夹爪转动。使用 `R_base_from_world @ (p_object_world - p_eef_world)`。先可视化确认轴正负语义；不要假设 +x 对应屏幕右。

类别：left/right/front/back/up/down/grasped。六方向取绝对值最大的坐标轴；grasped 仅来自当前可信抓持判据。只闭合夹爪不够，不使用未来抓取成功。不确定边界、对象身份模糊、位姿缺失或时刻错配标记为 invalid。

项目书未固定这一坐标系、槽位与阈值；这些写入 `direction_protocol.json` 和阶段日志中的决策记录。F2 先确定并质检可执行规则，G2 后不能按最终语言/视角测试失败临时修改。

### 5.2 标注数据输出

每个 sample 至少有：

```text
sample_id, annotation_version, instruction_hash
question_slots: [{slot_index, role, referring_expression, question_text}]
# role: manipulated_object | placement_target；完整保留 referring_expression
labels: [direction_name, ...]
valid: bool
invalid_reason: null | text_parse_failed | ambiguous_object | instance_mismatch | missing_pose | boundary | unsynced | grasp_uncertain
coordinate_frame_id, label_rule_hash
```

对象 GT ID、instance_binding_status、离线阶段 stage_label 留在标注/QC 元数据，不进入 PublicObservation。问题文字不能包含坐标数值、成功标签或未来动作；不能先用 GT 决定对象，再声称问题只来自文本。

抽查至少 300 个帧级问题，按任务/阶段/角色/同类物体消歧/类别分层，每题分别审“文本指代、离线实例、方向标签”。保存样本 ID、审核者、三项结论和错误原因；分多次会话完成且去重。建议明确可判定标签一致率 ≥95%，同时报告总体和任务×阶段×槽位覆盖、有效标签类别分布与 invalid 原因分布。

总体覆盖低于约 90% 需分析；超过该值也不能掩盖接近、抓持转换、放置接触阶段集中缺标。边界不确定与实例/位姿缺失分开统计，invalid 样本不猜方向类别。关键阶段系统性缺标需给出解释和采用决定后才能过 G2。Codex 可生成图和清单，无人工证据则写 BLOCKED_HUMAN，不能冒称完成 300 题检查；无依赖任务可继续。

T03 只生成无需训练的每槽位多数类规则与标签统计：由 train 有效标签确定类别，保存 train/规则 hash；并列多数类的确定性处理写入规则，无支持项明确标记。T05/T07 再用开发模型在独立开发数据的相同可评价集合上报告参照与模型准确率、混淆矩阵、逐类召回率和覆盖；T03 完成不依赖已有 SA/SAB 模型。不得用开发/最终标签重选多数类。GT 阶段参照仅离线诊断，不作公共输入公平基线。偏斜不是失败 gate，不自动加类别权重、重采样或删类。

主方向指标采用正式方向解码路径：仅公开观测与当前文本构造的问题前缀，自回归生成完整方向段，后续槽位/子 token 只能接收此前模型输出，不注入 GT 答案。按完整方向词及槽位匹配，另报整段全部所需槽位正确率；teacher-forced token 准确率仅作独立训练诊断。预测前用可靠 GT 固定可评价槽位/样本及分母，GT 仅供离线评分；可靠集合中的文本解析失败、方向生成失败单列且计未正确完成，不能事后只保留成功生成。无可靠 GT 的样本单列覆盖，不强判对错；逐类召回包含该类失败项，混淆矩阵保留失败列。受限解码的高合法率主要验证格式约束，不替代方向识别准确率。第 5.7 节预测方向分支复用此路径，GT/错误方向仅限其隔离诊断。

### 5.3 invalid 样本处理

本基础阶段采用简单且可审计的规则：某样本的方向监督不可靠时，保持原动作数据，在含 A 模型中对该样本使用 action-only 打包，不插入假方向答案。多槽位只要存在需要但不可信的标签，本版整个样本退回 action-only；规则对 SA/SAB 相同。

该处理只用于训练缺失监督。推理不能读取 valid 或 GT 决定绕过 A；只能因当前输入文本解析失败或模型方向生成错误采用冻结回退。分别统计训练缺失率和推理回退率，不能混为一个数字。

### 5.4 序列与 attention

建议逻辑模板（标记需编码测试，不要求逐字使用此示例）：

```text
PREFIX / 双向前缀：
  Task: <spatial questions + original instruction>, State: <baseline state string>;

SUFFIX / 因果后缀：
  Answer: <dir for slot 1>; <dir for slot 2>;
  Action: <original FAST action tokens>|<EOS>
```

A 关闭时使用项目 S 的 action-only 打包及严格长度/错误检查，不留空的 Answer 标记改变基线。合法序列的 token/mask 与上游回归，错误路径以第 3.4 节为准。

prefix 所有 token 可双向可见；整个答案和动作 suffix 为因果。图像 token 不能看到 GT 答案或动作。`token_ar_mask`、`token_loss_mask` 与真实长度同时扩展。训练 next-token shift 后，`direction_loss_mask[:,1:]`、`action_loss_mask[:,1:]` 与 targets 对齐。

不要把“答案算辅助标签”误解成可以把答案放进双向 prefix。仅 loss_mask=0 不能阻止 attention 泄漏。

### 5.5 方向生成与 FAST 隔离

将每类方向在真实 tokenizer 下编码为完整序列，建立候选序列 trie 或等价受限解码状态机。多槽位用明确分隔与固定顺序。Answer 段的格式/边界 token 可计入方向段损失；动作段的 mask 与基线保持相同定义。

方向不必单 token；不扩展基础词表，不随意重训 embedding。动作抽取必须从明确的 Action 段得到原始 FAST token；返回精确动作 shape 和错误状态，不能把任何英文方向 token 送入 FAST 解码器。

限制方向段最大生成长度；给动作段保留与 S 相同的可用动作 token 预算。四组动作采样温度和后处理相同，默认沿基线的确定性推理。A 可多用方向解码时间，需要报告。

正常流程为一次前缀预填充、方向续写、动作续写。恢复路径需要重新用 action-only 前缀建立一致缓存；不能从包含一半非法方向的 KV cache 直接声称回到了 S 路径。

### 5.6 A 验收测试

| 测试名建议 | 最低断言 |
|---|---|
| `test_a_disabled_matches_baseline` | A 结构关闭后与项目 S 一致；不恢复上游静默回退 |
| `test_direction_multitoken_roundtrip` | 七类及多槽位编码/边界完整，无单 token 假设 |
| `test_action_span_unchanged` | 给定相同专家动作，剥离 A 后 FAST token 序列与 S 相同 |
| `test_next_token_masks` | targets、logits、两类 loss mask 无 off-by-one |
| `test_direction_prefix_causality` | 改 GT 后缀不影响预测首个方向之前的状态 |
| `test_no_gt_in_public_inference` | 额外 GT 字段被拒绝/剥离，推理不访问监督文件 |
| `test_invalid_direction_training_sample` | action-only 回退保留动作样本，无假答案 |
| `test_inference_fallback_reset` | 格式错误有状态记录，回退不读取 GT，不污染缓存 |
| `test_language_perturbation_passthrough` | 扰动后的真实指令到达策略，不按 task_id 替换 |
| `test_question_roles_and_referring_expression` | 操作/放置角色正确，消歧表达不被丢弃，非前两名词规则 |
| `test_online_questions_independent_of_gt_binding` | 更换离线 GT 关联元数据不改变在线问题 |
| `test_qc_stage_coverage` | 分阶段/槽位缺标可见；同一人工题目不重复计数 |

λ_A=0 不等于 A 关闭；测试与配置不混淆这两个条件。

### 5.7 冻结前的离线方向条件诊断

F3 用开发 SA/SAB 模型在少量固定验证观测上（起点约 100 个）比较预测方向、GT 方向、预设合法错误方向。保持原图/状态/指令、槽位和动作目标不变，记录方向准确率、动作段 loss 和解码动作差异；错误序列须报告实际改变比例。不得把伪造输出当模型真实预测。

结果仅用于检查方向预测与动作条件化是否按设计工作，不设“GT 必须最好”或“必须达到某个差异”的伪门槛。GT/错误方向路径为隔离的 diagnostics 入口，正式服务不暴露替换答案能力。诊断可重复小前向，不能增加正式训练模型或用最终测试调权。额外闭环诊断可选。

变化大既可能是利用方向，也可能是错误干扰；少量观测变化小不否定训练期作用。不把敏感性当成有益性证明。

## 6. B 模块工程规格

### 6.1 教师侧

冻结 FastVGGT，eval/inference 模式、无梯度。每条样本输入同一 t 的真实两视图；不输入未来帧、GT 深度/点云或学生不可用的额外相机。视图顺序固定，不能按输入文件系统排序随意变化。

缓存取完整稠密几何隐表征，不取对象检测结果，也不只取最终深度标量。优先考察末端完整聚合特征，记录具体层、D_teacher、token offsets 和恢复规则。FastVGGT 的合并开关语义以固定源码为准，不能假定数值 0 必然表示关闭。[T1]

若需要调整加速设置以得到可对齐网格，在试缓存阶段完成并记决策；不能悄悄换成不同教师却仍标 FastVGGT。没有还原到完整视图网格的 merged tokens 不可直接作为逐位置目标。

T04 按第三人称/腕部视图分开报告有效位置、非有限值、零范数、归一化后空间及跨样本变化，并分视图展示可用原生深度/点图。范数变化不排除余弦方向近常量，平滑区域相似也不自动退化。记录实际运行合并路径、配置、视图顺序与恢复；源码中的视图不对称不等于腕部必然较差，恢复稠密位置不等于撤销信息聚合。不引入额外视图权重或探针。

### 6.2 坐标映射

教师/学生的 crop、resize、padding 都要可追溯到同一原始图像。对每个学生 patch 中心，经学生预处理逆映射到原图，再经教师预处理映射到教师特征网格，用固定插值方法取目标。

行对应 y、列对应 x；明确像素中心 convention 与插值 `align_corners` 等选项，不凭经验省略。用 synthetic 坐标场（特征通道分别编码 x/y）测试 crop/resize/翻转映射，避免把真实模型特征“看起来相似”当正确性证明。

缓存前下采样到学生真实 patch 网格，避免每次训练做大网格变换。默认 FP32 运算后存 FP16；记录是否 L2 归一化及在插值前/后的顺序，固定后不混用。

### 6.3 两种不同 mask

`model_image_attention_mask`：保持 π₀-FAST 原适配行为，四组一致。

`alignment_valid_mask`：只标真实图像对应的有效 patch；不包括零占位相机、padding、教师特殊 camera/register token。

当前官方 FAST 适配可能让零占位相机在主干 attention 中仍为 true；因此不能直接用它筛对齐位置。[O6] 真实两视图可以是 `base_0_rgb` 与 `left_wrist_0_rgb`，但实际名称以 checkout 和运行时 trace 为准。保存 `camera_layout.json`：键名、是否真实、token 起止位置、网格形状和排列。

q=1 的位置等权，q 不依赖任务、物体、前景、显著性或教师置信度。离线发现少量损坏目标时先报告并固定质量排除清单；正式运行遇到新的 NaN/损坏缓存应报错，不能用 broad exception 静默置 q=0。

### 6.4 学生侧

核对模型层数/宽度。当前公开 Gemma-2B/LoRA 起点为 18 层、2048 维；本计划选择第 12 层输出（索引 11）。[O4]

保持扫描模块与原参数树；在一次训练前向中返回选定层视觉隐状态。只保留必要层和真实视图，不能为了方便返回全部 18 层或多做一遍完整前向。检查 rematerialization/JIT 后实际内存，而不是只看 Python 层列表长度。

投影头提议：LN(2048) → Linear(1024) → GELU → Linear(D_teacher)。它只用于训练，与教师一起在正式推理导出时移除。此处为项目工程结构，不冒充 Spatial Forcing 完整原结构。[R2]

### 6.5 损失与反传

对每个样本，将所有有效真实 patch 的 `1 - cosine(projected_student, stop_gradient(teacher))` 求平均，再对 batch 求平均。关键归一化 FP32、eps 初始 1e-6；零范数和非有限值由缓存 QC/运行断言处理。

λ_B 从前 2,000 次有效更新的 0 线性升至 0.1，固定后 SB/SAB 的有效更新 step 含义一致。梯度路径测试使用非零 λ_B 或直接 L_3D，不在 warmup 第 0 步误判“无梯度”。

冻结原 SigLIP 与 Gemma 非 LoRA 参数。对齐发生在可训练 LoRA 影响的 Gemma 隐状态，而非完全固定的视觉编码器输出。只测投影头梯度不够；至少检查选定层上游共享 LoRA 的梯度与参数差异。下游层不直接收到 L_3D 梯度、LoRA 部分矩阵初始梯度为零均不自动构成错误。

### 6.6 B 验收测试

| 测试名建议 | 最低断言 |
|---|---|
| `test_b_disabled_matches_baseline` | 同共享权重、B 关闭的 logits/action loss 等价 |
| `test_zero_alignment_weight` | λ_B=0 不改变对应动作损失；不要求新投影头参数相同 |
| `test_dense_grid_mapping` | synthetic 坐标场与真实叠图均通过 |
| `test_real_camera_spans` | 对齐 gather 只含真实视图，顺序无误 |
| `test_alignment_masked_targets_invariant` | 改 q=0 的目标特征不改变损失 |
| `test_alignment_uniform_weights` | 所有有效 patch 同权，无任务加权字段 |
| `test_alignment_student_gradient` | 教师无梯度，共享 LoRA 存在有效梯度/更新 |
| `test_visual_prefix_no_future_leak` | 固定前缀，换 GT 动作/方向后缀，视觉隐状态不变 |
| `test_cache_id_and_hash` | 错 sample_id、版本、视图序导致显式错误 |
| `test_no_teacher_at_inference` | 无教师环境/缓存目录仍能加载 SB/SAB 策略 |

掩码不变性测试改的是无效目标特征，不是修改仍在主干 attention 中可见的占位图像；两者不要混淆。

B 硬门槛是接口、对应关系、数值、梯度/更新及稳定性正确；收益由正式四组实验检验，不要求提前胜过 S。Spatial Forcing 的 π₀ LoRA/RoboTwin 结果是相邻证据，不能替代当前 π₀-FAST 验证。[R2]

## 7. 教师缓存详细合同

### 7.1 阶段顺序

1. 100 个观测：只验证接口、形状与数值。
2. 1,000 个观测：完整读取—提取—网格映射—写盘—读回计时。
3. 100 张跨任务/视角/阶段叠图及 synthetic 映射测试通过；另抽约 30—50 帧保留固定教师可提供的原生深度/点图和特征分布排错材料，无接口则记录限制，不另训探针。
4. 固定教师 commit/权重/层、输入预处理与目标网格。
5. 全量 train 缓存；val 单独缓存，用于开发诊断，不并入 train。
6. 验证每个正式训练 sample_id 的目标和哈希；通过后写 completed 标记。

### 7.2 cache manifest 必要字段

```text
cache_schema_version
sample_id, split, raw_frame_hash
teacher_repo_commit, teacher_weights_hash
teacher_feature_layer, teacher_feature_dim, teacher_merging_config
real_camera_order, source_image_shapes
student_preprocess_hash, teacher_preprocess_hash, grid_mapping_hash
target_grid_shape, special_token_exclusion
feature_dtype, normalization_spec
shard_path, row_index, feature_shape, alignment_valid_mask_location
shard_checksum, completed
```

不得把整个文件路径暴露为模型输入。无监督需求的 S/SA 作业不打开大型 teacher shards。

### 7.3 存储与读取

采用分片 `.npy`/可 memory-map 数组或已验证的分片容器。格式在试缓存中依据读写速度选择，不为了“先进”引入大数据平台。一个 sample 一个小文件不作为默认。

每个分片先写 `.partial`，同步并校验后原子 rename；索引只指向完成分片。只允许从验证后的完整分片恢复，不把文件存在当完成。预取队列有上限，避免占满 RAM；不要同时运行过多 teacher cache workers 争抢同一 GPU。

本机工作区为 NFS，容量与吞吐分开审计；临时分片与最终文件须在同一文件系统。若从本地 staging 复制，先写目标侧临时文件、校验，再在目标侧原子提交，不能把跨文件系统搬运声称为原子 rename。

磁盘够大，不做为了省容量而删动作帧、借邻帧特征或暗中时间子采样。若吞吐有瓶颈先调分片/预取/本地读取，无法解决则作为正式协议变更提交，而不是改 loader 的样本集合。

## 8. 正式配置与损失集成

### 8.1 提议公共配置

下列 YAML 是**待实现项目配置草案**，不是可直接传给现有 openpi 的既有字段。实现配置解析/注册后才可运行。`null` 项在协议锁定前必须用真实审计值替换。

```yaml
schema_version: xiyuan_foundation_v3
protocol_id: foundation_spatial_v1
protocol_status: draft
model:
  family: pi0_fast
  framework: jax
  base_checkpoint: pi0_fast_base
  base_checkpoint_hash: null
  freeze_siglip: true
  freeze_gemma_non_lora: true
  trainable_parameter_manifest_hash: null
  baseline_contract: project_s_strict_v1
  action_decode_errors: explicit_status
  sequence_overflow: error
  lora_rank: 16
  lora_alpha: 16
  action_dim: 7
  action_horizon: 10
  execution_horizon: null
  max_token_len: null
  action_decode_budget: null
  dtype: bfloat16
  ema: false
training:
  effective_batch_size: 16
  micro_batch_size: null
  accumulation_steps: null
  num_effective_updates: 20000
  seeds: [0, 1, 2]
  optimizer: adamw
  beta1: 0.9
  beta2: 0.95
  eps: 1.0e-8
  weight_decay: 0.01
  learning_rate_peak: 3.0e-5
  learning_rate_end: 3.0e-6
  learning_rate_warmup_updates: 1000
  gradient_clip_global_norm: 1.0
  log_interval: 50
  save_interval: 1000
  retain_updates: [5000, 10000, 15000, 20000]
  loss_reduction: per_sample_then_batch_mean
  deterministic_image_preprocessing: true
  preprocessing_runtime_audit_hash: null
direction:
  enabled: false                  # sa/sab 覆盖为 true
  head: shared_lm_output
  lambda: 0.3
  max_question_slots: 2
  slot_roles: [manipulated_object, placement_target]
  preserve_full_referring_expression: true
  online_question_source: current_instruction_only
  relation: object_relative_to_eef_in_base_axes
  labels: [left, right, front, back, up, down, grasped]
  protocol_hash: null
  invalid_train_sample: action_only_packing
  inference_failure: logged_action_only_reprefill
alignment:
  enabled: false                  # sb/sab 覆盖为 true
  teacher: fastvggt
  teacher_commit: null
  teacher_weights_hash: null
  teacher_feature_layer: null
  teacher_feature_dim: null
  teacher_merging_config: null
  real_camera_order: null
  student_layer_1based: 12
  projector_hidden_dim: 1024
  projector_norm: layernorm
  projector_activation: gelu
  lambda: 0.1
  warmup_updates: 2000
  weighting: uniform_valid_tokens
  cache_dtype: float16
  cosine_compute_dtype: float32
  epsilon: 1.0e-6
  cache_manifest_hash: null
data:
  suite: libero_spatial
  train_manifest_hash: null
  val_manifest_hash: null
  normalization_hash: null
  transforms_hash: null
  sample_schedule_hashes: null
evaluation:
  primary_tier: P1-Lite
  core_packs_per_dimension: 1
  extension_packs_per_dimension: 1       # 仅预登记可行扩展；不存在则锁定为 0
  target_units_per_task_pack: 20        # 目标，不是假定每配置必有 20 初态
  actual_count_source: deduplicated_manifest
  sampling_mode: audited_official_configs_and_resets
  within_pack_config_weighting: equal
  unit_fingerprint_spec_hash: null
  inventory_hash: null
  dependency_cluster_manifest_hash: null
  bootstrap_spec_hash: null
  extension_activation: preregistered_resource_conditions_only
  extension_manifest_hash: null         # 无扩展时允许为空
  core_manifest_hash: null
  development_manifest_hash: null
  final_checkpoint_rule: fixed_update_K
  infrastructure_max_extra_retries: 2
execution:
  progression: stage_gates
  phase: F0
  unattended_queue_enabled: false      # runner 验证并由负责人批准后才启用
  approved_job_list_hash: null
  approved_gpu_hours_limit: null
  max_concurrent_major_jobs: 2
  approval_record: null
```

S/SB 配置应将无效的方向协议字段排除或保留为未使用状态；S/SA 同理不要求教师字段。正式 validator 对**启用分支**的关键 null 报错，不能为关闭分支强制下载标注/教师。无可行扩展时 `extension_packs_per_dimension=0` 并记录原因，扩展清单 hash 可为空；无人值守未启用时相关授权字段可为空。phase/队列授权是运行调度元数据，与模型/数据/评测语义 hash 分开计算，不能因从 F3 推进到 F4 就把科学协议改版。

共同 config 继承后，输出四份 resolved config，并自动 diff。除实验因素、对应字段、run_id/seed、附加投影头参数外，发现共享训练/数据/评测项差异要阻止启动。

### 8.2 训练损失合同

L_action：每样本只对基线定义的动作段有效 token 求均值，再 batch 平均。L_dir：每样本对方向回答段求均值，无方向样本为 0，再 batch 平均。L_3D：每样本对所有有效真实视觉位置求均值，再 batch 平均。

不要把三种 token/位置合并到同一个分母；不要让长动作序列冲淡 λ_A；不要因为 A 缺标签就改变动作样本权重。保存三项原始 loss、加权 loss、有效计数和每样本输出用于单元测试。

### 8.3 梯度累计与优化器

`effective_batch = micro_batch × accumulation_steps × data_parallel_replicas`。两张卡独立训练时每个作业 replicas=1，不把两组不同模型的 batch 相加。

实现累计时，每个微批次贡献按其样本数加权；一次有效更新只更新一次 optimizer、scheduler、EMA（本版关闭）和全局步号。全局裁剪在累计之后。大 batch 与累计的梯度测试在相同数据/随机设置下比较，不让 dropout 或采样不同混入。

freeze 参数表输出 name/shape/dtype/trainable/group/count，并保存短训练前后每类参数的摘要与实际差异；不能只生成预期表。冻结叶子必须不变，共享 LoRA 必须存在真实更新，B 仅额外更新预期 projector。optimizer 的 weight decay、mask、学习率组都写入 resolved config。缺失基础权重只允许项目新增头/LoRA 预期项；广泛 `strict=False` 会隐藏加载错误，不允许。

### 8.4 全训练路径可复现

记录有效步、已消耗 sample_id、loader/sampler 状态、模型 RNG、optimizer、scheduler。尽量让样本计划由 seed 和有效步直接确定，避免预取造成恢复跳样本。

恢复测试：训练 N 步保存，继续 M 步；独立进程从 N 恢复再跑 M 步，比较样本序列、步号和 loss/参数。在数值非确定场景中披露实际容差与设备条件，不假称逐位一致。

### 8.5 基线趋势与辅助梯度尺度

G1 的 S 小样本/1k—3k 开发检查只证明链路正确且可学习。G2 前保留多个开发检查点的动作 loss、闭环表现和失败类型，解释选择 K 的依据；必要时在有限预算内补开发检查，不要求完全收敛或追平论文。正式结论区分固定预算/学习效率收益与最终性能上限。

在少量固定开发批次，以共享 LoRA 参数 θ 分别测 `||∇L_action||`、`||λ_A∇L_dir||`、`||λ_B∇L_3D||` 及后两者相对动作梯度的比值。保存有效样本/位置比例、当前实际 λ_B、参数集合和测量步号；动作梯度近零标比值不稳定。低频诊断可增加小前向，不能改变正式 B 单次学生前向路径。

无需规定未验证的最佳比值，不按 loss 数字大小判断实际梯度，不用“增强组必须胜出”选权重。只有开发期允许预登记备选的有限比较；冻结后发现问题按 bug/探索性协议纪律处理。诊断成本计入 pilot，不新增大规模搜索。

已有梯度可按成本选做余弦夹角：同模型、同批数据、同一共享 LoRA 集合，不含 projector；近零梯度标不稳定。正负夹角只说明局部关系，不作 G2 同向阈值，不自动引入梯度手术。20k×16 表示约 32 万次样本抽取；用真实有效动作起始样本数和采样规则解释曝光，注明相邻帧/动作块相关。单终点只支持固定预算性能差异，“学习更快”需曲线；G2 前改变 K 时同步检查学习率和 checkpoint 日程，不无限延长开发。

## 9. 仿真工程规格

### 9.1 服务隔离

policy server 运行在 policy-train 环境；sim-clean 与 sim-plus 仅加载客户端。服务元数据包含 model_variant、training_seed、checkpoint_hash、protocol_hash 和动作 schema，不包含可用于作弊的任务答案。

一个服务一次只托管明确定义的模型，动态换权重必须 flush 队列并重新确认 hash。默认不把四个模型同时塞进一张卡赌显存。并行 worker 共享同一模型服务时必须确认请求安全及批量协议，不能依赖未验证的线程安全。

每回合 reset 动作队列、缓存、RNG；请求携带 episode/step 序号供审计。多个 worker 的 KV cache 若有持久状态，必须按 episode 隔离，不能复用到另一场景。

### 9.2 先库存，再冻结正式 manifest

T08 拆为前置资源库存与后续执行器：**F1 就盘点，不能等四组训练完成后才核实初态数量。**逐任务/维度列出官方配置 ID/资产路径、API 加载 shape、源初态文件、唯一 reset 数、生效的随机因素、源初态映射、可用开发/最终单元数。公开 Plus 接口对部分 `_add_`/`_level` 资源执行 `reshape(1,-1)`，实际 checkout 必须复核。[B3]

正式清单建议字段：

```text
protocol_id, eval_id
base_suite, base_task_id, task_asset_hash
dimension: clean|layout|camera|robot_initial|language|light|background|noise
variant_pack_id, official_config_id, variant_asset_hash
initial_state_id, initial_state_hash, source_init_asset_hash
rollout_seed, effective_randomization_spec, effective_randomization_key
instruction_raw_or_asset_ref, instruction_hash
camera_config_hash, reset_policy_hash, reset_hash
unit_fingerprint, base_init_cluster_id, cluster_mapping_evidence
within_pack_config_weight
max_env_steps, control_frequency, execution_horizon
video_capture_rule
```

`unit_fingerprint` 根据实际生效配置（含原始语言）、reset 状态、控制与随机化实现生成；确定性路径不用未生效 seed 制造唯一性。四组和不同训练 seed 共享环境清单是合法配对，去重检查针对清单内部的同一环境单元，不把四组运行本身删成一次。

`base_init_cluster_id` 保留跨干净/扰动条件的共同底层状态来源；扰动后 reset 哈希可以不同，不能靠相等比较推断所有关联。未知映射明确标记并在统计方案采用保守处理。仅不同单元不保证独立，重复稳定性诊断不进入主表分母。

每个变体包可含多个同维度官方配置；包内每个配置先平均单元，再按冻结权重（默认等权）汇总，不能把“一个初态的 20 次重放”当 20 初态。目标数不足可用更多预登记配置或实际数量，冻结实际清单与公式，而非伪造 19,200/36,000。

Plus 官方每任务一次试验指扩展后的官方任务/配置，不代表每个基础任务每维只有一个单元。[B2] T08 与 T02 联合完成第 4.1 节训练—开发—最终初态关系审计；不以 episode split 自动推断初态隔离。

语言扰动透传原文；机器人初态扰动保留官方状态，不为配对改回干净。配对是同 eval_id 的四组一致。没有官方强度依据只称 v1/v2。缺配置/纹理/reset 错误在 G2 前暴露，不用自造噪声冒充官方变体。

clean/Plus 匹配须检查控制、资产、成功判据与 reset；若存在官方 Plus 无扰动匹配配置，可在开发匹配支持上作单列参考，不替换原版 C、不自动增加正式回合。匹配不足时 C−R/G_drop 不能全归因于扰动，差分不保证消除模型×环境实现交互；同 Plus 协议内四组比较仍可报告。布局/机器人初态改变动作需求，七维统一称扰动鲁棒性/适应表现，不要求全部动作不变。

### 9.3 结果与错误 schema

```text
protocol_id, eval_id, model_variant, training_seed, checkpoint_hash
attempt_id, started_at, ended_at, worker_id, physical_gpu_ids
status: completed|infrastructure_error
success: true|false|null
failure_type: timeout|policy_nan|action_decode_error|policy_no_output|task_failure|...
environment_steps, elapsed_seconds, inference_latency_summary
direction_parse_status, direction_fallback_count
action_decode_status, action_decode_error_reason
action_shape, policy_config_hash, simulator_commit
video_path, log_path, exception_summary
```

只有 completed 才有明确 success；基础设施错误保留 null 并等待相同 eval_id 的规范重试。策略错误不能伪装成 infrastructure_error。明确重试上限（默认额外 2 次）；按 first valid completed attempt 计数，保留全部 attempts。

原始 JSONL 只追加不覆盖。汇总器基于逻辑键去重，发现同键多条冲突 completed 结果应报警，不自动选择成功记录。

### 9.4 覆盖与统计纪律

每组/seed/维度/变体报告 expected、completed、policy_failures、infrastructure_missing。缺失补齐前不要输出看似满额的主表。持续缺失时报告共同完成集合及缺失敏感性，不各自删掉困难样本。

视频规则固定，例如每任务条件前 2 个 eval_id 保存，另按统一规则保留失败样本；不只保存成功视频。不需要为全部正式回合开高成本视频编码。视频节约只改变日志成本，不改变输入图像、最大步数或成功判据。

### 9.5 仿真测试

| 测试 | 要点 |
|---|---|
| 专家回放 | 数据与当前控制链路匹配 |
| 干净/Plus 匹配 | 无扰动情况下接口和场景参数可解释 |
| 七维各单例 | 参数实际生效，资产确实加载 |
| 固定 init/seed | 四组同 eval_id 相同 reset 输入；随机因素确实生效 |
| 真实库存/重复 reset | API shape、唯一状态/配置、无作用 seed 重复可被发现 |
| 关联簇/开发隔离 | 底层状态来源可追溯；不以不同 ID 冒充开发/最终隔离 |
| 并发隔离 | worker 间状态/缓存不串场景 |
| 续跑去重 | 重启只运行缺失键，重复调度不增加分母 |
| 错误分类 | NaN/非法动作是策略失败；缺资产是基础设施错误 |
| 指令透传 | 改写文本保留，task_id 不恢复标准指令 |
| 无监督推理 | 不挂载标签/教师目录仍正常 |

## 10. 任务包与完成定义

T00—T12 保持原编号；阶段对应 F0—F6，不对应某个自然日。任务在产物与测试存在时才设 DONE。独立分支/worktree 开发；公共接口先统一并由集成人合并。以下路径均为拟新增，实际名称改变时同步阶段日志中的命令/路径记录。

### T00：现状、硬件、版本与资源盘点（F0）

**输入：**当前仓库/服务器。**动作：**只读检查 Git/GPU/环境/数据/权重/占用；确认环境隔离、两卡假设、既有脚本与旧结果。

**输出：**`artifacts/audits/hardware.json`、`versions.json`、`paths.json`、F0 阶段日志及现有产物复用清单。

**验收/暂停点：**每个已有项有真实路径/版本；不确定项可见；未覆盖环境/用户文件或抢卡。保存下一最小任务。若一张卡或不连续可用，立即更新卡时模型，不直接继承双卡估时。

### T01：项目 S 与环境、严格基础行为（F1）

**依赖：**T00。**动作：**固定 openpi、隔离环境与基础权重；未加 A/B 的前向/loss/FAST 合法回环；第 3.4 节真实 freeze 白名单、严格解码与长度检查、内部确定性预处理；建立策略服务/单回合。

**输出：**环境/base 哈希、项目 S 数值 fixture、实际命令、回放/策略视频、参数表和短训练前后更新审计、解码负面测试、训练入口图像审计。

**验收：**确为 π₀-FAST，未借 pi05；合法编码与上游一致；异常显式失败而非零动作；冻结原参数不变、LoRA 更新；可保存/恢复。dummy smoke 可与 T02 并行，真实小训练待数据。

### T02：数据、回放、清单、归一化与采样（F1）

**依赖：**T00，部分环境来自 T01。**动作：**核实 10 任务及轨迹，审计当前 RGB/位姿时序；episode split、稳定 sample_id、共享 norm、100 样本动作回环、采样 schedule。与 T08 同步开发/最终资源划分。

**输出：**train/val manifest、norm_stats、preprocess spec、camera layout、回放 QC、S 小样本及多检查点开发日志。

**验收/G1：**无 split 泄漏/跨 episode 动作块；state/action 语义正确；四组同 seed 的采样规格/前若干批 ID 相同（可用 loader fixture 验证，不依赖 A/B 模型已实现）；S 闭环正确且能学习；七维资源库存有初步结论。不把 G1 写成 K 已充分收敛。

### T03：方向角色、实例关联、标注与人工 QC（F2）

**依赖：**T02/G1。**动作：**定义基座轴/夹爪原点/七类；按操作/放置角色保留完整指代；在线问题构造与离线实例绑定分离；当前状态标注；invalid 原因与阶段覆盖；至少 300 问题三层 QC。

**输出：**`direction_protocol.json`、标注 shard/index、QC 图/审核表、任务×阶段×槽位覆盖、类别/invalid 分布。

**验收/人工点：**坐标/时刻/实例正确，无未来和隐藏 GT 回写文本；action-only 保留动作。人工审查可分批，未完成写 BLOCKED_HUMAN；已生成清单不等于人工通过。关键阶段缺标有分析与采用决定。

### T04：FastVGGT 试缓存、教师质量与全量缓存（F2，安装可在 F1 并行）

**依赖：**T02 的稳定 ID/预处理。**动作：**100/1,000 观测试缓存、层/合并机制、稠密坐标映射、真实两视图/特殊 token；100 叠图/synthetic 映射；少量原生深度或点图/特征检查；通过后原子分片全量缓存。

**输出：**teacher spec、cache manifest、网格/教师排错图、测试、真实吞吐/显存/读取报告和完整标记。

**验收/续接：**同样本提取—缓存读回一致，错 ID/hash 显式报错；无未来图像与额外区域缓存；正式训练样本全部覆盖。中断只续写未通过校验分片，不能因文件存在就跳过。

### T05：A 的 tokenizer、loss、生成与回退（F2）

**依赖：**T01、T03 协议；可先用明确 synthetic fixture。**动作：**前后缀、两类 mask/shift、完整方向候选序列、多槽位、FAST 隔离、回退重预填充。

**输出：**A 模块/全部 A 测试、200—500 样本小训练日志、至少 20 个开发闭环回合。

**验收：**A 关闭恢复项目 S，λ_A=0 与结构关闭区分；因果性/无 GT 推理、多 token/槽位、invalid 与回退通过；不能绕开共同严格 FAST 解码。

### T06：B 单层、投影、loss 与共享 LoRA 梯度（F2）

**依赖：**T01、T04 特征合同。**动作：**scan 返回第 12 层、真实 camera spans、均匀余弦、SB/SAB 有效步 warmup、freeze/梯度检查。

**输出：**B 模块、参数/更新报告、disabled/λ=0/causality/mask tests、小训练日志。

**验收：**正式训练一次学生前向；教师不反传；共享 LoRA 有梯度/更新；无占位视图/任务权重；不以 projector loss 独自下降验收。

### T07：四组集成、诊断、恢复与协议冻结（F3）

**依赖：**T05/T06、T02/T04 正式数据、T03 人工 QC、T08 清单、T09 启动检查。**动作：**四组配置 diff；各 200—500 有效更新 smoke（profile 稳态至少 300）；共享初始化/采样；真实 freeze/参数更新；内部预处理；累计/恢复；无 GT/教师推理；四组 profile；S 曲线和第 5.7/8.5 节诊断。

**输出：**resolved configs、共享初值/参数表哈希、诊断报告、G2 证据、`protocol.lock.yaml`、默认 P1-Lite 真实 manifest 与可行扩展、聚合/bootstrap 方案、正式队列和剩余卡时。

**验收/人工点：**启用分支关键字段非空；无越界配置差异；所有测试/QC 有证据；K 的学习阶段/预算解释清楚；负责人确认协议及可无人值守范围。没提升不构成不通过；真实 bug/泄漏/参数错训构成阻塞。

### T08：评测资源库存、执行器、manifest 与错误处理（F1 前置，F2/F3 完成接口，F4/F5 执行支持）

**依赖：**资源库存只需 T00 和仿真资源；闭环需 T01 策略服务。**动作：**先盘七维官方配置/真实初态/随机性/关联来源；开发/最终隔离；之后 clean/plus 路径隔离、七维单例、固定 manifest、结果 schema/重试去重、并发、完整吞吐。

**输出：**`artifacts/audits/eval_inventory.json`、reset/duplicate 审计、核心/扩展/开发清单、cluster 映射和统计规格、环境 spec、错误 fixture、profile。

**验收：**四组同 eval_id；维度确实生效；不同 ID/无作用 seed 的重复 reset 可发现；真实指令透传；失败不漏计；无 GT 输入；并发不串；资源不足不伪造计划回合数。

### T09：最小资源队列、命令登记与暂停/恢复（F2/F3）

**依赖：**T00 与 T07/T08 的接口草案（不依赖它们已通过 G2，避免循环依赖）；fixture 可先做。**动作：**run_id、GPU lease、日志/心跳、依赖 gate、工时、固定命令 registry、checkpoint/resume 与有限错误恢复；登记授权队列和资源上限。

**输出：**queue/job schema、最小 runner 或已验证现有调度器适配、dry-run、启动前检查、会话交接模板。

**验收：**不抢卡、不覆盖、不无限重试、不重复启动活跃作业；G2/人工门槛未过不长训。不建设通用调度平台，不为每个包装脚本重复造轮子。无人值守只执行批准列表，遇到协议/数据损坏停止相应项目任务并留证据。

### T10：正式训练、核心评测与条件扩展（F4/F5）

**依赖：**G2、资源队列和评测启动条件。**动作：**首轮 seed 0 四组与 P0 核心优先，接着 seeds 1/2；已完成模型流水线评测；首轮及资源变化后重估；有资源才执行已登记 P1 扩展。

**输出：**12 个计划模型或明确统一缩减范围、逐回合记录、训练曲线、成本/版本账本；G3a/G3b/G4 证据。

**验收：**同 K/同数据，无测试挑权重；不按效果删 seed/条件；同键结果不重复；旧失败/失效版本保留。暂停用完整 checkpoint 精确恢复，不能重新计步或换样本序列。

### T11：统计、成本与失败分析（F1/F2 可做 fixture；F4/F5 增量；F6 定稿）

**依赖：**结果/cluster schema；真实分析需 T10。**动作：**手算 fixture 验证包内官方配置权重、macro/增益/交互/G_drop、相关状态配对重采样、attempt/缺失；真实数据逐 seed/共同覆盖/成本与失败分析。

**输出：**四组/七维/逐 seed/效应/交互/G_drop 表，区间与重采样版本，缺失/成本/失败数据和报告。

**验收：**不被回合数不等支配；不把 rollout 当训练重复或把关联单元当独立；诊断 GT 不进主表，不制造显著性。G4 前的输出明确进行中，最终表对应锁定账本。

### T12：无教师导出、复现与交付（F6，接口可提前准备）

**依赖：**正式有效权重与 T11。**动作：**新进程/新运行目录验证 S 冒烟、四组加载及少量固定 eval_id，无方向标注/教师目录；整理实测命令和全部产物索引。

**输出：**F6 日志中的复现命令及交付索引、代码/权重/配置哈希、数据/缓存/协议/结果索引、复现报告、结论边界和真实未完成清单；G5。

**验收：**无仅当前 shell 可运行的隐藏步骤；人工审核结果解释；不声称必须 SAB 最好。只准备交付，未经另行授权不推公网。

## 11. 测试分层与数值标准

### 11.1 测试层级

L0 CPU/小 fixture：schema、split、sample_id、direction rule、mask、序列边界、结果去重、统计公式。

L1 GPU 小模型/单批：中间层、梯度、因果性、归一化、累计、同权重关闭分支等价。

L2 实际基础权重：前向、动作回环、模型加载、短训练/续跑、显存峰值。

L3 仿真闭环：专家回放、S、A、B、AB 快测；七维单例；并发与错误恢复。

L4 正式协议：被冻结的 K、seeds、manifest 和 checkpoint，只有通过前面层级才能启动。

### 11.2 容差与失败处理

token ID、mask、sample_id、配置哈希、shape 应精确一致。浮点比较可从 FP32 `rtol=1e-5, atol=1e-6`，BF16 路径约 `rtol=1e-2, atol=1e-2` 开始核查，但需结合上游重复前向本身的误差设置，不能遇到失败就不断放宽阈值。

对禁用分支的确定性路径，优先保持完全相同计算；若 logits 在近并列点导致 argmax 不同，保留差异和原因，不简单用“混合精度”解释所有动作不一致。

必须包含负面测试：错误 cache_id、错视图序、后缀泄漏、GT 注入、重复 eval_id、缺资产、非法动作、断电半分片、恢复跳 batch。只做 happy path 不足以通过 G2。

### 11.3 合成统计 fixture

至少提供一个人工四组小表，能够手算 R、ΔA、ΔB、ΔB|A、ΔA|B、I_AB 和 G_drop；构造包内多个官方配置、初态数量不等的表，验证配置/包/任务/维度的冻结权重；构造重复 reset、无作用 seed、重复 attempt、infra 缺失，验证分母；构造跨条件共享初态簇，验证重采样时四组和相关条件一起移动。

CI 或测试输出不得写入伪造实验成功率。fixture 必须放测试目录并明确标 synthetic，formal 汇总器拒绝 synthetic 来源。

## 12. 正式作业和双卡调度

### 12.1 作业元数据

每个 job 至少包含：run_id、job_type（pilot/formal/eval/cache/analysis）、phase（F0—F6）、variant、seed、科学协议/配置哈希、checkpoint 来源、GPU 数量/物理 ID、预计 GPU·小时、CPU workers、输出目录、依赖 gates、已验证命令、状态、PID/日志/心跳、恢复入口、授权记录与队列卡时上限。phase 是调度状态，不混入不可变科学协议 hash。

run_id 示例仅为命名模板：

```text
fndv1_formal_s_seed0_<timestamp>
fndv1_formal_sa_seed0_<timestamp>
fndv1_eval_s_seed0_<checkpoint_short_hash>_<manifest_shard>
```

同名目录存在默认报错；恢复必须显式声明 resume 并确认配置/数据/协议哈希。严禁默认 `--overwrite`。

### 12.2 队列优先顺序

先做两条并行训练槽：S/SA seed 0；空出后 SB/SAB seed 0。首轮四组完整优先于给某个组先跑完三个 seed。首轮权重通过加载后，插入 P0 核心评测，后续 seeds 排在统一队列中。

资源满足理想数量时 P0 约 6,400 回合，按中间吞吐约 53.3 GPU·小时；它属于默认 P1-Lite 19,200 的 seed 0 部分，也可复用于已登记 P1，不额外加预算。实际数量以清单为准。核心评测与后续训练只能错峰或分卡，不假设有第三张卡。

主训练结束后两卡主要评测。使用固定 shard 和 lease 防止重复调度；动态 worker 并发只用于吞吐，不改变输入、控制/动作块长或超时标准。

### 12.3 启动条件

正式训练需要：G1/G2、配置锁定、完整 train/cache（仅 B 组）、base hash、共享 norm、真实参数冻结/更新证据、QC 签收、内部预处理审计、dry-run 成功、资源空闲、预计工时与唯一 run_id。无人值守还需批准列表和上限，不因负责人不在线而自动扩大授权。

正式评测需要：checkpoint 已完整保存、推理无监督依赖测试通过、eval protocol 冻结、策略元数据核对、shard 未完成、资源可用。

发现数值 NaN、缓存损坏、错误 checkpoint、协议哈希改变或输入泄漏，先停止**本项目对应作业**并保留现场；不可删除日志后重跑制造连续成功记录。

## 13. 分阶段推进、会话交接与无人值守

### 13.1 固定阶段与出口

| phase | 任务主线 | 阶段出口 | 人工参与重点 |
|---|---|---|---|
| F0 | T00 | 真实现场/资源/状态清楚 | 仅处理无法从现场确定的授权或冲突 |
| F1 | T01/T02；T08 库存前置 | G1：项目 S 链路正确且能学，资源可行性清楚 | 看回放、控制语义和 S 多检查点趋势 |
| F2 | T03/T04/T05/T06；T08/T09/T11 无依赖部分 | A/B 独立测试、QC/缓存、单项 profile | 分批审核 ≥300 问题和必要映射证据 |
| F3 | T07，收齐 T08/T09，有限诊断 | G2：四组/协议/真实清单/统计/预算冻结 | 集中确认 QC、语义决策和自动队列范围 |
| F4 | T10 seed 0 四组及 P0 核心 | G3a：首轮同 K 权重与核心覆盖 | 看异常和卡时，不按效果调参 |
| F5 | T10 seeds 1/2、核心补齐；条件扩展 | G3b：训练范围；G4：结果/分母冻结 | 仅确认资源缩减/预登记扩展及异常处理 |
| F6 | T11/T12 | G5：报告、索引、复现和边界完整 | 审核结论与真实完成范围 |

不绑定自然日；一阶段允许多次会话完成。F4/F5 可流水线重叠：首轮四组优先，但已完成模型可评测，批准的后续 seed 可运行；无需等负责人每天登录。G3a 仍需完整记录，未过 G2 绝不提前正式训练。

### 13.2 每次进入的最短流程

检查适用规则与 Git 状态，从 README 进入当前阶段，读取 PLAN、LOG 顶部及最近记录，再按需查阅两份主文档。先看已在运行的 PID/心跳、已完成 checkpoint/shard 和协议哈希，防止重复启动。已有通过且依赖/版本未变的测试可引用证据，不机械全量重跑。

选择最早未完成且依赖满足的任务，拆成当前会话可闭环的一项工作。存在 BLOCKED_HUMAN 时只汇总待确认材料；继续无依赖任务，不假造签收。负责人不在线本身不是缩小研究范围的理由。

### 13.3 暂停前交接

写清 phase、Txx、已过 gate、真实改动/未提交分支、已运行命令/测试、协议和输入哈希；登记全部活跃作业的 run_id/PID/GPU/日志/心跳、最近完整 checkpoint、有效步号、缓存/评测已完成分片。指出下次应该“观察已有作业”“精确 resume”还是“运行下一条命令”，不能只写“继续训练”。

检查点保存按更新数或已验证的周期机制，不依赖操作者在线。用户要求停止机器作业时使用已验证的安全停止流程；不要随意 kill 他人或未知 PID。暂停中的不完整 checkpoint/shard 不作为恢复源。

### 13.4 无人值守授权边界

只允许实际实现并测过的服务器 runner 执行事先批准的固定队列，登记命令/代码/协议 hash、允许 GPU、最大并发、总卡时/作业数量、日志、检查点、有限重试和停止条件。未启用 runner、会话结束或仅写了文档，不代表后台会自动推进。

自动可做：固定参数训练/精确恢复、已锁定缓存分片、已登记清单评测/有限基础设施重试、校验/去重/生成进行中统计。不能自动做：人工 QC 签收、越过 gate、调超参/改词典/换 K、创建未登记扩展、修改环境/驱动、公开发布、无限重试。训练 NaN/数据损坏/协议不符先停止对应项目作业并保留证据；不能把评测中本应计失败的策略错误改成无限重跑。

阶段转移只更新调度状态，不改科学协议。已授权的 seeds 1/2 可在首轮模型具备条件后接续；P1 扩展仅在批准且满足预登记资源条件时启用。负责人长时间无空时可暂停等待，不自动取消差 seed 或降低 K。

### 13.5 负责人少量集中投入的使用方式

将人工工作集中为：F1 基础链路检查；F2 分批方向/实例/网格 QC；F3 协议与队列批准；F6 报告/复现审核。每次提供一页摘要和可定位证据，把未决语义与普通工程问题分开。其余固定机器作业可在授权后继续，不把“每天在线”当依赖。

## 14. 工时模型与决策规则

### 14.1 必须实测

训练：每组 JIT/预热单列，至少 300 稳态有效更新的 P50/P90、样本/s、峰值显存和缓存等待，JAX 计时阻塞到设备完成。教师：1,000 观测完整读/算/写/读回吞吐，据真实观测数估全量。仿真：完整 server+worker+renderer 的回合/GPU·小时，正式首轮后按成功/失败长度校准。

### 14.2 预算公式

```text
T_train_gpu_h = Σ_runs(remaining_updates × measured_seconds_per_effective_update
                      × allocated_gpu_count / 3600) + uncounted_compile_save_recovery
T_cache_gpu_h = remaining_observations / measured_observations_per_gpu_hour
T_eval_gpu_h  = remaining_unique_model_eval_keys / measured_episodes_per_gpu_hour
T_remaining  = T_train + T_cache + T_eval + remaining_pilot + uncounted_overhead
continuous_capacity_gpu_h = num_available_gpus × 24 × remaining_days × availability_fraction
```

同一开销不重复计入实测和 overhead。独立双作业不让总卡时除二；只在推算机器墙钟时间时考虑并发。无连续窗口时累计真实可用时段；再按依赖和人工验收等待估关键路径，不能仅用整月总卡时保证完工。

### 14.3 默认 Lite 与扩展预算

平均 6 秒/更新、K=20k、12 次单卡训练约 400 卡时。资源足够时，Lite 19,200 回合按 120 回合/GPU·小时约 160，P1 36,000 约 300。加冒烟/有限诊断 40、教师 20—40、可选闭环 0—10、缓冲 50—100：**P1-Lite 670—750，中心 700；P1 810—890，中心 840 GPU·小时**。不是服务器实测。

G2 后按训练+评测+后段 70 卡时缓冲估算：Lite 630、P1 770。两卡 70% 持续可用时约 18.8/22.9 个自然日等效资源占用，不含前置开发与后续人工交付，也不是无条件周期承诺。30 日两卡 70% 的理论 1,008 卡时仅作参考。

原先“一个月内尽快”的意图保留为优先推进，不转化为逐日考核。负责人没空时先顺延自然日，不自动降级；约一个月只有在开发/人工/机器窗口都满足时才可能成立。没有实际目标截止日，不凭空写完工日期。

### 14.4 事件触发重估

F1 初测、F3/G2 冻结、F4 首轮四组完成、可用 GPU/吞吐/窗口变化、长暂停后恢复时重估。报告真实剩余卡时、已批准自动队列、下一人工阻塞及完成区间。

先优化读取/服务并发/去重；取消可选额外闭环；已纳入扩展则退回默认 P1-Lite；仍有硬限制才统一两 seed，最后 P0。保留四组、七维、同 K/样本和失败分母。不按成绩删组/seed/配置。P1 扩展需核心不受影响、第二包已登记且有效、剩余资源允许。

K/学习率/方向规则仅 G2 前可作统一开发决定。G2 后看最终核心结果再调整必须新协议并标探索性，记录重训/重评影响。只有一张卡时 30 日 70% 约 504 卡时，重新选择自然日窗口或统一缩减，不沿用双卡承诺。磁盘够大仍需测 I/O。

## 15. 统计实现规格

### 15.1 主表聚合

对每个训练 seed：先在 task×dimension×variant_pack×official_config 内平均真实有效单元；再按冻结的包内配置权重（默认等权）得到包值；之后对包、task、七维等权得 R。包仅含一个配置时退化为 V2。干净 C 用固定清单、任务等权单列。不能直接 pooling 导致有更多初态/回合的配置或维度权重更高。

显示每 seed 的 C/R/七维、三 seed 均值/标准差、实际 N/覆盖。成本统计区分 teacher preprocessing、train、inference、simulation/renderer 与失败重跑。

### 15.2 效果计算

```text
Delta_A_given_no_B = R_sa - R_s
Delta_B_given_no_A = R_sb - R_s
Delta_B_given_A    = R_sab - R_sa
Delta_A_given_B    = R_sab - R_sb
A_average_effect  = 0.5 * [(R_sa - R_s) + (R_sab - R_sb)]
B_average_effect  = 0.5 * [(R_sb - R_s) + (R_sab - R_sa)]
Interaction_AB    = R_sab - R_sa - R_sb + R_s
G_drop_m          = (C_s - R_s) - (C_m - R_m)
                  = (R_m - R_s) - (C_m - C_s)
```

乘 100 输出百分点。R 仍为主指标，G_drop 只是绝对下降是否缩小的描述量；联合 C/R/七维/天花板解释，不把它单独当鲁棒性裁判。不要把扰动表现提高自动等同于敏感性降低，也不把正交互等同于显著协同。

### 15.3 重采样与检验

默认固定已选任务/官方配置的聚合权重，在任务内按可验证的 `base_init_cluster_id` 分层配对 cluster bootstrap；同簇跨干净/扰动条件及四组的记录整体移动，不逐条件独立抽。真实随机重复的层级、单状态配置的处理在 G2 前登记，不虚构条件内方差；计算 G_drop 也保持 C/R 的相关性。共享簇按覆盖结构分层或采用经 fixture 验证的等效方案，保证固定聚合单元可计算；不能遇到空配置就静默删配置并改权重。单单元重复重采样产生零宽区间时，明确其无法估计该条件内不确定性。

该区间是所选任务/配置下的条件性评测不确定性。任务层整块重采样可另报敏感性，不推断未见任务总体；状态来源关联未知时另报任务整块敏感性分析及适用限制，不能当独立。训练 seed 逐一展示均值/标准差/差值，不能靠更多 rollout 替代训练重复，只有 3 seed 的证据仍有限。

可默认做 10,000 次有固定随机种子的配对 bootstrap 来报告区间（计算量通常在 CPU 侧，实际实现先 profile），但该次数是本项目分析参数，不是统计显著性的保证。正式函数要保存重采样方案版本。

主对比 SA−S、SB−S、SAB−S、I_AB 预先登记；需要 p 值时另外设计合适的配对零假设检验，不从普通 bootstrap 符号比例冒充 p 值。多重对比用预先定义的 Holm 等规则，并保留未校正效果量。

T11 的 G2 前补充采用真实 manifest/关联簇做评测精度分析；跨训练方差未知时仅设明确标注假设的情景，一个开发 seed 不支持可靠跨训练方差或精确最小可检测增益。报告不同波动下小/中/大差异的判断限制，不承诺检出固定百分点，不新增功效 gate、训练模型或无限预实验。未显著不等于无有意义增益，后者需区间与预定义实际意义范围支持。

### 15.4 不得混入主表的结果

pilot、训练步数不同且未预注册的检查点、GT 方向替换、打乱方向、修改后未重锁协议、synthetic fixture、基础设施半回合、同逻辑键的重复尝试，均不得混入正式四组主表。

## 16. CLI 合同与实际命令管理

### 16.1 拟新增 CLI

以下是候选接口合同，不是必须逐个造出的包装脚本，更不是现在即可运行的文件。优先最小适配已有可靠入口；采用的接口才实现/注册，须有 `--help`、输入验证、结构化结果、非零错误退出码和不覆盖默认。T09 不建设通用平台。

| 拟新增脚本 | 责任 | 关键参数合同（实施后校验） |
|---|---|---|
| `audit_runtime.py` | 环境/资源审计 | 输出目录、只读 |
| `prepare_manifests.py` | 数据 split、stable ID | data root、suite、split seed、输出 |
| `annotate_directions.py` | 当前状态标签/QC | train manifest、direction protocol、输出 |
| `cache_teacher_features.py` | FastVGGT 缓存 | manifest、teacher spec、paths、shard、resume |
| `validate_cache.py` | ID/形状/hash/数值 | cache manifest、train manifest、报告 |
| `validate_protocol.py` | 四组配置、实际参数/预处理证据、真实清单/关联检查 | protocol、resolved configs、inventory、gates |
| `profile_pipeline.py` | train/cache/eval 全链路 profile | mode、variant、observations/updates/episodes |
| `run_training.py` | 项目训练包装/注册 | config、run_id、seed、resume |
| `serve_policy.py` | 项目学生服务适配 | checkpoint、policy spec、端口 |
| `run_evaluation.py` | fixed manifest 仿真 | protocol、manifest shard、server、run_id、resume |
| `summarize_results.py` | 去重/覆盖/主表/区间 | protocol、raw results、输出 |
| `launch_queue.py` | 最小 GPU/job 调度或现有 runner 适配 | queue、dry-run、授权列表、资源上限、resume |

实际实现以注册的入口为准。若使用上游 train/serve 入口而非包装脚本，应从表中删除未采用项并写清对应关系，不能同时维护两个行为不一致的训练入口。

### 16.2 命令模板的使用规则

先验证脚本存在并运行 `--help`，再写命令；自定义 schema 字段不自动等于上游 CLI 参数。未实现的脚本只登记为 planned，不列为阶段日志中的已验证命令。

**只有四组配置已经在实际 checkout 注册、且参数经 --help 核验后**，下列上游风格模板才可采用：

```bash
# 模板，不是当前已验证命令；运行前替换为注册后的真实配置名。
CUDA_VISIBLE_DEVICES=0 \
XLA_PYTHON_CLIENT_MEM_FRACTION=0.85 \
uv run scripts/train.py xiyuan_s --exp-name=REPLACE_WITH_UNIQUE_RUN_ID --seed=0
```

内存比例 0.85 只是起点，不是显存硬上限或与渲染并存保证。实际值依据独占/共享资源实测。不得复制上游示例的 `--overwrite` 到正式任务默认命令。

### 16.3 command_registry 建议字段

`name, script_path, status(planned|implemented|verified), docs_commit, implementation_repo_root, verified_implementation_commit, upstream_commits, help_log, smoke_command, smoke_exit_code, required_environment, required_gate`。verified_implementation_commit 是命令测试时的实际代码提交，不能填文档提交；上游命令尚未适配时记其所属仓库和实际上游提交，不伪造项目实现。runner 只执行 implemented 且通过必要 smoke 的入口，正式长作业还要满足相应 gates。

## 17. 排错顺序

| 现象 | 优先检查 | 禁止的捷径 |
|---|---|---|
| S 全任务相同异常动作 | 图像翻转、norm、delta 正逆、夹爪符号、权重与状态维度 | 换 pi05、先堆 A/B |
| A 训练正常推理差 | GT 进 prefix、teacher forcing 差距、分段、长度、对象解析、回退缓存 | 正式推理填 GT |
| A 标签覆盖低 | 当前帧/坐标/对象/阈值、抓持判据 | 猜标签、删除动作样本 |
| B loss 降低但控制无变化 | projector 独自学习、freeze、stop_gradient、层索引 | 仅报对齐曲线证明有效 |
| B 形状正确但 loss 异常 | 原图坐标映射、camera spans、unmerge、特殊 token | 直接 reshape、强行广播 |
| B 组比 S 少样本 | cache loader join、missing fallback、split | 接受不公平数据比较 |
| 显存溢出 | 并存进程、全层返回、文本长度、缓存 batch、物理 batch | 随机改动作块长或主干 |
| 仿真吞吐低 | CPU/render/I/O/服务并发、视频、重复作业 | 缩短单组超时、删难任务 |
| clean/plus 路径互相影响 | 用户级 LIBERO 配置、共享 HOME/资源路径 | 反复覆盖 pip 包 |
| 重复结果看似提高成功率 | logical key、attempt 分类、去重策略 | 保留成功重试删失败首试 |
| 工作窗口不足 | 先顺延自然日；有硬限制再按实测统一缩减 | 因操作者不每天在线就删 seed/改 K |
| 初态数远小于目标回合数 | 官方库存、实际 reset、随机因素、配置抽样 | 复制确定性 reset 或换无作用 seed 凑数 |
| 方向计算正确但行为异常 | 完整文本指代、离线实例绑定、阶段覆盖 | 只查坐标而不查目标对象 |

首次无法定位时保留最小复现、堆栈、版本、输入摘要与失败日志。将证据写入状态表，不猜测已经修好。

## 18. 每次提交和会话结束的格式

```text
阶段/任务：Fxx / Txx / 范围；当前 gate
实际变更：文件路径与接口
运行命令：真实执行过的命令
测试证据：日志路径、退出码、通过/失败/未运行
实验影响：数据/缓存/协议/检查点是否失效
资源使用：run_id/PID/GPU/心跳、实际/预计剩余卡时
暂停续接：协议/代码哈希、最近完整 checkpoint/分片、恢复入口
自动队列：是否真实运行、已批准范围、卡时/作业上限
剩余问题：尚未完成或待人工确认
下一步：最早可执行任务/已验证命令；待人工时标 BLOCKED_HUMAN
```

不要只说“完成模块”。未运行的测试写未运行，模拟数据写 synthetic，计划值写 estimate。没有真实产物路径不标 DONE；没有固定日志不声称“成功跑完一轮”。

## 19. 最终完成定义

四组实际完成的所有 seed 权重与同 K 证明；基础/adapter/投影头的元数据与哈希；固定代码/环境；共享数据/归一化/动作变换；方向角色/实例/标签及阶段覆盖、教师缓存的版本/QC；真实 freeze/更新/内部预处理/严格解码证据；P0/P1-Lite/P1 实际采用范围；官方资源库存、唯一单元/关联簇和所有正式 eval_id 的有效结果或缺失解释；四组与七维表；配对增益、交互、G_drop 和冻结的重采样方案；资源实测；失败视频；无监督目录推理验证；阶段日志中经新进程恢复验证的命令。

报告必须明确哪些是项目书技术主线，哪些是本基础阶段工程简化；不声称完整原论文复现、不声称未做的全量基准、不声称无对照即可证明唯一三维因果机制。

**执行原则：先把项目 S 的真实链路做对，再独立做 A、B，再组合；首轮完整四组优先，再补三 seed 核心；先冻结协议，再看最终测试。按阶段证据推进，可暂停续接，不以操作者每天在线或 SAB 必须最好为前提。**



---

## 资料依据与核对范围

**[P1] 项目书。**用户提供《曦源项目申请书》，原项目名称为“基于聚焦式 3D 表征对齐的视觉-语言-动作模型扰动鲁棒性实证研究”，共 20 页。本文引用第 2 页 LoRA 工作安排、第 9 页 2.1，第 9—10 页 2.2，第 10 页 2.3/2.4，第 11 页 2.6/3.1，第 12 页 3.2/3.3。原题目与原项目包含更多后续研究内容，**不代表本基础阶段要完成聚焦或真机**；本基础阶段范围以用户最新四组要求为准。本文未复制申请书中的个人联系方式、学号或签名。

**资料核对范围：**沿用 V2（2026-09-05）的项目书/方法背景引用；V3 于 2026-09-06 复核 openpi README、配置、FAST/Gemma/tokenizer、图像预处理与 LIBERO 适配，以及 LIBERO-Plus 初态接口和 Codex AGENTS 文档。其他论文与教师背景链接保留为来源入口，不声称本版重新逐项复现。以下地址不是依赖锁；服务器须记录实际 commit、权重/hash 和命令。所有阶段设计、阈值、超参、诊断和预算均为本项目工作方案/估算，非引用来源的实测结论。

| 引用 | 资料及本次用途 | 公开地址 |
|---|---|---|
| [O1] | Physical-Intelligence/openpi 官方 README：主模型、JAX/PyTorch 支持范围、基本训练/服务路线 | `https://github.com/Physical-Intelligence/openpi` |
| [O2] | 官方 `training/config.py`：π₀-FAST LIBERO/LoRA 起点、动作配置、归一化与 extra delta 处理 | `https://raw.githubusercontent.com/Physical-Intelligence/openpi/main/src/openpi/training/config.py` |
| [O3] | 官方 `models/pi0_fast.py`：前缀注意力、next-token loss、采样与冻结路径 | `https://raw.githubusercontent.com/Physical-Intelligence/openpi/main/src/openpi/models/pi0_fast.py` |
| [O4] | 官方 `models/gemma_fast.py`：Gemma-2B 层数/宽度、LoRA 起点与 scan | `https://raw.githubusercontent.com/Physical-Intelligence/openpi/main/src/openpi/models/gemma_fast.py` |
| [O5] | 官方 `models/tokenizer.py`：FAST 前后缀、mask 和动作抽取 | `https://raw.githubusercontent.com/Physical-Intelligence/openpi/main/src/openpi/models/tokenizer.py` |
| [O6] | 官方 `policies/libero_policy.py`：图像/状态映射、FAST 零占位视图行为 | `https://raw.githubusercontent.com/Physical-Intelligence/openpi/main/src/openpi/policies/libero_policy.py` |
| [O7] | 官方 `models/model.py`：训练内部图像预处理/随机增强审计 | `https://raw.githubusercontent.com/Physical-Intelligence/openpi/main/src/openpi/models/model.py` |
| [R1] | InSpire: Vision-Language-Action Models with Intrinsic Spatial Reasoning，arXiv:2505.13888v3：显式方向回答机制参考 | `https://arxiv.org/html/2505.13888v3` |
| [R2] | Spatial Forcing: Implicit Spatial Representation Alignment for Vision-language-action Model，arXiv:2510.12276v1：冻结教师、视觉表征对齐及原结构差异参考 | `https://arxiv.org/html/2510.12276v1` |
| [T1] | FastVGGT 官方代码：教师依赖、特征/合并机制审计入口；不用于直接推断本项目双视图吞吐 | `https://github.com/mystorm16/FastVGGT` |
| [B1] | LIBERO 官方仓库：任务与数据/回放资源入口 | `https://github.com/Lifelong-Robot-Learning/LIBERO` |
| [B2] | LIBERO-Plus 官方仓库：七维扰动、全量规模、安装与资源配置 | `https://github.com/sylvestf/LIBERO-plus` |
| [B3] | LIBERO-Plus `benchmark/__init__.py`：初态加载、来源映射和部分资源单状态路径 | `https://raw.githubusercontent.com/sylvestf/LIBERO-plus/main/libero/libero/benchmark/__init__.py` |
| [H1] | NVIDIA RTX A6000 官方规格：48GB 显存；不代表本实验已测性能 | `https://www.nvidia.com/en-us/products/workstations/rtx-a6000/` |
| [C1] | OpenAI Codex 官方 AGENTS.md 文档：项目指令发现与大小限制 | `https://developers.openai.com/codex/agent-configuration/agents-md` |

所有公式工时均由本计划给定的假设参数计算；在得到服务器实测前，均应保留“估算/待校准”标记。
