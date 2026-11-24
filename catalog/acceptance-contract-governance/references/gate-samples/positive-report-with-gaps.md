# 执行报告正样例（虚构）

报告头：任务=GAP-DEMO；环境=UAT；执行模式=PARALLEL_UNATTENDED；无授权偏离。

## 第一节：关键旅程结论

| persona | 业务目标 | 实际连续路径 | 结论 | 用户可见结果 | 主证 |
|---|---|---|---|---|---|
| 归档员 | 完成票据归档 | 选择→复核→提交→详情 | 受阻 | 回调未返回，详情保持待确认 | evidence/AC-DEMO-002_callback.txt |

## 第二节：用户语言缺陷与残余风险

无已定性用户缺陷。残余风险=机器回调尚未验证，待定性与接受状态不影响本报告签发。

## 第三节：审计与执行状态

- 报告状态：FINAL-WITH-GAPS
- 运行状态：CLOSED
- READY=0；WAITING=0；DONE=1；BLOCKED=1；AC 总数=2；DONE+BLOCKED=2
- BLOCKED=1；OMITTED=0；BUDGET_EXHAUSTED=0
- UI PASS 主证覆盖：1/1

| AC | 状态 | 归因 | 有限努力证明 | 影响 | 解除条件 |
|---|---|---|---|---|---|
| AC-DEMO-002 | BLOCKED | UNREACHABLE | 两次无进展、已换层和换路径、单项预算达到；附原始错误 | 仅机器回调点未验 | 回调测试端点恢复 |

保底报告能力：产物命名或链接失败时仍签发 INTERIM；风险接受 pending 不撤销本报告。
