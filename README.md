# Vault-of-Xiyuan

曦源项目基础阶段文档：π₀-FAST 的 S、S+A、S+B、S+A+B 四组及七维仿真评测。

## 当前阶段

2026-09-19更新：第九次参照补齐，实际SA服务回归通过（保留旧dtype证据限制）。SB沿原reader；SA-full/SB-full按34/42卡时预算启动，实际初始化、首3更新及前100步预算校准通过；随后现场GPU设备/驱动访问异常，两条run最后观察到SA=288、SB=241且尚无完整checkpoint。F2仍未验收，恢复前不重启。

**F2：IN_PROGRESS／未验收；S-full 覆盖对照已完成 3,000 步、三点诊断和终点 20-clean，终点评测 9/20 成功；实际曝光及460请求CPU复算完成。SA-full/SB-full已按新预算启动并通过前100步校准，但当前因GPU设备/驱动访问异常最后观察到SA=288、SB=241，尚无完整checkpoint、三点诊断或终点评测。** 已完成获批的 S-500 匹配对照：单卡内 3,000 步、step-1000/2000/3000 固定诊断和终点原 20-clean 评测；结果是 0/20 成功，不能据此放行 F2。原SA/SB训练、有效负结果、服务修复及回归保留；G1=PASS，F3/SAB和四组400步/24 GPU小时仍未授权。见 [F2 PLAN](docs/F2_AB模块实现/PLAN.md)、[F2 LOG](docs/F2_AB模块实现/LOG.md)及[既有聚合证据](docs/F2_AB模块实现/review/final-evidence/aggregates.json)。

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
