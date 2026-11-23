# Agentic UAT

**UAT by AI agents — 驾驭 AI 执行体完成验收测试，人类只做裁判。**

[English](#english) | [中文](#中文)

---

## English

Agentic UAT is a two-skill methodology package for the AI-agent era of acceptance testing:

- **acceptance-contract-governance** (v1.5.33) — turns "test feature X thoroughly" into a self-contained, executable acceptance-instruction document: AC-ID'd verification points with entry paths (`[UI]`/`[MACHINE]`), scenario modeling, four live sources of test basis (PRD cross-cut + defect ledger + live UI/endpoint recon + eleven-dimension depth drill), machine-checkable coverage metrics, and a final-report gate.
- **simulated-user-acceptance** (v3.5.9) — executes those instructions as real users: persona-based end-to-end journeys, bounded interaction control flow, UI-primary evidence, L1/L2/L3 coverage, blocked-attribution auditing (UNREACHABLE/OMITTED/DEFERRED/N-A), three-state reporting (FINAL / FINAL-WITH-GAPS / INTERIM), and artifact-tree gates.

**What it is not**: this is not a testing framework for agent harnesses, and not a unit-test generator. It is a *methodology* for letting AI agents do the walking of user-acceptance testing while humans keep judgment.

### How the two skills work together

```
你只说意图              生成侧治理                     执行侧验收              闭环回灌
"重点测 XX"  →  acceptance-contract-governance  →  simulated-user-acceptance  →  经验回项目侧
              ├─ 四活源挖素材（PRD/台账/侦察/十一维）   ├─ persona 真实入口执行        （B 表变异假设、
              ├─ 场景建模 + AC-ID 验收点表             ├─ UI 主证 + 受阻归因审计        fixtures/playbooks，
              ├─ 自包含任务书（判据/手册/降级内嵌）      ├─ 三态报告（FINAL/FINAL-WITH-   新验收点递单回流
              └─ 门槛标注（⛔挂台账/⏭️/⏸️）    ─────▶   GAPS/INTERIM）+ 产物树       ─────▶  下一轮生成侧）
                   生成单 ≠ 执行单                          执行期只看本轮任务书+本轮实测
```

Division of labor (single source: `references/skill-interface.md` inside the governance skill):

| Concern | Owner | Notes |
|---|---|---|
| 判据从哪来（test basis、AC-ID、覆盖度量口径） | governance | coverage-metrics.md is the only authority |
| 产物长什么样（报告三态/产物树/证据命名） | execution | artifact-conventions.md is the only authority |
| 执行期格式冲突 | execution wins | deviation logged and submitted for adjudication |
| 生成单 vs 执行单 | strictly separated | generate-then-execute without review固化s unvetted depth into execution |

Typical flow: state intent ("重点测 XX 模块 / 全项目深度测一遍") → governance skill produces the task book → any capable agent executes it book-in-hand under the execution skill's discipline → report lands in the artifact tree → lessons flow back to project-side knowledge. One sentence per side; the books carry everything else.

### Design tenets

- Human-first judgment: real UI is the primary evidence entry; API is for test-data prep or machine personas only
- Bounded autonomy: fresh-perception contract, authoritative-layer verification, progress-proof retry ladder, hard budgets — every task terminates
- Machine-checkable quality gates: coverage metrics computed, not vibes; docs-drift and attribution audited
- Carrier-neutral: works with any AI execution agent that can receive task injection (CLI headless, IDE automation, client APIs); OpenCode is one example carrier, not a dependency
- Standards-aligned: ISO/IEC/IEEE 29119-3 test documentation split, three-state reporting, artifact conventions

### Install

**Route A — HTTP catalog (native, zero copy)** — add to `opencode.json(c)`:

```jsonc
{
  "skills": ["https://cdn.jsdelivr.net/gh/linjz3376/agentic-uat@v1/catalog/"]
}
```

**Route B — npm**:

```bash
npm i -g @linjz/agentic-uat
# then copy (or symlink) the bundled skills into your skills source:
# Windows:  xcopy /E /I "%APPDATA%\npm\node_modules\@linjz\agentic-uat\skills\*" "%USERPROFILE%\.config\opencode\skills\"
# mac/linux: cp -r "$(npm root -g)/@linjz/agentic-uat/skills/*" ~/.config/opencode/skills/
```

**Route C — git**:

```bash
git clone https://github.com/linjz3376/agentic-uat
# add to opencode.json(c): "skills": ["/absolute/path/to/agentic-uat/skills"]
```

Then invoke: "重点测 XX 模块" routes to the governance skill (task-book generation); executing the book routes to the execution skill. Any AI execution agent with task-injection ability works — OpenCode, Claude Code, Codex, or your own harness.

### Repository layout

```
skills/   the two skills (directory form, portable)
catalog/  HTTP catalog (index.json + per-skill files) for Route A
```

Skill versions inside this package: acceptance-contract-governance 1.5.33, simulated-user-acceptance 3.5.9. Package semver tracks the bundle; skill changelogs carry their own history.

## 中文

Agentic UAT 是 AI 智能体时代的验收测试双技能方法学包：

- **acceptance-contract-governance**（v1.5.33）——把「重点测 XX」变成自包含可执行的验收任务书：AC-ID 化验证点（带 `[UI]`/`[MACHINE]` 入口判定路径）、场景建模、test basis 四活源（PRD 横切+缺陷台账+现行侦察+十一维加钻）、机验覆盖度量、收官闸
- **simulated-user-acceptance**（v3.5.9）——以真实用户身份执行任务书：persona 端到端旅程、有界交互控制流、UI 主证、L1/L2/L3 覆盖、受阻归因审计（UNREACHABLE/OMITTED/DEFERRED/N-A）、三态报告、产物树闸门

**不是什么**：不是测 agent 脚手架的框架，不是单测生成器——是「让 AI 执行体跑验收、人类只做裁判」的方法学。

### 两技能如何协作

分工总表（单源=governance 技能内 `references/skill-interface.md`）：**判据从哪来**（test basis/AC-ID/覆盖度量口径）归生成侧治理技能（coverage-metrics.md 唯一权威）；**产物长什么样**（报告三态/产物树/证据命名）归执行侧技能（artifact-conventions.md 唯一权威）；执行期格式冲突以执行侧为准并记偏差提交裁定；**生成单≠执行单**——只生成不执行、或不经评审直接执行，都会把未过闸的深度固化进执行。

典型流程：你说一句意图（「重点测 XX 模块」/「全项目深度测一遍」）→ 治理技能产出自包含任务书（四活源素材+场景+AC-ID 验收点+门槛标注）→ 任意具备任务注入能力的执行体**拿着任务书**按执行侧纪律跑验收（persona 真实入口/UI 主证/受阻归因/三态报告）→ 报告落产物树 → 经验回灌项目侧知识库、新验收点递单回流生成侧。两侧各一句话即可触发，其余一切信息都在任务书内自包含。

### 安装

路由 A（HTTP catalog，原生零拷贝）/ 路由 B（npm）/ 路由 C（git）见上方 English 节，三种方式等价。

### 许可

MIT © linjz
