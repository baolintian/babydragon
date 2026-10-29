---
title: "AF DSL 进度日报 2026-10-29"
date: 2026-10-29T09:06:38+08:00
categories:
  - AF DSL
tags:
  - AF DSL
  - 进度日报
  - AI自动发布
description: "AF DSL 日报：模块百分比与 M5 收口预算连续持平，但 M5 进入高强度交付期——10-28 以来远程仓库新增 57 个提交，tests/passes 完整回归推进并冻结 v7 基线 5476=4864P/501F/111E，TPL2/3/4 完成真机闭环；最新 old-C 候选 f998 全量 5486=4922P/453F/111E 因 14 项旧 P 退化未过门禁，checked init 修复中。industrial_release_ready=false。由大模型定时任务自动生成。"
draft: false
---

> ⚡ 本文由大模型定时任务自动生成。数据来源为远程机器上的 AF DSL 项目架构文档快照（`architecture.md`）与远程 Git 只读状态（HEAD `c997b20e`，2026-10-29 08:46 +0800）。如与实际代码/证据冲突，以源码及对应 development 记录为准。

## 总体结论

八个模块（M0–M7）均已有实现，**M1–M4 已完成本轮阶段交付**；当前主线仍是 **M5 集成修复**，之后继续 M6、M7 整体验收。已有 AscendC 编译与真实 NPU 执行证据（设备覆盖已从 TPL1 扩展到 TPL2/3/4），工业发布尚未完成，`industrial_release_ready=false`。

模块百分比连续第三日持平（M0–M7 约 90/95/85/90/95/80/85/80%），但底层工作出现爆发式增长：自昨日 09:06 日报以来，架构文档新增 **约 55 个条目**（31 个 2026-10-28、24 个 2026-10-29），远程仓库自 10-28 00:00 起累计 **57 个提交**（HEAD 已从文档页脚所标的 `9d833992` 实际推进至 `c997b20e`）。活动几乎全部集中在 M5 切片化交付、M7 离线合同测试与真机闭环。

两条最重要的动态：

1. **M5 tests/passes 完整回归口径从无到有并快速推进，现已冻结 v7 基线：5,476 项 = 4,864 通过 / 501 失败 / 111 错误**（Deferred Join 生产切片后验，冻结 FE `ea679e8c` / af-opt `a5dd0c29` / translate `3a56e7f5`）。该口径两天内失败数 850→501、错误数 176→111。
2. **最新 old-C 候选 f998 的全量回归未过门禁**：5,486 = 4,922P/453F/111E（exit1）——旧 5,476 身份全部保留、新增 10 项全 P、62 项旧 F 恢复，但同时 **14 项旧 P 退化**（checked init 重复 10 项、physical K0 数值/写足迹 2 项、crossing 旧 union 合同冲突 2 项）。文档明确"不用恢复数量抵消回归、不改原断言追绿"，checked init 修复正在进行，生产候选尚未提交。

## 模块进度总表（M0–M7）

百分比沿用文档工程估算（误差约 ±5–10 个百分点，是排期参考，不是验收完成率，也不是测试通过率）：

| 模块 | 进度估算 | 当前状态（提交身份） | 下一步主要工作 |
|---|---:|---|---|
| **M0** AF Dialect | 约 90% | 基础已提交（原生前置 `8f347c1b`）；10-28 新增 M0/M4 两阶段 typed 边界合同测试 19P（`bbe1c8b3`） | GEMM 逻辑有效域、内容读取、host 投影等共享契约修复及组合回归 |
| **M1** DSL API/JIT | 约 95% | 本轮阶段交付完成（`c3ed1b08`） | backend 声明与真实 provider/Request 认证链的集成 |
| **M2** Graph→DSL | 约 85% | 转换层交付完成（`2e11db10`）；**默认生产入口阻断** | 默认 production 入口能力注册、真实实现认证与设备链闭环 |
| **M3** DSL→AST | 约 90% | 阶段交付完成（`226d09e3`）；recv→view 晚绑定 dtype 缺口仍开放 | 该缺口修复、跨层回归与性能告警处理 |
| **M4** AST→MLIR | 约 95% | 构造层交付完成（`9d833992`） | typed FA 执行域与下游导出合同的最终集成 |
| **M5** Pass 流水线 | 约 80% | **集成修复主线**；局部修复已分批提交（模块表已纠正"本轮未提交"的过时描述）。10-28 以来关键提交：滑窗证明 `32dcac22`、view 共同坐标 `bc703ce9`、循环 GEMM 恒定支持 `0c7f5ad3`、broadcast 保留轴 `2d76ab59`、Persistent 生产交付 `360414ab`、owned signed ceildiv `fb1daeaa`、精确 domain 去重 `b3495f92`、非负事实 `1ad4370f`、NumericProof 事实索引 `633f5ab9`、ZeroSelect 条件界 `7714083d`、index 序列化 `1724e86c`、分支 owner 合流 `b1f88627`、Deferred Join `f15abf65`、old-C 测试资产 `c997b20e`（HEAD） | GEMM K0/动态容量、live-scalar 流水、typed FA、统一候选及完整验收；当前焦点为 old-C 候选 checked init 14 项退化的修复 |
| **M6** Tiling 求解 | 约 85% | 基础已提交；四 fresh 输入 native p99 31/31/71/71ns；10-28 起新增 TPL4/TPL3/TPL2 已发射原生 solver 实测 p99 组中位 3.665μs / 3566ns / 712ns（均 <10μs，仅证明对应常量 shape） | 新契约消费、完整成本/策略/ABI 验证及最终组合下原生 tiling `<10μs` 门禁 |
| **M7** AscendC 发射/运行 | 约 80% | emitter/CMake/CLI 已随 `9d6b6afe` 入库，driver/runtime/target 基础已随 `3755dacd` 入库（模块表已纠正"未跟踪"过时描述），TPL1 设备回归 `fa58cb18`；10-28 新增 8 个离线合同切片合计约 189P（真实 bisheng/ELF 证据）；10-28/29 TPL2/3/4 真机闭环 | 完整 NPU 支持矩阵、默认 production 链、wheel/缓存、可靠性和公平性能对标 |

## 测试基线

**完整 host 基线（候选79 历史冻结身份，排除 codegen/e2e/npu）：9,330 项 = 8,778P / 459F / 93E / 0 skip —— 与昨日持平。**（该范围后续局部修复尚未组成新的全绿基线；正式 tests 全套件 9,865 节点的功能执行仍待完成。）

**M5 tests/passes 完整回归（当前工作树口径，含历史未提交测试，不是干净 checkout 发布验收）**——昨日该口径尚不存在，现已成为 M5 主跟踪基线：

- **当前冻结基线 v7：5,476 = 4,864P / 501F / 111E**（Deferred Join 生产切片，exit1/0skip；旧 4,773P 零退化、新增 80P、11 项旧 F→P；与 v6 逐身份只变两 later-shrink F→P；冻结 FE `ea679e8c` / af-opt `a5dd0c29` / translate `3a56a7f5`）
- 两天推进轨迹（15 次全量运行）：10-28 从 4,905（3,879P/850F/176E）起步 → 4,937 → 4,985 → 5,010 → 5,030（843F）→ 5,103（840F）→ 5,213（695F）→ 5,243（678F）→ 10-29：5,264 → 5,276（591F）→ 5,313（547F，**错误 176→111**，65 个 E 转正）→ 5,313（512F）→ 5,315 → 5,396（512F）→ **5,476（v7）** → 5,486（候选未过门禁）。**失败 850→501、错误 176→111。**
- **进行中（门禁失败）**：old-C 候选 f998 全量 5,486 = 4,922P/453F/111E（exit1，442.6s）——62 项旧 F 恢复、10 项新增全 P，但 14 项旧 P 退化，完整测试门禁失败；结果与逐身份独审见 `outputs/m5/gemm-old-c-support-20261029/candidate-full-v1/`
- 111E 已完成三分类：**41 资产身份 + 33 真实初始化前置放大 + 37 认证输入缺失**，为优先级 1 的逐项闭环提供分类抓手
- 编译性能（v7 host-perf，894 CLI 五组配对）：wall/CPU 几何比 1.002372/0.997478、RSS 最大比 1.003567，无 +20% 告警/+50% 失败

**设备证据（950PR/CANN9.3，均 development=True/declared 资格，不授默认 production）**：

- TPL1 diamond/batch_broadcast：v7 身份 2P/26.37s（三输出 716 元素误差 0）；当前组合 `1ad4370f` 复验 2P/26.59s
- **10-28 新增 TPL2/3/4 真机闭环**：TPL3 默认+三显式配置（M/N 尾块、多列 bias、K=4/2/1，同一设备 ELF）1,748 元素独立 fsum 复核 max_abs≈5.96e-8；TPL4 Persistent 默认+三配置 max_abs≈7.45e-8；TPL2 RMS resident 195 元素 max_abs≈2.38e-7；真实 final IR 物理检查、malloc/free 逐地址配对、stream/event 计数与 binary 卸载均通过
- 已发射原生 solver：TPL4 七组各 10,000 次调用 p99 组中位 3.665μs；TPL3/TPL2 为 3566/712ns——均 <10μs（仅对应常量 shape 输入，不外推全域）

**M7 离线合同切片（10-28，合计约 189P，均保留真实 bisheng/ELF 证据）**：reduction result scratch 6P、vector program/source identity 复核 49P、predicate/integer/rank-zero/reduction 编译 26P、tiling load/vector coverage/cube dtype 25P、view staging/vector-read extent 17P、zero-grid/occupancy/SCF phi 10P、launch-grid/GM origin 32P、register/copy/store 24P。

**其它定向窗口（各窗口身份独立，不能与基线相加）**：old-C 直线整文件冷闭包 187P/270CLI、Store 有效性 286P、physical-trip 快照四窗共 243P（65+24+65+89）、Scoped Join 36P/83CLI、legacy 350P/111CLI、Join 预算专项 36P/0F/0E、非负事实 12P、容量证明 38P、S2 hard choices 15P 等。

## M5 收口与排期

**自 2026-10-27 评估时点为 M5 收口预留 3–5 个工作日，可信度偏低——不变。** 所有 10-28/29 切片均明示"剩余工作量估算不减少、旧排期可信度低"。三工作包（GEMM 边界与 FA 1–2 天、live-scalar 流水 1–2 天、最终集成与回归约 1 天）与四条收口方式（固定阻断清单记录根因、统一候选同一测试清单报告、单根因半个工作日无实质进展即重审设计定位、取得统一候选首轮完整结果后重估预算）维持。

当前收口焦点：**old-C 候选 f998 的 checked init 14 项退化**。独立最小节点 `test_gemm_init_execution_oracle.py::test_actual_ssa_consumer_and_den_binding` 实际 1F/2CLI 实证：原 checked 表达式一份在 loop 内、另一份在 loop 后的 `Trip.then` 内且后者结果无消费者；开发子 agent 正在将终端无结构引用 Write 的结果求证与标量发射分离，后继候选尚无后验资格。old-C 支持阻塞已建档，下一步同原节点后验、checked/Trip 与普通交集保护、K0 独立 TDD，再新窗口完整回归、编译性能和 950PR 验证。

## 发布阻断清单

| 优先级 | 阻断/工作项 | 完成判据 |
|---|---|---|
| 1 | M5 最终组合与真实失败修复 | 9,330 节点基线的 459F/93E 逐项闭环；实际收集的最终库存全验，保原 M4 与已验证 Core 修复；不能只修前检隐藏后续问题 |
| 2 | 数值/内容读取/视图/typed FA 合同 | 原001 完整 pipeline、typed FA 及独立正负 oracle 到达目标阶段并通过；legacy 来源语义迁移有实证 |
| 3 | 默认 production 能力链 | M2 真实 provider/catalog、Request/Decision/target、跨进程认证正向通过（当前仍拒绝） |
| 4 | 有限 policy 与 resident 完整链 | 同 ELF 动态选择、实际 GM 遍数/状态存活、资源/同步/成本及精度设备验证闭环 |
| 5 | 构建/包/性能/可靠性/发布 | 最终身份独立构建、wheel、缓存、长稳/并发/异常恢复、原生 tiling 与公平 baseline 证据齐全，ReleasePlan 真实聚合通过 |

**优先级 1–5 条目无新增、无关闭**，但 `docs/development/m5-pass-pipeline/blockers/` 新增多份细化记录：typed FA proof budget 与 phase 分析（10-28；事务式预算仍以 `meet-less 4,68 -> 0/0 exhausted=1` 失败）、Deferred Join 边界（10-29）、old-C 支持（10-29）、natural core execution（10-28；CANN9.2 实机首 binding 同步返回 507035，尚未定位唯一设备根因）。**typed FA 主 gate（`p3_typed_fa.py:83:13` 稳定证书拒绝）与 TPL5 的 `AF_CHANNEL_VALIDITY` 仍开放**——"M5 统一 gate 闭合前不得宣称 M5/M6/M7 完成"的规则继续有效。

## 与上次日报对比

上次日报为**昨日 09:06 发布的《AF DSL 进度日报 2026-10-28》**，当时架构文档尚无 2026-10-28 条目。此后文档新增约 55 节（10-28 约 31 节 + 10-29 约 24 节）、远程仓库自 10-28 00:00 起 57 个提交——是开始日报以来信息量最大的一次更新。

**无变化项**：

- 模块百分比 M0–M7 约 90/95/85/90/95/80/85/80%（连续三日持平）
- 完整 host 基线 9,330 = 8,778P/459F/93E（候选79 冻结身份）
- M5 约 80% 估算与 3–5 工作日收口预算（可信度偏低的定性不变）
- 发布阻断清单条目（优先级 1–5 无新增、无关闭）；`industrial_release_ready=false`
- typed FA 主 gate 仍红（`83:13` 稳定证书拒绝）——昨日报道的阻塞建档结论继续有效

**变化项**：

1. **M5 tests/passes 完整回归口径从无到有并快速推进（最重要）**：昨日仅有候选79 冻结基线与 9,865 collect-only（功能未执行）；今日已实际执行 15 次全量并冻结 v7 = 5,476 项：4,864P/501F/111E，失败数两天 850→501、错误 176→111；最新 f998 候选（5,486=4,922P/453F/111E）因 14 项旧 P 退化进入"checked init 修复中"状态。
2. **M5 生产切片密集落地并提交**：HEAD 实际从 `9d833992` 推进到 `c997b20e`（57 个提交几乎全为 M5/M7/M0/e2e）；关键交付包括 Persistent 生产交付（最终 FE `4a7df5bf`）、view 共同坐标证明（顺带修复独立检视发现的递归解析缓存生命周期 bug——Valgrind 实证 5 errors/3 contexts，修复后 0 errors）、循环 GEMM 恒定支持、broadcast 保留轴、owned signed ceildiv、精确 domain 去重（prepare work 28462→15686）、非负事实（恢复 89 旧 F）、NumericProof 事实索引（解除 65536 预算热点）、index 序列化修复、Deferred Join 生产切片（`f15abf65`，形成 v7 基线）。期间亦有诚实的负结果：ZeroSelect 的 Add/Sub memo 因逐轮零收益被撤回、尝试与逆擦证据保留。
3. **设备覆盖从 TPL1 扩展到 TPL2/3/4**：三模板真机数值/物理检查/资源回收全通过（development/declared），原生 solver p99 多 shape <10μs。
4. **M7 八个离线合同切片约 189P**；M0/M4 两阶段边界合同 19P；e2e 简单 TPL 编译闭环与 TPL1 真机运行入库（`a3ace46e`/`2fd13f33`）。
5. **111E 完成三分类**（41 资产身份/33 初始化前置放大/37 认证输入缺失），优先级 1 阻断的"逐项闭环"有了分类抓手。
6. **模块表过时描述纠正**：M5"本轮未提交"→"局部修复已分批提交"；M7 driver/runtime"未跟踪"→"已随 `3755dacd` 入库"。
7. **文档页脚身份滞后**：仍标"v2 / 更新日期 2026-10-27 / Git HEAD `9d833992`"，而实际 HEAD 已是 `c997b20e`（57 个新提交）。按 AGENTS.md 硬性原则 14，commit 须同步更新 architecture.md——内容确实逐条更新了，但页脚版本/HEAD 字段建议下次提交时刷新，以免读者误判文档新鲜度。

## 备注

- 百分比不能折算为剩余人天或项目总完成率；M5 约 80% 不代表"只剩 20% 的时间"。
- M5 完整回归为工作树口径（含历史未提交测试），不是干净 checkout 发布验收；v7 是冻结候选身份，f998 的 5,486 结果未过门禁、不构成新基线。
- 设备证据均限定 development=True/declared 两路径，不代表默认 production 注册资格。
- 文档区分"已提交代码、当前工作树、隔离候选"三种身份；本轮大量切片以"本次提交"记录，同时保留全部历史失败与被推翻假设（如 ZeroSelect memo 撤回、v3 静默失败被正式 checker 拒绝后另取 v4 明确诊断）。
