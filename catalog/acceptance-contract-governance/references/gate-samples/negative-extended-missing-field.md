# 扩展接管缺字段负样例（虚构）

- 执行模式：EXTENDED_UNATTENDED。
- 已有 writerId、leaseEpoch=fencingToken、acquiredAt、leaseExpiresAt、heartbeatAt、四队列、预算、triggerId、收判包和恢复步骤。
- 断点字段：AC-ID、queue、scope、currentStep、intent、acceptanceSignal、dependencies、blockCause、releaseCondition、resumeAfter、attemptCount、retryLevel、lastObservation、budgetUsed、sideEffectIds、idempotencyKeys、dataBaseline、nextAction、evidenceRefs、unregisteredJudgements。

刻意缺少 reconciliationSource。§22 必须逐字段输出 INVALID/FAIL，不得因 writerId 或 leaseEpoch 存在而通过。
