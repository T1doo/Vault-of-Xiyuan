# Vault-of-Xiyuan

曦源项目基础阶段文档：π₀-FAST 的 S、S+A、S+B、S+A+B 四组及七维仿真评测。

## 当前阶段

**当前为 F3 计划待审（TODO / PLANNED），尚未执行。** F2 的工程实现与小样本验证已获最终认可，稳定控制目标未达成；有效结果为 SA full-A 原结果与 SB 修复服务v1结果，两者均0/20成功。G1=PASS，G2未通过。下一步只审阅 [F3 PLAN](docs/F3_集成与协议冻结/PLAN.md)，计划状态见 [F3 LOG](docs/F3_集成与协议冻结/LOG.md)；F2依据见 [F2 LOG](docs/F2_AB模块实现/LOG.md)及[聚合结果](docs/F2_AB模块实现/review/final-evidence/aggregates.json)。本批不启动模型更新、SAB或正式队列。

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
