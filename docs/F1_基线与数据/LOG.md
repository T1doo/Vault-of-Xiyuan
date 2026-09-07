# F1：基线与数据——阶段日志

## 当前进展

最后核对：2026-09-07 16:26:21 +08:00。**F1 / T01 IN_PROGRESS；本轮学生环境准备及最小运行检查完成，G1未通过。** uv0.12.10、CPython3.11.16与policy-train已安装；依赖一致性、必要导入、训练/服务CLI help、单卡JAX合成编译/求导/BF16检查均通过。

工程根：/nfs_share/lijunhui2/upstream/openpi，来源commit 215abfb217dbac7d5f1273282331b9b1866c0479；未修改其pyproject/uv.lock或模型代码。入口文档commit 6c1cacc3d1a8b3981e5676c467603cdddabe8e18；最新文档以本文件所在提交为准。环境：/nfs_share/lijunhui2/envs/policy-train；证据：artifacts/audits/f1-env-20260907/（仓库外）。

安装与验证进程均已结束；无训练、仿真服务、科研后台队列或checkpoint。短时验证只用当时空闲GPU1，禁用JAX预分配；验证后进程快照已记录。没有下载模型权重、训练HDF5或教师资产，没有转换数据、回放或启动训练。

剩余：teacher/sim-clean/sim-plus环境和独立LIBERO配置尚未建立；实际模型加载、数据/动作链路、回放、冻结/更新及G1所有模型验收未执行。一次小矩阵JIT通过不能保证所有模型算子、显存或吞吐满足要求。

下一步建议：先依据F0中独立LIBERO与子模块/robosuite差异，选清实际仿真来源和版本，再准备clean/plus隔离环境；其他F1工作按PLAN依赖推进，不因本轮环境通过直接长训。官方数据获取后先检查一条HDF5演示再安全适配转换。本轮在学生环境验证结果处收口，供负责人/GPT审阅。

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
