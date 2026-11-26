# 覆盖归格负样例（虚构）

> 每行对应一个独立机验缺口；不得因存在大量 AC、场景或截图而放行。

| 缺陷锚 | 虚构记录 | 必须输出 |
|---|---|---|
| MISSING_CASE | 必需 `CELL-REP-X01` 为高风险 PLANNED，但没有任何主 Case | CELL_WITHOUT_CASE |
| MULTI_PRIMARY_AC | `CASE-ORD-X01` 同时写主 AC=`AC-ORD-001`、`AC-ORD-003` | CASE_PRIMARY_AC_NOT_UNIQUE |
| REVERIFY_NEW_CASE | `RUN-REVERIFY-88` 的 `reverify_of=RUN-OLD-01` 原指向 `CASE-REP-M04`，新明细却改成 `CASE-REP-R99` 以追加覆盖数 | REVERIFY_CASE_CHANGED |
| ORPHAN_EVIDENCE | `E-REP-X88` 被声明为计划主证，却只有 `REVERIFY-*` 文件名，Run/Case/AC 均无有效映射 | ORPHAN_EVIDENCE_NO_CREDIT |
| DUPLICATE_EXECUTION | 一次 Run 给主 Case 和两个次 AC 各计一次执行子用例，总数由 1 写成 3 | EXECUTION_DOUBLE_COUNT |
| SCENARIO_REKEY | S16 重排成 S18 时把原 `CASE-REP-M04` 改号为 `CASE-REP-S18-NEW` | CASE_ID_CHANGED_BY_SCENARIO |
| PAIRWISE_EXHAUSTIVE | 6 个 Pairwise 样本被标为「全部组合已穷尽」，无约束或证明 | PAIRWISE_FALSE_EXHAUSTIVE |
| UNMAPPED_BLOCKED | XXL-JOB 高风险分支只写「待验」，无 Cell/AC/Case、归因和解除条件 | BLOCKED_NOT_MAPPED |
| FIXED_TARGET | 未给维度、约束和选择策略，先声明「必须凑到 64 条即完整」 | FIXED_CASE_TARGET |
| CHERRY_PICK_AC | 同一主 AC 下 5 个计划 Case 仅 1 个 ✅，其余未执行，却将 AC 判 ✅ | AC_PASS_WITH_UNCLOSED_CASES |
| HISTORICAL_CREDIT | 把上一轮 REVERIFY 截图直接计入本轮执行覆盖率 | CROSS_ROUND_EVIDENCE_CREDIT |
| RUN_CASE_GRAIN_AMBIGUOUS | 同一 Run 包含 Case-A=✅、Case-B=❌，却只有一行 Run×Case 和一个总 verdict=✅ | RUN_CASE_GRAIN_AMBIGUOUS |
| SECONDARY_AC_FAILURE_IGNORED | 主 AC 全部主断言 ✅，但计划次断言 `AC-X#secondary=❌`，仍判该次 AC ✅ | SECONDARY_AC_FAILURE_IGNORED |
| STALE_RUN_SELECTED | INITIAL=✅，较晚 REVERIFY=❌，却绕过最新完整基线与部分 Run 时序选择旧 ✅ | STALE_RUN_SELECTED |
| ORPHAN_RELABEL_BYPASS | E-X 已被计划主证表引用，随后改标 UNMAPPED_OBSERVATION 以免除 Run/Case/AC 归格 | ORPHAN_RELABEL_BYPASS |

正例对照：`RUN-X01=EXPLORATORY` 的未映射观察若明确标 `UNMAPPED_OBSERVATION`、未用于计划覆盖声明，只进入探索转化评审，不得仅因 N/A 映射误报为孤儿证据；它仍不得提供覆盖 credit。
