# 已验证命令

2026-09-06，F0/T00 文档与服务器审阅。以下均为实际执行过的只读命令；没有可运行的项目训练 CLI，没有模型测试或 benchmark。

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
