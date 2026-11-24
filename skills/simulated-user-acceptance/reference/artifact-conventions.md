# 产物树、报告与执行恢复规范

> 配套 `simulated-user-acceptance` §4 至 §6。目标：任何项目、任何执行者均可检索、审计、恢复和终止。
> **优先级唯一口径**：SUA 的安全/主证/三态/归因核心约束 > 任务书选择的目录 profile 与项目扩展 > 本默认布局。任务书不得降低核心约束；两者冲突时记录偏差并采用更安全、证据更完整的一侧，不触发热切换判据。

## 1 目录结构

```text
<report-root>/                        # 默认 tests/reports/<YYYY-MM-DD>-<domain>/
├─ execution-log.md                   # 必需：环境/预算/最小断点/逐 AC；四队列仅扩展模式
├─ defects.md                         # 必需：已定性+待定性+风险接受+阻塞
├─ final-report.md                    # FINAL / FINAL-WITH-GAPS
├─ interim-report-<YYYY-MM-DD>.md     # INTERIM，与 final 二选一
├─ coverage-audit-<YYYY-MM-DD>.md      # 独立 DOC 审计；非 UAT 报告
├─ defect-ledger-<YYYY-MM-DD>.md      # 有台账义务时必需： dated snapshot
├─ evidence/                          # 必需；无能采集时含失败记录
├─ imports/                           # 有上传动作时必需
├─ exports/                           # 有下载动作时必需
├─ obs/
│  ├─ contract.md                     # 多回合、长窗或无人值守时必需
│  ├─ control-matrix.json             # 有 UI 涉验页时必需
│  └─ results.json                    # 使用机算结果文件时采用；非唯一载体
├─ cleanup-list.md                    # 有造数/清场债务时必需
├─ coverage-table.md                  # 任务书要求时必需
├─ appendix/                          # 可选扩展与清单
├─ tooling/                           # 项目授权内的本轮工具快照
└─ exploration/                       # 可选 G0/SBTM
```

- 目录按适用条件创建；任务书可增加产物或选择等效布局，但必须在头部给出映射。
- `results.json` 不是强制文件名；无脚本环境按覆盖公式手工对账并留痕，禁止虚构机算。
- 最终报告与阶段报告同根二选一；历史阶段报告须移入任务书声明的 appendix，不得让读者误认当前结论。

## 2 文件职责

| 文件 | 必须包含 | 禁止包含 |
|---|---|---|
| `execution-log.md` | 环境闸、授权引用、执行模式、预算、最小断点、逐 AC 状态/证据/缺陷、造数与恢复；四队列仅 EXTENDED_UNATTENDED | 大段报告正文；凭据；SINGLE 伪造四队列 |
| `defects.md` | 已定性表、待定性表、统计、风险接受清单、阻塞项、该轮新增十段式副本 | 无留痕关闭项；凭据 |
| `final-report.md` | 报告状态、未触核验、总览/域/维度、缺陷、对账、残留、结论 | 与当前 interim 并存 |
| `interim-report-<date>.md` | 未触/预算/证据缺口、补测任务单、已完成结论、final 暂缓原因 | 冒充产品已验收 |
| `evidence/` | 判定同时刻截图/DOM/网络/机器响应/对账；失败时原始错误与结构化尝试记录 | 事后补拍冒充；无关个人数据 |
| `imports/`、`exports/` | 原始文件及五要素清单 | 二次编辑冒充原件 |
| `obs/contract.md` | 本文 §6.1 至 §6.4 全部字段、写者租约、四队列、断点、收判触发器 | 硬编码密钥；无时间戳快照 |

## 3 命名细则

### 3.1 证据文件

```text
<AC-ID>_<步骤>_<yyyyMMddTHHmm>.<ext>
例：AC-NMPOOL-002_emptySearch_20261027T1135.png
```

- 步骤用英文短词，避免空格与中文；同一 AC 多张证据按步骤区分，不覆盖。
- 允许 png/json/txt/md/pdf/bin/xlsx 等任务书声明格式；无法采集时使用 `<AC-ID>_captureFailure_<时间戳>.txt` 留原始错误。

### 3.2 导入/导出与报告

```text
<用途>_<AC-ID>_<yyyyMMddTHHmm>.<原扩展名>
interim-report-<YYYY-MM-DD>.md；同日多份追加 -2、-3 并登记
```

保留源文件名时必须在清单登记映射。缺陷编号采用 `DEF-<项目>-<YYYYMMDD>-<三位序号>`，待定性采用 `-P<两位序号>`。

## 4 覆盖、归因与降级

### 4.1 环境、写入与轻量模式

| 情形 | 处理 |
|---|---|
| 任务书指定产物树 | 可改目录布局或扩展，不得降低 SUA 核心约束；映射写入报告 |
| 未指定报告根 | 仅从已获写授权的候选中依次选择项目声明验收目录、现有验收 reports 目录、任务书获准等效目录；默认 `tests/reports/<date>-<domain>/` 也须已有写授权，否则直接回复最小报告 |
| 报告根不可写 | 不请求任意新目录，不向未授权位置落盘；在已有获准位置写最小报告，或直接回复完整最小报告并注明落盘失败 |
| 单点小任务 | 可只保留 execution-log 与证据，报告并入台账末尾并标「轻量模式」 |
| 格式/命名/链接失败 | 记报告合规性偏差并降 INTERIM；**始终允许签发保底报告** |

### 4.2 BLOCKED 归因六分法

| 归因 | 适用条件 | 必备证明 | 对报告状态的影响 |
|---|---|---|---|
| UNREACHABLE | 外部条件不可达 | 有限努力证明：两次无进展、换层、换路径或排除依据、单项预算、原始记录 | 可 FINAL-WITH-GAPS |
| UNAUTHORIZED | 入口/动作存在但未授权 | 授权清单缺口、受影响对象、通道结论 | 可 FINAL-WITH-GAPS |
| OBSERVABILITY_BLOCKED | 产品可能可达，但观测或留证通道失效 | 工具原始错误；无响应时带时间戳的结构化尝试记录 | 可 FINAL-WITH-GAPS |
| DEFERRED | 主动决定本轮不做 | 决策记录、接受人、日期、理由、影响 | 可 FINAL-WITH-GAPS |
| BUDGET_EXHAUSTED | 已声明预算截止 | 预算快照、消耗、剩余项、下一估计 | 必须 INTERIM |
| OMITTED | 条件具备且预算可用却未做 | 补测任务单；不得伪造外部原因 | 必须 INTERIM |

- N/A 有功能不存在证据时改 ⏭️，不计 BLOCKED。
- UNREACHABLE 不要求换人、换时间或穷尽所有合法手段；替代努力到 §4 的有限边界即可停止当前项。
- 同域同条件已有成功而差异不明→先重观察；仍无法说明且本可执行→OMITTED，不得滥用 UNREACHABLE。

### 4.3 报告三态（共同前置+顺序判定）

UAT 身份前置=至少一条真实 persona 产品旅程有定性结论，DOC 不参与。所有模式共同前置=每个计划产品 AC 均为 DONE（✅/❌/PARTIAL/⏭️/⏸️）或六类 BLOCKED。SINGLE_SESSION 另核最小断点无待续动作且无仍可执行独立项，不创建 READY/WAITING/四队列；EXTENDED_UNATTENDED 另核 READY=0、WAITING=0、DONE+BLOCKED=产品 AC 总数。按顺序取首个命中：

| 顺序 | 条件 | 状态 | 文件 |
|---:|---|---|---|
| 1 | 任一入口 PARTIAL；或 OMITTED/BUDGET_EXHAUSTED；或 UI 主证/计划产物缺口；或 ⏸️ 缺待定性/默认建议；或 P=O+S+G 不成立；或任一 S/G 缺稳定 ID、证据或合法映射 | INTERIM | `interim-report-<date>.md` |
| 2 | 不满足 INTERIM，且存在至少一个 BLOCKED/未裁定 ⏸️/控件 S/G；BLOCKED 全为合法外部归因或获接受 DEFERRED；⏸️ 要件齐；S/G 逐项有同类合法映射 | FINAL-WITH-GAPS | `final-report.md`，头部声明全部 BLOCKED/⏸️/S/G 缺口 |
| 3 | 不满足前两类；BLOCKED=0、PARTIAL=0、未裁定 ⏸️=0、UI PASS=0 时主证覆盖率=N/A+理由，UI PASS>0 时对应入口主证=100%、计划产物完整、S+G=0 且 O=P | FINAL | `final-report.md` |

- ❌ 不阻止 FINAL：产品失败不等于执行未完成。
- 自主模式全项有证据且无缺口可 FINAL；有未裁定 ⏸️、外部 BLOCKED 或合法控件缺口时不得 FINAL。只有 ⏸️ 或 S/G、没有 BLOCKED 也可 FINAL-WITH-GAPS；完全没有缺口时禁止 WITH-GAPS。
- INTERIM 可签发并 CLOSED；补测和签收受后续任务管理，不阻止本轮结束。
- FINAL-WITH-GAPS 头部必须有缺口数、归因分布、影响、解除条件；DEFERRED 另附接受人。

### 4.4 补测状态机

```text
BLOCKED → 归因审计
  ├─ OMITTED → OMITTED-PENDING → 补测 → ✅/❌/PARTIAL 或合法转归因
  ├─ BUDGET_EXHAUSTED → 下轮预算任务单 → 补测
  ├─ UNREACHABLE/UNAUTHORIZED/OBSERVABILITY_BLOCKED → 解除条件/触发器 → 收判或 FINAL-WITH-GAPS
  └─ DEFERRED → 接受记录+影响评估 → 保持 FINAL-WITH-GAPS；不得伪装外部触发器
```

- OMITTED 优先补不依赖外部条件的项；补测不改原标准。
- OMITTED-PENDING 不得进入 FINAL 系；可以带补测任务单进入 INTERIM 并 CLOSED。
- 补测任务单字段：AC、前置、预计耗时、负责人、期限、状态、触发器或 N/A。
- UNREACHABLE 防伪抽查只复现有证据的路径；观测通道失效不得要求伪造截图。

## 5 场景、主证与产物闸

### 5.1 报告必须有的三张执行表

**身份与叙事前置**：UAT 报告至少一条真实 persona 产品旅程，否则只能 coverage-audit。报告首节先列「persona｜业务目标｜实际连续路径｜完成/失败/受阻｜用户可见结果｜主证」，不设旅程通过率 KPI；DOC 不进 AC/场景/需求分子/UAT 三态，TECH-OBS 与 UAT 缺陷分开。

1. **场景完成表**：S-ID/persona/基本流/备选流/异常流/交接/挂靠 AC/完成状态/证据；每场景一句「作为 persona，我能/不能完成业务目标」。
2. **入口证据表**：仅列 UI 与 MACHINE；UI 记 AC 数/PASS 数/有对应入口主证的 PASS 数/覆盖率，MACHINE 列契约证据。DOC 对账只进 coverage-audit，UAT 报告最多附「不计入 UAT」的明确链接。
3. **G0/SBTM 记录**：时间盒、四栏笔记、新增 B 点/缺陷/未覆盖。

规则：UI PASS 必须有该形态对应主证——GUI=判定时截图/DOM，TUI/CLI=终端读屏/输出，人类 API=调用方可见响应+最终业务结果；缺对应主证才降 PARTIAL。MACHINE PASS 须有契约响应与内容断言，状态变更另核副作用，纯只读不得造副作用；DOC 只在独立 coverage-audit 判对账。事后补证保留事实但不升级原判定。

### 5.2 产物完整性与保底报告

出报告前检查以下项；失败时修复可修部分，无法修复则记录偏差并出 INTERIM，**任何一项都不得阻止保底报告**：

1. 实际树与任务书 profile 映射一致；必需文件按适用条件存在。
2. 根目录仅含任务树或本规范白名单；扩展集中 obs/appendix/tooling。
3. 证据命名与 AC/步骤/时间戳可反查。
4. 脚本只在 tooling 或任务书指定目录并登记。
5. 证据超过 20 个时生成 manifest 或等效索引。
6. 报告附录声明扩展产物、脚本目录和 manifest。
7. Markdown 相对链接零断链；树外引用有相对路径和「树外」标注。
8. 台账快照有日期后缀及来源；活源不可达时按任务书头部快照和本轮新增重建。
9. 缺陷十段式唯一事实源为跨轮台账；defects.md 为本轮汇总及新增副本。
10. 修复产物缺口后全扫同类实例；校验器须通过一正一反样例才可采信。

### 5.3 报告内容 15 项

1. 系统/版本/环境/执行时间、角色账户矩阵（不含凭据）。
2. 每场景 persona、目标、完成情况、用户感受、关键证据。
3. 每 AC 终态及入口主证。
4. BLOCKED 清单：归因、证明、解除条件、影响。
5. 补测审计：OMITTED、预算耗尽、已补测、残余 PENDING 及任务单。
6. L1/L2/L3、计划 B 点执行率、方法覆盖回执。
7. 用户语言缺陷（persona+目标+入口+实际/预期）、双口径统计、风险接受；TECH-OBS 单列。
8. 路由、需求、不变量、数量对账。
9. 残留数据与清场。
10. 缺陷集群、下轮建议、未覆盖和残余风险。
11. 场景完成表。
12. 入口证据表；UI PASS 主证不足直接 INTERIM。
13. G0/SBTM 记录。
14. 覆盖度量三行；机算优先，工具缺失按统一公式手工对账并留痕，分母 0 标 N/A+理由。
15. 控件清单与计划操作：同时列 I/P/O/S/G、原始 O/P、O/(P-S)、每个 S/G 的 control_id+证据+归因。S 只收获接受 DEFERRED；外部不可达/未授权/观测限制的未操作项进入 G。FINAL 要 S+G=0 且 O=P；S/G 逐项有合法映射才可达 FINAL-WITH-GAPS；缺证明、无映射、OMITTED/BUDGET_EXHAUSTED 均 INTERIM。执行前未纳入 P 的探索档候选不算缺口，禁止执行后改列抽样或 S。

控件统一按稳定 `control_id` 去重，动作行数不等于控件数：`I`=范围内清单控件数，`P`=执行前锁定的计划操作控件数，`O`=P 中完成有效操作的唯一控件数（计划外操作另列），`S`=P 中获接受 DEFERRED 的唯一控件数，`G=P\(O∪S)`=其余计划缺口的唯一控件集合。必须满足 P=O+S+G，O/S/G 互斥。未授权、不可达、观测受限未操作项进 G 并带对应归因，不得挤入 S。清单覆盖率=已落 AC/N/A 的唯一控件数/I；操作率同时展示原始 O/P 与 `O/(P-S)`，分母 0 标 N/A+理由，不得只展示扣除后的 100%。DOM、a11y、视觉按稳定键合并，a11y 只提供差集线索。

## 6 执行契约与收判包 schema（仅 EXTENDED_UNATTENDED）

### 6.0 模式边界与状态机

- SINGLE_SESSION：单会话/单写者/无长窗，只保留场景、AC、当前步骤、下一动作、预算、证据、缺陷和最小断点；不创建 obs/contract，不携带租约/CAS/fencing/四队列。最小停止链=当前动作完成或最小作用域 BLOCKED→继续下一独立项→全部计划产品 AC 为 DONE/BLOCKED 且无待续/可执行独立项→REPORTING→CLOSED。
- EXTENDED_UNATTENDED：有多回合、长窗、并行写者或无人值守接管时才启用本节完整规范；任务书只冻结实际所选模式。

```text
PREFLIGHT → RUNNING ⇄ WAITING
PREFLIGHT/RUNNING/WAITING → GLOBAL_NO_GO → REPORTING → CLOSED
RUNNING/WAITING → BUDGET_EXHAUSTED → REPORTING → CLOSED
RUNNING/WAITING ──收判完成且 READY=0、WAITING=0──→ REPORTING → CLOSED
```

WAITING 解除未判→READY，收判完成→DONE，失败/超时→BLOCKED；携带 WAITING 不得 CLOSED。活动 WAITING 必须有一次性收判、可验证解除事件与超时处置；收判安排只有本轮无 WAITING/长窗项才可为 N/A。无法安排收判时立即转有归因 BLOCKED，继续其他 READY 或出 INTERIM，不得保持无出口 WAITING。进入全局支路先把剩余 READY/WAITING 逐项转有归因 BLOCKED。DONE 仅含 ✅/❌/PARTIAL/⏭️/⏸️，已判数=DONE+BLOCKED。

当前动作受阻后：保全证据→判定最小作用域→登记原因/依赖→写断点→移动队列→选择下一个 READY。READY 内先关键业务目标与高风险，再按任务书旅程次序；不得沿原路径空转。READY/WAITING 均空才进入 REPORTING。

### 6.1 契约字段

`obs/contract.md` 在长窗、多回合或无人值守时创建，至少包含：

1. **登记写者与租约**：writerId、leaseEpoch、fencingToken=leaseEpoch、acquiredAt、leaseExpiresAt、heartbeatAt；过户使用原子锁/等效 CAS，所有主状态写入必须校验当前 fencingToken。
2. **状态唯一事实源**：本报告树状态表路径；并行执行者可写分片日志，只有当前合法写者归并主状态。
3. **运行状态与进度**：PREFLIGHT/RUNNING/WAITING/REPORTING/GLOBAL_NO_GO/BUDGET_EXHAUSTED/CLOSED、各队列数量、已判/总数；进入 REPORTING/CLOSED 前 READY=0 且 WAITING=0。
4. **未登记判定清单**：初始空，判定当回合登记并划除。
5. **四队列明细**：READY/WAITING/BLOCKED/DONE；每项见 §6.2。
6. **任务与下一拍**：目的、验收信号、下一动作、风险优先级。
7. **闸门与红线**：授权清单引用、撞车红线、允许写入位置。
8. **预算**：全局/场景/AC/路径上限、已消耗、墙钟截止、报告预留。
9. **用例间基线**：仅校验会影响下一场景证据或状态的在途、pending ask、队列和数据基线；无关偏离登记后不得阻塞。
10. **最后新鲜观察与恢复点**：权威源、时间戳、证据路径、最近完整日志序号。
11. **收判安排**：一次性 triggerId、resumeAfter、解除条件、超时处置；仅无 WAITING/长窗项时可 N/A。
12. **报告状态与外部待办**：报告路径/签发时刻；风险接受、经验回灌单独标记，不参与执行 CLOSED 判定。

### 6.2 每个未完成 AC 的断点字段

```text
AC-ID | queue | scope | currentStep | intent | acceptanceSignal
dependencies | blockCause | releaseCondition | resumeAfter
attemptCount | retryLevel | lastObservation | budgetUsed
sideEffectIds | idempotencyKeys | dataBaseline | reconciliationSource
nextAction | evidenceRefs | unregisteredJudgements
```

恢复顺序：读租约和最近完整记录→以 CAS 获取写者权和新 fencingToken→枚举全部 pending triggerId 并迁移/重建一次性收判→向权威源核对 `sideEffectIds` 与数据基线→再决定续作/跳过/重放→更新四队列。无法证明副作用是否发生时，禁止盲目重放不可逆动作；无法迁移触发器时不得假称接管成功，受影响 WAITING 转 BLOCKED，恢复完整性缺口按 INTERIM 处理。

### 6.3 写者失效接管

- 写者必须续租；租约过期且心跳/进程权威观察证明失效后，其他载体可按原子锁取得新 `leaseEpoch=fencingToken`。
- 接管必须枚举所有 pending triggerId，验证目标、契约路径和 epoch，迁移或重建一次性收判后再更新契约；旧收判包先读当前契约并安全重登记，不得因旧 epoch 直接丢弃唯一唤醒，也不得重复执行已核副作用。
- 旧写者恢复后携带旧 fencingToken 的写入必须拒绝，先读新契约再作为分片写者加入。
- 无原子锁能力时不得多写者并写主状态；采用单写者加只追加分片日志，并在保底报告中声明接管能力缺口。
- 无法安全接管或迁移触发器→受影响 WAITING 转有归因 BLOCKED，出 INTERIM 保底报告；任何接管都不得成为无限等待理由。

### 6.4 四件套与收判包

- **四件套固定定义**：①进度对账 ②判定登记清零 ③断点与四队列更新 ④一次性收判触发器或 N/A。所有文档只使用这一口径。
- 收判包字段：triggerId、探针标识、期望判定要素、台账写入模板+路径、契约路径、leaseEpoch/fencingToken、resumeAfter、前置状态快照（可选）。旧包到期先读当前契约并请求安全重登记，不得静默丢弃或重复执行。
- 收判回合零上下文也必须能完成权威复核、判定、登记和四件套；无长窗项时不得为凑件创建常驻巡逻。

## 7 常见错误

| 错误 | 正确做法 |
|---|---|
| 因命名、断链或缺文件而不出报告 | 记录合规性偏差并出 INTERIM 保底报告 |
| 把 OMITTED-PENDING 清完才允许 INTERIM | PENDING 只阻止 FINAL 系；INTERIM 带任务单即可 CLOSED |
| 把 CLOSED 当产品通过或风险已接受 | 分开记录执行、报告、接受、回灌四层状态 |
| 并行会话无接管地等待唯一写者 | 使用租约、fencing token 与分片日志 |
| 恢复时直接重复未完成的有副作用动作 | 先查副作用 ID 与权威状态 |
| 无原子锁仍多写者并写主状态 | 单写者+分片日志，或出 INTERIM |
| 事后补截图当判定证据 | 标事后补证，不升级原判定 |
| 跨树裸文件名引用 | 使用可达相对路径并标树外 |
| 为凑四件套创建常驻巡逻 | 无长窗时第四件写 N/A |
