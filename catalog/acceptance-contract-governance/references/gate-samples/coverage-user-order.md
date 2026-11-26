# 领域夹具：用户办单覆盖 profile（虚构）

> 用途：提供常见的用户提交申请、办单、审批与通知示例，证明同一 schema 可承载表单校验、代办、草稿、审批状态机、幂等和通知恢复。所有规则、角色和数据均为虚构。

## 来源与选择策略

- 当前 test basis：虚构规则 `O-NEW`、`O-CHANGE`、`O-DELEGATE`、`O-FORM`、`O-DRAFT`、`O-APPROVAL`、`O-IDEMPOTENT`、`O-NOTIFY`。
- 固定基础上下文：`channel=WEB、formVersion=V1、notification=standard`；强制格可覆盖审批与通知上下文。
- 普通表单组合用 Pairwise 并标「非穷尽」；合法代办、草稿续办、重复提交、驳回恢复、通知重试和外部审批为强制格。Pairwise 只计算二元基线值域，draftSaved、external 等特殊状态由强制规则引入，不混入基线样本空间。
- `missingRequired` 只验证必填校验与数据保留，不伪造审批结果；代办提交必须先有有效授权。

| dimension | values | source | constraints/applicability |
|---|---|---|---|
| orderKind | new / change | O-NEW / O-CHANGE | 新办与变更分别核申请内容和最终办单状态 |
| applicant | self / delegated | O-DELEGATE | delegated 必须附代办授权，账户与申请对象不同 |
| formState | complete / missingRequired | O-FORM | 缺必填应在提交前或提交时拒绝，不能进入审批 |
| approvalMode | auto / manual | O-APPROVAL | 自动审批核终态；人工审批核待审、驳回和恢复 |

## Cell 与 Case（P01 至 P06 覆盖四个二元轴的全部两两组合）

| Cell-ID | Case-ID | 条件元组（未写项继承固定上下文） | 来源/风险 | 选择/计划 | 主 AC | 次 AC+断言 | S-ID/步骤 | 预期 |
|---|---|---|---|---|---|---|---|---|
| CELL-ORD-P01 | CASE-ORD-P01 | new、self、complete、auto | O-NEW/标准 | PAIRWISE/PLANNED | AC-ORD-001 | N/A | S-ORD-SELF/submit | 新办成功，办单号、状态和回执一致 |
| CELL-ORD-P02 | CASE-ORD-P02 | new、delegated、missingRequired、manual | O-FORM/标准 | PAIRWISE/PLANNED | AC-ORD-004 | N/A | S-ORD-DELEGATE/validate | 指出缺项并保留已填内容，不创建审批单 |
| CELL-ORD-P03 | CASE-ORD-P03 | change、self、complete、manual | O-CHANGE/标准 | PAIRWISE/PLANNED | AC-ORD-002 | AC-ORD-005#pending | S-ORD-SELF/submit | 变更进入待审，原办单与变更内容可追踪 |
| CELL-ORD-P04 | CASE-ORD-P04 | change、delegated、missingRequired、auto | O-FORM/标准 | PAIRWISE/PLANNED | AC-ORD-004 | N/A | S-ORD-DELEGATE/validate | 拒绝缺项且不改变原办单，不触发自动审批 |
| CELL-ORD-P05 | CASE-ORD-P05 | new、self、missingRequired、auto | O-FORM/标准 | PAIRWISE/PLANNED | AC-ORD-004 | N/A | S-ORD-SELF/validate | 缺项可见、已填内容保留、无重复草稿副作用 |
| CELL-ORD-P06 | CASE-ORD-P06 | new、delegated、complete、auto | O-DELEGATE/关键 | MANDATORY+PAIRWISE/PLANNED | AC-ORD-003 | AC-ORD-001#created | S-ORD-DELEGATE/submit | 代办授权有效才创建；办单归属与代办人不可混淆 |
| CELL-ORD-M01 | CASE-ORD-M01 | orderKind=change、applicant=delegated、formState=complete、approvalMode=manual | O-CHANGE/关键 | MANDATORY/PLANNED | AC-ORD-002 | AC-ORD-003#delegation、AC-ORD-005#pending | S-ORD-DELEGATE/submit | 合法代办变更进入人工待审，三项断言分别留证 |
| CELL-ORD-M02 | CASE-ORD-M02 | orderKind=new、applicant=self、formState=draftSaved、approvalMode=N/A、draft=resumeAfterReload | O-DRAFT/标准 | MANDATORY/PLANNED | AC-ORD-006 | N/A | S-ORD-SELF/reload | 重进后草稿内容、顺序和续办入口一致；未提交审批故审批轴 N/A |
| CELL-ORD-M03 | CASE-ORD-M03 | orderKind=new、applicant=self、formState=complete、approvalMode=auto、submitCount=2、sameIdempotencyKey | O-IDEMPOTENT/关键 | MANDATORY/PLANNED | AC-ORD-007 | N/A | S-ORD-SELF/retry | 只生成一个办单，重复回执不制造第二笔副作用 |
| CELL-ORD-M04 | CASE-ORD-M04 | orderKind=change、applicant=self、formState=complete、approvalMode=manual、review=rejectedThenCorrected | O-APPROVAL/关键 | MANDATORY/PLANNED | AC-ORD-008 | N/A | S-ORD-SELF/resubmit | 驳回原因可见，修正重提保留历史并进入正确状态 |
| CELL-ORD-M05 | CASE-ORD-M05 | orderKind=new、applicant=self、formState=complete、approvalMode=auto、notification=failedThenRetry | O-NOTIFY/关键 | MANDATORY/PLANNED | AC-ORD-009 | N/A | S-ORD-SELF/notification | 通知重试有界且不重复建单，用户可核最终办单状态 |
| CELL-ORD-M06 | CASE-ORD-M06 | orderKind=change、applicant=self、formState=complete、approvalMode=external、externalApproval=pending | O-APPROVAL/关键 | MANDATORY/PLANNED | AC-ORD-010 | N/A | S-ORD-SELF/external | 外部审批未完成不得提前办结；解除事件后恢复收判 |

## Run 与证据节选

| Run-ID | run_kind | reverify_of | Case-ID | primary AC | primary Cell | verdict | observed_at | evidence | 覆盖处理 |
|---|---|---|---|---|---|---|---|---|---|
| RUN-ORD-001 | INITIAL | N/A | CASE-ORD-M05 | AC-ORD-009 | CELL-ORD-M05 | ✅ | 2026-11-26T11:00Z | E-ORD-001 | 通知重试成功只归 M05，不扩展为全部审批/通知组合覆盖 |
| RUN-ORD-002 | REVERIFY | RUN-ORD-001 | CASE-ORD-M05 | AC-ORD-009 | CELL-ORD-M05 | ✅ | 2026-11-26T11:10Z | E-ORD-002 | 复用 M05；通知复验不新增办单 Case，credit 按唯一 Case 聚合 |
| RUN-ORD-003 | INITIAL | N/A | CASE-ORD-M06 | AC-ORD-010 | CELL-ORD-M06 | BLOCKED=UNREACHABLE | 2026-11-26T11:20Z | E-ORD-003 | 解除=收到外部审批完成事件；未验审批状态迁移与提前办结风险，保留格和 Case，不计执行覆盖 |

对账：必需 Cell=12、计划 Case=12、必需格归格率=12/12。P06、M01 的次 AC 均带独立断言，不增加执行子用例数；草稿、提交、审批和通知可以由同一连续旅程串联，但每格仍按自己的 Case 判定。完整执行指标由 Run 台账派生，本节选不虚构。
