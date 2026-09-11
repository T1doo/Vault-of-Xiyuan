# Vault-of-Xiyuan

曦源项目基础阶段文档：π₀-FAST 的 S、S+A、S+B、S+A+B 四组及七维仿真评测。

## 当前阶段

**F2：AB模块实现 — 固定300题图文QC包、A/B接口诊断、共享bounded入口、新版 GPU 有限回归、真实数据合同检查和生成侧恢复组件已完成限定验证，人工QC、获批监督、连续小训练和全量缓存仍待完成。** G1=PASS，F1已完成。打开 [PLAN.md](docs/F2_AB模块实现/PLAN.md) 审阅实施顺序、阶段边界和当前证据，打开 [LOG.md](docs/F2_AB模块实现/LOG.md) 查看结果与恢复位置。当前批次不做全量缓存或连续小训练，不进入F3或正式长训练。

## 完整方案

- [实验计划](docs/EXPERIMENT_PLAN.md)
- [工程执行手册](docs/CODEX_EXECUTION_GUIDE.md)

## 阶段导航

| 阶段 | 计划 | 日志 |
|---|---|---|
| F0：现状盘点 | [PLAN](docs/F0_现状盘点/PLAN.md) | [LOG](docs/F0_现状盘点/LOG.md) |
| F1：基线与数据 | [PLAN](docs/F1_基线与数据/PLAN.md) | [LOG](docs/F1_基线与数据/LOG.md) |
| F2：AB模块实现 | [PLAN](docs/F2_AB模块实现/PLAN.md) | [LOG](docs/F2_AB模块实现/LOG.md) |
| F3：集成与协议冻结 | [PLAN](docs/F3_集成与协议冻结/PLAN.md) | [LOG](docs/F3_集成与协议冻结/LOG.md) |
| F4：首轮四组实验 | [PLAN](docs/F4_首轮四组实验/PLAN.md) | [LOG](docs/F4_首轮四组实验/LOG.md) |
| F5：重复训练与评测 | [PLAN](docs/F5_重复训练与评测/PLAN.md) | [LOG](docs/F5_重复训练与评测/LOG.md) |
| F6：分析与交付 | [PLAN](docs/F6_分析与交付/PLAN.md) | [LOG](docs/F6_分析与交付/LOG.md) |

服务器完整规则在工作区 AGENTS.md，公开仓库仅保留 [短读取入口](AGENTS.md)。本仓库不存原始数据、权重或完整运行输出；阶段日志记录真实产物位置。原始文档来源：[T1doo/Xiyuan](https://github.com/T1doo/Xiyuan/tree/4b59def0d3d918719beaa37990038532311ce8fe/基础阶段/xiyuan_foundation_v3)。
