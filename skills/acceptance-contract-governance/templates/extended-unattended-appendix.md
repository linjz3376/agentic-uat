# PARALLEL_UNATTENDED 并行接管生成块（仅生成期模板）

> 仅当实际存在并行写者或外部接管时，把本块完整内联进任务书。长窗/多回合本身仍使用 SINGLE_WRITER 基础内核，不得仅因长任务选择本块。SINGLE_WRITER 不携带租约/CAS/fencingToken；最终任务书不得保留外部文件指针。

## 扩展执行契约（冻结快照）

### 状态机与队列

```text
PREFLIGHT → RUNNING ⇄ WAITING
PREFLIGHT/RUNNING/WAITING → GLOBAL_NO_GO → REPORTING → CLOSED
RUNNING/WAITING → BUDGET_EXHAUSTED → REPORTING → CLOSED
RUNNING/WAITING ──收判完成且 READY=0、WAITING=0──→ REPORTING → CLOSED
```

- 每 AC 仅属 READY/WAITING/BLOCKED/DONE 一队；DONE=✅/❌/PARTIAL/⏭️/⏸️，已判数=DONE+BLOCKED。
- 动作受阻→保全证据→判定最小作用域→登记原因/依赖→写断点→移动队列→选择下一个 READY；READY 内先关键目标/高风险，再按旅程次序，不得空转。
- 解除但未判→WAITING→READY；收判完成→DONE；失败/超时→BLOCKED。携带 WAITING 不得 REPORTING/CLOSED。活动 WAITING 必须有一次性收判/解除事件/超时处置；只有无 WAITING/长窗项才允许收判安排 N/A；无法安排立即转有归因 BLOCKED，继续其他 READY 或出 INTERIM。
- GLOBAL_NO_GO/BUDGET_EXHAUSTED 时先把剩余 READY/WAITING 逐项转有归因 BLOCKED，双空后才报告。
- 回合四件套：进度对账、判定登记清零、断点与四队列更新、一次性收判或 N/A；无长窗不建常驻巡逻。

### 扩展执行契约字段

| 字段组 | 必填字段/规则 |
|---|---|
| 写者 | writerId、leaseEpoch、fencingToken=leaseEpoch、acquiredAt、leaseExpiresAt、heartbeatAt；主状态写入 CAS 校验 fencingToken |
| 状态 | 状态表路径、运行状态、READY/WAITING/BLOCKED/DONE 数量、已判数、未登记判定清单 |
| 调度 | 队列、依赖、阻塞作用域/归因、解除条件、resumeAfter、下一动作、验收信号、风险优先级 |
| 安全与预算 | 授权清单引用、允许写入位置、项目红线、全局/场景/AC/路径上限与已耗、墙钟截止、报告预留 |
| 基线与观察 | 在途/pending ask/相关数据基线、最后权威观察、时间戳、证据路径、最近完整日志序号 |
| 收判与报告 | pending triggerId、收判包路径、报告路径/签发时刻；风险接受与经验回灌为独立外部待办 |

### 每个未完成 AC 的断点

```text
AC-ID | queue | scope | currentStep | intent | acceptanceSignal
dependencies | blockCause | releaseCondition | resumeAfter
attemptCount | retryLevel | lastObservation | budgetUsed
sideEffectIds | idempotencyKeys | dataBaseline | reconciliationSource
nextAction | evidenceRefs | unregisteredJudgements
```

### 接管与恢复

1. 写者续租；租约过期且权威观察证明失效后，接管者用原子锁/等效 CAS 取得新 leaseEpoch=fencingToken。
2. 枚举 pending triggerId，核目标/契约/epoch，迁移或重建一次性收判；旧收判包先读当前契约并安全重登记，不得静默丢弃或重复执行。
3. 向 reconciliationSource 核 sideEffectIds、幂等键和数据基线，再决定续作/跳过/重放；不可逆副作用不明不得盲放。
4. 旧 fencingToken 写入拒绝；旧写者只能重读契约后加入分片日志。无原子锁时单写者+只追加分片，不多写主状态。
5. 无法安全接管或迁移触发器→相关 WAITING 转有归因 BLOCKED，恢复完整性缺口出 INTERIM 保底报告，不得空等。

### 收判包

收判包=triggerId、探针标识、期望判定要素、台账写入模板+路径、契约路径、leaseEpoch/fencingToken、resumeAfter、可选前置状态快照。收判回合零上下文也必须完成权威复核、判定、登记和四件套。
