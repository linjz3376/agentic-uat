# 领域夹具：履约还款覆盖 profile（虚构）

> 用途：证明通用 Cell/Case/Run schema 可承载资金域；不是任何真实产品的需求、数据或固定用例数。夹具只有 12 个由本例选择策略派生的计划格，禁止把 64 或其他数量当通用目标。

## 来源与选择策略

- 当前 test basis：虚构规则 `R-NORMAL`、`R-ARREARS`、`R-CUTOVER`、`R-SETTLE`、`R-EXCESS`、`R-JOB`、`R-APPROVAL`。
- 固定基础上下文：`orderVersion=02、period=12、channel=USER、notification=standard`；强制格可显式覆盖固定值。
- 普通四维组合用 Pairwise，标记「非穷尽」；版本/期数代表、第35月、提前结清、溢缴款、严重违约、通知重试和外部审批作为强制格单列。Pairwise 只计算维度表二元基线值域，强制格可凭来源引入特殊类型/状态，不混入该基线样本空间。
- 资金与外部副作用均为关键档；不可达组合必须以来源裁剪，本例无静默 EXCLUDED。

| dimension | values | source | constraints/applicability |
|---|---|---|---|
| repaymentType | normal / arrears | R-NORMAL / R-ARREARS | 决定本金、欠费及剩余金额断言 |
| billStatus | unpaid / partiallyPaid | R-NORMAL | 部分已还时核剩余金额，不把原账单额重复扣款 |
| dateBoundary | day20 / day21 | R-CUTOVER | 两日分别代表切点两侧，必须保留 |
| amountRelation | exact / insufficient | R-NORMAL / R-ARREARS | 不足额不得误判全部结清 |

## Cell 与 Case（Pairwise P01 至 P06 已覆盖上述四维全部两两组合）

| Cell-ID | Case-ID | 条件元组（未写项继承固定上下文） | 来源/风险 | 选择/计划 | 主 AC | 次 AC+断言 | S-ID/步骤 | 预期 |
|---|---|---|---|---|---|---|---|---|
| CELL-REP-P01 | CASE-REP-P01 | normal、unpaid、day20、exact | R-NORMAL/关键 | MANDATORY+PAIRWISE/PLANNED | AC-REP-001 | N/A | S-REP-USER/confirm | 金额、状态、日期与渠道回执一致 |
| CELL-REP-P02 | CASE-REP-P02 | normal、partiallyPaid、day21、insufficient | R-NORMAL/关键 | MANDATORY+PAIRWISE/PLANNED | AC-REP-001 | N/A | S-REP-USER/confirm | 只扣可用金额并显示剩余应还，不假结清 |
| CELL-REP-P03 | CASE-REP-P03 | arrears、unpaid、day20、insufficient | R-ARREARS/关键 | MANDATORY+PAIRWISE/PLANNED | AC-REP-002 | N/A | S-REP-USER/confirm | 欠费与剩余债务清晰，失败结果可重试 |
| CELL-REP-P04 | CASE-REP-P04 | arrears、partiallyPaid、day21、exact | R-ARREARS/关键 | MANDATORY+PAIRWISE/PLANNED | AC-REP-002 | N/A | S-REP-USER/confirm | 欠费核销与剩余金额一致，不重复扣款 |
| CELL-REP-P05 | CASE-REP-P05 | normal、unpaid、day21、exact | R-CUTOVER/关键 | MANDATORY+PAIRWISE/PLANNED | AC-REP-001 | N/A | S-REP-USER/confirm | 切点后金额、账单日和状态按规则变化 |
| CELL-REP-P06 | CASE-REP-P06 | normal、partiallyPaid、day20、exact | R-NORMAL/关键 | MANDATORY+PAIRWISE/PLANNED | AC-REP-001 | N/A | S-REP-USER/confirm | 部分还款后剩余金额与通知一致 |
| CELL-REP-M01 | CASE-REP-M01 | orderVersion=04、period=24、repaymentType=normal、billStatus=unpaid、dateBoundary=day20、amountRelation=exact | R-NORMAL/关键 | MANDATORY/PLANNED | AC-REP-003 | N/A | S-REP-USER/confirm | 新版本与24期计划金额、期次一致 |
| CELL-REP-M02 | CASE-REP-M02 | period=36、currentPeriod=35、repaymentType=normal、billStatus=unpaid、dateBoundary=day20、amountRelation=exact | R-NORMAL/关键 | MANDATORY/PLANNED | AC-REP-004 | N/A | S-REP-USER/confirm | 第35月不越界、不提前结清，剩余期数正确 |
| CELL-REP-M03 | CASE-REP-M03 | repaymentType=earlySettlement、billStatus=unpaid、dateBoundary=day20、amountRelation=settlementExact、channel=USER | R-SETTLE/关键 | MANDATORY/PLANNED | AC-REP-005 | N/A | S-REP-USER/settle | 结清金额、后续期次和最终状态一致 |
| CELL-REP-M04 | CASE-REP-M04 | repaymentType=overpayment、billStatus=unpaid、dateBoundary=day20、amountRelation=excess、channel=USER | R-EXCESS/关键 | MANDATORY/PLANNED | AC-REP-006 | N/A | S-REP-USER/confirm | 溢缴款动态计算、余额去向和通知一致 |
| CELL-REP-M05 | CASE-REP-M05 | repaymentType=severeDelinquency、billStatus=unpaid、dateBoundary=day21、amountRelation=exact、channel=XXL_JOB、notification=retry_2 | R-JOB/关键 | MANDATORY/PLANNED | AC-REP-007 | N/A | S-REP-JOB/retry | 严重违约扣款、两次通知重试与幂等结果一致 |
| CELL-REP-M06 | CASE-REP-M06 | repaymentType=earlySettlement、billStatus=unpaid、dateBoundary=day20、amountRelation=settlementExact、channel=XXL_JOB、externalApproval=required、approvalState=pending | R-APPROVAL/关键 | MANDATORY/PLANNED | AC-REP-008 | N/A | S-REP-JOB/approve | 未获审批不得执行；获批后才允许收判 |

## Run 与证据节选（不是完整执行报告，不虚构执行覆盖率）

| Run-ID | run_kind | reverify_of | Case-ID | primary AC | primary Cell | verdict | observed_at | evidence | 覆盖处理 |
|---|---|---|---|---|---|---|---|---|---|
| RUN-REP-001 | INITIAL | N/A | CASE-REP-P01 | AC-REP-001 | CELL-REP-P01 | ✅ | 2026-11-26T09:00Z | E-REP-001 | 计入 P01；AC-REP-001 其他 Case 仍须独立完成 |
| RUN-REP-002 | REVERIFY | RUN-REP-001 | CASE-REP-P01 | AC-REP-001 | CELL-REP-P01 | ✅ | 2026-11-26T09:10Z | E-REP-002 | 复用普通 P01，不新增 Case；两个 Run 的 credit 按唯一 Case 聚合 |
| RUN-REP-003 | INITIAL | N/A | CASE-REP-M06 | AC-REP-008 | CELL-REP-M06 | BLOCKED=UNAUTHORIZED | 2026-11-26T09:20Z | E-REP-003 | 解除条件=获得外部审批执行窗口；未验审批后自动扣款与提前结清副作用，保留高风险残余并不计执行覆盖 |
| RUN-REP-X01 | EXPLORATORY | N/A | N/A | N/A | N/A | OBS | 2026-11-26T09:30Z | E-REP-X01 | UNMAPPED_OBSERVATION；新候选先评审，不计计划覆盖 |

对账：必需 Cell=12、计划 Case=12、每格至少一个主 Case，必需格归格率=12/12。P01 至 P06 只是普通组合 Pairwise 样本；M01 至 M06 不被 Pairwise 抽样替代。执行指标只能由完整 Run 台账计算，场景改名或 REVERIFY 文件增多均不改变上述设计分母。
