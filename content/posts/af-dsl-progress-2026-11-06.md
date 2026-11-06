---
title: "AF DSL 进度日报 2026-11-06"
date: 2026-11-06T09:20:00+08:00
categories:
  - AF DSL
tags:
  - AF DSL
  - 进度日报
  - AI自动发布
description: "AF DSL 日报：连续第二个活动日——11-05 11:56–23:38 约 11.7 小时 17,023 个文件（本轮零生产代码改动），GEMM 性能实验链从 AF vs TileLang 扩展为 AF vs TL vs PyTorch 三侧。头条发现：AF 自动选参在 8192³ 选小 tile 导致 54.4× 差距（1402→25.68ms，固定 64×64×256 后同 ELF 反超 TL）；确立 torch.mm 真上限基线（AF 固定后仍慢 8.7–9.3×，根因 MAC 15.4% vs 99.8%、逐 K PIPE_ALL 无分层 L1/L0 流水）；'动态选参=更快'与'逐 shape 选参有正收益'两项假设被实验推翻/未确认；全动态 K 三变体再证 M5 AF_STORE_VALIDITY 阻塞。零提交（HEAD 停留 b8bd773e，连续第六日）、architecture.md 零新条目，模块百分比连续第十日持平（M0–M7 约 90/95/85/90/95/80/85/80%），工作树 259→267 项，industrial_release_ready=false。由大模型定时任务自动生成。"
draft: false
---

> ⚡ 本文由大模型定时任务自动生成。数据来源为远程机器上的 AF DSL 项目架构文档快照（`architecture.md`，最后修改 2026-10-30 11:13:41 +0800，304,956 字节，md5 `edc93ae5530c19406f91ea093ae0ab85`，与 11-05 日报记录逐字节一致，最新条目日期仍为 2026-10-30，**本轮零新架构登记**）与远程 Git 只读状态（HEAD `b8bd773e`，2026-10-30 10:13 +0800，零新提交；`.git/index` mtime 仍为 10-30 10:13，无暂存/无提交；工作树 267 项未提交变更），以及本轮新增的全树文件级 mtime 核查（11-05 09:10 之后共 17,023 个文件被写入/修改，**全部位于 autofuse/dsl 子树内、dsl 之外与 .git 内均为 0**）与 11-05 晚间新增 4 份开发状态文档全文阅读（含 `gemm-large-af-status.md` 的固定消融/PyTorch 差距诊断/机制勘误三个追加节；4 份对应设计文档与 6 个新 examples 目录只做身份核查）。如与实际代码/证据冲突，以源码及对应 development 记录为准。

## 总体结论

八个模块（M0–M7）均已有实现，**M1–M4 已完成本轮阶段交付**；主线仍是 **M5 集成修复**，之后 M6、M7 整体验收。模块百分比连续第十日持平（M0–M7 约 90/95/85/90/95/80/85/80%），`industrial_release_ready=false`。

与 11-05 日报"11-04 17:49 起开发活动恢复"的判断衔接，**11-05 白天至深夜（11:56–23:38，约 11.7 小时）活动持续**，共 **17,023 个文件**被写入/修改（其中 16,974 个为 `outputs/` 实验产物——17 时一小时即有 15,755 个，全部来自 `gemm-large-dynamic-20261105` 的 TileLang Carver 自动调优搜索中间产物；35 个为 6 个新 `examples/` 实验目录；8 个为 4 对新设计/状态文档；6 个为 .ruff_cache）。**本轮 `src/` 与 `tests/` 零改动**——与上轮（含唯一生产改动 M7 L1 elision）不同，本轮为**纯实验日**。性质是 11-04 性能归因实验链的延续与扩展：对比基准从 AF vs TileLang 两侧扩展为 **AF vs TileLang vs PyTorch torch.mm 三侧**，共 7 个实验批次：

1. **动态 GEMM 序列累计比较（11:56–12:05，`examples/gemm-dynamic-sequence/`）**：六次请求累计 p50（含真实 `af_prepare_tiling`/分派、tiling H2D、launch、同步）——AF 真实模型逐请求选参 348.982μs vs TL 固定串行 90.664μs / TL 已知 shape 分派串行 84.986μs。**"动态选参能力=更快"被推翻**：AF 模型能按 M/N 变换 tile（128²→19×32×16、512²→103×32×16、2048²→64×64×16），但未证明其选择最优或近优；AF 独立 solver 探针 p50 仅 2.323–2.754μs，累计差值不能单独归因于 solver。
2. **动态 shape tiling 额外开销预算（15:54–16:02，`examples/gemm-tiling-budget/`）**：12 shape × 108 配置共 1,296 单元全部通过精度/canary；逐 shape 赢家 vs 全体共用配置的独立确认配对差值中位数 **-0.007μs（bootstrap 95% CI [-0.016, 0.011]μs，过零）——未确认正整体预算**，即逐 shape 选参收益在本候选集/shape 分布下不可区分于零；且 2048² 搜索选 stage2 在独立确认中反慢于共用 stage3（-0.576μs，**排序反转：调优赢家≠真实最优**）。本轮未运行 AF 或修改其模型。
3. **超大动态 GEMM：TL Carver vs PyTorch（18:04–18:15，`examples/gemm-large-dynamic/`）**：M/N/K 全符号，四组大 shape（8192³ 至 16384³）。**TileLang 比 torch.mm 慢 10.05–12.56×**（8192³：36.400 vs 2.956ms）；同时暴露上游问题清单——Carver 不识别 950（回退 910A profile）、**默认 L0K128 数值失败（512³ probe 253,244 个失败元素，max_abs=2.298555；L0K16 通过，CANN9.3+Persistent 独立复认失败）**、torch2.14/torch_npu2.14 导入即崩溃（allocator hook 重复注册，需隔离安装 2.12 wheel）。结论：**当前 TL 数字不代表公平性能上限**。
4. **AF 超大动态 GEMM（19:25–19:35，`examples/gemm-large-af/`）**：全动态 M/N/K 目标**仍阻塞**——裁剪分配、全 tile Persistent、显式首轮初始化三变体均在 M5 失败（`AF_STORE_VALIDITY: loop-carried prefix has no bounded stable certificate`），`af.fill` 探针在 M3 拒绝（不存在此原语），诊断全部封存于 `outputs/gemm-large-af-20261105/failed-*`。已完成路径为动态 M/N + K 静态（8192/12288/16384 分别编译，三 ELF）：8192³ AF 1402.190ms vs TL 36.415 vs PT 2.983ms——**自动选参选了 16×32×16 小 tile，差距 54.4×**；三侧同机重测、300 样本、96 点 FP64/canary/独立物理 checker 全过；native tiling 五批×1000 次最大 p95=0.611μs（<10μs 门禁达标）。
5. **固定 64×64×256 消融（20:51–20:53，`examples/gemm-large-af-fixed/`，本轮头条）**：同 ELF 仅换固定配置——8192³ 从 1396.821 → **25.677ms（54.4×加速）并反超 TL（36.356ms）**；四 shape 加速 54.4×/4.5×/2.9×/3.0×。score 下降 12.2–14.5% 而真实收益远大于评分差——**实证确认：默认无 domain 路径候选漏召回 + relative-work 评分量级不准确**。两配置复用同一 ELF（串行 K、28 核、group_size=1 原样例硬编码未寻优）。
6. **PyTorch mm 差距诊断（22:28–23:35，`examples/gemm-pytorch-mm/`）**：AF 固定 25.546ms vs TL 36.330 vs PT 2.951ms @8192³（AF/PT = 8.66–9.29×，profiler 物理task duration 口径）。**根因定位**：8192³ 正式 PMU——AF MAC ratio 15.4% vs TL 52.6% vs **PT 99.8%**（PT MAC 忙 2.946ms、MTE2 忙 2.186ms vs AF MTE2 忙 16.100ms）；AF 生成代码逐 K 存在 `PIPE_ALL`，CANN（torch.mm 实际走 aclnnMm→MatMulV3）采用分层 L1/L0、L0 ping-pong 与按 buffer 同步；tiling 解码 L1=256×256×256、L0=256×256×64，前三 shape BASIC、16384³ STREAM_K。理论 GM 输入读取 M/N 64→256 时 32GiB→8GiB（8192³，无跨 tile 缓存复用口径）；Memory 采集 GM 读带宽×aicore_time 相对量 AF/PT=3.979。
7. **MatMulV3 机制补充及缓冲字段勘误（23:38，`outputs/gemm-pytorch-mm-mechanisms-20261105/`）**：勘误——旧分析文字"l1BufferNum=3"有误，实际 packed 字节 [0,2,1,1] 即 **L1 双缓冲**（L1 512KiB×2、L0A/B 各 64KiB×2、L0C 256KiB 单缓冲，与真实日志容量一致；旧计时/配置不受影响，仅说明文字错误）。真实选择是 CANN host 策略注册表（前三 shape priority4 BasicAswt、16384³ priority3 StreamK），非逐配置上板 autotune；STREAM_K 该分支为 DP+SK：4096 输出块先 146 轮完整波次、剩 8 块各切 3 段 K 形成 24 子任务，FP32 workspace 7,340,032B，Vector 求和转 FP16。

**但工程身份未变**：全部工作停留在工作树，**零提交**（HEAD 仍 `b8bd773e`，自 10-30 10:13 未变）；`architecture.md` 未获任何新条目（md5 逐字节一致）；工作树待整合条目由 259 增至 **267 项**（27 处修改不变 + 240 个未跟踪；新增 8 项 = 4 份设计文档 + 4 份状态文档，另有 6 个新 examples 子目录不增加 status 条目）。今日（11-06）截至 09:08 检查**零新文件活动**。

## 模块进度总表（M0–M7）

百分比沿用文档工程估算（误差约 ±5–10 个百分点，是排期参考，不是验收完成率，也不是测试通过率）。**本轮零提交、零生产代码改动，八个模块提交身份与估算连续第十日不变**；本轮全部实验文档均自声明"不修改生产代码、不提交 commit、不改变模块进度或设备资格"：

| 模块 | 进度估算 | 当前状态（提交身份） | 下一步主要工作 |
|---|---:|---|---|
| **M0** AF Dialect | 约 90% | 基础已提交；v2 正值契约 verifier `cfd5e7fa`（10-30）。本轮无新提交、无改动 | GEMM 逻辑有效域、内容读取、host 投影等共享契约修复及组合回归 |
| **M1** DSL API/JIT | 约 95% | 阶段交付完成（`c3ed1b08`）。本轮无变化 | backend 声明与真实 provider/Request 认证链的集成 |
| **M2** Graph→DSL | 约 85% | 转换层交付完成（`2e11db10`）；默认生产入口阻断。本轮无新实证（上轮动态 M/N/K 复用失败已记录） | 默认 production 入口能力注册、真实实现认证与设备链闭环 |
| **M3** DSL→AST | 约 90% | 阶段交付完成（`226d09e3`）；recv→view 晚绑定 dtype 缺口仍开放。本轮新增边界证据：`af.fill` 探针在 M3 拒绝（不存在此原语，非合法 DSL 候选） | 该缺口修复、跨层回归与性能告警处理 |
| **M4** AST→MLIR | 约 95% | 构造层交付完成（`9d833992`）。本轮无变化 | typed FA 执行域与下游导出合同的最终集成 |
| **M5** Pass 流水线 | 约 80% | actual-domain 正值证明已提交 `1989493d`（10-30）；typed-FA 正 K 首红（`AF_CHANNEL_VALIDITY`）保持 open。**本轮阻塞证据 +1 但无修复**：全动态 M/N/K 三变体（裁剪分配/全 tile Persistent/显式首轮初始化）公共编译再次 M5 `AF_STORE_VALIDITY` 失败；513×509 bk32/64 尾块 18 份反例仍未修复（本轮实验再次确认未修复） | typed-FA 正 K 证明（可传递 precondition 与数学证书）、**定位修复 513×509 尾块 18 份反例**、动态 K=0 真实 artifact 迁移、deferred Join P2、最终组合与完整回归 |
| **M6** Tiling 求解 | 约 85% | v2 契约透传已提交 `c45887ab`（10-30）。本轮实验给出重要实证：native tiling 最大批次 p95=0.611μs（<10μs 门禁达标）；**自动选参质量缺陷实证**——8192³ 自动选 16×32×16 导致 1402ms（vs 固定 64×64×256 的 25.68ms，54.4×），确认默认无 domain 路径候选漏召回 + relative-work 评分量级不准；逐 shape 选参收益未确认（中位数 -0.007μs、CI 过零） | 完整成本/策略/ABI 验证及最终组合下原生 tiling `<10μs` 门禁；**auto 选型质量校准（通用联合候选生成修复漏召回 + 设备评分校准——本轮实证后成为最高优先级工程事项）**；group_size 语义及其性能收益单独验证 |
| **M7** AscendC 发射/运行 | 约 80% | TPL4 全流程闭环已提交 `b8bd773e`（10-30，当前 HEAD）。工作树未提交改动两批：10-30 的 CV 融合/大矩阵验证（6 个 src 文件，+218 行）+ 11-04 的 L1 elision（`AscendCEmitter.cc` +98 行，p50 降 15.45–19.66%）。本轮零新增 src 改动，但 PMU 机制诊断（AF MAC 15.4% vs PT 99.8%、逐 K PIPE_ALL、无分层流水）为优化方向提供依据 | **L1/L0 分层 tile 与跨 K 预取流水水的独立设计、资源/同步证明与性能消融（本轮诊断确立的新方向）**、Scalar 配置特化与循环不变量外提消融、view/非 identity 负向 UT、完整 NPU 支持矩阵、默认 production 链、wheel/缓存、可靠性与公平性能对标 |

## 测试基线

**完整 host 基线（候选79 冻结身份，排除 codegen/e2e/npu）：9,330 项 = 8,778P / 459F / 93E / 0 skip —— 不变，无新全量。**

**M5 tests/passes 完整回归（工作树口径）**：最近一次全量快照仍为 10-29 K0 政策重置后的 **5,864 = 5,042P / 445F / 377E**，本轮无重跑。

**本轮新增切片级验证（均真实执行、manifest 均 informational）**：

- **动态序列**：run exit0（5 条路径 PASS、两种同步模式各 5 组）；verify exit0——60 份输出、250 个累计样本、15,000 次请求复核、`failed_numeric_cases=[]`、max_abs=3.11e-7。
- **tiling 预算**：run exit0（12 shape 各 5 组搜索与独立确认）；verify exit0——1,296 单元、2,630 份输出引用、19,915 个有效样本、1,019,500 次计时 launch、max_abs=8.46e-8。
- **超大动态对比**：38 组候选/shape 全部通过精度与上板计时、16 个动态二进制；独立 verify exit0（4 直接二进制重放、96 点 FP64、最大双源差 0.0039）；`default_l0_probe` 如实保留 `accuracy_failed`（253,244 失败元素）。
- **AF 超大动态**：run exit0（`MN_PASS_DYNAMIC_K_BLOCKED` 4 cases 3 binaries）；verify exit0——4 case PASS、300 计时样本核验、96 点 FP64、16 次数值检查零失败、最大 FP64 抽样误差 0.0011；全动态 K 三变体 failed-\* 诊断封存。
- **固定消融**：run exit0（4 cases）；report exit0——24 项数值检查、400 计时样本、16,253 个冻结文件 SHA 保持。
- **PyTorch 诊断**：主采集 84 + 补采 21 + Memory 21 条真实 launch，18 项数值检查、4 个实际 tiling 解码；8192³ 正式 5/5 有效 PMU；26/84 条 PMU 缺失如实逐项报告样本数、不补零。
- **机制勘误**：303 个旧输出 SHA 不变；packed_bytes=[0,2,1,1]、memory_bytes=[524288,65536,65536,262144]、STREAM_K 尾波算例（tail_tiles=8、split_tasks=24、workspace=7,340,032B）核验通过；HTML 40 视图脚本校验 PASS。

**设备活动连续第三日**：11-05 全天多轮真机 benchmark 与 msprof 采集（19:25–23:38），但均为实验/切片级身份，不改变完整基线数字。

## M5 收口与排期

**M5 收口的 3–5 个工作日预算窗口（10-27 评估，11-03 到期）已过期三日，仍无重新校准记录、无 M5 提交。** 连续两个活动日全部投向 M6/M7 性能方向与示例实验，**M5 三工作包（GEMM 边界与 FA、live-scalar 流水、最终集成与回归）零推进**——typed-FA 主 gate（`AF_CHANNEL_VALIDITY` 稳定证书拒绝）保持 open。

本轮对 M5/M6 阻塞与选型的证据面变化：

- **全动态 K 阻塞第三轮定量复现**：三变体（裁剪分配、全 tile Persistent、显式首轮初始化）公共编译均 M5 `AF_STORE_VALIDITY: loop-carried prefix has no bounded stable certificate`；`af.fill` 探针 M3 拒绝。失败输入/诊断封存于 `outputs/gemm-large-af-20261105/failed-*`。
- **513×509 尾块反例仍未修复**（18 份 bk32/64 反例，动态序列实验文档明确"原有 AF 流水、动态 K 和 513×509 部分 bk32/64 尾块失败均未修复"）。
- **选型策略证据（对 M5 收口后的默认路径设计有直接影响）**：逐 shape tiling 选参收益未确认（CI 过零）+ 调优赢家存在排序反转——若后续默认路径依赖"逐 shape 自动选参"，需先证明其召回与评分质量，否则固定/少量配置可能更优。
- **待整合面继续扩大**：259 → 267 项（+8 份文档）。10-30 以来的工作树改动（CV 融合 + 大矩阵验证 + L1 elision）与 11-04/11-05 两轮共 30 份实验文档**均未登记 `architecture.md`**——后续任何提交须按 AGENTS 硬性原则 14 在同一 commit 内补齐架构登记。

## 发布阻断清单

| 优先级 | 阻断/工作项 | 完成判据 |
|---|---|---|
| 1 | M5 最终组合与真实失败修复 | 9,330 节点基线的 459F/93E 逐项闭环；K0 政策后的 445F/377E 按独审分类逐项处置；实际收集的最终库存全验 |
| 2 | 数值/内容读取/视图/typed FA 合同 | 原001 完整 pipeline、typed FA 及独立正负 oracle 到达目标阶段并通过；513×509 bk32/64 尾块 18 份反例定位修复（保持开放） |
| 3 | 默认 production 能力链 | M2 真实 provider/catalog、Request/Decision/target、跨进程认证正向通过 |
| 4 | 有限 policy 与 resident 完整链 | 同 ELF 动态选择、实际 GM 遍数/状态存活、资源/同步/成本及精度设备验证闭环（本轮动态 M/N 单 ELF 跨 shape 与固定配置复用同 ELF 属该方向的实验性证据，未关闭门禁） |
| 5 | 构建/包/性能/可靠性/发布 | 最终身份独立构建、wheel、缓存、长稳/并发/异常恢复、原生 tiling 与公平 baseline 证据齐全，ReleasePlan 真实聚合通过 |

**优先级 1–5 无正式新增、无关闭**；`industrial_release_ready=false`。本轮值得注意的三点：

- **优先级 5（公平性能对标）证据面大幅扩展**：对比对象从 TL 扩展到 PyTorch torch.mm（aclnnMm→MatMulV3）真上限基线；AF 固定配置后与 TL 的相对位置（8192³ 反超、其余 shape 仍慢 2.9–4.5×）与对 PT 的剩余差距（8.7–9.3×）及其机制根因（MAC 15.4% vs 99.8%、逐 K PIPE_ALL、无分层 L1/L0 流水）均有真机 PMU 数据。同时诚实保留：TL 侧受 Carver 无 950 profile 与 L0K16 限制、PT 走 CANN 预编译分支，均不代表各自上限。
- **M6 auto 选型质量从开放问题升级为有实证的最高优先级事项**：54.4× 差距实证 + 候选漏召回 + 评分不准确认（固定 64×64×256 均优于原 10 个候选中的赢家）。
- typed FA 主 gate、动态 K=0 prelaunch（本轮三变体再证阻塞）、完整嵌套 event replay、并发长稳/恢复、全局编译/选型性能、完整工业 Gate 均保持开放。

## 与上次日报对比

上次日报为 **11-05（周四）09:15 发布的《AF DSL 进度日报 2026-11-05》**，时点为 HEAD `b8bd773e`（10-30 10:13）、`architecture.md` 最后修改 10-30 11:13（304,956 字节）、工作树 259 项变更、检查时点 09:07（当日 08:17 后无新活动）。

**无变化项**：

- 模块百分比 M0–M7 约 90/95/85/90/95/80/85/80%（**连续第十日持平**，自 10-27 评估以来从未变化）
- HEAD `b8bd773e`（**零新提交，连续第六日**，自 10-30 10:13 未变；`.git/index` 亦未变，无暂存）；`architecture.md` 无新条目（md5 `edc93ae5…ab85` 逐字节一致）
- 完整 host 基线 9,330 = 8,778P/459F/93E；M5 tests/passes 快照 5,864 = 5,042P/445F/377E（均无新全量）
- 发布阻断清单优先级 1–5 无正式新增、无关闭；`industrial_release_ready=false`
- typed FA 主 gate 仍红（`AF_CHANNEL_VALIDITY`）；513×509 bk32/64 尾块 18 份反例保持开放；工作树内 27 处修改（含 L1 elision 与 10-30 CV 融合/大矩阵改动）仍未提交

**变化项**：

1. **活动持续但性质转为纯实验（本轮头条背景）**：11-05 11:56–23:38 共 17,023 个文件（上轮为 11-04 17:49–11-05 08:17 的 9,065 个），连续第二个活动日；本轮 **`src/`/`tests/` 零改动**（上轮有唯一生产改动 `AscendCEmitter.cc` L1 elision），新增 6 个 examples 实验目录 + 4 对设计/状态文档 + 16,974 个 outputs 产物（17 时 Carver 调优搜索单项即 15,755 个）。
2. **对比基准扩展为三侧**：AF vs TileLang vs PyTorch torch.mm（上轮仅 AF vs TL）——确立 torch.mm 为真上限基线；同时证明 TL 当前数字不代表公平上限（Carver 无 950 profile 回退 910A、默认 L0K128 数值失败、TL 大矩阵比 torch.mm 慢 10–12.5×）。
3. **AF 自动选参质量缺陷实证（本轮最重要发现）**：8192³ 自动选 16×32×16 → 1402ms vs 固定 64×64×256 → 25.68ms（同 ELF，54.4×），固定后反超 TL；确认候选漏召回 + relative-work 评分不准——直接命中 M6 auto 选型质量校准。
4. **机制诊断深化至指令/流水级**：MAC ratio AF 15.4% / TL 52.6% / PT 99.8%；AF 逐 K `PIPE_ALL` vs CANN 分层 L1/L0 + L0 ping-pong + 按 buffer 同步；STREAM_K（DP+SK）波次/子任务/workspace 细节；`l1BufferNum=3` 勘误为 L1 双缓冲（[0,2,1,1]，512KiB×2）。下一步方向明确：L1/L0 分层 tile 与跨 K 预取的独立设计及消融。
5. **两项假设被实验推翻/未确认**："动态选参能力=更快"被推翻（AF 逐请求选参累计 348.98μs vs TL 固定串行 90.66μs）；"逐 shape tiling 选参有正收益"未确认（配对差值中位数 -0.007μs、CI 过零、2048² 排序反转）。
6. **全动态 K 阻塞第三轮复现**：三变体 + `af.fill` 探针（M3 拒绝）失败诊断封存——M5 `AF_STORE_VALIDITY` 阻塞证据继续累积，仍无修复。
7. **工作树 259 → 267 项**（27 M 不变 + 232→240 未跟踪；+8 = 4 份设计文档 + 4 份状态文档；另 6 个新 examples 子目录不增加 status 条目）。
8. **今日（11-06）截至 09:08 检查零新活动**；上轮结束后（11-05 08:18–09:07）曾有 50 分钟空窗后活动于 11:56 恢复。

## 备注

- 百分比不能折算为剩余人天或项目总完成率；M5 约 80% 不代表"只剩 20% 的时间"。
- 计时口径区分：动态序列/大矩阵主表为 ACL event span/launch p50（含提交空隙，不含编译/tiling/拷贝）；PyTorch 诊断表为带 profiler 的物理 task duration p50——两者不可混用，也均非纯 device 指令耗时；PMU busy 可重叠，不能相加归因；MAC ratio 是计算流水忙碌比例，不是理论峰值利用率。
- tiling 预算结论仅适用于本候选集/shape 分布/计时口径，且额外成本暴露在关键路径时；不外推为通用数学上限。
- 全部实验产物属"未跟踪的工作树变更"身份，manifest 均 informational；TL/PT 数字引自各实验自身文档与 verification 记录（本轮 AF/TL/PT 同机同轮重测）。
- M5 收口预算已过期三日且连续两轮无 M5 推进，后续日报继续跟踪排期是否重新校准。
- 后续日报沿用既定口径：Git 只读状态（HEAD + index + 工作树计数）+ architecture.md 字节级比对 + 全树（含被忽略目录与 .git）文件级 mtime 核查 + 新增状态文档全文阅读。
