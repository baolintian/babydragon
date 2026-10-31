---
title: "AF DSL 进度日报 2026-10-31"
date: 2026-10-31T09:00:00+08:00
categories:
  - AF DSL
tags:
  - AF DSL
  - 进度日报
  - AI自动发布
description: "AF DSL 日报：模块百分比连续第五日持平（M0–M7 约 90/95/85/90/95/80/85/80%），但技术主线实质突破——TPL4 分块 GEMM 正值契约链（schema v2 joint_positive）设计→M0→M5→M6→M7 五连提交（d0c56d24/cfd5e7fa/1989493d/c45887ab/b8bd773e）并完成 AscendC→950PR 真机全流程闭环（8 次执行零失败元素、native p95/p99 <10μs）；另 L0C→UB CV 融合支持域与 (2049,2053,513) 大矩阵两轮验证在工作树完成但未提交（236 项变更待整合）。industrial_release_ready=false。由大模型定时任务自动生成。"
draft: false
---

> ⚡ 本文由大模型定时任务自动生成。数据来源为远程机器上的 AF DSL 项目架构文档快照（`architecture.md`，最后修改 2026-10-30 11:13 +0800）与远程 Git 只读状态（HEAD `b8bd773e`，2026-10-30 10:13 +0800；工作树另有 236 项未提交变更）。如与实际代码/证据冲突，以源码及对应 development 记录为准。

## 总体结论

八个模块（M0–M7）均已有实现，**M1–M4 已完成本轮阶段交付**；主线仍是 **M5 集成修复**，之后 M6、M7 整体验收。模块百分比连续第五日持平（M0–M7 约 90/95/85/90/95/80/85/80%），`industrial_release_ready=false`。

自上次日报（10-30 09:10，HEAD `d5e52a60`）以来，远程仓库新增 **5 个提交**，全部集中在 10-30 上午 10:02–10:13 的 11 分钟内，构成 **TPL4 分块 GEMM 正值契约链（schema v2 `joint_positive`）** 的逐模块落地：设计 `d0c56d24` → M0 `cfd5e7fa` → M5 `1989493d` → M6 `c45887ab` → M7 `b8bd773e`。此外，**两轮大验证已在工作树完成并记录于架构文档（条目日期均为 10-30）但未执行 commit**——TPL4 大矩阵 (2049,2053,513) 核内多组/多K轮验证、真实 L0C→UB CV 融合支持域闭环；工作树共 236 项变更待整合提交。10-31 当天（截至 09:03）无新提交、无新架构条目，远程最后一次活动为 10-30 11:13。

本轮最重要的三件事：

1. **TPL4 正值契约 M0→M5→M6→M7 全线提交并完成 950PR 真机闭环**（本轮头条）。schema v2 `joint_positive=true` 表示**实际每次 GEMM 的 J>=1 证明**——不能以物理 full K 或分块配置替代，device assert 不授予 host 证书。M7 严格消费 v2 正值证书（v1 及 physical K=0 保持在 prepare/DeviceProgram/launch 前统一拒绝）。真机后验：原文 (19,23,13) 与 fresh GraphView (137,153,77) 各 auto+3fixed，共 **8 次完整 M3→M5→M6→M7→AscendC/bisheng ELF→ACL 执行**，float64 独立 oracle（atol 2e-5/rtol 2e-4）失败元素均为 0（max abs 4.82649e-8 / 6.41580e-7）；每个 native solver 5 组×1000 次，最差组 p95/p99 = 3655/3676ns 与 3936/3976ns，**均 <10μs**；含建议域外 k_l1=7、多 K 尾块/MN 尾块和不满 group，逐 case 同 ELF。
2. **真实 L0C→UB CV 融合支持域闭环（工作树，未提交）**：数据路径 Cube L0C→双 AIV UB→Vector epilogue→GM 输出，**移除该通道 GM workspace**；定位旧实验误路由 L1 的根因（公开 `CFG_ROW_MAJOR.isToUB=false`），显式 true 独立 API 实验 15 组、**1,013,760 元素精确匹配**；S5 修复先 Cube handle 合并导致的 Vector 读 guard 丢失。三 shape 真实设备对拍（fixed 5 组×3 次）**22 份归档数值记录失败元素 0**（maxabs 4.157414e-6）。资格限定：只授 FP32、1 Cube:2 AIV 通道。
3. **TPL4 大矩阵核内多组/多K轮验证（工作树，未提交）**：FP32 (2049,2053,513) 真实 GraphView→DSL→MLIR→AscendC→950PR 回归；M6 修复静态 wave 空区间在 guard 前折叠除零；auto/fixed 同 ELF 两次 float64 oracle 零失败（max abs 4.15742e-6）；大矩阵固定 128×128、K64/group4，**289 tile/73 group、每核 2–3 组、每 tile 9 轮 K**。

## 模块进度总表（M0–M7）

百分比沿用文档工程估算（误差约 ±5–10 个百分点，是排期参考，不是验收完成率，也不是测试通过率）：

| 模块 | 进度估算 | 当前状态（提交身份） | 下一步主要工作 |
|---|---:|---|---|
| **M0** AF Dialect | 约 90% | 基础已提交；**本轮新增 v2 正值契约 verifier `cfd5e7fa`**：AFRegistry 接受 schema v2 `joint_positive=true`，严格检查三 joint 来源字段互斥/类型/版本；v1 常量/scalar 兼容，错误类型字段不被 positive 掩盖；M0 verifier 11 项全绿（独立组合 18P/0.70s）。CV 融合轮扩展的 verifier 使用域在工作树未提交 | GEMM 逻辑有效域、内容读取、host 投影等共享契约修复及组合回归 |
| **M1** DSL API/JIT | 约 95% | 阶段交付完成（`c3ed1b08`），本轮无新提交 | backend 声明与真实 provider/Request 认证链的集成 |
| **M2** Graph→DSL | 约 85% | 转换层交付完成（`2e11db10`）；**默认生产入口阻断**，本轮无新提交 | 默认 production 入口能力注册、真实实现认证与设备链闭环 |
| **M3** DSL→AST | 约 90% | 阶段交付完成（`226d09e3`）；recv→view 晚绑定 dtype 缺口仍开放；本轮无新提交 | 该缺口修复、跨层回归与性能告警处理 |
| **M4** AST→MLIR | 约 95% | 构造层交付完成（`9d833992`），本轮无新提交；v2 契约设计明确 TilingData/native ABI 与 M1–M4 数学接口不变、无旧 Autofuse/SuperKernel 依赖 | typed FA 执行域与下游导出合同的最终集成 |
| **M5** Pass 流水线 | 约 80% | **actual-domain 正值证明已提交 `1989493d`**：`queryGemmReduction` 在实际 shared recipe 上用有预算 NumericProof 证明 J>=1；任一 GEMM 或 A/B 公开来源未知时不发布整个 module contract（防 partial table 掩盖缺项）；每次清除重算旧 marker。独立组合 22P/0.64s；零 K 拒绝/来源旧套件 17P17F（13 项零 K 拒绝 + 4 项正 K/oracle 通过；17 项旧零 K 中性/empty 期望被统一拒绝规定阻断）。CV 融合轮的 S5 通道/共享 UB/实现摘要（含 Vector 读 guard 修复）在工作树未提交 | typed-FA 正 K 证明（`AF_CHANNEL_VALIDITY` 首红保持 open）、deferred Join P2 frontier、old-C first/Task/monotone/86 族真实首红、动态 K=0 真实 artifact 迁移 |
| **M6** Tiling 求解 | 约 85% | **v2 契约透传已提交 `c45887ab`**：typed model 严格消费 schema v1/v2，positive 只接受 v2 bool true，三来源按字段存在计数，畸形/冲突/越界/未知来源拒绝；metadata 保留版本及证书，v1 字段集合保持。TDD 首红 2F24P → 独立最终 **38P/4.29s**。大矩阵静态 wave 除零修复 + CV 融合实际契约/独立物理检查在工作树未提交 | 完整成本/策略/ABI 验证及最终组合下原生 tiling `<10μs` 门禁（本轮 TPL4 native 采样 p95/p99 已 <10μs，但完整门禁未关闭；auto 选 k_l1=1 的选型质量待校准） |
| **M7** AscendC 发射/运行 | 约 80% | **TPL4 全流程闭环已提交 `b8bd773e`**：独立 host 后验 27P/1.37s + 真实 950PR 2P/27.10s；8 次完整链路执行零失败元素、native p95/p99 <10μs；最终只读核对 source/test/native snapshot 与设备采集一致。公开 Fixpipe UB 能力探测/helper/emitter（CV 融合轮）在工作树未提交 | 完整 NPU 支持矩阵、默认 production 链、wheel/缓存、可靠性与公平性能对标；动态 K=0 真实 artifact/950PR 义务开放 |

## 测试基线

**完整 host 基线（候选79 冻结身份，排除 codegen/e2e/npu）：9,330 项 = 8,778P / 459F / 93E / 0 skip —— 与昨日持平，本轮无新全量。**

**M5 tests/passes 完整回归（工作树口径，非干净 checkout 验收）**：最近一次全量快照仍为 **5,864 = 5,042P / 445F / 377E**（K0 政策重置后），本轮未重跑全量。

**本轮新增定向窗口（TPL4 正值契约链 / 大矩阵 / CV 融合，各窗口身份独立）**：

- 设计首红（v2 契约，`d0c56d24` 时点）：contract 8F/2P（0.70s）、原文/fresh NPU 流程 2F/19.94s（均未到 launch）——随后被五个提交逐模块修复
- M0/M5 v2 postcheck：18P/0.70s（M0 verifier 11 + M5 producer 4 + direct metadata 3）
- M5 独立组合 22P/0.64s；零 K 拒绝/来源旧套件 17P17F
- M6 独立最终 38P/4.29s（TDD 首红 2F24P）
- M7 独立 host 27P/1.37s；真实 950PR 2P/27.10s
- TPL4 大矩阵：候选三套 76P/10.73s、大矩阵 1P/19.70s、原 TPL4 回归 2P/28.15s；`git diff --check` 通过
- CV 融合：冻结专项 24P/97.78s、provider 缓存真实边界 8P/36.23s；manifest 3 项校验（路径/环境/SHA）PASS，source/native/test snapshot 与当前工作树一致
- **扩展旧回归 70P / 1F / 26E**：失败与错误均由旧 matmul_epilogue 的 S1 `AF_CHANNEL_VALIDITY` 证书阻断（只跑 S1 亦复现），未进入本轮修改阶段，失败证据 `legacy-s1/` 保持，未改 fixture/门禁

**设备证据（950PR/CANN，均 development=True/declared 资格）**：三 shape（19×23×13 / 137×153×77 / 2049×2053×513）各 auto+3fixed 同 ELF，含 k_l1=7 与 fixed 5 组×3 次真实设备对拍，22 份归档数值记录失败元素 0（maxabs 4.157414e-6）；native solver 5×1000 采样最差组 p95 = 3506/3765/3625ns。真实 UB arena 132,608B、通道 slot 33,024B（含读 guard）；单有界 loop 固定事件回放及提前 free/缺 free/漏 drain/错地址负控通过。**大矩阵 GM/direct 设备时间随组波动，不据此宣称全局稳定加速。**

## M5 收口与排期

**自 2026-10-27 评估时点为 M5 收口预留 3–5 个工作日（约 10-30～11-03 到期），可信度偏低——判断不变。** 今日为 10-31（周六），按工作日口径窗口只剩约 1–2 天（11-02/11-03），M5 仍约 80%，与"可信度偏低"的预判一致。三工作包（GEMM 边界与 FA、live-scalar 流水、最终集成与回归）与四条收口方式维持；本轮 TPL4 正值契约链属于 GEMM 边界工作包的实质推进，但各条目均明示"整体剩余工作量估算不变、不把本轮局部功能完成扩写为整个项目完成"。

## 发布阻断清单

| 优先级 | 阻断/工作项 | 完成判据 |
|---|---|---|
| 1 | M5 最终组合与真实失败修复 | 9,330 节点基线的 459F/93E 逐项闭环；K0 政策后的 445F/377E 按独审分类逐项处置；实际收集的最终库存全验 |
| 2 | 数值/内容读取/视图/typed FA 合同 | 原001 完整 pipeline、typed FA 及独立正负 oracle 到达目标阶段并通过；typed-FA 正 K 证明需先建立可传递 precondition 与数学证书 |
| 3 | 默认 production 能力链 | M2 真实 provider/catalog、Request/Decision/target、跨进程认证正向通过（当前仍拒绝） |
| 4 | 有限 policy 与 resident 完整链 | 同 ELF 动态选择、实际 GM 遍数/状态存活、资源/同步/成本及精度设备验证闭环 |
| 5 | 构建/包/性能/可靠性/发布 | 最终身份独立构建、wheel、缓存、长稳/并发/异常恢复、原生 tiling 与公平 baseline 证据齐全，ReleasePlan 真实聚合通过 |

**优先级 1–5 条目本身无新增、无关闭**；`industrial_release_ready=false`。本轮条目明确点名的开放项：完整嵌套 event replay、并发长稳/恢复、全局编译/选型性能、auto 选 k_l1=1 的选型质量、完整工业 Gate 与竞品性能对标。**typed FA 主 gate（`AF_CHANNEL_VALIDITY` 稳定证书拒绝）与 TPL5 仍开放**——扩展旧回归的 1F/26E 即其影响面的新证据。

## 与上次日报对比

上次日报为**昨日 09:10 发布的《AF DSL 进度日报 2026-10-30》**，时点为 HEAD `d5e52a60`（10-30 05:47）。此后 5 个提交（10-30 10:02–10:13），HEAD 推进至 `b8bd773e`；另有两轮工作树验证未提交。

**无变化项**：

- 模块百分比 M0–M7 约 90/95/85/90/95/80/85/80%（**连续第五日持平**）
- 完整 host 基线 9,330 = 8,778P/459F/93E；M5 tests/passes 快照 5,864 = 5,042P/445F/377E（均无新全量重跑）
- M5 约 80% 估算与 3–5 工作日收口预算（可信度偏低的定性不变）
- 发布阻断清单优先级 1–5 无新增、无关闭；`industrial_release_ready=false`
- typed FA 主 gate 仍红（`AF_CHANNEL_VALIDITY` 稳定证书拒绝）

**变化项**：

1. **TPL4 正值契约从 schema v1 升级到 v2 并真机闭环**（本轮头条）：昨日报道的 v1 链（`8c77352b`/`6b88fcbd`/`ee3174f6`）覆盖"静态契约透传 + host preflight"；本轮 v2 `joint_positive` 把证书升级为"实际每次 GEMM 的 J>=1 证明"（不能以物理 full K 或分块配置替代），并以五连提交打通 M3→M5→M6→M7→AscendC/bisheng→ACL 真机全流程（8 次执行零失败元素、native p95/p99 <10μs）。设计条目记录的起点首阻塞 M6 `AF_M6_CONTRACT_MISSING` 已被该链关闭。
2. **CV 融合（L0C→UB）首次出现在日报**：新能力方向——Cube L0C→双 AIV UB→Vector epilogue→GM 输出、移除该通道 GM workspace；定位 `CFG_ROW_MAJOR.isToUB=false` 误路由 L1 根因；15 组/1,013,760 元素精确匹配 + 真机对拍零失败。**未提交**（feat-direct-cv-fusion 四列全绿并关闭，待整合提交）。
3. **大矩阵 (2049,2053,513) 验证（未提交）**：每 tile 9 轮 K、289 tile/73 group 的核内多组/多K轮真实性验证；M6 静态 wave 空区间除零修复。
4. **扩展旧回归新增 1F/26E 证据**：均归因旧 matmul_epilogue 的 S1 `AF_CHANNEL_VALIDITY` 证书阻断——typed FA 主 gate 红在旧回归中的具体体现，失败证据冻结于 `legacy-s1/`，未改 fixture/门禁追绿。
5. **工作树待整合量大**：236 项变更（含 `src/passes`、`src/tiling`、`src/codegen` 生产文件与 feat-direct-cv-fusion 设计/状态、大矩阵验证文档），下一步大概率是一次较大的整合提交；10-31 当天（截至 09:03）尚无新提交，远程最后活动为 10-30 11:13。

## 备注

- 百分比不能折算为剩余人天或项目总完成率；M5 约 80% 不代表"只剩 20% 的时间"。
- 两轮未提交验证均为工作树身份，最终以 commit 后的冻结身份为准；CV 融合资格限定 FP32、1 Cube:2 AIV 通道，"不把本轮局部功能完成扩写为整个项目完成"。
- 大矩阵 GM/direct 设备时间随组波动，不据此宣称全局稳定加速；完整嵌套 event replay、并发长稳/恢复、全局编译/选型性能待补。
- 动态 K=0 preflight 仍为 host adapter 证据，真实动态 artifact 完整链路（`AF_STORE_VALIDITY` 前置阻塞）与 950PR ACL 动态数值/资源计数义务保持开放。
- 设备证据均限定 development=True/declared 两路径，不代表默认 production 注册资格。
- 文档区分"已提交代码、当前工作树、隔离候选"三种身份；本轮旧回归的 1F/26E 如实保留，未追绿。
