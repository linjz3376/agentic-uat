# 领域夹具：权限治理覆盖 profile（虚构）

> 用途：证明同一 schema 可承载角色、资源、操作和租户组合。规则均为虚构；权限拒绝格不得因 Pairwise 降维而被抽样跳过。

## 来源与选择策略

- 当前 test basis：虚构规则 `P-OWN`、`P-TEAM`、`P-ARCHIVE`、`P-TENANT`、`P-REVOKE`、`P-CLIENT`。
- 固定基础上下文：`tenant=T1、channel=WEB、caller=human`；强制格可覆盖调用方和租户。
- 普通组合用 Pairwise 并标「非穷尽」；member 跨成员、跨租户、归档导出、撤权会话和机器令牌为强制拒绝格。Pairwise 只计算二元基线值域，serviceClient、revokedMember 等特殊角色由强制规则引入，不混入基线样本空间。
- `own` 表示本人资源，`team` 表示他人资源且默认在 T1；`resourceTenant` 可显式覆盖资源所在租户。允许与拒绝必须有正向基线，拒绝只挂在对应角色/操作/对象状态上。

| dimension | values | source | constraints/applicability |
|---|---|---|---|
| role | member / admin | P-OWN / P-TEAM | 使用对应测试账户，不以管理员代替 member |
| resourceOwnership | own / team | P-OWN / P-TEAM | own=本人资源；team=同租户他人资源 |
| action | read / export | P-OWN / P-ARCHIVE | read 与 export 分开判定，不用 read 放行推导 export |
| objectState | active / archived | P-ARCHIVE | 归档可读不代表归档可导出 |

## Cell 与 Case（P01 至 P06 覆盖四个二元轴的全部两两组合）

| Cell-ID | Case-ID | 条件元组（未写项继承固定上下文） | 来源/风险 | 选择/计划 | 主 AC | 次 AC+断言 | S-ID/步骤 | 预期 |
|---|---|---|---|---|---|---|---|---|
| CELL-PRM-P01 | CASE-PRM-P01 | member、own、read、active | P-OWN/关键 | MANDATORY+PAIRWISE/PLANNED | AC-PRM-001 | N/A | S-PRM-MEMBER/open | member 可见本人有效资源及正确字段 |
| CELL-PRM-P02 | CASE-PRM-P02 | member、team、export、archived | P-TEAM/关键 | MANDATORY+PAIRWISE/PLANNED | AC-PRM-002 | N/A | S-PRM-MEMBER/export | 拒绝且无导出文件；仅归 member 跨成员限制，不据此证明归档规则 |
| CELL-PRM-P03 | CASE-PRM-P03 | admin、own、read、archived | P-ARCHIVE/关键 | MANDATORY+PAIRWISE/PLANNED | AC-PRM-003 | N/A | S-PRM-ADMIN/open | admin 可读归档资源，状态标识准确 |
| CELL-PRM-P04 | CASE-PRM-P04 | admin、team、export、active | P-TEAM/关键 | MANDATORY+PAIRWISE/PLANNED | AC-PRM-001 | N/A | S-PRM-ADMIN/export | admin 可导出同租户有效资源且内容一致 |
| CELL-PRM-P05 | CASE-PRM-P05 | member、own、export、active | P-OWN/关键 | MANDATORY+PAIRWISE/PLANNED | AC-PRM-001 | N/A | S-PRM-MEMBER/export | member 可导出本人有效资源且不含他人数据 |
| CELL-PRM-P06 | CASE-PRM-P06 | member、team、read、active | P-TEAM/关键 | MANDATORY+PAIRWISE/PLANNED | AC-PRM-002 | N/A | S-PRM-MEMBER/open | 拒绝同租户他人资源访问且不泄露字段 |
| CELL-PRM-M01 | CASE-PRM-M01 | role=member、resourceOwnership=team、action=read、objectState=archived | P-TEAM/关键 | MANDATORY/PLANNED | AC-PRM-002 | N/A | S-PRM-MEMBER/open | 拒绝且不泄露归档内容；仅归 member 跨成员限制，归档读由可区分 Case 证明 |
| CELL-PRM-M02 | CASE-PRM-M02 | role=member、resourceOwnership=team、action=export、objectState=active | P-TEAM/关键 | MANDATORY/PLANNED | AC-PRM-002 | N/A | S-PRM-MEMBER/export | 拒绝且无文件、无导出任务副作用 |
| CELL-PRM-M03 | CASE-PRM-M03 | role=admin、resourceOwnership=team、action=export、objectState=archived | P-ARCHIVE/关键 | MANDATORY/PLANNED | AC-PRM-003 | N/A | S-PRM-ADMIN/export | admin 仍不可导出归档资源，拒绝原因可见 |
| CELL-PRM-M04 | CASE-PRM-M04 | resourceTenant=T2、role=member、resourceOwnership=team、action=read、objectState=active | P-TENANT/关键 | MANDATORY/PLANNED | AC-PRM-004 | N/A | S-PRM-MEMBER/directId | 跨租户拒绝且不泄露存在性或字段 |
| CELL-PRM-M05 | CASE-PRM-M05 | role=revokedMember、resourceOwnership=own、action=read、objectState=active | P-REVOKE/关键 | MANDATORY/PLANNED | AC-PRM-005 | N/A | S-PRM-MEMBER/reopen | 撤权后旧会话不得继续读取 |
| CELL-PRM-M06 | CASE-PRM-M06 | caller=integrationClient、role=serviceClient、resourceOwnership=team、token=revoked、action=read、objectState=active | P-CLIENT/关键 | MANDATORY/PLANNED | AC-PRM-006 | N/A | S-PRM-CLIENT/call | MACHINE 契约返回 401 与稳定业务错误码，无数据泄露 |

## Run 与证据节选

| Run-ID | run_kind | reverify_of | Case-ID | primary AC | primary Cell | verdict | observed_at | evidence | 覆盖处理 |
|---|---|---|---|---|---|---|---|---|---|
| RUN-PRM-001 | INITIAL | N/A | CASE-PRM-M04 | AC-PRM-004 | CELL-PRM-M04 | ✅ | 2026-11-26T10:00Z | E-PRM-001 | 跨租户拒绝主证只归 M04；不得扩展为所有权限组合通过 |
| RUN-PRM-002 | REVERIFY | RUN-PRM-001 | CASE-PRM-M04 | AC-PRM-004 | CELL-PRM-M04 | ✅ | 2026-11-26T10:10Z | E-PRM-002 | 复用 M04；复验不新增权限拒绝格，credit 按唯一 Case 聚合 |
| RUN-PRM-003 | INITIAL | N/A | CASE-PRM-M05 | AC-PRM-005 | CELL-PRM-M05 | BLOCKED=UNAUTHORIZED | 2026-11-26T10:20Z | E-PRM-003 | 解除=获得撤权测试授权及可恢复账号；未验旧会话越权风险，保留计划 Case 分母，若 AC 仍 BLOCKED 另计基础未触率分子 |

对账：必需 Cell=12、计划 Case=12、必需格归格率=12/12。P02、M01 的拒绝可由 member 跨成员限制完全解释，故不叠加归档次 AC；归档规则由 P03、M03 等可区分 Case 承接。管理员证据不得替代 member 或 integrationClient 的对应入口主证。完整执行指标由 Run 台账派生，本节选不虚构。
