---
title: "AF DSL 进度日报 2026-10-28"
date: 2026-10-28T09:06:36+08:00
categories:
  - AF DSL
tags:
  - AF DSL
  - 进度日报
  - AI自动发布
description: "AF DSL 架构文档（v2，Git HEAD 9d833992）日报：M0–M7 百分比与完整 host 基线（9330 项：8778P/459F/93E）与昨日持平；typed FA 的 AF_CHANNEL_VALIDITY 正式建档为独立 M5/S1 阻塞项，新增 M7 Python 编译驱动（af/compiler.py）与 msprof profiling 等约 30 项切片记录。由大模型定时任务自动生成。"
draft: false
---

> ⚡ 本文由大模型定时任务自动生成。数据来源为远程机器上的 AF DSL 项目架构文档快照（`architecture.md`，v2，更新日期 2026-10-27，Git HEAD `9d833992`）。如与实际代码/证据冲突，以源码及对应 development 记录为准。

## 总体结论

八个模块（M0–M7）均已有实现，**M1–M4 已完成本轮阶段交付**；当前主线是 **M5 集成修复**，之后继续 M6、M7 整体验收。已有 AscendC 编译与真实 NPU 执行证据，但工业发布尚未完成，`industrial_release_ready=false`。

架构文档版本（v2）、更新日期（2026-10-27）与 Git HEAD（`9d833992`）均与昨日持平，尚无 2026-10-28 新条目；模块百分比、完整 host 基线与发布阻断清单全部维持不变。但文档在昨日 10:30 日报之后新增了大量 2026-10-27 当日增量，其中两项结构性变化值得注意：

1. **typed FA 阻塞正式建档**：新增 `docs/development/m5-pass-pipeline/blockers/`，将 typed FA 的 `AF_CHANNEL_VALIDITY` 作为独立 M5/S1 阻塞项管理。单 FA 仍在 `p3_typed_fa.py:83:13` 因 loop-carried prefix 缺少 bounded stable certificate 退出；body-only interval 候选可编译但在 `76 -> 41 -> 40` recipe 链回退。文档明确：该阻塞及 M5 统一 gate 闭合前**不得宣称 M5、M6 或 M7 完成**。
2. **M7 Python 编译驱动切片**：新增 `af/compiler.py`、wheel 运行依赖与独立 E2E/生命周期测试，完成 **JIT → M3/M4 → M5 → M6 → AscendC/bisheng → launcher 的可复现 host 编译闭包**；950PR/CANN 环境实测 E2E `8 passed in 127.52s`、生命周期/隔离/wheel `7 passed`、真实 artifact `5 passed`，`compileall` 退出码 0。文档同时声明该切片不改变 M5 typed FA 阻塞，也不宣称 M7 设备数值、性能或 Release Gate 完成。

## 模块进度总表（M0–M7）

百分比沿用文档 v2 的工程估算（误差约 ±5–10 个百分点，是排期参考，不是验收完成率，也不是测试通过率）：

| 模块 | 进度估算 | 当前状态 | 下一步主要工作 |
|---|---:|---|---|
| **M0** AF Dialect | 约 90% | 基础已提交（原生前置 `8f347c1b`）；本轮依赖修复进行中 | GEMM 逻辑有效域、内容读取、host 投影等共享契约修复及组合回归 |
| **M1** DSL API/JIT | 约 95% | 本轮阶段交付完成（`c3ed1b08`） | backend 声明与真实 provider/Request 认证链的集成 |
| **M2** Graph→DSL | 约 85% | 转换层交付完成（`2e11db10`）；**默认生产入口阻断** | 默认 production 入口能力注册、真实实现认证与设备链闭环 |
| **M3** DSL→AST | 约 90% | 阶段交付完成（`226d09e3`）；**新缺口开放**（recv→view 晚绑定 dtype 误拒） | recv→view 晚绑定 dtype 缺口修复、跨层回归与性能告警处理 |
| **M4** AST→MLIR | 约 95% | 构造层交付完成（`9d833992`，即当前 HEAD）；新增 typed entry export 切片（`IRExport` 复用 `bindHostEntryScalar` 补齐函数入口 block argument 的 typed symbol 导出，公共 API 矩阵 24P） | typed FA 执行域与下游导出合同的最终集成 |
| **M5** Pass 流水线 | 约 80% | **当前集成修复主线**；S1–S6 均已有实现，本轮未提交；typed FA 阻塞已独立建档 | GEMM K0/动态容量、live-scalar 流水、typed FA、统一候选及完整验收 |
| **M6** Tiling 求解 | 约 85% | 基础已提交；trip/执行域候选（`src/tiling/model.py`/`codegen.py`）已实际构建通过，对应 TDD 当前 14P/15F（失败均在进入 M6 模型前命中 `AF_STORE_VALIDITY` 上游 gate） | 新契约消费、完整成本/策略/ABI 验证及最终组合下原生 tiling `<10μs` 门禁 |
| **M7** AscendC 发射/运行 | 约 80% | 已有实现及部分提交（原生 emitter/CMake/CLI 已随 `9d6b6afe` 入库）；新增 Python 编译驱动（`af/compiler.py` 编译闭包）与 msprof profiling（`src/codegen/profiling.py`）等切片 | 完整 NPU 支持矩阵、默认 production 链、wheel/缓存、可靠性和公平性能对标 |

## 测试基线

**最新完整 host 基线：9,330 项 —— 8,778 通过 / 459 失败 / 93 错误 / 0 跳过**（候选79 candidate02-first02 的历史冻结身份；后续局部修复尚未组成新的全绿基线）。该范围排除 `tests/codegen`、`tests/e2e`、`tests/npu`，不能代表全项目验收。**与昨日持平。**

另：正式 tests 最新仅完成 collect-only——排除 codegen/e2e/npu 后 **9,865 个唯一节点、0 收集错误**；这 9,865 项功能尚未执行，不替代上述历史完整结果。

文档记录的定向测试窗口（2026-10-27，各窗口身份独立、不能与基线相加，大部分为昨日日报未报道的新增记录）：

- **M0 切片合计 392P**：schema/effects contract 279P、shared host analysis 108P（含实际 M4 GEMM guard）、content-read interface 4P、finite scope native 1P
- **M5 切片**：标量算术回放契约 226P + 27 subtests（追加 unsigned arithmetic fixture 后 replay oracle 288P）、prefetch/host consumer 119P、GEMM scratch/reduction axes 59P、independent oracle 46P、字面量与谓词契约 40P、metadata size dependency/projection 32P、CLI 诊断等 31P、pre-S1 声明字段准备 23P、参数化 lowering TDD 16P、S6 dynamic GEMM compact-capacity 12P
- **M6**：trip/执行域候选构建通过；对应 TDD 14P/15F（失败均命中 `AF_STORE_VALIDITY` 上游 gate）；完整 `tests/tiling + tests/codegen` 当前 **1,478P / 102F / 200E**
- **M7**：离线 ABI 与程序契约 64P、msprof 物理 profile 50P（不可变 provenance）、vector program contract 32P、布局与谓词设备编译 17P（bisheng 真实编译、每项产生并检查 ELF）、归约与 rank-zero 编译 12P、evidence identity 3P；Python 编译驱动 E2E 8P（127.52s）+ 生命周期/隔离/wheel 7P + artifact 5P（950PR/CANN 实测）
- **M4**：typed entry export 公共 API 矩阵 24P
- **typed FA 阻塞 gate 封存**：主窗口 **4 PASS / 1 PRODUCT_RED / 12 CLI**，兼容窗口 10 PASS / 22 CLI；唯一主窗口产品红仍是 `p3_typed_fa.py:83:13` 稳定证书拒绝
- **S2/S3 有限通道阻塞记录窗口**：39P / 9F（9 项均在 S1 预处理以 `AF_CHANNEL_VALIDITY` 拒绝，未执行 S2/S3 目标断言）
- **schema06 实际后验**：strict139 全部 P/0 CLI，原 C3 三 CLI 均 exit0，但三项为 `L2_ADDITIONAL_CHECK_OBLIGATIONS_UNPROVEN` 产品 F；总计 142 = 139P/3F
- **生产03组合后验（未提交）**：`af-opt -j8` 构建 exit0/18.786s；总 15 节点 14P/1 产品 F/34 CLI；原 FA 仍无实际产物/数值/设备通过证据
- **其它 M5 内部窗口**：正式旧C测试分层迁移 39 唯一节点 31P/8 产品 F/68 CLI（false8 通过、true8 仍败于真实零归约中性化断言）；FA BodySupport P02 1P/4 产品 F/10 CLI（三个最小正例与原 FA 均在 `af.div` 稳定证书拒绝）；零 K identity 资源 TDD 从 6P/15F 增至 21P

其余冻结证据（与昨日一致）：候选79 联合专项 535P/159F；后继86 原62 11P/51F；actual79 除法独立语义 81P；M6 四 fresh 输入 native p99 31/31/71/71 ns（`<10μs` 局部证据）；M7 codegen 556P、query/E2E 17P、GEMM 36P/144 次执行、原 16 组 156 次 launch 重放通过。

## M5 收口与排期

**从评估时点起为 M5 收口预留 3–5 个工作日，可信度偏低**（排期预算而非交付承诺；前提是持续投入、范围冻结、工具链与设备可用，且不再发现新的跨层语义/接口缺口）。**与昨日持平。**

| 工作包 | 当前状态与剩余工作 | 排期预算 |
|---|---|---:|
| GEMM 边界与 FA | 正 K 40P、channel 6P 已通过；K0 构件后继 5P/1 产品 F，f16 reinterpret 布局仍红；动态 L0 容量 production02 已通过（原 12 及 generic6 共 18P）；**原 FA 仍稳定证书拒绝，现已独立建档（blockers/）并封存最新 gate 证据**；通用 J0/oldC、资源及原 FA 数值仍开放 | 1–2 个工作日 |
| live-scalar 流水 | 涉及 M0 执行域认证、SSA carry 及新增 guard 独立安全证明；public16 局部通过，完整 C3 严格语义与观察器 schema 适配仍未闭环 | 1–2 个工作日 |
| 最终集成与回归 | 多项修复仍是隔离候选；测试路由/冷启动、统一生产组合、完整 host/golden/编译预算及设备集成待验证；正式 tests 已 collect-only 9,865 节点（未执行） | 约 1 个工作日 |

新增的阻塞管理规则：typed FA 阻塞及 M5 统一 gate 闭合前不得宣称 M5/M6/M7 完成；允许其他具有独立证据的 M5 子项继续按模块开发和提交。收口方式四条（固定阻断清单记录根因、统一候选同一测试清单报告、单根因半个工作日无实质进展即重审设计定位、取得统一候选首轮完整结果后重估预算）维持昨日报道不变。

## 发布阻断清单

| 优先级 | 阻断/工作项 | 完成判据 |
|---|---|---|
| 1 | M5 最终组合与真实失败修复 | 9330 节点基线的 459F/93E 逐项闭环；实际收集的最终库存全验，保原 M4 与已验证 Core 修复；不能只修前检隐藏后续问题 |
| 2 | 数值/内容读取/视图/typed FA 合同 | 原001 完整 pipeline、typed FA 及独立正负 oracle 到达目标阶段并通过；legacy 来源语义迁移有实证 |
| 3 | 默认 production 能力链 | M2 真实 provider/catalog、Request/Decision/target、跨进程认证正向通过（当前仍拒绝） |
| 4 | 有限 policy 与 resident 完整链 | 同 ELF 动态选择、实际 GM 遍数/状态存活、资源/同步/成本及精度设备验证闭环 |
| 5 | 构建/包/性能/可靠性/发布 | 最终身份独立构建、wheel、缓存、长稳/并发/异常恢复、原生 tiling 与公平 baseline 证据齐全，ReleasePlan 真实聚合通过 |

**条目无新增、无关闭。** 变化在于管理方式：优先级 2 涉及的 typed FA `AF_CHANNEL_VALIDITY` 阻塞已从散落描述升级为独立建档（复现、证据、禁止的绕过方式、下一步实验、关闭判据），并补充 S2/S3 有限通道下游阻塞记录。

## 与上次日报对比

上次日报为**昨日 10:30 发布的《AF DSL 进度日报 2026-10-27》**。架构文档此后又经历多轮更新（全部条目仍标注 2026-10-27，更新日期与 Git HEAD 未变）。

**无变化项**：

- 文档版本 v2、更新日期 2026-10-27、Git HEAD `9d833992`
- 模块进度百分比：M0–M7 约 90/95/85/90/95/80/85/80%
- 完整 host 基线：9,330 = 8,778P + 459F + 93E
- M5 约 80% 估算与 3–5 个工作日收口预算（可信度偏低的定性不变）
- 发布阻断清单（优先级 1–5）：条目无新增、无关闭
- `industrial_release_ready=false`

**变化项（均为昨日 10:30 日报后文档新增的 2026-10-27 记录）**：

1. **typed FA 阻塞正式建档（最重要的结构性变化）**：新增 `docs/development/m5-pass-pipeline/blockers/`，`AF_CHANNEL_VALIDITY` 成为独立 M5/S1 阻塞项；最新 gate 封存结果主窗口 4P/1 产品 F/12 CLI、兼容窗口 10P/22 CLI；并明确"M5 统一 gate 闭合前不得宣称 M5/M6/M7 完成"。
2. **M7 Python 编译驱动落地**：`af/compiler.py` 完成 JIT→M3/M4→M5→M6→AscendC/bisheng→launcher 的可复现 host 编译闭包，950PR/CANN 实测 E2E 8P/生命周期 7P/artifact 5P。
3. **M7 新增 msprof 物理 profile 模块**：`src/codegen/profiling.py` + PMU 128-byte fixture + SQLite/物理聚合测试（不可变 provenance），50P。
4. **约 30 项 M0–M7 定向切片记录**（绝大多数全绿，见测试基线一节）：M0 合计 392P；M5 十余个窗口；M7 六个窗口；M4 typed entry export 24P；M6 trip 候选 14P/15F（失败均为上游 gate）且完整 tiling+codegen 为 1,478P/102F/200E。文档逐项声明"不改变模块百分比或发布结论"。
5. **变更记录表从 2 行增至 7 行**（均"待提交"）：新增 M5 阻塞记录增补（S2/S3 有限通道，39P/9F）、M5-S5 静态 J0 identity 25P、M5-S5 workspace grid 6P、M6 typed true division 193P、M7 target foundation 34P 共五个切片条目。
6. **schema 系列推进**：昨日报道"schema04 静审后独立测试"；实际 schema04 首正控 HARNESS_STOP（gate 重复分派 `_task`），schema05 strict139 95P/1 HARNESS，schema06 完成实际后验 139P/3 产品 F（替代"待执行"状态）。
7. **request-snapshot 窗口实际结果**：昨日报道为计划中的"8 节点/最多 14 CLI"；实际 3 节点 2P/1F/3 CLI，第三例因 raw 观察器将 vector 几何除法误计请求触发 HARNESS_STOP，原窗冻结，准备仅修 producer 身份的 tests-only 后继。
8. **Trip 候选与正式 FE 合入**：完整条件域 Trip 候选独立 fresh 构建后 41 项 39P/2 产品 F/72 CLI/0 HARNESS；FE 已合正式（`1bae44ca`）；typed-byte4hunk 与 guard 幂等 3hunk 亦精确合入正式 FE（`c9bc4a9e`），新正式组合未构建/测试。
9. **正式组合静审**：独立纯静审 43 项通过；原79 中 16 个合法表 inventory 负控恢复 91 行（独立 20 项静审通过，未功能执行）；正式 149 增量 L2 carry 集成 11 hunk（独立 16 项静审通过）。
10. **正式 tests collect-only 完成**：9,865 唯一节点、0 收集错误（功能尚未执行）。
11. **M5 pass-contract checkpoint**：当前可编译的 `src/dialect/` 与 `src/passes/` 候选已保存为一个 checkpoint 提交（保留未解决的 typed FA 产品红）；文档与候选状态同步集中固化。

## 备注

- 百分比不能折算为剩余人天或项目总完成率；M5 约 80% 不代表"只剩 20% 的时间"。
- 文档区分"已提交代码、当前工作树、隔离候选"三种身份；本轮新增切片多数以"本次提交"措辞记录为已提交，但变更记录表 7 行条目仍为"待提交"状态。按 AGENTS.md 硬性原则 14，任何 commit 都必须在同一次提交中包含 architecture.md 的实际更新。
- typed FA 阻塞建档的意义：把"未知证明不能当成成功"固化为可跟踪的关闭判据，防止 body-only interval 候选的局部可编译被误报为通过。
