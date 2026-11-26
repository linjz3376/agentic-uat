# 执行期夹具：Run、Run-Case 与当前判定（虚构）

> 用途：验证执行粒度和聚合规则，不承载行业维度设计。所有 AC/Case 均为同环境 `ENV-E1`、同判据哈希 `H1` 的冻结计划；预期报告为 FINAL-WITH-GAPS，不宣称产品通过或领域验证完整。

## Run 主表（一行一次执行，Run-ID 唯一）

| Run-ID | run_kind | reverify_of | environment | criteriaHash | observed_at |
|---|---|---|---|---|---|
| RUN-EXEC-001 | INITIAL | N/A | ENV-E1 | H1 | 2026-11-26T12:00Z |
| RUN-EXEC-002 | INITIAL | N/A | ENV-E1 | H1 | 2026-11-26T12:10Z |
| RUN-EXEC-003 | REVERIFY | RUN-EXEC-002 | ENV-E1 | H1 | 2026-11-26T12:20Z |
| RUN-EXEC-004 | INITIAL | N/A | ENV-E1 | H1 | 2026-11-26T12:30Z |
| RUN-EXEC-005 | INITIAL | N/A | ENV-E1 | H1 | 2026-11-26T12:40Z |
| RUN-EXEC-006 | REVERIFY | RUN-EXEC-005 | ENV-E1 | H1 | 2026-11-26T12:50Z |
| RUN-EXEC-007 | INITIAL | N/A | ENV-E1 | H1 | 2026-11-26T13:00Z |
| RUN-EXEC-008 | REVERIFY | RUN-EXEC-007 | ENV-E1 | H1 | 2026-11-26T13:10Z |

## Run-Case 明细（联合键 Run×Case 唯一）

| Run-ID | Case-ID | primary AC | primary Cell | assertionCoverage | assertions（AC-ID#assertion=verdict） | evidence（Run×Case×assertion） | Case verdict |
|---|---|---|---|---|---|---|---|
| RUN-EXEC-001 | CASE-EXEC-001 | AC-EXEC-001 | CELL-EXEC-001 | FULL | AC-EXEC-001#main=✅；AC-EXEC-002#secondary=❌ | E-EXEC-001A、E-EXEC-001B | ❌ |
| RUN-EXEC-001 | CASE-EXEC-002 | AC-EXEC-001 | CELL-EXEC-002 | FULL | AC-EXEC-001#main=✅ | E-EXEC-002A | ✅ |
| RUN-EXEC-002 | CASE-EXEC-003 | AC-EXEC-003 | CELL-EXEC-003 | FULL | AC-EXEC-003#main=❌ | E-EXEC-003A | ❌ |
| RUN-EXEC-003 | CASE-EXEC-003 | AC-EXEC-003 | CELL-EXEC-003 | FULL | AC-EXEC-003#main=✅ | E-EXEC-003B | ✅ |
| RUN-EXEC-004 | CASE-EXEC-005 | AC-EXEC-005 | CELL-EXEC-005 | FULL | AC-EXEC-005#main=⏸️ | E-EXEC-005A | ⏸️ |
| RUN-EXEC-005 | CASE-EXEC-008 | AC-EXEC-008 | CELL-EXEC-008 | FULL | AC-EXEC-008#main=✅；AC-EXEC-008#boundary=✅ | E-EXEC-008A | ✅ |
| RUN-EXEC-006 | CASE-EXEC-008 | AC-EXEC-008 | CELL-EXEC-008 | PARTIAL | AC-EXEC-008#boundary=❌ | E-EXEC-008B | ❌ |
| RUN-EXEC-007 | CASE-EXEC-009 | AC-EXEC-009 | CELL-EXEC-009 | FULL | AC-EXEC-009#main=❌；AC-EXEC-009#secondary=❌ | E-EXEC-009A | ❌ |
| RUN-EXEC-008 | CASE-EXEC-009 | AC-EXEC-009 | CELL-EXEC-009 | PARTIAL | AC-EXEC-009#secondary=✅ | E-EXEC-009B | ✅ |

RUN-EXEC-001 同时支撑两个 Case 且结果不同，不允许合并成一个 Run 总 verdict。RUN-EXEC-003 的 reverify_of=RUN-EXEC-002，Case/AC/Cell 完全一致，故 CASE-EXEC-003 当前取最新完整 FULL 基线的 ✅；历史 ❌ 仍保留。

RUN-EXEC-005 是 CASE-EXEC-008 的完整 ✅基线，RUN-EXEC-006 只复验 boundary 却得到 ❌，该部分反例必须覆盖旧 ✅。RUN-EXEC-007 是 CASE-EXEC-009 的完整 ❌基线，RUN-EXEC-008 只有 secondary=✅，不能替代完整计划断言，故当前仍为 ❌；部分通过只追加证据，不整体翻绿。

## Case 当前状态（无执行 Run 也能按证据类型合法闭合）

| Case-ID | primary AC | primary Cell | currentRun-ID/N/A | current verdict | proof/解除条件 |
|---|---|---|---|---|---|
| CASE-EXEC-001 | AC-EXEC-001 | CELL-EXEC-001 | RUN-EXEC-001 | ❌ | 次断言 AC-EXEC-002#secondary=❌；E-EXEC-001B |
| CASE-EXEC-002 | AC-EXEC-001 | CELL-EXEC-002 | RUN-EXEC-001 | ✅ | 主断言 ✅；E-EXEC-002A |
| CASE-EXEC-003 | AC-EXEC-003 | CELL-EXEC-003 | RUN-EXEC-003 | ✅ | 本轮最新完整 FULL 基线 Run；E-EXEC-003B |
| CASE-EXEC-004 | AC-EXEC-004 | CELL-EXEC-004 | N/A | ⏭️ | 功能未实现的存在性/通道证据 E-EXEC-004A，无产品执行 Run |
| CASE-EXEC-005 | AC-EXEC-005 | CELL-EXEC-005 | RUN-EXEC-004 | ⏸️ | 实测观察 E-EXEC-005A、冲突 C-EXEC-001、默认处置建议齐 |
| CASE-EXEC-006 | AC-EXEC-006 | CELL-EXEC-006 | N/A | BLOCKED=UNAUTHORIZED | 缺授权证明 E-EXEC-006A；解除=取得指定测试授权，无执行 Run |
| CASE-EXEC-007 | AC-EXEC-007 | CELL-EXEC-007 | N/A | BLOCKED=DEFERRED | 接受记录 E-EXEC-007A 含接受人/日期/影响；解除=接受人撤销延期 |
| CASE-EXEC-008 | AC-EXEC-008 | CELL-EXEC-008 | RUN-EXEC-005 | ❌ | 最新完整基线 RUN-EXEC-005=✅，但较晚 PARTIAL RUN-EXEC-006 的 boundary=❌覆盖旧通过；E-EXEC-008B |
| CASE-EXEC-009 | AC-EXEC-009 | CELL-EXEC-009 | RUN-EXEC-007 | ❌ | 最新完整基线 RUN-EXEC-007=❌；PARTIAL RUN-EXEC-008 只有 secondary=✅，不足以翻绿；E-EXEC-009A、E-EXEC-009B |

## AC 当前聚合（主/次断言全部参与）

| AC-ID | 当前聚合依据 | AC 当前状态 |
|---|---|---|
| AC-EXEC-001 | CASE-EXEC-001 与 CASE-EXEC-002 的主断言均 ✅；CASE-EXEC-001 的 ❌来自次 AC，不篡改主断言事实 | ✅ |
| AC-EXEC-002 | CASE-EXEC-001 的次断言 secondary=❌，即使其他主 AC 为 ✅，仍不得判 ✅ | ❌ |
| AC-EXEC-003 | INITIAL ❌后 REVERIFY ✅，取本轮最新完整 FULL 基线 Run-Case | ✅ |
| AC-EXEC-004 | ⏭️ 有存在性证据，不计执行覆盖，不冒充产品验证 | ⏭️ |
| AC-EXEC-005 | ⏸️ 有实测观察和完整待定要件 | ⏸️ |
| AC-EXEC-006 | 合法 UNAUTHORIZED，保留解除条件 | BLOCKED=UNAUTHORIZED |
| AC-EXEC-007 | 获接受 DEFERRED，保留接受记录 | BLOCKED=DEFERRED |
| AC-EXEC-008 | 较晚部分 boundary=❌必须覆盖旧完整 ✅，不得被旧 Run 吞没 | ❌ |
| AC-EXEC-009 | 部分 secondary=✅不能替代完整计划断言，当前仍为主断言 ❌ | ❌ |

## 报告预期

所有计划 AC/Case 均已终态或有归因 BLOCKED；无 PARTIAL、OMITTED、BUDGET_EXHAUSTED、孤儿主证或映射缺口。CASE-EXEC-006 为合法外部 BLOCKED，CASE-EXEC-007 为获接受 DEFERRED，AC-EXEC-005 为要件齐的 ⏸️，故 **FINAL-WITH-GAPS**。CASE-EXEC-001、AC-EXEC-002、CASE-EXEC-008、CASE-EXEC-009 的 ❌均不阻止该状态；CASE-EXEC-004 的 ⏭️不计执行覆盖且不宣称领域验证完整。

执行覆盖率按唯一 Case 聚合，本例为 6/9：CASE-EXEC-003 的 INITIAL 与 REVERIFY 只计一个 Case，CASE-EXEC-008/009 的部分反例或部分通过分别按规则计入当前判定，不把 Run 数当 Case 数；CASE-EXEC-006/007 无执行 Run 也能闭合，但不计执行覆盖。CLOSED 只要求终态/归因、判定登记、缺口记录和保底报告签发，证据完整性不再另作 CLOSED 前置。
