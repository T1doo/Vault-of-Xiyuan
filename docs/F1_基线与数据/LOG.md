# F1：基线与数据——阶段日志

## 当前进展

最后核对：2026-09-07。**F1 / T01、T02 IN_PROGRESS，G1未通过。** 四个隔离环境、数据/权重/tokenizer/Plus资产准备已完成；61,750个样本的完整FAST审计通过。数据v2为450/50整episode、55,682/6,068动作起点，norm仅训练集；初态关联仍有未知项，不宣称严格初态留出。

**300步小样本pilot已完成并退出0。** 200个train样本池、有效batch16、累计8×物理2，共4,800次样本抽取；前10步平均action loss13.606，末10步1.929。全部300更新的样本顺序独立复核一致，数值有限。实际100→200步全部10个LoRA叶子更新，32个冻结叶子不变；第100步冻结叶子与基础权重一致。完整检查点为run a/checkpoints/100、200、300。

**独立恢复诊断已精确通过**：统一step类型/分片，并固定确定性GPU执行设置后，模型、optimizer、step、样本和接续更新逐参数一致。实际累计入口已运行，但GPU上累计与等效大batch的数值比较仍待完成。

**开发闭环尚未通过。** 第100、200步固定checkpoint在同一预留clean开发单元的首请求均出现动作系数长度错误；第200步生成56个，要求70个，未补零/截断，未向仿真发送动作。训练loss下降不能替代策略闭环成功。最初浮点token承载类型的服务适配错误已单独修复并保留失败记录。

七维各一个真实单例全部执行成功；重复加载同状态精确一致，布局例仅1个118维状态，其余例为50×92库存、实际检查2行。机器人姿态扰动须检查预热后的观测，噪声例实际改变第三人称图像；这不等于最终manifest或全部初态唯一性已验收。专家完整开环仍9/10成功，失败例补充GT状态恢复诊断，不改写失败。

当前没有本项目活跃GPU作业。2,000步全数据S开发训练配置已准备，尚未启动；下一步完成其启动前检查并按固定配置推进，同时做第300步开发检查、GPU累计对应及多checkpoint闭环。负责人代表回放/控制语义与曲线审阅仍待完成。F1不会因300步pilot结束而自动通过G1。

工程根upstream/openpi；训练pilot执行版本与provenance保存在run目录，后续服务/检查点保留修复另记工程Git版本，不追溯冒充旧作业代码。证据位于artifacts/audits/f1-resources-20260907/；阶段日志以下保留历史运行状态及失败，以上为最新恢复入口。

## 执行记录

### 2026-09-07｜负责人批准进入F1环境准备

完整读取 [GPT复审回复](https://chatgpt.com/s/t_6a9e70cd88588191849e2ce5dc143374) 的公开正文（3631字符，完成状态）。负责人要求按意见继续，采用其F1环境准备范围。F0结论保留，README切至F1；真实会话已阅读工程入口、F1 PLAN/LOG、F0交接与主文档相关正文，不以文件hash代替理解。不重复入口测试，不扩大到训练。

### 2026-09-07｜uv固定版校验通过，开始准备Python

官方发行版uv 0.12.10已下载至项目tools/uv，压缩包SHA256与官方sha256文件及GitHub asset digest一致：173d95a0c32d18c896c46ba6fafbf3cf9c14ab74b033f81b76c883ef492a976b。uv --version及python install/sync --help均退出0；原始清单见uv-bootstrap.json。未修改全局PATH；后续使用绝对路径。正安装用户目录Python3.11，结果待核验。

### 2026-09-07｜Python就绪，处理安装中的NFS归属检查

`uv python install 3.11 --install-dir /nfs_share/lijunhui2/tools/python --no-bin` 退出0，实际安装CPython3.11.16。后续锁定该补丁版本，不修改系统Python或全局PATH。

学生环境使用UV_PROJECT_ENVIRONMENT=工作区/envs/policy-train、UV_CACHE_DIR=工作区/cache/uv、UV_LINK_MODE=copy，GIT_LFS_SKIP_SMUDGE=1，按固定openpi `uv sync --frozen --no-dev` 安装。初次及第一次重试分别被uv的LeRobot Git缓存目录及.git别名的NFS归属检查阻止（退出1）；保留policy-sync*.log/.exit。仅在本轮临时git-safe.config列出已知项目缓存/checkout精确路径，未设通配信任、未改用户全局配置。第二次重试已通过LeRobot构建，继续安装；上游pyproject/uv.lock未改，尚不能标环境通过。

安装进展：第二次重试已完成189个包准备（安装日志报告4m29s），开始写入隔离环境；仍待最终退出码和运行验证，不提前标安装通过。PyTorch是固定上游运行/数据依赖，学生模型路线仍为JAX/Flax，不执行PyTorch移植。

### 2026-09-07｜学生环境与实际运行检查结果

| 项目 | 结果 | 证据文件（本轮审计目录） |
|---|---|---|
| uv发布版校验 | PASS：0.12.10，官方SHA256一致 | uv-bootstrap.json |
| Python安装 | PASS：3.11.16，项目tools目录，系统Python未改 | python-install.log/.exit、python-runtime.json |
| 固定锁安装 | PASS：第二次重试退出0；前两次失败保留 | policy-sync*.log/.exit |
| 包一致性 | PASS：202包无冲突 | pip-check.log/.exit |
| 环境/导入路径 | PASS：解释器prefix为policy-train，11项导入通过 | policy-import-check.json/.log/.exit |
| 训练/服务CLI | PASS：两个--help退出0，没有开始训练或服务 | train-help.log/.exit、serve_policy-help.log/.exit |
| 离线锁同步检查 | PASS：Would make no changes | policy-sync-check.log/.exit |
| 单卡GPU合成检查 | PASS：FP32 JIT、自动求导、BF16矩阵乘法；GPU1对应进程内cuda:0 | gpu-smoke-run.json、jax-gpu-check.json/.log/.exit |
| 上游文件完整性 | PASS：pyproject.toml、uv.lock、.python-version与安装前一致 | source-lock-before.json、environment-validation.json |

实际主要版本：JAX/jaxlib0.5.3、Flax0.10.2、Torch2.7.1、torchvision0.22.1、NumPy1.26.4、transformers4.53.2、Orbax0.11.13、ml-dtypes0.4.1、tensorstore0.1.74。openpi/openpi-client以本地固定源码导入，LeRobot来自锁定0cf864870cf29f4738d3ade893e6fd13fbd7cdb5。完整安装列表在policy-freeze.txt，其hash与解释器hash已写environment-validation.json。默认dev与可选RLDS依赖未安装，不能据此声称开发测试工具或RLDS转换环境齐备。

GPU验证是明确的SYNTHETIC环境检查：128×128单位矩阵上的编译、梯度与BF16乘法，与数学结果比较；浮点梯度容差仅用于此合成算式，不替代项目模型容差。记录的约4秒为脚本内计算段时间，不是训练step benchmark、模型吞吐或总启动耗时。未升级驱动；535.274.02在这个固定环境上的该检查通过，复杂模型编译仍待实测。

已执行命令（以下路径属于本机；不需激活全局环境）：

```sh
UV_CACHE_DIR=/nfs_share/lijunhui2/cache/uv /nfs_share/lijunhui2/tools/uv/bin/uv python install 3.11 --install-dir /nfs_share/lijunhui2/tools/python --no-bin

env -u PYTHONPATH -u PYTHONHOME -u VIRTUAL_ENV UV_CACHE_DIR=/nfs_share/lijunhui2/cache/uv UV_PYTHON_INSTALL_DIR=/nfs_share/lijunhui2/tools/python UV_PROJECT_ENVIRONMENT=/nfs_share/lijunhui2/envs/policy-train UV_LINK_MODE=copy GIT_LFS_SKIP_SMUDGE=1 GIT_CONFIG_GLOBAL=/nfs_share/lijunhui2/artifacts/audits/f1-env-20260907/git-safe.config /nfs_share/lijunhui2/tools/uv/bin/uv sync --project /nfs_share/lijunhui2/upstream/openpi --frozen --no-dev --python /nfs_share/lijunhui2/tools/python/cpython-3.11.16-linux-x86_64-gnu/bin/python3.11

UV_CACHE_DIR=/nfs_share/lijunhui2/cache/uv /nfs_share/lijunhui2/tools/uv/bin/uv pip check --python /nfs_share/lijunhui2/envs/policy-train/bin/python
```

首条是实际初装历史命令，后续复现锁定已得到的3.11.16，不用浮动3.11请求升级补丁；已存在环境无需重装。sync使用临时精确safe.directory配置，覆盖本项目LeRobot缓存路径，不能删除此配置后假设NFS归属问题自动消失。源码锁hash：793488b5a55bb87200db90a61fd0af51922b686d94e1da4f4c587ab119b37d74。

CPU导入检查使用本轮check-policy-env.py：调用时仅进程级清除PYTHONPATH/PYTHONHOME/LD_LIBRARY_PATH，设置CUDA_VISIBLE_DEVICES为空、JAX_PLATFORMS=cpu、HF_HUB_OFFLINE=1、TRANSFORMERS_OFFLINE=1、WANDB_MODE=disabled，以及项目缓存路径，未修改用户全局环境。两个CLI --help沿用该隔离调用方式。GPU检查由run-gpu-check.py先查询UUID/空闲显存/计算进程，再选择一张空闲卡，设置JAX_PLATFORMS=cuda和XLA_PYTHON_CLIENT_PREALLOCATE=false，90秒超时；此次退出0。脚本及原始记录均保留在本轮审计目录，不是新增科研训练入口。

安装期间一次du查询遇到正在原子改名的临时文件消失而退出1，不是安装失败；后续未用该瞬时容量作为完成依据。完成依据为安装退出0、独立包/导入/锁检查及实际运行结果。没有隐藏或清除失败记录，也没有为通过检查放宽模型精度标准。

本轮只完成PLAN“本轮执行范围”的三项，四环境总步骤仍未勾选。来源依据为固定openpi源码与 [uv官方安装说明](https://docs.astral.sh/uv/getting-started/installation/)、[JAX官方安装说明](https://docs.jax.dev/en/latest/installation.html)，最终兼容结论以本机检查的具体范围为准。文档将按负责人持续授权提交推送并给出固定版本链接；不公开环境、源码缓存或原始设备信息。

发布前文档检查通过（退出0）：本轮四个文档的链接/围栏、README当前阶段、三项限定范围勾选；上游锁文件不变，其余环境及模型/数据/监督目录未创建。`git diff --check`退出0，远端fetch成功；仅推送Vault文档，安装与验证原件留工作区。

### 2026-09-07｜持续目标：完成F1

负责人明确持续目标完成F1，首批学生环境完成为实质进展；现继续余下环境、官方资源、数据/回放与项目S验证，最终以完整G1证据为准。保留原环境和日志，不重复安装；本批证据放artifacts/audits/f1-resources-20260907。正式训练仍未授权越过G2。

### 2026-09-07｜余下环境与原始数据开始准备

Python3.8.20/3.10.21已装入项目tools目录，sim-clean/sim-plus/teacher虚拟环境已建立但安装尚在进行。选用固定openpi示例核心robosuite1.4.1/MuJoCo3.2.3（与LIBERO requirements中的1.4.0差异显式记录），clean/plus保持一致；实际运行将导入独立LIBERO/Plus checkout，不使用未初始化子模块。环境兼容候选将示例Numba0.53.1/llvmlite0.36.0更新为0.58.1/0.41.1以配合NumPy1.22.4；其余库按源要求解析，真实导入/回放后再判可用，不称官方环境逐项复现。独立解析结果均退出0，保存在sim-clean/sim-plus/teacher.resolved.txt。

首个官方Spatial HDF5已下载并校验长度及LFS SHA256（508779600字节）。真实打开后确认50条完整演示；首条98步，actions7维，ee_pos3+ee_ori3+gripper_states2可组成基线8维本体状态，robot_states另为9维，不能混用或截断；两路RGB均128×128，states92维，demo含init_state/model_file。结构可用不代表时序或控制链路已验证。原始证据first-hdf5-inventory.json。现继续同revision全部10任务下载，复用第一项并逐个验证；尚未划分train/dev/final。

当前已启动并持有工具句柄：sim-clean安装43503、sim-plus安装与teacher安装句柄见本次工具记录，全部数据下载50568（状态/进程在spatial-download.json）。中断后先查.exit、实际进程或同一工具句柄，不据超时重复启动。没有GPU科研作业、没有训练checkpoint；本批日志与锁文件在artifacts/audits/f1-resources-20260907。

### 2026-09-07｜实际数据时序审计与负责人确认

全部10任务、500条演示已下载并校验，共62250行原始动作。逐演示比较真实joint_states与states中的机器人关节位置，在所有61750个可比较位置上，obs[i].joint_states与states[i+1]对应位置逐元素精确相同；同索引误差显著。结合固定源码create_dataset.py在env.step后采集obs的路径，确认原始数据不能直接同索引配对。原始证据full-hdf5-timing-audit.json、joint-state-lag-diagnostic.json；不是synthetic数据。

负责人明确回复“同意”，采用统一修正：obs[i]、states[i+1]、actions[i+1:]，i从0到L-2。每条轨迹缺少动作前原始图像的首个动作起点不训练，共排除500个（约0.8%），保留61750个，其余不得因A/B监督差异删样本。不重新渲染训练图像，原HDF5保持不变。批准映射写入工作区protocols/data-timing-v1.json，四组共用；尚未锁定正式protocol.lock或生成划分/norm。

clean/plus基础安装及独立源码包安装已完成。首次导入暴露上游setup.py未发现外层namespace导致的现代editable空映射：只在各自环境增加精确source-root .pth，分别绑定独立LIBERO/Plus，未改全局PYTHONPATH或源码。clean随后导入通过。plus缺ImageMagick，已将Ubuntu官方包按APT SHA256校验后解压到项目tools/imagemagick，并补齐liblqr/libfftw3；仅在plus进程设置原生库路径，最终plus导入通过，无sudo/系统安装。

teacher安装与pip check通过；初次导入受到继承的/share/apps/cuda/12.2的不可读libnvJitLink影响，进程级清除LD_LIBRARY_PATH后Torch2.3.1+cu121、torchvision0.18.1+cu121和FastVGGT/pycolmap/pyceres/open3d导入通过。原失败日志保留。基础模型与Plus资产仍在下载校验；FAST tokenizer文件已校验，尚未执行其远程代码。

真实专家前三步回放完成，记录了原始XML到本机资产的路径映射（不改相机/几何参数）、图像朝向及位姿误差；raw朝向明显优于flip/rotate180。前三步未完成任务不算失败，现完整回放首条演示检验真实成功谓词。后续训练图像与在线输入采用同一经审计的确定性约定，不直接复制上游RLDS专用180度旋转。

准备真实权重合成集成检查：run_id=f1-synthetic-model-20260907-a，GPU0启动前确认空闲，上限600秒/2次更新、物理与有效batch均1，仅诊断不计正式或学习曲线。配置/脚本hash及本地实现commit见real-model-check-registration.json，输出runs/diagnostics/f1-synthetic-model-20260907-a；验证冻结叶子不变、LoRA实际更新及不同RNG的模型入口预处理。依赖已满足，尚未宣称通过。

### 2026-09-07｜资源、真实模型与数据验证汇总

本节更新前文安装中/下载中/未划分等历史状态，不删除失败记录。

| 已执行项 | 结果及证据（本轮审计目录） |
|---|---|
| Spatial下载 | 10文件/500演示/62,250原始动作行；源revision及每文件LFS SHA256校验，spatial-download.json COMPLETE |
| 基础模型 | 官方35对象共10,850,405,453字节，base-download.json COMPLETE；真实参数32叶子、2,923,335,408参数 |
| FAST/PaliGemma | 本地文件校验完成，fast-download.json、paligemma-tokenizer.json；FAST处理器代码先审计后离线加载 |
| 三个新增环境 | sim-clean125包、sim-plus131包、teacher127包依赖检查及必要导入通过；sim各自LIBERO_CONFIG_PATH隔离 |
| Plus资产 | 固定revision ZIP校验，448,799文件解压及CRC检查完成，plus-extraction.json |
| 专家回放 | expert-replay-run.json COMPLETE，十条视频及JSON保留；9/10成功 |
| 七维资源 | plus-inventory.json：实际API枚举2,402配置，0加载错误，385个单行配置；不是2,402个独立评测单元 |
| FAST测试 | tokenizer-tests-retry.log/.exit：9项PASS，退出0 |
| 数据测试 | data-tests.log/.exit：4项synthetic PASS，退出0 |
| 实际模型 | real-model-check-b.log/.exit及run result.json：两步synthetic PASS，退出0 |
| 真实动作回环 | real-data-roundtrip.json：100个样本通过，尚未做全量长度审计 |
| norm独立复核 | norm-provenance-check.json/.log/.exit：train-only mean/std/q01/q99与独立重算完全一致，退出0 |

Plus资产ZIP保存在NFS `data/raw/libero-plus-assets/assets.zip`；为避免约45万小文件的NFS开销，解压缓存位于 `/tmp/lijunhui2-libero-plus-assets-96764a4bfbda`，Plus assets链接指向它。该缓存可丢失，恢复先验完成标记/路径，缺失从保留ZIP重建；不能把临时盘当唯一持久证据。

失败回放为table-center任务demo0（103帧），最终末端位置与记录差约0.0159米；视频显示放置靠近盘缘。原因仍待核实，不能断言只是模拟器误差，也不能换成功演示掩盖失败。原数据生成器可强制写末帧reward/done，验收采用实际仿真成功谓词。当前原始RGB与在线视图保留一致朝向，不复制RLDS专用180度旋转。

实际模型首次诊断a因未先batch的image mask构造失败，修复为先batch字典再构建Observation；重试前发现GPU0已被其他用户占用，未触碰其进程，改用空闲GPU1运行b。b实际更新两步，loss 4.64285755→3.75446439，所有10个LoRA叶子变化、冻结参数hash保持一致，模型预处理仅调用一次且train=False，跨RNG输入图像精确一致。首步编译约33.55秒、第二步约0.31秒均仅此batch1合成诊断，不是正式吞吐或真实数据可学习证据。

FAST实现保留合法编码并直接从token ID恢复动作段。发现上游先decode为文本再strip/encode会在一个可复现fixture中丢失首FAST token，70个系数变68个并触发静默零动作；项目S改为明确边界和长度校验。IDCT遗漏ortho归一化及EOS测试误用bos ID的问题已纠正，失败记录保留；最终与直接上游FAST处理器比较，不以已损坏的外层文本回环作正确性标准。修正属于四组共用S，不计A/B方法贡献。

数据v1在任何训练前补强初态来源审计并生成v2，v1标SUPERSEDED_BEFORE_ANY_TRAINING；train/val/norm文件hash完全一致，没有看模型效果重划分。500条演示init与官方初态行无精确匹配，这不证明独立，标UNKNOWN_NOT_INDEPENDENT。预留clean官方行0—4为开发候选、5—49为最终候选，但正式manifest未冻结，Plus跨条件血缘仍待核查。

已验证的复核命令（CPU、无训练；工作目录为工程根）：

```sh
env -u PYTHONPATH -u PYTHONHOME -u LD_LIBRARY_PATH CUDA_VISIBLE_DEVICES= JAX_PLATFORMS=cpu /nfs_share/lijunhui2/envs/policy-train/bin/python -m unittest discover -s /nfs_share/lijunhui2/upstream/openpi/tests/xiyuan -p test_data.py
env -u PYTHONPATH -u PYTHONHOME -u LD_LIBRARY_PATH CUDA_VISIBLE_DEVICES= JAX_PLATFORMS=cpu /nfs_share/lijunhui2/envs/policy-train/bin/python /nfs_share/lijunhui2/artifacts/audits/f1-resources-20260907/check-norm-provenance.py
```

训练入口、累计/完整恢复、真实数据短训及七维单例仍未验收，不提供假设存在的训练启动命令。下一步从data-v2和上述实现commit继续，先查作业再启动。文档提交仅公开计划和结果摘要，数据、权重、视频、环境及私人规则留工作区。

### 2026-09-07｜全量FAST长度与完整动作解码审计

新增并实际运行工程CLI `scripts/xiyuan/audit_token_lengths.py`，实现commit `d29c2a44f3a118895cd02d114334b3647b9838a2`。只使用已批准data-v2的状态、动作、当前指令及train-only norm；不读取监督或使用GPU。逐样本经过实际delta/normalize与严格FAST编码、完整解码，覆盖训练55,682和验证6,068个样本，退出0。

训练长度范围60—91，验证61—92；两者P50/P95/P99均72/83/86，最长公共prefix58，无样本超过128。全部61,750个动作解码均为有限10×7数组。由此128可用于当前S开发训练；不据此认定加入A方向后也不会溢出，也不代表编解码无量化误差。证据 `full-token-audit.json/.log`，记录manifest/norm hash及最长样本ID；耗时约105.94秒为本CPU审计耗时。

已执行CLI先通过--help，实际命令：

```sh
env -u PYTHONPATH -u PYTHONHOME -u LD_LIBRARY_PATH CUDA_VISIBLE_DEVICES= JAX_PLATFORMS=cpu HF_HOME=/nfs_share/lijunhui2/cache/huggingface HF_MODULES_CACHE=/nfs_share/lijunhui2/cache/huggingface/modules HF_HUB_OFFLINE=1 OPENBLAS_NUM_THREADS=1 OMP_NUM_THREADS=1 /nfs_share/lijunhui2/envs/policy-train/bin/python /nfs_share/lijunhui2/upstream/openpi/scripts/xiyuan/audit_token_lengths.py --data-dir /nfs_share/lijunhui2/protocols/data-v2 --raw-root /nfs_share/lijunhui2/data/raw/libero/libero_spatial --tokenizer-root /nfs_share/lijunhui2/weights/tokenizers --output /nfs_share/lijunhui2/artifacts/audits/f1-resources-20260907/full-token-audit.json
```

输出已存在时脚本拒绝覆盖；复核使用新的输出名。原始norm和manifest保持不变。

### 2026-09-07｜独立进程真实恢复诊断进行中

`check-real-resume.py --help`退出0后，登记 `real-resume-registration.json`，运行save模式：真实基础权重、data-v2实际样本、物理/有效batch1，固定初始化RNG123、模型RNG456、sampler seed0；保存第2次有效更新，并拟用第3次更新建立连续运行参考。独立restore进程将校验模型/optimizer/step hash，再比较同样本第3次更新的参数和指标。诊断配置为constant lr3e-5/warmup0，仅用于恢复比较，不是正式训练配置。

当前save实际完成两步，action loss分别16.190626和11.707721，数据样本不同，不以此判断学习趋势。Orbax完整保存仍在 `runs/diagnostics/f1-real-resume-20260907-a/checkpoints/2.orbax-checkpoint-tmp-0`；尚未产生expected.json，也没有PASS结果。禁止把临时目录当完整checkpoint，禁止在session91909未结束时重复启动。保存进程仍真实存活；检查点包括norm及模型/optimizer，外侧provenance绑定数据/norm/基座来源/代码hash。最终完整恢复验证仍待执行。

启动前GPU1空闲；启动后核实另一项目渲染任务也出现在GPU1，PID371408不属本次任务，未终止或修改。当前自己的PID372761已进入保存收尾，最多600秒；后续restore改在重新确认空闲GPU进行。本次不用于吞吐benchmark，保留资源竞争事实。

实际启动（stdout/stderr保存为本轮real-resume-save.log）：

```sh
env -u PYTHONPATH -u PYTHONHOME -u LD_LIBRARY_PATH CUDA_VISIBLE_DEVICES=GPU-414c52ba-72c6-fc45-95d6-1e9750bbc21b JAX_PLATFORMS=cuda XLA_PYTHON_CLIENT_PREALLOCATE=false HF_HOME=/nfs_share/lijunhui2/cache/huggingface HF_MODULES_CACHE=/nfs_share/lijunhui2/cache/huggingface/modules HF_HUB_OFFLINE=1 OMP_NUM_THREADS=2 OPENBLAS_NUM_THREADS=2 timeout 600 /nfs_share/lijunhui2/envs/policy-train/bin/python /nfs_share/lijunhui2/artifacts/audits/f1-resources-20260907/check-real-resume.py --mode save --run-dir /nfs_share/lijunhui2/runs/diagnostics/f1-real-resume-20260907-a
```

这条命令是已启动历史记录，不可原样重复（run目录存在会拒绝）。恢复当前工作先poll工具session91909并读取save-status/log；只有checkpoint提交完成且expected.json存在后才允许进入restore。restore仅--help注册过，尚未验证执行成功。G1继续IN_PROGRESS，不存在无人值守后续训练队列。

保存更新：session91909已退出0，save-status为COMPLETE；正式提交的诊断checkpoint路径为 `runs/diagnostics/f1-real-resume-20260907-a/checkpoints/2`（约4.6G），snapshot.json绑定完整模型/optimizer/step，expected.json含连续第3步参数hash/样本/指标。原临时目录状态已结束。重新查询GPU2空闲后启动独立restore，工具session79933/PID380944，GPU UUID见registration更新；600秒限时保持不变。restore使用同一已注册CLI，将mode改为restore，CUDA_VISIBLE_DEVICES改为GPU2的UUID，日志real-resume-restore.log。启动本身不等于恢复验证通过。

### 2026-09-07｜恢复诊断结果：状态恢复通过，接续更新失败

独立restore session79933已终止，退出1；失败处为 `resumed continuation differs exactly`。在这之前，第2步的全部模型参数、optimizer叶子及step与保存前snapshot逐项shape/dtype/SHA256精确一致，继续采样的sample_id也与连续运行相同。连续第3步loss=14.61308575、grad_norm=26.53478622；独立恢复第3步loss=14.64101601、grad_norm=26.78891373，最终参数hash不同。不能把“成功读取checkpoint”当作完整恢复验收，也不能根据这次跨GPU比较放宽容差。

完整checkpoint和expected/snapshot/provenance均保留，失败日志real-resume-restore.log保留；没有训练/教师/评测活跃作业。该差异可能涉及跨进程编译、模型静态状态或数值执行路径，现有证据尚未定位原因。下一步在相同GPU上独立复验并保存输入batch/hash、图结构及重复前向/更新误差，区分输入、状态和编译问题；未经证实不归咎GPU。原诊断脚本hash已绑定provenance，新增诊断用新版本/新输出，不覆盖历史证据。恢复gate仍未通过，F1与真实S学习训练均继续待办。

### 2026-09-07｜同卡恢复复验与梯度累计入口

同一原GPU1、同脚本、同checkpoint复验退出1：第3步loss=14.613085746765137，与连续参考精确相同；grad_norm=26.534759521484375，参考26.534786224365234，参数hash仍不同。证据real-resume-same-gpu.log；原跨GPU失败日志不覆盖。此证据缩小排查范围，不自动放宽容差或判恢复通过。

新增v2诊断记录训练输入叶子的值hash/shape/dtype/weak_type/sharding、模型静态图和StableHLO。第一次误在已忙GPU1启动后，立即精确匹配并终止自己的PID394933（退出143），未触碰其他项目进程；该run b保留。随后启动保护在同一次调用中两次检查选定GPU的显存和利用率，忙卡拒绝，再在GPU2登记run c。它是本次有限诊断入口，不是通用调度器，仍存在检查后外部作业启动的竞态，须运行中检查。

run c完成两步及完整checkpoint保存，但新增step.lower检查在JAX ArgInfo重建TrainState时触发jaxtyping类型错误，save退出1；不是模型前向或checkpoint写入失败。保存前inputs/graph和完整snapshot已留存。仅在lower检查局部使用上游array_typing.disable_typechecking，实际训练的类型校验保持开启；新probe复用已有checkpoint，不重新生成训练参考或重复下载。当前probe对同一恢复状态、同一输入重复更新3次，测量重复误差，不提前指定为恢复PASS。

梯度累计已实现并本地提交：src/xiyuan/training.py及tests/xiyuan/test_training.py，commit ef844cf（完整hash以工程Git为准）。输入限制为等大小微批次，先逐样本均值、累积后平均梯度，再统一裁剪/optimizer更新一次；有效步只加1。CPU synthetic测试比较3×2与batch6、独立解析梯度/一次裁剪及冻结参数不变，还确认错误的逐微批次裁剪会产生不同结果。测试退出0，证据accumulation-cpu.log；语法检查退出0。真实模型GPU累计、大batch对应和吞吐尚未运行，不称已支持完整生产训练。

已验证命令：

```sh
env -u PYTHONPATH -u PYTHONHOME -u LD_LIBRARY_PATH CUDA_VISIBLE_DEVICES= JAX_PLATFORMS=cpu /nfs_share/lijunhui2/envs/policy-train/bin/python -m unittest discover -s /nfs_share/lijunhui2/upstream/openpi/tests/xiyuan -p test_training.py
```

当前恢复点：run c完整checkpoints/2、snapshot.json、save-inputs.json/save-graph.txt；重复误差probe工具session49854、PID405355、GPU2、上限600秒，登记real-repeat-c-restore-registration.json，日志real-repeat-c-restore.log。先poll同一session和登记终态，不以状态文件尚未更新就重启。G1仍未通过，正式/真实开发学习长训尚未开始。

### 2026-09-07｜重复误差与实际输入差异已实测

probe session49854退出0；同一恢复状态、同一样本重复3次更新，loss均14.6259765625、grad_norm均26.91887664794922，全部10个LoRA参数叶子相对首次结果的最大绝对差均为0。证据run c/repeat-probe-result.json、repeat-probe-lora.npz，结论只适用于此已编译程序，不是恢复gate通过。

save-inputs与restore-inputs共78个叶子：所有值hash、shape、dtype及分片均一致，唯一差异是state.step的weak_type从True变False。原始比对resume-input-diff.json已留存。此标记可能改变JAX编译，但尚未证明是全部差异的原因。重复误差为0，不能以“自然随机波动”解释或直接容忍此前差异。

新增canonicalize_training_state明确将step表示为JAX int32非weak scalar，供初始化和恢复后共同调用，不改变数值或采样游标。CPU合成测试验证fresh/host-roundtrip后相同aval和值；与累计回归一起2项通过，退出0，证据accumulation-counter-cpu.log。函数尚未接入真实训练诊断，因此不能声称问题已修复。下一步将它接入新版本的实际诊断，固定真实JIT输入/输出分片，再做同卡独立恢复；若仍不同，比较编译与确定性设置，继续保留原证据与精度标准。

本轮所有进程均已终止。最近完整checkpoint为run c/checkpoints/2，另保留run a/checkpoints/2及其连续第3步expected；没有可用于学习结论的训练checkpoint。F1/G1继续IN_PROGRESS，真实累计GPU对应、小样本与1k—3k训练、开发闭环、失败专家回放及七维生效检查仍未完成。

### 2026-09-07｜显式类型/分片复验、训练入口与失败回放诊断

v3/run d采用统一int32非weak步数及显式JIT输入/输出分片。save退出0，restore退出1。恢复前后78个输入/状态叶子的全部值hash、shape、dtype、weak_type和sharding一致，StableHLO文本也完全一致（268,086字符）；但第三步连续loss14.65947247、恢复loss14.58457088，参数hash仍不同。证据run d/save-inputs、restore-inputs、save/restore-stablehlo及real-resume-v3-d-*.log。类型差异已消除，但不足以解决独立执行差异，未放宽容差。

本机已安装jaxlib二进制包含xla_gpu_deterministic_ops与xla_gpu_autotune_level；新v4/run e仅在项目进程设置 `XLA_FLAGS=--xla_gpu_deterministic_ops=true --xla_gpu_autotune_level=0`，将运行参数写入provenance，重新做完整保存/恢复。没有改系统驱动、全局环境或研究损失。此时是诊断候选，未因开启参数就称确定性通过。资源/上限仍由launch-resume-v4.py在GPU2两次空闲检查后登记，单进程600秒，最多3次save更新/1次restore更新。

F1训练入口已实现：工程scripts/xiyuan/train_s.py，本地commit9f50f7741e02bdf4f5ba7e699e1c9552f09783f0。支持有效batch16的等大小微批次累计、step对应固定sample_id计划、完整checkpoint/严格恢复、配置/数据/norm/tokenizer/实现及上游关键源码hash、分attempt追加日志和限时停止；拒绝formal及超过3000步的作业。--help、语法和拒绝20k作业检查退出0。真实模型CPU eval_shape检查8×2累计路径通过，42个参数叶子保留，证据accum-model-shapes.json/.log，实际更新0、未加载权重；不能替代GPU数值验证。

工作区protocols/f1-s-overfit-planned.json仅PLANNED：首200个train样本、300更新、物理2/有效16、开发warmup10、max_token_len128、最大3600秒；与正式配置分离，尚未启动。恢复和真实累计验收未满足前，不宣称真实训练入口可用或已有学习曲线。下一步先2步入口检查并验证完整恢复，再推进小样本与1k—3k训练。

失败专家演示追加CPU纯状态诊断（关闭图像与renderer，无GPU）：table-center/demo0完整开环仍失败；恢复记录末状态时成功谓词已为True，最后动作后仍True；逐步恢复states[i]再执行actions[i]最终也True。证据failed-expert-state-diagnostic.json/.log，退出0。这支持开环累积偏离是待解释现象，而非末记录状态在本环境必然无法满足成功；不证明具体是控制器、接触物理或环境版本的哪一项导致。9/10开环结果不改写，GT状态恢复仅离线诊断，不进入正式策略请求。

实际CPU诊断命令：

```sh
env -u PYTHONPATH -u PYTHONHOME -u LD_LIBRARY_PATH CUDA_VISIBLE_DEVICES= MUJOCO_GL=egl PYNPUT_BACKEND=dummy LIBERO_CONFIG_PATH=/nfs_share/lijunhui2/local/libero-clean MPLCONFIGDIR=/nfs_share/lijunhui2/cache/matplotlib-clean NUMBA_CACHE_DIR=/nfs_share/lijunhui2/cache/numba-clean OMP_NUM_THREADS=2 OPENBLAS_NUM_THREADS=2 timeout 180 /nfs_share/lijunhui2/envs/sim-clean/bin/python /nfs_share/lijunhui2/artifacts/audits/f1-resources-20260907/check-failed-expert-state.py
```

当前v4 save工具session27687仍在执行；此前v3和CPU诊断均终止。恢复先查real-resume-v4-e-save-registration.json及原session；只有save COMPLETE和expected.json存在才启动restore。F1/G1未通过，未启动后台学习训练队列。

v4保存更新：session27687退出0，完整run e/checkpoints/2及expected.json已生成；连续第3步loss14.65478515625、grad_norm26.65860939025879。当前独立恢复session65924，GPU2，登记real-resume-v4-e-restore-registration.json；恢复结果仍待核实，不能提前判PASS。

### 2026-09-07｜独立恢复精确通过，进入真实累计入口检查

v4/run e的save与restore均退出0。恢复模型/optimizer/step全部hash一致；接续第3步同sample_id，loss14.65478515625、grad_norm26.65860939025879以及全部更新参数hash精确一致。证据run e/result.json明确PASS，两个registration为COMPLETE，原失败记录全部保留。采用统一step表示+显式分片+确定性GPU操作/关闭运行时自动调优的组合，不从组合实验声称单个参数独自解决全部原因；没有放宽任何容差。该结果证明这条真实模型恢复诊断，不自动覆盖新累计入口、其他硬件或F1可学习性。

训练入口已要求配置中的xla_flags与启动环境精确匹配，缺失则拒绝启动，代码commit357c33d（完整hash以工程Git为准）。主文档同步当前共享S运行要求，四组不分别选GPU编译算法设置。真实成本必须按这个已验证设置测量，不复用旧默认编译下的短时步速。

开始F1实际入口限定2次有效更新：launch-s-entry.py经GPU2空闲保护启动，工具session26126；物理batch2×累计8=有效16，配置仍为300步小样本pilot的固定总时程，但本段--stop-after 2，不运行余下298步。上限900秒，日志f1-s-entry-entry-two-updates.log，登记同名前缀registration.json；输出runs/pilot/f1-s-overfit-20260907-a。恢复按原总时程保持schedule，不因分段重设LR或采样。warmup10只属于小样本pilot；当前没有长训或正式作业。

真实入口首次session26126在run创建前因Fast tokenizer的.cache目录被文件hash循环读取而退出1，没有执行更新。修复只遍历实际文件，独立确认5个tokenizer文件可读、run尚不存在；本地代码commit daa1a1d，原失败日志保留。限定2更新重试已启动，session24145/PID446501，GPU2，输出同计划run a，新日志f1-s-entry-entry-two-updates-retry.log及新registration。当前仍在模型初始化/首次编译阶段，未宣称累计GPU通过。

### 2026-09-07｜实际累计入口通过两步，启动300步小样本pilot

session24145退出0，COMPLETE_REQUESTED_SEGMENT：两次有效更新均完成，每步物理2×累计8=有效16，action loss分别14.98901367/13.66802406，梯度范数15.68289757/14.32157516，数值有限。第2步完整checkpoint位于runs/pilot/f1-s-overfit-20260907-a/checkpoints/2，status记录checkpoint_update=2。首步含编译/数据19.66秒、第二步含数据6.53秒；未达到稳定300更新profile要求，不作正式吞吐。两步不同样本loss不能作为已学会任务的证据，GPU累计对等大batch的数值检查仍待补齐。

按持续完成F1授权，launch-s-overfit.py从这个完整第2步checkpoint恢复至预先固定的300步，配置、200样本池、seed、总scheduler时程和有效batch不变；不是重新初始化或重置样本cursor。启动保护已确认GPU2空闲并登记：session89825，最多额外298更新，外层限时3600秒；内部每100步/段末保存，完整checkpoint后才更新状态。日志f1-s-entry-overfit-resume-to-300.log，登记同名前缀registration.json。此前planned配置现在已被实际入口使用，权威resolved-config/provenance/schedule.sha256位于run目录；不修改运行中的配置与代码。

此小样本pilot仅F1诊断，不进formal主表，不越G2。实际后台只有已启动的这一有限作业，未启动1k—3k全数据训练或正式四组队列。恢复先poll session89825，再核对status、metrics各attempt和最近完整checkpoint；不要因对话结束重复启动。完整目标仍包含小样本曲线、全数据短训/多checkpoint开发闭环、GPU累计对应与七维生效/初态审计、代表回放人工检查；G1尚未通过。

### 2026-09-07｜七维真实单例、开发服务与300步pilot收口

七维单例session42827退出0，7个子进程均退出0；登记plus-singletons-run.json为COMPLETE，GPU3已释放。每例通过实际OffScreenRenderEnv创建、官方state shape检查、重复reset和10步预热；两个视图128×128×3 uint8。背景/机器人/视角/语言/噪声/布局/光照分别选官方index0/258/608/984/1374/1725/2110，详细JSON/PNG/日志在plus-singleton-*；总体及组件比较为plus-singletons-summary/effects.json。不是策略rollout或正式评测manifest。

机器人例native reset相对参考最大关节差0.06668，加载同一官方状态后差为0，10步预热后差0.04272，说明不能只看set_init_state瞬间就判扰动无效，也不能向策略发送预热前缓存观测。噪声例noise=3，wrapper与同一步未加噪图像的第三人称平均绝对像素差8.187，腕部为0。布局库存仅1×118，实际不能靠换seed凑20单元。语言例的完整改写真实可用；其他类别API语言包含文件配置后缀，需要在评测清单构造时排除机械后缀，而非让策略按task_id恢复答案。

已生成2402配置的公开指令映射草案protocols/plus-public-instructions-draft.json：390个语言变体逐字保留官方BDDL/API文本，其余2012个使用对应官方clean指令。仅在离线清单中解析已知配置来源，策略接收literal instruction、不接收task_id。该文件仍DRAFT，不是最终manifest或在线解析器；CPU审计退出0，证据plus-instruction-audit.log。

本地S服务scripts/xiyuan/serve_s.py只绑定127.0.0.1、只接收四个公开字段，严格加载完整checkpoint及其中norm，逐请求重新生成，无跨episode动作/KV状态；B/教师未进入服务。首次20步smoke在首请求暴露上游sample_actions用默认float32零数组承载整数ID，而上游ExtractFASTActions无条件cast int32。项目服务现在只在有限、整数值、范围/shape合法时无损转换，不允许小数/NaN/溢出静默cast；2项synthetic回归退出0，serving-token-tests.log。该修复不改训练参数或核心tokenizer，不影响正在运行pilot的provenance。

第100步修复后的smoke和第200步smoke均在第一请求返回invalid_coefficient_length，实际执行0个策略动作，success=False。前者原服务适配错误与后者真实生成长度失败分开记录；第200步保留生成token IDs，离线解码得到13个FAST token、56个系数，要求10×7=70，Action标记与EOS存在，不是256预算截断。证据s-loop-100-result、s-loop-100-retry-result、s-loop-200-result及s-loop-200-coefficient-diagnostic.json。未截尾/补零、未更改成功判据。各服务/renderer均由有时限launcher终止，当前已无遗留服务。

pilot续段session89825已退出0，实际完成300更新；run status=COMPLETE_REQUESTED_SEGMENT、checkpoint_update=300。完整逐步/采样汇总pilot-overfit-summary.json，全部1—300连续无重复更新，每步16个sample_id与独立PCG64重建一致。第100步32个冻结叶子与实际基础指纹完全一致（trained-freeze-100.json），100→200全部10个LoRA叶子变化、32个冻结叶子不变（pilot-updates-100-200.json），CPU检查均退出0。首10/末10平均loss13.605976/1.928849，仅训练集小池结果，不代表控制成功或完整数据泛化。

上游保留策略在提交100步时自动回收了早期非周期的2步入口检查点；当前100/200/300均完整保留，2步历史日志仍在但不能再作为现存恢复点。后续训练入口已改为保留全部检查点，CPU测试保存2/100/101并恢复最早2成功，退出0，checkpoint-retention.log；不覆盖或删除现存实验产物。本轮训练完成后才改入口，不改变旧pilot执行代码；后续作业用新版本。

全数据2,000步S配置protocols/f1-s-full-2000-planned.json已准备但未启动：同一基础权重重新初始化、全部train样本、有效16/物理2、warmup1000、每500步保存、单段18000秒上限。不是从小池微调模型继续训练后冒充原始初始化基线，也不是正式20k四组训练。下一步仍需GPU累计对应、真实入口确定性细查、多checkpoint开发闭环、初态/指令清单收口和人工审阅。当前真实完成范围仅以上证据，G1未通过。
