# F2：AB模块实现——阶段日志

## 当前进展

最后核对：2026-09-11。**F2已完成候选规则统计、300题图文QC包、A/B独立接口诊断、公共S回归和SA/SB诊断恢复；人工QC、正式公共训练入口集成和连续小训练仍待完成；G1=PASS。** 本轮依据后续技术审阅，未重复原S KV排查，也未重新提取1,000观测；只对已有A试标和B分片做派生统计/读取恢复测试，并完成真实A生成/回退、SA/SB有限入口及独立恢复。

活跃GPU/训练作业：无。B真实单批测试已退出0并释放GPU；没有教师、学生、仿真或无人值守作业。本轮没有新训练checkpoint、全量缓存或可比较模型权重；B测试仅对临时内存模型执行1次更新，F1开发checkpoint和已有1,000观测试缓存保持原样。

完整恢复材料：本机A试标的selection/protocol草案、候选规则版本、candidate_labels、300题QC准备/图文包、summary和checksums；B试缓存的contract/manifest/50分片/summary/cost_report/provenance，以及production_reader/test_production_reader脚本和测试结果；A/B真实诊断、SA/SB三步run与独立resume diagnostics checkpoints、Gemma dirty补丁和KV四份精度对照/registration/summary仍作历史依据。精确路径和命令保留本机，公开摘要用占位符；旧生成器仍拒绝覆写已有manifest，生产reader的精确resume账本已在已有分片上验证但未接入全量生成队列。

未决：A参考点差异保留、至少300题人工三层QC和阶段unknown；2 mm是候选死区而非冻结阈值，当前双侧fingerpad接触是候选grasped代理，候选标签仍未获训练许可。B第三人称特征高相似度仍只作描述性质量材料；全量合同、生产接入和自动续写队列待后续。**原S固定样本KV残差解释经负责人审阅正式收口，不再重复测试；A新增生成路径另做回归。**

本轮已授权并整理[审阅副本](review/)：A四张合图、[40题表](review/a-questions.json)、[草案协议摘录](review/a-protocol-excerpt.json)，B十观测双视图RGB/网格/深度、[对应表](review/b-observations.json)与[合同及统计摘录](review/b-contract-excerpt.json)，以及固定300题的[图文QC分组与索引](review/qc300/)。所有300题审核栏仍空，发布不等于获准训练或质量通过。图片与定义见文末；原始服务器材料保留。

下一步：完成并记录至少300题真实人工三层QC，整理正式公共SA/SB入口（当前bounded diagnostic CLI和Gemma层捕获补丁仍需合并/审计），再做F3前四组公平性、恢复和无监督推理测试；候选规则、reader、单批和bounded diagnostics不替代这些验收。当前不启动全量缓存、连续小训练、F3或正式作业。

## 执行记录

迁移时原占位记录（2026-09-07）：尚无本阶段实际执行记录。


### 2026-09-08｜F2计划细化与执行暂停

负责人接受F1收尾结论并提出F2推进顺序，随后明确要求“先完成plan，然后push，交给GPT审阅没问题再继续”。本次以最新指令为准：只交付待审计划和阶段入口，不将最初的实验推进授权继续用于当前执行。

开始时读取工作区/仓库规则、README、F1收尾和F2现有PLAN/LOG，核对文档仓库干净，HEAD为`d80775b1e91682b12f35605984c3abaa5d874c34`。现有工程HEAD为`16295beccf737e1e180718fe78af963cd8707999`，其未跟踪的规则入口保留。本轮不修改公共模型、tokenizer或训练入口。

暂停前已发生的操作如实保留：宿主只读查询显示8张RTX A6000，3张当时空闲；没有据此启动GPU实验或认领持续资源。教师侧只读源码检查及一次权重下载先于计划审阅发生，下载现已停止，退出130；本机保留707,600,384字节的`.partial`，没有完整教师权重或缓存验收。KV侧生成独立诊断草稿、编译检查退出0，但未做真实前向。源码提示默认计算/缓存精度为BF16，历史诊断将输出转换为FP32不能证明全程FP32；这只是后续受控诊断线索，不是残差归因结论。

本轮修改范围仅README及F2的PLAN/LOG：继承data-v2和F1基础；细化公共接口、逐记录状态试标、30—50问题首批材料、至少300问题人工QC、教师100/1,000观测及网格/分视图检查、全量缓存前置、A/B独立测试和小训练、F3交接边界。实验步骤均保持未勾选；没有新增管理文件、没有重分数据、没有改变科学协议或F1历史。README改为F2计划待审入口，F1既有阶段总结直接引用。

文档验证：`git diff --check`、相对Markdown链接存在性与计划未误勾选检查；三项均通过、退出0；此次仅文档改动，未运行模型或实验测试。Git目录归属检查使用临时精确路径配置解决，不修改全局信任。公开文档不含本机完整路径、GPU UUID、PID、私人规则、原始数据/权重或本轮未获发布授权的QC材料。

发布前核对：沙箱网络下首次`git fetch origin main`连接失败；切换获准的宿主网络执行后退出0。远端HEAD仍为本次起点提交，未覆盖远端更新；本轮只暂存上述三个文档文件并正常提交/推送，推送结果以最终交接中的远端核验为准。

### 2026-09-08｜方案审阅通过与首批授权

负责人审阅提交`51e53e809cee61a54604e0975ab17db5cdd08d26`后批准方案，要求定点修订后执行首批。已纠正第12层输出的编号（1-based=12、0-based=11），明确F2只做SB实际无教师导出、SAB组合留F3；补充演示内实例身份、固定参考点、不推进时间刷新、教师批次隔离、真实学生batch读缓存、QC去重和统计分母。原计划方法/质检数量不变，本批明确排除全量缓存及真实小训练。

公共接口源码核对：现有`AlignedHDF5Dataset`→`ProjectSPipeline.training_batch`输出`(Observation, actions)`，`train_s.py`显式限定F1，尚无A/B训练CLI。不能把手册schema草案直接传给该入口。本批先固定host-only监督连接合同，公共请求四键不变；A/B损失入口和单层输出仍标PLANNED，后续实现复用累计/冻结/恢复基础。data-v2清单、norm及spec登记的哈希已实际匹配，核对脚本退出0；原始源码指纹和接口合同留本机工程目录。

### 2026-09-08｜首批A真实状态材料、公共批次和KV定点结果

**A真实试标（DRAFT，不用于训练）：**固定每任务按数字排序的前5条train轨迹，共50条完整轨迹、6,274个动作起点、12,548个槽位，保留data-v2原sample_id。CPU逐状态提取实测65.45秒，无renderer/GPU；`states[i+1]`恢复后完整状态精确一致，`sim.forward()`不推进仿真时间。对象取已绑定实例root body原点，夹爪取`gripper0_grip_site`，基座取`robot0_base`；以BDDL目标操作数提供离线绑定证据，演示内不随位置重选实例，在线完整指代函数不接收GT。

几何有限且绝对值主轴非精确并列为12,548/12,548，这不是可靠标签覆盖率：边界阈值尚未采用，最小轴差约0.000000702米。草案轴符号计数为−z 6,604、+x 1,521、+y 1,895、−y 2,520、−x 8、+z 0；候选轴词映射为+x/front、+y/left、+z/up及相反方向，仍待审。双指垫接触证据2,149槽位，均在操作对象，不据此直接判grasped。获准训练标签、人审数均为0。

40个去重题目已从全集按早期、首次双垫接触（缺则中段）、最小主轴差和最后放置目标选出，覆盖10任务；题键为sample_id+槽位+协议版本，80张原图副本逐像素读回一致。它是有意富集边界/接触的审阅集合，不用于估计总体覆盖。阶段均保留unknown，选例代理不冒称已核定接近/抓持/放置阶段。另生成4张脱敏合图和配套40题表，审核栏空、明确DRAFT；公开发布另行申请，不把本机材料存在写成远端已可阅。

恢复夹爪参考点与原记录`ee_pos`存在非零差异：中位0.373毫米、最大0.676毫米；尚未归因，不随意放宽容差，不认定所有接触/边界方向可靠。该差异、对象绑定/轴语义、边界阈值和当前抓持判据构成本批集中审阅材料；A试标不改变动作样本或F1结论。

**公共输入和B读取：**2个真实样本经过已有学生`training_batch`及确定性预处理，三个相机张量均为[2,224,224,3]，第三占位attention仍为true；额外task_id、方向、教师特征、sample_id四类公共请求键均被拒绝。首次本机检查脚本因preprocess参数顺序写反退出1，纠正独立脚本后退出0，未改公共实现。3个KV固定样本另核对训练/推理prefix token、图像、state和image masks精确一致，prefix长度55/54/55与动作loss首位置对应，退出0。

实际已完成前向的trace记录三相机依次各输出[1,256,2048]，视觉总长768、文本容量128、合并长896，Gemma共18层；真实视图spans为[0,256)、[256,512)，占位为[512,768)。从100观测试缓存已提交分片取2个真实样本，逆manifest顺序按ID读取后接入现有学生训练打包入口，核对源图hash、视图、q、[2,2,256,2048]目标、[2,2,256]有效mask及上述真实视觉布局；缺ID/错源hash/错split/错合同均显式失败，检查退出0。这只证明实际数据打包和缓存连接，B loss/optimizer及SAB组合尚未实现验收。

**有限KV诊断：**同F1最终3,000步checkpoint、3固定训练样本、每样本9个logits位置，保持原权重数值、输入、位置与mask；比较计算精度，0训练更新。四个作业全部退出0，合计单卡进程占用250.46秒（约0.0696 GPU·小时，含加载/编译，不是kernel吞吐）。

| 受控设置 | 修正offset=0的suffix RMS（3样本） |
|---|---|
| 生产BF16路径 | 0.117986 / 0.168774 / 0.119567 |
| 模型计算FP32，原checkpoint参数dtype | 0.030730 / 0.032533 / 0.035901 |
| FP32计算与highest矩阵精度，原参数dtype | 0.028444 / 0.028727 / 0.032432 |
| 所有浮点参数转FP32并highest | 0.000035653 / 0.000020858 / 0.000030058 |

最后一项27/27 argmax一致、最大绝对差≤0.0003014；旧offset=1同设置suffix RMS仍为0.395376/0.461563/0.765549。该证据支持固定样本主要残差来自低精度计算路径，原位置修正仍必要；综合dtype控制不唯一归因到某一个embedding运算。不修改生产BF16、checkpoint或协议，不重评F1、不扩大容差。此S定点遗留项已有实质解释依据，不声称全输入逐位等价；A新增后缀/完整生成仍须在F3/G2前回归。

原始证据分别保留在本机A试标目录、接口检查目录和KV诊断目录，含selection、protocol草稿、逐题结果、registration、源码/hash、退出码及恢复信息；公开摘要不是直接可执行命令。已验证入口包括`trial.py --fixture`、`trial.py --limit-episodes 50 --output <本机试标目录>`、`check_pipeline.py --output <本机结果>`和`check_teacher_batch.py --cache-root <试缓存> --contract-hash <实际hash> --embed-trace <实测trace> --output <本机结果>`，此处含占位符，精确命令见本机registration。没有新训练checkpoint或全量缓存。

KV补充元数据核查：原checkpoint的input_embedding实际dtype为BF16、shape为[257152,2048]。CPU沙箱元数据首读长时间无输出，停止本项目查询后退出143；宿主CPU限时读取0.65秒退出0，失败与成功均保留。它支持存在低精度入口，不把该元数据单独当作唯一原因证明。

### 2026-09-08｜教师100观测与1,000试缓存启动

FastVGGT固定commit为`6526e275a29572653a034762bb3c6c9ce280ff55`。之前中断的权重已续传，完整文件5,026,885,758字节，与官方LFS SHA256匹配后才加载。teacher环境仅补固定h5py依赖并记录实际依赖/导入修复；未升级学生或仿真环境。

单观测smoke及100观测真实检查均退出0，严格权重加载无缺失参数。教师实际输入为[1,2,3,518,518]，每次只处理一个当前双视图观测，不跨sample_id拼场景/共享状态；源图实际128×128，启动前已纠正早期256假设。末端聚合特征为[1,2,1374,2048]，排除前5个特殊token得到每视图37×37网格，FP32双线性中心映射到学生16×16后保存FP16，输出[2,256,2048]。此处reshape仅恢复已验证稠密网格，映射用插值，不靠shape硬配。

实际合并设置merging=0、ratio=0.9，0表示从首块启用而非关闭；固定上游恢复稠密位置，但未撤销信息聚合。各真实视图有效位置等权，第三占位相机只在alignment中排除。教师无梯度，不读方向标注或未来图像。

100观测端到端58.30秒、1.715观测/秒（包含预热、40观测原生深度、100张图及写盘）；完整读回另0.992秒。峰值allocated约3.24GB、reserved约3.93GB。两视图均无NaN/零范数，原生深度80张均有限且非恒定，但没有GT深度精度验收。synthetic中心映射覆盖真实128方图resize及明确synthetic crop/padding/flip四项，真实方图最大误差约7.63×10⁻⁶原图像素；100张真实双网格叠图已生成并关联ID/源图hash，人工审核栏空。

特征分布须保留限制：第三人称/腕部的跨样本平均方向cosine均值约0.993/0.886，单位方向空间变化RMS中位约0.208/0.455。不是严格常量，但第三人称相似度很高；范数、shape和网格正确不证明监督有效，不据此换教师或加权。

100观测使用逐样本文件作接口pilot，不能作为全量默认。后续已登记启动的1,000观测试缓存采用每20个独立观测一片、manifest带row_index、校验后原子提交；只使用一张确认空闲卡，按固定1,000观测上限停止，不做全量。当前prototype遇已有manifest拒绝覆写，自动断点续写入口未实现，不能宣称已有生产全量自动恢复。已完成分片可由严格reader校验保留。

1,000格式也已从实际分片取2样本接入真实学生pipeline，ID/source hash/q/布局及4项负面检查退出0；另有row合同/视图序/row越界/payload ID/坏片hash/缺片/构造合同7项负面检查通过。完成时另补最终吞吐/覆盖；原始精确命令和心跳在本机registration，不用估算冒充实测完成。

本批关键产物SHA256（原件留本机，文件摘要不是验收替代）：

| 产物 | SHA256 |
|---|---|
| A固定试标清单 | `6e850eb8d4d49fb6a6227b57c53fd70bb0adeb486866c1e3754376cb2032f774` |
| A草案协议 | `c62bf955537185cb37aa99d71f84cbabc653d906523ffbc8197ce0ec4019e555` |
| A40题审阅清单 | `abe5eb49991da957eb00b9715b0360efac8e9bdb4c998d26feda03762f73450e` |
| B100合同 | `e96c0025f426260573e0c7c6e59e58b8d3ead17941c74cb30b836a0d1fdfe8d4` |
| B1000合同 | `a804b68badfa14c2085b86d9f2249138c2f4a77c57b3a12f6a4b54b40f378b09` |
| KV综合FP32结果 | `f28b241d474dcd3091b27977399dbae4b0837adc5c4c7e2400151c358bbfcec6` |

### 2026-09-08｜1,000试缓存全流程收口与本批交接

1,000观测全部提取、写入并逐ID严格读回，作业退出0，GPU释放。完整50片共2,098,104,180字节；提取写入循环436.764秒、末尾严格读回61.709秒，完整端到端498.473秒、2.006观测/秒。循环内预处理34.125秒、提取/映射330.557秒、写入/校验51.433秒；其余20.649秒包含原生深度、图像与QC等未单独细分，不伪造分段计时。去前10观测的稳态提取/映射均值0.3299秒/观测，仅描述该部分。

每片20个独立双视图观测，全部1,000个sample_id唯一；分片只是存储容器，教师仍逐样本B=1/V=2前向，不跨时刻共享输入。原子完整分片和manifest保留，可严格读取；生成器未实现自动断点续写，后续生产reader/恢复流程仍待实现及验证。逐ID读取重复计算整片SHA约41.96GB及读取payload约41.94GB，为逻辑访问量而非实测物理磁盘流量，不能把这份prototype称为已优化生产缓存。

按本次相同路径线性外推61,750观测，约8.55小时、129.56GB（十进制），仅为粗估，包含重复pilot排错和当前低效读回口径；不是全量benchmark，也不构成全量执行授权。正式全量前须完成质量审阅、完整合同/版本绑定、生产读取和恢复检查，按改进后的实测重估。

本轮文档仅更新README、现有F2 PLAN/LOG，实验实现与原始材料保留本机工程目录。两种缓存格式的真实学生批次连接均通过，未以桥接测试替代B训练和梯度验收。方案中已完成的接口、固定试标、草案/统计、试缓存及映射材料条目据证据勾选；状态/图像精确对应的eef残差、人工QC、全量、模块训练和F3均不勾选。新QC发布申请未获答复时不上传图片。

本轮验证与发布前检查：实际试标、KV四项、教师smoke/100/1,000及CPU桥接最终均退出0；脚本参数顺序失败和元数据查询中止记录保留。Markdown相对链接、敏感路径扫描和`git diff --check`通过。发布前fetch确认远端仍为`51e53e809cee61a54604e0975ab17db5cdd08d26`；仅提交本次相关文档，不改F1历史或科学协议，发布结果以最终回复中的远端核验为准。

1,000观测分视图收尾：各256,000个目标位置有限且非零；单位方向空间RMS中位为base0.2083/wrist0.4636，跨样本平均方向cosine中位为0.9934/0.8912。全部CPU汇总也已退出0，无后续活跃作业。完整manifest SHA256为`4fa9a0f58b747dd08eea2857906724fcb59d7a2a53d61a3caec66453b3c315dd`；summary SHA256为`7056413e26369695d326750bfd00f92dbb20b77b5c3abd0ed317a32323d05c6f`。

### 2026-09-08｜首批审阅结论、原S缓存事项收口与材料发布

负责人完整审阅`c2880139fc2255e5bf97fbe670bb4dbcaa296625`后认可首批接口/试标/试缓存的限定完成范围，并接受原S固定样本缓存残差解释项收口：修正位置的高精度suffix RMS约2—4×10⁻⁵、27/27 argmax一致，而旧偏移同精度仍明显不一致。审阅依据是既有仓库记录，不是负责人重新运行实验。本轮没有重复KV诊断，不改变生产BF16、checkpoint、F1结果或失败记录；A新增后缀、分段生成和回退缓存的回归属于新增实现测试。

本轮明确授权下列审阅副本。只更新F2 PLAN/LOG并加入review材料，README及两份主文档不改；未发布完整标注、HDF5、权重、特征缓存、私人规则、凭据或完整运行日志。发布与语义/质量采用、人审签收分开，F2尚未完成，G1仍PASS。

#### A：40题草稿与参考点证据

[直接读取40题JSON](review/a-questions.json) · [方向草案协议摘录](review/a-protocol-excerpt.json)。题表保留sample_id、槽位、协议版本、完整指代、固定离线实例、具体参考点/坐标、候选方向、主轴差、当前接触、不确定原因、选例依据及对应合图；reviewer和三层审核结论均为null。只复制原40题，不上传全部12,548槽位。合图与服务器审批候选逐字节一致，未重新生成标签。

参考点来源摘录：当前仿真工具以`sim.data.body_xpos[env.obj_body_id[instance]]`取物体root body原点，以`sim.data.site_xpos[robot.eef_site_id]`取`gripper0_grip_site`，`robot0_base`旋转给定基座轴，单位米。固定环境`SingleArm._setup_observables`中的`eef_pos`也读取`site_xpos[self.eef_site_id]`；这只核实当前源码参考点，不证明原始HDF5采集时的更新时刻/字段提取过程与恢复路径完全相同。既有最大0.676毫米差异仍未归因，不修改时序/参考点、也不要求硬修到零。

当前`_check_grasp`对gripper默认取left_fingerpad和right_fingerpad两组接触几何，并要求**每组至少一个几何体与目标contact_geoms接触**；它没有检验未来抬起/成功，也没有在这条检查中加入持续时间、承载力或稳定抓持判据。本材料据此仅称“双垫接触证据”，不直接采用grasped。+x/front、+y/left、+z/up为待审草案；boundary_threshold仍为null，几何非并列不等于高置信标签。阶段仍unknown，不为覆盖率猜标签。

![A Q01—Q10：DRAFT，审核栏空](review/a-contact-sheet-1.png)

![A Q11—Q20：DRAFT，审核栏空](review/a-contact-sheet-2.png)

![A Q21—Q30：DRAFT，审核栏空](review/a-contact-sheet-3.png)

![A Q31—Q40：DRAFT，审核栏空](review/a-contact-sheet-4.png)

#### B：选样、特征合同和统计定义

[十观测及源图/副本hash对应JSON](review/b-observations.json) · [合同与既有统计JSON](review/b-contract-excerpt.json)。按原100观测清单的任务文件排序，每任务取**第一个同时已有两视图mapping和native-depth输出**的观测，共10个；没有按成功或特征数值选例，不声称语义阶段全覆盖。原始RGB从相同不可变HDF5的既有observation_row导出，逐像素校验；网格/深度PNG从已有文件逐字节复制，没有重跑教师或改变权重。100观测来源图与1,000观测统计的清单/合同hash分别保留，不混为同一分母。

合同摘要：固定FastVGGT commit `6526e275a29572653a034762bb3c6c9ce280ff55`，权重SHA256 `b08a43baa2db1aad9718e71e098831b8ad32f6f6826c802e9eb714aa34420969`；每次B=1/V=2，当前raw 128²图，教师518²/patch14，学生224²/patch14。取教师末端第24层聚合输出（0-based23，返回列表索引3），每视图排除5个特殊token后37²稠密网格，FP32 bilinear/align_corners=False映射16²、存FP16，不预先L2归一化。merging=0表示从首块启用，ratio=0.9，恢复稠密位置不撤销聚合。此处教师第24层不混同学生拟取第12层输出。全部是pilot合同，生产reader与自动精确恢复尚未完成。

以下定义直接摘自既有`feature_qc1000.py`，本轮未重算特征：对每个观测i、固定视图v、256个位置p，读取映射后FP16目标并转FP32为aᵢᵥₚ，令uᵢᵥₚ=aᵢᵥₚ/‖aᵢᵥₚ‖₂，μᵢᵥ=(1/256)Σₚuᵢᵥₚ。

- **空间RMS**：每观测/视图计算sqrt[(1/256)Σₚ‖uᵢᵥₚ−μᵢᵥ‖₂²]；先对2048通道平方求和，再对256位置平均、开根，不再除以通道数。表中中位数取该视图1,000个观测的RMS。
- **跨样本平均方向cosine**：先将每个μᵢᵥ单位化为mᵢᵥ，再计算mᵢᵥ·mᵢ₋₁,ᵥ。原脚本按分片首次出现及片内manifest顺序收集，当前写入顺序与manifest相同；`np.roll(...,1,axis=0)`使第一条与最后一条也配对。不是全配对或独立随机样本估计，包含时间/任务关联及任务边界，表中为这1,000对的中位数。
- **补充空间shift127 cosine**：对展平256位置序列，将u循环移动127个位置后与原位置点乘，汇总该视图全部观测/位置的分位数；不代表二维相邻patch相似度。

| 1,000观测既有统计 | 第三人称base | 腕部left_wrist |
|---|---:|---:|
| 跨样本平均方向cosine中位数 | 0.9934 | 0.8912 |
| 单位方向空间RMS中位数 | 0.2083 | 0.4636 |

高平均方向相似度不等于每个位置恒定；非恒定也不证明监督有效。这些描述性指标没有有效性通过阈值，不据此换教师、加权或新增探针。网格图蓝线为原图坐标中的教师37×37边界，红十字为学生16×16中心；保留raw OpenGL方向。已有深度图按**每张图自己的2%/98%分位数**映射灰度，不能跨图比较灰度为统一米制深度，也没有GT深度精度验收。下列源RGB为真实128×128内容，可点击PNG查看；网格512²、深度518²。

**B01｜原100观测索引 0** — pick up the black bowl between the plate and the ramekin and place it on the plate

| 视图 | 原始RGB | 双网格 | 原生深度可视化 |
|---|---|---|---|
| base_0_rgb | ![RGB](review/b-01-v0-rgb.png) | ![Grid](review/b-01-v0-mapping.png) | ![Depth](review/b-01-v0-depth.png) |
| left_wrist_0_rgb | ![RGB](review/b-01-v1-rgb.png) | ![Grid](review/b-01-v1-mapping.png) | ![Depth](review/b-01-v1-depth.png) |

**B02｜原100观测索引 10** — pick up the black bowl from table center and place it on the plate

| 视图 | 原始RGB | 双网格 | 原生深度可视化 |
|---|---|---|---|
| base_0_rgb | ![RGB](review/b-02-v0-rgb.png) | ![Grid](review/b-02-v0-mapping.png) | ![Depth](review/b-02-v0-depth.png) |
| left_wrist_0_rgb | ![RGB](review/b-02-v1-rgb.png) | ![Grid](review/b-02-v1-mapping.png) | ![Depth](review/b-02-v1-depth.png) |

**B03｜原100观测索引 20** — pick up the black bowl in the top drawer of the wooden cabinet and place it on the plate

| 视图 | 原始RGB | 双网格 | 原生深度可视化 |
|---|---|---|---|
| base_0_rgb | ![RGB](review/b-03-v0-rgb.png) | ![Grid](review/b-03-v0-mapping.png) | ![Depth](review/b-03-v0-depth.png) |
| left_wrist_0_rgb | ![RGB](review/b-03-v1-rgb.png) | ![Grid](review/b-03-v1-mapping.png) | ![Depth](review/b-03-v1-depth.png) |

**B04｜原100观测索引 30** — pick up the black bowl next to the cookie box and place it on the plate

| 视图 | 原始RGB | 双网格 | 原生深度可视化 |
|---|---|---|---|
| base_0_rgb | ![RGB](review/b-04-v0-rgb.png) | ![Grid](review/b-04-v0-mapping.png) | ![Depth](review/b-04-v0-depth.png) |
| left_wrist_0_rgb | ![RGB](review/b-04-v1-rgb.png) | ![Grid](review/b-04-v1-mapping.png) | ![Depth](review/b-04-v1-depth.png) |

**B05｜原100观测索引 40** — pick up the black bowl next to the plate and place it on the plate

| 视图 | 原始RGB | 双网格 | 原生深度可视化 |
|---|---|---|---|
| base_0_rgb | ![RGB](review/b-05-v0-rgb.png) | ![Grid](review/b-05-v0-mapping.png) | ![Depth](review/b-05-v0-depth.png) |
| left_wrist_0_rgb | ![RGB](review/b-05-v1-rgb.png) | ![Grid](review/b-05-v1-mapping.png) | ![Depth](review/b-05-v1-depth.png) |

**B06｜原100观测索引 50** — pick up the black bowl next to the ramekin and place it on the plate

| 视图 | 原始RGB | 双网格 | 原生深度可视化 |
|---|---|---|---|
| base_0_rgb | ![RGB](review/b-06-v0-rgb.png) | ![Grid](review/b-06-v0-mapping.png) | ![Depth](review/b-06-v0-depth.png) |
| left_wrist_0_rgb | ![RGB](review/b-06-v1-rgb.png) | ![Grid](review/b-06-v1-mapping.png) | ![Depth](review/b-06-v1-depth.png) |

**B07｜原100观测索引 60** — pick up the black bowl on the cookie box and place it on the plate

| 视图 | 原始RGB | 双网格 | 原生深度可视化 |
|---|---|---|---|
| base_0_rgb | ![RGB](review/b-07-v0-rgb.png) | ![Grid](review/b-07-v0-mapping.png) | ![Depth](review/b-07-v0-depth.png) |
| left_wrist_0_rgb | ![RGB](review/b-07-v1-rgb.png) | ![Grid](review/b-07-v1-mapping.png) | ![Depth](review/b-07-v1-depth.png) |

**B08｜原100观测索引 76** — pick up the black bowl on the ramekin and place it on the plate

| 视图 | 原始RGB | 双网格 | 原生深度可视化 |
|---|---|---|---|
| base_0_rgb | ![RGB](review/b-08-v0-rgb.png) | ![Grid](review/b-08-v0-mapping.png) | ![Depth](review/b-08-v0-depth.png) |
| left_wrist_0_rgb | ![RGB](review/b-08-v1-rgb.png) | ![Grid](review/b-08-v1-mapping.png) | ![Depth](review/b-08-v1-depth.png) |

**B09｜原100观测索引 86** — pick up the black bowl on the stove and place it on the plate

| 视图 | 原始RGB | 双网格 | 原生深度可视化 |
|---|---|---|---|
| base_0_rgb | ![RGB](review/b-09-v0-rgb.png) | ![Grid](review/b-09-v0-mapping.png) | ![Depth](review/b-09-v0-depth.png) |
| left_wrist_0_rgb | ![RGB](review/b-09-v1-rgb.png) | ![Grid](review/b-09-v1-mapping.png) | ![Depth](review/b-09-v1-depth.png) |

**B10｜原100观测索引 96** — pick up the black bowl on the wooden cabinet and place it on the plate

| 视图 | 原始RGB | 双网格 | 原生深度可视化 |
|---|---|---|---|
| base_0_rgb | ![RGB](review/b-10-v0-rgb.png) | ![Grid](review/b-10-v0-mapping.png) | ![Depth](review/b-10-v0-depth.png) |
| left_wrist_0_rgb | ![RGB](review/b-10-v1-rgb.png) | ![Grid](review/b-10-v1-mapping.png) | ![Depth](review/b-10-v1-depth.png) |

#### 发布核验与恢复边界

本机导出入口`export_review.py`仅处理授权副本，退出0：40题唯一键、审核栏空；10任务各1观测/两视图，20张raw RGB逐像素一致、44张现有PNG逐字节一致（A4+B网格/深度40）。共64个PNG及4份JSON，约7.02MB，无新模型/仿真/GPU作业。原件、完整试标和50个试缓存分片继续保留，不覆盖或搬迁。

本轮下一步仅等待对已发布材料的审阅；A参考点/边界/轴/抓持规则和B质量仍待采用决定。生产reader及自动恢复是全量前工程任务，未实现、不冒充可运行接口。本轮不重复实验，后续无新任务时不循环检查等待状态。文档/副本的格式、敏感内容、Git范围及推送后远端逐文件内容核验结果记最终交接。

发布前实际验证：`export_review.py`与`verify_local.py`均退出0；64张PNG可解码、4份JSON可读取，40题唯一且审核栏空，B十任务来源/两视图文件齐全，导出hash、相对链接和敏感路径扫描通过；`git diff --check`退出0。fetch核对远端与本地起点均为`c2880139fc2255e5bf97fbe670bb4dbcaa296625`。推送后再以固定提交原始文件地址逐文件下载、核对SHA256和实际PNG/JSON内容，结果留本机发布验证回执及最终回复，不把本机检查冒充远端检查。

### 2026-09-11｜后续技术审阅、2 mm候选统计与生产reader pilot

负责人审阅共享交接后确认：原S固定样本KV排查正式收口，不重复同样本/同精度对照；现有物体root body原点、`gripper0_grip_site`、固定基座轴和米制单位作为标签定义采用；`+x/front、+y/left、+z/up`及相反方向作为命名约定采用。采用约定不等于40题或全量方向标签已通过视觉/人工QC，G1保持PASS。

对既有`f2-work/annotations/trial-50-v1/labels.jsonl`执行CPU-only派生程序`boundary_candidate.py`，没有重放轨迹、推进物理时间、访问未来成功或改写原draft。新版本`direction-candidate-20260911-v2`按“身份/状态有效 → 已绑定操作对象当前双侧fingerpad接触候选grasped → 六方向主轴gap”判定，2 mm（0.002 m）仅为审阅提出的candidate dead zone：gap小于阈值才判candidate invalid，接触优先于该死区。原始`labels.jsonl`未覆盖，新candidate全部`approved_for_training=false`、`valid=false`、`label=null`。

输入12,548个槽位/6,274个样本唯一键检查通过。候选结果为12,181/12,548槽位`candidate_accepted`、367个槽位进入`invalid_boundary_dead_zone`；其中操作对象6,118接受（3,969非接触六方向、2,149双垫接触grasped），非接触操作对象总数为4,125，其中156个进入死区；放置目标6,063接受；样本层按任一槽位无效回退为action-only候选，5,925样本两槽位均接受、349样本action-only。受影响367个题键及任务/角色/接触分布在本机`boundary-candidate-v2/summary.json`，该文件较大且不发布。候选类别计数为down5,488、grasped2,149、left1,573、right1,553、front1,411、back7、invalid367；这不是最终训练分布，也不构成类别平衡依据。synthetic接触优先/阈值边界fixture退出0；全量派生退出0。阶段unknown、人工审核0、获准训练标签0保持不变。

上述实现保留了待解释的恢复eef与原记录差异（最大0.676 mm），不把它硬修为零；接触代理不证明稳定承载或未来成功。后续至少300题人工三层QC仍为BLOCKED_HUMAN，需按`sample_id+slot_index+协议版本`去重，并分别审文本指代、实例绑定、方向标签。

按新规则生成了内部300题质检准备包`qc-300-prep`：保留既有40题，按任务/角色/接触/候选状态以稳定SHA-256顺序从其余固定试标槽位补260题；题键绑定`direction-candidate-20260911-v2`，300题唯一，human_reviewed=0、approved_for_training=0、stage仍unknown。该准备包只含索引/候选字段和空审核栏，没有新图片，不计为三层人工QC，也未上传完整标签。

B生产reader新增本机`f2-teacher/production_reader.py`，测试入口为`f2-teacher/test_production_reader.py`。它对现有cache1000-v2的50个20-row分片按唯一shard只计算一次hash和结构校验，按manifest row_index核对payload ID/shape/dtype/q/非有限值，`get_many`保持调用方请求顺序；reader不加载教师模型、不重提特征。原子`ExactResumeState`绑定contract SHA和manifest SHA，完成ID只在调用方成功读取后写入`.partial`再rename；恢复时完成集和待处理序列不能重排。对已有1,000行、50片完整校验以及逆序5样本读取通过：hash计算50次、payload加载51次、教师提取调用0次。实际结果`f2-work/interfaces/production-reader-result.json`，测试退出0。

### 2026-09-11｜候选标签统计与生产reader精确恢复

reader负面fixture均通过：错误shard hash、`.partial`引用和篡改manifest hash均显式失败；首次恢复测试发现并修复了待处理列表被错误改成manifest顺序的问题，修复后保持显式请求顺序，失败记录保留在本机命令日志。该reader是pilot级读取/恢复实现，不宣称已接入全量生成、train/val生产缓存或A/B训练；没有重新生成1,000观测，没有全量缓存，没有GPU作业。

本轮实际验证命令与结果：`boundary_candidate.py --self-test`退出0；固定50轨迹候选派生退出0；`qc_prep.py`生成300题内部质检准备包退出0（保留旧40题、稳定分层新增260题，human_reviewed=0、approved_for_training=0、无新图片）；`test_production_reader.py --cache-root f2-teacher/cache1000-v2 --output <本机结果>`最终退出0，并额外验证A→B→A重复请求的顺序与内容保留；`python3 -m py_compile`对新增脚本退出0；文档`git diff --check`待发布前执行。新增脚本/候选结果/QC准备包仅在工作区工程目录，不上传完整标注或特征分片。下一步进入真实人工QC与A/B独立模块实现准备；本阶段仍不启动真实小训练、全量缓存或F3。

本轮最终产物摘要：候选规则脚本SHA256=`ca1867d7eaecb828a7d923654fd1047a168d35ebcf5a7dacf610dd36cefedefa`，候选协议SHA256=`db00be577d5593956c791914fae632b13eaf3544c43e66517613e936c38cb1bb`，candidate_labels SHA256=`8d844391caa85bb37df517fbcedd1338885285a3dd829b2635a21007f26ec71e`，候选summary SHA256=`8801f4b9ed1d715894e09bbe8c0fd28887ca7f7c91e4b0708fa7f7863b2b71c2`；reader脚本SHA256=`9df6ad1581ba9313752612c5d6fb6b42243af61f6a5596fa61e6229f609eb711`，reader测试SHA256=`3def4a9232300e07cd1db3b20aff3207aecb51d8ae7c434c2394682db7ead1c8`，测试结果SHA256=`e5dd03a33001a68f72884d78e8a0d1c3742893480fc64cbbe81ced8da7493963`；QC准备脚本SHA256=`f8754fa5df67ba1848c5aed02d5b26aa5b2626e474b434336c908c2ef1283797`，300题准备JSONL SHA256=`9bb180777b4b72aa9960ef6c83f6a056210ae8aa695c2fe2c014cd8032c7eced`，准备summary SHA256=`45813a20080dcc254a61aaa26c06394cf39347b58a6ad85ffa01ad6fcd5ee815`。最终GPU只读快照显示0—7均14/15 MiB、0%利用率，所有F2子作业已退出；没有活跃训练/教师/仿真进程。初次reader测试的两个失败（恢复顺序断言、临时目录mkdir）均保留于本机命令历史，修复后回归退出0。

### 2026-09-11｜A/B独立实现接口测试与真实B梯度检查

A独立实现位于本机`f2-work/a_module.py`：`DirectionSequenceBuilder`保持S的原Action段，A启用时将完整候选方向序列置于同一因果后缀，方向/动作loss mask和next-token target同步移位；无效监督返回action-only，不插假答案；`DirectionTrie`支持多token候选和完整边界；方向非法由`plan_action_only_fallback`返回全新的action-only prefill计划，半段方向token仅作为被丢弃记录。`test_a_module.py`合成CPU测试退出0，覆盖A关闭等价、multi-token、overflow拒绝、trie边界、回退和分样本损失分母。`test_a_tokenizer_real.py`使用固定本地tokenizer退出0：七类中`grasped`编码为2个子token，其余为1；S前缀长度51、Action段7 token，A关闭的token/mask逐项一致，加入`left,grasped`后Action段仍逐项一致；没有读取真实方向标签或训练。

B独立实现位于本机`f2-work/b_module.py`，只 gather审计的真实视图span `[0,256)`/`[256,512)`，不取占位`[512,768)`；FP32归一化/余弦按每样本有效位置均值再batch均值，教师stop_gradient，投影头为LN→Linear(1024)→GELU→Linear。`test_b_module.py`合成CPU测试退出0，覆盖q=0目标不变、均匀权重、学生/投影梯度和教师零梯度。

为取得真实学生第12层输出，在dirty的`upstream/openpi` checkout给Gemma `Module.__call__`增加可选`return_layer`路径：默认路径保持原行为；捕获路径在同一次层scan中只保留目标层输出，返回的是第12个Transformer block输出（0-based代码索引11、最终norm之前），没有保存全部层。当前dirty源码`gemma_fast.py` SHA256=`4b46624557fe4875499bc8599577b3cdebc35880f0ebd67e076861f643aa734f`，基线implementation commit仍为`16295beccf737e1e180718fe78af963cd8707999`；不向上游远端提交。默认S路径未改其调用参数/行为，后续F3仍需公共路径回归。

在物理GPU5（启动前两次14 MiB/0%/P8检查）运行`test_b_real.py`，最终作业退出0；前面四次尝试均在真实学生前向前失败并保留：reader导入路径、1步cosine日程、manifest无split字段、单样本目标缺batch维。最终只执行一次真实学生前向、一次反向和一次临时内存optimizer更新，未保存checkpoint，未读取A标签，教师提取调用0。

真实结果：sample为data-v2首样本，教师合同SHA256=`a804b68badfa14c2085b86d9f2249138c2f4a77c57b3a12f6a4b54b40f378b09`；目标单样本`[2,256,2048]`经batch包装后接入，q有效计数512；学生层输出`[1,896,2048]`，Gemma depth18，真实两视图spans和占位排除均符合trace。alignment loss=1.0069363，反向wall约22.83秒；十个共享LoRA叶子梯度均非零（最大约0.01158），projector梯度全局范数1.08560；临时更新后LoRA参数最大差异0.02417，冻结参数最大差异0。该证据证明本批alignment loss确实能到达共享LoRA并且B的真实视图/第12层接口可运行，不证明教师监督有效、连续训练稳定性或四组公平性。

本批A/B测试和实际单批更新均为接口/诊断，未形成可比较模型，未改F1 checkpoint、production BF16或科学协议；GPU5作业结束后的宿主快照为14 MiB/0%/P8，无活跃相关进程。A/B公共训练入口、训练恢复与投影头导出仍待集成；至少300题人工QC、B连续小训练、全量train/val缓存、SB无教师导出和F3仍未完成。

本批新增实现当前hash：`a_module.py`=`9b74c24a294da3ee83ac46b6338099be648a58398d893778aebb6b0cd42ca34d`，A合成测试=`53b4b76e76fae0c4333ad724524ffaf7b838e410b2e03b31aa479af7c2397e10`，A真实tokenizer测试=`6834eb98ac053aaffc214c33f3ad4ca60f65e979064328154a2616d5dcd8e832`；`b_module.py`=`0f1d6812fd7a1b563a1f8a9d61f249203f68901c401935b86369db7e04c059c9`，B合成测试=`171721bf99b67182dd22e20aff88f2fde468af28bfda450509e3dd959c0fdc92`，B真实测试脚本=`802b4cbee49305f251e415b8697d1527bf6622648be7e145e71403bf00bb01d6`，B真实结果=`e7a5faf0f4934df04e7f476579ec45f46babd9fa081d4c3ab95a8ac04c01763b`；dirty Gemma源码=`4b46624557fe4875499bc8599577b3cdebc35880f0ebd67e076861f643aa734f`，其diff证据=`f2-work/interfaces/gemma_fast-layer-capture.patch`（SHA256=`b08a82e5c7fbc15e653d0f6813b0408f6280ac438ba87ec6958439d5a9856003`）。这些实现和原始结果只保留本机，Vault仅发布本阶段文档摘要。

### 2026-09-11｜固定300题图文审阅与SA/SB入口恢复回归

本轮不重新生成或发布QC图片；固定的300题图文包仍为`review/qc300/`，当前审核数保持0。只在阶段日志明确审阅列：后续QC使用`qc_rule_version`及其`candidate_label/candidate_status`，旧`protocol_version/candidate_direction`仅作历史追溯，不可直接作为训练标签。已核对的分项口径保持：非接触操作对象4,125个，其中3,969个候选接受、156个进入2 mm死区；不修改原始labels或上一轮统计。

**SA/SB可执行诊断入口：**新增本机`f2-work/train_ab_diagnostic.py`，复用F1的`init_train_state`、数据pipeline、冻结过滤和Optax超参，并按`--variant sa|sb --mode run|resume`做输入校验。SA在固定三步schedule上使用synthetic方向`left,grasped`，真实模型前向中两槽位的顺序/分隔/动作段均进入同一loss；不读取A candidate/QC标签。SB使用已有cache覆盖的实际schedule `[0,44,89]`，同时计算非零动作loss与`0.1 * alignment_loss`，投影头与共享LoRA分别进入临时优化器；不扩充cache或读取未来信息。该CLI是F2 bounded diagnostics，不是正式效果训练入口。

SA `run`在第1—3次有效更新均完成，第2步保存完整诊断检查点；独立`resume`进程从第2步恢复并完成第3步。恢复后的sample index/ID、action/direction loss、LoRA trainable fingerprint和model optimizer fingerprint与run第3步逐项相同，外部比较脚本退出0。SA有效更新总数3，checkpoint写入1，resume执行1；无真实标签、无正式模型。

SB `run`在第1—3次有效更新均完成，第2步保存包含LoRA、projector、两个optimizer状态、step、schedule和RNG的诊断检查点；独立`resume`从第2步完成第3步。恢复后的sample index/ID、动作loss、alignment loss、LoRA/模型optimizer/projector/projector optimizer fingerprint逐项相同，比较退出0。SB第1步动作loss=15.3125、alignment=0.996945；第2步14.0/0.998881；第3步14.5625/0.991707；这些是接口诊断数值，不是收敛或效果结论。SA/SB检查点分别约266 MB/311 MB，均不进入正式主表、不覆盖F1 checkpoint。

恢复实现过程中保留并修复了真实阻断：SA首次保存使用相对路径被Orbax拒绝；随后发现Optax内部NNX State和namedtuple经直接恢复丢失类型，最终改为纯数组叶子加当前tree definition重建，v5恢复通过。SB首次使用cache未覆盖的索引50/12345而显式KeyError；第二次误用inference_batch使动作mask全0，动作loss=0，均未计为通过；改为带真实动作后缀的training_batch并按实际缓存sample_id重跑v3后通过。失败registration和不完整目录保留，不把失败尝试隐藏成成功。

**受影响公共S路径回归：**`test_public_layer_regression.py`在同一基础权重/真实训练样本上比较Gemma层捕获关闭与开启但不引入B损失的完整路径，最终pre-logits/logits/action loss差异均为0；替换一个有效动作后缀token后，层12视觉prefix 512 token最大差异和RMS也均为0。捕获输出为`[1,895,2048]`（前向去掉最后预测位），退出0；不读标签、不更新参数、不保存checkpoint。该结果支持当前可选层捕获不改变公共S路径，同时A/B正式入口仍需统一集成回归。

**A真实路径补充：**此前真实模型诊断继续保留一个公开前缀→受限七类方向生成→动作生成，以及受控错误→新action-only prefill；本轮没有重复同一生成。现有结果仅支持方向候选`left`和回退确实执行，正常/回退动作均返回严格`invalid_coefficient_length`；该错误按策略失败记录，不补零、不截断，不声称A完整推理通过。SA三步入口的两个synthetic槽位进一步覆盖了真实模型输入顺序和动作边界，但不替代自回归双槽位方向预测验收。

本轮实际验证：A/B helper合成测试、固定tokenizer smoke、QC准备包唯一性、生产reader回归、SA/SB三步run/resume、公共S层捕获回归均有退出0记录；所有GPU作业结束后宿主快照回到低占用，未留下本任务进程。当前剩余任务是：至少300题真实人工三层QC；将bounded诊断逻辑整理进正式共享SA/SB训练入口并保留动作/方向/对齐独立分母；A正式双槽位生成/回退回归；SB无教师导出；连续小训练；全量train/val缓存；F3四组集成与G2冻结。当前不启动正式长训练。

### 2026-09-11｜固定300题图文QC包与真实A生成/回退

按已固定的`qc-300-prep`集合导出F2审阅材料，没有重新选题、重放轨迹或生成新标签。每题从与candidate相同的train manifest `observation_row`读取两路128×128 RGB，按10题一组生成30张可读PNG，并生成一份300行`qc_questions.jsonl`和`index.json`；两路原图逐题读取校验600/600，30组PNG均可解析，所有审核栏为null，`human_reviewed=0`、`approved_for_training=0`、stage继续unknown。输出目录为`review/qc300/`，未包含HDF5、全量标签、权重、教师特征或内部路径。该材料现在可供逐题审阅，但发布不等于QC通过或训练许可。

本机导出验证退出0：固定300题唯一键、40题历史保留、10个任务均有样本、候选状态/接触/参考点字段齐全；30张分组图逐一解码。已抽查首组、中间组和末组，第三人称/腕部图与右侧字段可读，审核空栏和`candidate=invalid/grasped`状态清楚。完整文件hash与来源hash写在`qc300/checksums.json`和`qc300/index.json`，不把27 MB发布包的大小当作质量证据。

A真实模型诊断使用基础权重、固定公共原始指令/两路RGB/8维状态和全七类候选Trie，不读方向标签。模型自回归生成了完整合法方向候选`left`（token IDs `[1672,108]`，末token为答案边界），随后将这段模型自身输出接到256步FAST动作生成。触发受控方向错误后，丢弃3个方向token并重新用原始77-token action-only前缀prefill，再次运行FAST动作生成；`fallback_matches_original_prefix=true`。两次动作解码均为显式`invalid_coefficient_length`，没有补零、截断或送入仿真；这说明真实方向→动作和错误回退路径确实运行，不能当作策略成功或动作格式已稳定。该作业在物理GPU4启动前两次检查，最终退出0、有效更新0、无checkpoint；结束后GPU4回到12 MiB/0%/P8。

A真实路径的原始动作token来自上游采样器的float32容器，但所有值为整数，诊断通过与公共服务相同的显式int32适配后才调用严格FAST解码；非有限或非整数值会直接报错。首次因未做该适配而产生的`invalid_token_shape_or_dtype`已保留，不将适配错误混入模型解码结论。当前模型生成仍有长度错误，后续A回归需修复/报告，不能声称完整A推理通过。

本批新增QC/真实A产物只在本机工程目录保留：`f2-work/annotations/qc-300-prep`为无图片准备索引，`f2-work/publication/export_qc300.py`为可复用导出入口，`f2-work/interfaces/a-real-result.json`和registration为真实生成/回退回执，`f2-work/interfaces/b-real-result.json`为B单批回执。A真实脚本SHA256=`75c8594ee01508dda0d08dd2645ec6425b2d571803cce942498c39b34db8211b`，QC导出包26,081,379字节；精确hash见本机registration，公开文档不上传这些完整运行产物。

本轮状态：A真实模型路径已验证但动作长度仍失败；B真实单批梯度/冻结已验证；固定300题图文材料已发布供人工审阅；A/B公共训练入口、训练恢复、连续小训练、全量train/val缓存、至少300题人工三层QC、SB无教师导出和SAB/F3仍未完成。下一步不重复统计、reader或KV诊断，先处理真实QC反馈并把SA/SB接入同一公共入口，按每条诊断路径最多5次有效更新的边界执行。

本轮新增本机产物hash：`test_a_real.py`=`75c8594ee01508dda0d08dd2645ec6425b2d571803cce942498c39b34db8211b`，A真实结果=`cffb1caa938db1e41bfcf8c946370a11bf834b34702ae47c01daea12743a01ee`，QC导出脚本=`d0e4852d3aa1c70aeede8e2c5414a25508b7a523b3fce81338415e17176b8ec6`，QC300 JSONL=`c7d5ed972bb084df51b36ad27fc484c63a5c158ab5c6f7c78611b2ed6e2c876e`，QC300 index=`9e08658e89c9a13113154f9f0dbd0d93c882b5681441626b889e764fd90279fa`。A首次动作解码适配错误保留在历史registration，修正后最终诊断退出0。

发布核验：固定提交`db7db0044f342bf1913aa8090c8a1763e6a90a53`推送成功，远端`main`指向该SHA；从GitHub raw地址实际下载F2 PLAN/LOG及review下全部文件共103个，合计33,166,692字节，逐文件SHA256与本地提交内容一致，退出0。QC300 30张分组PNG、300行JSONL、index/checksums均包含在这103个文件中；审核栏仍空，发布不等于人工QC或训练许可。

本轮新增诊断回执hash：`train_ab_diagnostic.py`=`bfb7c7224b128b1e931dc2f1a9ed5a7fb52a9314ee5f27b7ad145bc82a13da05`；SA run/resume=`ffb51b1adee76e0a623389d823ad17e7888e07ae319eab68d577f6a190459fb4`/`fd3df2213294cfb0f187fd3defec7588d12e9e1e79714441ab7cdff19b56b00d`；SB run/resume=`e9f7ed63bd99c5a8c8a2d11fad5c943d4aaf19bffcd952c48d1be42520785fb7`/`e3fdabbe28b242b782f7caa1df7943d0484c4545fae8892e781a7c820652d966`；公共S回归脚本/结果=`c789d43058609c559805501c2f7583c26654f60fb8f5e4f71629c60c29ea7848`/`880d143c6db69b54f08c5d6497f1dbb24c72878ce79e12960b26fc9001a5272f`。SA/SB诊断checkpoint目录约266/311 MB，均为本机diagnostics，不进入Git或正式主表；详细registration、失败回执和恢复路径留本机。
