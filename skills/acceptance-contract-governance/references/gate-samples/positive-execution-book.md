# 执行控制正样例（虚构）

- 执行模式：EXTENDED_UNATTENDED，因多回合长窗而启用完整契约。
- 拟人身份：归档员为完成票据归档，从待发清单连续执行选择→复核→提交→详情回显；最终给出完成/失败/受阻定性结论。
- DOC/AUDIT 只在独立覆盖审计区，不进产品 AC、场景矩阵或需求验证分子。
- 缺陷摘要包含 persona+业务目标+真实入口+实际/预期结果；控制台/日志先记 TECH-OBS，关联入口失败才形成 UAT ❌。
- 冻结快照哈希：`<hash>`；运行时规范外部依赖=0。
- Execution-Contract：writerId、leaseEpoch、fencingToken=leaseEpoch、acquiredAt、leaseExpiresAt、heartbeatAt、状态源、未登记判定、READY/WAITING/BLOCKED/DONE、预算、新鲜观察、一次性收判、报告字段。
- 断点完整字段：AC-ID、queue、scope、currentStep、intent、acceptanceSignal、dependencies、blockCause、releaseCondition、resumeAfter、attemptCount、retryLevel、lastObservation、budgetUsed、sideEffectIds、idempotencyKeys、dataBaseline、reconciliationSource、nextAction、evidenceRefs、unregisteredJudgements。
- 收判包完整字段：triggerId、探针标识、期望判定要素、台账写入模板+路径、契约路径、leaseEpoch/fencingToken、resumeAfter、可选前置状态快照。
- 接管：CAS 新 epoch→枚举并迁移 pending triggerId→权威源核副作用→续跑；旧包安全重登记；无法迁移则相关 WAITING 转 BLOCKED 并出 INTERIM 保底报告。
- 授权：操作者明确确认只读导航、指定测试对象写入和报告根；secrets 由提供方受控注入；未授权优雅重启不执行。
- 阻塞：按 ACTION/AC/SCENARIO/DOMAIN/GLOBAL 取最小范围；凭据失效只移动相关 AC 至 BLOCKED，独立 READY 继续。
- 有限归因：两次无进展后换层、换路径并达到单项预算，且有原始错误/时间戳/步骤记录，可判 UNREACHABLE；观测失败有原始错误或结构化尝试记录，可判 OBSERVABILITY_BLOCKED。
- 预算：全局耗尽先把剩余 READY/WAITING 逐项转 BUDGET_EXHAUSTED，确认 READY=0、WAITING=0，再出 INTERIM 并 CLOSED。
- 迟发依赖：记录影响与解除条件，聚合通知，不打回整书。
- Final-Report-Gate：FINAL、FINAL-WITH-GAPS、INTERIM 分支分别核验；格式失败仍签保底报告。
- 状态反例：自主全 ✅ 且 READY=WAITING=0、无 ⏸️/BLOCKED/控件缺口→FINAL；任一 MACHINE PARTIAL→INTERIM；⏸️ 缺待定性/默认建议→INTERIM。
- 控件反例：S/G>0 不得 FINAL；每项必须有稳定 control_id、证据和合法归因；外部不可达 UI 未操作项进 G，可 FINAL-WITH-GAPS，不因 O<P-S 自动 INTERIM。
- 四件套：进度对账、判定登记清零、断点与四队列更新、一次性收判或 N/A。
