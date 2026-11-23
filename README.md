# Agentic UAT

**UAT by AI agents — 驾驭 AI 执行体完成验收测试，人类只做裁判。**

[English](#english) | [中文](#中文)

---

## English

Agentic UAT is a two-skill methodology package for the AI-agent era of acceptance testing:

- **acceptance-contract-governance** (v1.5.33) — turns "test feature X thoroughly" into a self-contained, executable acceptance-instruction document: AC-ID'd verification points with entry paths (`[UI]`/`[MACHINE]`), scenario modeling, four live sources of test basis (PRD cross-cut + defect ledger + live UI/endpoint recon + eleven-dimension depth drill), machine-checkable coverage metrics, and a final-report gate.
- **simulated-user-acceptance** (v3.5.9) — executes those instructions as real users: persona-based end-to-end journeys, bounded interaction control flow, UI-primary evidence, L1/L2/L3 coverage, blocked-attribution auditing (UNREACHABLE/OMITTED/DEFERRED/N-A), three-state reporting (FINAL / FINAL-WITH-GAPS / INTERIM), and artifact-tree gates.

**What it is not**: this is not a testing framework for agent harnesses, and not a unit-test generator. It is a *methodology* for letting AI agents do the walking of user-acceptance testing while humans keep judgment.

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

Then invoke: "重点测 XX 模块" routes to the governance skill (task-book generation); executing the book routes to the execution skill. Any OpenCode-compatible agent host works.

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

### 安装

路由 A（HTTP catalog，原生零拷贝）/ 路由 B（npm）/ 路由 C（git）见上方 English 节，三种方式等价。

### 许可

MIT © linjz
